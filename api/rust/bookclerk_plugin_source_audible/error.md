**bookclerk_plugin_source_audible > error**

# Module: error

## Contents

**Enums**

- [`AudibleError`](#audibleerror) - Failures from Audible login, library sync, licensing, or download/decrypt.

**Type Aliases**

- [`Result`](#result) - Result alias for Audible auth, sync, license, and download operations.

---

## bookclerk_plugin_source_audible::error::AudibleError

*Enum*

Failures from Audible login, library sync, licensing, or download/decrypt.

**Variants:**
- `Auth(String)` - Login, token refresh, or authenticator load failed.
- `NoAccounts(String)` - No Audible accounts are configured (safe to skip in multi-source scan).
- `AccountNotFound(String)` - Requested account id is not present in `encrypted_secrets`.
- `Import(String)` - Import of an external auth / Libation / mkb79 file failed.
- `Sync(String)` - Library listing or metadata sync against Audible APIs failed.
- `License(String)` - Content license request was rejected or returned an unexpected payload.
- `NoAaxcAsset{ asin: String, message: String }` - Adrm licenserequest returned 000307 — title is Widevine-only (no aaxc).
- `Widevine(String)` - Widevine CDM load, provisioning, or CENC decrypt failed.
- `Download(String)` - HTTP download of audio/PDF/cover bytes failed.
- `Io(std::io::Error)` - Local filesystem I/O while reading or writing cache files.
- `Other(anyhow::Error)` - Unexpected failure wrapped from `anyhow` / library layers.

**Methods:**

- `fn is_no_aaxc_asset(self: &Self) -> bool` - Returns true when this error is `Self::NoAaxcAsset` (retry via Widevine).

**Trait Implementations:**

- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **From**
  - `fn from(err: audible_rs::api::client::ApiError) -> Self`
- **From**
  - `fn from(err: audible_rs::auth::login::LoginError) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(err: bookclerk_library::LibraryError) -> Self`
- **From**
  - `fn from(err: audible_rs::auth::AuthError) -> Self`
- **From**
  - `fn from(source: anyhow::Error) -> Self`
- **From**
  - `fn from(source: std::io::Error) -> Self`



## bookclerk_plugin_source_audible::error::Result

*Type Alias*: `std::result::Result<T, AudibleError>`

Result alias for Audible auth, sync, license, and download operations.



