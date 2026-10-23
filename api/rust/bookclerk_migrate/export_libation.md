**bookclerk_migrate > export_libation**

# Module: export_libation

## Contents

**Structs**

- [`LibationExportOptions`](#libationexportoptions) - Options for Libation-compatible export.
- [`LibationExportSummary`](#libationexportsummary) - Summary of a Libation export.

**Functions**

- [`export_libation`](#export_libation) - Write Settings.json and LibationContext.db.

---

## bookclerk_migrate::export_libation::LibationExportOptions

*Struct*

Options for Libation-compatible export.

**Fields:**
- `files_dir: std::path::PathBuf` - Bookclerk or Libation files directory root for this operation.
- `dest: std::path::PathBuf` - Destination path for the export archive or directory.
- `force: bool` - When true, overwrite existing data instead of failing on conflict.
- `dry_run: bool` - When true, report what would change without writing files.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> LibationExportOptions`



## bookclerk_migrate::export_libation::LibationExportSummary

*Struct*

Summary of a Libation export.

**Fields:**
- `settings: bool` - Count of settings keys imported or exported.
- `storefronts: usize` - Count of storefront identities exported (JSON key remains `accounts`).
- `books: usize` - Count of book rows imported or exported.
- `warnings: Vec<String>` - Non-fatal warnings collected during the run (operator-facing).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> LibationExportSummary`
- **Clone**
  - `fn clone(self: &Self) -> LibationExportSummary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_migrate::export_libation::export_libation

*Function*

Write Settings.json and LibationContext.db.

`AccountsSettings.json` is written separately by
[`crate::write_classic_storefront_file`](classic_storefront.md#bookclerk_migrateclassic_storefrontwrite_classic_storefront_file) so the operator-facing summary
never carries `list_accounts()` payloads.

# Arguments

* `opts` - Options struct for this operation.

# Returns

On success, the inner `LibationExportSummary` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn export_libation(opts: LibationExportOptions) -> crate::error::Result<LibationExportSummary>
```



