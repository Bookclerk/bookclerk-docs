**bookclerk_workerd > config**

# Module: config

## Contents

**Structs**

- [`BindingSpec`](#bindingspec) - One workerd binding the wrapper or author isolate may receive.
- [`GeneratedConfig`](#generatedconfig) - Materialize bridge assets + config under `root`, return config path and socket addr hint.

**Enums**

- [`BindingTarget`](#bindingtarget) - Realization of a [`BindingSpec`](config.md#bookclerk_workerdconfigbindingspec).
- [`EntrypointSource`](#entrypointsource) - How the plugin worker entrypoint is produced.
- [`ListenSpec`](#listenspec) - How the bridge HTTP socket is exposed to `bookclerk-workerd`.

**Functions**

- [`adapter_binding_plan`](#adapter_binding_plan) - Bindings the generated adapter isolate receives (not the author isolate).
- [`adapter_entrypoint_bindings`](#adapter_entrypoint_bindings) - `PLUGIN_<ENTRYPOINT>` service bindings the adapter isolate receives for
- [`egress_domains_for`](#egress_domains_for) - Egress allowlist: plugin domains, plus Pyodide CDN hosts when Python + Outbound.
- [`entrypoint_binding`](#entrypoint_binding) - Adapter binding name and author-exported class name for one named
- [`generated_backend_proxy_plan`](#generated_backend_proxy_plan) - Entrypoint + bindings for a generated native-behind-workerd proxy.
- [`host_state_dir`](#host_state_dir) - Host-chosen session directory from [`WORKERD_STATE_DIR_ENV`](config.md#bookclerk_workerdconfigworkerd_state_dir_env).
- [`manifest_describe_json`](#manifest_describe_json) - Manifest-derived `PluginDescribe` bound on the adapter as `PLUGIN_DESCRIBE`.
- [`materialize`](#materialize) - Materialize bridge assets + Cap'n Proto config into a writable state dir.
- [`materialize_native_backend`](#materialize_native_backend) - Materialize a generated control-plane adapter isolate for a native guest
- [`plugin_global_outbound`](#plugin_global_outbound) - Plugin isolate `globalOutbound`: Deny → blocked; Outbound → egress proxy.
- [`validate_host_state_dir`](#validate_host_state_dir) - Require `dir` to be an existing owner-only directory under `$TMPDIR`.
- [`workerd_state_dir`](#workerd_state_dir) - Where bridge assets + Cap'n Proto config are written.

**Constants**

- [`SDK_JS_MODULE_NAMES`](#sdk_js_module_names) - Package import names authors use for the workerd `BookclerkEntrypoint` SDK.
- [`SDK_PY_DB_VALUE_MODULE`](#sdk_py_db_value_module) - Python module path for typed SQL value / execute codec helpers.
- [`SDK_PY_GUEST_SQL_MODULE`](#sdk_py_guest_sql_module) - Python module path for guest-side SQL statement classification helpers.
- [`SDK_PY_INIT_MODULE`](#sdk_py_init_module) - Python module path that initializes the sparse workerd guest SDK.
- [`SDK_PY_PRODUCT_ABI_MODULE`](#sdk_py_product_abi_module) - Python module path for the generated product ABI constants.
- [`SDK_PY_WORKERD_MODULE`](#sdk_py_workerd_module) - Python package path for `from bookclerk_plugin_sdk.workerd import …`.
- [`WORKERD_SERVE_EXPERIMENTAL`](#workerd_serve_experimental) - `workerd serve` flag that unlocks `$experimental` compatibility flags.
- [`WORKERD_STATE_DIR_ENV`](#workerd_state_dir_env) - Env var the host sets to the per-session workerd state directory.

---

## bookclerk_workerd::config::BindingSpec

*Struct*

One workerd binding the wrapper or author isolate may receive.

**Fields:**
- `name: String` - Binding name (`HTTP`, `STORAGE`, `GRANTED`, …).
- `target: BindingTarget` - Where the binding is realized.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BindingSpec`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_workerd::config::BindingTarget

*Enum*

Realization of a [`BindingSpec`](config.md#bookclerk_workerdconfigbindingspec).

**Variants:**
- `IsolateService{ service: String }` - Another named workerd service (isolate RPC).
- `ExternalFetch{ address: String }` - External `fetch()` / raw streaming HTTP (Unix socket on POSIX).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> BindingTarget`



## bookclerk_workerd::config::EntrypointSource

*Enum*

How the plugin worker entrypoint is produced.

The hybrid executor lives above this crate (`bookclerk-plugin-host`).
`bookclerk-workerd` only materializes these seams. Do not add
`PluginExecutor` here — a later host executor consumes [`EntrypointSource`](config.md#bookclerk_workerdconfigentrypointsource)
+ [`BindingSpec`](config.md#bookclerk_workerdconfigbindingspec).

**Variants:**
- `AuthorModules{ modules_dir: String, main_module: String, entrypoint: Option<String> }` - Author modules from the install `modules/` directory.
- `GeneratedBackendProxy{ module_source: String, entrypoint: String }` - Generated first-party backend-proxy worker (native jail / later container).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EntrypointSource`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_workerd::config::GeneratedConfig

*Struct*

Materialize bridge assets + config under `root`, return config path and socket addr hint.

**Fields:**
- `config_path: std::path::PathBuf` - Path to the generated workerd config file.
- `listen: ListenSpec` - Listen address for the bridge socket.
- `state_dir: std::path::PathBuf` - Writable state directory for the isolate (outside the install root).
- `import_path: std::path::PathBuf` - Pass to `workerd serve -I` so Cap'n Proto `/modules/…` embeds resolve



## bookclerk_workerd::config::ListenSpec

*Enum*

How the bridge HTTP socket is exposed to `bookclerk-workerd`.

**Variants:**
- `TcpLoopback(u16)` - `127.0.0.1:port` — workerd binds itself (unconfined smoke / Windows).
- `InheritedTcp{ port: u16 }` - Parent already bound `127.0.0.1:0`; workerd inherits via `--socket-fd`.

**Methods:**

- `fn workerd_socket_line(self: &Self) -> String` - Cap'n Proto `sockets` entry for the bridge RPC listener.
- `fn client_base_url(self: &Self) -> String` - HTTP origin the workerd isolate uses to call back into Bookclerk.
- `fn port(self: &Self) -> u16` - TCP port the bridge socket binds.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> ListenSpec`



## bookclerk_workerd::config::SDK_JS_MODULE_NAMES

*Constant*: `&[&str]`

Package import names authors use for the workerd `BookclerkEntrypoint` SDK.



## bookclerk_workerd::config::SDK_PY_DB_VALUE_MODULE

*Constant*: `&str`

Python module path for typed SQL value / execute codec helpers.



## bookclerk_workerd::config::SDK_PY_GUEST_SQL_MODULE

*Constant*: `&str`

Python module path for guest-side SQL statement classification helpers.



## bookclerk_workerd::config::SDK_PY_INIT_MODULE

*Constant*: `&str`

Python module path that initializes the sparse workerd guest SDK.



## bookclerk_workerd::config::SDK_PY_PRODUCT_ABI_MODULE

*Constant*: `&str`

Python module path for the generated product ABI constants.



## bookclerk_workerd::config::SDK_PY_WORKERD_MODULE

*Constant*: `&str`

Python package path for `from bookclerk_plugin_sdk.workerd import …`.



## bookclerk_workerd::config::WORKERD_SERVE_EXPERIMENTAL

*Constant*: `&str`

`workerd serve` flag that unlocks `$experimental` compatibility flags.

Required so the egress worker can export a `connect()` handler. Author
isolates do not receive this compat flag unless their own manifest lists it.



## bookclerk_workerd::config::WORKERD_STATE_DIR_ENV

*Constant*: `&str`

Env var the host sets to the per-session workerd state directory.



## bookclerk_workerd::config::adapter_binding_plan

*Function*

Bindings the generated adapter isolate receives (not the author isolate).

# Arguments

* `granted` - When true, include the host reverse-channel `GRANTED` binding.

```rust
fn adapter_binding_plan(granted: bool) -> Vec<BindingSpec>
```



## bookclerk_workerd::config::adapter_entrypoint_bindings

*Function*

`PLUGIN_<ENTRYPOINT>` service bindings the adapter isolate receives for
every named entrypoint in `manifest.entrypoints`, in manifest order.

# Arguments

* `manifest` - Parsed plugin manifest.

# Returns

`(binding, class)` pairs; empty when the plugin exports no named entrypoint.

```rust
fn adapter_entrypoint_bindings(manifest: &bookclerk_plugin_manifest::PluginManifest) -> Vec<(String, String)>
```



## bookclerk_workerd::config::egress_domains_for

*Function*

Egress allowlist: plugin domains, plus Pyodide CDN hosts when Python + Outbound.

Uses the same host set as [`bookclerk_plugin_manifest::consent_domains_for`](../bookclerk_plugin_manifest/egress.md#bookclerk_plugin_manifestegressconsent_domains_for) so
materialize cannot silently widen beyond what `consent_request` / grants cover.

# Arguments

* `needs_python` - Boolean flag `needs_python`.
* `mode` - Network mode from the plugin manifest.
* `base` - String `base` for this call.

# Returns

Collected results (may be empty).

```rust
fn egress_domains_for(needs_python: bool, mode: bookclerk_plugin_manifest::NetworkMode, base: &[String]) -> Vec<String>
```



## bookclerk_workerd::config::entrypoint_binding

*Function*

Adapter binding name and author-exported class name for one named
entrypoint (`storefront` → `PLUGIN_STOREFRONT` / `Storefront`).

# Arguments

* `entrypoint` - Named entrypoint from the manifest.

# Returns

`(binding, class)` pair the generated config binds on the adapter isolate.

```rust
fn entrypoint_binding(entrypoint: bookclerk_plugin_abi::Entrypoint) -> (String, String)
```



## bookclerk_workerd::config::generated_backend_proxy_plan

*Function*

Entrypoint + bindings for a generated native-behind-workerd proxy.

Does not require an author `[workerd]` module tree and binds no data-plane
service: the adapter isolate only owns the control plane (`GRANTED`,
`BRIDGE_TOKEN`, `PLUGIN_DESCRIBE`).

```rust
fn generated_backend_proxy_plan() -> (EntrypointSource, Vec<BindingSpec>)
```



## bookclerk_workerd::config::host_state_dir

*Function*

Host-chosen session directory from [`WORKERD_STATE_DIR_ENV`](config.md#bookclerk_workerdconfigworkerd_state_dir_env).

# Errors

Returns an error when the variable is unset or [`validate_host_state_dir`](config.md#bookclerk_workerdconfigvalidate_host_state_dir)
rejects the path.

```rust
fn host_state_dir() -> anyhow::Result<std::path::PathBuf>
```



## bookclerk_workerd::config::manifest_describe_json

*Function*

Manifest-derived `PluginDescribe` bound on the adapter as `PLUGIN_DESCRIBE`.

The adapter answers `describe` from this projection and lets the author's
optional `describe()` refine presentation fields only.

# Arguments

* `manifest` - Parsed plugin manifest.

# Errors

Returns an error when the describe struct cannot be serialized as JSON.

```rust
fn manifest_describe_json(manifest: &bookclerk_plugin_manifest::PluginManifest) -> anyhow::Result<String>
```



## bookclerk_workerd::config::materialize

*Function*

Materialize bridge assets + Cap'n Proto config into a writable state dir.

**TMPDIR contents (guest-writable scratch)** — only generated launcher state:
- `.bookclerk/` — first-party bridge / egress / host stub / injected SDK
- `workerd-config.capnp`
- `granted.sock` (created by the launcher, not this function)

**Author `modules/` stay in the read-only install root.** Cap'n Proto paths
that look absolute (`/…`) are resolved via `--import-path` (same rules as
`import "/workerd/workerd.capnp"`), not the filesystem root — so embeds are
`/modules/…` with the plugin install root passed as `-I`.

Today's launcher materializes `EntrypointSource::AuthorModules` from the
install tree **and** a distinct adapter isolate that alone receives
`GRANTED` / `BRIDGE_TOKEN`. `EntrypointSource::GeneratedBackendProxy` and
[`BindingSpec`](config.md#bookclerk_workerdconfigbindingspec) are seams for a later host executor (native jail / container
behind a generated `fetch()` proxy) — that path must not require author
`[workerd]` modules. Direct native Cap'n Proto remains a host-selected
fallback, not plugin-selectable policy bypass.

`granted_addr` is an optional workerd `external` address (`host:port`,
`unix:/path`, `unix:granted.sock`, or Linux `unix-abstract:name`) for the
adapter-private `GRANTED` capability channel.

`bridge_token` is a per-isolate bearer shared by the launcher and the bridge
`/rpc` + `/health` routes (`BRIDGE_TOKEN` binding).

# Arguments

* `root` - Cargo workspace root directory.
* `manifest` - Parsed plugin manifest.
* `egress` - `egress` input for this call.
* `listen` - Daemon listen address (`host:port` or URL).
* `granted_addr` - Optional `GRANTED` external service address.
* `bridge_token` - Per-isolate bridge bearer token.
* `state_dir` - Existing session directory from [`workerd_state_dir`](config.md#bookclerk_workerdconfigworkerd_state_dir), or
  `None` to allocate a unique directory.

# Returns

On success, the inner `GeneratedConfig` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn materialize(root: &std::path::Path, manifest: &bookclerk_plugin_manifest::PluginManifest, egress: &crate::egress::EgressProxy, limits: bookclerk_plugin_manifest::EffectiveWorkerdLimits, listen: ListenSpec, granted_addr: Option<&str>, bridge_token: &str, state_dir: Option<&std::path::Path>) -> anyhow::Result<GeneratedConfig>
```



## bookclerk_workerd::config::materialize_native_backend

*Function*

Materialize a generated control-plane adapter isolate for a native guest
(no author `[workerd]` modules).

The host executor owns the process tree: it launches workerd and the
verified native guest, and forwards every entrypoint call to the guest as
typed Cap'n Proto. The adapter isolate decides `describe` / `open` policy
from the manifest projection bound as `PLUGIN_DESCRIBE`. Plugin input
cannot choose the executable or weaken the sandbox.

`state_dir` is an existing session directory from [`workerd_state_dir`](config.md#bookclerk_workerdconfigworkerd_state_dir), or
`None` to allocate a unique directory.

# Errors

Returns an error when state-dir I/O, describe serialization, or the config
write fails.

```rust
fn materialize_native_backend(root: &std::path::Path, manifest: &bookclerk_plugin_manifest::PluginManifest, egress: &crate::egress::EgressProxy, limits: bookclerk_plugin_manifest::EffectiveWorkerdLimits, listen: ListenSpec, granted_addr: Option<&str>, bridge_token: &str, state_dir: Option<&std::path::Path>) -> anyhow::Result<GeneratedConfig>
```



## bookclerk_workerd::config::plugin_global_outbound

*Function*

Plugin isolate `globalOutbound`: Deny → blocked; Outbound → egress proxy.

# Arguments

* `mode` - Network mode from the plugin manifest.

# Returns

`&'static str` result.

```rust
fn plugin_global_outbound(mode: bookclerk_plugin_manifest::NetworkMode) -> &'static str
```



## bookclerk_workerd::config::validate_host_state_dir

*Function*

Require `dir` to be an existing owner-only directory under `$TMPDIR`.

# Errors

Returns an error when the path is relative, contains `..`, is not a
directory, escapes the process temp root, or cannot be chmodded owner-only.

```rust
fn validate_host_state_dir(dir: &std::path::Path) -> anyhow::Result<std::path::PathBuf>
```



## bookclerk_workerd::config::workerd_state_dir

*Function*

Where bridge assets + Cap'n Proto config are written.

Prefer `$TMPDIR` (the guest scratch dir inside a jail). The plugin install
root is read-only under Landlock, so materializing `.bookclerk/` there fails.
Each call allocates a unique leaf (`w` + root prefix + nonce) with exclusive
`create_dir`, so concurrent sessions of the same plugin cannot clobber
`workerd-config.capnp`. GRANTED / native-proxy sockets use Linux abstract
names (or a relative `unix:granted.sock`) so `$TMPDIR` may exceed
`sockaddr_un`.

Callers that bind sockets first (the launcher) must pass the returned path
into [`materialize`](config.md#bookclerk_workerdconfigmaterialize) / [`materialize_native_backend`](config.md#bookclerk_workerdconfigmaterialize_native_backend) so config and sockets
share one directory.

# Arguments

* `plugin_root` - Filesystem path (`plugin_root`).

# Returns

On success, the inner `PathBuf` value.

# Errors

Returns an error when the directory cannot be created exclusively.

```rust
fn workerd_state_dir(plugin_root: &std::path::Path) -> anyhow::Result<std::path::PathBuf>
```



