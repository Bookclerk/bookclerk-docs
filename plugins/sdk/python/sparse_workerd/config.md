# bookclerk_plugin_sdk.sparse_workerd.config

Materialize Cap'n Proto workerd config + bridge (mirrors config.rs).

Writes `.bookclerk/` bridge + adapter assets and a Cap'n Proto config that embeds plugin
modules plus the injected Python/JS SDK. Used by the sparse smoke launcher and
authoring tools that do not ship the Rust `bookclerk-workerd` binary.

| Name | Kind | Summary |
| --- | --- | --- |
| [`GeneratedConfig`](#generatedconfig) | class | Paths produced by `materialize_config`. |
| [`manifest_capabilities`](#manifest_capabilities) | function | Typed capability declaration a manifest implies (mirrors Rust `capabilities()`). |
| [`manifest_describe_json`](#manifest_describe_json) | function | `PLUGIN_DESCRIBE` JSON the adapter answers `describe()` from. |
| [`escape_capnp`](#escape_capnp) | function | Escape a string for embedding inside Cap'n Proto double quotes. |
| [`is_legacy_sdk_embed`](#is_legacy_sdk_embed) | function | Return whether `name` is a legacy vendored SDK path to skip. |
| [`module_field_for`](#module_field_for) | function | Map a module filename to its Cap'n Proto embed field. |
| [`collect_modules`](#collect_modules) | function | Collect embeddable module files under `directory`. |
| [`plugin_global_outbound`](#plugin_global_outbound) | function | Map network mode to the plugin worker `globalOutbound` service name. |
| [`egress_domains_for`](#egress_domains_for) | function | Build the egress allow-list, adding Pyodide hosts when needed. |
| [`allocate_workerd_state_dir`](#allocate_workerd_state_dir) | function | Allocate a unique writable session directory for workerd generated embeds. |
| [`materialize_config`](#materialize_config) | function | Write bridge assets and Cap'n Proto under a host session directory. |
| [`SDK_JS_MODULE_NAMES`](#sdk_js_module_names) | constant | Module names used when embedding the TypeScript workerd SDK. |
| [`SDK_PY_WORKERD_MODULE`](#sdk_py_workerd_module) | constant | Isolate module path for the Python workerd `BookclerkEntrypoint` base. |
| [`SDK_PY_INIT_MODULE`](#sdk_py_init_module) | constant | Isolate module path for the package `__init__` stub. |
| [`PYODIDE_EGRESS_HOSTS`](#pyodide_egress_hosts) | constant | Extra egress hosts allowed for Python Workers / Pyodide package fetch. |
| [`SDK_PY_INIT`](#sdk_py_init) | constant | Source text embedded as `bookclerk_plugin_sdk/__init__.py` in the isolate. |
| [`ADAPTER_JS`](#adapter_js) | constant | First-party adapter isolate module (mirrors `ADAPTER_JS` in `config.rs`). |
| [`ENTRYPOINT_SERVICE_BINDINGS`](#entrypoint_service_bindings) | constant | `(binding, exported class)` the adapter receives per manifest entrypoint. |

## Classes

### GeneratedConfig

```python
class GeneratedConfig(NamedTuple)
```

Paths produced by [`materialize_config`](#materialize_config).

**Fields**

- `config_path: Path`
- `listen_addr: str`
- `state_dir: Path`
- `import_path: Path`

## Functions

### manifest_capabilities

```python
def manifest_capabilities(m: dict[str, Any]) -> dict[str, Any]
```

Typed capability declaration a manifest implies (mirrors Rust `capabilities()`).

**Args**

- `m`: Validated manifest dict.

**Returns**

Wire-shaped `PluginCapabilities` object.

### manifest_describe_json

```python
def manifest_describe_json(m: dict[str, Any]) -> str
```

`PLUGIN_DESCRIBE` JSON the adapter answers `describe()` from.

**Args**

- `m`: Validated manifest dict.

**Returns**

Serialized wire-shaped `PluginDescribe` projection.

### escape_capnp

```python
def escape_capnp(s: str) -> str
```

Escape a string for embedding inside Cap'n Proto double quotes.

**Args**

- `s`: Raw string to escape.

**Returns**

Escaped string safe for Cap'n Proto text literals.

### is_legacy_sdk_embed

```python
def is_legacy_sdk_embed(name: str) -> bool
```

Return whether `name` is a legacy vendored SDK path to skip.

**Args**

- `name`: Module path relative to the plugin `modules/` directory.

**Returns**

`True` if the file should be omitted in favor of host injection.

### module_field_for

```python
def module_field_for(name: str) -> tuple[str, bool]
```

Map a module filename to its Cap'n Proto embed field.

**Args**

- `name`: Module path (used for extension detection).

**Returns**

A `(field_name, is_python)` pair such as `("pythonModule", True)`.

**Raises**

- `ValueError`: When the extension is not implemented yet.

### collect_modules

```python
def collect_modules(directory: Path, *, plugin_root: Path | None=None) -> list[Path]
```

Collect embeddable module files under `directory`.

Symlinks are refused (matching the Rust workerd walker) so a nested
`modules/leak -> /outside` cannot recurse or embed files outside the
plugin install tree. The modules root itself is checked against
`plugin_root` (or `directory` when omitted) so a symlinked modules
directory cannot promote an outside tree into the embed set.

**Args**

- `directory`: Plugin modules root to walk recursively.
- `plugin_root`: Original trusted plugin root; defaults to `directory`.

**Returns**

Sorted list of `.js`/`.mjs`/`.py`/`.wasm`/`.json` file paths.

**Raises**

- `ValueError`: When a directory entry or the modules root is a symlink.

### plugin_global_outbound

```python
def plugin_global_outbound(mode: str) -> str
```

Map network mode to the plugin worker `globalOutbound` service name.

**Args**

- `mode`: Manifest network mode (`outbound` or deny-style).

**Returns**

`"egress"` for outbound mode, otherwise `"blocked"`.

### egress_domains_for

```python
def egress_domains_for(needs_python: bool, mode: str, base: list[str]) -> list[str]
```

Build the egress allow-list, adding Pyodide hosts when needed.

**Args**

- `needs_python`: Whether the plugin embeds Python modules.
- `mode`: Network mode from the manifest.
- `base`: Domains declared in `capabilities.network.domains`.

**Returns**

Domain list possibly extended with [`PYODIDE_EGRESS_HOSTS`](#pyodide_egress_hosts).

### allocate_workerd_state_dir

```python
def allocate_workerd_state_dir(plugin_root: Path) -> Path
```

Allocate a unique writable session directory for workerd generated embeds.

Keys the leaf by a short hash of the plugin root plus a random suffix from
`tempfile.mkdtemp` under the OS temp dir — never mkdir of a raw operator
`TMPDIR` / plugin-root join (local threat-model path sink).

**Args**

- `plugin_root`: Plugin install root (used only as an opaque id seed).

**Returns**

Absolute session directory.

### materialize_config

```python
def materialize_config(
    plugin_root: Path,
    manifest: dict[str, Any],
    *,
    listen_port: int,
    bridge_token: str,
    sdk_root: Path | None=None,
    state_dir: Path | None=None,
    config_name: str='workerd-config.capnp',
) -> GeneratedConfig
```

Write bridge assets and Cap'n Proto under a host session directory.

Generated embeds (`.bookclerk/`, Cap'n Proto) are written under
`state_dir` (allocated under `$TMPDIR` when omitted). Author modules
stay on the read-only install root and are referenced via Cap'n Proto
`/modules/…` embeds resolved with `workerd serve --import-path`.

**Args**

- `plugin_root`: Plugin directory that contains `modules/`.
- `manifest`: Parsed `plugin.toml` mapping (must include `[workerd]`).
- `listen_port`: Loopback TCP port for the bridge RPC socket.
- `bridge_token`: Bearer token required by the bridge HTTP surface.
- `sdk_root`: SDK package root for bridge/SDK embeds (defaults to this package).
- `state_dir`: Existing session directory, or `None` to allocate one.
- `config_name`: Output Cap'n Proto filename under the session directory.

**Returns**

[`GeneratedConfig`](#generatedconfig) with config path, listen address, session dir,
and import path for `--import-path`.

**Raises**

- `ValueError`: If `[workerd]` is missing, the compatibility date or flags are rejected, KV or Queues are declared, or `bridge_token` is empty.
- `FileNotFoundError`: If bridge assets, modules, main module, or SDK embeds cannot be found.

## Constants

### SDK_JS_MODULE_NAMES

```python
SDK_JS_MODULE_NAMES = ('@bookclerk/plugin-sdk/workerd', '@bookclerk/plugin-sdk')
```

Module names used when embedding the TypeScript workerd SDK.

### SDK_PY_WORKERD_MODULE

```python
SDK_PY_WORKERD_MODULE = 'bookclerk_plugin_sdk/workerd.py'
```

Isolate module path for the Python workerd `BookclerkEntrypoint` base.

### SDK_PY_INIT_MODULE

```python
SDK_PY_INIT_MODULE = 'bookclerk_plugin_sdk/__init__.py'
```

Isolate module path for the package `__init__` stub.

### PYODIDE_EGRESS_HOSTS

```python
PYODIDE_EGRESS_HOSTS = ('cdn.jsdelivr.net', 'pypi.org', 'files.pythonhosted.org')
```

Extra egress hosts allowed for Python Workers / Pyodide package fetch.

### SDK_PY_INIT

```python
SDK_PY_INIT = '"""Bookclerk plugin SDK (workerd isolate).\n\nUse: from bookclerk_plugin_sdk...
```

Source text embedded as `bookclerk_plugin_sdk/__init__.py` in the isolate.

### ADAPTER_JS

```python
ADAPTER_JS = 'import { wrapPluginFromBinding } from "@bookclerk/plugin-sdk/workerd";\nexpo...
```

First-party adapter isolate module (mirrors `ADAPTER_JS` in `config.rs`).

### ENTRYPOINT_SERVICE_BINDINGS

```python
ENTRYPOINT_SERVICE_BINDINGS: dict[str, tuple[str, str]] = {'storefront': ('PLUGIN_STOREFRONT', 'Storefront'), 'storage': ('PLUGIN_STORA...
```

`(binding, exported class)` the adapter receives per manifest entrypoint.
