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
| [`LimitExceeded`](#limitexceeded) | struct | One exceeded negotiated limit: which cap, its value, and the value that exceeded it. |
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

#### PluginError::limit_exceeded

```rust
pub fn limit_exceeded(limit: LimitExceeded) -> Self
```

Builds a [`PluginErrorCode::LimitExceeded`](#pluginerrorcode) error whose `details`
carry `limit`, `cap` and `used` and whose message is
[`LimitExceeded::to_message`](#limitexceededto_message).

#### PluginError::limit

```rust
pub fn limit(&self) -> Option<LimitExceeded>
```

The exceeded limit when this is a [`PluginErrorCode::LimitExceeded`](#pluginerrorcode)
error.

Reads the structured `details` first and falls back to the message
text, so an error relayed by a hop that dropped `details` still
names its limit.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Error`, `PartialEq`, `Serialize`

### LimitExceeded

```rust
pub struct LimitExceeded {
    pub limit: String,
    pub cap: u64,
    pub used: u64,
}
```

One exceeded negotiated limit: which cap, its value, and the value that
exceeded it.

Every emitter (adapter SDK, host reply checks, SDK request checks) formats
the same message through [`Self::to_message`](#limitexceededto_message), and every classifier parses
it back through [`Self::from_message`](#limitexceededfrom_message), so a limit error stays typed across
a `DbErr` string hop.

**Fields**

- `limit`: Capability field name as advertised (`maxResultBytes`, `maxResultRows`,
  `maxRequestBytes`, ...).
- `cap`: The negotiated cap.
- `used`: The observed value that exceeded `cap`.

#### LimitExceeded::new

```rust
pub fn new(limit: impl Into<String>, cap: u64, used: u64) -> Self
```

Builds the record.

#### LimitExceeded::to_message

```rust
pub fn to_message(&self) -> String
```

Canonical message: `limit exceeded: maxResultBytes is 262144; used 300000`.

#### LimitExceeded::from_message

```rust
pub fn from_message(message: &str) -> Option<Self>
```

Parses a message produced by [`Self::to_message`](#limitexceededto_message), possibly followed by
further text after a `;` or `:`.

#### LimitExceeded::details

```rust
pub fn details(&self) -> Map<String, Value>
```

The `details` object: `{ "limit": ..., "cap": ..., "used": ... }`.

#### LimitExceeded::from_details

```rust
pub fn from_details(details: &Map<String, Value>) -> Option<Self>
```

Reads the record back from a `details` object; `None` when any field
is missing or not of the expected type, or when `limit` is not a
name [`Self::from_message`](#limitexceededfrom_message) would accept, so a record read from
`details` always round-trips through its canonical message.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

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
    LimitExceeded,
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
- `PayloadTooLarge`: A scalar RPC value exceeded [`crate::MAX_SCALAR_BYTES`](README.md#max_scalar_bytes).
- `DeadlineExceeded`: The invocation deadline elapsed before the call completed.
- `InvalidCursor`: List cursor is missing, stale, or not from this backend.
- `Cancelled`: The invocation was cancelled (host fence / guest abort).
- `Conflict`: The operation conflicts with current state (conditional put, …).
- `LimitExceeded`: A negotiated database limit was exceeded; [`PluginError::limit`](#pluginerrorlimit)
  names which one.
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
