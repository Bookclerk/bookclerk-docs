**bookclerkd > api**

# Module: api

## Contents

**Structs**

- [`AcquireRequestBody`](#acquirerequestbody) - Optional body for `POST /api/library/acquire`.
- [`AppState`](#appstate) - Shared daemon state.
- [`IntegrationScanRequest`](#integrationscanrequest) - Optional body for `POST /integrations/{id}/scan`.
- [`JobInfo`](#jobinfo) - Snapshot of one queued or running control-plane job.
- [`ScanRequest`](#scanrequest) - Optional body for `POST /api/library/scan`.
- [`TrayHandoffTicket`](#trayhandoffticket) - In-process tray Open Bookclerk ticket: hash of a one-time code plus expiry.

**Functions**

- [`bind_listen_addrs`](#bind_listen_addrs) - Bind every configured listen address; skip failures unless none succeed.
- [`build_operator_auth`](#build_operator_auth) - Build operator auth runtime from config + library DB (env / encrypted_secrets).
- [`preflight_listen_bind`](#preflight_listen_bind) - Preflight-bind addresses in `new` that are not already held by `old`.
- [`reload_daemon_config`](#reload_daemon_config) - Reload `config.toml` from disk and publish a complete candidate runtime.
- [`resolve_ui_dist`](#resolve_ui_dist) - Resolve the Vite build output directory for the GUI.
- [`revert_listen_after_bind_failure`](#revert_listen_after_bind_failure) - After a failed rebind, restore the last successfully bound listen set in live config.
- [`router`](#router) - Build the HTTP router (API + optional static UI + SPA portal APIs).
- [`start_integration_watchers`](#start_integration_watchers) - Start integration watchers with claim-ticket minting (shared by startup + reload).
- [`validate_daemon_listen`](#validate_daemon_listen) - Parse `daemon.listen` and reject unsafe auth/listen combinations.
- [`validate_daemon_listen_against_auth`](#validate_daemon_listen_against_auth) - Validate listen, and the node API's TLS listener (which serves the web

---

## bookclerkd::api::AcquireRequestBody

*Struct*

Optional body for `POST /api/library/acquire`.

**Fields:**
- `asin: Option<String>` - Audible ASIN filter; first of uuid/asin/isbn/product_id wins.
- `uuid: Option<String>` - Library title UUID filter.
- `isbn: Option<String>` - ISBN filter for non-Audible stores.
- `product_id: Option<String>` - Store product id filter.
- `account: Option<String>` - Restrict acquire to this store account.
- `plugin_instance_id: Option<String>` - When set, fetch through this deployed plugin instance.
- `titles: Vec<String>` - More title ids; each (with the single filter above) becomes its own
- `force: bool` - Re-download titles already acquired.
- `pdf_only: bool` - Fetch only each title's companion PDF.

**Trait Implementations:**

- **Default**
  - `fn default() -> AcquireRequestBody`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::api::AppState

*Struct*

Shared daemon state.

**Fields:**
- `config: std::sync::Arc<tokio::sync::RwLock<bookclerk_config::Config>>` - Live daemon config swapped under `reload_lock`; handlers clone then drop the read lock.
- `library: std::sync::Arc<tokio::sync::RwLock<bookclerk_library::LibraryStore>>` - Shared library store; clone the handle and drop the lock before awaiting DB I/O.
- `database_registry: std::sync::Arc<tokio::sync::RwLock<bookclerk_plugin_host::DatabaseRegistry>>` - Loaded database plugin guests; replaced when `[database].plugin` reloads.
- `job_notify: std::sync::Arc<tokio::sync::Notify>` - Wakes the durable job worker when a new row is admitted.
- `job_runtime: std::sync::Arc<tokio::sync::RwLock<()>>` - Shared barrier covering admission, claim, and handler execution.
- `work_lock: tokio::sync::Mutex<()>` - Serialize scan/acquire work so jobs do not thrash the same accounts.
- `discover_gate: std::sync::Arc<tokio::sync::Semaphore>` - Cap concurrent discover/embed work so ONNX load + inference cannot saturate
- `integrations: std::sync::Arc<tokio::sync::RwLock<bookclerk_integrations::IntegrationRegistry>>` - Loaded integration guests used by watchers, the portal, and listening sync.
- `sources: std::sync::Arc<tokio::sync::RwLock<bookclerk_source::SourceRegistry>>` - Loaded storefront source guests used by scan, acquire, and discover.
- `destinations: std::sync::Arc<tokio::sync::RwLock<bookclerk_plugin_host::DestinationRegistry>>` - Loaded output destinations that receive acquire artifacts.
- `auth: std::sync::Arc<tokio::sync::RwLock<std::sync::Arc<crate::auth::OperatorAuthState>>>` - Operator auth runtime (enabled flag + token + sessions). Replaced on reload.
- `reload_lock: tokio::sync::Mutex<()>` - Serializes config reload so candidate runtime swaps are not interleaved.
- `listen_reload: std::sync::Arc<tokio::sync::Notify>` - Wakes the HTTP server to rebind when `daemon.listen` changes on config reload.
- `last_bound_listen: tokio::sync::RwLock<Option<bookclerk_config::ListenAddrs>>` - Last successfully bound listen set; used to roll back a failed rebind.
- `tray: tokio::sync::RwLock<Option<bookclerk_tray::SharedTrayConfig>>` - Optional tray handle so reload can refresh token / auth_enabled / listen.
- `tray_handoff: tokio::sync::Mutex<Option<TrayHandoffTicket>>` - Single-use tray Open Bookclerk ticket (hash + deadline). Replaced on prepare; restart clears it.
- `deployment_runtime: std::sync::OnceLock<std::sync::Arc<bookclerk_plugin_host::LiveDeploymentRuntime>>` - Guest sessions for deployed instances. One map for the process lifetime.
- `node_api_tls_bound: std::sync::OnceLock<std::net::SocketAddr>` - Where the node API's TLS listener is bound, once it is. That listener
- `library_swaps: tokio::sync::watch::Sender<u64>` - Counts database swaps: bumped after `Self::library` is replaced so

**Methods:**

- `fn library_snapshot(self: &Self) -> LibraryStore` - Cheap clone of the live library handle; drop the read lock before awaiting DB I/O.
- `fn auth_snapshot(self: &Self) -> Arc<OperatorAuthState>` - Clone the live operator auth `Arc` and drop the lock before further `.await`s.
- `fn deployment_runtime(self: &Self) -> Arc<bookclerk_plugin_host::LiveDeploymentRuntime>` - Reconciler session map for this process. The first call creates it.



## bookclerkd::api::IntegrationScanRequest

*Struct*

Optional body for `POST /integrations/{id}/scan`.

**Fields:**
- `force: Option<bool>` - When true, the integration rescans even if it considers itself current.

**Trait Implementations:**

- **Default**
  - `fn default() -> IntegrationScanRequest`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::api::JobInfo

*Struct*

Snapshot of one queued or running control-plane job.

**Fields:**
- `id: String` - Opaque job id returned to the client when the job is accepted.
- `kind: String` - Job type (`scan`, `acquire`, `listen_sync`).
- `status: String` - Lifecycle string (`pending`, `running`, `succeeded`, `failed`, `cancelled`).
- `detail: Option<String>` - Optional operator-facing progress or error text.
- `progress: Option<String>` - Human-readable progress string while the job is running.
- `attempt_count: i64` - Number of times a worker has claimed this row.
- `max_attempts: i64` - Maximum claims before a failure is terminal.
- `error_kind: Option<String>` - Structured error kind when failed or cancelled.
- `created_at: Option<String>` - RFC 3339 timestamp when the row was inserted.
- `finished_at: Option<String>` - RFC 3339 timestamp when the row reached a terminal state.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobInfo`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::api::ScanRequest

*Struct*

Optional body for `POST /api/library/scan`.

**Fields:**
- `account: Option<String>` - When set, scan this account even if `scan_enabled` is false.
- `plugin_instance_id: Option<String>` - When set, scan only this deployed plugin instance.

**Trait Implementations:**

- **Default**
  - `fn default() -> ScanRequest`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::api::TrayHandoffTicket

*Struct*

In-process tray Open Bookclerk ticket: hash of a one-time code plus expiry.

**Fields:**
- `code_hash: String` - SHA-256 hex of the plaintext code returned to the tray (never the code itself).
- `deadline: std::time::Instant` - Instant after which GET must fail closed.



## bookclerkd::api::bind_listen_addrs

*Function*

Bind every configured listen address; skip failures unless none succeed.

```rust
fn bind_listen_addrs(listen: &bookclerk_config::ListenAddrs) -> anyhow::Result<Vec<tokio::net::TcpListener>>
```



## bookclerkd::api::build_operator_auth

*Function*

Build operator auth runtime from config + library DB (env / encrypted_secrets).

```rust
fn build_operator_auth(config: &bookclerk_config::Config, library: &bookclerk_library::LibraryStore) -> anyhow::Result<crate::auth::OperatorAuthState>
```



## bookclerkd::api::preflight_listen_bind

*Function*

Preflight-bind addresses in `new` that are not already held by `old`.

Addresses that share a port with a current listener cannot be probed while
the old sockets are live; those transitions rely on post-rebind rollback via
`AppState::last_bound_listen`.

```rust
fn preflight_listen_bind(old: &bookclerk_config::ListenAddrs, new: &bookclerk_config::ListenAddrs) -> anyhow::Result<()>
```



## bookclerkd::api::reload_daemon_config

*Function*

Reload `config.toml` from disk and publish a complete candidate runtime.

Auth is always rebuilt and swapped **before** listen rebind notification so a
public listener never outruns the middleware that enforces authentication.

```rust
fn reload_daemon_config(state: &AppState) -> anyhow::Result<String>
```



## bookclerkd::api::resolve_ui_dist

*Function*

Resolve the Vite build output directory for the GUI.

Prefer `BOOKCLERK_UI_DIST`, then paths beside the running binary (desktop
launches often have cwd=`$HOME`), then cwd-relative / compile-time paths.

```rust
fn resolve_ui_dist() -> Option<std::path::PathBuf>
```



## bookclerkd::api::revert_listen_after_bind_failure

*Function*

After a failed rebind, restore the last successfully bound listen set in live config.

```rust
fn revert_listen_after_bind_failure(state: &AppState) -> bool
```



## bookclerkd::api::router

*Function*

Build the HTTP router (API + optional static UI + SPA portal APIs).

```rust
fn router(state: std::sync::Arc<AppState>, ui_dist: Option<std::path::PathBuf>) -> axum::Router
```



## bookclerkd::api::start_integration_watchers

*Function*

Start integration watchers with claim-ticket minting (shared by startup + reload).

Awaits `start_all` on a registry snapshot so a delayed task cannot race a later
reload and double-start the live integrations.

```rust
fn start_integration_watchers(state: &AppState)
```



## bookclerkd::api::validate_daemon_listen

*Function*

Parse `daemon.listen` and reject unsafe auth/listen combinations.

```rust
fn validate_daemon_listen(config: &bookclerk_config::Config) -> anyhow::Result<()>
```



## bookclerkd::api::validate_daemon_listen_against_auth

*Function*

Validate listen, and the node API's TLS listener (which serves the web
app too), against the effective runtime auth flag (not only TOML).

# Errors

Returns an error when `daemon.listen` does not parse or is empty, or when
auth is off and either listener is reachable beyond loopback.

```rust
fn validate_daemon_listen_against_auth(config: &bookclerk_config::Config, auth_enabled: bool) -> anyhow::Result<()>
```



