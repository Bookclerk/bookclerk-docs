# TypeScript plugin SDK

This page shows how to write a Bookclerk plugin in TypeScript with
`@bookclerk/plugin-sdk` (`api_version = 3`). A TypeScript plugin runs in
a workerd isolate; `bookclerk-workerd` injects the SDK into the isolate
under its package name, so your module imports it like any dependency.

| Import | What it holds |
| --- | --- |
| `@bookclerk/plugin-sdk/workerd` | `BookclerkEntrypoint` and the named `*Entrypoint` bases (they extend `WorkerEntrypoint`) |
| `@bookclerk/plugin-sdk` | The ABI types, codecs and the same author classes |
| `@bookclerk/plugin-sdk/sparse-workerd` | Run a workerd plugin without a built host (`smoke`) |

## Install

Depend on the package from npm, a `file:` path or a workspace. Never
import a relative path to the embed; the module name is what the host
injects.

```json
{
  "dependencies": {
    "@bookclerk/plugin-sdk": "file:../../packages/plugin-sdk"
  }
}
```

## Author model

A plugin is a Workers-shaped module. The **default export** extends
`BookclerkEntrypoint` and implements the triggers `plugin.toml` declares;
every **named entrypoint** in `entrypoints = [...]` is an exported class
with the matching name (`storefront` → `Storefront`, `cli` → `Cli`, …)
extending its base:

| `plugin.toml` | Export | Base class |
| --- | --- | --- |
| `[[events.consumers]]` | `default` | `BookclerkEntrypoint.event(batch)` |
| `[triggers] jobs = [...]` | `default` | `BookclerkEntrypoint.job(job)` |
| `[triggers] crons = [...]` | `default` | `BookclerkEntrypoint.scheduled(controller)` |
| `[[databases]]` | `default` | `BookclerkEntrypoint.databaseMigrations(binding)` |
| `entrypoints = ["storefront"]` | `Storefront` | `StorefrontEntrypoint` |
| `entrypoints = ["storage"]` | `Storage` | `StorageEntrypoint` |
| `entrypoints = ["remoteLibrary"]` | `RemoteLibrary` | `RemoteLibraryEntrypoint` |
| `entrypoints = ["databaseAdapter"]` | `DatabaseAdapter` | `DatabaseAdapterEntrypoint` |
| `entrypoints = ["cli"]` | `Cli` | `CliEntrypoint` |
| `entrypoints = ["oidc"]` | `Oidc` | `OidcEntrypoint` |
| `entrypoints = ["metadata"]` | `Metadata` | `MetadataEntrypoint` |
| `entrypoints = ["covers"]` | `Covers` | `CoverEntrypoint` |

```ts
import {
  BookclerkEntrypoint,
  CliEntrypoint,
  cliArgs,
  jsonPayload,
  type CliInvokeParams,
  type EventBatch,
} from "@bookclerk/plugin-sdk/workerd";
import type { Env } from "./bookclerk-configuration.js";

export class Cli extends CliEntrypoint<Env> {
  async describe() {
    return { commands: [{ name: "ping", about: "Probe", args: [] }] };
  }
  async invoke(params: CliInvokeParams) {
    const { message = "hi" } = cliArgs(params);
    return { exitCode: 0, stdout: `pong: ${message}\n`, stderr: "", payload: jsonPayload({ pong: message }) };
  }
}

export default class MyPlugin extends BookclerkEntrypoint<Env> {
  async event(batch: EventBatch) {
    for (const msg of batch.messages) {
      if (msg.type !== "book_acquired") {
        msg.reject(`unexpected ${msg.type}`);
        continue;
      }
      await this.env.EVENTS.publish({ eventType: "echo_seen", payload: msg.body });
      msg.ack();
    }
  }
}
```

Identity (`apiVersion`, `id`, capabilities) comes from `plugin.toml`; an
optional `describe()` on the default export only refines `displayName`
and similar fields.

A database adapter runs each host batch through `executeAdapterBatch` or
`executeAdapterBatchStreamed`, which stores the guest's replay receipt
before `COMMIT` and, for a streamed reply, rolls back when the frames do
not fit one scalar or the adapter's advertised streamed budget
(`executeStreamLimits(caps)`) (see
[Isolate adapter batches](../../sql-contract/v1.md#isolate-adapter-batches-and-replay-receipts)).

## Triggers

- `event(batch)` mirrors Workers `queue(batch)`. Each `EventMessage`
  records exactly one outcome, `ack()`, `retry({ retryAt | delaySeconds,
  reason })`, `reject(reason)`, `deadLetter(reason)` or `suspend({
  checkpoint, wakeAt, wakeOnEventType, wakeOnFilter })`, and the first
  call wins. Untouched messages ack when the handler returns and retry
  when it throws.
- `job(job)` mirrors Workers `scheduled(controller)`. Return (optionally
  `{ message, bytesCopied }`) to complete, throw to reject
  (`unavailable` and `deadline_exceeded` errors are retryable,
  `cancelled` is cancelled), or call `job.suspend({ checkpoint, wakeAt
  })` or `job.retryLater({ retryAt, reason })`. `job.input`,
  `job.output`, `job.progress()` and `job.signal` expose the granted
  streams and host cancellation.

## The `env` bindings

The host grants bindings per invocation on `this.env`: `CONFIG` (`[vars]`
plus operator settings), `SECRETS` (`[secrets]`), `EVENTS`
(`[[events.producers]]`), `WORK_FS` (`[work_fs]`), and one
`DatabaseBinding` per `[[databases]]` entry under its `binding` name.
A binding's `limits()` returns the `DbCapabilities` its session negotiated
(`maxRequestBytes`, `maxStatements`, `maxResultBytes`, `maxResultRows`,
…); a request or result over one of them rejects with a `PluginError`
whose `code` is `limit_exceeded` and whose `limit` names the cap. On a
host that predates `limits()` the call rejects with `unsupported`.
`all()` reads through the session's streamed budget when it advertises
one, `prepare(sql).iterate()` yields rows one at a time from the same
read, `db.chunk(statements)` splits a long list into batches that each
fit one request, and `db.writeChunks(...)` commits those batches
resumably; see
[Working within database limits](../../database-limits.md).
Generate the `Env` type from `plugin.toml`:

```bash
npx bookclerk-plugin types .            # writes bookclerk-configuration.d.ts
npx bookclerk-plugin types . --out src/env.d.ts
```

## Testing database code

`createTestDatabase()` from `@bookclerk/plugin-sdk/testkit` gives a
Node test suite the same binding a host grants, served by the
`bookclerk-db-testkit` executable under one capability profile with the
plugin's `databaseMigrations` applied. `bookclerk-plugin test .` runs the
suite once per profile (`first-party`, `d1` and `minimum`), so code that
passes on the shipped adapters is also proven against D1's caps and the
smallest advertisement a host accepts:

```ts
import { createTestDatabase } from "@bookclerk/plugin-sdk/testkit";

const db = await createTestDatabase({ migrations: MIGRATIONS }); // profile from `bookclerk-plugin test`
try {
  await addNotes(db.binding, ["hello"]);
} finally {
  await db.close();
}
```

The `echo_workerd_ts` example (`examples/plugins-echo-workerd-ts/`) is a
worked plugin: a `notes` table it writes with `chunk()`, lists with
`iterate()` or key-paged reads, sizes with `limits()`, and tests under
every profile. See
[Testing against the limits](../../database-limits.md#testing-against-the-limits).

## Author tools

```bash
npx bookclerk-plugin check .
npx bookclerk-plugin types [dir] [--out <file>]
npx bookclerk-plugin fmt [--check] plugin.toml
npx bookclerk-plugin package --out dist/plugins .
npx bookclerk-plugin smoke .   # download the pinned workerd, then describe + health
```

`check` enforces the v3 author model: the main module imports the SDK,
does not reference the removed `BookclerkPlugin` base, and exports one
class per named entrypoint.

`smoke` does not need a built `bookclerk-workerd`. It downloads the
pinned Cloudflare `workerd` into `~/.cache/bookclerk/workerd` (override
with `BOOKCLERK_WORKERD_CACHE` or `BOOKCLERK_WORKERD_BIN`).

## Next steps

- [plugin.toml](../../manifest.md): declare the entrypoints and
  triggers the classes above implement.
- [Entrypoints, events and the ABI](../../abi.md): every method and
  event in detail.
- [Try the Echo example](../../README.md#try-the-echo-example): a
  complete TypeScript plugin under `examples/`.

## Reference

The sections below are generated from the SDK's source: one page per
module, and what the package root exports.

## Modules

`@bookclerk/plugin-sdk` — ABI types + workerd author runtime + tools.

Package root re-exports the workerd [BookclerkEntrypoint](plugin.md#bookclerkentrypoint) base, the
named `*Entrypoint` bases, and the camelCase ABI types from `generated.ts`.
Prefer the dedicated subpath imports when writing guests:

- Workerd: `import { BookclerkEntrypoint } from "@bookclerk/plugin-sdk/workerd"`
- Native:  Rust `serve` / `PluginWorker`
- Tools: `npx bookclerk-plugin check|fmt|types|package`
- Sparse workerd: `import { runSmoke } from "@bookclerk/plugin-sdk/sparse-workerd"`

See `docs/plugins/README.md` and `docs/contributing/code-documentation.md`.

- [abi](abi.md)
- [cli](cli.md)
- [db-execute](db-execute.md)
- [db-value](db-value.md)
- [env](env.md)
- [generated](generated.md)
- [plugin](plugin.md)
- [sparse-workerd](sparse-workerd.md)
- [testkit](testkit.md)
- [workerd](workerd.md)

## Classes

### ExecuteStreamDecoder

Sans-IO decoder of a streamed execute reply: feed it bytes in any split,
then take the reassembled [ExecuteReply](db-execute.md#executereply).

It fails closed on more raw bytes than the budget allows, on a statement
with more rows than the row cap, and on any frame out of sequence.

#### Constructors

##### Constructor

```ts
new ExecuteStreamDecoder(limits, options?): ExecuteStreamDecoder;
```

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `limits` | [`ExecuteStreamLimits`](#executestreamlimits) | Budget the stream is held to. |
| `options?` | \{ `retainRows?`: `boolean`; \} | `retainRows: false` counts rows against the budget but drops them from the collected reply (for `iterate()`). |
| `options.retainRows?` | `boolean` | - |

###### Returns

[`ExecuteStreamDecoder`](#executestreamdecoder)

#### Accessors

##### columns

###### Get Signature

```ts
get columns(): DbColumn[];
```

Columns of the statement whose rows are arriving.

###### Returns

[`DbColumn`](generated.md#dbcolumn)[]

The open statement's columns, or an empty list between statements.

##### finished

###### Get Signature

```ts
get finished(): boolean;
```

Whether the final frame (`end` or `err`) has been decoded.

###### Returns

`boolean`

True once the stream is complete.

#### Methods

##### finish()

```ts
finish(): ExecuteReply;
```

Takes the reassembled reply once the stream has ended.

###### Returns

[`ExecuteReply`](db-execute.md#executereply)

The reply the frames described.

###### Throws

The adapter's `PluginError` when the stream ended with `err`, and
  `unavailable` when it ended before its final frame.

##### push()

```ts
push(bytes): ExecuteStreamFrame[];
```

Feeds the next bytes of the stream.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bytes` | `Uint8Array` | Any slice of the frame sequence. |

###### Returns

[`ExecuteStreamFrame`](generated.md#executestreamframe)[]

The frames completed by these bytes, in order.

###### Throws

`PluginError` `limit_exceeded` when the stream exceeds the byte
  or row budget, `invalid_params` when a frame is malformed or out of
  sequence or bytes follow the final frame.

***

### PluginError

Thrown by the SDK when a wire union carries `err`. Unknown codes are kept.

#### Extends

- `Error`

#### Extended by

- [`WriteChunksError`](db-execute.md#writechunkserror)
- [`InvokeError`](workerd.md#invokeerror)

#### Constructors

##### Constructor

```ts
new PluginError(
   code, 
   message, 
   details?
): PluginError;
```

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `code` | `string` | Wire error code. |
| `message` | `string` | Operator-facing error text. |
| `details?` | `string` \| `Readonly`\<`Record`\<`string`, `unknown`\>\> | Detail object, or its JSON text as carried on the wire. |

###### Returns

[`PluginError`](#pluginerror)

###### Overrides

`Error.constructor`

#### Properties

##### code

```ts
readonly code: string;
```

Known `PluginErrorCode` wire string, or `unknown`.

##### details

```ts
readonly details: Readonly<Record<string, unknown>> | undefined;
```

Structured detail object from the wire (`details` JSON), when the error carries one.

##### wireCode

```ts
readonly wireCode: string;
```

Raw wire code, including codes this SDK does not know.

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

##### limit

###### Get Signature

```ts
get limit(): LimitExceeded | undefined;
```

The exceeded cap when this is a `limit_exceeded` error.

###### Returns

[`LimitExceeded`](#limitexceeded-1) \| `undefined`

The cap from `details` first, then from the canonical message;
`undefined` for every other code.

#### Methods

##### fromWire()

```ts
static fromWire(
   code, 
   message, 
   details?
): PluginError;
```

Construct a [PluginError](#pluginerror) from a Cap'n Proto / JSON wire code.

Unknown codes become `unknown` on [PluginError.code](#code) while
[PluginError.wireCode](#wirecode) keeps the raw value.

###### Parameters

| Parameter | Type | Default value | Description |
| ------ | ------ | ------ | ------ |
| `code` | `string` | `undefined` | Wire error code (known or unknown). |
| `message` | `string` | `undefined` | Operator-facing error text. |
| `details?` | `string` | `""` | `details` JSON text from the wire (empty when none). |

###### Returns

[`PluginError`](#pluginerror)

Typed plugin error.

##### limitExceeded()

```ts
static limitExceeded(limit): PluginError;
```

Builds a `limit_exceeded` error whose message and `details` both carry
the exceeded cap.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `limit` | [`LimitExceeded`](#limitexceeded-1) | Exceeded cap. |

###### Returns

[`PluginError`](#pluginerror)

Typed plugin error.

## Interfaces

### AdapterBatchOptions

Options shared by the batch runners.

#### Extended by

- [`AdapterBatchStreamedOptions`](#adapterbatchstreamedoptions)

#### Properties

##### timingSource

```ts
timingSource: string;
```

Observability source on the reply timing (for example `sqlite_txn`).

***

### AdapterBatchRequest

The parts of an `AdapterExecuteRequest` the batch runner reads.

#### Properties

##### operationId

```ts
operationId: string;
```

Host-chosen idempotency key, echoed on the reply and keyed on the receipt.

##### receipt

```ts
receipt: AdapterReceipt;
```

Receipt hint; `guestLen` `0` means the batch carries no guest receipt.

##### statements

```ts
statements: AdapterBatchStatement[];
```

Wrapped statements in execution order (proofs are not read).

***

### AdapterBatchStreamedOptions

Options of [executeAdapterBatchStreamed](#executeadapterbatchstreamed).

#### Extends

- [`AdapterBatchOptions`](#adapterbatchoptions)

#### Properties

##### limits

```ts
limits: ExecuteStreamLimits;
```

The streamed budget this adapter advertises
(`executeStreamLimits(caps)`); the host decodes the reply under it.

##### timingSource

```ts
timingSource: string;
```

Observability source on the reply timing (for example `sqlite_txn`).

###### Inherited from

[`AdapterBatchOptions`](#adapterbatchoptions).[`timingSource`](#timingsource)

***

### AdapterTransaction

One open transaction on the adapter's engine.

The runner calls `run` once per statement in order, including the finalize
statements it adds (canonical SQL with `?` placeholders, lowered by the
adapter like any other statement), then `commit`, or `rollback` when any
step failed.

#### Methods

##### commit()

```ts
commit(): Promise<void>;
```

Commits the transaction.

###### Returns

`Promise`\<`void`\>

Resolves once the batch is durable.

##### rollback()

```ts
rollback(): Promise<void>;
```

Rolls the transaction back.

###### Returns

`Promise`\<`void`\>

Resolves once nothing of the batch remains.

##### run()

```ts
run(statement): Promise<StatementResult>;
```

Runs one statement inside the transaction.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `statement` | [`AdapterBatchStatement`](#adapterbatchstatement) | Canonical statement to lower and execute. |

###### Returns

`Promise`\<[`StatementResult`](db-execute.md#statementresult)\>

The statement's result (rows only when `resultSelection` is `rows`).

***

### ExecuteStreamLimits

What a receiver lets a streamed reply grow to.

#### Properties

##### bytesLimit

```ts
bytesLimit: "maxStreamedResultBytes" | "maxAtomicResultBytes";
```

Capability field `maxBytes` came from, for the error it raises.

##### maxBytes

```ts
maxBytes: number;
```

Encoded reply budget in bytes (`0` = only the framing slack).

##### maxRows

```ts
maxRows: number;
```

Rows one statement may return (`0` = unlimited).

##### rowsLimit

```ts
rowsLimit: "maxStreamedResultRows" | "maxResultRows";
```

Capability field `maxRows` came from, for the error it raises.

***

### LimitExceeded

A negotiated SQL-v1 cap that a request or its result exceeded
(`limit_exceeded` errors).

#### Properties

##### cap

```ts
cap: number;
```

The negotiated cap.

##### limit

```ts
limit: string;
```

Capability field name as advertised (`maxResultBytes`, `maxRequestBytes`, ...).

##### used

```ts
used: number;
```

The observed value that exceeded `cap`.

## Type Aliases

### AdapterBatchStatement

```ts
type AdapterBatchStatement = Pick<AdapterStatement, "sql" | "parameters" | "kind" | "maxRows" | "resultSelection">;
```

The parts of an [AdapterStatement](generated.md#adapterstatement) an adapter executes.

## Variables

### GUEST\_READ\_ONLY\_REPLAY\_PAYLOAD

```ts
const GUEST_READ_ONLY_REPLAY_PAYLOAD: "{\"readOnly\":true}" = '{"readOnly":true}';
```

Payload stored for a read-only guest batch instead of its rows: reads are
not gated, so they run again on replay and need no recorded result.

***

### GUEST\_RECEIPT\_CLAIM\_LOST

```ts
const GUEST_RECEIPT_CLAIM_LOST: "guest receipt claimed by a concurrent attempt; retry to replay it" = "guest receipt claimed by a concurrent attempt; retry to replay it";
```

`unavailable` message when a concurrent attempt committed the receipt
between this attempt's prior-receipt `SELECT` and its stub `INSERT`.

***

### GUEST\_RECEIPT\_RESULT\_LOST

```ts
const GUEST_RECEIPT_RESULT_LOST: "guest receipt committed without payload; original response lost" = "guest receipt committed without payload; original response lost";
```

Error text when a receipt committed but the reply it should hold was lost.

***

### GUEST\_RECEIPT\_STUB\_SUFFIX

```ts
const GUEST_RECEIPT_STUB_SUFFIX: 1 = 1;
```

Receipt stub `INSERT` after the guest statements.

***

### GUEST\_RECEIPT\_WRAP\_PREFIX

```ts
const GUEST_RECEIPT_WRAP_PREFIX: 2 = 2;
```

Prune + prior-receipt `SELECT` statements ahead of the guest statements.

***

### GUEST\_REPLAY\_PAYLOAD\_MAX\_BYTES

```ts
const GUEST_REPLAY_PAYLOAD_MAX_BYTES: 65536 = 65536;
```

Largest replay payload, in UTF-8 bytes, a receipt stores for a write batch.

***

### STREAM\_FRAME\_TARGET\_BYTES

```ts
const STREAM_FRAME_TARGET_BYTES: number;
```

Encoded size an encoder aims for per `rows` frame (one wider row may exceed it).

## Functions

### collectExecuteStream()

```ts
function collectExecuteStream(chunks, limits): Promise<ExecuteReply>;
```

Collects an async chunk source into one reply under `limits`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `chunks` | `AsyncIterable`\<`Uint8Array`\<`ArrayBufferLike`\>\> | Frame bytes as the transport delivers them. |
| `limits` | [`ExecuteStreamLimits`](#executestreamlimits) | Budget the stream is held to. |

#### Returns

`Promise`\<[`ExecuteReply`](db-execute.md#executereply)\>

The reassembled reply.

***

### decodeExecuteStream()

```ts
function decodeExecuteStream(bytes, limits): ExecuteReply;
```

Decodes a whole frame sequence under `limits`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bytes` | `Uint8Array` | Every frame, concatenated. |
| `limits` | [`ExecuteStreamLimits`](#executestreamlimits) | Budget the stream is held to. |

#### Returns

[`ExecuteReply`](db-execute.md#executereply)

The reassembled reply.

#### Throws

See [ExecuteStreamDecoder.push](#push) and [ExecuteStreamDecoder.finish](#finish).

***

### encodeExecuteStream()

```ts
function encodeExecuteStream(
   reply, 
   frameTargetBytes?, 
   maxBytes?
): Uint8Array;
```

Frames `reply` as the byte sequence `executeStreamed` answers: `head`,
each statement with its rows split near `frameTargetBytes`, then `end`. A
slice whose frame still exceeds one scalar is halved until each frame fits
or holds one row.

#### Parameters

| Parameter | Type | Default value | Description |
| ------ | ------ | ------ | ------ |
| `reply` | [`ExecuteReply`](db-execute.md#executereply) | `undefined` | The committed reply. |
| `frameTargetBytes?` | `number` | `STREAM_FRAME_TARGET_BYTES` | Estimated encoded size to aim for per `rows` frame. |
| `maxBytes?` | `number` | `Number.POSITIVE_INFINITY` | Stop as soon as the frames pass this many bytes. |

#### Returns

`Uint8Array`

Every frame, concatenated.

#### Throws

`PluginError` `limit_exceeded` (`maxScalarBytes`) when the frames
  pass `maxBytes`.

***

### executeAdapterBatch()

```ts
function executeAdapterBatch(
   request, 
   begin, 
   options
): Promise<ExecuteReply>;
```

Runs one host batch in a transaction and commits it with its replay receipt.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`AdapterBatchRequest`](#adapterbatchrequest) | The adapter request (`execute`). |
| `begin` | () => `Promise`\<[`AdapterTransaction`](#adaptertransaction)\> | Opens a transaction on the adapter's engine. |
| `options` | [`AdapterBatchOptions`](#adapterbatchoptions) | Timing source. |

#### Returns

`Promise`\<[`ExecuteReply`](db-execute.md#executereply)\>

The committed reply, wrap statements included (the host unwraps it).

#### Throws

Whatever a statement, the finalize or `commit` raised, after the
  transaction was rolled back; `PluginError` `unavailable` when a prior
  receipt committed without its reply.

***

### executeAdapterBatchStreamed()

```ts
function executeAdapterBatchStreamed(
   request, 
   begin, 
   options
): Promise<Uint8Array<ArrayBufferLike>>;
```

Runs one host batch for `executeStreamed` and commits it only when the
host can receive its frames.

The frames are encoded inside the transaction. Encoding stops with
`limit_exceeded` (`maxScalarBytes`) as soon as they pass one invoke
scalar, the finished frames are decoded under `options.limits` the way
the host decodes them, and the reassembled reply's encoded size is held to
`options.limits.maxBytes` the way the host validates it (the decoder alone
allows framing slack past the budget). Any failure rolls the batch back,
so nothing commits that the host would refuse.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`AdapterBatchRequest`](#adapterbatchrequest) | The adapter request (`executeStreamed`). |
| `begin` | () => `Promise`\<[`AdapterTransaction`](#adaptertransaction)\> | Opens a transaction on the adapter's engine. |
| `options` | [`AdapterBatchStreamedOptions`](#adapterbatchstreamedoptions) | Timing source and the advertised streamed budget. |

#### Returns

`Promise`\<`Uint8Array`\<`ArrayBufferLike`\>\>

Every frame of the committed reply, concatenated.

#### Throws

`PluginError` `limit_exceeded` when the frames do not fit one
  scalar or the budget, or whatever [executeAdapterBatch](#executeadapterbatch) throws.

***

### executeStreamLimits()

```ts
function executeStreamLimits(caps): ExecuteStreamLimits;
```

The streamed budget of `caps`, or its inline caps when the session does
not stream.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `caps` | [`DbCapabilities`](generated.md#dbcapabilities) | Session capabilities (`limits()`). |

#### Returns

[`ExecuteStreamLimits`](#executestreamlimits)

Limits for an [ExecuteStreamDecoder](#executestreamdecoder).

***

### guestBatchIsReadOnly()

```ts
function guestBatchIsReadOnly(statements, guestLen): boolean;
```

True when every guest statement inside the receipt wrap is a `SELECT`.

An empty or out-of-range guest slice is not read-only, so a write batch
never skips its replay payload.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `statements` | readonly `Pick`\<[`AdapterBatchStatement`](#adapterbatchstatement), `"kind"`\>[] | The wrapped statements. |
| `guestLen` | `number` | Guest statement count from the receipt hint. |

#### Returns

`boolean`

Whether the batch only reads.

***

### guestReceiptFinalizeStatements()

```ts
function guestReceiptFinalizeStatements(
   partial, 
   guestLen, 
   guestHash, 
   readOnly
): AdapterBatchStatement[];
```

Statements that persist the guest reply on the claimed receipt.

Empty when the prior receipt already holds a payload, belongs to a
different request (the host reports the conflict), or is an in-flight
claim whose stub this attempt did not insert.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `partial` | [`ExecuteReply`](db-execute.md#executereply) | The wrapped reply so far (prefix, guest and stub results). |
| `guestLen` | `number` | Guest statement count from the receipt hint. |
| `guestHash` | `string` | Guest `requestHash` from the receipt hint. |
| `readOnly` | `boolean` | Whether the guest slice only reads ([guestBatchIsReadOnly](#guestbatchisreadonly)). |

#### Returns

[`AdapterBatchStatement`](#adapterbatchstatement)[]

The finalize statements to run before `COMMIT`.

#### Throws

`PluginError` `unavailable` when a prior receipt committed without
  its reply, `internal` when the wrap shape is malformed or the payload is
  over [GUEST\_REPLAY\_PAYLOAD\_MAX\_BYTES](#guest_replay_payload_max_bytes).

***

### guestReplayPayload()

```ts
function guestReplayPayload(reply): string;
```

The replay payload for a guest reply: the JSON envelope the host decodes,
`{"operation_id", "statements", "timing"}` with each value in the host's
serde form.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `reply` | [`ExecuteReply`](db-execute.md#executereply) | The guest slice of the reply. |

#### Returns

`string`

The payload text.

#### Throws

`PluginError` `internal` when a cell does not encode or the
  payload is over [GUEST\_REPLAY\_PAYLOAD\_MAX\_BYTES](#guest_replay_payload_max_bytes).

***

### limitExceededMessage()

```ts
function limitExceededMessage(limit): string;
```

Canonical message for a limit error: `limit exceeded: maxResultBytes is 262144; used 300000`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `limit` | [`LimitExceeded`](#limitexceeded-1) | Exceeded cap. |

#### Returns

`string`

Message text the host and every SDK parse back with [parseLimitExceeded](#parselimitexceeded).

***

### parseLimitExceeded()

```ts
function parseLimitExceeded(message): LimitExceeded | undefined;
```

Parses a message produced by [limitExceededMessage](#limitexceededmessage), possibly wrapped
in further text.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `message` | `string` | Error text. |

#### Returns

[`LimitExceeded`](#limitexceeded-1) \| `undefined`

The exceeded cap, or `undefined` when the text is not a limit error.

***

### rawStreamCeiling()

```ts
function rawStreamCeiling(limits): number;
```

Raw frame bytes a receiver accepts before failing closed: the reply budget
plus one eighth for frame headers and one window of slack.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `limits` | [`ExecuteStreamLimits`](#executestreamlimits) | Budget. |

#### Returns

`number`

Byte ceiling on the whole frame sequence and on any one frame.

***

### requirePluginMigrationRegistration()

```ts
function requirePluginMigrationRegistration(migrations): PluginMigration[];
```

Rejects a plugin migration registration that exceeds ABI resource limits.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `migrations` | [`PluginMigration`](generated.md#pluginmigration)[] | Ordered `databaseMigrations` registration. |

#### Returns

[`PluginMigration`](generated.md#pluginmigration)[]

The same array when within limits.

#### Throws

Error with `code` / `wireCode` `payload_too_large` when a limit is exceeded.

***

### runAdapterBatch()

```ts
function runAdapterBatch(
   request, 
   txn, 
   options
): Promise<ExecuteReply>;
```

Runs the wrapped statements and the receipt finalize on an open
transaction, without committing.

After the prior-receipt `SELECT`, a completed receipt for this operation
skips the remaining guest work (its results are padded so the wrap shape
stays intact) and the host replays the stored reply.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`AdapterBatchRequest`](#adapterbatchrequest) | The adapter request. |
| `txn` | `Pick`\<[`AdapterTransaction`](#adaptertransaction), `"run"`\> | The open transaction. |
| `options` | [`AdapterBatchOptions`](#adapterbatchoptions) | Timing source. |

#### Returns

`Promise`\<[`ExecuteReply`](db-execute.md#executereply)\>

The reply to deliver once the transaction commits.

#### Throws

`PluginError` `unavailable` when a prior receipt committed without
  its reply, `internal` when the wrap shape is malformed or the payload is
  over [GUEST\_REPLAY\_PAYLOAD\_MAX\_BYTES](#guest_replay_payload_max_bytes), `invalid_params` when a
  result cell is not a valid `DbValue`, or whatever `run` threw.

***

### shouldSkipRemainingGuestWork()

```ts
function shouldSkipRemainingGuestWork(
   results, 
   total, 
   guestHash
): boolean;
```

True when the prefix results show a completed prior receipt, so the guest
statements must not run again.

An in-flight claim with the same hash and a read-only receipt with the
same hash do not skip; a receipt with a different hash does (the host
reports the conflict).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `results` | readonly [`StatementResult`](db-execute.md#statementresult)[] | Results so far. |
| `total` | `number` | Wrapped statement count. |
| `guestHash` | `string` | Guest `requestHash` from the receipt hint. |

#### Returns

`boolean`

Whether to pad the remaining results instead of running them.

***

### streamsResults()

```ts
function streamsResults(caps): boolean;
```

True when `caps` advertises a streamed result budget.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `caps` | [`DbCapabilities`](generated.md#dbcapabilities) | Session capabilities (`limits()`). |

#### Returns

`boolean`

Whether `executeStreamed` is worth calling on this session.

## References

### Abridgement

Re-exports [Abridgement](abi.md#abridgement)

***

### AccountCredential

Re-exports [AccountCredential](generated.md#accountcredential)

***

### AdapterDatabaseSession

Re-exports [AdapterDatabaseSession](plugin.md#adapterdatabasesession)

***

### AdapterEnv

Re-exports [AdapterEnv](plugin.md#adapterenv)

***

### AdapterExecuteRequest

Re-exports [AdapterExecuteRequest](generated.md#adapterexecuterequest)

***

### AdapterReceipt

Re-exports [AdapterReceipt](generated.md#adapterreceipt)

***

### AdapterSessionReply

Re-exports [AdapterSessionReply](generated.md#adaptersessionreply)

***

### AdapterStatement

Re-exports [AdapterStatement](generated.md#adapterstatement)

***

### allowedEntrypointFamilies

Re-exports [allowedEntrypointFamilies](plugin.md#allowedentrypointfamilies)

***

### AtomicTransport

Re-exports [AtomicTransport](db-execute.md#atomictransport)

***

### AuthenticateUserParams

Re-exports [AuthenticateUserParams](generated.md#authenticateuserparams)

***

### AuthorStub

Re-exports [AuthorStub](plugin.md#authorstub)

***

### Bindings

Re-exports [Bindings](generated.md#bindings)

***

### BookclerkEntrypoint

Re-exports [BookclerkEntrypoint](plugin.md#bookclerkentrypoint)

***

### BookclerkEnv

Re-exports [BookclerkEnv](env.md#bookclerkenv)

***

### Brand

Re-exports [Brand](generated.md#brand)

***

### ByteRange

Re-exports [ByteRange](generated.md#byterange)

***

### ByteSource

Re-exports [ByteSource](generated.md#bytesource)

***

### Cancellation

Re-exports [Cancellation](generated.md#cancellation)

***

### CancelledOutcome

Re-exports [CancelledOutcome](generated.md#cancelledoutcome)

***

### CancelWatchLike

Re-exports [CancelWatchLike](plugin.md#cancelwatchlike)

***

### canonicalExecuteRequestHash

Re-exports [canonicalExecuteRequestHash](db-execute.md#canonicalexecuterequesthash)

***

### CatalogCapabilities

Re-exports [CatalogCapabilities](generated.md#catalogcapabilities)

***

### CatalogDetail

Re-exports [CatalogDetail](generated.md#catalogdetail)

***

### CatalogDetailParams

Re-exports [CatalogDetailParams](generated.md#catalogdetailparams)

***

### CatalogDetailReply

Re-exports [CatalogDetailReply](generated.md#catalogdetailreply)

***

### CatalogField

Re-exports [CatalogField](abi.md#catalogfield)

***

### CatalogHit

Re-exports [CatalogHit](generated.md#cataloghit)

***

### CatalogHits

Re-exports [CatalogHits](generated.md#cataloghits)

***

### CatalogHitsReply

Re-exports [CatalogHitsReply](generated.md#cataloghitsreply)

***

### CatalogSort

Re-exports [CatalogSort](abi.md#catalogsort)

***

### ChapterMarker

Re-exports [ChapterMarker](generated.md#chaptermarker)

***

### Checkpoint

Re-exports [Checkpoint](plugin.md#checkpoint-4)

***

### chunkIndexes

Re-exports [chunkIndexes](db-execute.md#chunkindexes)

***

### CliArg

Re-exports [CliArg](generated.md#cliarg)

***

### CliArgKind

Re-exports [CliArgKind](abi.md#cliargkind)

***

### cliArgs

Re-exports [cliArgs](plugin.md#cliargs)

***

### CliArgSpec

Re-exports [CliArgSpec](generated.md#cliargspec)

***

### CliCommandSpec

Re-exports [CliCommandSpec](generated.md#clicommandspec)

***

### CliEntrypoint

Re-exports [CliEntrypoint](plugin.md#clientrypoint)

***

### CliInvokeParams

Re-exports [CliInvokeParams](generated.md#cliinvokeparams)

***

### CliInvokeReply

Re-exports [CliInvokeReply](generated.md#cliinvokereply)

***

### CliInvokeResult

Re-exports [CliInvokeResult](generated.md#cliinvokeresult)

***

### CliSchema

Re-exports [CliSchema](generated.md#clischema)

***

### CliSchemaReply

Re-exports [CliSchemaReply](generated.md#clischemareply)

***

### ColumnReference

Re-exports [ColumnReference](generated.md#columnreference)

***

### CompletedOutcome

Re-exports [CompletedOutcome](generated.md#completedoutcome)

***

### ConfigOption

Re-exports [ConfigOption](generated.md#configoption)

***

### ConfigOptionValue

Re-exports [ConfigOptionValue](generated.md#configoptionvalue)

***

### ContentSource

Re-exports [ContentSource](generated.md#contentsource)

***

### CopyReply

Re-exports [CopyReply](generated.md#copyreply)

***

### CopyResult

Re-exports [CopyResult](plugin.md#copyresult)

***

### CoverCondition

Re-exports [CoverCondition](generated.md#covercondition)

***

### CoverEntrypoint

Re-exports [CoverEntrypoint](plugin.md#coverentrypoint)

***

### CoverFetcher

Re-exports [CoverFetcher](generated.md#coverfetcher)

***

### CoverFetchParams

Re-exports [CoverFetchParams](generated.md#coverfetchparams)

***

### CoverImage

Re-exports [CoverImage](generated.md#coverimage)

***

### CoverImages

Re-exports [CoverImages](generated.md#coverimages)

***

### CoverImagesReply

Re-exports [CoverImagesReply](generated.md#coverimagesreply)

***

### createDatabaseBinding

Re-exports [createDatabaseBinding](db-execute.md#createdatabasebinding)

***

### CreateTableSchema

Re-exports [CreateTableSchema](generated.md#createtableschema)

***

### Credit

Re-exports [Credit](generated.md#credit)

***

### CreditLevel

Re-exports [CreditLevel](abi.md#creditlevel)

***

### CreditRole

Re-exports [CreditRole](abi.md#creditrole)

***

### D1ExecResult

Re-exports [D1ExecResult](db-execute.md#d1execresult)

***

### D1Meta

Re-exports [D1Meta](db-execute.md#d1meta)

***

### D1Result

Re-exports [D1Result](db-execute.md#d1result)

***

### Database

Re-exports [Database](generated.md#database)

***

### DatabaseAdapterConfig

Re-exports [DatabaseAdapterConfig](generated.md#databaseadapterconfig)

***

### DatabaseAdapterEntrypoint

Re-exports [DatabaseAdapterEntrypoint](plugin.md#databaseadapterentrypoint)

***

### DatabaseBinding

Re-exports [DatabaseBinding](db-execute.md#databasebinding)

***

### DatabaseBindingOptions

Re-exports [DatabaseBindingOptions](db-execute.md#databasebindingoptions)

***

### dataMigrationOp

Re-exports [dataMigrationOp](plugin.md#datamigrationop)

***

### DbBootstrap

Re-exports [DbBootstrap](generated.md#dbbootstrap)

***

### DbBootstrapReply

Re-exports [DbBootstrapReply](generated.md#dbbootstrapreply)

***

### DbCapabilities

Re-exports [DbCapabilities](generated.md#dbcapabilities)

***

### DbCapabilitiesReply

Re-exports [DbCapabilitiesReply](generated.md#dbcapabilitiesreply)

***

### DbColumn

Re-exports [DbColumn](db-execute.md#dbcolumn)

***

### DbResultSelection

Re-exports [DbResultSelection](abi.md#dbresultselection)

***

### DbRow

Re-exports [DbRow](db-execute.md#dbrow)

***

### DbStatement

Re-exports [DbStatement](generated.md#dbstatement)

***

### DbStatementKind

Re-exports [DbStatementKind](abi.md#dbstatementkind)

***

### DbTiming

Re-exports [DbTiming](db-execute.md#dbtiming)

***

### DbType

Re-exports [DbType](abi.md#dbtype)

***

### DbValue

Re-exports [DbValue](db-value.md#dbvalue)

***

### decodeDbCapabilitiesReply

Re-exports [decodeDbCapabilitiesReply](db-execute.md#decodedbcapabilitiesreply)

***

### decodeDbValue

Re-exports [decodeDbValue](db-value.md#decodedbvalue)

***

### decodeExecuteRequest

Re-exports [decodeExecuteRequest](db-execute.md#decodeexecuterequest)

***

### decodeExecuteResultReply

Re-exports [decodeExecuteResultReply](db-execute.md#decodeexecuteresultreply)

***

### decodeExtensibleConfig

Re-exports [decodeExtensibleConfig](plugin.md#decodeextensibleconfig)

***

### DeliveredCheckpoint

Re-exports [DeliveredCheckpoint](plugin.md#deliveredcheckpoint)

***

### DescribeReply

Re-exports [DescribeReply](generated.md#describereply)

***

### Destination

Re-exports [Destination](plugin.md#destination)

***

### DiagnoseReply

Re-exports [DiagnoseReply](generated.md#diagnosereply)

***

### DiagnoseResult

Re-exports [DiagnoseResult](generated.md#diagnoseresult)

***

### DomainEvent

Re-exports [DomainEvent](generated.md#domainevent)

***

### EmptyReply

Re-exports [EmptyReply](generated.md#emptyreply)

***

### encodeDbCapabilitiesReply

Re-exports [encodeDbCapabilitiesReply](db-execute.md#encodedbcapabilitiesreply)

***

### encodeDbValue

Re-exports [encodeDbValue](db-value.md#encodedbvalue)

***

### encodeExecuteRequest

Re-exports [encodeExecuteRequest](db-execute.md#encodeexecuterequest)

***

### encodeExecuteResultReply

Re-exports [encodeExecuteResultReply](db-execute.md#encodeexecuteresultreply)

***

### Entrypoint

Re-exports [Entrypoint](abi.md#entrypoint)

***

### ENTRYPOINT\_BINDINGS

Re-exports [ENTRYPOINT_BINDINGS](plugin.md#entrypoint_bindings)

***

### ENTRYPOINT\_CLASSES

Re-exports [ENTRYPOINT_CLASSES](plugin.md#entrypoint_classes)

***

### EntrypointName

Re-exports [EntrypointName](plugin.md#entrypointname)

***

### Entrypoints

Re-exports [Entrypoints](generated.md#entrypoints)

***

### EntrypointsReply

Re-exports [EntrypointsReply](generated.md#entrypointsreply)

***

### EventAck

Re-exports [EventAck](generated.md#eventack)

***

### EventBatch

Re-exports [EventBatch](plugin.md#eventbatch)

***

### EventBatchReply

Re-exports [EventBatchReply](generated.md#eventbatchreply)

***

### eventBatchResults

Re-exports [eventBatchResults](plugin.md#eventbatchresults)

***

### EventConsumer

Re-exports [EventConsumer](generated.md#eventconsumer-1)

***

### EventConsumerSpec

Re-exports [EventConsumerSpec](generated.md#eventconsumerspec)

***

### EventDeadLetter

Re-exports [EventDeadLetter](generated.md#eventdeadletter)

***

### EventMessage

Re-exports [EventMessage](plugin.md#eventmessage)

***

### EventOutcome

Re-exports [EventOutcome](plugin.md#eventoutcome)

***

### EventPollReply

Re-exports [EventPollReply](generated.md#eventpollreply)

***

### EventPollResult

Re-exports [EventPollResult](generated.md#eventpollresult)

***

### EventPublisher

Re-exports [EventPublisher](generated.md#eventpublisher)

***

### EventPublisherBinding

Re-exports [EventPublisherBinding](env.md#eventpublisherbinding)

***

### EventReject

Re-exports [EventReject](generated.md#eventreject)

***

### EventResult

Re-exports [EventResult](generated.md#eventresult)

***

### EventRetry

Re-exports [EventRetry](generated.md#eventretry)

***

### EventSuspended

Re-exports [EventSuspended](generated.md#eventsuspended)

***

### EventSuspendOptions

Re-exports [EventSuspendOptions](plugin.md#eventsuspendoptions)

***

### ExecuteReply

Re-exports [ExecuteReply](db-execute.md#executereply)

***

### executeReplyToD1Results

Re-exports [executeReplyToD1Results](db-execute.md#executereplytod1results)

***

### ExecuteRequest

Re-exports [ExecuteRequest](db-execute.md#executerequest)

***

### ExecuteResultReply

Re-exports [ExecuteResultReply](generated.md#executeresultreply)

***

### ExecuteStreamFrame

Re-exports [ExecuteStreamFrame](generated.md#executestreamframe)

***

### ExecuteStreamHead

Re-exports [ExecuteStreamHead](generated.md#executestreamhead)

***

### ExecuteStreamReply

Re-exports [ExecuteStreamReply](generated.md#executestreamreply)

***

### ExecuteStreamStatement

Re-exports [ExecuteStreamStatement](generated.md#executestreamstatement)

***

### ExpandCandidatesParams

Re-exports [ExpandCandidatesParams](generated.md#expandcandidatesparams)

***

### ExpressionKind

Re-exports [ExpressionKind](abi.md#expressionkind)

***

### ExtensibleConfig

Re-exports [ExtensibleConfig](generated.md#extensibleconfig)

***

### ExternalUser

Re-exports [ExternalUser](generated.md#externaluser)

***

### ExternalUserReply

Re-exports [ExternalUserReply](generated.md#externaluserreply)

***

### FEATURE\_SCALAR\_LIMITS

Re-exports [FEATURE_SCALAR_LIMITS](abi.md#feature_scalar_limits)

***

### FEATURE\_STORAGE\_COPY

Re-exports [FEATURE_STORAGE_COPY](abi.md#feature_storage_copy)

***

### FEATURE\_STREAMS

Re-exports [FEATURE_STREAMS](abi.md#feature_streams)

***

### FetchOptions

Re-exports [FetchOptions](generated.md#fetchoptions)

***

### FetchTitleParams

Re-exports [FetchTitleParams](generated.md#fetchtitleparams)

***

### FetchTitleReply

Re-exports [FetchTitleReply](generated.md#fetchtitlereply)

***

### ForeignKeyConstraint

Re-exports [ForeignKeyConstraint](generated.md#foreignkeyconstraint)

***

### GetOk

Re-exports [GetOk](generated.md#getok)

***

### GetReply

Re-exports [GetReply](generated.md#getreply)

***

### GrantedContext

Re-exports [GrantedContext](plugin.md#grantedcontext)

***

### GrantedDatabaseTransport

Re-exports [GrantedDatabaseTransport](plugin.md#granteddatabasetransport)

***

### GrantedFetcher

Re-exports [GrantedFetcher](plugin.md#grantedfetcher)

***

### GrantedJobCapabilities

Re-exports [GrantedJobCapabilities](plugin.md#grantedjobcapabilities)

***

### GuestDatabase

Re-exports [GuestDatabase](generated.md#guestdatabase)

***

### GuestDatabaseReply

Re-exports [GuestDatabaseReply](generated.md#guestdatabasereply)

***

### HandleReply

Re-exports [HandleReply](generated.md#handlereply)

***

### HeadOk

Re-exports [HeadOk](generated.md#headok)

***

### HeadReply

Re-exports [HeadReply](generated.md#headreply)

***

### HealthOk

Re-exports [HealthOk](generated.md#healthok)

***

### HealthReply

Re-exports [HealthReply](generated.md#healthreply)

***

### Identifier

Re-exports [Identifier](generated.md#identifier)

***

### IdentifierScheme

Re-exports [IdentifierScheme](abi.md#identifierscheme)

***

### IdentityExportReply

Re-exports [IdentityExportReply](generated.md#identityexportreply)

***

### IdentityHighWater

Re-exports [IdentityHighWater](generated.md#identityhighwater)

***

### Instant

Re-exports [Instant](plugin.md#instant)

***

### IntegerArithKind

Re-exports [IntegerArithKind](abi.md#integerarithkind)

***

### IntegerArithSite

Re-exports [IntegerArithSite](generated.md#integerarithsite)

***

### Invocation

Re-exports [Invocation](generated.md#invocation)

***

### invocationOf

Re-exports [invocationOf](plugin.md#invocationof)

***

### IsolationReq

Re-exports [IsolationReq](abi.md#isolationreq)

***

### JobCheckpoint

Re-exports [JobCheckpoint](plugin.md#jobcheckpoint)

***

### JobCompletion

Re-exports [JobCompletion](plugin.md#jobcompletion)

***

### JobController

Re-exports [JobController](plugin.md#jobcontroller)

***

### JobInvocation

Re-exports [JobInvocation](generated.md#jobinvocation)

***

### JobOutcome

Re-exports [JobOutcome](generated.md#joboutcome)

***

### jobOutcomeFor

Re-exports [jobOutcomeFor](plugin.md#joboutcomefor)

***

### JobOutcomeRecord

Re-exports [JobOutcomeRecord](plugin.md#joboutcomerecord)

***

### JobRetryOptions

Re-exports [JobRetryOptions](plugin.md#jobretryoptions)

***

### JobRunner

Re-exports [JobRunner](generated.md#jobrunner-1)

***

### JobRunnerContext

Re-exports [JobRunnerContext](plugin.md#jobrunnercontext)

***

### JobSuspendOptions

Re-exports [JobSuspendOptions](plugin.md#jobsuspendoptions)

***

### JsonObject

Re-exports [JsonObject](env.md#jsonobject)

***

### jsonPayload

Re-exports [jsonPayload](plugin.md#jsonpayload)

***

### KvDeleteOk

Re-exports [KvDeleteOk](generated.md#kvdeleteok)

***

### KvDeleteReply

Re-exports [KvDeleteReply](generated.md#kvdeletereply)

***

### KvGetOk

Re-exports [KvGetOk](generated.md#kvgetok)

***

### KvGetReply

Re-exports [KvGetReply](generated.md#kvgetreply)

***

### KvListKey

Re-exports [KvListKey](generated.md#kvlistkey)

***

### KvListKeyBinding

Re-exports [KvListKeyBinding](env.md#kvlistkeybinding)

***

### KvListOk

Re-exports [KvListOk](generated.md#kvlistok)

***

### KvListOptions

Re-exports [KvListOptions](generated.md#kvlistoptions)

***

### KvListOptionsBinding

Re-exports [KvListOptionsBinding](env.md#kvlistoptionsbinding)

***

### KvListReply

Re-exports [KvListReply](generated.md#kvlistreply)

***

### KvListResultBinding

Re-exports [KvListResultBinding](env.md#kvlistresultbinding)

***

### KvNamespace

Re-exports [KvNamespace](generated.md#kvnamespace)

***

### KvNamespaceBinding

Re-exports [KvNamespaceBinding](env.md#kvnamespacebinding)

***

### KvPutOk

Re-exports [KvPutOk](generated.md#kvputok)

***

### KvPutOptions

Re-exports [KvPutOptions](generated.md#kvputoptions)

***

### KvPutOptionsBinding

Re-exports [KvPutOptionsBinding](env.md#kvputoptionsbinding)

***

### KvPutReply

Re-exports [KvPutReply](generated.md#kvputreply)

***

### KvValueType

Re-exports [KvValueType](env.md#kvvaluetype)

***

### KvValueWithMetadata

Re-exports [KvValueWithMetadata](env.md#kvvaluewithmetadata)

***

### ListDealsParams

Re-exports [ListDealsParams](generated.md#listdealsparams)

***

### ListeningProgress

Re-exports [ListeningProgress](generated.md#listeningprogress)

***

### ListOptions

Re-exports [ListOptions](plugin.md#listoptions)

***

### ListPage

Re-exports [ListPage](plugin.md#listpage)

***

### ListReply

Re-exports [ListReply](generated.md#listreply)

***

### LoginCompleteParams

Re-exports [LoginCompleteParams](generated.md#logincompleteparams)

***

### LoginParams

Re-exports [LoginParams](generated.md#loginparams)

***

### LoginReply

Re-exports [LoginReply](generated.md#loginreply)

***

### LoginResult

Re-exports [LoginResult](generated.md#loginresult)

***

### LoginStartReply

Re-exports [LoginStartReply](generated.md#loginstartreply)

***

### LoginStartResult

Re-exports [LoginStartResult](generated.md#loginstartresult)

***

### MAX\_CHECKPOINT\_BYTES

Re-exports [MAX_CHECKPOINT_BYTES](abi.md#max_checkpoint_bytes)

***

### MAX\_LIST\_PAGE

Re-exports [MAX_LIST_PAGE](abi.md#max_list_page)

***

### MAX\_PLUGIN\_MIGRATION\_OPS

Re-exports [MAX_PLUGIN_MIGRATION_OPS](abi.md#max_plugin_migration_ops)

***

### MAX\_PLUGIN\_MIGRATION\_REGISTRATION\_BYTES

Re-exports [MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES](abi.md#max_plugin_migration_registration_bytes)

***

### MAX\_PLUGIN\_MIGRATION\_TOTAL\_OPS

Re-exports [MAX_PLUGIN_MIGRATION_TOTAL_OPS](abi.md#max_plugin_migration_total_ops)

***

### MAX\_SCALAR\_BYTES

Re-exports [MAX_SCALAR_BYTES](abi.md#max_scalar_bytes)

***

### MAX\_STREAM\_WINDOW\_BYTES

Re-exports [MAX_STREAM_WINDOW_BYTES](abi.md#max_stream_window_bytes)

***

### MAX\_WRITE\_KEY\_BYTES

Re-exports [MAX_WRITE_KEY_BYTES](db-execute.md#max_write_key_bytes)

***

### MediaKind

Re-exports [MediaKind](abi.md#mediakind)

***

### MetadataChapter

Re-exports [MetadataChapter](generated.md#metadatachapter)

***

### MetadataChapters

Re-exports [MetadataChapters](generated.md#metadatachapters)

***

### MetadataChaptersParams

Re-exports [MetadataChaptersParams](generated.md#metadatachaptersparams)

***

### MetadataChaptersReply

Re-exports [MetadataChaptersReply](generated.md#metadatachaptersreply)

***

### MetadataEntrypoint

Re-exports [MetadataEntrypoint](plugin.md#metadataentrypoint)

***

### MetadataLookupParams

Re-exports [MetadataLookupParams](generated.md#metadatalookupparams)

***

### MetadataProvider

Re-exports [MetadataProvider](generated.md#metadataprovider)

***

### MetadataRecord

Re-exports [MetadataRecord](generated.md#metadatarecord)

***

### MetadataRecords

Re-exports [MetadataRecords](generated.md#metadatarecords)

***

### MetadataRecordsReply

Re-exports [MetadataRecordsReply](generated.md#metadatarecordsreply)

***

### MetadataSearchParams

Re-exports [MetadataSearchParams](generated.md#metadatasearchparams)

***

### NamedDatabase

Re-exports [NamedDatabase](plugin.md#nameddatabase)

***

### NamedEntrypoint

Re-exports [NamedEntrypoint](plugin.md#namedentrypoint)

***

### NamedKv

Re-exports [NamedKv](plugin.md#namedkv)

***

### NamedQueue

Re-exports [NamedQueue](plugin.md#namedqueue)

***

### NamedSqlType

Re-exports [NamedSqlType](generated.md#namedsqltype)

***

### NamedStub

Re-exports [NamedStub](plugin.md#namedstub)

***

### ObjectInfo

Re-exports [ObjectInfo](plugin.md#objectinfo)

***

### ObjectMetadata

Re-exports [ObjectMetadata](plugin.md#objectmetadata)

***

### Oidc

Re-exports [Oidc](generated.md#oidc-1)

***

### OidcClientsOk

Re-exports [OidcClientsOk](generated.md#oidcclientsok)

***

### OidcClientsReply

Re-exports [OidcClientsReply](generated.md#oidcclientsreply)

***

### OidcClientTemplate

Re-exports [OidcClientTemplate](generated.md#oidcclienttemplate)

***

### OidcEntrypoint

Re-exports [OidcEntrypoint](plugin.md#oidcentrypoint)

***

### OpenOk

Re-exports [OpenOk](generated.md#openok)

***

### OpenReply

Re-exports [OpenReply](generated.md#openreply)

***

### OptionalColumnReference

Re-exports [OptionalColumnReference](generated.md#optionalcolumnreference)

***

### parseDbValue

Re-exports [parseDbValue](db-value.md#parsedbvalue)

***

### PhysicalAccess

Re-exports [PhysicalAccess](generated.md#physicalaccess)

***

### PlainFetch

Re-exports [PlainFetch](generated.md#plainfetch)

***

### PlainPart

Re-exports [PlainPart](generated.md#plainpart)

***

### PluginCapabilities

Re-exports [PluginCapabilities](generated.md#plugincapabilities)

***

### PluginCli

Re-exports [PluginCli](generated.md#plugincli)

***

### PluginDescribe

Re-exports [PluginDescribe](plugin.md#plugindescribe)

***

### PluginErrorCode

Re-exports [PluginErrorCode](abi.md#pluginerrorcode)

***

### PluginEvent

Re-exports [PluginEvent](generated.md#pluginevent)

***

### PluginMigration

Re-exports [PluginMigration](generated.md#pluginmigration)

***

### PluginMigrationOp

Re-exports [PluginMigrationOp](generated.md#pluginmigrationop)

***

### PluginMigrationsOk

Re-exports [PluginMigrationsOk](generated.md#pluginmigrationsok)

***

### PluginMigrationsReply

Re-exports [PluginMigrationsReply](generated.md#pluginmigrationsreply)

***

### PluginWorker

Re-exports [PluginWorker](generated.md#pluginworker)

***

### PortalAuthMode

Re-exports [PortalAuthMode](abi.md#portalauthmode)

***

### PreparedStatement

Re-exports [PreparedStatement](db-execute.md#preparedstatement)

***

### PRODUCT\_API\_VERSION

Re-exports [PRODUCT_API_VERSION](abi.md#product_api_version)

***

### ProgressSink

Re-exports [ProgressSink](plugin.md#progresssink)

***

### PublishEvent

Re-exports [PublishEvent](env.md#publishevent)

***

### PublishOk

Re-exports [PublishOk](generated.md#publishok)

***

### PublishReply

Re-exports [PublishReply](generated.md#publishreply)

***

### PullOk

Re-exports [PullOk](generated.md#pullok)

***

### PullReply

Re-exports [PullReply](generated.md#pullreply)

***

### PurchaseHint

Re-exports [PurchaseHint](generated.md#purchasehint-1)

***

### PurchaseHintParams

Re-exports [PurchaseHintParams](generated.md#purchasehintparams)

***

### PurchaseHintReply

Re-exports [PurchaseHintReply](generated.md#purchasehintreply)

***

### PurchaseHintResult

Re-exports [PurchaseHintResult](generated.md#purchasehintresult)

***

### PurchaseHintsParams

Re-exports [PurchaseHintsParams](generated.md#purchasehintsparams)

***

### PurchaseHintsReply

Re-exports [PurchaseHintsReply](generated.md#purchasehintsreply)

***

### PurchaseHintsResult

Re-exports [PurchaseHintsResult](generated.md#purchasehintsresult)

***

### PutReply

Re-exports [PutReply](generated.md#putreply)

***

### PutResult

Re-exports [PutResult](plugin.md#putresult)

***

### QueueBatch

Re-exports [QueueBatch](plugin.md#queuebatch)

***

### QueueBatchOk

Re-exports [QueueBatchOk](generated.md#queuebatchok)

***

### QueueBatchReply

Re-exports [QueueBatchReply](generated.md#queuebatchreply)

***

### queueBatchResults

Re-exports [queueBatchResults](plugin.md#queuebatchresults)

***

### QueueBinding

Re-exports [QueueBinding](env.md#queuebinding)

***

### QueueConsumer

Re-exports [QueueConsumer](generated.md#queueconsumer-1)

***

### QueueMessage

Re-exports [QueueMessage](plugin.md#queuemessage)

***

### QueueOutcome

Re-exports [QueueOutcome](plugin.md#queueoutcome)

***

### QueueProducer

Re-exports [QueueProducer](generated.md#queueproducer)

***

### QueueResult

Re-exports [QueueResult](generated.md#queueresult)

***

### QueueRetry

Re-exports [QueueRetry](generated.md#queueretry)

***

### QueueRetryOptions

Re-exports [QueueRetryOptions](plugin.md#queueretryoptions)

***

### QueueSendBatchEntry

Re-exports [QueueSendBatchEntry](env.md#queuesendbatchentry)

***

### QueueSendMessage

Re-exports [QueueSendMessage](generated.md#queuesendmessage)

***

### QueueSendOk

Re-exports [QueueSendOk](generated.md#queuesendok)

***

### QueueSendOptions

Re-exports [QueueSendOptions](env.md#queuesendoptions)

***

### QueueSendReply

Re-exports [QueueSendReply](generated.md#queuesendreply)

***

### ReadOptions

Re-exports [ReadOptions](plugin.md#readoptions)

***

### ReadResult

Re-exports [ReadResult](plugin.md#readresult)

***

### RejectedOutcome

Re-exports [RejectedOutcome](generated.md#rejectedoutcome)

***

### RemoteLibrary

Re-exports [RemoteLibrary](generated.md#remotelibrary-1)

***

### RemoteLibraryEntrypoint

Re-exports [RemoteLibraryEntrypoint](plugin.md#remotelibraryentrypoint)

***

### ResolvedAssignment

Re-exports [ResolvedAssignment](generated.md#resolvedassignment)

***

### ResolvedSqlType

Re-exports [ResolvedSqlType](abi.md#resolvedsqltype)

***

### ResolvedStatement

Re-exports [ResolvedStatement](generated.md#resolvedstatement)

***

### RetryableOutcome

Re-exports [RetryableOutcome](generated.md#retryableoutcome)

***

### RetryOptions

Re-exports [RetryOptions](plugin.md#retryoptions)

***

### RetryToken

Re-exports [RetryToken](db-execute.md#retrytoken)

***

### RevokeAccountParams

Re-exports [RevokeAccountParams](generated.md#revokeaccountparams)

***

### ScalarLimits

Re-exports [ScalarLimits](generated.md#scalarlimits-1)

***

### ScanBook

Re-exports [ScanBook](generated.md#scanbook)

***

### ScanLibraryParams

Re-exports [ScanLibraryParams](generated.md#scanlibraryparams)

***

### ScanPage

Re-exports [ScanPage](generated.md#scanpage)

***

### ScanParams

Re-exports [ScanParams](generated.md#scanparams)

***

### ScanReply

Re-exports [ScanReply](generated.md#scanreply)

***

### ScheduledController

Re-exports [ScheduledController](generated.md#scheduledcontroller)

***

### ScheduledHandler

Re-exports [ScheduledHandler](generated.md#scheduledhandler-1)

***

### SchemaAction

Re-exports [SchemaAction](generated.md#schemaaction-1)

***

### SchemaCreate

Re-exports [SchemaCreate](generated.md#schemacreate)

***

### schemaMigrationOp

Re-exports [schemaMigrationOp](plugin.md#schemamigrationop)

***

### SearchCatalogParams

Re-exports [SearchCatalogParams](generated.md#searchcatalogparams)

***

### SeriesEntry

Re-exports [SeriesEntry](generated.md#seriesentry)

***

### Source

Re-exports [Source](plugin.md#source-2)

***

### SourceAccount

Re-exports [SourceAccount](generated.md#sourceaccount)

***

### SourceAccounts

Re-exports [SourceAccounts](generated.md#sourceaccounts)

***

### SourceAccountsReply

Re-exports [SourceAccountsReply](generated.md#sourceaccountsreply)

***

### SqlSpan

Re-exports [SqlSpan](generated.md#sqlspan)

***

### StatementResult

Re-exports [StatementResult](db-execute.md#statementresult)

***

### statementResultToD1Result

Re-exports [statementResultToD1Result](db-execute.md#statementresulttod1result)

***

### StorageBinding

Re-exports [StorageBinding](env.md#storagebinding)

***

### StorageEntrypoint

Re-exports [StorageEntrypoint](plugin.md#storageentrypoint)

***

### StorefrontEntrypoint

Re-exports [StorefrontEntrypoint](plugin.md#storefrontentrypoint)

***

### SuspendedOutcome

Re-exports [SuspendedOutcome](generated.md#suspendedoutcome)

***

### SyncListeningReply

Re-exports [SyncListeningReply](generated.md#synclisteningreply)

***

### SyncListeningResult

Re-exports [SyncListeningResult](generated.md#synclisteningresult)

***

### TableConstraint

Re-exports [TableConstraint](generated.md#tableconstraint)

***

### TextCollateSite

Re-exports [TextCollateSite](generated.md#textcollatesite)

***

### toBridgeJson

Re-exports [toBridgeJson](plugin.md#tobridgejson)

***

### TypedDbStatement

Re-exports [TypedDbStatement](db-execute.md#typeddbstatement)

***

### UserRelationsReply

Re-exports [UserRelationsReply](generated.md#userrelationsreply)

***

### WireEventBatch

Re-exports [WireEventBatch](plugin.md#wireeventbatch)

***

### WireQueueBatch

Re-exports [WireQueueBatch](plugin.md#wirequeuebatch)

***

### WorkKind

Re-exports [WorkKind](abi.md#workkind)

***

### wrapPluginFromBinding

Re-exports [wrapPluginFromBinding](plugin.md#wrappluginfrombinding)

***

### wrapPluginFromNative

Re-exports [wrapPluginFromNative](plugin.md#wrappluginfromnative)

***

### WriteChunksError

Re-exports [WriteChunksError](db-execute.md#writechunkserror)

***

### writeChunksOperationId

Re-exports [writeChunksOperationId](db-execute.md#writechunksoperationid)

***

### WriteChunksOptions

Re-exports [WriteChunksOptions](db-execute.md#writechunksoptions)

***

### WriteOptions

Re-exports [WriteOptions](plugin.md#writeoptions)

***

### WriteProgress

Re-exports [WriteProgress](db-execute.md#writeprogress)
