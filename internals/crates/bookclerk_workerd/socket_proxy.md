# bookclerk_workerd::socket_proxy

Host-mediated TCP proxy for native-behind-workerd guests.

Native plugins must not open ambient `AF_INET` sockets. They speak HTTP
CONNECT over an inherited multiplexed link ([`spawn_link`](#spawn_link)); the launcher
applies the same [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#egresspolicy) as workerd `fetch()`/`connect()` and
splices bytes. [`spawn_unix`](#spawn_unix) remains for unit tests that bind a pathname
listener.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ProxyServer`](#proxyserver) | struct | In-flight host CONNECT proxy. Dropping it cancels accepts and handlers. |
| [`spawn_unix`](#spawn_unix) | function | Serve HTTP CONNECT on a Unix listener until the task is cancelled. |
| [`spawn_link`](#spawn_link) | function | Serve HTTP CONNECT on mux-accepted streams from an inherited sibling link. |
| [`spawn_link_with_challenge`](#spawn_link_with_challenge) | function | `spawn_link` that reads a 32-byte session challenge before any mux frame. |
| [`spawn_halves`](#spawn_halves) | function | Serve HTTP CONNECT on mux halves that are already separate pipes. |
| [`spawn_halves_with_challenge`](#spawn_halves_with_challenge) | function | `spawn_halves` that requires `challenge` on the read half before mux frames. |
| [`spawn_link_with_challenge_tag`](#spawn_link_with_challenge_tag) | function | `spawn_link_with_challenge` that answers `TEST_CHANNEL_IDENT_HOST`. |
| [`spawn_halves_with_challenge_tag`](#spawn_halves_with_challenge_tag) | function | `spawn_halves_with_challenge` that answers `TEST_CHANNEL_IDENT_HOST`. |
| [`parse_authority`](#parse_authority) | function | Parses `host:port` or `[ipv6]:port`. |
| [`resolved_addresses_denied`](#resolved_addresses_denied) | function | True when every resolved address for `host` is denied by `policy`. |
| [`SOCKET_PROXY_ENV`](#socket_proxy_env) | constant | Env var naming the socket proxy path for the native SDK. |
| [`TEST_CHANNEL_IDENT_HOST`](#test_channel_ident_host) | constant | Reserved CONNECT host answered with the session channel tag. |
| [`TEST_CHANNEL_IDENT_PORT`](#test_channel_ident_port) | constant | Port paired with `TEST_CHANNEL_IDENT_HOST`. |

## Structs

### ProxyServer

```rust
pub struct ProxyServer {
    /* private fields */
}
```

In-flight host CONNECT proxy. Dropping it cancels accepts and handlers.

The [`Self::fence`](#proxyserverfence) flag is the session cancel token. Authority revocation
sets that flag directly; handlers race it against parsing, DNS, dial, and
the byte relay.

#### ProxyServer::fence

```rust
pub fn fence(&self) -> Arc<AtomicBool>
```

Cancel flag shared with the session owner.

#### ProxyServer::cancel

```rust
pub fn cancel(&self)
```

Signal handlers to stop. In-flight work selects on [`Self::fence`](#proxyserverfence).

#### ProxyServer::unfinished_handlers

```rust
pub fn unfinished_handlers(&self) -> usize
```

Handler tasks that have not finished.

Implements: `Drop`

## Functions

### spawn_unix

```rust
pub fn spawn_unix(
    listener: UnixListener,
    policy: EgressPolicy,
    fence: Arc<AtomicBool>,
) -> Result<ProxyServer>
```

Serve HTTP CONNECT on a Unix listener until the task is cancelled.

#### Errors

Returns an error when the listener cannot be marked non-blocking or
converted to a Tokio listener.

### spawn_link

```rust
pub fn spawn_link<S>(
    link: S,
    policy: EgressPolicy,
    fence: Arc<AtomicBool>,
) -> Result<ProxyServer>
where
    S: AsyncRead + AsyncWrite + Send + Unpin + 'static,
```

Serve HTTP CONNECT on mux-accepted streams from an inherited sibling link.

#### Errors

Returns an error only if the accept task cannot be spawned (it does not).

### spawn_link_with_challenge

```rust
pub fn spawn_link_with_challenge<S>(
    link: S,
    policy: EgressPolicy,
    fence: Arc<AtomicBool>,
    challenge: [u8; 32],
) -> Result<ProxyServer>
where
    S: AsyncRead + AsyncWrite + Send + Unpin + 'static,
```

[`spawn_link`](#spawn_link) that reads a 32-byte session challenge before any mux frame.

A mismatch, EOF, or fence closes the link and serves nothing. The numeric
descriptor is not treated as the session identity.

#### Errors

Returns an error only if the accept task cannot be spawned (it does not).

### spawn_halves

```rust
pub fn spawn_halves<R, W>(
    reader: R,
    writer: W,
    policy: EgressPolicy,
    fence: Arc<AtomicBool>,
) -> Result<ProxyServer>
where
    R: AsyncRead + Send + Unpin + 'static,
    W: AsyncWrite + Send + Unpin + 'static,
```

Serve HTTP CONNECT on mux halves that are already separate pipes.

Windows product spawns pass two unidirectional handles. Splitting one
duplex pipe deadlocks the mux when a read and a write are in flight.

#### Errors

Returns an error only if the accept task cannot be spawned (it does not).

### spawn_halves_with_challenge

```rust
pub fn spawn_halves_with_challenge<R, W>(
    reader: R,
    writer: W,
    policy: EgressPolicy,
    fence: Arc<AtomicBool>,
    challenge: [u8; 32],
) -> Result<ProxyServer>
where
    R: AsyncRead + Send + Unpin + 'static,
    W: AsyncWrite + Send + Unpin + 'static,
```

[`spawn_halves`](#spawn_halves) that requires `challenge` on the read half before mux frames.

#### Errors

Returns an error only if the accept task cannot be spawned (it does not).

### spawn_link_with_challenge_tag

```rust
pub fn spawn_link_with_challenge_tag<S>(
    link: S,
    policy: EgressPolicy,
    fence: Arc<AtomicBool>,
    challenge: [u8; 32],
    channel_tag: &str,
) -> Result<ProxyServer>
where
    S: AsyncRead + AsyncWrite + Send + Unpin + 'static,
```

[`spawn_link_with_challenge`](#spawn_link_with_challenge) that answers [`TEST_CHANNEL_IDENT_HOST`](#test_channel_ident_host).

The tag is not a privilege. CONNECT to that host returns it and does not
dial. Every other target still uses `policy`. Production calls
[`spawn_link_with_challenge`](#spawn_link_with_challenge), which passes no tag.

#### Errors

Returns an error when `channel_tag` is empty, longer than 64 bytes, or
contains a byte other than ASCII alphanumeric, `-`, or `_`, or when the
accept task cannot be spawned.

### spawn_halves_with_challenge_tag

```rust
pub fn spawn_halves_with_challenge_tag<R, W>(
    reader: R,
    writer: W,
    policy: EgressPolicy,
    fence: Arc<AtomicBool>,
    challenge: [u8; 32],
    channel_tag: &str,
) -> Result<ProxyServer>
where
    R: AsyncRead + Send + Unpin + 'static,
    W: AsyncWrite + Send + Unpin + 'static,
```

[`spawn_halves_with_challenge`](#spawn_halves_with_challenge) that answers [`TEST_CHANNEL_IDENT_HOST`](#test_channel_ident_host).

#### Errors

Returns an error when `channel_tag` is not a safe token, or when the accept
task cannot be spawned. See [`spawn_link_with_challenge_tag`](#spawn_link_with_challenge_tag).

### parse_authority

```rust
pub fn parse_authority(target: &str) -> Result<(String, u16)>
```

Parses `host:port` or `[ipv6]:port`.

#### Errors

Returns an error when the target is missing a port or the port is not a
`u16`.

### resolved_addresses_denied

```rust
pub fn resolved_addresses_denied(policy: &EgressPolicy, ips: &[IpAddr]) -> bool
```

True when every resolved address for `host` is denied by `policy`.

## Constants

### SOCKET_PROXY_ENV

```rust
pub const SOCKET_PROXY_ENV: &str = "BOOKCLERK_SOCKET_PROXY";
```

Env var naming the socket proxy path for the native SDK.

### TEST_CHANNEL_IDENT_HOST

```rust
pub const TEST_CHANNEL_IDENT_HOST: &str = "bookclerk-test-channel";
```

Reserved CONNECT host answered with the session channel tag.

Production proxies pass no tag, so this host is an ordinary denied dial.
A test sets the tag on the host proxy only. The guest is not given the tag
in its environment.

### TEST_CHANNEL_IDENT_PORT

```rust
pub const TEST_CHANNEL_IDENT_PORT: u16 = 1;
```

Port paired with [`TEST_CHANNEL_IDENT_HOST`](#test_channel_ident_host).
