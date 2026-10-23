**bookclerk_enrich > error**

# Module: error

## Contents

**Enums**

- [`EnrichError`](#enricherror) - Errors from public-catalog enrichment and match scoring.

**Type Aliases**

- [`Result`](#result) - Result alias for this crate's error type.

---

## bookclerk_enrich::error::EnrichError

*Enum*

Errors from public-catalog enrichment and match scoring.

**Variants:**
- `Sync(String)` - Failure during a blocking / sync enrich step.
- `Io(std::io::Error)` - Underlying filesystem or stream I/O failure.
- `Library(bookclerk_library::LibraryError)` - Error propagated from [`bookclerk_library`](../bookclerk_library/bookclerk_library.md).

**Trait Implementations:**

- **From**
  - `fn from(source: bookclerk_library::LibraryError) -> Self`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_enrich::error::Result

*Type Alias*: `std::result::Result<T, EnrichError>`

Result alias for this crate's error type.



