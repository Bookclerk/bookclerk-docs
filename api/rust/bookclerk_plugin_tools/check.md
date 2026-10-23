**bookclerk_plugin_tools > check**

# Module: check

## Contents

**Enums**

- [`MainModuleLanguage`](#mainmodulelanguage) - Source language of a workerd `main_module`, chosen by file extension.

**Functions**

- [`check_main_module_source`](#check_main_module_source) - Checks a workerd main module against the api_version 3 author model.
- [`check_plugin`](#check_plugin) - Validates `plugin.toml` and the on-disk layout under `plugin_dir`.
- [`entrypoint_export_class`](#entrypoint_export_class) - Exported class name the v3 author model expects for a named entrypoint
- [`sync_embed`](#sync_embed) - Optionally vendors the workerd JS embed under the plugin modules tree.

---

## bookclerk_plugin_tools::check::MainModuleLanguage

*Enum*

Source language of a workerd `main_module`, chosen by file extension.

**Variants:**
- `Js` - `.js` / `.mjs` module authored against `@bookclerk/plugin-sdk/workerd`.
- `Python` - `.py` Python Worker authored against `bookclerk_plugin_sdk.workerd`.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MainModuleLanguage`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &MainModuleLanguage) -> bool`



## bookclerk_plugin_tools::check::check_main_module_source

*Function*

Checks a workerd main module against the api_version 3 author model.

Mirrors `checkMainModuleSource` in the TypeScript SDK and
`check_main_module_source` in the Python SDK: the module must import the
SDK, must not reference the removed `BookclerkPlugin` base, must not
subclass bare `WorkerEntrypoint`, and must export (JS) or define (Python)
one class per named entrypoint in `entrypoints`.

# Arguments

* `main_name` - Module file name used in error messages.
* `src` - Module source text.
* `entrypoints` - Named entrypoints declared in `plugin.toml`.
* `language` - Module language chosen from the file extension.

# Errors

Returns [`SdkError`](../bookclerk_plugin_sdk/error.md#bookclerk_plugin_sdkerrorsdkerror) describing the first violated rule.

```rust
fn check_main_module_source(main_name: &str, src: &str, entrypoints: &[bookclerk_plugin_manifest::Entrypoint], language: MainModuleLanguage) -> bookclerk_plugin_sdk::Result<()>
```



## bookclerk_plugin_tools::check::check_plugin

*Function*

Validates `plugin.toml` and the on-disk layout under `plugin_dir`.

Checks embedded logo paths, native `command` presence when
`.require-binary` is set, and workerd `modules_dir` / `main_module` plus the
api_version 3 author-model heuristics of [`check_main_module_source`](check.md#bookclerk_plugin_toolscheckcheck_main_module_source).

# Arguments

* `plugin_dir` - Directory containing `plugin.toml` (usually `.` or the
  staged plugin root).

# Returns

Human-readable success line (`ok id=… kind=… runtime=…`) for CLI stdout.

# Errors

Returns [`SdkError`](../bookclerk_plugin_sdk/error.md#bookclerk_plugin_sdkerrorsdkerror) when the manifest is missing/invalid, required files
are absent, or a workerd main module fails the author-model heuristics
(removed `BookclerkPlugin`, bare `WorkerEntrypoint`, missing entrypoint class).

```rust
fn check_plugin(plugin_dir: &std::path::Path) -> bookclerk_plugin_sdk::Result<String>
```



## bookclerk_plugin_tools::check::entrypoint_export_class

*Function*

Exported class name the v3 author model expects for a named entrypoint
(`storefront` → `Storefront`, `databaseAdapter` → `DatabaseAdapter`).

# Arguments

* `entrypoint` - Named entrypoint from `plugin.toml`.

# Returns

Class name the adapter isolate resolves through `PLUGIN_<ENTRYPOINT>`.

```rust
fn entrypoint_export_class(entrypoint: bookclerk_plugin_manifest::Entrypoint) -> String
```



## bookclerk_plugin_tools::check::sync_embed

*Function*

Optionally vendors the workerd JS embed under the plugin modules tree.

Writes [`bookclerk_plugin_sdk::workerd::EMBED_BOOKCLERK_PLUGIN_JS_SRC`](../bookclerk_plugin_sdk/workerd.md#bookclerk_plugin_sdkworkerdembed_bookclerk_plugin_js_src) to
`modules/@bookclerk/plugin-sdk/workerd.js` (paths from the manifest). Hosts
normally inject this package at runtime — use sync only for offline /
hermetic trees.

# Arguments

* `plugin_dir` - Workerd plugin root (`runtime = "workerd"`).

# Returns

Success message including the destination path.

# Errors

Returns [`SdkError`](../bookclerk_plugin_sdk/error.md#bookclerk_plugin_sdkerrorsdkerror) when the manifest is not workerd, `main_module` is not
`.js`/`.mjs`, or filesystem writes fail.

```rust
fn sync_embed(plugin_dir: &std::path::Path) -> bookclerk_plugin_sdk::Result<String>
```



