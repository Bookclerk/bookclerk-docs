**bookclerk_plugin_sdk > error**

# Module: error

## Contents

**Enums**

- [`SdkError`](#sdkerror) - Errors from the guest stdio loop and SDK helpers (not plugin business logic).

**Type Aliases**

- [`Result`](#result) - Result alias for guest helpers that fail with [`SdkError`](error.md#bookclerk_plugin_sdkerrorsdkerror).

---

## bookclerk_plugin_sdk::error::Result

*Type Alias*: `std::result::Result<T, SdkError>`

Result alias for guest helpers that fail with [`SdkError`](error.md#bookclerk_plugin_sdkerrorsdkerror).



## bookclerk_plugin_sdk::error::SdkError

*Enum*

Errors from the guest stdio loop and SDK helpers (not plugin business logic).

Prefer mapping operator-visible failures into
[`bookclerk_plugin_abi::PluginError`](../bookclerk_plugin_abi/error.md#bookclerk_plugin_abierrorpluginerror) inside [`crate::PluginWorker`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolespluginworker)
methods. Use [`SdkError`](error.md#bookclerk_plugin_sdkerrorsdkerror) for framing, JSON, and filesystem problems in the
SDK itself.

**Variants:**
- `Message(String)` - Operator-facing or diagnostic text with no structured ABI error code.
- `Io(std::io::Error)` - Underlying OS I/O failure (stdio, files, sockets) while running SDK helpers.
- `Json(serde_json::Error)` - JSON serialize/deserialize failure for Workers RPC request or response lines.

**Methods:**

- `fn message<impl Into<String>>(msg: impl Trait) -> Self` - Builds a `SdkError::Message` from any displayable string.

**Trait Implementations:**

- **From**
  - `fn from(source: serde_json::Error) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`



