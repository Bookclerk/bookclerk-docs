**bookclerk_acquire > error**

# Module: error

## Contents

**Enums**

- [`AcquireError`](#acquireerror) - Failures from the acquire / convert / reconcile pipeline.

**Type Aliases**

- [`Result`](#result) - Result alias for [`AcquireError`](error.md#bookclerk_acquireerroracquireerror).

---

## bookclerk_acquire::error::AcquireError

*Enum*

Failures from the acquire / convert / reconcile pipeline.

**Variants:**
- `Source(bookclerk_source::SourceError)` - Failure originating in a content-source plugin.
- `Storage(bookclerk_storage::StorageError)` - Failure originating in an object-storage backend.
- `Media(bookclerk_media::MediaError)` - Failure originating in media encode/remux/fixup.
- `Library(bookclerk_library::LibraryError)` - Failure originating in the library database layer.
- `Io(std::io::Error)` - Filesystem or other I/O failure.
- `Other(anyhow::Error)` - Catch-all for otherwise unclassified failures.

**Trait Implementations:**

- **From**
  - `fn from(source: bookclerk_source::SourceError) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(source: anyhow::Error) -> Self`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **From**
  - `fn from(source: bookclerk_library::LibraryError) -> Self`
- **From**
  - `fn from(source: bookclerk_media::MediaError) -> Self`
- **From**
  - `fn from(source: bookclerk_storage::StorageError) -> Self`



## bookclerk_acquire::error::Result

*Type Alias*: `std::result::Result<T, AcquireError>`

Result alias for [`AcquireError`](error.md#bookclerk_acquireerroracquireerror).



