**bookclerk_migrate > library**

# Module: library

## Contents

**Structs**

- [`LibraryImportSummary`](#libraryimportsummary) - Counts and warnings from importing a classic `LibationContext.db`.

**Functions**

- [`import_library_db`](#import_library_db) - Import books from classic EF Core SQLite `LibationContext.db`.

---

## bookclerk_migrate::library::LibraryImportSummary

*Struct*

Counts and warnings from importing a classic `LibationContext.db`.

**Fields:**
- `books: usize` - Library rows upserted from classic `LibraryBooks` (deleted rows skipped).
- `acquired: usize` - Titles whose classic `BookStatus` maps to [`AcquireStatus::Acquired`].
- `storage_keys: usize` - Titles that resolved a local audio path under `books_root`.
- `warnings: Vec<String>` - Operator-facing import notes (reserved; currently unused by the importer).



## bookclerk_migrate::library::import_library_db

*Function*

Import books from classic EF Core SQLite `LibationContext.db`.

# Arguments

* `classic_db` - Filesystem path (`classic_db`).
* `store` - `store` input for this call.
* `audio_paths` - Filesystem path (`audio_paths`).
* `books_root` - Filesystem path (`books_root`).
* `account_id_map` - String `account_id_map` for this call.
* `dry_run` - Boolean flag `dry_run`.

# Returns

On success, the inner `LibraryImportSummary` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn import_library_db(classic_db: &std::path::Path, store: &bookclerk_library::LibraryStore, plugin_key: &str, audio_paths: &std::collections::HashMap<String, std::path::PathBuf>, books_root: &std::path::Path, account_id_map: &std::collections::HashMap<(String, String), String>, _dry_run: bool) -> crate::error::Result<LibraryImportSummary>
```



