**bookclerkd > node_api > transport**

# Module: node_api::transport

## Contents

**Structs**

- [`ConnAddrsMakeService`](#connaddrsmakeservice) - Make-service for `axum::serve` that records both ends of each accepted
- [`ConnAddrsService`](#connaddrsservice) - Per-connection service built by [`ConnAddrsMakeService`](transport.md#bookclerkdnode_apitransportconnaddrsmakeservice).
- [`ConnInfo`](#conninfo) - What is known about one connection.
- [`LocalAddr`](#localaddr) - The local address of the connection a request arrived on, attached as a
- [`TransportConfig`](#transportconfig) - Settings [`classify_transport`](transport.md#bookclerkdnode_apitransportclassify_transport) reads.

**Enums**

- [`AllowedMode`](#allowedmode) - Mode a connection was allowed in.
- [`ListenerKind`](#listenerkind) - Which listener accepted the connection. The built-in TLS listener
- [`RefusalReason`](#refusalreason) - Why a connection was refused.
- [`TransportDecision`](#transportdecision) - Outcome of [`classify_transport`](transport.md#bookclerkdnode_apitransportclassify_transport).

**Functions**

- [`classify_transport`](#classify_transport) - Classifies one node-API connection. Pure: no I/O, no clock.
- [`refused_response`](#refused_response) - The 403 answer for a refused connection.

---

## bookclerkd::node_api::transport::AllowedMode

*Enum*

Mode a connection was allowed in.

**Variants:**
- `Acme` - Built-in listener, ACME certificate.
- `Proxy` - Trusted TLS-terminating proxy.
- `SelfSigned` - Built-in listener, pinned self-signed certificate.
- `Loopback` - Same machine, no proxy.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AllowedMode`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &AllowedMode) -> bool`



## bookclerkd::node_api::transport::ConnAddrsMakeService

*Struct*

Make-service for `axum::serve` that records both ends of each accepted
TCP connection: the peer as `ConnectInfo<SocketAddr>` (as
`into_make_service_with_connect_info` would) and the socket's local
address as [`LocalAddr`](transport.md#bookclerkdnode_apitransportlocaladdr).

**Fields:**
- `app: axum::Router` - Router served on every connection.

**Methods:**

- `fn new(app: Router) -> Self` - Wraps `app` so every connection's addresses reach its handlers.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ConnAddrsMakeService`
- **Service**
  - `fn poll_ready(self: & mut Self, _cx: & mut Context) -> Poll<Result<(), <Self as >::Error>>`
  - `fn call(self: & mut Self, stream: IncomingStream<TcpListener>) -> <Self as >::Future`



## bookclerkd::node_api::transport::ConnAddrsService

*Struct*

Per-connection service built by [`ConnAddrsMakeService`](transport.md#bookclerkdnode_apitransportconnaddrsmakeservice).

**Fields:**
- `app: axum::Router` - Router serving the request.
- `peer: std::net::SocketAddr` - Remote peer of the connection.
- `local: Option<std::net::SocketAddr>` - Local address of the connection, when the socket reported one.

**Trait Implementations:**

- **Service**
  - `fn poll_ready(self: & mut Self, cx: & mut Context) -> Poll<Result<(), <Self as >::Error>>`
  - `fn call(self: & mut Self, request: Request) -> <Self as >::Future`
- **Clone**
  - `fn clone(self: &Self) -> ConnAddrsService`



## bookclerkd::node_api::transport::ConnInfo

*Struct*

What is known about one connection.

**Fields:**
- `peer: Option<std::net::SocketAddr>` - Remote peer, when the server recorded it.
- `local: Option<std::net::SocketAddr>` - Local address of the connection, when the server recorded it.
- `listener: ListenerKind` - Listener that accepted the connection.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ConnInfo) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ConnInfo`



## bookclerkd::node_api::transport::ListenerKind

*Enum*

Which listener accepted the connection. The built-in TLS listener
([`super::tls`](tls.md)) attaches its kind as a request extension; a request
without one came through a plain `daemon.listen` listener.

**Variants:**
- `Plain` - The plain-HTTP `daemon.listen` listener.
- `AcmeTls` - The built-in rustls listener serving an ACME certificate.
- `SelfSignedTls` - The built-in rustls listener serving the pinned self-signed

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ListenerKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ListenerKind) -> bool`



## bookclerkd::node_api::transport::LocalAddr

*Struct*

The local address of the connection a request arrived on, attached as a
request extension by [`ConnAddrsMakeService`](transport.md#bookclerkdnode_apitransportconnaddrsmakeservice).

This is the accepted socket's own address, not the listener's bind
address: on a wildcard bind (`0.0.0.0` / `[::]`) a same-machine client
still shows a loopback local address here.

**Tuple Struct**: `(std::net::SocketAddr)`

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LocalAddr`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &LocalAddr) -> bool`



## bookclerkd::node_api::transport::RefusalReason

*Enum*

Why a connection was refused.

**Variants:**
- `PlainHttp` - Plain HTTP from a peer that is not loopback and not a trusted proxy.
- `UnknownPeer` - The server did not record the peer or local address.
- `UntrustedForwarding` - Forwarding headers from a loopback peer that is not a trusted proxy.
- `ProxyProtoMissing` - A trusted proxy sent no `X-Forwarded-Proto`.
- `ProxyProtoAmbiguous` - A trusted proxy sent more than one forwarded protocol, or the
- `ProxyProtoNotHttps` - A trusted proxy forwarded plain HTTP.
- `PublicOriginNotHttps` - Proxy mode needs an https `integrations.public_origin`.
- `ListenerModeMismatch` - A TLS listener accepted the connection under a different mode.
- `Misconfigured` - `[daemon.node_api]` is incomplete for its mode.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Stable code for the response body and logs.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RefusalReason`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RefusalReason) -> bool`



## bookclerkd::node_api::transport::TransportConfig

*Struct*

Settings [`classify_transport`](transport.md#bookclerkdnode_apitransportclassify_transport) reads.

**Generic Parameters:**
- 'a

**Fields:**
- `node_api: &'a bookclerk_config::NodeApiConfig` - `[daemon.node_api]`.
- `trusted_proxies: &'a [String]` - `daemon.trusted_proxies`.
- `public_origin: Option<&'a str>` - `integrations.public_origin`.

**Methods:**

- `fn from_config(cfg: &'a bookclerk_config::Config) -> Self` - The policy inputs of a loaded [`bookclerk_config::Config`](../../bookclerk_config/settings.md#bookclerk_configsettingsconfig).

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> TransportConfig<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::node_api::transport::TransportDecision

*Enum*

Outcome of [`classify_transport`](transport.md#bookclerkdnode_apitransportclassify_transport).

**Variants:**
- `Allowed(AllowedMode)` - Serve the request.
- `Refused(RefusalReason)` - Answer 403 `transport_insecure`.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> TransportDecision`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &TransportDecision) -> bool`



## bookclerkd::node_api::transport::classify_transport

*Function*

Classifies one node-API connection. Pure: no I/O, no clock.

```rust
fn classify_transport(conn: &ConnInfo, headers: &axum::http::HeaderMap, cfg: &TransportConfig) -> TransportDecision
```



## bookclerkd::node_api::transport::refused_response

*Function*

The 403 answer for a refused connection.

```rust
fn refused_response(reason: RefusalReason) -> axum::response::Response
```



