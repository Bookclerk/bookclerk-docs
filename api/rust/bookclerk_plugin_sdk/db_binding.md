**bookclerk_plugin_sdk > db_binding**

# Module: db_binding

## Contents

**Structs**

- [`D1ExecResult`](#d1execresult) - Cloudflare [`D1ExecResult`](https://developers.cloudflare.com/d1/worker-api/return-object/).
- [`D1Meta`](#d1meta) - Cloudflare [`D1Result::meta`](https://developers.cloudflare.com/d1/worker-api/return-object/) shape.
- [`D1Result`](#d1result) - Cloudflare [`D1Result`](https://developers.cloudflare.com/d1/worker-api/return-object/) projection.
- [`DatabaseBinding`](#databasebinding) - Host-mediated typed SQL surface for plugin guests.
- [`DatabaseBindingOptions`](#databasebindingoptions) - Options for [`DatabaseBinding`](db_binding.md#bookclerk_plugin_sdkdb_bindingdatabasebinding).
- [`PreparedStatement`](#preparedstatement) - Cloudflare-style prepared statement.
- [`RetryToken`](#retrytoken) - Explicit retry identity: reuse both `operationId` and `requestHash`.

**Functions**

- [`execute_reply_to_d1_results`](#execute_reply_to_d1_results) - Maps a typed execute reply to Cloudflare [`D1Result`](db_binding.md#bookclerk_plugin_sdkdb_bindingd1result) per statement.
- [`statement_result_to_d1_result`](#statement_result_to_d1_result) - Maps one typed statement result to Cloudflare [`D1Result`](db_binding.md#bookclerk_plugin_sdkdb_bindingd1result).

---

## bookclerk_plugin_sdk::db_binding::D1ExecResult

*Struct*

Cloudflare [`D1ExecResult`](https://developers.cloudflare.com/d1/worker-api/return-object/).

**Fields:**
- `count: u32` - Number of statements executed (one per non-empty `exec` line).
- `duration: f64` - Total duration in milliseconds.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> D1ExecResult`
- **PartialEq**
  - `fn eq(self: &Self, other: &D1ExecResult) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_sdk::db_binding::D1Meta

*Struct*

Cloudflare [`D1Result::meta`](https://developers.cloudflare.com/d1/worker-api/return-object/) shape.

**Fields:**
- `duration: f64` - Engine-reported SQL duration in milliseconds.
- `changes: u64` - Rows changed by the statement (`rowsAffected`).
- `last_row_id: i64` - Last inserted row id when the adapter exposes it (else `0`).
- `changed_db: bool` - `true` when `changes > 0`.
- `rows_read: u32` - Rows returned to the guest for this statement.
- `rows_written: u64` - Rows written (`changes` for DML).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> D1Meta`
- **PartialEq**
  - `fn eq(self: &Self, other: &D1Meta) -> bool`



## bookclerk_plugin_sdk::db_binding::D1Result

*Struct*

Cloudflare [`D1Result`](https://developers.cloudflare.com/d1/worker-api/return-object/) projection.

**Fields:**
- `success: bool` - Always `true` on success; errors are returned as `Result::Err`.
- `results: Option<Vec<std::collections::HashMap<String, bookclerk_plugin_abi::DbValue>>>` - Row maps for selects; `None` when not applicable (DML).
- `meta: D1Meta` - Cloudflare-shaped timing and change metadata.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> D1Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &D1Result) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_sdk::db_binding::DatabaseBinding

*Struct*

Host-mediated typed SQL surface for plugin guests.

**Methods:**

- `fn from_session(session: Arc<dyn GuestDatabase>) -> Self` - Wraps a host-granted [`GuestDatabase`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolesguestdatabase).
- `fn from_session_with(session: Arc<dyn GuestDatabase>, options: DatabaseBindingOptions) -> Self` - Wraps a host-granted session with request-budget knobs.
- `fn take_named_from_bindings(bindings: & mut Bindings, name: &str) -> Option<Self>` - Takes the named plugin database binding `name` from the `open`
- `fn prepare<impl Into<String>>(self: &Self, sql: impl Trait) -> PreparedStatement` - Prepare one canonical-SQL statement (`?` placeholders).
- `fn batch(self: &Self, statements: Vec<PreparedStatement>, retry: Option<RetryToken>) -> Result<Vec<D1Result>>` - Run prepared statements as one typed atomic batch.
- `fn exec<impl Into<String>>(self: &Self, query: impl Trait, retry: Option<RetryToken>) -> Result<D1ExecResult>` - Execute raw SQL without bind parameters (Cloudflare `D1Database::exec`).
- `fn execute(self: &Self, batch: Vec<TypedDbStatement>, retry: Option<RetryToken>) -> Result<ExecuteReply>` - Internal typed-batch transport. Prefer `Self::prepare`.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DatabaseBinding`



## bookclerk_plugin_sdk::db_binding::DatabaseBindingOptions

*Struct*

Options for [`DatabaseBinding`](db_binding.md#bookclerk_plugin_sdkdb_bindingdatabasebinding).

**Fields:**
- `max_request_bytes: u32` - Negotiated `maxRequestBytes` (`0` = unlimited).
- `max_result_rows: u32` - Default `maxRows` for `PreparedStatement::all` (`0` = host adapter cap).
- `retry: Option<RetryToken>` - Default retry token.
- `deadline_unix_ms: u64` - Guest-visible deadline (unix ms).

**Trait Implementations:**

- **Default**
  - `fn default() -> DatabaseBindingOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> DatabaseBindingOptions`



## bookclerk_plugin_sdk::db_binding::PreparedStatement

*Struct*

Cloudflare-style prepared statement.

**Methods:**

- `fn bind(self: Self, values: Vec<DbValue>) -> Self` - Replace bound parameters.
- `fn as_run(self: Self) -> Self` - Mark DML intent for `DatabaseBinding::batch`.
- `fn as_first(self: Self) -> Self` - Mark `maxRows = 1` row intent for `DatabaseBinding::batch`.
- `fn as_all(self: Self) -> Self` - Mark row-returning intent for `DatabaseBinding::batch`.
- `fn run(self: Self, retry: Option<RetryToken>) -> Result<D1Result>` - Execute as Cloudflare `run()` (functionally equivalent to `Self::all`).
- `fn first(self: Self, retry: Option<RetryToken>) -> Result<Option<Vec<(String, DbValue)>>>` - First row as a name→value map, or `None` (Cloudflare `first()` without `colName`).
- `fn first_column(self: Self, col_name: &str, retry: Option<RetryToken>) -> Result<Option<DbValue>>` - One column from the first row, or `None` when no row exists.
- `fn raw(self: Self, retry: Option<RetryToken>) -> Result<Vec<Vec<DbValue>>>` - Positional cell values per row (Cloudflare `raw()`).
- `fn all(self: Self, retry: Option<RetryToken>) -> Result<D1Result>` - Execute as a row-returning query. Returns a Cloudflare-shaped [`D1Result`](db_binding.md#bookclerk_plugin_sdkdb_bindingd1result).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PreparedStatement`



## bookclerk_plugin_sdk::db_binding::RetryToken

*Struct*

Explicit retry identity: reuse both `operationId` and `requestHash`.

**Fields:**
- `operation_id: String` - Caller-chosen idempotency key.
- `request_hash: String` - Canonical Cap'n request hash stamped by the host.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &RetryToken) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> RetryToken`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_sdk::db_binding::execute_reply_to_d1_results

*Function*

Maps a typed execute reply to Cloudflare [`D1Result`](db_binding.md#bookclerk_plugin_sdkdb_bindingd1result) per statement.

```rust
fn execute_reply_to_d1_results(reply: &bookclerk_plugin_abi::ExecuteReply) -> Vec<D1Result>
```



## bookclerk_plugin_sdk::db_binding::statement_result_to_d1_result

*Function*

Maps one typed statement result to Cloudflare [`D1Result`](db_binding.md#bookclerk_plugin_sdkdb_bindingd1result).

```rust
fn statement_result_to_d1_result(stmt: &bookclerk_plugin_abi::StatementResult, timing: &bookclerk_plugin_abi::DbTiming) -> D1Result
```



