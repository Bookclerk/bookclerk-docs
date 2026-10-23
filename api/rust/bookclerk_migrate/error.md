**bookclerk_migrate > error**

# Module: error

## Contents

**Enums**

- [`MigrateError`](#migrateerror) - Errors from Libation import / native backup export and import.

**Type Aliases**

- [`Result`](#result) - Result alias for this crate's error type.

---

## bookclerk_migrate::error::MigrateError

*Enum*

Errors from Libation import / native backup export and import.

**Variants:**
- `Source(String)` - Failure while reading or interpreting the migration source tree.
- `Settings(String)` - Failure while importing or exporting settings.
- `Accounts(String)` - Failure while importing or exporting accounts / credentials.
- `Library(String)` - Error propagated from [`bookclerk_library`](../bookclerk_library/bookclerk_library.md).
- `Config(bookclerk_config::ConfigError)` - Error propagated from [`bookclerk_config`](../bookclerk_config/index.md).
- `Store(bookclerk_library::LibraryError)` - Failure while writing into the destination library store.
- `Io(std::io::Error)` - Underlying filesystem or stream I/O failure.
- `Sqlite(rusqlite::Error)` - SQLite error while reading a classic Libation database.
- `Other(anyhow::Error)` - Opaque error wrapped from `anyhow`.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(source: anyhow::Error) -> Self`
- **From**
  - `fn from(source: rusqlite::Error) -> Self`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **From**
  - `fn from(source: bookclerk_library::LibraryError) -> Self`
- **From**
  - `fn from(source: bookclerk_config::ConfigError) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`



## bookclerk_migrate::error::Result

*Type Alias*: `std::result::Result<T, MigrateError>`

Result alias for this crate's error type.



