**bookclerk_plugin_catalog > error**

# Module: error

## Contents

**Enums**

- [`CatalogError`](#catalogerror) - Failures from discovery, manifest validation, download, or installation.

**Type Aliases**

- [`Result`](#result) - Result alias for [`CatalogError`](error.md#bookclerk_plugin_catalogerrorcatalogerror).

---

## bookclerk_plugin_catalog::error::CatalogError

*Enum*

Failures from discovery, manifest validation, download, or installation.

**Variants:**
- `Message(String)` - Operator-facing error text with no structured code.
- `ReceiptNotFound` - `receipt.json` is genuinely absent from the install directory.
- `Io(std::io::Error)` - Filesystem I/O failure during download, extract, or activate.
- `Json(serde_json::Error)` - JSON encode/decode failure (static index, receipt, …).
- `Toml(toml::de::Error)` - TOML decode failure for package manifests.
- `Url(url::ParseError)` - Invalid URL in a coordinate, artifact, or index.

**Methods:**

- `fn message<impl Into<String>>(msg: impl Trait) -> Self` - Builds a `CatalogError::Message` from any displayable string.
- `fn is_receipt_not_found(self: &Self) -> bool` - True when this error is a genuinely missing install receipt.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(source: toml::de::Error) -> Self`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **From**
  - `fn from(source: url::ParseError) -> Self`
- **From**
  - `fn from(source: serde_json::Error) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`



## bookclerk_plugin_catalog::error::Result

*Type Alias*: `std::result::Result<T, CatalogError>`

Result alias for [`CatalogError`](error.md#bookclerk_plugin_catalogerrorcatalogerror).



