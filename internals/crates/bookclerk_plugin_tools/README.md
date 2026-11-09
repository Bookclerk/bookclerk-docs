# bookclerk_plugin_tools

Plugin authoring tools behind the `bookclerk-plugin` CLI: `check`, `fmt`,
`sync-embed`, `package`, `smoke`, and `test`.

Audience: plugin maintainers validating `plugin.toml` layout and packaging
archives for distribution. Kept out of `bookclerk-plugin-sdk` so guest
plugins never link `bookclerk-workerd` or the packaging stack.

| Helper | Purpose |
| --- | --- |
| [`check_plugin`](#check_plugin) / [`fmt_plugin_toml`](#fmt_plugin_toml) / [`package_plugin`](#package_plugin) | Validate, format, archive |
| [`sync_embed`](#sync_embed) | Optional workerd JS vendor |
| [`run_tools_cli`](#run_tools_cli) / [`smoke_plugin`](#smoke_plugin) | CLI entry + live workerd smoke |
| [`test_plugin`](#test_plugin) | The plugin's suite under every database profile |

See `docs/plugins/README.md` and the crate README.

| Item | Kind | Summary |
| --- | --- | --- |
| [`MainModuleLanguage`](#mainmodulelanguage) | enum | Source language of a workerd `main_module`, chosen by file extension. |
| [`TestRunner`](#testrunner) | enum | The test runner a plugin directory's layout names. |
| [`check_main_module_source`](#check_main_module_source) | function | Checks a workerd main module against the api_version 3 author model. |
| [`check_plugin`](#check_plugin) | function | Validates `plugin.toml` and the on-disk layout under `plugin_dir`. |
| [`entrypoint_export_class`](#entrypoint_export_class) | function | Exported class name the v3 author model expects for a named entrypoint (`storefront` → `Storefront`, `databaseAdapter` → `DatabaseAdapter`). |
| [`sync_embed`](#sync_embed) | function | Optionally vendors the workerd JS embed under the plugin modules tree. |
| [`run_tools_cli`](#run_tools_cli) | function | Runs the authoring CLI (`check` / `fmt` / `sync-embed` / `package` / `smoke` / `test`). |
| [`package_plugin`](#package_plugin) | function | Packages a plugin directory into `out_dir` as a `.tar.gz` plus `SHA256SUMS`. |
| [`detect_test_runner`](#detect_test_runner) | function | Picks the test runner from the files in `plugin_dir`. |
| [`kit_binary`](#kit_binary) | function | Locates the `bookclerk-db-testkit` binary. |
| [`test_plugin`](#test_plugin) | function | Runs the plugin's test suite once per profile. |
| [`smoke_plugin`](#smoke_plugin) | function | Smokes a `runtime = "workerd"` plugin: ensure pin → materialize → describe + health. |
| [`fmt_plugin_toml`](#fmt_plugin_toml) | function | Formats `plugin.toml` text to the canonical serialized form. |
| [`load_manifest`](#load_manifest) | function | Parses and validates `plugin.toml` text without rewriting it. |
| [`KIT_BINARY_ENV`](#kit_binary_env) | constant | Environment variable the SDK wrappers read for the kit binary's path. |
| [`KIT_PROFILE_ENV`](#kit_profile_env) | constant | Environment variable the SDK wrappers read as their default profile. |
| [`TEST_DATABASE_PROFILES`](#test_database_profiles) | constant | The capability profiles the kit serves, in the order a run reports them. |

## Enums

### MainModuleLanguage

```rust
pub enum MainModuleLanguage {
    Js,
    Python,
}
```

Source language of a workerd `main_module`, chosen by file extension.

**Variants**

- `Js`: `.js` / `.mjs` module authored against `@bookclerk/plugin-sdk/workerd`.
- `Python`: `.py` Python Worker authored against `bookclerk_plugin_sdk.workerd`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### TestRunner

```rust
pub enum TestRunner {
    Npm,
    Pytest,
    Cargo,
}
```

The test runner a plugin directory's layout names.

**Variants**

- `Npm`: `package.json` with a `test` script: `npm test`.
- `Pytest`: `pyproject.toml`: `python -m pytest`.
- `Cargo`: `Cargo.toml`: `cargo test`.

#### TestRunner::argv

```rust
pub fn argv(self) -> Vec<String>
```

The program and leading arguments that run this runner's suite.

##### Returns

The executable name followed by the arguments before any the author
passes after `--`.

#### TestRunner::name

```rust
pub fn name(self) -> &'static str
```

Short name for reports (`npm`, `pytest`, `cargo`).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### check_main_module_source

```rust
pub fn check_main_module_source(
    main_name: &str,
    src: &str,
    entrypoints: &[Entrypoint],
    language: MainModuleLanguage,
) -> Result<()>
```

Checks a workerd main module against the api_version 3 author model.

Mirrors `checkMainModuleSource` in the TypeScript SDK and
`check_main_module_source` in the Python SDK: the module must import the
SDK, must not reference the removed `BookclerkPlugin` base, must not
subclass bare `WorkerEntrypoint`, and must export (JS) or define (Python)
one class per named entrypoint in `entrypoints`.

#### Arguments

* `main_name` - Module file name used in error messages.
* `src` - Module source text.
* `entrypoints` - Named entrypoints declared in `plugin.toml`.
* `language` - Module language chosen from the file extension.

#### Errors

Returns [`SdkError`](../../../plugins/sdk/rust/README.md#sdkerror) describing the first violated rule.

### check_plugin

```rust
pub fn check_plugin(plugin_dir: &Path) -> Result<String>
```

Validates `plugin.toml` and the on-disk layout under `plugin_dir`.

Checks embedded logo paths, native `command` presence when
`.require-binary` is set, and workerd `modules_dir` / `main_module` plus the
api_version 3 author-model heuristics of [`check_main_module_source`](#check_main_module_source).

#### Arguments

* `plugin_dir` - Directory containing `plugin.toml` (usually `.` or the
  staged plugin root).

#### Returns

Human-readable success line (`ok id=… kind=… runtime=…`) for CLI stdout.

#### Errors

Returns [`SdkError`](../../../plugins/sdk/rust/README.md#sdkerror) when the manifest is missing/invalid, required files
are absent, or a workerd main module fails the author-model heuristics
(removed `BookclerkPlugin`, bare `WorkerEntrypoint`, missing entrypoint class).

### entrypoint_export_class

```rust
pub fn entrypoint_export_class(entrypoint: Entrypoint) -> String
```

Exported class name the v3 author model expects for a named entrypoint
(`storefront` → `Storefront`, `databaseAdapter` → `DatabaseAdapter`).

#### Arguments

* `entrypoint` - Named entrypoint from `plugin.toml`.

#### Returns

Class name the adapter isolate resolves through `PLUGIN_<ENTRYPOINT>`.

### sync_embed

```rust
pub fn sync_embed(plugin_dir: &Path) -> Result<String>
```

Optionally vendors the workerd JS embed under the plugin modules tree.

Writes [`bookclerk_plugin_sdk::workerd::EMBED_BOOKCLERK_PLUGIN_JS_SRC`](../../../plugins/sdk/rust/workerd.md#embed_bookclerk_plugin_js_src) to
`modules/@bookclerk/plugin-sdk/workerd.js` (paths from the manifest). Hosts
normally inject this package at runtime — use sync only for offline /
hermetic trees.

#### Arguments

* `plugin_dir` - Workerd plugin root (`runtime = "workerd"`).

#### Returns

Success message including the destination path.

#### Errors

Returns [`SdkError`](../../../plugins/sdk/rust/README.md#sdkerror) when the manifest is not workerd, `main_module` is not
`.js`/`.mjs`, or filesystem writes fail.

### run_tools_cli

```rust
pub fn run_tools_cli() -> ExitCode
```

Runs the authoring CLI (`check` / `fmt` / `sync-embed` / `package` / `smoke` / `test`).

Reads `std::env::args` after the binary name. Unknown commands or missing
required flags print usage to stderr and return exit code `2`.

#### Returns

Process [`ExitCode`]: `0` on success, `1` on command failure, `2` on usage
errors.

#### Examples

```rust
// from src/bin/bookclerk-plugin.rs
fn main() -> std::process::ExitCode {
    bookclerk_plugin_tools::run_tools_cli()
}
```

### package_plugin

```rust
pub fn package_plugin(plugin_dir: &Path, out_dir: &Path) -> Result<PathBuf>
```

Packages a plugin directory into `out_dir` as a `.tar.gz` plus `SHA256SUMS`.

Native guests include the binary named by `command` (mode `0755` on Unix)
and name the archive with the host target triple. Workerd guests copy the
`modules_dir` tree and use a `-workerd` archive stem. Existing
`SHA256SUMS` lines for the same archive name are replaced.

#### Arguments

* `plugin_dir` - Directory containing `plugin.toml` and guest payload.
* `out_dir` - Destination directory for the archive and checksums file
  (created if missing).

#### Returns

Absolute or relative path to the written `.tar.gz` archive.

#### Errors

Returns [`SdkError`](../../../plugins/sdk/rust/README.md#sdkerror) when the manifest is invalid, required binaries /
modules are missing, or archive / checksum I/O fails.

### detect_test_runner

```rust
pub fn detect_test_runner(plugin_dir: &Path) -> Result<TestRunner>
```

Picks the test runner from the files in `plugin_dir`.

`package.json` wins when it declares a `test` script, then
`pyproject.toml`, then `Cargo.toml`, so a TypeScript plugin that also
carries a `Cargo.toml` for a WASM module runs its JavaScript suite.

#### Arguments

* `plugin_dir` - Directory containing `plugin.toml`.

#### Returns

The runner to invoke.

#### Errors

Returns [`SdkError`](../../../plugins/sdk/rust/README.md#sdkerror) when no runner is recognized or `package.json`
cannot be parsed.

### kit_binary

```rust
pub fn kit_binary() -> Result<PathBuf>
```

Locates the `bookclerk-db-testkit` binary.

Order: `BOOKCLERK_DB_TESTKIT`, then a sibling of this executable (a
workspace `target/<profile>` holds both after
`cargo build -p bookclerk-db-testkit`), then `PATH`.

#### Returns

The binary's path.

#### Errors

Returns [`SdkError`](../../../plugins/sdk/rust/README.md#sdkerror) naming the three places searched when none holds the
binary.

### test_plugin

```rust
pub fn test_plugin(
    plugin_dir: &Path,
    profiles: &[String],
    runner_args: &[String],
) -> Result<String>
```

Runs the plugin's test suite once per profile.

Each run inherits stdio and gets `BOOKCLERK_DB_TESTKIT` and
`BOOKCLERK_DB_TESTKIT_PROFILE` in its environment. Every profile runs
even after one fails, so a report names them all.

#### Arguments

* `plugin_dir` - Directory containing `plugin.toml` and the test runner's
  manifest.
* `profiles` - Profile names to run, each one of
  [`TEST_DATABASE_PROFILES`](#test_database_profiles).
* `runner_args` - Extra arguments appended to the runner's command line
  (after `--` for `npm test` and `cargo test`).

#### Returns

One `profile <name>: ok` line per profile for CLI stdout.

#### Errors

Returns [`SdkError`](../../../plugins/sdk/rust/README.md#sdkerror) when a profile name is unknown, no runner is found,
the kit binary is missing, a runner cannot start, or any profile's run
exits non-zero (the message lists the failed profiles).

### smoke_plugin

```rust
pub fn smoke_plugin(plugin_dir: &Path) -> Result<String, String>
```

Smokes a `runtime = "workerd"` plugin: ensure pin → materialize → describe + health.

Spawns unconfined `workerd serve` on a free loopback port (jailed guests use
a listen FD via the launcher instead). Kills the child before returning.

#### Arguments

* `plugin_dir` - Directory containing `plugin.toml` with `runtime = "workerd"`.

#### Returns

Multi-line success text including pretty-printed describe/health JSON.

#### Errors

Returns a plain `String` error when the manifest is not workerd, workerd
cannot be ensured, health never becomes ready, or RPC calls fail.

### fmt_plugin_toml

```rust
pub fn fmt_plugin_toml(text: &str) -> Result<String, Error>
```

Formats `plugin.toml` text to the canonical serialized form.

Parses with the manifest crate, then re-emits a stable key order / spacing
suitable for `bookclerk-plugin fmt` and CI `--check` diffs.

#### Arguments

* `text` - Raw contents of a `plugin.toml` file.

#### Returns

Canonical TOML string (typically ending with a trailing newline).

#### Errors

Returns a manifest parse/format error when the text is invalid TOML or
fails schema validation.

### load_manifest

```rust
pub fn load_manifest(text: &str) -> Result<PluginManifest, Error>
```

Parses and validates `plugin.toml` text without rewriting it.

#### Arguments

* `text` - Raw contents of a `plugin.toml` file.

#### Returns

Structured [`PluginManifest`](../bookclerk_plugin_manifest/types.md#pluginmanifest) ready for inspection.

#### Errors

Returns a manifest error when parsing or validation fails.

## Constants

### KIT_BINARY_ENV

```rust
pub const KIT_BINARY_ENV: &str = "BOOKCLERK_DB_TESTKIT";
```

Environment variable the SDK wrappers read for the kit binary's path.

### KIT_PROFILE_ENV

```rust
pub const KIT_PROFILE_ENV: &str = "BOOKCLERK_DB_TESTKIT_PROFILE";
```

Environment variable the SDK wrappers read as their default profile.

### TEST_DATABASE_PROFILES

```rust
pub const TEST_DATABASE_PROFILES: [&str; 3] = _;
```

The capability profiles the kit serves, in the order a run reports them.

The kit binary owns the definitions (`bookclerk_db_testkit::Profile`); it
refuses a name outside this list, so the two stay aligned by its test.
