# bookclerk_migrate

Import/export classic Libation Files and native Bookclerk backups.

## Audience

CLI migration commands and host tooling. Converts classic Libation directories
into Bookclerk `files` layouts, and round-trips native `.tar.zst` backups.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ClassicSource`](#classicsource) | struct | Paths discovered in a classic Libation Files directory. |
| [`LibationExportOptions`](#libationexportoptions) | struct | Options for Libation-compatible export. |
| [`LibationExportSummary`](#libationexportsummary) | struct | Summary of a Libation export. |
| [`NativeExportOptions`](#nativeexportoptions) | struct | Options for exporting a native backup. |
| [`NativeExportSummary`](#nativeexportsummary) | struct | Summary of a native export. |
| [`NativeImportOptions`](#nativeimportoptions) | struct | Options for importing a native backup. |
| [`NativeImportSummary`](#nativeimportsummary) | struct | Summary of a native import. |
| [`ClassicPluginSettings`](#classicpluginsettings) | struct | Classic settings that belong to plugin instances rather than `config.toml`. |
| [`MigrateOptions`](#migrateoptions) | struct | Options for a full classic Libation Files directory import. |
| [`MigrateSummary`](#migratesummary) | struct | Summary of a migration run. |
| [`MigrateError`](#migrateerror) | enum | Errors from Libation import / native backup export and import. |
| [`import_accounts`](#import_accounts) | function | Import account metadata from `AccountsSettings.json`. |
| [`write_classic_storefront_file`](#write_classic_storefront_file) | function | Writes `AccountsSettings.json` from library identity rows. |
| [`discover_source`](#discover_source) | function | Discover importable files under `root`. |
| [`export_libation`](#export_libation) | function | Write Settings.json and LibationContext.db. |
| [`load_audio_paths`](#load_audio_paths) | function | Load audio paths keyed by ASIN. |
| [`import_library_db`](#import_library_db) | function | Import books from classic EF Core SQLite `LibationContext.db`. |
| [`export_native`](#export_native) | function | Export Bookclerk files-dir essentials into a `.tar.gz` archive. |
| [`import_native`](#import_native) | function | Restore a native backup archive into `dest_files_dir`. |
| [`apply_settings_json`](#apply_settings_json) | function | Apply classic Settings.json keys onto `config`. |
| [`config_to_settings_json`](#config_to_settings_json) | function | Best-effort reverse of `apply_settings_json` for Libation Settings.json export. |
| [`load_settings_json`](#load_settings_json) | function | Load Settings.json as a JSON object. |
| [`migrate`](#migrate) | function | Import classic Libation Files into a bookclerk data directory. |
| [`resolve_classic_files_dir`](#resolve_classic_files_dir) | function | Resolve a classic Libation path from CLI / env. |
| [`Result`](#result) | type alias | Result alias for this crate's error type. |
| [`NATIVE_BACKUP_FORMAT_VERSION`](#native_backup_format_version) | constant | Current native backup format version. |

## Structs

### ClassicSource

```rust
pub struct ClassicSource {
    pub root: PathBuf,
    pub settings_json: Option<PathBuf>,
    pub accounts_settings: Option<PathBuf>,
    pub library_db: Option<PathBuf>,
    pub file_locations: Option<PathBuf>,
}
```

Paths discovered in a classic Libation Files directory.

**Fields**

- `root`: Filesystem root of the classic Libation Files directory.
- `settings_json`: Path to Libation `Settings.json` when present.
- `accounts_settings`: Path to Libation `AccountsSettings.json` when present.
- `library_db`: Path to Libation `libation.db` (or equivalent) when present.
- `file_locations`: Path to Libation file-locations JSON when present.

Implements: `Clone`, `Debug`

### LibationExportOptions

```rust
pub struct LibationExportOptions {
    pub files_dir: PathBuf,
    pub dest: PathBuf,
    pub force: bool,
    pub dry_run: bool,
}
```

Options for Libation-compatible export.

**Fields**

- `files_dir`: Bookclerk or Libation files directory root for this operation.
- `dest`: Destination path for the export archive or directory.
- `force`: When true, overwrite existing data instead of failing on conflict.
- `dry_run`: When true, report what would change without writing files.

Implements: `Clone`, `Debug`

### LibationExportSummary

```rust
pub struct LibationExportSummary {
    pub settings: bool,
    pub storefronts: usize,
    pub books: usize,
    pub warnings: Vec<String>,
}
```

Summary of a Libation export.

**Fields**

- `settings`: Count of settings keys imported or exported.
- `storefronts`: Count of storefront identities exported (JSON key remains `accounts`).
- `books`: Count of book rows imported or exported.
- `warnings`: Non-fatal warnings collected during the run (operator-facing).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### NativeExportOptions

```rust
pub struct NativeExportOptions {
    pub files_dir: PathBuf,
    pub dest: PathBuf,
    pub bookclerk_version: String,
    pub include_plugin_manifests: bool,
    pub include_cache: bool,
    pub include_logs: bool,
    pub include_plugin_databases: bool,
}
```

Options for exporting a native backup.

**Fields**

- `files_dir`: Bookclerk or Libation files directory root for this operation.
- `dest`: Destination archive path. Absolute, or relative to the process cwd.
  This is an operator choice and is not jailed under [`Self::files_dir`](#nativeexportoptions).
- `bookclerk_version`: Bookclerk version string recorded in the backup manifest.
- `include_plugin_manifests`: Include `plugins/**/plugin.toml` (not plugin binaries).
- `include_cache`: Include `cache/` (large; off by default).
- `include_logs`: Include `logs/` (off by default).
- `include_plugin_databases`: Include `plugin-databases/` (SQLite binding files; plugin DDL is not migrated).

Implements: `Clone`, `Debug`

### NativeExportSummary

```rust
pub struct NativeExportSummary {
    pub archive: String,
    pub files: usize,
    pub included: Vec<String>,
}
```

Summary of a native export.

**Fields**

- `archive`: Path to the native backup `.tar.zst` (or similar) archive.
- `files`: Count of files packaged or restored.
- `included`: Logical paths included in the archive.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### NativeImportOptions

```rust
pub struct NativeImportOptions {
    pub archive: PathBuf,
    pub dest_files_dir: PathBuf,
    pub force: bool,
    pub dry_run: bool,
}
```

Options for importing a native backup.

**Fields**

- `archive`: Path to the native backup `.tar.zst` (or similar) archive.
- `dest_files_dir`: Destination Bookclerk files directory for import.
- `force`: When true, overwrite existing data instead of failing on conflict.
- `dry_run`: When true, report what would change without writing files.

Implements: `Clone`, `Debug`

### NativeImportSummary

```rust
pub struct NativeImportSummary {
    pub files: usize,
    pub format_version: u32,
    pub warnings: Vec<String>,
}
```

Summary of a native import.

**Fields**

- `files`: Count of files packaged or restored.
- `format_version`: Native backup format version actually read from the archive.
- `warnings`: Non-fatal warnings collected during the run (operator-facing).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### ClassicPluginSettings

```rust
pub struct ClassicPluginSettings {
    pub books_root: Option<PathBuf>,
    pub audible_bitrate: Option<String>,
}
```

Classic settings that belong to plugin instances rather than `config.toml`.

**Fields**

- `books_root`: Libation `Books` directory: the `local` destination's `root`.
- `audible_bitrate`: Libation `FileDownloadQuality` as the Audible plugin's `bitrate`
  (`high` or `normal`).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### MigrateOptions

```rust
pub struct MigrateOptions {
    pub source: PathBuf,
    pub dest_files_dir: PathBuf,
    pub force: bool,
    pub skip_auth: bool,
    pub dry_run: bool,
    pub audible_plugin_key: String,
    pub local_plugin_key: Option<String>,
    pub host_config: Option<Config>,
}
```

Options for a full classic Libation Files directory import.

**Fields**

- `source`: Absolute path to the classic Libation Files directory to import.
- `dest_files_dir`: Destination files dir (auth, library.db, config.toml).
- `force`: When true, overwrite an existing destination `config.toml`.
- `skip_auth`: Retained for CLI compatibility; credentials are never imported by migrate
  (use the Audible plugin: `auth login` / `auth import`).
- `dry_run`: Report only; do not write.
- `audible_plugin_key`: Canonical PluginKey of the Audible storefront plugin. Imported
  accounts and books are owned by that plugin; the caller resolves it
  from the installed plugin (`bookclerk plugins list`).
- `local_plugin_key`: Canonical PluginKey of the `local` destination plugin, when installed.
  The classic `Books` directory is written to its default instance as
  `root`, so acquired files keep landing where Libation put them.
- `host_config`: Host configuration used to enrol the installed plugins into the
  destination database before instance settings are seeded. A fresh
  files directory has no pinned schema and no default instance until a
  host enrols, so without this the `root` and `bitrate` seeds have
  nowhere to go. `None` skips enrollment (tests, dry runs).

Implements: `Clone`, `Debug`

### MigrateSummary

```rust
pub struct MigrateSummary {
    pub settings_imported: bool,
    pub accounts: usize,
    pub credentials: usize,
    pub books: usize,
    pub acquired: usize,
    pub storage_keys: usize,
    pub plugin_settings: usize,
    pub warnings: Vec<String>,
}
```

Summary of a migration run.

**Fields**

- `settings_imported`: When true, `Settings.json` was found and applied (or would be, in dry-run).
- `accounts`: Count of accounts imported or exported.
- `credentials`: Count of credential blobs imported.
- `books`: Count of book rows imported or exported.
- `acquired`: Count of books marked acquired after import.
- `storage_keys`: Count of storage-key mappings written.
- `plugin_settings`: Count of plugin instance settings written (`local` root, Audible bitrate).
- `warnings`: Non-fatal warnings collected during the run (operator-facing).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

## Enums

### MigrateError

```rust
pub enum MigrateError {
    Source(String),
    Settings(String),
    Accounts(String),
    Library(String),
    Config(ConfigError),
    Store(LibraryError),
    Io(Error),
    Sqlite(Error),
    Other(Error),
}
```

Errors from Libation import / native backup export and import.

**Variants**

- `Source`: Failure while reading or interpreting the migration source tree.
- `Settings`: Failure while importing or exporting settings.
- `Accounts`: Failure while importing or exporting accounts / credentials.
- `Library`: Error propagated from [`bookclerk_library`](../bookclerk_library/index.md).
- `Config`: Error propagated from [`bookclerk_config`](../bookclerk_config/index.md).
- `Store`: Failure while writing into the destination library store.
- `Io`: Underlying filesystem or stream I/O failure.
- `Sqlite`: SQLite error while reading a classic Libation database.
- `Other`: Opaque error wrapped from `anyhow`.

Implements: `Debug`, `Display`, `Error`, `From<ConfigError>`, `From<Error>`, `From<LibraryError>`

## Functions

### import_accounts

```rust
pub async fn import_accounts(
    path: &Path,
    store: &LibraryStore,
    plugin_key: &str,
    _force: bool,
    _skip_auth: bool,
    dry_run: bool,
) -> Result<AccountsImportSummary>
```

Import account metadata from `AccountsSettings.json`.

`skip_auth` is retained for CLI compatibility; credentials are never sealed
here (Audible IdentityTokens are not converted in migrate).

#### Arguments

* `path` - Filesystem path involved in this operation.
* `store` - Destination library (opened by the caller through the adapter).
* `force` - When true, overwrite or force a full remote rescan.
* `skip_auth` - Boolean flag `skip_auth`.
* `dry_run` - Boolean flag `dry_run`.

#### Returns

On success, the inner `AccountsImportSummary` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### write_classic_storefront_file

```rust
pub async fn write_classic_storefront_file(
    files_dir: &Path,
    dest_dir: &Path,
) -> Result<()>
```

Writes `AccountsSettings.json` from library identity rows.

#### Arguments

* `files_dir` - Bookclerk files directory containing `library.db`.
* `dest_dir` - Classic Libation Files directory to write into.

#### Returns

`Ok(())` when the file is written, or when `library.db` is absent.

#### Errors

Returns an error when listing identities or writing the JSON file fails.

### discover_source

```rust
pub fn discover_source(root: &Path) -> Result<ClassicSource>
```

Discover importable files under `root`.

Accepts either a classic Libation Files directory or a directory that contains
one of the expected files (for Docker `/config` layouts).

#### Arguments

* `root` - Cargo workspace root directory.

#### Returns

On success, the inner `ClassicSource` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### export_libation

```rust
pub async fn export_libation(opts: LibationExportOptions) -> Result<LibationExportSummary>
```

Write Settings.json and LibationContext.db.

`AccountsSettings.json` is written separately by
[`crate::write_classic_storefront_file`](#write_classic_storefront_file) so the operator-facing summary
never carries `list_accounts()` payloads.

#### Arguments

* `opts` - Options struct for this operation.

#### Returns

On success, the inner `LibationExportSummary` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### load_audio_paths

```rust
pub fn load_audio_paths(path: &Path) -> Result<HashMap<String, PathBuf>>
```

Load audio paths keyed by ASIN.

#### Arguments

* `path` - Filesystem path involved in this operation.

#### Returns

On success, the inner `AudioPathMap` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### import_library_db

```rust
pub async fn import_library_db(
    classic_db: &Path,
    store: &LibraryStore,
    plugin_key: &str,
    audio_paths: &HashMap<String, PathBuf>,
    books_root: &Path,
    account_id_map: &HashMap<(String, String), String>,
    _dry_run: bool,
) -> Result<LibraryImportSummary>
```

Import books from classic EF Core SQLite `LibationContext.db`.

#### Arguments

* `classic_db` - Filesystem path (`classic_db`).
* `store` - `store` input for this call.
* `audio_paths` - Filesystem path (`audio_paths`).
* `books_root` - Filesystem path (`books_root`).
* `account_id_map` - String `account_id_map` for this call.
* `dry_run` - Boolean flag `dry_run`.

#### Returns

On success, the inner `LibraryImportSummary` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### export_native

```rust
pub fn export_native(opts: NativeExportOptions) -> Result<NativeExportSummary>
```

Export Bookclerk files-dir essentials into a `.tar.gz` archive.

#### Arguments

* `opts` - Options struct for this operation.

#### Returns

On success, the inner `NativeExportSummary` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### import_native

```rust
pub fn import_native(opts: NativeImportOptions) -> Result<NativeImportSummary>
```

Restore a native backup archive into `dest_files_dir`.

#### Arguments

* `opts` - Options struct for this operation.

#### Returns

On success, the inner `NativeImportSummary` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### apply_settings_json

```rust
pub fn apply_settings_json(config: &mut Config, settings: &Value) -> ClassicPluginSettings
```

Apply classic Settings.json keys onto `config`.

Keys that configure a plugin (the books directory, the Audible bitrate)
are returned instead so the caller can write them on the plugin instance.

#### Arguments

* `config` - Loaded Bookclerk configuration.
* `settings` - `settings` input for this call.

### config_to_settings_json

```rust
pub fn config_to_settings_json(
    config: &Config,
    plugin_settings: &ClassicPluginSettings,
) -> Value
```

Best-effort reverse of [`apply_settings_json`](#apply_settings_json) for Libation Settings.json export.

#### Arguments

* `config` - Loaded Bookclerk configuration.
* `plugin_settings` - Books directory and Audible bitrate from the plugin
  instances, when the caller could read them.

#### Returns

`Value` result.

### load_settings_json

```rust
pub fn load_settings_json(path: &Path) -> Result<Value>
```

Load Settings.json as a JSON object.

#### Arguments

* `path` - Filesystem path involved in this operation.

#### Returns

On success, the inner `Value` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### migrate

```rust
pub async fn migrate(opts: MigrateOptions) -> Result<MigrateSummary>
```

Import classic Libation Files into a bookclerk data directory.

#### Arguments

* `opts` - Options struct for this operation.

#### Returns

On success, the inner `MigrateSummary` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### resolve_classic_files_dir

```rust
pub fn resolve_classic_files_dir(explicit: Option<&Path>) -> Option<PathBuf>
```

Resolve a classic Libation path from CLI / env.

#### Arguments

* `explicit` - Filesystem path (`explicit`).

#### Returns

`Some(...)` when found / applicable; otherwise `None`.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, MigrateError>;
```

Result alias for this crate's error type.

## Constants

### NATIVE_BACKUP_FORMAT_VERSION

```rust
pub const NATIVE_BACKUP_FORMAT_VERSION: u32 = 1;
```

Current native backup format version.
