**bookclerk_integrations > error**

# Module: error

## Contents

**Enums**

- [`IntegrationError`](#integrationerror) - Errors from outbound integrations and the connect portal.

**Type Aliases**

- [`Result`](#result) - Result alias for this crate.

---

## bookclerk_integrations::error::IntegrationError

*Enum*

Errors from outbound integrations and the connect portal.

**Variants:**
- `Api{ status: u16, message: String }` - Remote HTTP API returned a non-success status.
- `Library(bookclerk_library::LibraryError)` - Error propagated from [`bookclerk_library`](../bookclerk_library/bookclerk_library.md).
- `Config(bookclerk_config::ConfigError)` - Error propagated from [`bookclerk_config`](../bookclerk_config/index.md).
- `Other(anyhow::Error)` - Opaque error wrapped from `anyhow`.
- `Message(String)` - Operator-facing error text with no structured code.

**Methods:**

- `fn message<impl Into<String>>(msg: impl Trait) -> Self` - Builds an error from a human-readable message string.
- `fn api<impl Into<String>>(status: u16, message: impl Trait) -> Self` - Builds an `IntegrationError::Api` from an HTTP status and response text.

**Trait Implementations:**

- **From**
  - `fn from(source: anyhow::Error) -> Self`
- **From**
  - `fn from(source: bookclerk_config::ConfigError) -> Self`
- **From**
  - `fn from(source: bookclerk_library::LibraryError) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_integrations::error::Result

*Type Alias*: `std::result::Result<T, IntegrationError>`

Result alias for this crate.



