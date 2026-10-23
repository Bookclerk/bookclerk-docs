**bookclerk_migrate > native**

# Module: native

## Contents

**Structs**

- [`NativeExportOptions`](#nativeexportoptions) - Options for exporting a native backup.
- [`NativeExportSummary`](#nativeexportsummary) - Summary of a native export.
- [`NativeImportOptions`](#nativeimportoptions) - Options for importing a native backup.
- [`NativeImportSummary`](#nativeimportsummary) - Summary of a native import.

**Functions**

- [`export_native`](#export_native) - Export Bookclerk files-dir essentials into a `.tar.gz` archive.
- [`import_native`](#import_native) - Restore a native backup archive into `dest_files_dir`.

**Constants**

- [`NATIVE_BACKUP_FORMAT_VERSION`](#native_backup_format_version) - Current native backup format version.

---

## bookclerk_migrate::native::NATIVE_BACKUP_FORMAT_VERSION

*Constant*: `u32`

Current native backup format version.



## bookclerk_migrate::native::NativeExportOptions

*Struct*

Options for exporting a native backup.

**Fields:**
- `files_dir: std::path::PathBuf` - Bookclerk or Libation files directory root for this operation.
- `dest: std::path::PathBuf` - Destination archive path. Absolute, or relative to the process cwd.
- `bookclerk_version: String` - Bookclerk version string recorded in the backup manifest.
- `include_plugin_manifests: bool` - Include `plugins/**/plugin.toml` (not plugin binaries).
- `include_cache: bool` - Include `cache/` (large; off by default).
- `include_logs: bool` - Include `logs/` (off by default).
- `include_plugin_databases: bool` - Include `plugin-databases/` (SQLite binding files; plugin DDL is not migrated).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NativeExportOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_migrate::native::NativeExportSummary

*Struct*

Summary of a native export.

**Fields:**
- `archive: String` - Path to the native backup `.tar.zst` (or similar) archive.
- `files: usize` - Count of files packaged or restored.
- `included: Vec<String>` - Logical paths included in the archive.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> NativeExportSummary`
- **Clone**
  - `fn clone(self: &Self) -> NativeExportSummary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_migrate::native::NativeImportOptions

*Struct*

Options for importing a native backup.

**Fields:**
- `archive: std::path::PathBuf` - Path to the native backup `.tar.zst` (or similar) archive.
- `dest_files_dir: std::path::PathBuf` - Destination Bookclerk files directory for import.
- `force: bool` - When true, overwrite existing data instead of failing on conflict.
- `dry_run: bool` - When true, report what would change without writing files.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NativeImportOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_migrate::native::NativeImportSummary

*Struct*

Summary of a native import.

**Fields:**
- `files: usize` - Count of files packaged or restored.
- `format_version: u32` - Native backup format version actually read from the archive.
- `warnings: Vec<String>` - Non-fatal warnings collected during the run (operator-facing).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> NativeImportSummary`
- **Clone**
  - `fn clone(self: &Self) -> NativeImportSummary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_migrate::native::export_native

*Function*

Export Bookclerk files-dir essentials into a `.tar.gz` archive.

# Arguments

* `opts` - Options struct for this operation.

# Returns

On success, the inner `NativeExportSummary` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn export_native(opts: NativeExportOptions) -> crate::error::Result<NativeExportSummary>
```



## bookclerk_migrate::native::import_native

*Function*

Restore a native backup archive into `dest_files_dir`.

# Arguments

* `opts` - Options struct for this operation.

# Returns

On success, the inner `NativeImportSummary` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn import_native(opts: NativeImportOptions) -> crate::error::Result<NativeImportSummary>
```



