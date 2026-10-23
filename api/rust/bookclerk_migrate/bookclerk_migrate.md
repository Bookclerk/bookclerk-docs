**bookclerk_migrate**

# Module: bookclerk_migrate

## Contents

**Structs**

- [`MigrateOptions`](#migrateoptions) - Options for a full classic Libation Files directory import.
- [`MigrateSummary`](#migratesummary) - Summary of a migration run.

**Functions**

- [`migrate`](#migrate) - Import classic Libation Files into a bookclerk data directory.
- [`resolve_classic_files_dir`](#resolve_classic_files_dir) - Resolve a classic Libation path from CLI / env.

---

## bookclerk_migrate::MigrateOptions

*Struct*

Options for a full classic Libation Files directory import.

**Fields:**
- `source: std::path::PathBuf` - Absolute path to the classic Libation Files directory to import.
- `dest_files_dir: std::path::PathBuf` - Destination files dir (auth, library.db, config.toml).
- `force: bool` - When true, overwrite an existing destination `config.toml`.
- `skip_auth: bool` - Retained for CLI compatibility; credentials are never imported by migrate
- `dry_run: bool` - Report only; do not write.
- `audible_plugin_key: String` - Canonical PluginKey of the Audible storefront plugin. Imported
- `local_plugin_key: Option<String>` - Canonical PluginKey of the `local` destination plugin, when installed.
- `host_config: Option<bookclerk_config::Config>` - Host configuration used to enrol the installed plugins into the

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MigrateOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_migrate::MigrateSummary

*Struct*

Summary of a migration run.

**Fields:**
- `settings_imported: bool` - When true, `Settings.json` was found and applied (or would be, in dry-run).
- `accounts: usize` - Count of accounts imported or exported.
- `credentials: usize` - Count of credential blobs imported.
- `books: usize` - Count of book rows imported or exported.
- `acquired: usize` - Count of books marked acquired after import.
- `storage_keys: usize` - Count of storage-key mappings written.
- `plugin_settings: usize` - Count of plugin instance settings written (`local` root, Audible bitrate).
- `warnings: Vec<String>` - Non-fatal warnings collected during the run (operator-facing).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> MigrateSummary`
- **Clone**
  - `fn clone(self: &Self) -> MigrateSummary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_migrate::migrate

*Function*

Import classic Libation Files into a bookclerk data directory.

# Arguments

* `opts` - Options struct for this operation.

# Returns

On success, the inner `MigrateSummary` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn migrate(opts: MigrateOptions) -> Result<MigrateSummary>
```



## bookclerk_migrate::resolve_classic_files_dir

*Function*

Resolve a classic Libation path from CLI / env.

# Arguments

* `explicit` - Filesystem path (`explicit`).

# Returns

`Some(...)` when found / applicable; otherwise `None`.

```rust
fn resolve_classic_files_dir(explicit: Option<&std::path::Path>) -> Option<std::path::PathBuf>
```



