**bookclerk_plugin_source_libro > error**

# Module: error

## Contents

**Enums**

- [`LibroError`](#libroerror) - Failures from Libro.fm OAuth, library listing, or DRM-free download.

**Type Aliases**

- [`Result`](#result) - Result alias for Libro.fm auth, sync, and download operations.

---

## bookclerk_plugin_source_libro::error::LibroError

*Enum*

Failures from Libro.fm OAuth, library listing, or DRM-free download.

**Variants:**
- `Auth(String)` - Password/token login or refresh failed.
- `NoAccounts(String)` - No Libro.fm accounts are configured (safe to skip in multi-source scan).
- `AccountNotFound(String)` - Requested account id is not present in `encrypted_secrets`.
- `Api(String)` - Upstream mobile API or JSON payload error.
- `Download(String)` - Title download or packaging into the preferred container failed.
- `Io(std::io::Error)` - Local filesystem I/O while reading or writing cache files.
- `Library(bookclerk_library::LibraryError)` - Library database upsert / secret storage failure.
- `Other(anyhow::Error)` - Unexpected failure wrapped from `anyhow`.

**Methods:**

- `fn auth<impl Into<String>>(msg: impl Trait) -> Self` - Builds an `Self::Auth` error from a displayable message.
- `fn no_accounts<impl Into<String>>(msg: impl Trait) -> Self` - Builds a `Self::NoAccounts` error (multi-source scan may skip this store).
- `fn api<impl Into<String>>(msg: impl Trait) -> Self` - Builds an `Self::Api` error from a displayable message.
- `fn download<impl Into<String>>(msg: impl Trait) -> Self` - Builds a `Self::Download` error from a displayable message.

**Trait Implementations:**

- **From**
  - `fn from(err: HttpError) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(err: serde_json::Error) -> Self`
- **From**
  - `fn from(source: anyhow::Error) -> Self`
- **From**
  - `fn from(source: bookclerk_library::LibraryError) -> Self`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`



## bookclerk_plugin_source_libro::error::Result

*Type Alias*: `std::result::Result<T, LibroError>`

Result alias for Libro.fm auth, sync, and download operations.



