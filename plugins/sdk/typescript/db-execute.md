# db-execute

## Classes

### WriteChunksError

A [DatabaseBinding.writeChunks](#writechunks) run that failed part way. `code`,
`message` and `details` are the failed batch's; `next` is the statements
committed before it, to resume from.

#### Extends

- [`PluginError`](README.md#pluginerror)

#### Constructors

##### Constructor

```ts
new WriteChunksError(next, cause): WriteChunksError;
```

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `next` | `number` | Statements committed before the failure. |
| `cause` | [`PluginError`](README.md#pluginerror) | The failed batch's error, whose code, message and details this error carries. |

###### Returns

[`WriteChunksError`](#writechunkserror)

###### Overrides

[`PluginError`](README.md#pluginerror).[`constructor`](README.md#constructor-1)

#### Properties

##### code

```ts
readonly code: string;
```

Known `PluginErrorCode` wire string, or `unknown`.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`code`](README.md#code)

##### details

```ts
readonly details: Readonly<Record<string, unknown>> | undefined;
```

Structured detail object from the wire (`details` JSON), when the error carries one.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`details`](README.md#details)

##### next

```ts
readonly next: number;
```

Statements committed before the failure; resume with `from: next`.

##### wireCode

```ts
readonly wireCode: string;
```

Raw wire code, including codes this SDK does not know.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`wireCode`](README.md#wirecode)

#### Accessors

##### detailsText

###### Get Signature

```ts
get detailsText(): string;
```

`details` as the JSON text the wire carries.

###### Returns

`string`

JSON object text, or an empty string when the error has none.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`detailsText`](README.md#detailstext)

##### limit

###### Get Signature

```ts
get limit(): LimitExceeded | undefined;
```

The exceeded cap when this is a `limit_exceeded` error.

###### Returns

[`LimitExceeded`](README.md#limitexceeded-1) \| `undefined`

The cap from `details` first, then from the canonical message;
`undefined` for every other code.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`limit`](README.md#limit)

#### Methods

##### fromWire()

```ts
static fromWire(
   code, 
   message, 
   details?
): PluginError;
```

Construct a [PluginError](README.md#pluginerror) from a Cap'n Proto / JSON wire code.

Unknown codes become `unknown` on [PluginError.code](README.md#code) while
[PluginError.wireCode](README.md#wirecode) keeps the raw value.

###### Parameters

| Parameter | Type | Default value | Description |
| ------ | ------ | ------ | ------ |
| `code` | `string` | `undefined` | Wire error code (known or unknown). |
| `message` | `string` | `undefined` | Operator-facing error text. |
| `details?` | `string` | `""` | `details` JSON text from the wire (empty when none). |

###### Returns

[`PluginError`](README.md#pluginerror)

Typed plugin error.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`fromWire`](README.md#fromwire)

##### limitExceeded()

```ts
static limitExceeded(limit): PluginError;
```

Builds a `limit_exceeded` error whose message and `details` both carry
the exceeded cap.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `limit` | [`LimitExceeded`](README.md#limitexceeded-1) | Exceeded cap. |

###### Returns

[`PluginError`](README.md#pluginerror)

Typed plugin error.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`limitExceeded`](README.md#limitexceeded)

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

##### executeStreamed()?

```ts
executeStreamed?(request): Promise<
  | Uint8Array<ArrayBufferLike>
| ReadableStream<Uint8Array<ArrayBufferLike>>>;
```

Runs the same typed atomic batch as [AtomicTransport.execute](#execute) but
answered as `ExecuteStreamFrame` bytes (`GuestDatabase.executeStreamed`),
so the result may grow to the session's streamed budget. Transports
that predate it leave it out; the binding then reads inline.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`ExecuteRequest`](#executerequest) | Structured Cap'n `ExecuteRequest`. |

###### Returns

`Promise`\<
  \| `Uint8Array`\<`ArrayBufferLike`\>
  \| `ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\>\>

The frame sequence, whole or as a byte stream.

##### limits()?

```ts
limits?(): Promise<DbCapabilities>;
```

Negotiated limits of the session (`DbCapabilities`). Transports that
predate `GuestDatabase.limits` leave it out; the binding then throws
`unsupported`.

###### Returns

`Promise`\<[`DbCapabilities`](generated.md#dbcapabilities)\>

The adapter's capability advertisement for this session.

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

##### chunk()

```ts
chunk(statements, options?): AsyncIterable<PreparedStatement[]>;
```

Splits `statements` into batches that each fit one request
(`maxStatements` of the session and the tighter of the session's and
the binding's `maxRequestBytes`, or the caps in `options`):
`for await (const b of db.chunk(stmts)) await db.batch(b)`. Each batch
is atomic; the loop is not. A single statement larger than the request
cap is yielded alone, so `batch()` reports the typed error. Batches are
sized with the UUID-shaped retry id the binding mints; a caller-chosen
retry id that is longer leaves that much less room.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `statements` | [`PreparedStatement`](#preparedstatement)[] | Prepared statements (binds and intent already applied). |
| `options?` | \{ `maxRequestBytes?`: `number`; `maxStatements?`: `number`; \} | Caps to split by instead of the session's limits; `0` given explicitly means no cap (one batch when both are `0`). |
| `options.maxRequestBytes?` | `number` | - |
| `options.maxStatements?` | `number` | - |

###### Returns

`AsyncIterable`\<[`PreparedStatement`](#preparedstatement)[]\>

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

##### limits()

```ts
limits(): Promise<DbCapabilities>;
```

The negotiated limits of this binding's session (`DbCapabilities`):
`maxRequestBytes`, `maxStatements`, `maxResultBytes`, `maxResultRows`
and the rest. Use them to size a batch or a read before sending.

###### Returns

`Promise`\<[`DbCapabilities`](generated.md#dbcapabilities)\>

###### Throws

`PluginError` with code `unsupported` on a host that predates
`GuestDatabase.limits`.

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

##### writeChunks()

```ts
writeChunks(statements, options): Promise<WriteProgress>;
```

Writes `statements` in batches that fit one request, resumably.

The batches are the ones [DatabaseBinding.chunk](#chunk) yields for
`statements.slice(from)`, each committed with `batch()` under an
operation id derived from `key` and the batch's offset
([writeChunksOperationId](#writechunksoperationid)). Store the returned `next` (for example
in a job checkpoint) and call again with the same list, the same `key`
and `from: next` to carry on: a batch whose reply was lost is resent
under the same id and the host replays its receipt. Receipts are kept
for 24 hours; a batch resent later runs again, so a write that may pause
that long uses statements safe to repeat.

`stop(next)` is asked before each batch; returning `true` pauses the
run there. The list and the caps must not change between calls with
one `key`, or a resent batch no longer matches its receipt and the host
answers `conflict`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `statements` | [`PreparedStatement`](#preparedstatement)[] | The whole list, the same on every call. |
| `options` | [`WriteChunksOptions`](#writechunksoptions) | `key` names the write (stable across resumes, at most [MAX\_WRITE\_KEY\_BYTES](#max_write_key_bytes) UTF-8 bytes); `from` is the statements already committed (default `0`); `stop` is asked before each batch; `maxStatements` / `maxRequestBytes` are as for `chunk()`. |

###### Returns

`Promise`\<[`WriteProgress`](#writeprogress)\>

`{ next, done }`: statements committed, and whether all are.

###### Throws

[WriteChunksError](#writechunkserror) carrying `next` when a batch fails or
  the input is invalid.

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

##### iterate()

```ts
iterate(options?): AsyncIterable<Record<string, DbValue>>;
```

Rows one at a time from the same single-snapshot streamed read as
[PreparedStatement.all](#all), without holding the whole result:
`for await (const row of stmt.iterate()) …`. The read still fails closed
(`limit_exceeded`) past the session's streamed budget. On a session that
does not stream the rows come from one inline `all()`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | \{ `retry?`: [`RetryToken`](#retrytoken); \} | Optional retry token. |
| `options.retry?` | [`RetryToken`](#retrytoken) | - |

###### Returns

`AsyncIterable`\<`Record`\<`string`, [`DbValue`](db-value.md#dbvalue)\>\>

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

***

### WriteChunksOptions

Options for [DatabaseBinding.writeChunks](#writechunks).

#### Properties

##### from?

```ts
from?: number;
```

Statements already committed; `0` on the first call.

##### key

```ts
key: string;
```

Names this write, stable across resumes (a job id plus a step name).

##### maxRequestBytes?

```ts
maxRequestBytes?: number;
```

Request bytes per batch instead of the session's cap (`0` = no cap).

##### maxStatements?

```ts
maxStatements?: number;
```

Statements per batch instead of the session's cap (`0` = no cap).

##### stop?

```ts
stop?: (next) => boolean;
```

Asked before each batch with the statements committed so far; `true` pauses.

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `next` | `number` |

###### Returns

`boolean`

***

### WriteProgress

Where a [DatabaseBinding.writeChunks](#writechunks) run stands.

#### Properties

##### done

```ts
done: boolean;
```

True when every statement is committed; false when `stop` paused the run.

##### next

```ts
next: number;
```

Statements committed from the start of the list; always a batch boundary.

## Variables

### MAX\_WRITE\_KEY\_BYTES

```ts
const MAX_WRITE_KEY_BYTES: 256 = 256;
```

Longest `key` [DatabaseBinding.writeChunks](#writechunks) accepts, in UTF-8 bytes.

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

### chunkIndexes()

```ts
function chunkIndexes(
   typed, 
   maxStatements, 
   maxRequestBytes
): number[][];
```

Greedy split of `typed` into index runs whose encoded request stays within
`maxRequestBytes` and whose length stays within `maxStatements` (`0` lifts
a cap). Sizes are estimated per statement and each run is verified by one
exact encode; a run the estimate let through is halved until it fits, and
a lone statement over the cap is yielded on its own.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `typed` | [`TypedDbStatement`](#typeddbstatement)[] | Statements to split. |
| `maxStatements` | `number` | Statements per request (`0` = no cap). |
| `maxRequestBytes` | `number` | Encoded request bytes (`0` = no cap). |

#### Returns

`number`[][]

Index runs in order, covering every statement once.

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

### decodeDbCapabilitiesReply()

```ts
function decodeDbCapabilitiesReply(bytes): DbCapabilities;
```

Decodes a standalone `DbCapabilitiesReply`. `err` is thrown as `PluginError`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bytes` | `Uint8Array` | Unpacked Cap'n stream. |

#### Returns

[`DbCapabilities`](generated.md#dbcapabilities)

The session's `DbCapabilities`.

#### Throws

When the union is `err` or the buffer is invalid.

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

### encodeDbCapabilitiesReply()

```ts
function encodeDbCapabilitiesReply(outcome): Uint8Array;
```

Encodes a standalone `DbCapabilitiesReply` (`ok` or `err`), the answer of
`GuestDatabase.limits` on the granted channel (`GET /db/limits`).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `outcome` | \| \{ `ok`: [`DbCapabilities`](generated.md#dbcapabilities); \} \| \{ `err`: \{ `code`: `string`; `details?`: `string`; `message`: `string`; \}; \} | Capability advertisement or a wire `PluginError`. |

#### Returns

`Uint8Array`

Unpacked Cap'n stream bytes.

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
| `outcome` | \| \{ `ok`: [`ExecuteReply`](#executereply); \} \| \{ `err`: \{ `code`: `string`; `details?`: `string`; `message`: `string`; \}; \} | Successful reply or a wire `PluginError`. |

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

***

### writeChunksOperationId()

```ts
function writeChunksOperationId(key, offset): Promise<string>;
```

The operation id [DatabaseBinding.writeChunks](#writechunks) sends for the batch
that starts at `offset` of the write named `key`: the first 16 bytes of
SHA-256 over `"bookclerk-write-chunks\0" + key + "\0" + offset`
(decimal), as lowercase hex grouped 8-4-4-4-12. The Rust and Python SDKs
derive the same id.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Name of the write. |
| `offset` | `number` | Index of the batch's first statement. |

#### Returns

`Promise`\<`string`\>

A UUID-shaped operation id.

## References

### DbResultSelection

Re-exports [DbResultSelection](abi.md#dbresultselection)

***

### DbStatementKind

Re-exports [DbStatementKind](abi.md#dbstatementkind)
