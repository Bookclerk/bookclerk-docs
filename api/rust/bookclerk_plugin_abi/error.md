**bookclerk_plugin_abi > error**

# Module: error

## Contents

**Structs**

- [`PluginError`](#pluginerror) - RPC / plugin failure payload returned to the host.

**Enums**

- [`PluginErrorCode`](#pluginerrorcode) - Stable error codes for plugin RPC failures (schema `PluginError.code`).

**Type Aliases**

- [`Result`](#result) - Result alias for ABI operations that fail as [`PluginError`](error.md#bookclerk_plugin_abierrorpluginerror).

---

## bookclerk_plugin_abi::error::PluginError

*Struct*

RPC / plugin failure payload returned to the host.

Wire shape: `{ "code": "…", "message": "…", "details"?: {…} }`. Display
formats as `{code}: {message}`. Unknown codes keep the raw wire string in
`Self::wire_code` / `Self::wire_str`.

**Fields:**
- `code: PluginErrorCode` - Machine-stable failure category (wire `code`).
- `message: String` - Operator-facing explanation; must not embed secrets.
- `details: Option<serde_json::Map<String, serde_json::Value>>` - Optional structured extras (validation paths, store status codes, …).
- `wire_code: Option<String>` - Raw wire code when `Self::code` is `PluginErrorCode::Unknown`.

**Methods:**

- `fn new<impl Into<String>>(code: PluginErrorCode, message: impl Trait) -> Self` - Builds an error with the given code and message and no `details`.
- `fn from_wire<impl Into<String>>(code: &str, message: impl Trait) -> Self` - Reconstruct from a wire `code` string without collapsing unknown codes.
- `fn wire_str(self: &Self) -> &str` - Wire `code` string (raw unknown code when present).
- `fn invalid_params<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::InvalidParams`.
- `fn unsupported<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::Unsupported`.
- `fn internal<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::Internal`.
- `fn not_found<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::NotFound`.
- `fn unavailable<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::Unavailable`.
- `fn payload_too_large<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::PayloadTooLarge`.
- `fn deadline_exceeded<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::DeadlineExceeded`.
- `fn forbidden<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::Forbidden`.
- `fn invalid_cursor<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::InvalidCursor`.
- `fn cancelled<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::Cancelled`.
- `fn conflict<impl Into<String>>(message: impl Trait) -> Self` - Convenience for `PluginErrorCode::Conflict`.

**Traits:** Eq, Error

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Serialize**
  - `fn serialize<S>(self: &Self, serializer: S) -> std::result::Result<<S as >::Ok, <S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginError) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PluginError`
- **Deserialize**
  - `fn deserialize<D>(deserializer: D) -> std::result::Result<Self, <D as >::Error>`



## bookclerk_plugin_abi::error::PluginErrorCode

*Enum*

Stable error codes for plugin RPC failures (schema `PluginError.code`).

Serialized with `snake_case` wire names (`invalid_params`, `not_found`, …).

**Variants:**
- `InvalidParams` - Request params failed validation or are missing required fields.
- `Unauthorized` - Caller is not authenticated for this method (credentials / token).
- `Forbidden` - Caller is authenticated but not allowed to perform this operation.
- `NotFound` - Requested account, object key, session, or row does not exist.
- `Unavailable` - Backend or dependency is temporarily unreachable (store API, DB, …).
- `Unsupported` - Method or capability is not implemented by this guest.
- `Internal` - Unexpected guest or host failure; see `PluginError::message`.
- `PayloadTooLarge` - A scalar RPC value exceeded [`crate::MAX_SCALAR_BYTES`](limits.md#bookclerk_plugin_abilimitsmax_scalar_bytes).
- `DeadlineExceeded` - The invocation deadline elapsed before the call completed.
- `InvalidCursor` - List cursor is missing, stale, or not from this backend.
- `Cancelled` - The invocation was cancelled (host fence / guest abort).
- `Conflict` - The operation conflicts with current state (conditional put, …).
- `Unknown` - Unrecognized wire code. See `PluginError::wire_str`.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the canonical wire / schema string for this known code.
- `fn from_wire(code: &str) -> Self` - Maps a wire `code` string onto a known variant, or `Self::Unknown`.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<D>(deserializer: D) -> std::result::Result<Self, <D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginErrorCode) -> bool`
- **Serialize**
  - `fn serialize<S>(self: &Self, serializer: S) -> std::result::Result<<S as >::Ok, <S as >::Error>`
- **Display**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> PluginErrorCode`



## bookclerk_plugin_abi::error::Result

*Type Alias*: `std::result::Result<T, PluginError>`

Result alias for ABI operations that fail as [`PluginError`](error.md#bookclerk_plugin_abierrorpluginerror).



