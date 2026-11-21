# bookclerk_workerd::egress_relay

The launcher's egress relay: a workerd isolate's only way out of its jail.

Every workerd jail is `NetPolicy::Deny`: no IPv4, IPv6 or packet sockets.
The generated config points the egress worker's `globalOutbound` at a
Unix socket (Windows: in-container loopback) this launcher owns, as a
workerd `external` service in HTTP proxy style. Workerd then sends:

- each `fetch()` as one absolute-form request (`GET https://host/path`),
  in plain text, because TLS to the origin is this relay's job; and
- each `connect()` socket as `CONNECT host:port`.

For either, the relay opens one stream on the host's `CONNECT` proxy link
(the same multiplexed link a native guest uses) and asks the host for a
tunnel to `host:port`. The host checks the tunnel against the plugin's
approved grant ([`crate::socket_proxy::TunnelRule::IsolateEgress`](socket_proxy.md#tunnelrule)),
resolves the name, checks every address, and dials. The relay then
originates TLS for `https` (rustls on the hosts' one provider, Mozilla's
roots), forwards the request in origin form, and splices bytes.

Nothing here widens what the plugin can reach: the host's check is the
boundary, and it runs outside the jail. The relay keeps TLS and HTTP inside
the jail so the unsandboxed host never parses an origin's response or
sees a plugin's plaintext.

One request per connection. The relay rewrites the request to
`connection: close` (or keeps `Upgrade` for WebSockets) and rewrites the
final response head to `connection: close`, so workerd never sends a
second request down a tunnel opened for another origin. Interim `1xx`
responses are forwarded and the next head is read, for the same reason.

Transport failures (a refused tunnel, a failed dial or TLS handshake)
close the workerd connection without a response, so the plugin's
`fetch()` rejects as it would on a lost connection, rather than resolving
to a synthetic status. The relay never logs URLs or headers, which carry
plugin tokens.

The parsing is sans-IO ([`parse_request_head`](#parse_request_head), [`parse_response_head`](#parse_response_head),
[`find_head_end`](#find_head_end)) so the `egress_relay_head` fuzz target and the unit
tests drive the same code the relay runs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`TunnelOpener`](#tunnelopener) | trait | Opens one stream on the host's proxy link. |
| [`TunnelTarget`](#tunneltarget) | struct | A tunnel target: host and port, as the host proxy's `CONNECT` line names them. |
| [`ResponseHead`](#responsehead) | struct | A parsed response head from an origin. |
| [`HeadError`](#headerror) | enum | Why a head was refused. |
| [`Scheme`](#scheme) | enum | URL scheme of a forwarded request. |
| [`RelayRequest`](#relayrequest) | enum | What workerd asked the relay to do with one connection. |
| [`find_head_end`](#find_head_end) | function | Byte offset just past the `\r\n\r\n` that ends a head, if `buf` holds one. |
| [`parse_request_head`](#parse_request_head) | function | Parses the head workerd sent and decides what to do with the connection. |
| [`parse_response_head`](#parse_response_head) | function | Parses a response head from an origin and rewrites a final one. |
| [`open_tunnel`](#open_tunnel) | function | Asks the host for a tunnel to `target` on a fresh stream. |
| [`host_link_from_env`](#host_link_from_env) | function | Opens the host's proxy link that the host delivered to this process. |
| [`serve`](#serve) | function | Accepts workerd connections and relays each through the host until the listener fails. |
| [`MAX_HEAD_BYTES`](#max_head_bytes) | constant | Largest request or response head the relay buffers, in bytes. |
| [`MAX_HEADER_LINES`](#max_header_lines) | constant | Most header lines the relay accepts in one head. |
| [`MAX_CONCURRENT_TUNNELS`](#max_concurrent_tunnels) | constant | Tunnels the relay keeps open at once. |

## Traits

### TunnelOpener

```rust
pub trait TunnelOpener: Send + Sync + 'static {
    type Stream: AsyncRead + AsyncWrite + Unpin + Send + 'static;
    fn open<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Self::Stream>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Opens one stream on the host's proxy link.

A trait so tests can stand in for the multiplexed link.

#### TunnelOpener::Stream

```rust
type Stream: AsyncRead + AsyncWrite + Unpin + Send + 'static
```

Byte stream the host proxy answers `CONNECT` on.

#### TunnelOpener::open

```rust
fn open<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Self::Stream>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Opens a fresh stream to the host proxy.

##### Errors

When the link is closed.

Implemented by: `Mux`

## Structs

### TunnelTarget

```rust
pub struct TunnelTarget {
    pub host: String,
    pub port: u16,
}
```

A tunnel target: host and port, as the host proxy's `CONNECT` line names
them.

**Fields**

- `host`: Lowercased host name or IP literal, without IPv6 brackets.
- `port`: Destination TCP port.

#### TunnelTarget::authority

```rust
pub fn authority(&self) -> String
```

`host:port` with IPv6 literals in brackets, for a `CONNECT` line.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ResponseHead

```rust
pub struct ResponseHead {
    pub status: u16,
    pub head: Vec<u8>,
}
```

A parsed response head from an origin.

**Fields**

- `status`: HTTP status code.
- `head`: The head to hand to workerd, ending in `\r\n\r\n`. Interim (`1xx`)
  and `101` heads are unchanged; a final head is rewritten to
  `connection: close`.

#### ResponseHead::is_interim

```rust
pub fn is_interim(&self) -> bool
```

Whether another head follows this one (an interim `1xx` other than
`101 Switching Protocols`).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### HeadError

```rust
pub enum HeadError {
    TooLarge,
    BadStartLine,
    BadHeader,
    BadTarget,
    NotText,
}
```

Why a head was refused.

**Variants**

- `TooLarge`: The head exceeds [`MAX_HEAD_BYTES`](#max_head_bytes) or [`MAX_HEADER_LINES`](#max_header_lines).
- `BadStartLine`: The request or status line is not well formed.
- `BadHeader`: A header line is not `name: value`, has an invalid name, or carries a
  stray CR or LF, or is an obsolete folded continuation.
- `BadTarget`: The request target is not absolute-form `http(s)://` or a `CONNECT`
  authority, or it carries userinfo or a fragment.
- `NotText`: The head is not valid UTF-8 where text is required.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### Scheme

```rust
pub enum Scheme {
    Http,
    Https,
}
```

URL scheme of a forwarded request.

**Variants**

- `Http`: Plain HTTP; the relay forwards bytes as they are.
- `Https`: HTTPS; the relay originates TLS to the origin.

#### Scheme::default_port

```rust
pub fn default_port(self) -> u16
```

Port a URL without an explicit port uses.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### RelayRequest

```rust
pub enum RelayRequest {
    Tunnel(TunnelTarget),
    Forward { scheme: Scheme, target: TunnelTarget, head: Vec<u8> },
}
```

What workerd asked the relay to do with one connection.

**Variants**

- `Tunnel`: A `connect()` socket: open a tunnel and splice raw bytes.
- `Forward`: A `fetch()` request: open a tunnel, add TLS for `https`, then send
  [`Self::Forward::head`](#relayrequest) followed by the request body as workerd sends
  it.
  - `scheme`: Scheme of the request URL.
  - `target`: Origin the request goes to.
  - `head`: The rewritten head in origin form, ending in `\r\n\r\n`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### find_head_end

```rust
pub fn find_head_end(buf: &[u8]) -> Option<usize>
```

Byte offset just past the `\r\n\r\n` that ends a head, if `buf` holds one.

#### Arguments

* `buf` - Bytes read so far.

#### Returns

`Some(n)` where `buf[..n]` is the head, or `None` when more bytes are
needed.

### parse_request_head

```rust
pub fn parse_request_head(head: &[u8]) -> Result<RelayRequest, HeadError>
```

Parses the head workerd sent and decides what to do with the connection.

#### Arguments

* `head` - The complete request head, through `\r\n\r\n`
  ([`find_head_end`](#find_head_end)).

#### Returns

[`RelayRequest::Tunnel`](#relayrequest) for `CONNECT host:port`, or
[`RelayRequest::Forward`](#relayrequest) with the head rewritten to origin form: `host`
set from the URL, proxy and hop-by-hop headers dropped, and `connection:
close` (or `connection: upgrade` when the request carries `upgrade`).

#### Errors

[`HeadError`](#headerror) when the head is malformed or the target is not an absolute
`http`/`https` URL or a `CONNECT` authority.

### parse_response_head

```rust
pub fn parse_response_head(head: &[u8]) -> Result<ResponseHead, HeadError>
```

Parses a response head from an origin and rewrites a final one.

#### Arguments

* `head` - The complete response head, through `\r\n\r\n`.

#### Returns

The status and the head to hand to workerd: interim and `101` heads
unchanged, a final head with its connection headers replaced by
`connection: close`.

#### Errors

[`HeadError`](#headerror) when the status line or a header is malformed.

### open_tunnel

```rust
pub async fn open_tunnel<O: TunnelOpener>(
    opener: &O,
    target: &TunnelTarget,
) -> Result<<O as >::Stream>
```

Asks the host for a tunnel to `target` on a fresh stream.

#### Arguments

* `opener` - Opens streams on the host's proxy link.
* `target` - Where the tunnel should go.

#### Returns

The stream, now carrying the tunnel's bytes.

#### Errors

When the stream cannot be opened, the host refuses (any status but 200),
or the host does not answer within 30 seconds.

### host_link_from_env

```rust
pub async fn host_link_from_env() -> Result<Option<Arc<Mux>>>
```

Opens the host's proxy link that the host delivered to this process.

The host passes the link in [`bookclerk_sandbox::WORKERD_EGRESS_ENV`](../bookclerk_sandbox/README.md#workerd_egress_env)
(plus [`bookclerk_sandbox::WORKERD_EGRESS_WRITE_ENV`](../bookclerk_sandbox/README.md#workerd_egress_write_env) on Windows) and the
session challenge in [`bookclerk_plugin_sdk::SESSION_CHALLENGE_ENV`](../../../plugins/sdk/rust/net.md#session_challenge_env). The
challenge is written first, then the link is multiplexed. The challenge is
removed from this process's environment once read, so a child (workerd)
never inherits it.

#### Returns

`None` when no link was delivered: the plugin's network is denied.

#### Errors

When the link spec or challenge is malformed, or writing the challenge
fails.

### serve

```rust
pub async fn serve<A, F, C, O>(
    accept: A,
    opener: Arc<O>,
    tls: Arc<ClientConfig>,
) -> Result<()>
where
    A: FnMut() -> F,
    F: Future<Output = Result<C>>,
    C: AsyncRead + AsyncWrite + Unpin + Send + 'static,
    O: TunnelOpener,
```

Accepts workerd connections and relays each through the host until the
listener fails.

#### Arguments

* `accept` - Yields the next workerd connection.
* `opener` - Opens streams on the host's proxy link.
* `tls` - Client TLS settings for `https` origins.

#### Errors

When accepting fails; individual connections fail on their own.

## Constants

### MAX_HEAD_BYTES

```rust
pub const MAX_HEAD_BYTES: usize = _;
```

Largest request or response head the relay buffers, in bytes.

Workers URLs may be up to 16 KiB; headers add more. Anything larger is
refused rather than buffered.

### MAX_HEADER_LINES

```rust
pub const MAX_HEADER_LINES: usize = 128;
```

Most header lines the relay accepts in one head.

### MAX_CONCURRENT_TUNNELS

```rust
pub const MAX_CONCURRENT_TUNNELS: usize = 16;
```

Tunnels the relay keeps open at once.

Matches the host proxy's per-link handler cap, so a burst of `fetch()`
calls waits here for a free tunnel instead of being dropped by the host.
