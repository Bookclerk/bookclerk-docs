# bookclerk_plugin_sdk.db_value

Universal Cap'n database value domain (`DbValue`) and unpacked codec.

Baseline cells are typed null, bool, int64 (`int`), finite float64, UTF-8
text, and bytes (`bytes`). U+0000 is forbidden in TEXT and allowed in
BYTES. Unknown `kind` values fail closed. JSON
[`parse_db_value`](#parse_db_value) still accepts a `b64:` string for bytes; the codec
always uses the domain types.

| Name | Kind | Summary |
| --- | --- | --- |
| [`NullValue`](#nullvalue) | class | Typed SQL NULL with a declared `DbType`. |
| [`BoolValue`](#boolvalue) | class | Boolean cell. |
| [`Int64Value`](#int64value) | class | Signed 64-bit integer cell (arbitrary-precision `int` in range). |
| [`Float64Value`](#float64value) | class | Finite IEEE-754 float64 cell. |
| [`TextValue`](#textvalue) | class | UTF-8 text cell (U+0000 forbidden; allowed only in `bytes`). |
| [`BytesValue`](#bytesvalue) | class | Binary cell as raw `bytes` (not a `b64:` string). |
| [`TypedDbStatement`](#typeddbstatement) | class | One statement in a typed atomic batch. |
| [`ExecuteRequest`](#executerequest) | class | Typed `execute` request (adapter wire: statements + transport metadata only). |
| [`StatementResult`](#statementresult) | class | Result of one statement in an atomic batch. |
| [`DbTiming`](#dbtiming) | class | Handler/engine timing on `ExecuteReply`. |
| [`ExecuteReply`](#executereply) | class | Typed `execute` reply. |
| [`D1Meta`](#d1meta) | class | Cloudflare D1Result.meta projection. |
| [`D1Result`](#d1result) | class | Cloudflare D1Result projection for plugin guests. |
| [`D1ExecResult`](#d1execresult) | class | Cloudflare D1ExecResult projection. |
| [`DatabaseBinding`](#databasebinding) | class | Host-mediated Cloudflare-style SQL binding for plugin guests. |
| [`WriteProgress`](#writeprogress) | class | Where a `DatabaseBinding.write_chunks` run stands. |
| [`RetryToken`](#retrytoken) | class | Explicit retry identity: reuse both `operationId` and `requestHash`. |
| [`PreparedStatement`](#preparedstatement) | class | Cloudflare-style prepared statement over a `DatabaseBinding`. |
| [`ExecuteStreamLimits`](#executestreamlimits) | class | What a receiver lets a streamed reply grow to. |
| [`ExecuteStreamDecoder`](#executestreamdecoder) | class | Sans-IO decoder of a streamed execute reply. |
| [`AdapterBatchRequest`](#adapterbatchrequest) | class | The parts of an `AdapterExecuteRequest` the batch runner reads. |
| [`AdapterTransaction`](#adaptertransaction) | class | One open transaction on the adapter's engine. |
| [`parse_db_value`](#parse_db_value) | function | Parse a JSON `DbValue`. Unknown union members raise `ValueError`. |
| [`encode_db_value`](#encode_db_value) | function | Encode a standalone unpacked Cap'n `DbValue` message. |
| [`decode_db_value`](#decode_db_value) | function | Decode a standalone unpacked Cap'n `DbValue` message. |
| [`encode_execute_request`](#encode_execute_request) | function | Encode a standalone unpacked Cap'n `ExecuteRequest` message. |
| [`decode_execute_request`](#decode_execute_request) | function | Decode a standalone unpacked Cap'n `ExecuteRequest` message. |
| [`encode_execute_result_reply`](#encode_execute_result_reply) | function | Encode `ExecuteResultReply`. |
| [`decode_execute_result_reply`](#decode_execute_result_reply) | function | Decode `ExecuteResultReply`. `err` is raised as `PluginError`. |
| [`encode_db_capabilities_reply`](#encode_db_capabilities_reply) | function | Encode `DbCapabilitiesReply`, the `GuestDatabase.limits` answer. |
| [`decode_db_capabilities_reply`](#decode_db_capabilities_reply) | function | Decode `DbCapabilitiesReply`. `err` is raised as `PluginError`. |
| [`statement_result_to_d1_result`](#statement_result_to_d1_result) | function | Map one statement result to Cloudflare `D1Result`. |
| [`execute_reply_to_d1_results`](#execute_reply_to_d1_results) | function | Map an execute reply to one Cloudflare `D1Result` per statement. |
| [`create_database_binding`](#create_database_binding) | function | Build a host-mediated `DatabaseBinding` over an async transport. |
| [`write_chunks_operation_id`](#write_chunks_operation_id) | function | The operation id `DatabaseBinding.write_chunks` sends for a batch. |
| [`streams_results`](#streams_results) | function | Whether `caps` advertises a streamed result budget. |
| [`execute_stream_limits`](#execute_stream_limits) | function | The streamed budget of `caps`, or its inline caps when it does not stream. |
| [`raw_stream_ceiling`](#raw_stream_ceiling) | function | Raw frame bytes a receiver accepts before failing closed. |
| [`decode_execute_stream`](#decode_execute_stream) | function | Decode a whole frame sequence under `limits`. |
| [`execute_adapter_batch`](#execute_adapter_batch) | function | Run one host batch in a transaction and commit it with its replay receipt. |
| [`execute_adapter_batch_streamed`](#execute_adapter_batch_streamed) | function | Run one host batch for `executeStreamed`; commit only when the host can take it. |
| [`run_adapter_batch`](#run_adapter_batch) | function | Run the wrapped statements and the receipt finalize without committing. |
| [`guest_batch_is_read_only`](#guest_batch_is_read_only) | function | True when every guest statement inside the receipt wrap is a `SELECT`. |
| [`guest_receipt_finalize_statements`](#guest_receipt_finalize_statements) | function | Statements that persist the guest reply on the claimed receipt. |
| [`should_skip_remaining_guest_work`](#should_skip_remaining_guest_work) | function | True when the prefix results show a completed prior receipt. |
| [`guest_replay_payload`](#guest_replay_payload) | function | The replay payload for a guest reply, in the JSON form the host decodes. |
| [`encode_execute_stream`](#encode_execute_stream) | function | Frame `reply` as the byte sequence `executeStreamed` answers. |
| [`chunk_indexes`](#chunk_indexes) | function | Greedy split of `typed` into index runs that each fit one request. |
| [`canonical_execute_request_hash`](#canonical_execute_request_hash) | function | SHA-256 hex of the Cap'n request with transport metadata cleared. |
| [`KINDS`](#kinds) | constant |  |
| [`TYPES`](#types) | constant |  |
| [`I64_MIN`](#i64_min) | constant |  |
| [`I64_MAX`](#i64_max) | constant |  |
| [`DbValue`](#dbvalue) | constant |  |
| [`DbStatementKind`](#dbstatementkind) | constant |  |
| [`DbResultSelection`](#dbresultselection) | constant |  |
| [`AtomicTransport`](#atomictransport) | constant |  |
| [`LimitsTransport`](#limitstransport) | constant |  |
| [`StreamedTransport`](#streamedtransport) | constant | `GuestDatabase.executeStreamed` projection: frame bytes, whole or chunked. |
| [`MAX_WRITE_KEY_BYTES`](#max_write_key_bytes) | constant | Longest `key` `DatabaseBinding.write_chunks` accepts, in UTF-8 bytes. |
| [`GUEST_RECEIPT_WRAP_PREFIX`](#guest_receipt_wrap_prefix) | constant | Prune + prior-receipt `SELECT` statements ahead of the guest statements. |
| [`GUEST_RECEIPT_STUB_SUFFIX`](#guest_receipt_stub_suffix) | constant | Receipt stub `INSERT` after the guest statements. |
| [`GUEST_REPLAY_PAYLOAD_MAX_BYTES`](#guest_replay_payload_max_bytes) | constant | Largest replay payload, in UTF-8 bytes, a receipt stores for a write batch. |
| [`GUEST_READ_ONLY_REPLAY_PAYLOAD`](#guest_read_only_replay_payload) | constant | Payload stored for a read-only guest batch instead of its rows (reads run again on replay). |
| [`GUEST_RECEIPT_RESULT_LOST`](#guest_receipt_result_lost) | constant | Error text when a receipt committed but the reply it should hold was lost. |
| [`GUEST_RECEIPT_CLAIM_LOST`](#guest_receipt_claim_lost) | constant | Error text when a concurrent attempt committed the receipt between this attempt's prior-receipt `SELECT` and its stub `INSERT`. |
| [`STREAM_FRAME_TARGET_BYTES`](#stream_frame_target_bytes) | constant | Encoded size an encoder aims for per `rows` frame (one wider row may exceed it). |

## Classes

### NullValue

```python
class NullValue(TypedDict)
```

Typed SQL NULL with a declared `DbType`.

**Fields**

- `kind: Literal['null']`
- `value: DbType`

### BoolValue

```python
class BoolValue(TypedDict)
```

Boolean cell.

**Fields**

- `kind: Literal['boolean']`
- `value: bool`

### Int64Value

```python
class Int64Value(TypedDict)
```

Signed 64-bit integer cell (arbitrary-precision `int` in range).

**Fields**

- `kind: Literal['int64']`
- `value: int`

### Float64Value

```python
class Float64Value(TypedDict)
```

Finite IEEE-754 float64 cell.

**Fields**

- `kind: Literal['float64']`
- `value: float`

### TextValue

```python
class TextValue(TypedDict)
```

UTF-8 text cell (U+0000 forbidden; allowed only in `bytes`).

**Fields**

- `kind: Literal['text']`
- `value: str`

### BytesValue

```python
class BytesValue(TypedDict)
```

Binary cell as raw `bytes` (not a `b64:` string).

**Fields**

- `kind: Literal['bytes']`
- `value: bytes`

### TypedDbStatement

```python
class TypedDbStatement(TypedDict)
```

One statement in a typed atomic batch.

**Fields**

- `sql: str`
- `parameters: list[DbValue]`
- `kind: DbStatementKind`
- `maxRows: int`
- `resultSelection: DbResultSelection`

### ExecuteRequest

```python
class ExecuteRequest(TypedDict)
```

Typed `execute` request (adapter wire: statements + transport metadata only).

**Fields**

- `operationId: str`
- `requestHash: str`
- `statements: list[TypedDbStatement]`
- `deadlineUnixMs: int`

### StatementResult

```python
class StatementResult(TypedDict)
```

Result of one statement in an atomic batch.

**Fields**

- `rows: list[dict[str, Any]]`
- `columns: list[dict[str, Any]]`
- `rowsAffected: int`

### DbTiming

```python
class DbTiming(TypedDict)
```

Handler/engine timing on `ExecuteReply`.

**Fields**

- `attemptElapsedUs: int`
- `dbExecutionUs: int`
- `dbTimingSource: str`

### ExecuteReply

```python
class ExecuteReply(TypedDict)
```

Typed `execute` reply.

**Fields**

- `operationId: str`
- `statements: list[StatementResult]`
- `timing: DbTiming`

### D1Meta

```python
class D1Meta(TypedDict)
```

Cloudflare D1Result.meta projection.

**Fields**

- `duration: float`
- `changes: int`
- `last_row_id: int`
- `changed_db: bool`
- `rows_read: int`
- `rows_written: int`

### D1Result

```python
class D1Result(TypedDict)
```

Cloudflare D1Result projection for plugin guests.

**Fields**

- `success: bool`
- `results: list[dict[str, Any]] | None`
- `meta: D1Meta`

### D1ExecResult

```python
class D1ExecResult(TypedDict)
```

Cloudflare D1ExecResult projection.

**Fields**

- `count: int`
- `duration: float`

### DatabaseBinding

```python
class DatabaseBinding
```

Host-mediated Cloudflare-style SQL binding for plugin guests.

Public surface is `prepare().bind().run()/first()/all()` and
`batch()`. Raw `execute` is an internal transport used by those
methods. Each call without an explicit [`RetryToken`](#retrytoken) mints a fresh
operation id and leaves `requestHash` empty so the trusted host can stamp
the canonical digest after validation. A [`RetryToken`](#retrytoken) reuses both.

#### DatabaseBinding.__init__

```python
def __init__(
    self,
    execute: AtomicTransport,
    *,
    limits: LimitsTransport | None=None,
    execute_streamed: StreamedTransport | None=None,
    max_request_bytes: int=0,
    max_result_rows: int=0,
    operation_id: str | None=None,
    request_hash: str='',
    deadline_unix_ms: int=0,
) -> None
```

Create a binding over a host `execute` transport.

**Args**

- `execute`: Host session projection.
- `limits`: `GuestDatabase.limits` projection; `None` on a host that predates it ([`limits`](limits.md) then raises `unsupported`).
- `execute_streamed`: `GuestDatabase.executeStreamed` projection the same batch answered as `ExecuteStreamFrame` bytes. When set and the session advertises a streamed budget, every row-returning read goes through it; `None` reads inline.
- `max_request_bytes`: Negotiated cap (`0` = unlimited).
- `max_result_rows`: Default `maxRows` for [`PreparedStatement.all`](#preparedstatementall).
- `operation_id`: Default retry id; omitted calls mint a UUID.
- `request_hash`: Default retry hash; empty lets the host stamp one.
- `deadline_unix_ms`: Guest-visible deadline (unix ms).

#### DatabaseBinding.prepare

```python
def prepare(self, sql: str) -> PreparedStatement
```

Prepare one canonical-SQL statement (`?` placeholders).

**Args**

- `sql`: Host-mediated SQL. Kind and bounds are derived by the host.

**Returns**

A statement that can be bound and executed.

#### DatabaseBinding.limits

```python
async def limits(self) -> DbCapabilities
```

The negotiated limits of this binding's session (`DbCapabilities`).

`maxRequestBytes`, `maxStatements`, `maxResultBytes`,
`maxResultRows` and the rest: use them to size a batch or a read
before sending.

**Returns**

The adapter's capability advertisement for this session.

**Raises**

- `PluginError`: `unsupported` on a host that predates `GuestDatabase.limits`.

#### DatabaseBinding.chunk

```python
def chunk(
    self,
    statements: list[PreparedStatement],
    *,
    max_statements: int | None=None,
    max_request_bytes: int | None=None,
) -> AsyncIterator[list[PreparedStatement]]
```

Split `statements` into batches that each fit one request.

`async for b in db.chunk(stmts): await db.batch(b)`. Each batch is
atomic; the loop is not. The caps come from the session's limits
(`maxStatements`, and the tighter of the session's and the
binding's `maxRequestBytes`) unless given here; zero caps given
explicitly mean one batch. A single statement larger than the
request cap is yielded alone, so [`batch`](#databasebindingbatch) reports the typed
error. Batches are sized with the UUID-shaped retry id the binding
mints; a caller-chosen retry id that is longer leaves that much less
room.

**Args**

- `statements`: Prepared statements (binds and intent already applied).
- `max_statements`: Statements per batch (`0` = no cap); the session's `maxStatements` when `None`.
- `max_request_bytes`: Encoded request bytes per batch (`0` = no cap); the tighter of the session's and the binding's cap when `None`.

**Returns**

Batches in order, together covering every statement once.

**Raises**

- `PluginError`: `unsupported` when neither the session's limits nor explicit caps are available.

#### DatabaseBinding.write_chunks

```python
async def write_chunks(
    self,
    statements: list[PreparedStatement],
    *,
    key: str,
    start: int=0,
    stop: Callable[[int], bool] | None=None,
    max_statements: int | None=None,
    max_request_bytes: int | None=None,
) -> WriteProgress
```

Write `statements` in batches that fit one request, resumably.

The batches are the ones [`chunk`](#databasebindingchunk) yields for
`statements[start:]`, each committed with [`batch`](#databasebindingbatch) under an
operation id derived from `key` and the batch's offset
([`write_chunks_operation_id`](#write_chunks_operation_id)). Store the returned `next` (for
example in a job checkpoint) and call again with the same list, the
same `key` and `start=next` to carry on: a batch whose reply was
lost is resent under the same id and the host replays its receipt.
Receipts are kept for 24 hours; a batch resent later runs again, so a
write that may pause that long uses statements safe to repeat.
The list and the caps must not change between calls with one `key`,
or a resent batch no longer matches its receipt and the host answers
`conflict`.

**Args**

- `statements`: The whole list, the same on every call.
- `key`: Names this write, stable across resumes (a job id plus a step name); at most [`MAX_WRITE_KEY_BYTES`](#max_write_key_bytes) UTF-8 bytes.
- `start`: Statements already committed; `0` on the first call.
- `stop`: Asked before each batch with the statements committed so far; returning `True` pauses the run there.
- `max_statements`: As for [`chunk`](#databasebindingchunk).
- `max_request_bytes`: As for [`chunk`](#databasebindingchunk).

**Returns**

`{"next": n, "done": bool}`: statements committed, and whether
all of them are.

**Raises**

- `WriteChunksError`: Carrying `next` when the input is invalid or a batch fails.

#### DatabaseBinding.batch

```python
async def batch(
    self,
    statements: list[PreparedStatement],
    *,
    retry: RetryToken | None=None,
) -> list[D1Result]
```

Run `statements` as one typed atomic batch.

Returns one Cloudflare-shaped [`D1Result`](#d1result) per statement.

#### DatabaseBinding.exec

```python
async def exec(self, query: str, *, retry: RetryToken | None=None) -> D1ExecResult
```

Execute raw SQL without bind parameters (Cloudflare `D1Database.exec`).

#### DatabaseBinding.execute

```python
async def execute(
    self,
    batch: list[TypedDbStatement],
    *,
    retry: RetryToken | None=None,
) -> ExecuteReply
```

Internal typed-batch transport. Prefer [`prepare`](#databasebindingprepare) / [`batch`](#databasebindingbatch).

### WriteProgress

```python
class WriteProgress(TypedDict)
```

Where a [`DatabaseBinding.write_chunks`](#databasebindingwrite_chunks) run stands.

**Fields**

- `next: int`
- `done: bool`

### RetryToken

```python
class RetryToken
```

Explicit retry identity: reuse both `operationId` and `requestHash`.

#### RetryToken.__init__

```python
def __init__(self, operation_id: str, request_hash: str) -> None
```

### PreparedStatement

```python
class PreparedStatement
```

Cloudflare-style prepared statement over a [`DatabaseBinding`](#databasebinding).

#### PreparedStatement.__init__

```python
def __init__(
    self,
    binding: DatabaseBinding,
    sql: str,
    parameters: list[DbValue],
    result_selection: DbResultSelection | None=None,
    max_rows: int | None=None,
    *,
    max_result_rows: int=0,
    intent: tuple[DbResultSelection, int] | None=None,
) -> None
```

#### PreparedStatement.bind

```python
def bind(self, *values: DbValue) -> PreparedStatement
```

Replace bound parameters with `values` (universal `DbValue` only).

#### PreparedStatement.as_run

```python
def as_run(self) -> PreparedStatement
```

Mark DML intent for [`DatabaseBinding.batch`](#databasebindingbatch).

#### PreparedStatement.as_first

```python
def as_first(self) -> PreparedStatement
```

Mark `maxRows = 1` row intent for [`DatabaseBinding.batch`](#databasebindingbatch).

#### PreparedStatement.as_all

```python
def as_all(self) -> PreparedStatement
```

Mark row-returning intent for [`DatabaseBinding.batch`](#databasebindingbatch).

#### PreparedStatement.run

```python
async def run(self, *, retry: RetryToken | None=None) -> D1Result
```

Execute as Cloudflare `run()` (functionally equivalent to [`all`](#preparedstatementall)).

#### PreparedStatement.first

```python
async def first(
    self,
    col_name: str | None=None,
    *,
    retry: RetryToken | None=None,
) -> dict[str, Any] | Any | None
```

Return the first row, or one column when `col_name` is set.

#### PreparedStatement.raw

```python
async def raw(self, *, retry: RetryToken | None=None) -> list[list[Any]]
```

Return positional cell values per row (Cloudflare `raw()`).

#### PreparedStatement.all

```python
async def all(self, *, retry: RetryToken | None=None) -> D1Result
```

Execute as a row-returning query. Returns a Cloudflare-shaped [`D1Result`](#d1result).

#### PreparedStatement.iterate

```python
def iterate(self, *, retry: RetryToken | None=None) -> AsyncIterator[dict[str, Any]]
```

Rows one at a time from the same single-snapshot read as [`all`](#preparedstatementall).

`async for row in stmt.iterate(): ...` decodes the streamed reply
frame by frame without holding the whole result. The read still fails
closed (`limit_exceeded`) past the session's streamed budget. On a
session that does not stream the rows come from one inline `all()`.

**Args**

- `retry`: Optional retry token.

**Returns**

Column name → typed cell, one mapping per row, in order.

### ExecuteStreamLimits

```python
@dataclass(frozen=True)
class ExecuteStreamLimits
```

What a receiver lets a streamed reply grow to.

**Attributes**

- `max_bytes`: Encoded reply budget in bytes (`0` = only the framing slack).
- `bytes_limit`: Capability field `max_bytes` came from, named by the error it raises (`maxStreamedResultBytes` or `maxAtomicResultBytes`).
- `max_rows`: Rows one statement may return (`0` = unlimited).
- `rows_limit`: Capability field `max_rows` came from (`maxStreamedResultRows` or `maxResultRows`).

### ExecuteStreamDecoder

```python
class ExecuteStreamDecoder
```

Sans-IO decoder of a streamed execute reply.

Feed it bytes in any split with [`push`](#executestreamdecoderpush), then take the reassembled
`ExecuteReply` with [`finish`](#executestreamdecoderfinish). It fails closed on more raw bytes
than the budget allows, on a statement with more rows than the row cap,
and on any frame out of sequence; fed in any split it yields the same
reply as one push.

#### ExecuteStreamDecoder.__init__

```python
def __init__(self, limits: ExecuteStreamLimits, *, retain_rows: bool=True) -> None
```

Create a decoder held to `limits`.

**Args**

- `limits`: Budget the stream is held to.
- `retain_rows`: `False` counts rows against the budget but drops them from the collected reply (for `iterate()`).

#### ExecuteStreamDecoder.finished

```python
@property
def finished(self) -> bool
```

Whether the final frame (`end` or `err`) has been decoded.

**Returns**

True once the stream is complete.

#### ExecuteStreamDecoder.columns

```python
@property
def columns(self) -> list[dict[str, Any]]
```

Columns of the statement whose rows are arriving.

**Returns**

The open statement's `DbColumn` list, empty between statements.

#### ExecuteStreamDecoder.push

```python
def push(self, data: bytes) -> list[dict[str, Any]]
```

Feed the next bytes of the stream.

**Args**

- `data`: Any slice of the frame sequence.

**Returns**

The `ExecuteStreamFrame` dicts completed by these bytes, in order.

**Raises**

- `PluginError`: `limit_exceeded` when the stream exceeds the byte or row budget; `invalid_params` when a frame is malformed or out of sequence or bytes follow the final frame.

#### ExecuteStreamDecoder.finish

```python
def finish(self) -> ExecuteReply
```

Take the reassembled reply once the stream has ended.

**Returns**

The reply the frames described.

**Raises**

- `PluginError`: The adapter's error when the stream ended with `err`; `unavailable` when it ended before its final frame.

### AdapterBatchRequest

```python
class AdapterBatchRequest(TypedDict)
```

The parts of an `AdapterExecuteRequest` the batch runner reads.

**Attributes**

- `operationId`: Host-chosen idempotency key, echoed on the reply and keyed on the receipt.
- `statements`: Wrapped statements in execution order (proofs are not read).
- `receipt`: `{"guestLen", "guestHash"}`; `guestLen` `0` means the batch carries no guest receipt.

### AdapterTransaction

```python
class AdapterTransaction(Protocol)
```

One open transaction on the adapter's engine.

The runner calls `run` once per statement in order, including the
finalize statements it adds (canonical SQL with `?` placeholders, lowered
by the adapter like any other statement), then `commit`, or
[`rollback`](#adaptertransactionrollback) when any step failed.

#### AdapterTransaction.run

```python
async def run(self, statement: TypedDbStatement) -> StatementResult
```

Run one statement inside the transaction.

**Args**

- `statement`: Canonical statement to lower and execute.

**Returns**

The statement's result (rows only when `resultSelection` is `rows`).

#### AdapterTransaction.commit

```python
async def commit(self) -> None
```

Commit the transaction.

#### AdapterTransaction.rollback

```python
async def rollback(self) -> None
```

Roll the transaction back.

## Functions

### parse_db_value

```python
def parse_db_value(raw: Any) -> DbValue
```

Parse a JSON `DbValue`. Unknown union members raise `ValueError`.

`bytes` accepts raw `bytes` or a `b64:` JSON string. `int64` is a
Python `int` (not a float).

### encode_db_value

```python
def encode_db_value(value: DbValue) -> bytes
```

Encode a standalone unpacked Cap'n `DbValue` message.

**Args**

- `value`: Domain value (`int` / `bytes`, not a JSON `b64:` string).

**Returns**

Unpacked Cap'n stream bytes (same encoding as the Rust SDK).

### decode_db_value

```python
def decode_db_value(data: bytes) -> DbValue
```

Decode a standalone unpacked Cap'n `DbValue` message.

**Args**

- `data`: Unpacked Cap'n stream.

**Returns**

Domain value.

**Raises**

- `ValueError`: If the buffer is not a valid `DbValue`.

### encode_execute_request

```python
def encode_execute_request(request: ExecuteRequest) -> bytes
```

Encode a standalone unpacked Cap'n `ExecuteRequest` message.

**Args**

- `request`: Structured request (non-empty `statements`).

**Returns**

Unpacked Cap'n stream bytes (same encoding as the Rust SDK).

**Raises**

- `ValueError`: If `statements` is empty.

### decode_execute_request

```python
def decode_execute_request(data: bytes) -> ExecuteRequest
```

Decode a standalone unpacked Cap'n `ExecuteRequest` message.

**Args**

- `data`: Unpacked Cap'n stream.

**Returns**

Structured request.

**Raises**

- `ValueError`: If the buffer is not a valid non-empty `ExecuteRequest`.

### encode_execute_result_reply

```python
def encode_execute_result_reply(
    outcome: ExecuteReply | tuple[str, str] | tuple[str, str, str],
) -> bytes
```

Encode `ExecuteResultReply`.

**Args**

- `outcome`: The `ok` reply, or an `err` as `(code, message)` or `(code, message, details_json)`.

**Returns**

Unpacked Cap'n stream bytes.

### decode_execute_result_reply

```python
def decode_execute_result_reply(data: bytes) -> ExecuteReply
```

Decode `ExecuteResultReply`. `err` is raised as `PluginError`.

### encode_db_capabilities_reply

```python
def encode_db_capabilities_reply(
    outcome: DbCapabilities | tuple[str, str] | tuple[str, str, str],
) -> bytes
```

Encode `DbCapabilitiesReply`, the `GuestDatabase.limits` answer.

**Args**

- `outcome`: The `ok` capabilities, or an `err` as `(code, message)` or `(code, message, details_json)`.

**Returns**

Unpacked Cap'n stream bytes (the `GET /db/limits` body).

### decode_db_capabilities_reply

```python
def decode_db_capabilities_reply(data: bytes) -> DbCapabilities
```

Decode `DbCapabilitiesReply`. `err` is raised as `PluginError`.

**Args**

- `data`: Unpacked Cap'n stream.

**Returns**

The session's `DbCapabilities`.

**Raises**

- `PluginError`: When the union is `err`.
- `ValueError`: When the buffer is not a `DbCapabilitiesReply`.

### statement_result_to_d1_result

```python
def statement_result_to_d1_result(stmt: StatementResult, timing: DbTiming) -> D1Result
```

Map one statement result to Cloudflare [`D1Result`](#d1result).

### execute_reply_to_d1_results

```python
def execute_reply_to_d1_results(reply: ExecuteReply) -> list[D1Result]
```

Map an execute reply to one Cloudflare [`D1Result`](#d1result) per statement.

### create_database_binding

```python
def create_database_binding(
    transport: AtomicTransport,
    *,
    limits: LimitsTransport | None=None,
    execute_streamed: StreamedTransport | None=None,
    max_request_bytes: int=0,
    max_result_rows: int=0,
    operation_id: str | None=None,
    request_hash: str='',
    deadline_unix_ms: int=0,
) -> DatabaseBinding
```

Build a host-mediated [`DatabaseBinding`](#databasebinding) over an async transport.

**Args**

- `transport`: Host `execute` projection.
- `limits`: Host `limits` projection, when the host offers one.
- `execute_streamed`: Host `executeStreamed` projection, when the host offers one; reads then stream on sessions that advertise a streamed budget.
- `max_request_bytes`: Negotiated cap (`0` = unlimited).
- `max_result_rows`: Default `maxRows` for `all()`.
- `operation_id`: Default retry id; omitted calls mint a UUID.
- `request_hash`: Default retry hash; empty lets the host stamp one.
- `deadline_unix_ms`: Guest-visible deadline (unix ms).

**Returns**

A binding whose terminal methods route through `transport`.

### write_chunks_operation_id

```python
def write_chunks_operation_id(key: str, offset: int) -> str
```

The operation id [`DatabaseBinding.write_chunks`](#databasebindingwrite_chunks) sends for a batch.

The first 16 bytes of SHA-256 over
`"bookclerk-write-chunks\0" + key + "\0" + offset` (decimal), as
lowercase hex grouped 8-4-4-4-12. The Rust and TypeScript SDKs derive
the same id.

**Args**

- `key`: Name of the write.
- `offset`: Index of the batch's first statement.

**Returns**

A UUID-shaped operation id.

### streams_results

```python
def streams_results(caps: DbCapabilities) -> bool
```

Whether `caps` advertises a streamed result budget.

**Args**

- `caps`: Session capabilities (`limits()`).

**Returns**

True when `executeStreamed` is worth calling on this session.

### execute_stream_limits

```python
def execute_stream_limits(caps: DbCapabilities) -> ExecuteStreamLimits
```

The streamed budget of `caps`, or its inline caps when it does not stream.

**Args**

- `caps`: Session capabilities (`limits()`).

**Returns**

Limits for an [`ExecuteStreamDecoder`](#executestreamdecoder).

### raw_stream_ceiling

```python
def raw_stream_ceiling(limits: ExecuteStreamLimits) -> int
```

Raw frame bytes a receiver accepts before failing closed.

The reply budget plus one eighth for frame headers and one window of slack.

**Args**

- `limits`: Budget.

**Returns**

Byte ceiling on the whole frame sequence and on any one frame.

### decode_execute_stream

```python
def decode_execute_stream(data: bytes, limits: ExecuteStreamLimits) -> ExecuteReply
```

Decode a whole frame sequence under `limits`.

**Args**

- `data`: Every frame, concatenated.
- `limits`: Budget the stream is held to.

**Returns**

The reassembled reply.

**Raises**

- `PluginError`: See [`ExecuteStreamDecoder.push`](#executestreamdecoderpush) and [`ExecuteStreamDecoder.finish`](#executestreamdecoderfinish).

### execute_adapter_batch

```python
async def execute_adapter_batch(
    request: AdapterBatchRequest,
    begin: Callable[[], Awaitable[AdapterTransaction]],
    timing_source: str,
) -> ExecuteReply
```

Run one host batch in a transaction and commit it with its replay receipt.

**Args**

- `request`: The adapter request (`execute`).
- `begin`: Opens a transaction on the adapter's engine.
- `timing_source`: Observability source on the reply timing (`sqlite_txn`, …).

**Returns**

The committed reply, wrap statements included (the host unwraps it).

**Raises**

- `PluginError`: `unavailable` when a prior receipt committed without its reply, `internal` when the wrap shape is malformed or the payload is over [`GUEST_REPLAY_PAYLOAD_MAX_BYTES`](#guest_replay_payload_max_bytes).
- `Exception`: Whatever a statement or `commit` raised, after the transaction was rolled back.

### execute_adapter_batch_streamed

```python
async def execute_adapter_batch_streamed(
    request: AdapterBatchRequest,
    begin: Callable[[], Awaitable[AdapterTransaction]],
    timing_source: str,
    limits: ExecuteStreamLimits,
) -> bytes
```

Run one host batch for `executeStreamed`; commit only when the host can take it.

An isolate adapter's frames cross the invoke reply as one scalar. The
frames are encoded inside the transaction: encoding stops with
`limit_exceeded` (`maxScalarBytes`) as soon as they pass
`MAX_SCALAR_BYTES`, the finished frames are decoded under `limits` the
way the host decodes them, and the reassembled reply's encoded size is
held to `limits.max_bytes` the way the host validates it (the decoder
alone allows framing slack past the budget). Any failure rolls the batch
back, so nothing commits that the host would refuse.

**Args**

- `request`: The adapter request (`executeStreamed`).
- `begin`: Opens a transaction on the adapter's engine.
- `timing_source`: Observability source on the reply timing.
- `limits`: The streamed budget this adapter advertises (`execute_stream_limits(caps)`).

**Returns**

Every frame of the committed reply, concatenated.

**Raises**

- `PluginError`: `limit_exceeded` when the frames do not fit one scalar or `limits`, or what [`execute_adapter_batch`](#execute_adapter_batch) raises.

### run_adapter_batch

```python
async def run_adapter_batch(
    request: AdapterBatchRequest,
    txn: AdapterTransaction,
    timing_source: str,
) -> ExecuteReply
```

Run the wrapped statements and the receipt finalize without committing.

After the prior-receipt `SELECT`, a completed receipt for this operation
skips the remaining guest work (its results are padded so the wrap shape
stays intact) and the host replays the stored reply.

**Args**

- `request`: The adapter request.
- `txn`: The open transaction (only [`AdapterTransaction.run`](#adaptertransactionrun) is used).
- `timing_source`: Observability source on the reply timing.

**Returns**

The reply to deliver once the transaction commits.

**Raises**

- `PluginError`: See [`execute_adapter_batch`](#execute_adapter_batch); `invalid_params` when a result cell is not a valid `DbValue`.

### guest_batch_is_read_only

```python
def guest_batch_is_read_only(statements: list[TypedDbStatement], guest_len: int) -> bool
```

True when every guest statement inside the receipt wrap is a `SELECT`.

An empty or out-of-range guest slice is not read-only, so a write batch
never skips its replay payload.

**Args**

- `statements`: The wrapped statements.
- `guest_len`: Guest statement count from the receipt hint.

**Returns**

Whether the batch only reads.

### guest_receipt_finalize_statements

```python
def guest_receipt_finalize_statements(
    partial: ExecuteReply,
    guest_len: int,
    guest_hash: str,
    read_only: bool,
) -> list[TypedDbStatement]
```

Statements that persist the guest reply on the claimed receipt.

Empty when the prior receipt already holds a payload, belongs to a
different request (the host reports the conflict), or is an in-flight
claim whose stub this attempt did not insert.

**Args**

- `partial`: The wrapped reply so far (prefix, guest and stub results).
- `guest_len`: Guest statement count from the receipt hint.
- `guest_hash`: Guest `requestHash` from the receipt hint.
- `read_only`: Whether the guest slice only reads ([`guest_batch_is_read_only`](#guest_batch_is_read_only)).

**Returns**

The finalize statements to run before `COMMIT`.

**Raises**

- `PluginError`: `unavailable` when a prior receipt committed without its reply, `internal` when the wrap shape is malformed or the payload is over [`GUEST_REPLAY_PAYLOAD_MAX_BYTES`](#guest_replay_payload_max_bytes).

### should_skip_remaining_guest_work

```python
def should_skip_remaining_guest_work(
    results: list[StatementResult],
    total: int,
    guest_hash: str,
) -> bool
```

True when the prefix results show a completed prior receipt.

An in-flight claim with the same hash and a read-only receipt with the
same hash do not skip; a receipt with a different hash does (the host
reports the conflict).

**Args**

- `results`: Results so far.
- `total`: Wrapped statement count.
- `guest_hash`: Guest `requestHash` from the receipt hint.

**Returns**

Whether to pad the remaining results instead of running them.

### guest_replay_payload

```python
def guest_replay_payload(reply: ExecuteReply) -> str
```

The replay payload for a guest reply, in the JSON form the host decodes.

**Args**

- `reply`: The guest slice of the reply.

**Returns**

`{"operation_id", "statements", "timing"}` with each value in the
host's serde form.

**Raises**

- `PluginError`: `internal` when a cell does not encode or the payload is over [`GUEST_REPLAY_PAYLOAD_MAX_BYTES`](#guest_replay_payload_max_bytes).

### encode_execute_stream

```python
def encode_execute_stream(
    reply: ExecuteReply,
    frame_target_bytes: int=STREAM_FRAME_TARGET_BYTES,
    max_bytes: int | None=None,
) -> bytes
```

Frame `reply` as the byte sequence `executeStreamed` answers.

`head`, each statement with its rows split near `frame_target_bytes`,
then `end`. A slice whose frame still exceeds one scalar is halved until
each frame fits or holds one row.

**Args**

- `reply`: The committed reply.
- `frame_target_bytes`: Estimated encoded size to aim for per `rows` frame.
- `max_bytes`: Stop as soon as the frames pass this many bytes.

**Returns**

Every frame, concatenated.

**Raises**

- `PluginError`: `limit_exceeded` (`maxScalarBytes`) when the frames pass `max_bytes`.

### chunk_indexes

```python
def chunk_indexes(
    typed: list[TypedDbStatement],
    max_statements: int,
    max_request_bytes: int,
) -> list[list[int]]
```

Greedy split of `typed` into index runs that each fit one request.

Sizes are estimated per statement against one probe statement and each
run is verified by one exact encode; a run the estimate let through is
halved until it fits, and a lone statement over the cap is yielded on
its own.

**Args**

- `typed`: Statements to split.
- `max_statements`: Statements per request (`0` = no cap).
- `max_request_bytes`: Encoded request bytes per request (`0` = no cap).

**Returns**

Index runs in order, covering every statement once.

### canonical_execute_request_hash

```python
def canonical_execute_request_hash(request: ExecuteRequest) -> str
```

SHA-256 hex of the Cap'n request with transport metadata cleared.

`operationId`, `requestHash`, and `deadlineUnixMs` are omitted so a
retry can refresh the remaining deadline. Matches the host digest.

## Constants

### KINDS

```python
KINDS = frozenset({'null', 'boolean', 'int64', 'float64', 'text', 'bytes'})
```

### TYPES

```python
TYPES = frozenset(DB_TYPES)
```

### I64_MIN

```python
I64_MIN = -2 ** 63
```

### I64_MAX

```python
I64_MAX = 2 ** 63 - 1
```

### DbValue

```python
DbValue = NullValue | BoolValue | Int64Value | Float64Value | TextValue | BytesValue
```

### DbStatementKind

```python
DbStatementKind = Literal['execute', 'select', 'returning']
```

### DbResultSelection

```python
DbResultSelection = Literal['discard', 'affectedRows', 'rows']
```

### AtomicTransport

```python
AtomicTransport = Callable[[ExecuteRequest], Awaitable[ExecuteReply]]
```

### LimitsTransport

```python
LimitsTransport = Callable[[], Awaitable['DbCapabilities']]
```

### StreamedTransport

```python
StreamedTransport = Callable[[ExecuteRequest], Awaitable['bytes | AsyncIterable[bytes]']]
```

`GuestDatabase.executeStreamed` projection: frame bytes, whole or chunked.

### MAX_WRITE_KEY_BYTES

```python
MAX_WRITE_KEY_BYTES = 256
```

Longest `key` [`DatabaseBinding.write_chunks`](#databasebindingwrite_chunks) accepts, in UTF-8 bytes.

### GUEST_RECEIPT_WRAP_PREFIX

```python
GUEST_RECEIPT_WRAP_PREFIX = 2
```

Prune + prior-receipt `SELECT` statements ahead of the guest statements.

### GUEST_RECEIPT_STUB_SUFFIX

```python
GUEST_RECEIPT_STUB_SUFFIX = 1
```

Receipt stub `INSERT` after the guest statements.

### GUEST_REPLAY_PAYLOAD_MAX_BYTES

```python
GUEST_REPLAY_PAYLOAD_MAX_BYTES = 65536
```

Largest replay payload, in UTF-8 bytes, a receipt stores for a write batch.

### GUEST_READ_ONLY_REPLAY_PAYLOAD

```python
GUEST_READ_ONLY_REPLAY_PAYLOAD = '{"readOnly":true}'
```

Payload stored for a read-only guest batch instead of its rows (reads run again on replay).

### GUEST_RECEIPT_RESULT_LOST

```python
GUEST_RECEIPT_RESULT_LOST = 'guest receipt committed without payload; original response lost'
```

Error text when a receipt committed but the reply it should hold was lost.

### GUEST_RECEIPT_CLAIM_LOST

```python
GUEST_RECEIPT_CLAIM_LOST = 'guest receipt claimed by a concurrent attempt; retry to replay it'
```

Error text when a concurrent attempt committed the receipt between this
attempt's prior-receipt `SELECT` and its stub `INSERT`.

### STREAM_FRAME_TARGET_BYTES

```python
STREAM_FRAME_TARGET_BYTES = 64 * 1024
```

Encoded size an encoder aims for per `rows` frame (one wider row may exceed it).
