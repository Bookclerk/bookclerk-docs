**bookclerk_source > error**

# Module: error

## Contents

**Enums**

- [`SourceError`](#sourceerror) - Errors from content-source auth, scan, or fetch.

**Type Aliases**

- [`Result`](#result) - Result alias for content-source operations.

---

## bookclerk_source::error::Result

*Type Alias*: `std::result::Result<T, SourceError>`

Result alias for content-source operations.



## bookclerk_source::error::SourceError

*Enum*

Errors from content-source auth, scan, or fetch.

**Variants:**
- `NoAccounts(String)` - No accounts are configured for this source (safe to skip in multi-source scan).
- `Auth(String)` - Authentication or credential failure (operator-facing message).
- `Api(String)` - Upstream store HTTP / protocol failure (operator-facing message).
- `Io(std::io::Error)` - Local filesystem failure while caching or writing downloads.
- `Library(bookclerk_library::LibraryError)` - Library database / secret-store failure.
- `Other(anyhow::Error)` - Catch-all for plugin-internal failures wrapped as [`anyhow::Error`](https://docs.rs/anyhow/latest/anyhow/struct.Error.html).

**Methods:**

- `fn no_accounts<impl Into<String>>(msg: impl Trait) -> Self` - Construct `Self::NoAccounts` with an operator-facing message.
- `fn auth<impl Into<String>>(msg: impl Trait) -> Self` - Construct `Self::Auth` with an operator-facing message.
- `fn api<impl Into<String>>(msg: impl Trait) -> Self` - Construct `Self::Api` with an operator-facing message.

**Trait Implementations:**

- **From**
  - `fn from(source: std::io::Error) -> Self`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **From**
  - `fn from(source: bookclerk_library::LibraryError) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(source: anyhow::Error) -> Self`



