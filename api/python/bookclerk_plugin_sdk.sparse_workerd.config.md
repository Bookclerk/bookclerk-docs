# bookclerk\_plugin\_sdk.sparse\_workerd.config

Materialize Cap'n Proto workerd config + bridge (mirrors config.rs).

Writes ``.bookclerk/`` bridge + adapter assets and a Cap'n Proto config that embeds plugin
modules plus the injected Python/JS SDK. Used by the sparse smoke launcher and
authoring tools that do not ship the Rust ``bookclerk-workerd`` binary.

#### SDK\_JS\_MODULE\_NAMES

Module names used when embedding the TypeScript workerd SDK.

#### SDK\_PY\_WORKERD\_MODULE

Isolate module path for the Python workerd ``BookclerkEntrypoint`` base.

#### SDK\_PY\_INIT\_MODULE

Isolate module path for the package ``__init__`` stub.

#### PYODIDE\_EGRESS\_HOSTS

Extra egress hosts allowed for Python Workers / Pyodide package fetch.

#### SDK\_PY\_INIT

Source text embedded as ``bookclerk_plugin_sdk/__init__.py`` in the isolate.

#### ADAPTER\_JS

First-party adapter isolate module (mirrors ``ADAPTER_JS`` in ``config.rs``).

#### ENTRYPOINT\_SERVICE\_BINDINGS

``(binding, exported class)`` the adapter receives per manifest entrypoint.

#### manifest\_capabilities

```python
def manifest_capabilities(m: dict[str, Any]) -> dict[str, Any]
```

Typed capability declaration a manifest implies (mirrors Rust ``capabilities()``).

**Arguments**:

- `m` - Validated manifest dict.
  

**Returns**:

  Wire-shaped ``PluginCapabilities`` object.

#### manifest\_describe\_json

```python
def manifest_describe_json(m: dict[str, Any]) -> str
```

``PLUGIN_DESCRIBE`` JSON the adapter answers ``describe()`` from.

**Arguments**:

- `m` - Validated manifest dict.
  

**Returns**:

  Serialized wire-shaped ``PluginDescribe`` projection.

#### escape\_capnp

```python
def escape_capnp(s: str) -> str
```

Escape a string for embedding inside Cap'n Proto double quotes.

**Arguments**:

- `s` - Raw string to escape.
  

**Returns**:

  Escaped string safe for Cap'n Proto text literals.

#### is\_legacy\_sdk\_embed

```python
def is_legacy_sdk_embed(name: str) -> bool
```

Return whether ``name`` is a legacy vendored SDK path to skip.

**Arguments**:

- `name` - Module path relative to the plugin ``modules/`` directory.
  

**Returns**:

  ``True`` if the file should be omitted in favor of host injection.

#### module\_field\_for

```python
def module_field_for(name: str) -> tuple[str, bool]
```

Map a module filename to its Cap'n Proto embed field.

**Arguments**:

- `name` - Module path (used for extension detection).
  

**Returns**:

  A ``(field_name, is_python)`` pair such as ``("pythonModule", True)``.
  

**Raises**:

- `ValueError` - When the extension is not implemented yet.

#### collect\_modules

```python
def collect_modules(directory: Path,
                    *,
                    plugin_root: Path | None = None) -> list[Path]
```

Collect embeddable module files under ``directory``.

Symlinks are refused (matching the Rust workerd walker) so a nested
``modules/leak -> /outside`` cannot recurse or embed files outside the
plugin install tree. The modules root itself is checked against
``plugin_root`` (or ``directory`` when omitted) so a symlinked modules
directory cannot promote an outside tree into the embed set.

**Arguments**:

- `directory` - Plugin modules root to walk recursively.
- `plugin_root` - Original trusted plugin root; defaults to ``directory``.
  

**Returns**:

  Sorted list of ``.js``/``.mjs``/``.py``/``.wasm``/``.json`` file paths.
  

**Raises**:

- `ValueError` - When a directory entry or the modules root is a symlink.

#### plugin\_global\_outbound

```python
def plugin_global_outbound(mode: str) -> str
```

Map network mode to the plugin worker ``globalOutbound`` service name.

**Arguments**:

- `mode` - Manifest network mode (``outbound`` or deny-style).
  

**Returns**:

  ``"egress"`` for outbound mode, otherwise ``"blocked"``.

#### egress\_domains\_for

```python
def egress_domains_for(needs_python: bool, mode: str,
                       base: list[str]) -> list[str]
```

Build the egress allow-list, adding Pyodide hosts when needed.

**Arguments**:

- `needs_python` - Whether the plugin embeds Python modules.
- `mode` - Network mode from the manifest.
- `base` - Domains declared in ``capabilities.network.domains``.
  

**Returns**:

  Domain list possibly extended with :data:`PYODIDE_EGRESS_HOSTS`.

## GeneratedConfig

```python
class GeneratedConfig(NamedTuple)
```

Paths produced by :func:`materialize_config`.

#### allocate\_workerd\_state\_dir

```python
def allocate_workerd_state_dir(plugin_root: Path) -> Path
```

Allocate a unique writable session directory for workerd generated embeds.

Keys the leaf by a short hash of the plugin root plus a random suffix from
``tempfile.mkdtemp`` under the OS temp dir — never mkdir of a raw operator
``TMPDIR`` / plugin-root join (local threat-model path sink).

**Arguments**:

- `plugin_root` - Plugin install root (used only as an opaque id seed).
  

**Returns**:

  Absolute session directory.

#### materialize\_config

```python
def materialize_config(
        plugin_root: Path,
        manifest: dict[str, Any],
        *,
        listen_port: int,
        bridge_token: str,
        sdk_root: Path | None = None,
        state_dir: Path | None = None,
        config_name: str = "workerd-config.capnp") -> GeneratedConfig
```

Write bridge assets and Cap'n Proto under a host session directory.

Generated embeds (``.bookclerk/``, Cap'n Proto) are written under
``state_dir`` (allocated under ``$TMPDIR`` when omitted). Author modules
stay on the read-only install root and are referenced via Cap'n Proto
``/modules/…`` embeds resolved with ``workerd serve --import-path``.

**Arguments**:

- `plugin_root` - Plugin directory that contains ``modules/``.
- `manifest` - Parsed ``plugin.toml`` mapping (must include ``[workerd]``).
- `listen_port` - Loopback TCP port for the bridge RPC socket.
- `bridge_token` - Bearer token required by the bridge HTTP surface.
- `sdk_root` - SDK package root for bridge/SDK embeds (defaults to this package).
- `state_dir` - Existing session directory, or ``None`` to allocate one.
- `config_name` - Output Cap'n Proto filename under the session directory.
  

**Returns**:

  :class:`GeneratedConfig` with config path, listen address, session dir,
  and import path for ``--import-path``.
  

**Raises**:

- `ValueError` - If ``[workerd]`` is missing, the compatibility date or flags
  are rejected, KV or Queues are declared, or ``bridge_token`` is empty.
- `FileNotFoundError` - If bridge assets, modules, main module, or SDK embeds
  cannot be found.

