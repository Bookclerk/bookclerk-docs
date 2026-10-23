**bookclerkd > node_api > tls**

# Module: node_api::tls

## Contents

**Structs**

- [`NodeApiGate`](#nodeapigate) - Request extension every listener carries: whether the node API is
- [`NodeApiTls`](#nodeapitls) - The started (or refused) TLS listener.

**Enums**

- [`NodeApiStatus`](#nodeapistatus) - Whether the node API is being served.

**Functions**

- [`node_api_mounted`](#node_api_mounted) - True when the node API may serve a request in `mode` (the live config).
- [`start_with`](#start_with) - Starts the TLS listener the configured mode needs, serving `web` (the

---

## bookclerkd::node_api::tls::NodeApiGate

*Struct*

Request extension every listener carries: whether the node API is
mounted, and the `[daemon.node_api]` mode it was started for. Modes
without a TLS listener are always mounted.

**Fields:**
- `status: watch::Receiver<NodeApiStatus>` - Status published by the TLS listener (or fixed at start).
- `mode: bookclerk_config::NodeApiMode` - The mode [`start_with`](tls.md#bookclerkdnode_apitlsstart_with) ran for; a reloaded config naming another mode
- `origin: Option<String>` - Canonical owner origin the TLS listener was started for (`acme` and

**Methods:**

- `fn origin_matches(self: &Self, live: Option<&str>) -> bool` - True when `live` (the canonical owner origin in the current config)
- `fn status(self: &Self) -> NodeApiStatus` - Current status.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> NodeApiGate`



## bookclerkd::node_api::tls::NodeApiStatus

*Enum*

Whether the node API is being served.

**Variants:**
- `Starting` - The TLS listener has no certificate yet.
- `Ready` - Serving.
- `Failed` - The TLS listener could not start or obtain a certificate; the node

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &NodeApiStatus) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NodeApiStatus`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::node_api::tls::NodeApiTls

*Struct*

The started (or refused) TLS listener.

**Fields:**
- `gate: NodeApiGate` - Mount state for every listener.
- `local: Option<std::net::SocketAddr>` - Bound address, when the listener is up.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::node_api::tls::node_api_mounted

*Function*

True when the node API may serve a request in `mode` (the live config).
Modes with a TLS listener need a gate reporting
`NodeApiStatus::Ready`; a request without a gate is refused in those
modes (fail closed). A gate started for another mode means
`[daemon.node_api]` changed on reload: the listeners still match the old
mode, so the node API stays unmounted until a restart.

```rust
fn node_api_mounted(mode: bookclerk_config::NodeApiMode, gate: Option<&NodeApiGate>) -> bool
```



## bookclerkd::node_api::tls::start_with

*Function*

Starts the TLS listener the configured mode needs, serving `web` (the
daemon's whole router, which mounts the node API too) or, given none,
only the node API routes. Never fails: a mode that cannot start reports
`NodeApiStatus::Failed` and the node API stays unmounted.

```rust
fn start_with(state: &std::sync::Arc<crate::api::AppState>, cfg: &bookclerk_config::Config, files_dir: &std::path::Path, web: Option<axum::Router>) -> NodeApiTls
```



