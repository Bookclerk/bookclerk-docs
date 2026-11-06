# db-execute

## Interfaces

### AtomicTransport

Host `GuestDatabase.execute` projection.

#### Extended by

- [`GrantedDatabaseTransport`](plugin.md#granteddatabasetransport)

#### Methods

##### execute()

```ts
execute(request): Promise<ExecuteReply>;
```

Runs one typed atomic batch.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`ExecuteRequest`](#executerequest) | Structured Cap'n `ExecuteRequest`. |

###### Returns

`Promise`\<[`ExecuteReply`](#executereply)\>

Structured `ExecuteReply`.

***

### D1ExecResult

Cloudflare [D1ExecResult](https://developers.cloudflare.com/d1/worker-api/return-object/).

#### Properties

##### count

```ts
count: number;
```

Number of statements executed (one per non-empty `exec` line).

##### duration

```ts
duration: number;
```

Total duration in milliseconds.

***

### D1Meta

Cloudflare [D1Result.meta](https://developers.cloudflare.com/d1/worker-api/return-object/)
shape (Bookclerk fills fields available from the typed execute reply).

#### Properties

##### changed\_db

```ts
changed_db: boolean;
```

`true` when `changes > 0`.

##### changes

```ts
changes: number;
```

Rows changed by the statement (`rowsAffected`).

##### duration

```ts
duration: number;
```

Engine-reported SQL duration in milliseconds.

##### last\_row\_id

```ts
last_row_id: number;
```

Last inserted row id when the adapter exposes it (else `0`).

##### rows\_read

```ts
rows_read: number;
```

Rows returned to the guest for this statement.

##### rows\_written

```ts
rows_written: number;
```

Rows written (`changes` for DML).

***

### D1Result

Cloudflare [D1Result](https://developers.cloudflare.com/d1/worker-api/return-object/)
projection for plugin guests. Errors throw; successful calls always set `success: true`.

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| <a id="type-parameter-t"></a> `T` | `Record`\<`string`, [`DbValue`](db-value.md#dbvalue)\> |

#### Properties

##### meta

```ts
meta: D1Meta;
```

##### results

```ts
results: T[] | null;
```

Row objects for selects; `[]` when empty; `null` when not applicable (DML).

##### success

```ts
success: true;
```

***

### DatabaseBinding

Host-mediated typed SQL surface for plugin guests.

Public API is Cloudflare-style `prepare().bind().run()/first()/all()` plus
atomic `batch()`. Raw `execute` stays internal.

#### Methods

##### batch()

```ts
batch(statements, options?): Promise<D1Result<Record<string, DbValue>>[]>;
```

Run prepared statements as one typed atomic batch.

Ordinary bound prepared statements use Cloudflare `run()` semantics
(`resultSelection: rows`) by default. Optional `asRun` / `asFirst` /
`asAll` override per-statement intent. Returns one Cloudflare-shaped
[D1Result](#d1result) per statement.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `statements` | [`PreparedStatement`](#preparedstatement)[] | Prepared statements (binds and intent already applied). |
| `options?` | \{ `retry?`: [`RetryToken`](#retrytoken); \} | Retry token. |
| `options.retry?` | [`RetryToken`](#retrytoken) | - |

###### Returns

`Promise`\<[`D1Result`](#d1result)\<`Record`\<`string`, [`DbValue`](db-value.md#dbvalue)\>\>[]\>

##### exec()

```ts
exec(query, options?): Promise<D1ExecResult>;
```

Execute raw SQL without bind parameters (Cloudflare `D1Database.exec`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `query` | `string` | Canonical SQL string. |
| `options?` | \{ `retry?`: [`RetryToken`](#retrytoken); \} | Optional retry token. |
| `options.retry?` | [`RetryToken`](#retrytoken) | - |

###### Returns

`Promise`\<[`D1ExecResult`](#d1execresult)\>

##### execute()

```ts
execute(batch, options?): Promise<ExecuteReply>;
```

Internal typed-batch transport. Prefer [DatabaseBinding.prepare](#prepare).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `batch` | [`TypedDbStatement`](#typeddbstatement)[] | Ordered statements with typed `DbValue` parameters. |
| `options?` | \{ `retry?`: [`RetryToken`](#retrytoken); \} | Optional retry token. |
| `options.retry?` | [`RetryToken`](#retrytoken) | - |

###### Returns

`Promise`\<[`ExecuteReply`](#executereply)\>

##### prepare()

```ts
prepare(sql): PreparedStatement;
```

Prepare one canonical-SQL statement (`?` placeholders).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `sql` | `string` | Host-mediated SQL. Kind and bounds are derived by the host. |

###### Returns

[`PreparedStatement`](#preparedstatement)

***

### DatabaseBindingOptions

Options for [createDatabaseBinding](#createdatabasebinding).

#### Properties

##### deadlineUnixMs?

```ts
deadlineUnixMs?: number;
```

Guest-visible deadline (unix ms).

##### maxRequestBytes?

```ts
maxRequestBytes?: number;
```

Negotiated `maxRequestBytes` (`0` = unlimited).

##### maxResultRows?

```ts
maxResultRows?: number;
```

Default `maxRows` for `all()` (`0` = host adapter cap).

##### operationId?

```ts
operationId?: string;
```

Idempotency key; generated when omitted.

##### requestHash?

```ts
requestHash?: string;
```

SHA-256 hex of the request; empty when omitted.

##### retry?

```ts
retry?: RetryToken;
```

Default retry token; omitted calls mint a UUID and leave the hash empty.

***

### DbColumn

One column in a typed result set.

#### Properties

##### dbType

```ts
dbType: "unspecified" | "bool" | "text" | "int64" | "float64" | "bytes";
```

Declared / inferred type.

##### name

```ts
name: string;
```

Column name.

***

### DbRow

One positional result row.

#### Properties

##### values

```ts
values: DbValue[];
```

Cells in column order.

***

### DbTiming

Handler/engine timing on `ExecuteReply`.

#### Properties

##### attemptElapsedUs

```ts
attemptElapsedUs: number;
```

Whole-attempt elapsed microseconds.

##### dbExecutionUs

```ts
dbExecutionUs: number;
```

Engine-reported SQL microseconds when available.

##### dbTimingSource

```ts
dbTimingSource: string;
```

Observability source (`sqlite_txn`, `d1_sql_duration`, …).

***

### ExecuteReply

Typed `execute` reply.

#### Properties

##### operationId

```ts
operationId: string;
```

Echo of the request `operationId`.

##### statements

```ts
statements: StatementResult[];
```

Per-statement results, in plan order.

##### timing

```ts
timing: DbTiming;
```

Handler/engine timing.

***

### ExecuteRequest

Typed `execute` request.

#### Properties

##### deadlineUnixMs

```ts
deadlineUnixMs: number;
```

Guest-visible deadline (unix ms). Zero means omitted.

##### operationId

```ts
operationId: string;
```

Caller-chosen idempotency key.

##### requestHash

```ts
requestHash: string;
```

SHA-256 hex of the idempotency-relevant request; empty when omitted.

##### statements

```ts
statements: TypedDbStatement[];
```

Ordered statements (must be non-empty).

***

### PreparedStatement

Cloudflare-style prepared statement. Kind, bounds, and request hash are
derived by the trusted host. Only universal [DbValue](db-value.md#dbvalue)s are public.

#### Methods

##### all()

```ts
all(options?): Promise<D1Result<Record<string, DbValue>>>;
```

Execute as a row-returning query. Returns a Cloudflare-shaped [D1Result](#d1result).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | \{ `retry?`: [`RetryToken`](#retrytoken); \} | Optional retry token. |
| `options.retry?` | [`RetryToken`](#retrytoken) | - |

###### Returns

`Promise`\<[`D1Result`](#d1result)\<`Record`\<`string`, [`DbValue`](db-value.md#dbvalue)\>\>\>

##### asAll()

```ts
asAll(): PreparedStatement;
```

Mark row-returning intent for [DatabaseBinding.batch](#batch).

###### Returns

[`PreparedStatement`](#preparedstatement)

##### asFirst()

```ts
asFirst(): PreparedStatement;
```

Mark `maxRows = 1` row intent for [DatabaseBinding.batch](#batch).

###### Returns

[`PreparedStatement`](#preparedstatement)

##### asRun()

```ts
asRun(): PreparedStatement;
```

Mark DML intent for [DatabaseBinding.batch](#batch).

###### Returns

[`PreparedStatement`](#preparedstatement)

##### bind()

```ts
bind(...values): PreparedStatement;
```

Replace bound parameters.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| ...`values` | [`DbValue`](db-value.md#dbvalue)[] | Universal `DbValue`s only. |

###### Returns

[`PreparedStatement`](#preparedstatement)

##### first()

```ts
first(colName?, options?): Promise<
  | DbValue
  | Record<string, DbValue>
| null>;
```

First row as a name→value map, or one column when `colName` is set.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `colName?` | `string` | Optional column name (Cloudflare `first(colName)`). |
| `options?` | \{ `retry?`: [`RetryToken`](#retrytoken); \} | Optional retry token. |
| `options.retry?` | [`RetryToken`](#retrytoken) | - |

###### Returns

`Promise`\<
  \| [`DbValue`](db-value.md#dbvalue)
  \| `Record`\<`string`, [`DbValue`](db-value.md#dbvalue)\>
  \| `null`\>

##### raw()

```ts
raw(options?): Promise<DbValue[][]>;
```

Positional cell values per row (Cloudflare `raw()`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | \{ `retry?`: [`RetryToken`](#retrytoken); \} | Optional retry token. |
| `options.retry?` | [`RetryToken`](#retrytoken) | - |

###### Returns

`Promise`\<[`DbValue`](db-value.md#dbvalue)[][]\>

##### run()

```ts
run(options?): Promise<D1Result<Record<string, DbValue>>>;
```

Execute as DML. Returns a Cloudflare-shaped [D1Result](#d1result).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | \{ `retry?`: [`RetryToken`](#retrytoken); \} | Optional retry token. |
| `options.retry?` | [`RetryToken`](#retrytoken) | - |

###### Returns

`Promise`\<[`D1Result`](#d1result)\<`Record`\<`string`, [`DbValue`](db-value.md#dbvalue)\>\>\>

***

### RetryToken

Explicit retry identity. Reuses both `operationId` and `requestHash`.

#### Properties

##### operationId

```ts
operationId: string;
```

Caller-chosen idempotency key.

##### requestHash

```ts
requestHash: string;
```

Canonical Cap'n request hash stamped by the host.

***

### StatementResult

Result of one statement in an atomic batch.

#### Properties

##### columns

```ts
columns: DbColumn[];
```

Column metadata aligned with each row.

##### rows

```ts
rows: DbRow[];
```

Positional rows.

##### rowsAffected

```ts
rowsAffected: number;
```

Engine `rowsAffected` (0 for `Select`).

***

### TypedDbStatement

One statement in a typed atomic batch.

#### Properties

##### kind

```ts
kind: "execute" | "select" | "returning";
```

Host-authored kind (adapters must not reparse SQL).

##### maxRows

```ts
maxRows: number;
```

Proven row upper bound (`0` = unproven).

##### parameters

```ts
parameters: DbValue[];
```

Ordered typed binds.

##### resultSelection

```ts
resultSelection: "discard" | "affectedRows" | "rows";
```

Which result fields the caller needs.

##### sql

```ts
sql: string;
```

Canonical Bookclerk SQL (`?` placeholders).

## Functions

### canonicalExecuteRequestHash()

```ts
function canonicalExecuteRequestHash(request): Promise<string>;
```

SHA-256 hex of the Cap'n request with id and hash cleared.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`ExecuteRequest`](#executerequest) | Structured request. |

#### Returns

`Promise`\<`string`\>

Hex digest matching the host canonical hash.

***

### createDatabaseBinding()

```ts
function createDatabaseBinding(transport, options?): DatabaseBinding;
```

Builds a host-mediated [DatabaseBinding](#databasebinding) over an `execute` transport.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `transport` | [`AtomicTransport`](#atomictransport) | Host session projection. |
| `options?` | [`DatabaseBindingOptions`](#databasebindingoptions) | Request-budget and idempotency knobs. |

#### Returns

[`DatabaseBinding`](#databasebinding)

Binding whose `execute(batch)` encodes Cap'n then calls the host.

***

### decodeExecuteRequest()

```ts
function decodeExecuteRequest(bytes): ExecuteRequest;
```

Decodes a standalone unpacked Cap'n `ExecuteRequest` message.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bytes` | `Uint8Array` | Unpacked Cap'n stream. |

#### Returns

[`ExecuteRequest`](#executerequest)

Structured request.

#### Throws

When the buffer is not a valid non-empty `ExecuteRequest`.

***

### decodeExecuteResultReply()

```ts
function decodeExecuteResultReply(bytes): ExecuteReply;
```

Decodes a standalone `ExecuteResultReply`. `err` is thrown as `PluginError`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bytes` | `Uint8Array` | Unpacked Cap'n stream. |

#### Returns

[`ExecuteReply`](#executereply)

Structured `ExecuteReply`.

#### Throws

When the union is `err` or the buffer is invalid.

***

### encodeExecuteReply()

```ts
function encodeExecuteReply(reply): Uint8Array;
```

Encodes a standalone unpacked Cap'n `ExecuteReply` message.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `reply` | [`ExecuteReply`](#executereply) | Structured reply. |

#### Returns

`Uint8Array`

Unpacked Cap'n stream bytes.

***

### encodeExecuteRequest()

```ts
function encodeExecuteRequest(request): Uint8Array;
```

Encodes a standalone unpacked Cap'n `ExecuteRequest` message.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`ExecuteRequest`](#executerequest) | Structured request (non-empty `statements`). |

#### Returns

`Uint8Array`

Unpacked Cap'n stream bytes (same encoding as the Rust SDK).

#### Throws

When `statements` is empty.

***

### encodeExecuteResultReply()

```ts
function encodeExecuteResultReply(outcome): Uint8Array;
```

Encodes `ExecuteResultReply` (`ok` or `err`).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `outcome` | \| \{ `ok`: [`ExecuteReply`](#executereply); \} \| \{ `err`: \{ `code`: `string`; `message`: `string`; \}; \} | Successful reply or a wire `PluginError`. |

#### Returns

`Uint8Array`

Unpacked Cap'n stream bytes.

***

### executeReplyToD1Results()

```ts
function executeReplyToD1Results(reply): D1Result<Record<string, DbValue>>[];
```

Maps a typed execute reply to Cloudflare [D1Result](#d1result) per statement.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `reply` | [`ExecuteReply`](#executereply) | Decoded `ExecuteReply` from the host transport. |

#### Returns

[`D1Result`](#d1result)\<`Record`\<`string`, [`DbValue`](db-value.md#dbvalue)\>\>[]

One `D1Result` per statement, in plan order.

***

### statementResultToD1Result()

```ts
function statementResultToD1Result(stmt, timing): D1Result;
```

Maps one typed statement result to Cloudflare [D1Result](#d1result).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `stmt` | [`StatementResult`](#statementresult) | Per-statement rows, columns, and `rowsAffected`. |
| `timing` | [`DbTiming`](#dbtiming) | Handler/engine timing from the execute reply. |

#### Returns

[`D1Result`](#d1result)

Cloudflare-shaped `{ success, results, meta }`.

## References

### DbResultSelection

Re-exports [DbResultSelection](abi.md#dbresultselection)

***

### DbStatementKind

Re-exports [DbStatementKind](abi.md#dbstatementkind)
