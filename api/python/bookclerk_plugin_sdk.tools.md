# bookclerk_plugin_sdk.tools

check / fmt / types / package / sync-embed — mirrors Rust/TS author tools.

Validates `plugin.toml`, formats manifests, generates the `Env` typing stub,
vendors workerd embeds, and packs release archives. Workerd authors:

- `from bookclerk_plugin_sdk.workerd import BookclerkEntrypoint, js`
  (`bookclerk-workerd` injects that module — no relative filepath embed)

See `docs/plugins.md` for manifest fields and runtime requirements.

| Name | Kind | Summary |
| --- | --- | --- |
| [`AppliedCompatibilityDate`](#appliedcompatibilitydate) | class | Date passed to workerd, and a fallback warning when the author date is newer. |
| [`validate_author_compatibility_date`](#validate_author_compatibility_date) | function | Reject a compatibility date that is not a real `YYYY-MM-DD`. |
| [`apply_author_compatibility_date`](#apply_author_compatibility_date) | function | Resolve an author compatibility date against this Bookclerk release. |
| [`validate_author_compatibility_flags`](#validate_author_compatibility_flags) | function | Enforce the Python flag pair. Does not insert missing flags. |
| [`declares_python`](#declares_python) | function | True when the manifest declares Python. Flags are not evidence. |
| [`unimplemented_surface`](#unimplemented_surface) | function | Spawn/load refusal for a declared-but-unbuilt surface. |
| [`is_valid_queue_name`](#is_valid_queue_name) | function | True when `name` is `[a-z0-9][a-z0-9._-]*` and at most 63 chars. |
| [`validate_module_declarations`](#validate_module_declarations) | function | Check main and `[[modules]]` extensions. |
| [`module_load_key`](#module_load_key) | function | Relative key a modules-directory walk uses for a `[[modules]]` path. |
| [`workerd_module_is_embedded`](#workerd_module_is_embedded) | function | True when the walk embeds this filename. |
| [`validate_logo`](#validate_logo) | function | Classify and validate a `plugin.toml` logo value. |
| [`validate_plugin_id`](#validate_plugin_id) | function | Validate a plugin id against the strict `[a-z0-9_]{2,32}` grammar. |
| [`validate_manifest`](#validate_manifest) | function | Validate a parsed `plugin.toml` mapping. |
| [`check_main_module_source`](#check_main_module_source) | function | Check a workerd main module against the v3 author model. |
| [`check_plugin`](#check_plugin) | function | Validate a plugin directory and its `plugin.toml`. |
| [`sync_embed`](#sync_embed) | function | Vendor SDK sources under `modules/` for offline workerd archives. |
| [`format_manifest`](#format_manifest) | function | Emit canonical TOML matching Rust `format_manifest` gold fixtures. |
| [`fmt_plugin_toml`](#fmt_plugin_toml) | function | Format `plugin.toml` in place, or check that it is already canonical. |
| [`package_plugin`](#package_plugin) | function | Pack a plugin into a `.tar.gz` archive and update `SHA256SUMS`. |
| [`env_properties_for`](#env_properties_for) | function | Compute the `Env` protocol members for a validated manifest. |
| [`render_env_types`](#render_env_types) | function | Render the `bookclerk_configuration.py` typing stub for a validated manifest. |
| [`generate_types`](#generate_types) | function | Write `bookclerk_configuration.py` beside a plugin's `plugin.toml`. |
| [`PYTHON_WORKERD_FLAGS`](#python_workerd_flags) | constant | Compatibility flags required for Python Workers under bookclerk-workerd. |
| [`WORKERD_PIN_COMPAT_DATE`](#workerd_pin_compat_date) | constant | Newest `compatibility_date` the pinned workerd binary honors. |
| [`MAX_QUEUE_NAME_LEN`](#max_queue_name_len) | constant | Longest queue name. |
| [`RESERVED_QUEUE_PREFIXES`](#reserved_queue_prefixes) | constant | Queue-name prefixes only the host may use. |
| [`HOST_EVENT_TYPES`](#host_event_types) | constant | Domain event types only the host emits; plugins consume them but cannot produce them. |
| [`MAX_KV_NAMESPACE_VALUE_BYTES`](#max_kv_namespace_value_bytes) | constant | Largest `max_value_bytes` a `[[kv_namespaces]]` row may declare (1 MiB). |
| [`MAX_KV_NAMESPACE_KEYS`](#max_kv_namespace_keys) | constant | Largest `max_keys` a `[[kv_namespaces]]` row may declare. |
| [`LOGO_EXTENSIONS`](#logo_extensions) | constant | Allowed file extensions for embedded `plugin.toml` logo paths. |
| [`ENTRYPOINT_NAMES`](#entrypoint_names) | constant | Entrypoint wire names accepted in `entrypoints`. |
| [`ENTRYPOINT_EXPORT_CLASSES`](#entrypoint_export_classes) | constant | Exported class name the launcher binds for each `entrypoints` wire name. |
| [`TYPES_OUTPUT_FILE`](#types_output_file) | constant | Default `types` output filename next to `plugin.toml`. |

## Classes

### AppliedCompatibilityDate

```python
class AppliedCompatibilityDate(NamedTuple)
```

Date passed to workerd, and a fallback warning when the author date is newer.

Host surfaces (events, jobs, and later bindings) must follow `applied`.
That is the date workerd actually runs.

**Fields**

- `applied: str`
- `warning: str | None`

## Functions

### validate_author_compatibility_date

```python
def validate_author_compatibility_date(date: str) -> None
```

Reject a compatibility date that is not a real `YYYY-MM-DD`.

A date newer than the pin is still valid. Load falls back and warns; see
[`apply_author_compatibility_date`](#apply_author_compatibility_date).

**Args**

- `date`: `workerd.compatibility_date`.

**Raises**

- `ValueError`: When the date is not a calendar day.

### apply_author_compatibility_date

```python
def apply_author_compatibility_date(date: str) -> AppliedCompatibilityDate
```

Resolve an author compatibility date against this Bookclerk release.

workerd enables flags whose default-on date is on or before the date it is
given, and refuses a date newer than the one baked into the binary.
Wrangler warns and starts at the newest date that binary supports.

**Args**

- `date`: `workerd.compatibility_date`.

**Returns**

The date to pass to workerd, and a warning when it was clamped.

**Raises**

- `ValueError`: When the date is not a calendar day.

### validate_author_compatibility_flags

```python
def validate_author_compatibility_flags(flags: list[Any], python: bool) -> None
```

Enforce the Python flag pair. Does not insert missing flags.

**Args**

- `flags`: Author `compatibility_flags`.
- `python`: True when the manifest declares a Python module. A disk-only `.py` file is not enough.

**Raises**

- `ValueError`: When a flag is outside the allowlist or the pair is wrong.

### declares_python

```python
def declares_python(m: dict[str, Any]) -> bool
```

True when the manifest declares Python. Flags are not evidence.

An explicit `path` is the load-set key. `name` is that key only when
`path` is omitted.

**Args**

- `m`: Manifest dictionary.

**Returns**

Whether Python flags and Pyodide consent hosts apply.

### unimplemented_surface

```python
def unimplemented_surface(m: dict[str, Any]) -> str | None
```

Spawn/load refusal for a declared-but-unbuilt surface.

Every manifest surface is implemented today (`[[kv_namespaces]]` and
`[queues]` included); the hook stays so a future table can refuse load
with "not implemented yet" without touching the callers.

**Args**

- `m`: Manifest dictionary.

**Returns**

The refusal, or `None` when load may continue.

### is_valid_queue_name

```python
def is_valid_queue_name(name: str) -> bool
```

True when `name` is `[a-z0-9][a-z0-9._-]*` and at most 63 chars.

**Args**

- `name`: Candidate queue name.

**Returns**

Whether the grammar accepts it.

### validate_module_declarations

```python
def validate_module_declarations(main_module: str, modules: list[Any]) -> None
```

Check main and `[[modules]]` extensions.

**Args**

- `main_module`: `[workerd].main_module`.
- `modules`: `[[modules]]` rows.

**Raises**

- `ValueError`: When a file would not be embedded or its type disagrees.

### module_load_key

```python
def module_load_key(modules_dir: str, raw: str) -> str
```

Relative key a modules-directory walk uses for a `[[modules]]` path.

`.` segments are dropped, then a leading modules-dir prefix is removed,
so `./modules/index.js` and `modules/pkg/./echo.wasm` match walk keys.
`..` segments are left in place.

**Args**

- `modules_dir`: `[workerd].modules_dir`.
- `raw`: Author path or name.

**Returns**

Slash-separated key with a leading modules-dir prefix removed.

### workerd_module_is_embedded

```python
def workerd_module_is_embedded(path: str) -> bool
```

True when the walk embeds this filename.

**Args**

- `path`: Module path or filename.

**Returns**

Whether the extension is `.js`, `.mjs`, `.py`, `.wasm`, or `.json`.

### validate_logo

```python
def validate_logo(raw: str) -> tuple[str, str]
```

Classify and validate a `plugin.toml` logo value.

Mirrors Rust `validate_logo`. Accepts absolute `http`/`https` URLs or
relative image paths under the plugin root.

**Args**

- `raw`: Raw `logo` string from the manifest.

**Returns**

A `(\"remote\"|\"embedded\", value)` pair with the validated URL or
normalized relative path.

**Raises**

- `ValueError`: If the logo is empty, uses a bad scheme, includes userinfo, or is an unsafe / non-image embedded path.

### validate_plugin_id

```python
def validate_plugin_id(id: str) -> None
```

Validate a plugin id against the strict `[a-z0-9_]{2,32}` grammar.

Mirrors Rust `validate_plugin_id`. Ids are globally unique across plugins.
Invalid characters are rejected — never rewritten — so `a/b` and `a_b`
cannot collide. Leading/trailing whitespace is rejected (non-lossy), not
stripped.

**Args**

- `id`: Candidate plugin id from `plugin.toml`.

**Raises**

- `ValueError`: If the id fails length, charset, or underscore rules.

### validate_manifest

```python
def validate_manifest(m: dict[str, Any]) -> None
```

Validate a parsed `plugin.toml` mapping.

**Args**

- `m`: Manifest dictionary (typically from `tomllib.loads`).

**Raises**

- `ValueError`: If required fields, runtime tables, or network capabilities are missing or inconsistent.

### check_main_module_source

```python
def check_main_module_source(
    main_name: str,
    src: str,
    entrypoints: list[str],
    language: str,
) -> None
```

Check a workerd main module against the v3 author model.

Requires the SDK import and a `BookclerkEntrypoint` default class, rejects
the removed `BookclerkPlugin` base, and requires a class per manifest
entrypoint (`class Storage(StorageEntrypoint)` for Python,
`export class Storage` for JavaScript).

**Args**

- `main_name`: Main module filename (for messages).
- `src`: Main module source text.
- `entrypoints`: Manifest `entrypoints` wire names.
- `language`: `"python"` or `"js"`.

**Raises**

- `ValueError`: When the module does not follow the author model.

### check_plugin

```python
def check_plugin(plugin_dir: Path) -> str
```

Validate a plugin directory and its `plugin.toml`.

**Args**

- `plugin_dir`: Path to the plugin root containing `plugin.toml`.

**Returns**

A short `ok id=... entrypoints=... runtime=...` status string.

**Raises**

- `ValueError`: If the manifest or Python workerd sources are invalid.
- `FileNotFoundError`: If required logo, modules, or native binaries are missing.
- `OSError`: If `plugin.toml` cannot be read.

**Examples**

```python
>>> # print(check_plugin(Path("./my-plugin")))
>>> # ok id=echo entrypoints=storefront runtime=workerd
```

### sync_embed

```python
def sync_embed(plugin_dir: Path) -> str
```

Vendor SDK sources under `modules/` for offline workerd archives.

Prefer package imports — `bookclerk-workerd` injects
`bookclerk_plugin_sdk.workerd` at runtime. This writes the same files so a
staged tree is self-contained without host injection. It does not insert
compatibility flags; the host and `check` require the author to write them.

**Args**

- `plugin_dir`: Path to a workerd Python plugin root.

**Returns**

Status string describing the synced path (and flag updates, if any).

**Raises**

- `ValueError`: If the plugin is not a Python workerd guest.
- `OSError`: If files cannot be read or written.

**Examples**

```python
>>> # print(sync_embed(Path("./my-python-workerd-plugin")))
```

### format_manifest

```python
def format_manifest(m: dict[str, Any]) -> str
```

Emit canonical TOML matching Rust `format_manifest` gold fixtures.

Flags are emitted as written. This function does not insert the Python pair.

**Args**

- `m`: Validated manifest dictionary.

**Returns**

Canonical `plugin.toml` text ending with a newline.

### fmt_plugin_toml

```python
def fmt_plugin_toml(path: Path, *, check_only: bool) -> str
```

Format `plugin.toml` in place, or check that it is already canonical.

**Args**

- `path`: Path to `plugin.toml`.
- `check_only`: When `True`, raise if reformatting would change the file.

**Returns**

Status string (`ok …` or `wrote …`).

**Raises**

- `ValueError`: If validation fails or `check_only` finds a drift.
- `OSError`: If the file cannot be read or written.

### package_plugin

```python
def package_plugin(plugin_dir: Path, out_dir: Path) -> Path
```

Pack a plugin into a `.tar.gz` archive and update `SHA256SUMS`.

**Args**

- `plugin_dir`: Path to the plugin root.
- `out_dir`: Destination directory for the archive and checksums file.

**Returns**

Path to the created `.tar.gz` archive.

**Raises**

- `ValueError`: If the manifest is invalid.
- `FileNotFoundError`: If a required native binary, modules tree, or logo is missing.
- `subprocess.CalledProcessError`: If `tar` fails.

**Examples**

```python
>>> # archive = package_plugin(Path("./my-plugin"), Path("./dist"))
>>> # print(f"packed {archive}")
```

### env_properties_for

```python
def env_properties_for(m: dict[str, Any]) -> list[tuple[str, str, str]]
```

Compute the `Env` protocol members for a validated manifest.

**Args**

- `m`: Validated manifest dict.

**Returns**

`(binding, python_type, doc)` tuples in stable emission order.

**Raises**

- `ValueError`: When a `[[databases]]` binding collides with a reserved name or a binding is declared twice.

### render_env_types

```python
def render_env_types(m: dict[str, Any]) -> str
```

Render the `bookclerk_configuration.py` typing stub for a validated manifest.

The stub declares an `Env` `typing.Protocol` naming every binding
the manifest grants so editors and type checkers can follow
`self.env.<BINDING>` inside `BookclerkEntrypoint` subclasses.

**Args**

- `m`: Validated manifest dict.

**Returns**

Python source text (ends with a newline).

**Raises**

- `ValueError`: When binding names collide.

### generate_types

```python
def generate_types(plugin_dir: Path, out_file: Path | None=None) -> str
```

Write `bookclerk_configuration.py` beside a plugin's `plugin.toml`.

**Args**

- `plugin_dir`: Plugin root containing `plugin.toml`.
- `out_file`: Optional output path (default `<plugin_dir>/bookclerk_configuration.py`).

**Returns**

Status string naming the written file.

**Raises**

- `ValueError`: When the manifest is invalid or bindings collide.
- `OSError`: If files cannot be read or written.

## Constants

### PYTHON_WORKERD_FLAGS

```python
PYTHON_WORKERD_FLAGS = ('python_workers', 'disable_python_external_sdk')
```

Compatibility flags required for Python Workers under bookclerk-workerd.

### WORKERD_PIN_COMPAT_DATE

```python
WORKERD_PIN_COMPAT_DATE = '2026-10-15'
```

Newest `compatibility_date` the pinned workerd binary honors.

### MAX_QUEUE_NAME_LEN

```python
MAX_QUEUE_NAME_LEN = 63
```

Longest queue name.

### RESERVED_QUEUE_PREFIXES

```python
RESERVED_QUEUE_PREFIXES = ('core.', 'node.')
```

Queue-name prefixes only the host may use.

### HOST_EVENT_TYPES

```python
HOST_EVENT_TYPES = frozenset({'book_acquired'})
```

Domain event types only the host emits; plugins consume them but cannot produce them.

### MAX_KV_NAMESPACE_VALUE_BYTES

```python
MAX_KV_NAMESPACE_VALUE_BYTES = 1048576
```

Largest `max_value_bytes` a `[[kv_namespaces]]` row may declare (1 MiB).

### MAX_KV_NAMESPACE_KEYS

```python
MAX_KV_NAMESPACE_KEYS = 1000000
```

Largest `max_keys` a `[[kv_namespaces]]` row may declare.

### LOGO_EXTENSIONS

```python
LOGO_EXTENSIONS = ('.png', '.jpg', '.jpeg', '.gif', '.webp', '.svg', '.ico')
```

Allowed file extensions for embedded `plugin.toml` logo paths.

### ENTRYPOINT_NAMES

```python
ENTRYPOINT_NAMES: tuple[str, ...] = ('storefront', 'storage', 'databaseAdapter', 'remoteLibrary', 'cli', 'oidc')
```

Entrypoint wire names accepted in `entrypoints`.

### ENTRYPOINT_EXPORT_CLASSES

```python
ENTRYPOINT_EXPORT_CLASSES: dict[str, str] = {'storefront': 'Storefront', 'storage': 'Storage', 'databaseAdapter': 'Databa...
```

Exported class name the launcher binds for each `entrypoints` wire name.

### TYPES_OUTPUT_FILE

```python
TYPES_OUTPUT_FILE = 'bookclerk_configuration.py'
```

Default `types` output filename next to `plugin.toml`.
