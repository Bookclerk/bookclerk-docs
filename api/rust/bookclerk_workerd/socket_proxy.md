**bookclerk_workerd > socket_proxy**

# Module: socket_proxy

## Contents

**Structs**

- [`ProxyServer`](#proxyserver) - In-flight host CONNECT proxy. Dropping it cancels accepts and handlers.

**Functions**

- [`parse_authority`](#parse_authority) - Parses `host:port` or `[ipv6]:port`.
- [`resolved_addresses_denied`](#resolved_addresses_denied) - True when every resolved address for `host` is denied by `policy`.
- [`spawn_halves`](#spawn_halves) - Serve HTTP CONNECT on mux halves that are already separate pipes.
- [`spawn_halves_with_challenge`](#spawn_halves_with_challenge) - [`spawn_halves`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_halves) that requires `challenge` on the read half before mux frames.
- [`spawn_halves_with_challenge_tag`](#spawn_halves_with_challenge_tag) - [`spawn_halves_with_challenge`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_halves_with_challenge) that answers [`TEST_CHANNEL_IDENT_HOST`](socket_proxy.md#bookclerk_workerdsocket_proxytest_channel_ident_host).
- [`spawn_link`](#spawn_link) - Serve HTTP CONNECT on mux-accepted streams from an inherited sibling link.
- [`spawn_link_with_challenge`](#spawn_link_with_challenge) - [`spawn_link`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_link) that reads a 32-byte session challenge before any mux frame.
- [`spawn_link_with_challenge_tag`](#spawn_link_with_challenge_tag) - [`spawn_link_with_challenge`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_link_with_challenge) that answers [`TEST_CHANNEL_IDENT_HOST`](socket_proxy.md#bookclerk_workerdsocket_proxytest_channel_ident_host).
- [`spawn_unix`](#spawn_unix) - Serve HTTP CONNECT on a Unix listener until the task is cancelled.

**Constants**

- [`SOCKET_PROXY_ENV`](#socket_proxy_env) - Env var naming the socket proxy path for the native SDK.
- [`TEST_CHANNEL_IDENT_HOST`](#test_channel_ident_host) - Reserved CONNECT host answered with the session channel tag.
- [`TEST_CHANNEL_IDENT_PORT`](#test_channel_ident_port) - Port paired with [`TEST_CHANNEL_IDENT_HOST`](socket_proxy.md#bookclerk_workerdsocket_proxytest_channel_ident_host).

---

## bookclerk_workerd::socket_proxy::ProxyServer

*Struct*

In-flight host CONNECT proxy. Dropping it cancels accepts and handlers.

The `Self::fence` flag is the session cancel token. Authority revocation
sets that flag directly; handlers race it against parsing, DNS, dial, and
the byte relay.

**Methods:**

- `fn fence(self: &Self) -> Arc<AtomicBool>` - Cancel flag shared with the session owner.
- `fn cancel(self: &Self)` - Signal handlers to stop. In-flight work selects on `Self::fence`.
- `fn unfinished_handlers(self: &Self) -> usize` - Handler tasks that have not finished.

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)`



## bookclerk_workerd::socket_proxy::SOCKET_PROXY_ENV

*Constant*: `&str`

Env var naming the socket proxy path for the native SDK.



## bookclerk_workerd::socket_proxy::TEST_CHANNEL_IDENT_HOST

*Constant*: `&str`

Reserved CONNECT host answered with the session channel tag.

Production proxies pass no tag, so this host is an ordinary denied dial.
A test sets the tag on the host proxy only. The guest is not given the tag
in its environment.



## bookclerk_workerd::socket_proxy::TEST_CHANNEL_IDENT_PORT

*Constant*: `u16`

Port paired with [`TEST_CHANNEL_IDENT_HOST`](socket_proxy.md#bookclerk_workerdsocket_proxytest_channel_ident_host).



## bookclerk_workerd::socket_proxy::parse_authority

*Function*

Parses `host:port` or `[ipv6]:port`.

# Errors

Returns an error when the target is missing a port or the port is not a
`u16`.

```rust
fn parse_authority(target: &str) -> anyhow::Result<(String, u16)>
```



## bookclerk_workerd::socket_proxy::resolved_addresses_denied

*Function*

True when every resolved address for `host` is denied by `policy`.

```rust
fn resolved_addresses_denied(policy: &bookclerk_plugin_manifest::EgressPolicy, ips: &[std::net::IpAddr]) -> bool
```



## bookclerk_workerd::socket_proxy::spawn_halves

*Function*

Serve HTTP CONNECT on mux halves that are already separate pipes.

Windows product spawns pass two unidirectional handles. Splitting one
duplex pipe deadlocks the mux when a read and a write are in flight.

# Errors

Returns an error only if the accept task cannot be spawned (it does not).

```rust
fn spawn_halves<R, W>(reader: R, writer: W, policy: bookclerk_plugin_manifest::EgressPolicy, fence: std::sync::Arc<std::sync::atomic::AtomicBool>) -> anyhow::Result<ProxyServer>
```



## bookclerk_workerd::socket_proxy::spawn_halves_with_challenge

*Function*

[`spawn_halves`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_halves) that requires `challenge` on the read half before mux frames.

# Errors

Returns an error only if the accept task cannot be spawned (it does not).

```rust
fn spawn_halves_with_challenge<R, W>(reader: R, writer: W, policy: bookclerk_plugin_manifest::EgressPolicy, fence: std::sync::Arc<std::sync::atomic::AtomicBool>, challenge: [u8; 32]) -> anyhow::Result<ProxyServer>
```



## bookclerk_workerd::socket_proxy::spawn_halves_with_challenge_tag

*Function*

[`spawn_halves_with_challenge`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_halves_with_challenge) that answers [`TEST_CHANNEL_IDENT_HOST`](socket_proxy.md#bookclerk_workerdsocket_proxytest_channel_ident_host).

# Errors

Returns an error when `channel_tag` is not a safe token, or when the accept
task cannot be spawned. See [`spawn_link_with_challenge_tag`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_link_with_challenge_tag).

```rust
fn spawn_halves_with_challenge_tag<R, W>(reader: R, writer: W, policy: bookclerk_plugin_manifest::EgressPolicy, fence: std::sync::Arc<std::sync::atomic::AtomicBool>, challenge: [u8; 32], channel_tag: &str) -> anyhow::Result<ProxyServer>
```



## bookclerk_workerd::socket_proxy::spawn_link

*Function*

Serve HTTP CONNECT on mux-accepted streams from an inherited sibling link.

# Errors

Returns an error only if the accept task cannot be spawned (it does not).

```rust
fn spawn_link<S>(link: S, policy: bookclerk_plugin_manifest::EgressPolicy, fence: std::sync::Arc<std::sync::atomic::AtomicBool>) -> anyhow::Result<ProxyServer>
```



## bookclerk_workerd::socket_proxy::spawn_link_with_challenge

*Function*

[`spawn_link`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_link) that reads a 32-byte session challenge before any mux frame.

A mismatch, EOF, or fence closes the link and serves nothing. The numeric
descriptor is not treated as the session identity.

# Errors

Returns an error only if the accept task cannot be spawned (it does not).

```rust
fn spawn_link_with_challenge<S>(link: S, policy: bookclerk_plugin_manifest::EgressPolicy, fence: std::sync::Arc<std::sync::atomic::AtomicBool>, challenge: [u8; 32]) -> anyhow::Result<ProxyServer>
```



## bookclerk_workerd::socket_proxy::spawn_link_with_challenge_tag

*Function*

[`spawn_link_with_challenge`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_link_with_challenge) that answers [`TEST_CHANNEL_IDENT_HOST`](socket_proxy.md#bookclerk_workerdsocket_proxytest_channel_ident_host).

The tag is not a privilege. CONNECT to that host returns it and does not
dial. Every other target still uses `policy`. Production calls
[`spawn_link_with_challenge`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_link_with_challenge), which passes no tag.

# Errors

Returns an error when `channel_tag` is empty, longer than 64 bytes, or
contains a byte other than ASCII alphanumeric, `-`, or `_`, or when the
accept task cannot be spawned.

```rust
fn spawn_link_with_challenge_tag<S>(link: S, policy: bookclerk_plugin_manifest::EgressPolicy, fence: std::sync::Arc<std::sync::atomic::AtomicBool>, challenge: [u8; 32], channel_tag: &str) -> anyhow::Result<ProxyServer>
```



## bookclerk_workerd::socket_proxy::spawn_unix

*Function*

Serve HTTP CONNECT on a Unix listener until the task is cancelled.

# Errors

Returns an error when the listener cannot be marked non-blocking or
converted to a Tokio listener.

```rust
fn spawn_unix(listener: std::os::unix::net::UnixListener, policy: bookclerk_plugin_manifest::EgressPolicy, fence: std::sync::Arc<std::sync::atomic::AtomicBool>) -> anyhow::Result<ProxyServer>
```



