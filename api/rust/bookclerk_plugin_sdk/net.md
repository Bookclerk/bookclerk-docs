**bookclerk_plugin_sdk > net**

# Module: net

## Contents

**Structs**

- [`ConnectOptions`](#connectoptions) - Options for [`connect`](net.md#bookclerk_plugin_sdknetconnect).
- [`PluginSocket`](#pluginsocket) - Bidirectional stream returned by [`connect`](net.md#bookclerk_plugin_sdknetconnect).
- [`SocketAddress`](#socketaddress) - Destination for [`connect`](net.md#bookclerk_plugin_sdknetconnect).

**Enums**

- [`ProxyStream`](#proxystream) - Byte stream to the host CONNECT proxy (past the HTTP handshake).
- [`SecureTransport`](#securetransport) - Cloudflare-compatible `secureTransport` values.

**Functions**

- [`connect`](#connect) - Open a TCP connection through the Bookclerk socket proxy.
- [`nested_native_jail_requested`](#nested_native_jail_requested) - True when [`NESTED_NATIVE_JAIL_ENV`](net.md#bookclerk_plugin_sdknetnested_native_jail_env) is `1`.

**Constants**

- [`GUEST_IPC_DIR_ENV`](#guest_ipc_dir_env) - Host-created directory for guest pathname sockets.
- [`NESTED_NATIVE_JAIL_ENV`](#nested_native_jail_env) - Optional fail-closed flag: deny ambient TCP when [`SOCKET_PROXY_ENV`](net.md#bookclerk_plugin_sdknetsocket_proxy_env) is unset.
- [`SESSION_CHALLENGE_ENV`](#session_challenge_env) - Hex-encoded 32-byte secret for the inherited proxy mux.
- [`SESSION_CHALLENGE_LEN`](#session_challenge_len) - Length of [`SESSION_CHALLENGE_ENV`](net.md#bookclerk_plugin_sdknetsession_challenge_env) before hex encoding.
- [`SOCKET_PROXY_ABSTRACT_PREFIX`](#socket_proxy_abstract_prefix) - Prefix for [`SOCKET_PROXY_ENV`](net.md#bookclerk_plugin_sdknetsocket_proxy_env) when the proxy is a Linux abstract socket.
- [`SOCKET_PROXY_ENV`](#socket_proxy_env) - Env var set by `bookclerk-workerd` for native-behind-workerd guests.
- [`SOCKET_PROXY_WRITE_ENV`](#socket_proxy_write_env) - Windows write half of the inherited socket-proxy mux (`handle:<n>`).
- [`TEST_EXTRA_CANDIDATE_ENV`](#test_extra_candidate_env) - Test-only identity of the candidate endpoint slot to classify.
- [`TEST_EXTRA_CANDIDATE_WRITE_ENV`](#test_extra_candidate_write_env) - Windows write half of [`TEST_EXTRA_CANDIDATE_ENV`](net.md#bookclerk_plugin_sdknettest_extra_candidate_env) (`handle:<n>`).
- [`TEST_EXTRA_ENDPOINT_ENV`](#test_extra_endpoint_env) - Test-only extra endpoint inherited beside [`SOCKET_PROXY_ENV`](net.md#bookclerk_plugin_sdknetsocket_proxy_env).
- [`TEST_EXTRA_ENDPOINT_WRITE_ENV`](#test_extra_endpoint_write_env) - Windows write half of [`TEST_EXTRA_ENDPOINT_ENV`](net.md#bookclerk_plugin_sdknettest_extra_endpoint_env) (`handle:<n>`).

---

## bookclerk_plugin_sdk::net::ConnectOptions

*Struct*

Options for [`connect`](net.md#bookclerk_plugin_sdknetconnect).

**Fields:**
- `secure_transport: SecureTransport` - TLS mode.
- `allow_half_open: bool` - Accepted for API compatibility; half-close is always allowed.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ConnectOptions) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ConnectOptions`
- **Default**
  - `fn default() -> ConnectOptions`



## bookclerk_plugin_sdk::net::GUEST_IPC_DIR_ENV

*Constant*: `&str`

Host-created directory for guest pathname sockets.

The native-behind-workerd guest may bind and connect Unix sockets only in
this directory (OAuth callback and the macOS Postgres mediator). Linux
Postgres still splices through `/proc/self/fd`. Unset on Windows.



## bookclerk_plugin_sdk::net::NESTED_NATIVE_JAIL_ENV

*Constant*: `&str`

Optional fail-closed flag: deny ambient TCP when [`SOCKET_PROXY_ENV`](net.md#bookclerk_plugin_sdknetsocket_proxy_env) is unset.

The host no longer sets this (sibling Deny + inherited `SOCKET_PROXY` is
the production contract). SDK HTTP and native TCP still fail closed when
this is `1` without a proxy, so tests and older launchers cannot fall
through to `socket(AF_INET)`.



## bookclerk_plugin_sdk::net::PluginSocket

*Struct*

Bidirectional stream returned by [`connect`](net.md#bookclerk_plugin_sdknetconnect).

**Methods:**

- `fn close(self: Self) -> Result<()>` - Close both directions.
- `fn stream(self: & mut Self) -> & mut ProxyStream` - Borrow the proxied stream (past CONNECT).
- `fn into_split(self: Self) -> Result<(tokio::net::unix::OwnedReadHalf, tokio::net::unix::OwnedWriteHalf)>` - Split into owned reader/writer halves (pathname Unix sockets only).
- `fn into_stream(self: Self) -> ProxyStream` - Consumes the socket, returning the CONNECT-established proxy stream.
- `fn into_unix_stream(self: Self) -> Result<tokio::net::UnixStream>` - Consumes the socket, returning the CONNECT-established Unix stream.
- `fn start_tls(self: & mut Self) -> Result<()>` - Start TLS on a `starttls` socket.



## bookclerk_plugin_sdk::net::ProxyStream

*Enum*

Byte stream to the host CONNECT proxy (past the HTTP handshake).

**Variants:**
- `Unix(tokio::net::UnixStream)` - Pathname / abstract Unix socket (tests and older launchers).
- `Mux(crate::mux::MuxStream)` - Inherited `fd:` / `handle:` multiplexed link.

**Trait Implementations:**

- **AsyncWrite**
  - `fn poll_write(self: std::pin::Pin<& mut Self>, cx: & mut std::task::Context, buf: &[u8]) -> std::task::Poll<std::io::Result<usize>>`
  - `fn poll_flush(self: std::pin::Pin<& mut Self>, cx: & mut std::task::Context) -> std::task::Poll<std::io::Result<()>>`
  - `fn poll_shutdown(self: std::pin::Pin<& mut Self>, cx: & mut std::task::Context) -> std::task::Poll<std::io::Result<()>>`
- **AsyncRead**
  - `fn poll_read(self: std::pin::Pin<& mut Self>, cx: & mut std::task::Context, buf: & mut tokio::io::ReadBuf) -> std::task::Poll<std::io::Result<()>>`



## bookclerk_plugin_sdk::net::SESSION_CHALLENGE_ENV

*Constant*: `&str`

Hex-encoded 32-byte secret for the inherited proxy mux.

The guest writes these bytes before any mux frame. The host accepts the
link only when they match the secret it put in this process's environment.
An unrelated child, or another session's guest, does not have this value.
A numeric `fd:` / `handle:` is not an identity across processes.



## bookclerk_plugin_sdk::net::SESSION_CHALLENGE_LEN

*Constant*: `usize`

Length of [`SESSION_CHALLENGE_ENV`](net.md#bookclerk_plugin_sdknetsession_challenge_env) before hex encoding.



## bookclerk_plugin_sdk::net::SOCKET_PROXY_ABSTRACT_PREFIX

*Constant*: `&str`

Prefix for [`SOCKET_PROXY_ENV`](net.md#bookclerk_plugin_sdknetsocket_proxy_env) when the proxy is a Linux abstract socket.



## bookclerk_plugin_sdk::net::SOCKET_PROXY_ENV

*Constant*: `&str`

Env var set by `bookclerk-workerd` for native-behind-workerd guests.



## bookclerk_plugin_sdk::net::SOCKET_PROXY_WRITE_ENV

*Constant*: `&str`

Windows write half of the inherited socket-proxy mux (`handle:<n>`).

Unset on Unix and in tests that pass one duplex `handle:`. Product spawns
set it so the guest does not read and write the same named pipe.



## bookclerk_plugin_sdk::net::SecureTransport

*Enum*

Cloudflare-compatible `secureTransport` values.

**Variants:**
- `Off` - Plain TCP (default).
- `On` - TLS from the first byte. Authors wrap the stream with rustls after CONNECT.
- `StartTls` - Plain TCP until `PluginSocket::start_tls`.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> SecureTransport`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SecureTransport) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> SecureTransport`



## bookclerk_plugin_sdk::net::SocketAddress

*Struct*

Destination for [`connect`](net.md#bookclerk_plugin_sdknetconnect).

**Fields:**
- `hostname: String` - Hostname or IP literal.
- `port: u16` - Destination port.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &SocketAddress) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> SocketAddress`



## bookclerk_plugin_sdk::net::TEST_EXTRA_CANDIDATE_ENV

*Constant*: `&str`

Test-only identity of the candidate endpoint slot to classify.

Unix value is `fd:<n>`. Windows value is `handle:<n>`, with the write half
in [`TEST_EXTRA_CANDIDATE_WRITE_ENV`](net.md#bookclerk_plugin_sdknettest_extra_candidate_write_env). This names the slot. It does not
grant the launcher permission to transfer the endpoint. Missing metadata
is `not-run`, not proof the endpoint is absent. A live object at the slot
that is not the candidate is an identification failure, not absence.



## bookclerk_plugin_sdk::net::TEST_EXTRA_CANDIDATE_WRITE_ENV

*Constant*: `&str`

Windows write half of [`TEST_EXTRA_CANDIDATE_ENV`](net.md#bookclerk_plugin_sdknettest_extra_candidate_env) (`handle:<n>`).



## bookclerk_plugin_sdk::net::TEST_EXTRA_ENDPOINT_ENV

*Constant*: `&str`

Test-only extra endpoint inherited beside [`SOCKET_PROXY_ENV`](net.md#bookclerk_plugin_sdknetsocket_proxy_env).

Unix value is `fd:<n>`. Windows value is `handle:<n>`, with the write half
in [`TEST_EXTRA_ENDPOINT_WRITE_ENV`](net.md#bookclerk_plugin_sdknettest_extra_endpoint_write_env). The launcher sets these only when it
actually transferred the endpoint. Unset is not an observation: the guest
classifies the slot named by [`TEST_EXTRA_CANDIDATE_ENV`](net.md#bookclerk_plugin_sdknettest_extra_candidate_env) instead, and
reports `not-run` when that metadata is missing. The guest does not write
[`SESSION_CHALLENGE_ENV`](net.md#bookclerk_plugin_sdknetsession_challenge_env) to the extra slot: the fixture authenticates the
endpoint with its own challenge before the guest opens the mux.



## bookclerk_plugin_sdk::net::TEST_EXTRA_ENDPOINT_WRITE_ENV

*Constant*: `&str`

Windows write half of [`TEST_EXTRA_ENDPOINT_ENV`](net.md#bookclerk_plugin_sdknettest_extra_endpoint_env) (`handle:<n>`).



## bookclerk_plugin_sdk::net::connect

*Function*

Open a TCP connection through the Bookclerk socket proxy.

Mirrors `import { connect } from "cloudflare:sockets"`.

# Errors

Returns [`SdkError`](error.md#bookclerk_plugin_sdkerrorsdkerror) when the proxy env is missing, CONNECT is denied, or
I/O fails.

```rust
fn connect(address: SocketAddress, options: ConnectOptions) -> crate::error::Result<PluginSocket>
```



## bookclerk_plugin_sdk::net::nested_native_jail_requested

*Function*

True when [`NESTED_NATIVE_JAIL_ENV`](net.md#bookclerk_plugin_sdknetnested_native_jail_env) is `1`.

```rust
fn nested_native_jail_requested() -> bool
```



