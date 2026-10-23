**bookclerk_library > error**

# Module: error

## Contents

**Enums**

- [`LibraryError`](#libraryerror) - Failures from library store, schema apply, secrets, or config.

**Type Aliases**

- [`Result`](#result) - Result alias for [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror).

---

## bookclerk_library::error::LibraryError

*Enum*

Failures from library store, schema apply, secrets, or config.

**Variants:**
- `Schema(String)` - Schema versioning, backup, or canonical SQL failure.
- `Orm(sea_orm::DbErr)` - SeaORM / database-plugin failure (`DbErr`).
- `Config(bookclerk_config::ConfigError)` - Configuration load or validation failure.
- `Io(std::io::Error)` - Filesystem or other I/O failure.
- `NotFound(String)` - Requested book (or similar library row) was not found.
- `LastAdministrator` - Demoting or disabling the last active administrator is refused.
- `LastOwner` - Demoting, disabling, or deleting the last active owner is refused.
- `InvalidEmail` - Contact email failed library-backed syntax validation.
- `Unavailable(String)` - Backend temporarily unreachable or an atomic RPC response was lost.
- `Conflict(String)` - Identity or ownership conflict (for example a taken OIDC `client_id`).
- `Forbidden(String)` - The caller may not perform this write (for example a configuration
- `Invalid(String)` - The request or stored document failed validation.
- `Other(anyhow::Error)` - Catch-all for otherwise unclassified failures.

**Methods:**

- `fn from_db_err(err: sea_orm::DbErr) -> Self` - Maps a SeaORM / adapter [`sea_orm::DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) onto a typed library error.
- `fn is_schema_apply_retryable(self: &Self) -> bool` - True when host schema apply may retry after re-reading durable state.

**Trait Implementations:**

- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **From**
  - `fn from(source: bookclerk_config::ConfigError) -> Self`
- **From**
  - `fn from(source: sea_orm::DbErr) -> Self`
- **From**
  - `fn from(source: anyhow::Error) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **From**
  - `fn from(source: std::io::Error) -> Self`



## bookclerk_library::error::Result

*Type Alias*: `std::result::Result<T, LibraryError>`

Result alias for [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror).



