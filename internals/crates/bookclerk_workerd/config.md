# bookclerk_workerd::config

Generate a real workerd Cap'n Proto config for one plugin isolate.

| Item | Kind | Summary |
| --- | --- | --- |
| [`BindingSpec`](#bindingspec) | struct | One workerd binding the wrapper or author isolate may receive. |
| [`GeneratedConfig`](#generatedconfig) | struct | Materialize bridge assets + config under `root`, return config path and socket addr hint. |
| [`ListenSpec`](#listenspec) | enum | How the bridge HTTP socket is exposed to `bookclerk-workerd`. |
| [`EntrypointSource`](#entrypointsource) | enum | How the plugin worker entrypoint is produced. |
| [`BindingTarget`](#bindingtarget) | enum | Realization of a `BindingSpec`. |
| [`workerd_state_dir`](#workerd_state_dir) | function | Where bridge assets + Cap'n Proto config are written. |
| [`host_state_dir`](#host_state_dir) | function | Host-chosen session directory from `WORKERD_STATE_DIR_ENV`. |
| [`validate_host_state_dir`](#validate_host_state_dir) | function | Require `dir` to be an existing owner-only directory under `$TMPDIR`. |
| [`adapter_binding_plan`](#adapter_binding_plan) | function | Bindings the generated adapter isolate receives (not the author isolate). |
| [`entrypoint_binding`](#entrypoint_binding) | function | Adapter binding name and author-exported class name for one named entrypoint (`storefront` → `PLUGIN_STOREFRONT` / `Storefront`). |
| [`adapter_entrypoint_bindings`](#adapter_entrypoint_bindings) | function | `PLUGIN_<ENTRYPOINT>` service bindings the adapter isolate receives for every named entrypoint in `manifest.entrypoints`, in manifest order. |
| [`manifest_describe_json`](#manifest_describe_json) | function | Manifest-derived `PluginDescribe` bound on the adapter as `PLUGIN_DESCRIBE`. |
| [`generated_backend_proxy_plan`](#generated_backend_proxy_plan) | function | Entrypoint + bindings for a generated native-behind-workerd proxy. |
| [`materialize_native_backend`](#materialize_native_backend) | function | Materialize a generated control-plane adapter isolate for a native guest (no author `[workerd]` modules). |
| [`materialize`](#materialize) | function | Materialize bridge assets + Cap'n Proto config into a writable state dir. |
| [`plugin_global_outbound`](#plugin_global_outbound) | function | Plugin isolate `globalOutbound`: Deny → blocked; Outbound → egress proxy. |
| [`egress_domains_for`](#egress_domains_for) | function | Egress allowlist: plugin domains, plus Pyodide CDN hosts when Python + Outbound. |
| [`WORKERD_SERVE_EXPERIMENTAL`](#workerd_serve_experimental) | constant | `workerd serve` flag that unlocks `$experimental` compatibility flags. |
| [`SDK_JS_MODULE_NAMES`](#sdk_js_module_names) | constant | Package import names authors use for the workerd `BookclerkEntrypoint` SDK. |
| [`SDK_PY_WORKERD_MODULE`](#sdk_py_workerd_module) | constant | Python package path for `from bookclerk_plugin_sdk.workerd import …`. |
| [`SDK_PY_DB_VALUE_MODULE`](#sdk_py_db_value_module) | constant | Python module path for typed SQL value / execute codec helpers. |
| [`SDK_PY_PRODUCT_ABI_MODULE`](#sdk_py_product_abi_module) | constant | Python module path for the generated product ABI constants. |
| [`SDK_PY_GUEST_SQL_MODULE`](#sdk_py_guest_sql_module) | constant | Python module path for guest-side SQL statement classification helpers. |
| [`SDK_PY_INIT_MODULE`](#sdk_py_init_module) | constant | Python module path that initializes the sparse workerd guest SDK. |
| [`WORKERD_STATE_DIR_ENV`](#workerd_state_dir_env) | constant | Env var the host sets to the per-session workerd state directory. |

## Re-exports

- [`bookclerk_plugin_manifest::PYODIDE_EGRESS_HOSTS`](../bookclerk_plugin_manifest/egress.md#pyodide_egress_hosts)

## Structs

### BindingSpec

```rust
pub struct BindingSpec {
    pub name: String,
    pub target: BindingTarget,
}
```

One workerd binding the wrapper or author isolate may receive.

**Fields**

- `name`: Binding name (`HTTP`, `STORAGE`, `GRANTED`, …).
- `target`: Where the binding is realized.

Implements: `Clone`, `Debug`

### GeneratedConfig

```rust
pub struct GeneratedConfig {
    pub config_path: PathBuf,
    pub listen: ListenSpec,
    pub state_dir: PathBuf,
    pub import_path: PathBuf,
}
```

Materialize bridge assets + config under `root`, return config path and socket addr hint.

**Fields**

- `config_path`: Path to the generated workerd config file.
- `listen`: Listen address for the bridge socket.
- `state_dir`: Writable state directory for the isolate (outside the install root).
- `import_path`: Pass to `workerd serve -I` so Cap'n Proto `/modules/…` embeds resolve
  against the read-only plugin install root.

## Enums

### ListenSpec

```rust
pub enum ListenSpec {
    TcpLoopback(u16),
    InheritedTcp { port: u16 },
}
```

How the bridge HTTP socket is exposed to `bookclerk-workerd`.

**Variants**

- `TcpLoopback`: `127.0.0.1:port` — workerd binds itself (unconfined smoke / Windows).
- `InheritedTcp`: Parent already bound `127.0.0.1:0`; workerd inherits via `--socket-fd`.
  Required under Linux Landlock `OutboundListen` (only `bind(port=0)` is
  allowed — rebinding a concrete ephemeral port is EPERM).
  - `port`: Ephemeral port already bound by the parent (informational for clients).

#### ListenSpec::workerd_socket_line

```rust
pub fn workerd_socket_line(&self) -> String
```

Cap'n Proto `sockets` entry for the bridge RPC listener.

Inherited FDs omit `address` — workerd gets `--socket-fd=rpc=<fd>`.

#### ListenSpec::client_base_url

```rust
pub fn client_base_url(&self) -> String
```

HTTP origin the workerd isolate uses to call back into Bookclerk.

#### ListenSpec::port

```rust
pub fn port(&self) -> u16
```

TCP port the bridge socket binds.

Implements: `Clone`, `Debug`

### EntrypointSource

```rust
pub enum EntrypointSource {
    AuthorModules { modules_dir: String, main_module: String, entrypoint: Option<String> },
    GeneratedBackendProxy { module_source: String, entrypoint: String },
}
```

How the plugin worker entrypoint is produced.

The hybrid executor lives above this crate (`bookclerk-plugin-host`).
`bookclerk-workerd` only materializes these seams. Do not add
`PluginExecutor` here — a later host executor consumes [`EntrypointSource`](#entrypointsource)
+ [`BindingSpec`](#bindingspec).

**Variants**

- `AuthorModules`: Author modules from the install `modules/` directory.
  - `modules_dir`: Directory containing author ES modules (relative to plugin root).
  - `main_module`: Main module filename inside `modules_dir`.
  - `entrypoint`: Optional exported entrypoint class name.
- `GeneratedBackendProxy`: Generated first-party backend-proxy worker (native jail / later container).
  - `module_source`: Module source the launcher writes into the state dir.
  - `entrypoint`: Exported entrypoint class name.

Implements: `Clone`, `Debug`

### BindingTarget

```rust
pub enum BindingTarget {
    IsolateService { service: String },
    ExternalFetch { address: String },
}
```

Realization of a [`BindingSpec`](#bindingspec).

**Variants**

- `IsolateService`: Another named workerd service (isolate RPC).
  - `service`: Service name in the generated config.
- `ExternalFetch`: External `fetch()` / raw streaming HTTP (Unix socket on POSIX).
  - `address`: `unix:/path` or `host:port`.

Implements: `Clone`, `Debug`

## Functions

### workerd_state_dir

```rust
pub fn workerd_state_dir(plugin_root: &Path) -> Result<PathBuf>
```

Where bridge assets + Cap'n Proto config are written.

Prefer `$TMPDIR` (the guest scratch dir inside a jail). The plugin install
root is read-only under Landlock, so materializing `.bookclerk/` there fails.
Each call allocates a unique leaf (`w` + root prefix + nonce) with exclusive
`create_dir`, so concurrent sessions of the same plugin cannot clobber
`workerd-config.capnp`. GRANTED / native-proxy sockets use Linux abstract
names (or a relative `unix:granted.sock`) so `$TMPDIR` may exceed
`sockaddr_un`.

Callers that bind sockets first (the launcher) must pass the returned path
into [`materialize`](#materialize) / [`materialize_native_backend`](#materialize_native_backend) so config and sockets
share one directory.

#### Arguments

* `plugin_root` - Filesystem path (`plugin_root`).

#### Returns

On success, the inner `PathBuf` value.

#### Errors

Returns an error when the directory cannot be created exclusively.

### host_state_dir

```rust
pub fn host_state_dir() -> Result<PathBuf>
```

Host-chosen session directory from [`WORKERD_STATE_DIR_ENV`](#workerd_state_dir_env).

#### Errors

Returns an error when the variable is unset or [`validate_host_state_dir`](#validate_host_state_dir)
rejects the path.

### validate_host_state_dir

```rust
pub fn validate_host_state_dir(dir: &Path) -> Result<PathBuf>
```

Require `dir` to be an existing owner-only directory under `$TMPDIR`.

#### Errors

Returns an error when the path is relative, contains `..`, is not a
directory, escapes the process temp root, or cannot be chmodded owner-only.

### adapter_binding_plan

```rust
pub fn adapter_binding_plan(granted: bool) -> Vec<BindingSpec>
```

Bindings the generated adapter isolate receives (not the author isolate).

#### Arguments

* `granted` - When true, include the host reverse-channel `GRANTED` binding.

### entrypoint_binding

```rust
pub fn entrypoint_binding(entrypoint: Entrypoint) -> (String, String)
```

Adapter binding name and author-exported class name for one named
entrypoint (`storefront` → `PLUGIN_STOREFRONT` / `Storefront`).

#### Arguments

* `entrypoint` - Named entrypoint from the manifest.

#### Returns

`(binding, class)` pair the generated config binds on the adapter isolate.

### adapter_entrypoint_bindings

```rust
pub fn adapter_entrypoint_bindings(manifest: &PluginManifest) -> Vec<(String, String)>
```

`PLUGIN_<ENTRYPOINT>` service bindings the adapter isolate receives for
every named entrypoint in `manifest.entrypoints`, in manifest order.

#### Arguments

* `manifest` - Parsed plugin manifest.

#### Returns

`(binding, class)` pairs; empty when the plugin exports no named entrypoint.

### manifest_describe_json

```rust
pub fn manifest_describe_json(manifest: &PluginManifest) -> Result<String>
```

Manifest-derived `PluginDescribe` bound on the adapter as `PLUGIN_DESCRIBE`.

The adapter answers `describe` from this projection and lets the author's
optional `describe()` refine presentation fields only.

#### Arguments

* `manifest` - Parsed plugin manifest.

#### Errors

Returns an error when the describe struct cannot be serialized as JSON.

### generated_backend_proxy_plan

```rust
pub fn generated_backend_proxy_plan() -> (EntrypointSource, Vec<BindingSpec>)
```

Entrypoint + bindings for a generated native-behind-workerd proxy.

Does not require an author `[workerd]` module tree and binds no data-plane
service: the adapter isolate only owns the control plane (`GRANTED`,
`BRIDGE_TOKEN`, `PLUGIN_DESCRIBE`).

### materialize_native_backend

```rust
pub fn materialize_native_backend(
    root: &Path,
    manifest: &PluginManifest,
    egress: &EgressProxy,
    limits: EffectiveWorkerdLimits,
    listen: ListenSpec,
    granted_addr: Option<&str>,
    bridge_token: &str,
    state_dir: Option<&Path>,
) -> Result<GeneratedConfig>
```

Materialize a generated control-plane adapter isolate for a native guest
(no author `[workerd]` modules).

The host executor owns the process tree: it launches workerd and the
verified native guest, and forwards every entrypoint call to the guest as
typed Cap'n Proto. The adapter isolate decides `describe` / `open` policy
from the manifest projection bound as `PLUGIN_DESCRIBE`. Plugin input
cannot choose the executable or weaken the sandbox.

`state_dir` is an existing session directory from [`workerd_state_dir`](#workerd_state_dir), or
`None` to allocate a unique directory.

#### Errors

Returns an error when state-dir I/O, describe serialization, or the config
write fails.

### materialize

```rust
pub fn materialize(
    root: &Path,
    manifest: &PluginManifest,
    egress: &EgressProxy,
    limits: EffectiveWorkerdLimits,
    listen: ListenSpec,
    granted_addr: Option<&str>,
    bridge_token: &str,
    state_dir: Option<&Path>,
) -> Result<GeneratedConfig>
```

Materialize bridge assets + Cap'n Proto config into a writable state dir.

**TMPDIR contents (guest-writable scratch)** — only generated launcher state:
- `.bookclerk/` — first-party bridge / egress / host stub / injected SDK
- `workerd-config.capnp`
- `granted.sock` (created by the launcher, not this function)

**Author `modules/` stay in the read-only install root.** Cap'n Proto paths
that look absolute (`/…`) are resolved via `--import-path` (same rules as
`import "/workerd/workerd.capnp"`), not the filesystem root — so embeds are
`/modules/…` with the plugin install root passed as `-I`.

Today's launcher materializes [`EntrypointSource::AuthorModules`](#entrypointsource) from the
install tree **and** a distinct adapter isolate that alone receives
`GRANTED` / `BRIDGE_TOKEN`. [`EntrypointSource::GeneratedBackendProxy`](#entrypointsource) and
[`BindingSpec`](#bindingspec) are seams for a later host executor (native jail / container
behind a generated `fetch()` proxy) — that path must not require author
`[workerd]` modules. Direct native Cap'n Proto remains a host-selected
fallback, not plugin-selectable policy bypass.

`granted_addr` is an optional workerd `external` address (`host:port`,
`unix:/path`, `unix:granted.sock`, or Linux `unix-abstract:name`) for the
adapter-private `GRANTED` capability channel.

`bridge_token` is a per-isolate bearer shared by the launcher and the bridge
`/rpc` + `/health` routes (`BRIDGE_TOKEN` binding).

#### Arguments

* `root` - Cargo workspace root directory.
* `manifest` - Parsed plugin manifest.
* `egress` - `egress` input for this call.
* `listen` - Daemon listen address (`host:port` or URL).
* `granted_addr` - Optional `GRANTED` external service address.
* `bridge_token` - Per-isolate bridge bearer token.
* `state_dir` - Existing session directory from [`workerd_state_dir`](#workerd_state_dir), or
  `None` to allocate a unique directory.

#### Returns

On success, the inner `GeneratedConfig` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### plugin_global_outbound

```rust
pub fn plugin_global_outbound(mode: NetworkMode) -> &'static str
```

Plugin isolate `globalOutbound`: Deny → blocked; Outbound → egress proxy.

#### Arguments

* `mode` - Network mode from the plugin manifest.

#### Returns

`&'static str` result.

### egress_domains_for

```rust
pub fn egress_domains_for(
    needs_python: bool,
    mode: NetworkMode,
    base: &[String],
) -> Vec<String>
```

Egress allowlist: plugin domains, plus Pyodide CDN hosts when Python + Outbound.

Uses the same host set as [`bookclerk_plugin_manifest::consent_domains_for`](../bookclerk_plugin_manifest/egress.md#consent_domains_for) so
materialize cannot silently widen beyond what `consent_request` / grants cover.

#### Arguments

* `needs_python` - Boolean flag `needs_python`.
* `mode` - Network mode from the plugin manifest.
* `base` - String `base` for this call.

#### Returns

Collected results (may be empty).

## Constants

### WORKERD_SERVE_EXPERIMENTAL

```rust
pub const WORKERD_SERVE_EXPERIMENTAL: &str = "--experimental";
```

`workerd serve` flag that unlocks `$experimental` compatibility flags.

Required so the egress worker can export a `connect()` handler. Author
isolates do not receive this compat flag unless their own manifest lists it.

### SDK_JS_MODULE_NAMES

```rust
pub const SDK_JS_MODULE_NAMES: &[&str] = _;
```

Package import names authors use for the workerd `BookclerkEntrypoint` SDK.

### SDK_PY_WORKERD_MODULE

```rust
pub const SDK_PY_WORKERD_MODULE: &str = "bookclerk_plugin_sdk/workerd.py";
```

Python package path for `from bookclerk_plugin_sdk.workerd import …`.

### SDK_PY_DB_VALUE_MODULE

```rust
pub const SDK_PY_DB_VALUE_MODULE: &str = "bookclerk_plugin_sdk/db_value.py";
```

Python module path for typed SQL value / execute codec helpers.

### SDK_PY_PRODUCT_ABI_MODULE

```rust
pub const SDK_PY_PRODUCT_ABI_MODULE: &str = "bookclerk_plugin_sdk/_abi.py";
```

Python module path for the generated product ABI constants.

### SDK_PY_GUEST_SQL_MODULE

```rust
pub const SDK_PY_GUEST_SQL_MODULE: &str = "bookclerk_plugin_sdk/guest_sql.py";
```

Python module path for guest-side SQL statement classification helpers.

### SDK_PY_INIT_MODULE

```rust
pub const SDK_PY_INIT_MODULE: &str = "bookclerk_plugin_sdk/__init__.py";
```

Python module path that initializes the sparse workerd guest SDK.

### WORKERD_STATE_DIR_ENV

```rust
pub const WORKERD_STATE_DIR_ENV: &str = bookclerk_sandbox::WORKERD_STATE_DIR_ENV;
```

Env var the host sets to the per-session workerd state directory.
