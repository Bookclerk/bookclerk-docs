**bookclerk_library > backend_migrate**

# Module: backend_migrate

## Contents

**Structs**

- [`BackendMigrateOptions`](#backendmigrateoptions) - Options for a backend-to-backend library copy.
- [`BackendMigrateSummary`](#backendmigratesummary) - Per-table row counts copied (or that would be copied in dry-run mode).

**Functions**

- [`migrate_library_backend`](#migrate_library_backend) - Copy all library tables from `source` into `dest`.

---

## bookclerk_library::backend_migrate::BackendMigrateOptions

*Struct*

Options for a backend-to-backend library copy.

**Fields:**
- `dry_run: bool` - When true, report counts only — do not write to the destination.
- `force: bool` - Allow copying into a destination that already has library rows.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BackendMigrateOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::backend_migrate::BackendMigrateSummary

*Struct*

Per-table row counts copied (or that would be copied in dry-run mode).

**Fields:**
- `tables: std::collections::BTreeMap<String, usize>` - Map of table name → rows copied (or counted in dry-run).
- `dry_run: bool` - When true, counts rows without writing to the destination.

**Methods:**

- `fn total_rows(self: &Self) -> usize` - Sums row counts across all tables in the summary.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> BackendMigrateSummary`
- **Clone**
  - `fn clone(self: &Self) -> BackendMigrateSummary`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::backend_migrate::migrate_library_backend

*Function*

Copy all library tables from `source` into `dest`.

The destination should be empty unless `BackendMigrateOptions::force` is set.
Runs in a single transaction on the destination when not dry-run.

# Arguments

* `source` - Existing library backend to read from.
* `dest` - Target backend to write into (must already be migrated).
* `opts` - Dry-run / force behavior.

# Returns

Per-table row counts copied (or that would be copied when `dry_run`).

# Errors

Returns [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror) when the destination is non-empty without `force`,
or when any read/write fails.

```rust
fn migrate_library_backend(source: &sea_orm::DatabaseConnection, dest: &sea_orm::DatabaseConnection, opts: &BackendMigrateOptions) -> crate::error::Result<BackendMigrateSummary>
```



