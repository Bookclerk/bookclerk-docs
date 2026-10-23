# bookclerk\_plugin\_sdk.db\_value

Universal Cap'n database value domain (``DbValue``) and unpacked codec.

Baseline cells are typed null, bool, int64 (``int``), finite float64, UTF-8
text, and bytes (``bytes``). U+0000 is forbidden in TEXT and allowed in
BYTES. Unknown ``kind`` values fail closed. JSON
:func:`parse_db_value` still accepts a ``b64:`` string for bytes; the codec
always uses the domain types.

## NullValue

```python
class NullValue(TypedDict)
```

Typed SQL NULL with a declared ``DbType``.

## BoolValue

```python
class BoolValue(TypedDict)
```

Boolean cell.

## Int64Value

```python
class Int64Value(TypedDict)
```

Signed 64-bit integer cell (arbitrary-precision ``int`` in range).

## Float64Value

```python
class Float64Value(TypedDict)
```

Finite IEEE-754 float64 cell.

## TextValue

```python
class TextValue(TypedDict)
```

UTF-8 text cell (U+0000 forbidden; allowed only in ``bytes``).

## BytesValue

```python
class BytesValue(TypedDict)
```

Binary cell as raw ``bytes`` (not a ``b64:`` string).

## TypedDbStatement

```python
class TypedDbStatement(TypedDict)
```

One statement in a typed atomic batch.

## ExecuteRequest

```python
class ExecuteRequest(TypedDict)
```

Typed ``execute`` request (adapter wire: statements + transport metadata only).

## StatementResult

```python
class StatementResult(TypedDict)
```

Result of one statement in an atomic batch.

## DbTiming

```python
class DbTiming(TypedDict)
```

Handler/engine timing on ``ExecuteReply``.

## ExecuteReply

```python
class ExecuteReply(TypedDict)
```

Typed ``execute`` reply.

#### parse\_db\_value

```python
def parse_db_value(raw: Any) -> DbValue
```

Parse a JSON ``DbValue``. Unknown union members raise ``ValueError``.

``bytes`` accepts raw ``bytes`` or a ``b64:`` JSON string. ``int64`` is a
Python ``int`` (not a float).

#### encode\_db\_value

```python
def encode_db_value(value: DbValue) -> bytes
```

Encode a standalone unpacked Cap'n ``DbValue`` message.

**Arguments**:

- `value` - Domain value (``int`` / ``bytes``, not a JSON ``b64:`` string).
  

**Returns**:

  Unpacked Cap'n stream bytes (same encoding as the Rust SDK).

#### decode\_db\_value

```python
def decode_db_value(data: bytes) -> DbValue
```

Decode a standalone unpacked Cap'n ``DbValue`` message.

**Arguments**:

- `data` - Unpacked Cap'n stream.
  

**Returns**:

  Domain value.
  

**Raises**:

- `ValueError` - If the buffer is not a valid ``DbValue``.

#### encode\_execute\_request

```python
def encode_execute_request(request: ExecuteRequest) -> bytes
```

Encode a standalone unpacked Cap'n ``ExecuteRequest`` message.

**Arguments**:

- `request` - Structured request (non-empty ``statements``).
  

**Returns**:

  Unpacked Cap'n stream bytes (same encoding as the Rust SDK).
  

**Raises**:

- `ValueError` - If ``statements`` is empty.

#### decode\_execute\_request

```python
def decode_execute_request(data: bytes) -> ExecuteRequest
```

Decode a standalone unpacked Cap'n ``ExecuteRequest`` message.

**Arguments**:

- `data` - Unpacked Cap'n stream.
  

**Returns**:

  Structured request.
  

**Raises**:

- `ValueError` - If the buffer is not a valid non-empty ``ExecuteRequest``.

#### encode\_execute\_result\_reply

```python
def encode_execute_result_reply(
        outcome: ExecuteReply | tuple[str, str]) -> bytes
```

Encode ``ExecuteResultReply`` (``ok`` reply or ``(code, message)`` err).

#### decode\_execute\_result\_reply

```python
def decode_execute_result_reply(data: bytes) -> ExecuteReply
```

Decode ``ExecuteResultReply``. ``err`` is raised as ``PluginError``.

## D1Meta

```python
class D1Meta(TypedDict)
```

Cloudflare D1Result.meta projection.

## D1Result

```python
class D1Result(TypedDict)
```

Cloudflare D1Result projection for plugin guests.

## D1ExecResult

```python
class D1ExecResult(TypedDict)
```

Cloudflare D1ExecResult projection.

#### statement\_result\_to\_d1\_result

```python
def statement_result_to_d1_result(stmt: StatementResult,
                                  timing: DbTiming) -> D1Result
```

Map one statement result to Cloudflare :class:`D1Result`.

#### execute\_reply\_to\_d1\_results

```python
def execute_reply_to_d1_results(reply: ExecuteReply) -> list[D1Result]
```

Map an execute reply to one Cloudflare :class:`D1Result` per statement.

## DatabaseBinding

```python
class DatabaseBinding()
```

Host-mediated Cloudflare-style SQL binding for plugin guests.

Public surface is ``prepare().bind().run()/first()/all()`` and
``batch()``. Raw ``execute`` is an internal transport used by those
methods. Each call without an explicit :class:`RetryToken` mints a fresh
operation id and leaves ``requestHash`` empty so the trusted host can stamp
the canonical digest after validation. A :class:`RetryToken` reuses both.

#### DatabaseBinding.\_\_init\_\_

```python
def __init__(execute: AtomicTransport,
             *,
             max_request_bytes: int = 0,
             max_result_rows: int = 0,
             operation_id: str | None = None,
             request_hash: str = "",
             deadline_unix_ms: int = 0) -> None
```

Create a binding over a host ``execute`` transport.

**Arguments**:

- `execute` - Host session projection.
- `max_request_bytes` - Negotiated cap (``0`` = unlimited).
- `max_result_rows` - Default ``maxRows`` for :meth:`PreparedStatement.all`.
- `operation_id` - Default retry id; omitted calls mint a UUID.
- `request_hash` - Default retry hash; empty lets the host stamp one.
- `deadline_unix_ms` - Guest-visible deadline (unix ms).

#### DatabaseBinding.prepare

```python
def prepare(sql: str) -> PreparedStatement
```

Prepare one canonical-SQL statement (``?`` placeholders).

**Arguments**:

- `sql` - Host-mediated SQL. Kind and bounds are derived by the host.
  

**Returns**:

  A statement that can be bound and executed.

#### DatabaseBinding.batch

```python
async def batch(statements: list[PreparedStatement],
                *,
                retry: RetryToken | None = None) -> list[D1Result]
```

Run ``statements`` as one typed atomic batch.

Returns one Cloudflare-shaped :class:`D1Result` per statement.

#### DatabaseBinding.exec

```python
async def exec(query: str, *, retry: RetryToken | None = None) -> D1ExecResult
```

Execute raw SQL without bind parameters (Cloudflare ``D1Database.exec``).

#### DatabaseBinding.execute

```python
async def execute(batch: list[TypedDbStatement],
                  *,
                  retry: RetryToken | None = None) -> ExecuteReply
```

Internal typed-batch transport. Prefer :meth:`prepare` / :meth:`batch`.

#### create\_database\_binding

```python
def create_database_binding(transport: AtomicTransport,
                            *,
                            max_request_bytes: int = 0,
                            max_result_rows: int = 0,
                            operation_id: str | None = None,
                            request_hash: str = "",
                            deadline_unix_ms: int = 0) -> DatabaseBinding
```

Build a host-mediated :class:`DatabaseBinding` over an async transport.

## RetryToken

```python
class RetryToken()
```

Explicit retry identity: reuse both ``operationId`` and ``requestHash``.

## PreparedStatement

```python
class PreparedStatement()
```

Cloudflare-style prepared statement over a :class:`DatabaseBinding`.

#### PreparedStatement.bind

```python
def bind(*values: DbValue) -> PreparedStatement
```

Replace bound parameters with ``values`` (universal ``DbValue`` only).

#### PreparedStatement.as\_run

```python
def as_run() -> PreparedStatement
```

Mark DML intent for :meth:`DatabaseBinding.batch`.

#### PreparedStatement.as\_first

```python
def as_first() -> PreparedStatement
```

Mark ``maxRows = 1`` row intent for :meth:`DatabaseBinding.batch`.

#### PreparedStatement.as\_all

```python
def as_all() -> PreparedStatement
```

Mark row-returning intent for :meth:`DatabaseBinding.batch`.

#### PreparedStatement.run

```python
async def run(*, retry: RetryToken | None = None) -> D1Result
```

Execute as Cloudflare ``run()`` (functionally equivalent to :meth:`all`).

#### PreparedStatement.first

```python
async def first(
        col_name: str | None = None,
        *,
        retry: RetryToken | None = None) -> dict[str, Any] | Any | None
```

Return the first row, or one column when ``col_name`` is set.

#### PreparedStatement.raw

```python
async def raw(*, retry: RetryToken | None = None) -> list[list[Any]]
```

Return positional cell values per row (Cloudflare ``raw()``).

#### PreparedStatement.all

```python
async def all(*, retry: RetryToken | None = None) -> D1Result
```

Execute as a row-returning query. Returns a Cloudflare-shaped :class:`D1Result`.

#### canonical\_execute\_request\_hash

```python
def canonical_execute_request_hash(request: ExecuteRequest) -> str
```

SHA-256 hex of the Cap'n request with transport metadata cleared.

``operationId``, ``requestHash``, and ``deadlineUnixMs`` are omitted so a
retry can refresh the remaining deadline. Matches the host digest.

