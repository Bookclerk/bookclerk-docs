# bookclerk_plugin_sdk::net

Mediated TCP sockets for native guests (Workers `connect()` equivalent).

Native plugins must not call `socket(AF_INET)` / `connect` themselves. The
guest jail denies ambient internet; this module speaks HTTP CONNECT through
`BOOKCLERK_SOCKET_PROXY`, which applies the same
[`bookclerk_plugin_manifest::EgressPolicy`](../bookclerk_plugin_manifest/egress.md#egresspolicy) as workerd `fetch()`/`connect()`.

Production native-behind-workerd sets `fd:<n>` or `handle:<n>` and multiplexes
CONNECT streams over that inherited link ([`crate::mux`](mux.md)). A per-session
challenge ([`SESSION_CHALLENGE_ENV`](#session_challenge_env)) is written before the first mux frame
so a numeric descriptor is not an identity. Pathname, `abstract:`, and
`\\.\pipe\` forms remain for tests and older launchers. Native plugins must
be rebuilt against this SDK; there is no ambient TCP fallback.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SocketAddress`](#socketaddress) | struct | Destination for `connect`. |
| [`ConnectOptions`](#connectoptions) | struct | Options for `connect`. |
| [`PluginSocket`](#pluginsocket) | struct | Bidirectional stream returned by `connect`. |
| [`ProxyStream`](#proxystream) | enum | Byte stream to the host CONNECT proxy (past the HTTP handshake). |
| [`SecureTransport`](#securetransport) | enum | Cloudflare-compatible `secureTransport` values. |
| [`nested_native_jail_requested`](#nested_native_jail_requested) | function | True when `NESTED_NATIVE_JAIL_ENV` is `1`. |
| [`connect`](#connect) | function | Open a TCP connection through the Bookclerk socket proxy. |
| [`SOCKET_PROXY_ENV`](#socket_proxy_env) | constant | Env var set by `bookclerk-workerd` for native-behind-workerd guests. |
| [`SOCKET_PROXY_WRITE_ENV`](#socket_proxy_write_env) | constant | Windows write half of the inherited socket-proxy mux (`handle:<n>`). |
| [`NESTED_NATIVE_JAIL_ENV`](#nested_native_jail_env) | constant | Optional fail-closed flag: deny ambient TCP when `SOCKET_PROXY_ENV` is unset. |
| [`GUEST_IPC_DIR_ENV`](#guest_ipc_dir_env) | constant | Host-created directory for guest pathname sockets. |
| [`SESSION_CHALLENGE_ENV`](#session_challenge_env) | constant | Hex-encoded 32-byte secret for the inherited proxy mux. |
| [`SESSION_CHALLENGE_LEN`](#session_challenge_len) | constant | Length of `SESSION_CHALLENGE_ENV` before hex encoding. |
| [`TEST_EXTRA_ENDPOINT_ENV`](#test_extra_endpoint_env) | constant | Test-only extra endpoint inherited beside `SOCKET_PROXY_ENV`. |
| [`TEST_EXTRA_ENDPOINT_WRITE_ENV`](#test_extra_endpoint_write_env) | constant | Windows write half of `TEST_EXTRA_ENDPOINT_ENV` (`handle:<n>`). |
| [`TEST_EXTRA_CANDIDATE_ENV`](#test_extra_candidate_env) | constant | Test-only identity of the candidate endpoint slot to classify. |
| [`TEST_EXTRA_CANDIDATE_WRITE_ENV`](#test_extra_candidate_write_env) | constant | Windows write half of `TEST_EXTRA_CANDIDATE_ENV` (`handle:<n>`). |
| [`SOCKET_PROXY_ABSTRACT_PREFIX`](#socket_proxy_abstract_prefix) | constant | Prefix for `SOCKET_PROXY_ENV` when the proxy is a Linux abstract socket. |

## Structs

### SocketAddress

```rust
pub struct SocketAddress {
    pub hostname: String,
    pub port: u16,
}
```

Destination for [`connect`](#connect).

**Fields**

- `hostname`: Hostname or IP literal.
- `port`: Destination port.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ConnectOptions

```rust
pub struct ConnectOptions {
    pub secure_transport: SecureTransport,
    pub allow_half_open: bool,
}
```

Options for [`connect`](#connect).

**Fields**

- `secure_transport`: TLS mode.
- `allow_half_open`: Accepted for API compatibility; half-close is always allowed.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### PluginSocket

```rust
pub struct PluginSocket {
    /* private fields */
}
```

Bidirectional stream returned by [`connect`](#connect).

#### PluginSocket::close

```rust
pub async fn close(self) -> Result<()>
```

Close both directions.

##### Errors

Returns [`SdkError`](index.md#sdkerror) when shutdown fails.

#### PluginSocket::stream

```rust
pub fn stream(&mut self) -> &mut ProxyStream
```

Borrow the proxied stream (past CONNECT).

#### PluginSocket::into_split

```rust
pub fn into_split(self) -> Result<(OwnedReadHalf, OwnedWriteHalf)>
```

Split into owned reader/writer halves (pathname Unix sockets only).

Production native plugins use the mux `fd:` / `handle:` link and
[`Self::into_stream`](#pluginsocketinto_stream). This does not open a TCP socket.

##### Errors

Returns an unsupported-transport error when the proxy is a multiplexed
`fd:` / `handle:` link.

#### PluginSocket::into_stream

```rust
pub fn into_stream(self) -> ProxyStream
```

Consumes the socket, returning the CONNECT-established proxy stream.

#### PluginSocket::into_unix_stream

```rust
pub fn into_unix_stream(self) -> Result<UnixStream>
```

Consumes the socket, returning the CONNECT-established Unix stream.

Production native plugins use [`Self::into_stream`](#pluginsocketinto_stream). This does not open
a TCP socket when the proxy is a mux link.

##### Errors

Returns an unsupported-transport error when the proxy is a multiplexed
`fd:` / `handle:` link.

#### PluginSocket::start_tls

```rust
pub async fn start_tls(&mut self) -> Result<()>
```

Start TLS on a `starttls` socket.

The proxy is byte-transparent, so TLS is end-to-end. Wrap
[`Self::into_stream`](#pluginsocketinto_stream) with `tokio-rustls` (same pattern as workerd
`socket.startTls()` after `secureTransport: "starttls"`). Pathname
sockets may use [`Self::into_split`](#pluginsocketinto_split). Mux links have no TCP fallback.

##### Errors

Returns [`SdkError`](index.md#sdkerror) when `secureTransport` was not `starttls`.

## Enums

### ProxyStream

```rust
pub enum ProxyStream {
    Unix(UnixStream),
    Mux(MuxStream),
}
```

Byte stream to the host CONNECT proxy (past the HTTP handshake).

**Variants**

- `Unix`: Pathname / abstract Unix socket (tests and older launchers).
- `Mux`: Inherited `fd:` / `handle:` multiplexed link.

Implements: `AsyncRead`, `AsyncWrite`

### SecureTransport

```rust
pub enum SecureTransport {
    Off,
    On,
    StartTls,
}
```

Cloudflare-compatible `secureTransport` values.

**Variants**

- `Off`: Plain TCP (default).
- `On`: TLS from the first byte. Authors wrap the stream with rustls after CONNECT.
- `StartTls`: Plain TCP until [`PluginSocket::start_tls`](#pluginsocketstart_tls).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

## Functions

### nested_native_jail_requested

```rust
pub fn nested_native_jail_requested() -> bool
```

True when [`NESTED_NATIVE_JAIL_ENV`](#nested_native_jail_env) is `1`.

### connect

```rust
pub async fn connect(
    address: SocketAddress,
    options: ConnectOptions,
) -> Result<PluginSocket>
```

Open a TCP connection through the Bookclerk socket proxy.

Mirrors `import { connect } from "cloudflare:sockets"`.

#### Errors

Returns [`SdkError`](index.md#sdkerror) when the proxy env is missing, CONNECT is denied, or
I/O fails.

## Constants

### SOCKET_PROXY_ENV

```rust
pub const SOCKET_PROXY_ENV: &str = "BOOKCLERK_SOCKET_PROXY";
```

Env var set by `bookclerk-workerd` for native-behind-workerd guests.

### SOCKET_PROXY_WRITE_ENV

```rust
pub const SOCKET_PROXY_WRITE_ENV: &str = "BOOKCLERK_SOCKET_PROXY_WRITE";
```

Windows write half of the inherited socket-proxy mux (`handle:<n>`).

Unset on Unix and in tests that pass one duplex `handle:`. Product spawns
set it so the guest does not read and write the same named pipe.

### NESTED_NATIVE_JAIL_ENV

```rust
pub const NESTED_NATIVE_JAIL_ENV: &str = "BOOKCLERK_NESTED_NATIVE_JAIL";
```

Optional fail-closed flag: deny ambient TCP when [`SOCKET_PROXY_ENV`](#socket_proxy_env) is unset.

The host no longer sets this (sibling Deny + inherited `SOCKET_PROXY` is
the production contract). SDK HTTP and native TCP still fail closed when
this is `1` without a proxy, so tests and older launchers cannot fall
through to `socket(AF_INET)`.

### GUEST_IPC_DIR_ENV

```rust
pub const GUEST_IPC_DIR_ENV: &str = "BOOKCLERK_GUEST_IPC_DIR";
```

Host-created directory for guest pathname sockets.

The native-behind-workerd guest may bind and connect Unix sockets only in
this directory (OAuth callback and the macOS Postgres mediator). Linux
Postgres still splices through `/proc/self/fd`. Unset on Windows.

### SESSION_CHALLENGE_ENV

```rust
pub const SESSION_CHALLENGE_ENV: &str = "BOOKCLERK_SESSION_CHALLENGE";
```

Hex-encoded 32-byte secret for the inherited proxy mux.

The guest writes these bytes before any mux frame. The host accepts the
link only when they match the secret it put in this process's environment.
An unrelated child, or another session's guest, does not have this value.
A numeric `fd:` / `handle:` is not an identity across processes.

### SESSION_CHALLENGE_LEN

```rust
pub const SESSION_CHALLENGE_LEN: usize = 32;
```

Length of [`SESSION_CHALLENGE_ENV`](#session_challenge_env) before hex encoding.

### TEST_EXTRA_ENDPOINT_ENV

```rust
pub const TEST_EXTRA_ENDPOINT_ENV: &str = "BOOKCLERK_TEST_EXTRA_ENDPOINT";
```

Test-only extra endpoint inherited beside [`SOCKET_PROXY_ENV`](#socket_proxy_env).

Unix value is `fd:<n>`. Windows value is `handle:<n>`, with the write half
in [`TEST_EXTRA_ENDPOINT_WRITE_ENV`](#test_extra_endpoint_write_env). The launcher sets these only when it
actually transferred the endpoint. Unset is not an observation: the guest
classifies the slot named by [`TEST_EXTRA_CANDIDATE_ENV`](#test_extra_candidate_env) instead, and
reports `not-run` when that metadata is missing. The guest does not write
[`SESSION_CHALLENGE_ENV`](#session_challenge_env) to the extra slot: the fixture authenticates the
endpoint with its own challenge before the guest opens the mux.

### TEST_EXTRA_ENDPOINT_WRITE_ENV

```rust
pub const TEST_EXTRA_ENDPOINT_WRITE_ENV: &str = "BOOKCLERK_TEST_EXTRA_ENDPOINT_WRITE";
```

Windows write half of [`TEST_EXTRA_ENDPOINT_ENV`](#test_extra_endpoint_env) (`handle:<n>`).

### TEST_EXTRA_CANDIDATE_ENV

```rust
pub const TEST_EXTRA_CANDIDATE_ENV: &str = "BOOKCLERK_TEST_EXTRA_CANDIDATE";
```

Test-only identity of the candidate endpoint slot to classify.

Unix value is `fd:<n>`. Windows value is `handle:<n>`, with the write half
in [`TEST_EXTRA_CANDIDATE_WRITE_ENV`](#test_extra_candidate_write_env). This names the slot. It does not
grant the launcher permission to transfer the endpoint. Missing metadata
is `not-run`, not proof the endpoint is absent. A live object at the slot
that is not the candidate is an identification failure, not absence.

### TEST_EXTRA_CANDIDATE_WRITE_ENV

```rust
pub const TEST_EXTRA_CANDIDATE_WRITE_ENV: &str = "BOOKCLERK_TEST_EXTRA_CANDIDATE_WRITE";
```

Windows write half of [`TEST_EXTRA_CANDIDATE_ENV`](#test_extra_candidate_env) (`handle:<n>`).

### SOCKET_PROXY_ABSTRACT_PREFIX

```rust
pub const SOCKET_PROXY_ABSTRACT_PREFIX: &str = "abstract:";
```

Prefix for [`SOCKET_PROXY_ENV`](#socket_proxy_env) when the proxy is a Linux abstract socket.
