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
| [`RetryToken`](#retrytoken) | class | Explicit retry identity: reuse both `operationId` and `requestHash`. |
| [`PreparedStatement`](#preparedstatement) | class | Cloudflare-style prepared statement over a `DatabaseBinding`. |
| [`ExecuteStreamLimits`](#executestreamlimits) | class | What a receiver lets a streamed reply grow to. |
| [`ExecuteStreamDecoder`](#executestreamdecoder) | class | Sans-IO decoder of a streamed execute reply. |
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
| [`streams_results`](#streams_results) | function | Whether `caps` advertises a streamed result budget. |
| [`execute_stream_limits`](#execute_stream_limits) | function | The streamed budget of `caps`, or its inline caps when it does not stream. |
| [`raw_stream_ceiling`](#raw_stream_ceiling) | function | Raw frame bytes a receiver accepts before failing closed. |
| [`decode_execute_stream`](#decode_execute_stream) | function | Decode a whole frame sequence under `limits`. |
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
