# bookclerk_plugin_abi::error

Plugin error model shared by host and guests.

Capability failures serialize as [`PluginError`](#pluginerror) in the Cap'n Proto
`PluginError` struct (or the equivalent workerd reject payload). Codes are
stable `snake_case` strings matching the `PluginErrorCode` enum in
`schema/plugin.capnp`. Unknown future codes are preserved as
[`PluginErrorCode::Unknown`](#pluginerrorcode) plus the raw wire string — they are never
collapsed to [`PluginErrorCode::Internal`](#pluginerrorcode).

| Item | Kind | Summary |
| --- | --- | --- |
| [`PluginError`](#pluginerror) | struct | RPC / plugin failure payload returned to the host. |
| [`PluginErrorCode`](#pluginerrorcode) | enum | Stable error codes for plugin RPC failures (schema `PluginError.code`). |
| [`Result`](#result) | type alias | Result alias for ABI operations that fail as `PluginError`. |

## Structs

### PluginError

```rust
pub struct PluginError {
    pub code: PluginErrorCode,
    pub message: String,
    pub details: Option<Map<String, Value>>,
    pub wire_code: Option<String>,
}
```

RPC / plugin failure payload returned to the host.

Wire shape: `{ "code": "…", "message": "…", "details"?: {…} }`. Display
formats as `{code}: {message}`. Unknown codes keep the raw wire string in
[`Self::wire_code`](#pluginerror) / [`Self::wire_str`](#pluginerrorwire_str).

**Fields**

- `code`: Machine-stable failure category (wire `code`).
- `message`: Operator-facing explanation; must not embed secrets.
- `details`: Optional structured extras (validation paths, store status codes, …).
  Omitted from JSON when `None`.
- `wire_code`: Raw wire code when [`Self::code`](#pluginerror) is [`PluginErrorCode::Unknown`](#pluginerrorcode).

#### PluginError::new

```rust
pub fn new(code: PluginErrorCode, message: impl Into<String>) -> Self
```

Builds an error with the given code and message and no `details`.

##### Arguments

* `code` - Stable [`PluginErrorCode`](#pluginerrorcode) for the wire `code` field.
* `message` - Human-readable explanation shown to operators.

#### PluginError::from_wire

```rust
pub fn from_wire(code: &str, message: impl Into<String>) -> Self
```

Reconstruct from a wire `code` string without collapsing unknown codes.

#### PluginError::wire_str

```rust
pub fn wire_str(&self) -> &str
```

Wire `code` string (raw unknown code when present).

#### PluginError::invalid_params

```rust
pub fn invalid_params(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::InvalidParams`](#pluginerrorcode).

#### PluginError::unsupported

```rust
pub fn unsupported(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::Unsupported`](#pluginerrorcode).

#### PluginError::internal

```rust
pub fn internal(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::Internal`](#pluginerrorcode).

#### PluginError::not_found

```rust
pub fn not_found(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::NotFound`](#pluginerrorcode).

#### PluginError::unavailable

```rust
pub fn unavailable(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::Unavailable`](#pluginerrorcode).

Use for ambiguous transport failures (lost HTTP/RPC responses, timeouts)
where the caller should retry the same idempotency key.

#### PluginError::payload_too_large

```rust
pub fn payload_too_large(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::PayloadTooLarge`](#pluginerrorcode).

#### PluginError::deadline_exceeded

```rust
pub fn deadline_exceeded(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::DeadlineExceeded`](#pluginerrorcode).

#### PluginError::forbidden

```rust
pub fn forbidden(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::Forbidden`](#pluginerrorcode).

#### PluginError::invalid_cursor

```rust
pub fn invalid_cursor(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::InvalidCursor`](#pluginerrorcode).

#### PluginError::cancelled

```rust
pub fn cancelled(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::Cancelled`](#pluginerrorcode).

#### PluginError::conflict

```rust
pub fn conflict(message: impl Into<String>) -> Self
```

Convenience for [`PluginErrorCode::Conflict`](#pluginerrorcode).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Error`, `PartialEq`, `Serialize`

## Enums

### PluginErrorCode

```rust
pub enum PluginErrorCode {
    InvalidParams,
    Unauthorized,
    Forbidden,
    NotFound,
    Unavailable,
    Unsupported,
    Internal,
    PayloadTooLarge,
    DeadlineExceeded,
    InvalidCursor,
    Cancelled,
    Conflict,
    Unknown,
}
```

Stable error codes for plugin RPC failures (schema `PluginError.code`).

Serialized with `snake_case` wire names (`invalid_params`, `not_found`, …).

**Variants**

- `InvalidParams`: Request params failed validation or are missing required fields.
- `Unauthorized`: Caller is not authenticated for this method (credentials / token).
- `Forbidden`: Caller is authenticated but not allowed to perform this operation.
- `NotFound`: Requested account, object key, session, or row does not exist.
- `Unavailable`: Backend or dependency is temporarily unreachable (store API, DB, …).
- `Unsupported`: Method or capability is not implemented by this guest.
- `Internal`: Unexpected guest or host failure; see [`PluginError::message`](#pluginerror).
- `PayloadTooLarge`: A scalar RPC value exceeded [`crate::MAX_SCALAR_BYTES`](index.md#max_scalar_bytes).
- `DeadlineExceeded`: The invocation deadline elapsed before the call completed.
- `InvalidCursor`: List cursor is missing, stale, or not from this backend.
- `Cancelled`: The invocation was cancelled (host fence / guest abort).
- `Conflict`: The operation conflicts with current state (conditional put, …).
- `Unknown`: Unrecognized wire code. See [`PluginError::wire_str`](#pluginerrorwire_str).

#### PluginErrorCode::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical wire / schema string for this known code.

#### PluginErrorCode::from_wire

```rust
pub fn from_wire(code: &str) -> Self
```

Maps a wire `code` string onto a known variant, or [`Self::Unknown`](#pluginerrorcode).

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `PartialEq`, `Serialize`

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, PluginError>;
```

Result alias for ABI operations that fail as [`PluginError`](#pluginerror).
