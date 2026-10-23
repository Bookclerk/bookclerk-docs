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
| [`parse_db_value`](#parse_db_value) | function | Parse a JSON `DbValue`. Unknown union members raise `ValueError`. |
| [`encode_db_value`](#encode_db_value) | function | Encode a standalone unpacked Cap'n `DbValue` message. |
| [`decode_db_value`](#decode_db_value) | function | Decode a standalone unpacked Cap'n `DbValue` message. |
| [`encode_execute_request`](#encode_execute_request) | function | Encode a standalone unpacked Cap'n `ExecuteRequest` message. |
| [`decode_execute_request`](#decode_execute_request) | function | Decode a standalone unpacked Cap'n `ExecuteRequest` message. |
| [`encode_execute_result_reply`](#encode_execute_result_reply) | function | Encode `ExecuteResultReply` (`ok` reply or `(code, message)` err). |
| [`decode_execute_result_reply`](#decode_execute_result_reply) | function | Decode `ExecuteResultReply`. `err` is raised as `PluginError`. |
| [`statement_result_to_d1_result`](#statement_result_to_d1_result) | function | Map one statement result to Cloudflare `D1Result`. |
| [`execute_reply_to_d1_results`](#execute_reply_to_d1_results) | function | Map an execute reply to one Cloudflare `D1Result` per statement. |
| [`create_database_binding`](#create_database_binding) | function | Build a host-mediated `DatabaseBinding` over an async transport. |
| [`canonical_execute_request_hash`](#canonical_execute_request_hash) | function | SHA-256 hex of the Cap'n request with transport metadata cleared. |
| [`KINDS`](#kinds) | constant |  |
| [`TYPES`](#types) | constant |  |
| [`I64_MIN`](#i64_min) | constant |  |
| [`I64_MAX`](#i64_max) | constant |  |
| [`DbValue`](#dbvalue) | constant |  |
| [`DbStatementKind`](#dbstatementkind) | constant |  |
| [`DbResultSelection`](#dbresultselection) | constant |  |
| [`AtomicTransport`](#atomictransport) | constant |  |

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
def encode_execute_result_reply(outcome: ExecuteReply | tuple[str, str]) -> bytes
```

Encode `ExecuteResultReply` (`ok` reply or `(code, message)` err).

### decode_execute_result_reply

```python
def decode_execute_result_reply(data: bytes) -> ExecuteReply
```

Decode `ExecuteResultReply`. `err` is raised as `PluginError`.

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
    max_request_bytes: int=0,
    max_result_rows: int=0,
    operation_id: str | None=None,
    request_hash: str='',
    deadline_unix_ms: int=0,
) -> DatabaseBinding
```

Build a host-mediated [`DatabaseBinding`](#databasebinding) over an async transport.

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
DbValue = Union[NullValue, BoolValue, Int64Value, Float64Value, TextValue, BytesValue]
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
