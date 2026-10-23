# index

## Classes

### AdapterDatabaseSession

Host ↔ database adapter session returned by [DatabaseAdapterEntrypoint.openSession](#opensession).

#### Extends

- `RpcTarget`

#### Constructors

##### Constructor

```ts
new AdapterDatabaseSession(): AdapterDatabaseSession;
```

###### Returns

[`AdapterDatabaseSession`](#adapterdatabasesession)

###### Inherited from

```ts
RpcTarget.constructor
```

#### Methods

##### assertRestoreConstraints()

```ts
assertRestoreConstraints(): Promise<void>;
```

Re-check constraints after a unit restore; optional.

###### Returns

`Promise`\<`void`\>

Resolves when every constraint holds.

##### bootstrap()

```ts
bootstrap(): Promise<DbBootstrap>;
```

Bootstrap-only diagnostic metadata (`DbBootstrap`); optional.

###### Returns

`Promise`\<[`DbBootstrap`](#dbbootstrap)\>

Engine identity for diagnostics.

##### capabilities()

```ts
capabilities(): Promise<unknown>;
```

Typed SQL-contract advertisement.

###### Returns

`Promise`\<`unknown`\>

Guest `DbCapabilities`.

##### close()

```ts
close(): Promise<void>;
```

Close the adapter session.

###### Returns

`Promise`\<`void`\>

Resolves when the session is closed.

##### dropUserRelations()

```ts
dropUserRelations(_names): Promise<void>;
```

Drop the named user relations; optional.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_names` | `string`[] | Relation names from [AdapterDatabaseSession.listUserRelations](#listuserrelations). |

###### Returns

`Promise`\<`void`\>

Resolves when the relations are gone.

##### execute()

```ts
execute(_request): Promise<ExecuteReply>;
```

Typed atomic batch (`ExecuteRequest` → `ExecuteReply`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_request` | [`ExecuteRequest`](#executerequest) | Cap'n `ExecuteRequest` (structured Workers RPC object). |

###### Returns

`Promise`\<[`ExecuteReply`](#executereply)\>

`ExecuteReply`.

##### exportIdentity()

```ts
exportIdentity(): Promise<IdentityHighWater[]>;
```

Identity high-water marks for a logical export; optional.

###### Returns

`Promise`\<[`IdentityHighWater`](#identityhighwater)[]\>

One row per identity column.

##### importIdentity()

```ts
importIdentity(_rows): Promise<void>;
```

Restore identity high-water marks after a logical import; optional.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_rows` | [`IdentityHighWater`](#identityhighwater)[] | Rows from an earlier [AdapterDatabaseSession.exportIdentity](#exportidentity). |

###### Returns

`Promise`\<`void`\>

Resolves when the marks are applied.

##### listUserRelations()

```ts
listUserRelations(): Promise<string[]>;
```

Names of user relations the host may drop before a restore; optional.

###### Returns

`Promise`\<`string`[]\>

Relation names.

##### prepareUnitRestore()

```ts
prepareUnitRestore(): Promise<void>;
```

Prepare the engine for a unit restore (disable constraints, …); optional.

###### Returns

`Promise`\<`void`\>

Resolves when the engine is ready for the restore.

***

### BookclerkEntrypoint

Default-export base. Optional trigger methods: [BookclerkEntrypoint.event](#event)
and [BookclerkEntrypoint.job](#job); optional [BookclerkEntrypoint.describe](#describe)
(identity beyond `plugin.toml`), [BookclerkEntrypoint.databaseMigrations](#databasemigrations),
and [BookclerkEntrypoint.shutdown](#shutdown).

The `bookclerk*` methods are the adapter-facing dispatch surface; authors
neither call nor override them.

#### Example

```ts
import { BookclerkEntrypoint, type EventBatch } from "@bookclerk/plugin-sdk/workerd";
import type { Env } from "./bookclerk-configuration.js";

export default class MyPlugin extends BookclerkEntrypoint<Env> {
  async event(batch: EventBatch) {
    for (const msg of batch.messages) {
      await this.env.DB.prepare("INSERT INTO seen (id) VALUES (?)").bind(msg.id).run();
      msg.ack();
    }
  }
}
```

#### Extends

- `WorkerEntrypoint`\<`Env`\>

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `Env` *extends* [`BookclerkEnv`](#bookclerkenv) | [`BookclerkEnv`](#bookclerkenv) |

#### Constructors

##### Constructor

```ts
new BookclerkEntrypoint<Env>(ctx, env): BookclerkEntrypoint<Env>;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `ExecutionContext` |
| `env` | `Env` |

###### Returns

[`BookclerkEntrypoint`](#bookclerkentrypoint)\<`Env`\>

###### Inherited from

```ts
WorkerEntrypoint<Env>.constructor
```

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

```ts
WorkerEntrypoint.ctx
```

##### env

```ts
readonly env: Env;
```

###### Inherited from

```ts
WorkerEntrypoint.env
```

#### Methods

##### bookclerkDatabaseMigrations()

```ts
bookclerkDatabaseMigrations(binding): Promise<PluginMigration[]>;
```

**`Internal`**

Adapter dispatch for `databaseMigrations(binding)`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `binding` | `string` | Binding name from `[[databases]]`. |

###### Returns

`Promise`\<[`PluginMigration`](#pluginmigration)[]\>

Bounded registration (`[]` when not implemented).

##### bookclerkDescribe()

```ts
bookclerkDescribe(): Promise<Partial<PluginDescribe> | null>;
```

**`Internal`**

Adapter dispatch for `describe()`.

###### Returns

`Promise`\<`Partial`\<`PluginDescribe`\> \| `null`\>

Author refinement or `null` when not implemented.

##### bookclerkEvent()

```ts
bookclerkEvent(context, wireBatch): Promise<EventOutcome[]>;
```

**`Internal`**

Adapter dispatch for the `event(batch)` trigger.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `wireBatch` | [`WireEventBatch`](#wireeventbatch) | Wire `EventBatch`. |

###### Returns

`Promise`\<[`EventOutcome`](#eventoutcome)[]\>

One outcome per event, in order.

##### bookclerkJob()

```ts
bookclerkJob(
   context, 
   invocation, 
   granted
): Promise<JobOutcomeRecord>;
```

**`Internal`**

Adapter dispatch for the `job(controller)` trigger.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `invocation` | `Partial`\<[`JobInvocation`](#jobinvocation)\> | Durable command envelope. |
| `granted` | [`GrantedJobCapabilities`](#grantedjobcapabilities) | Granted input / output / progress / cancel stubs. |

###### Returns

`Promise`\<[`JobOutcomeRecord`](#joboutcomerecord)\>

Wire job outcome.

##### bookclerkQueue()

```ts
bookclerkQueue(context, wireBatch): Promise<QueueOutcome[]>;
```

**`Internal`**

Adapter dispatch for the `queue(batch)` trigger.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `wireBatch` | [`WireQueueBatch`](#wirequeuebatch) | Wire `QueueBatch`. |

###### Returns

`Promise`\<[`QueueOutcome`](#queueoutcome)[]\>

One outcome per message, in order.

##### bookclerkShutdown()

```ts
bookclerkShutdown(): Promise<void>;
```

**`Internal`**

Adapter dispatch for `shutdown()`.

###### Returns

`Promise`\<`void`\>

Resolves when the author hook has run.

##### databaseMigrations()?

```ts
optional databaseMigrations(binding): 
  | PluginMigration[]
| Promise<PluginMigration[]>;
```

Complete ordered plugin-owned migration sequence for one named binding.
The host calls this at binding initialization, before ordinary execute.

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `binding` | `string` |

###### Returns

  \| [`PluginMigration`](#pluginmigration)[]
  \| `Promise`\<[`PluginMigration`](#pluginmigration)[]\>

##### describe()?

```ts
optional describe(): 
  | Partial<PluginDescribe>
| Promise<Partial<PluginDescribe>>;
```

Optional identity refinement merged over the manifest projection.

###### Returns

  \| `Partial`\<`PluginDescribe`\>
  \| `Promise`\<`Partial`\<`PluginDescribe`\>\>

##### event()?

```ts
optional event(batch): void | Promise<void>;
```

`[[events.consumers]]` trigger: handle one delivered batch. Record an
outcome per message; untouched messages ack on return and retry on throw.

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `batch` | [`EventBatch`](#eventbatch) |

###### Returns

`void` \| `Promise`\<`void`\>

##### fetch()

```ts
fetch(_request?): Promise<Response>;
```

Rejects HTTP fetch — workerd guests are Workers-RPC only.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_request?` | `Request` | Incoming HTTP request when the entrypoint is fetch-facing. |

###### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

##### job()?

```ts
optional job(job): 
  | void
  | JobCompletion
| Promise<void | JobCompletion>;
```

`[triggers] jobs` trigger: run one durable job. Return to complete,
throw to reject, or call `job.suspend()` / `job.retryLater()`.

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `job` | [`JobController`](#jobcontroller) |

###### Returns

  \| `void`
  \| [`JobCompletion`](#jobcompletion)
  \| `Promise`\<`void` \| [`JobCompletion`](#jobcompletion)\>

##### queue()?

```ts
optional queue(batch): void | Promise<void>;
```

`[[queues.consumers]]` trigger: handle one delivered batch from one
queue. Record an outcome per message; untouched messages ack on return
and retry on throw.

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `batch` | [`QueueBatch`](#queuebatch) |

###### Returns

`void` \| `Promise`\<`void`\>

##### shutdown()?

```ts
optional shutdown(): void | Promise<void>;
```

Releases guest resources.

###### Returns

`void` \| `Promise`\<`void`\>

***

### CliEntrypoint

`cli` entrypoint (`PluginCli`): `describe()` → `CliSchema`,
`invoke(params)` → `CliInvokeResult`. Export it as
`export class Cli extends CliEntrypoint`.

#### Extends

- [`NamedEntrypoint`](#namedentrypoint)\<`Env`\>

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `Env` *extends* [`BookclerkEnv`](#bookclerkenv) | [`BookclerkEnv`](#bookclerkenv) |

#### Constructors

##### Constructor

```ts
new CliEntrypoint<Env>(ctx, env): CliEntrypoint<Env>;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `ExecutionContext` |
| `env` | `Env` |

###### Returns

[`CliEntrypoint`](#clientrypoint)\<`Env`\>

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-8)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-3)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-7)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-4)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[];
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-2)

#### Methods

##### bookclerkInvoke()

```ts
bookclerkInvoke(
   context, 
   method, 
   ...args
): Promise<unknown>;
```

**`Internal`**

Adapter dispatch: install granted bindings, then call `method`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `method` | `string` | Method name from the static allowlist. |
| ...`args` | `unknown`[] | Method arguments. |

###### Returns

`Promise`\<`unknown`\>

Method result.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-2)

##### describe()

```ts
describe(): Promise<CliSchema>;
```

Guest CLI schema.

###### Returns

`Promise`\<[`CliSchema`](#clischema)\>

Declared commands (`{ commands: [] }` when the guest has no CLI).

##### fetch()

```ts
fetch(_request?): Promise<Response>;
```

Rejects HTTP fetch — workerd guests are Workers-RPC only.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_request?` | `Request` | Incoming HTTP request when the entrypoint is fetch-facing. |

###### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-3)

##### invoke()

```ts
invoke(_params): Promise<CliInvokeResult>;
```

Invokes a guest CLI command.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`CliInvokeParams`](#cliinvokeparams) | Command name plus named argument values. |

###### Returns

`Promise`\<[`CliInvokeResult`](#cliinvokeresult)\>

Exit code, captured output, optional structured payload.

***

### DatabaseAdapterEntrypoint

`databaseAdapter` entrypoint: opens typed SQL sessions for the host
library. Export it as `export class DatabaseAdapter extends DatabaseAdapterEntrypoint`.

The adapter isolate drives sessions through two dispatch verbs on
[DatabaseAdapterEntrypoint.bookclerkInvoke](#bookclerkinvoke-1): `openSession` returns a
fresh session id, and `session(id, method, ...params)` calls `method` on
the retained [AdapterDatabaseSession](#adapterdatabasesession); `close` releases the entry.

#### Extends

- [`NamedEntrypoint`](#namedentrypoint)\<`Env`\>

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `Env` *extends* [`BookclerkEnv`](#bookclerkenv) | [`BookclerkEnv`](#bookclerkenv) |

#### Constructors

##### Constructor

```ts
new DatabaseAdapterEntrypoint<Env>(ctx, env): DatabaseAdapterEntrypoint<Env>;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `ExecutionContext` |
| `env` | `Env` |

###### Returns

[`DatabaseAdapterEntrypoint`](#databaseadapterentrypoint)\<`Env`\>

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-8)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-3)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-7)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-4)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[];
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-2)

#### Methods

##### bookclerkInvoke()

```ts
bookclerkInvoke(
   context, 
   method, 
   ...args
): Promise<unknown>;
```

**`Internal`**

Adapter dispatch: `openSession` → session id; `dropUnit`; `session` → call
on a retained session (`args` = `[id, method, ...params]`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `method` | `string` | `openSession`, `dropUnit`, or `session`. |
| ...`args` | `unknown`[] | Method arguments. |

###### Returns

`Promise`\<`unknown`\>

Session id for `openSession`, else the session method result.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-2)

##### dropUnit()

```ts
dropUnit(_unitRef): Promise<void>;
```

Physically drops one provisioned binding unit.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_unitRef` | `string` | Adapter-native unit identity. |

###### Returns

`Promise`\<`void`\>

Resolves when the unit is dropped.

##### fetch()

```ts
fetch(_request?): Promise<Response>;
```

Rejects HTTP fetch — workerd guests are Workers-RPC only.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_request?` | `Request` | Incoming HTTP request when the entrypoint is fetch-facing. |

###### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-3)

##### openSession()

```ts
openSession(): Promise<AdapterDatabaseSession>;
```

Opens a session. Sessions cannot survive suspension.

###### Returns

`Promise`\<[`AdapterDatabaseSession`](#adapterdatabasesession)\>

Adapter session capability.

***

### Destination

Object store capability: `job.output` and the `WORK_FS` binding. The
runtime stub *is* the capability; abort is stream cancel.

#### Extends

- `RpcTarget`

#### Constructors

##### Constructor

```ts
new Destination(): Destination;
```

###### Returns

[`Destination`](#destination)

###### Inherited from

```ts
RpcTarget.constructor
```

#### Methods

##### abortStage()

```ts
abortStage(_key, _commitToken): Promise<void>;
```

Abort a destination-side staged object.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |
| `_commitToken` | `string` | Staging token to discard. |

###### Returns

`Promise`\<`void`\>

Rejects with typed `unsupported` unless overridden.

##### commit()

```ts
commit(_key, _commitToken): Promise<PutResult>;
```

Finalize a destination-side staged object.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |
| `_commitToken` | `string` | Idempotency / commit token. |

###### Returns

`Promise`\<[`PutResult`](#putresult)\>

Published object metadata.

##### copy()

```ts
copy(_from, _to): Promise<CopyResult>;
```

Server-side copy when the backend supports it.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_from` | `string` | Source key. |
| `_to` | `string` | Destination key. |

###### Returns

`Promise`\<[`CopyResult`](#copyresult)\>

Bytes copied.

##### delete()

```ts
delete(_key): Promise<void>;
```

Delete a key (no-op if missing).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |

###### Returns

`Promise`\<`void`\>

Resolves when the delete is complete.

##### get()

```ts
get(_key, _options?): Promise<ReadResult>;
```

Streamed read. The body is a transferred stream, not a scalar.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |
| `_options?` | [`ReadOptions`](#readoptions) | Optional byte range. |

###### Returns

`Promise`\<[`ReadResult`](#readresult)\>

Metadata plus a transferred body stream.

##### head()

```ts
head(_key): Promise<ObjectMetadata | null>;
```

Metadata without a body; `null` when the key is missing.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |

###### Returns

`Promise`\<[`ObjectMetadata`](#objectmetadata) \| `null`\>

Metadata or `null` when the key is missing.

##### list()

```ts
list(_options): Promise<ListPage>;
```

One page of keys under `options.prefix`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_options` | [`ListOptions`](#listoptions) | Prefix, cursor, and limit. |

###### Returns

`Promise`\<[`ListPage`](#listpage)\>

One page of object keys.

##### put()

```ts
put(
   _key, 
   _body, 
   _options?
): Promise<PutResult>;
```

Streamed write. `body` ownership is transferred to the destination.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |
| `_body` | `ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\> | Byte stream. |
| `_options?` | [`WriteOptions`](#writeoptions) | Optional content type / length. |

###### Returns

`Promise`\<[`PutResult`](#putresult)\>

Bytes written and optional etag / sha256.

***

### EventBatch

One `event(batch)` delivery (`EventBatch` struct), Workers `MessageBatch`-shaped.

#### Constructors

##### Constructor

```ts
new EventBatch(events, invocation?): EventBatch;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `events` | readonly `Partial`\<[`DomainEvent`](#domainevent)\>[] \| `undefined` |
| `invocation?` | [`Invocation`](#invocation-11) |

###### Returns

[`EventBatch`](#eventbatch)

#### Properties

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of this delivery.

##### messages

```ts
readonly messages: readonly EventMessage[];
```

Messages in delivery order.

#### Accessors

##### type

###### Get Signature

```ts
get type(): string;
```

Event type shared by the batch.

###### Returns

`string`

The shared type, or `""` when mixed or empty.

#### Methods

##### ackAll()

```ts
ackAll(): void;
```

Ack every message.

###### Returns

`void`

##### retryAll()

```ts
retryAll(options?): void;
```

Retry every message.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`RetryOptions`](#retryoptions) | Redelivery timing and reason. |

###### Returns

`void`

***

### EventMessage

One delivered domain event. Exactly one outcome is recorded per message;
the first of `ack` / `retry` / `reject` / `deadLetter` / `suspend` wins and
later calls are ignored (as with Workers queue messages). Messages left
untouched are acked when `event()` resolves and retried when it throws.

#### Constructors

##### Constructor

```ts
new EventMessage(event): EventMessage;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `event` | `Partial`\<[`DomainEvent`](#domainevent)\> \| `undefined` |

###### Returns

[`EventMessage`](#eventmessage)

#### Properties

##### accountId

```ts
readonly accountId: string;
```

Account scope; empty for host-wide events.

##### attempts

```ts
readonly attempts: number;
```

Delivery counter, starting at 1.

##### body

```ts
readonly body: Uint8Array;
```

Encoded payload bytes.

##### causationId

```ts
readonly causationId: string;
```

Id of the command or event that caused this one.

##### checkpoint

```ts
readonly checkpoint: DeliveredCheckpoint | null;
```

Checkpoint from the prior `suspend()`, or `null`.

##### correlationId

```ts
readonly correlationId: string;
```

Trace correlation id.

##### deduplicationKey

```ts
readonly deduplicationKey: string;
```

Consumer-side idempotency key; stable across redeliveries.

##### id

```ts
readonly id: string;
```

Outbox event id.

##### invocationSequence

```ts
readonly invocationSequence: number;
```

Resume ordinal; distinct from `attempts`.

##### raw

```ts
readonly raw: Partial<DomainEvent>;
```

Raw wire envelope.

##### resumePending

```ts
readonly resumePending: boolean;
```

True when this delivery resumes a prior `suspend()`.

##### schemaVersion

```ts
readonly schemaVersion: number;
```

Schema version of [EventMessage.body](#body).

##### source

```ts
readonly source: string;
```

Producer plugin id; empty when unknown.

##### timestamp

```ts
readonly timestamp: Date;
```

When the producer observed the fact.

##### type

```ts
readonly type: string;
```

Event type (`book_acquired`, …).

#### Accessors

##### result

###### Get Signature

```ts
get result(): EventOutcome | null;
```

Recorded outcome.

###### Returns

[`EventOutcome`](#eventoutcome) \| `null`

The outcome, or `null` while undecided.

#### Methods

##### ack()

```ts
ack(): void;
```

Mark handled; the host marks the event delivered.

###### Returns

`void`

##### deadLetter()

```ts
deadLetter(reason?): void;
```

Park for operator review.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `reason?` | `string` | Short operator-facing reason. |

###### Returns

`void`

##### json()

```ts
json<T>(): T;
```

Decode [EventMessage.body](#body) as JSON.

###### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `T` | [`JsonObject`](#jsonobject) |

###### Returns

`T`

Parsed payload (`{}` for an empty body).

##### reject()

```ts
reject(reason?): void;
```

Stop delivering; the host records `reason`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `reason?` | `string` | Short operator-facing reason. |

###### Returns

`void`

##### retry()

```ts
retry(options?): void;
```

Redeliver later: at `retryAt`, after `delaySeconds`, or — when both are
omitted — whenever the host's backoff chooses.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`RetryOptions`](#retryoptions) | Redelivery timing and reason. |

###### Returns

`void`

##### suspend()

```ts
suspend(options?): void;
```

Release with a bounded checkpoint; the host redelivers at `wakeAt` or
when a `wakeOnEventType` event arrives.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`EventSuspendOptions`](#eventsuspendoptions) | Checkpoint and wake conditions. |

###### Returns

`void`

***

### JobController

Everything one job invocation may touch: the durable envelope, `input` /
`output` streams, `progress()`, `signal`, and the terminal-outcome recorders
`suspend()` / `retryLater()`. Returning normally without a recorded outcome
completes the job; throwing rejects it (`unavailable` /
`deadline_exceeded` errors are retryable, `cancelled` is cancelled).

#### Constructors

##### Constructor

```ts
new JobController(invocation, granted?): JobController;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `invocation` | `Partial`\<[`JobInvocation`](#jobinvocation)\> \| `undefined` |
| `granted?` | [`GrantedJobCapabilities`](#grantedjobcapabilities) |

###### Returns

[`JobController`](#jobcontroller)

#### Properties

##### attempt

```ts
readonly attempt: number;
```

Failure retry counter, starting at 1.

##### causationId

```ts
readonly causationId: string;
```

Id of the event or command that caused this job.

##### checkpoint

```ts
readonly checkpoint: DeliveredCheckpoint | null;
```

Checkpoint from the prior `suspend()`, or `null`.

##### correlationId

```ts
readonly correlationId: string;
```

Trace correlation id.

##### deadlineUnixMs

```ts
readonly deadlineUnixMs: number;
```

Deadline hint (Unix ms); the host fence is authoritative.

##### id

```ts
readonly id: string;
```

Unique id of this invocation attempt.

##### idempotencyKey

```ts
readonly idempotencyKey: string;
```

Caller idempotency key.

##### input

```ts
readonly input: Source | null;
```

Granted byte source, when the job has input.

##### invocation

```ts
readonly invocation: Readonly<Partial<JobInvocation>>;
```

Full durable envelope as delivered.

##### invocationSequence

```ts
readonly invocationSequence: number;
```

Resume ordinal.

##### output

```ts
readonly output: Destination | null;
```

Granted object store, when the job has output.

##### payloadJson

```ts
readonly payloadJson: string;
```

Command payload JSON text.

##### payloadSchemaVersion

```ts
readonly payloadSchemaVersion: number;
```

Schema version of [JobController.payloadJson](#payloadjson).

##### signal

```ts
readonly signal: AbortSignal;
```

Aborts when the host cancels the invocation.

##### stepId

```ts
readonly stepId: string;
```

Step id within a multi-step command; empty when single-step.

##### type

```ts
readonly type: string;
```

Command type the handler dispatches on.

#### Accessors

##### result

###### Get Signature

```ts
get result(): JobOutcomeRecord | null;
```

Recorded terminal outcome.

###### Returns

[`JobOutcomeRecord`](#joboutcomerecord) \| `null`

The outcome, or `null` while the job is still running.

#### Methods

##### cancel()

```ts
cancel(): void;
```

Host-side cancellation observed (adapter-internal).

###### Returns

`void`

##### json()

```ts
json<T>(): T;
```

Decode [JobController.payloadJson](#payloadjson).

###### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `T` | [`JsonObject`](#jsonobject) |

###### Returns

`T`

Parsed payload (`{}` when empty).

##### progress()

```ts
progress(percent, message?): Promise<void>;
```

Report `percent` in `0..=100` with an operator-facing `message`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `percent` | `number` | Completion percent. |
| `message?` | `string` | Operator-facing status. |

###### Returns

`Promise`\<`void`\>

Resolves when the host records progress.

##### retryLater()

```ts
retryLater(options?): void;
```

Give up this attempt and let the host retry at `retryAt` (or its default).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`JobRetryOptions`](#jobretryoptions) | Retry time and reason. |

###### Returns

`void`

##### suspend()

```ts
suspend(options?): void;
```

Release with a bounded checkpoint; the host resumes at `wakeAt`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`JobSuspendOptions`](#jobsuspendoptions) | Checkpoint and wake time. |

###### Returns

`void`

***

### NamedEntrypoint

Base for named entrypoints. Subclasses list their RPC surface on the static
`bookclerkMethods`; the adapter reaches them only through `bookclerkInvoke`,
which installs the granted bindings on `env` before dispatching.

#### Extends

- `WorkerEntrypoint`\<`Env`\>

#### Extended by

- [`CliEntrypoint`](#clientrypoint)
- [`DatabaseAdapterEntrypoint`](#databaseadapterentrypoint)
- [`OidcEntrypoint`](#oidcentrypoint)
- [`RemoteLibraryEntrypoint`](#remotelibraryentrypoint)
- [`StorageEntrypoint`](#storageentrypoint)
- [`StorefrontEntrypoint`](#storefrontentrypoint)

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `Env` *extends* [`BookclerkEnv`](#bookclerkenv) | [`BookclerkEnv`](#bookclerkenv) |

#### Constructors

##### Constructor

```ts
new NamedEntrypoint<Env>(ctx, env): NamedEntrypoint<Env>;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `ExecutionContext` |
| `env` | `Env` |

###### Returns

[`NamedEntrypoint`](#namedentrypoint)\<`Env`\>

###### Inherited from

```ts
WorkerEntrypoint<Env>.constructor
```

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

```ts
WorkerEntrypoint.ctx
```

##### env

```ts
readonly env: Env;
```

###### Inherited from

```ts
WorkerEntrypoint.env
```

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[] = [];
```

Methods the adapter may dispatch on this entrypoint.

#### Methods

##### bookclerkInvoke()

```ts
bookclerkInvoke(
   context, 
   method, 
   ...args
): Promise<unknown>;
```

**`Internal`**

Adapter dispatch: install granted bindings, then call `method`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `method` | `string` | Method name from the static allowlist. |
| ...`args` | `unknown`[] | Method arguments. |

###### Returns

`Promise`\<`unknown`\>

Method result.

##### fetch()

```ts
fetch(_request?): Promise<Response>;
```

Rejects HTTP fetch — workerd guests are Workers-RPC only.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_request?` | `Request` | Incoming HTTP request when the entrypoint is fetch-facing. |

###### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

***

### OidcEntrypoint

`oidc` entrypoint: relying-party client templates and credential
verification. Export it as `export class Oidc extends OidcEntrypoint`.

#### Extends

- [`NamedEntrypoint`](#namedentrypoint)\<`Env`\>

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `Env` *extends* [`BookclerkEnv`](#bookclerkenv) | [`BookclerkEnv`](#bookclerkenv) |

#### Constructors

##### Constructor

```ts
new OidcEntrypoint<Env>(ctx, env): OidcEntrypoint<Env>;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `ExecutionContext` |
| `env` | `Env` |

###### Returns

[`OidcEntrypoint`](#oidcentrypoint)\<`Env`\>

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-8)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-3)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-7)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-4)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[];
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-2)

#### Methods

##### authenticateUser()

```ts
authenticateUser(_params): Promise<ExternalUser>;
```

Verifies remote credentials on behalf of the host.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`AuthenticateUserParams`](#authenticateuserparams) | Username / password pair. |

###### Returns

`Promise`\<[`ExternalUser`](#externaluser)\>

External user identity.

##### bookclerkInvoke()

```ts
bookclerkInvoke(
   context, 
   method, 
   ...args
): Promise<unknown>;
```

**`Internal`**

Adapter dispatch: install granted bindings, then call `method`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `method` | `string` | Method name from the static allowlist. |
| ...`args` | `unknown`[] | Method arguments. |

###### Returns

`Promise`\<`unknown`\>

Method result.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-2)

##### clients()

```ts
clients(): Promise<OidcClientTemplate[]>;
```

Plugin-provided OIDC authorization-server client templates. The host
materializes `oidc_clients` rows; plugins never mint tokens.

###### Returns

`Promise`\<[`OidcClientTemplate`](#oidcclienttemplate)[]\>

Templates (`[]` when unused).

##### fetch()

```ts
fetch(_request?): Promise<Response>;
```

Rejects HTTP fetch — workerd guests are Workers-RPC only.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_request?` | `Request` | Incoming HTTP request when the entrypoint is fetch-facing. |

###### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-3)

***

### PluginError

Thrown by the SDK when a wire union carries `err`. Unknown codes are kept.

#### Extends

- `Error`

#### Extended by

- [`InvokeError`](workerd.md#invokeerror)

#### Constructors

##### Constructor

```ts
new PluginError(code, message): PluginError;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `code` | `string` |
| `message` | `string` |

###### Returns

[`PluginError`](#pluginerror)

###### Overrides

```ts
Error.constructor
```

#### Properties

##### code

```ts
readonly code: string;
```

Known `PluginErrorCode` wire string, or `unknown`.

##### wireCode

```ts
readonly wireCode: string;
```

Raw wire code, including codes this SDK does not know.

#### Methods

##### fromWire()

```ts
static fromWire(code, message): PluginError;
```

Construct a [PluginError](#pluginerror) from a Cap'n Proto / JSON wire code.

Unknown codes become `unknown` on [PluginError.code](#code) while
[PluginError.wireCode](#wirecode) keeps the raw value.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `code` | `string` | Wire error code (known or unknown). |
| `message` | `string` | Operator-facing error text. |

###### Returns

[`PluginError`](#pluginerror)

Typed plugin error.

***

### ProgressSink

Progress reports for a job invocation (never carries media).

#### Extends

- `RpcTarget`

#### Constructors

##### Constructor

```ts
new ProgressSink(): ProgressSink;
```

###### Returns

[`ProgressSink`](#progresssink)

###### Inherited from

```ts
RpcTarget.constructor
```

#### Methods

##### report()

```ts
report(_percent, _message): Promise<void>;
```

Reports `percent` in `0..=100` and an operator-facing `message`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_percent` | `number` | Completion percent. |
| `_message` | `string` | Operator-facing status. |

###### Returns

`Promise`\<`void`\>

Resolves when the host records progress.

***

### QueueBatch

One `queue(batch)` delivery (`QueueBatch` struct), Workers `MessageBatch`-shaped.

#### Constructors

##### Constructor

```ts
new QueueBatch(
   queue, 
   messages, 
   invocation?
): QueueBatch;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `queue` | `string` \| `undefined` |
| `messages` | readonly `Partial`\<`QueueMessage`\>[] \| `undefined` |
| `invocation?` | [`Invocation`](#invocation-11) |

###### Returns

[`QueueBatch`](#queuebatch)

#### Properties

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of this delivery.

##### messages

```ts
readonly messages: readonly QueueMessage[];
```

Messages in delivery order.

##### queue

```ts
readonly queue: string;
```

Queue the messages were sent to.

#### Methods

##### ackAll()

```ts
ackAll(): void;
```

Ack every message.

###### Returns

`void`

##### retryAll()

```ts
retryAll(options?): void;
```

Retry every message.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`QueueRetryOptions`](#queueretryoptions) | Redelivery timing and reason. |

###### Returns

`void`

***

### QueueMessage

One delivered queue message (`QueueMessage` struct), Workers `Message`-shaped.

Exactly one outcome is recorded per message; the first of `ack` / `retry`
wins and later calls are ignored. Messages left untouched are acked when
`queue()` resolves and retried when it throws.

#### Constructors

##### Constructor

```ts
new QueueMessage(message): QueueMessage;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `message` | `Partial`\<`QueueMessage`\> \| `undefined` |

###### Returns

[`QueueMessage`](#queuemessage)

#### Properties

##### attempts

```ts
readonly attempts: number;
```

Delivery counter, starting at 1.

##### body

```ts
readonly body: Uint8Array;
```

Encoded body bytes.

##### contentType

```ts
readonly contentType: string;
```

Body encoding: `json`, `text`, or `bytes`.

##### id

```ts
readonly id: string;
```

Outbox row id; stable across redeliveries.

##### raw

```ts
readonly raw: Partial<WireQueueMessage>;
```

Raw wire envelope.

##### source

```ts
readonly source: string;
```

Producer plugin id; empty when the host sent it.

##### timestamp

```ts
readonly timestamp: Date;
```

When the producer sent the message.

#### Accessors

##### result

###### Get Signature

```ts
get result(): QueueOutcome | null;
```

Recorded outcome.

###### Returns

[`QueueOutcome`](#queueoutcome) \| `null`

The outcome, or `null` while undecided.

#### Methods

##### ack()

```ts
ack(): void;
```

Mark handled; the host marks the message delivered.

###### Returns

`void`

##### json()

```ts
json<T>(): T;
```

Decode [QueueMessage.body](#body-1) as JSON.

###### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `T` | `unknown` |

###### Returns

`T`

Parsed body (`null` for an empty body).

##### retry()

```ts
retry(options?): void;
```

Redeliver later: after `delaySeconds`, or — when omitted — after the
consumer's `retry_delay`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`QueueRetryOptions`](#queueretryoptions) | Redelivery timing and reason. |

###### Returns

`void`

##### text()

```ts
text(): string;
```

Decode [QueueMessage.body](#body-1) as UTF-8 text.

###### Returns

`string`

The body text.

***

### RemoteLibraryEntrypoint

`remoteLibrary` entrypoint: lifecycle, rescan, listening sync, and user
polling for a remote library server. Export it as
`export class RemoteLibrary extends RemoteLibraryEntrypoint`.

#### Extends

- [`NamedEntrypoint`](#namedentrypoint)\<`Env`\>

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `Env` *extends* [`BookclerkEnv`](#bookclerkenv) | [`BookclerkEnv`](#bookclerkenv) |

#### Constructors

##### Constructor

```ts
new RemoteLibraryEntrypoint<Env>(ctx, env): RemoteLibraryEntrypoint<Env>;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `ExecutionContext` |
| `env` | `Env` |

###### Returns

[`RemoteLibraryEntrypoint`](#remotelibraryentrypoint)\<`Env`\>

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-8)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-3)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-7)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-4)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[] = REMOTE_LIBRARY_METHODS;
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-2)

#### Methods

##### bookclerkInvoke()

```ts
bookclerkInvoke(
   context, 
   method, 
   ...args
): Promise<unknown>;
```

**`Internal`**

Adapter dispatch: install granted bindings, then call `method`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `method` | `string` | Method name from the static allowlist. |
| ...`args` | `unknown`[] | Method arguments. |

###### Returns

`Promise`\<`unknown`\>

Method result.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-2)

##### diagnose()

```ts
diagnose(): Promise<string[]>;
```

Operator-facing diagnostic lines.

###### Returns

`Promise`\<`string`[]\>

Probe lines.

##### fetch()

```ts
fetch(_request?): Promise<Response>;
```

Rejects HTTP fetch — workerd guests are Workers-RPC only.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_request?` | `Request` | Incoming HTTP request when the entrypoint is fetch-facing. |

###### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-3)

##### health()

```ts
health(): Promise<HealthOk>;
```

Reports whether the remote library session is usable.

###### Returns

`Promise`\<[`HealthOk`](#healthok)\>

Health flag plus detail.

##### pollEvents()

```ts
pollEvents(): Promise<ExternalUser[]>;
```

Drains external users observed since the last poll.

###### Returns

`Promise`\<[`ExternalUser`](#externaluser)[]\>

Newly observed users.

##### scanLibrary()

```ts
scanLibrary(_params): Promise<void>;
```

Re-syncs the remote library.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`ScanLibraryParams`](#scanlibraryparams) | Scan scope. |

###### Returns

`Promise`\<`void`\>

Resolves when the scan has been accepted.

##### start()

```ts
start(): Promise<void>;
```

Starts long-running work for this invocation.

###### Returns

`Promise`\<`void`\>

Resolves when running.

##### stop()

```ts
stop(): Promise<void>;
```

Stops long-running work for this invocation.

###### Returns

`Promise`\<`void`\>

Resolves when stopped.

##### syncListening()

```ts
syncListening(): Promise<ListeningProgress[]>;
```

Push / pull listening progress; the host upserts the rows.

###### Returns

`Promise`\<[`ListeningProgress`](#listeningprogress)[]\>

Progress rows.

***

### Source

Byte source capability: `job.input`.

#### Extends

- `RpcTarget`

#### Constructors

##### Constructor

```ts
new Source(): Source;
```

###### Returns

[`Source`](#source-2)

###### Inherited from

```ts
RpcTarget.constructor
```

#### Methods

##### open()

```ts
open(_key): Promise<ReadResult>;
```

Opens `key` for streamed reading.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |

###### Returns

`Promise`\<[`ReadResult`](#readresult)\>

Metadata plus a transferred body stream.

***

### StorageEntrypoint

`storage` entrypoint (`Destination` interface): an object store the host
writes acquired media into. Export it as
`export class Storage extends StorageEntrypoint`.

#### Extends

- [`NamedEntrypoint`](#namedentrypoint)\<`Env`\>

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `Env` *extends* [`BookclerkEnv`](#bookclerkenv) | [`BookclerkEnv`](#bookclerkenv) |

#### Constructors

##### Constructor

```ts
new StorageEntrypoint<Env>(ctx, env): StorageEntrypoint<Env>;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `ExecutionContext` |
| `env` | `Env` |

###### Returns

[`StorageEntrypoint`](#storageentrypoint)\<`Env`\>

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-8)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-3)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-7)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-4)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[] = STORAGE_METHODS;
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-2)

#### Methods

##### abortStage()

```ts
abortStage(_key, _commitToken): Promise<void>;
```

Abort a destination-side staged object.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |
| `_commitToken` | `string` | Staging token to discard. |

###### Returns

`Promise`\<`void`\>

Rejects with typed `unsupported` unless overridden.

##### bookclerkInvoke()

```ts
bookclerkInvoke(
   context, 
   method, 
   ...args
): Promise<unknown>;
```

**`Internal`**

Adapter dispatch: install granted bindings, then call `method`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `method` | `string` | Method name from the static allowlist. |
| ...`args` | `unknown`[] | Method arguments. |

###### Returns

`Promise`\<`unknown`\>

Method result.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-2)

##### commit()

```ts
commit(_key, _commitToken): Promise<PutResult>;
```

Finalize a destination-side staged object.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |
| `_commitToken` | `string` | Idempotency / commit token. |

###### Returns

`Promise`\<[`PutResult`](#putresult)\>

Published object metadata.

##### copy()

```ts
copy(_from, _to): Promise<CopyResult>;
```

Server-side copy when the backend supports it.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_from` | `string` | Source key. |
| `_to` | `string` | Destination key. |

###### Returns

`Promise`\<[`CopyResult`](#copyresult)\>

Bytes copied.

##### delete()

```ts
delete(_key): Promise<void>;
```

Delete a key (no-op if missing).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |

###### Returns

`Promise`\<`void`\>

Resolves when the delete is complete.

##### fetch()

```ts
fetch(_request?): Promise<Response>;
```

Rejects HTTP fetch — workerd guests are Workers-RPC only.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_request?` | `Request` | Incoming HTTP request when the entrypoint is fetch-facing. |

###### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-3)

##### get()

```ts
get(_key, _options?): Promise<ReadResult>;
```

Streamed read. The body is a transferred stream, not a scalar.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |
| `_options?` | [`ReadOptions`](#readoptions) | Optional byte range. |

###### Returns

`Promise`\<[`ReadResult`](#readresult)\>

Metadata plus a transferred body stream.

##### head()

```ts
head(_key): Promise<ObjectMetadata | null>;
```

Metadata without a body; `null` when the key is missing.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |

###### Returns

`Promise`\<[`ObjectMetadata`](#objectmetadata) \| `null`\>

Metadata or `null` when the key is missing.

##### list()

```ts
list(_options): Promise<ListPage>;
```

One page of keys under `options.prefix`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_options` | [`ListOptions`](#listoptions) | Prefix, cursor, and limit. |

###### Returns

`Promise`\<[`ListPage`](#listpage)\>

One page of object keys.

##### put()

```ts
put(
   _key, 
   _body, 
   _options?
): Promise<PutResult>;
```

Streamed write. `body` ownership is transferred to the destination.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_key` | `string` | Object key. |
| `_body` | `ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\> | Byte stream. |
| `_options?` | [`WriteOptions`](#writeoptions) | Optional content type / length. |

###### Returns

`Promise`\<[`PutResult`](#putresult)\>

Bytes written and optional etag / sha256.

***

### StorefrontEntrypoint

`storefront` entrypoint (`ContentSource` interface). Export it as
`export class Storefront extends StorefrontEntrypoint`. Every method takes
and returns the typed ABI structs generated from `schema/plugin.capnp`.

#### Extends

- [`NamedEntrypoint`](#namedentrypoint)\<`Env`\>

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `Env` *extends* [`BookclerkEnv`](#bookclerkenv) | [`BookclerkEnv`](#bookclerkenv) |

#### Constructors

##### Constructor

```ts
new StorefrontEntrypoint<Env>(ctx, env): StorefrontEntrypoint<Env>;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `ExecutionContext` |
| `env` | `Env` |

###### Returns

[`StorefrontEntrypoint`](#storefrontentrypoint)\<`Env`\>

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-8)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-3)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-7)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-4)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[] = STOREFRONT_METHODS;
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-2)

#### Methods

##### bookclerkInvoke()

```ts
bookclerkInvoke(
   context, 
   method, 
   ...args
): Promise<unknown>;
```

**`Internal`**

Adapter dispatch: install granted bindings, then call `method`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `method` | `string` | Method name from the static allowlist. |
| ...`args` | `unknown`[] | Method arguments. |

###### Returns

`Promise`\<`unknown`\>

Method result.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-2)

##### catalogDetail()

```ts
catalogDetail(_params): Promise<CatalogHit | null>;
```

Full catalog record for one product; `null` when unknown.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`CatalogDetailParams`](#catalogdetailparams) | Product identifiers. |

###### Returns

`Promise`\<[`CatalogHit`](#cataloghit) \| `null`\>

Catalog hit or `null`.

##### diagnose()

```ts
diagnose(): Promise<string[]>;
```

Operator-facing diagnostic lines.

###### Returns

`Promise`\<`string`[]\>

Probe lines.

##### expandCandidates()

```ts
expandCandidates(_params): Promise<CatalogHit[]>;
```

Related-title expansion from a seed title.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`ExpandCandidatesParams`](#expandcandidatesparams) | Seed title plus limits. |

###### Returns

`Promise`\<[`CatalogHit`](#cataloghit)[]\>

Candidate hits.

##### fetch()

```ts
fetch(_request?): Promise<Response>;
```

Rejects HTTP fetch — workerd guests are Workers-RPC only.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_request?` | `Request` | Incoming HTTP request when the entrypoint is fetch-facing. |

###### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-3)

##### fetchTitle()

```ts
fetchTitle(_params): Promise<PlainFetch>;
```

Fetches one title into `params.cacheDir` and returns plain media paths.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`FetchTitleParams`](#fetchtitleparams) | Title identifiers, credentials, fetch options. |

###### Returns

`Promise`\<[`PlainFetch`](#plainfetch)\>

Plain media parts plus metadata.

##### health()

```ts
health(): Promise<HealthOk>;
```

Reports whether the storefront session is usable.

###### Returns

`Promise`\<[`HealthOk`](#healthok)\>

Health flag plus detail.

##### listAccounts()

```ts
listAccounts(): Promise<SourceAccount[]>;
```

Accounts the guest knows about.

###### Returns

`Promise`\<[`SourceAccount`](#sourceaccount)[]\>

Account list.

##### listDeals()

```ts
listDeals(_params): Promise<CatalogHit[]>;
```

Current storefront deals.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`ListDealsParams`](#listdealsparams) | Optional filters. |

###### Returns

`Promise`\<[`CatalogHit`](#cataloghit)[]\>

Deal hits.

##### login()

```ts
login(_params): Promise<LoginResult>;
```

Password or one-shot OAuth login. The host seals
`LoginResult.credentials` into `encrypted_secrets`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`LoginParams`](#loginparams) | Login parameters. |

###### Returns

`Promise`\<[`LoginResult`](#loginresult)\>

Account identity plus opaque credentials.

##### loginComplete()

```ts
loginComplete(_params): Promise<LoginResult>;
```

Finishes a login started by [StorefrontEntrypoint.loginStart](#loginstart).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`LoginCompleteParams`](#logincompleteparams) | Continuation plus user input. |

###### Returns

`Promise`\<[`LoginResult`](#loginresult)\>

Account identity plus opaque credentials.

##### loginStart()

```ts
loginStart(_params): Promise<LoginStartResult>;
```

Begins an interactive OAuth login.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`LoginParams`](#loginparams) | Login parameters. |

###### Returns

`Promise`\<[`LoginStartResult`](#loginstartresult)\>

Continuation for [StorefrontEntrypoint.loginComplete](#logincomplete).

##### purchaseHint()

```ts
purchaseHint(_params): Promise<PurchaseHint | null>;
```

Purchase link / price hint; `null` when the title is unknown.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`PurchaseHintParams`](#purchasehintparams) | Title identifiers. |

###### Returns

`Promise`\<[`PurchaseHint`](#purchasehint-2) \| `null`\>

Hint or `null`.

##### scan()

```ts
scan(_params): Promise<ScanSummary>;
```

Library scan; the host upserts `ScanSummary.books`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`ScanParams`](#scanparams) | Scan parameters (accounts + credentials). |

###### Returns

`Promise`\<[`ScanSummary`](#scansummary)\>

Books discovered plus per-account counters.

##### searchCatalog()

```ts
searchCatalog(_params): Promise<CatalogHit[]>;
```

Free-text storefront catalog search.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`SearchCatalogParams`](#searchcatalogparams) | Query parameters. |

###### Returns

`Promise`\<[`CatalogHit`](#cataloghit)[]\>

Catalog hits.

## Interfaces

### AccountCredential

Host-sealed credentials for one account, delivered on `scan`.

#### Properties

##### accountId

```ts
accountId: string;
```

Account id the blob belongs to.

##### credentials

```ts
credentials: Uint8Array;
```

Opaque credential bytes exactly as the guest returned them at login.

***

### AdapterEnv

Trusted adapter env. Authors never see this type on their class.

#### Properties

##### BRIDGE\_TOKEN?

```ts
optional BRIDGE_TOKEN?: string;
```

Isolate-to-host bridge bearer.

##### GRANTED?

```ts
optional GRANTED?: GrantedFetcher;
```

Per-invocation grant reverse channel.

##### PLUGIN?

```ts
optional PLUGIN?: AuthorStub;
```

Author default entrypoint.

##### PLUGIN\_CLI?

```ts
optional PLUGIN_CLI?: NamedStub;
```

##### PLUGIN\_DATABASE\_ADAPTER?

```ts
optional PLUGIN_DATABASE_ADAPTER?: NamedStub;
```

##### PLUGIN\_DESCRIBE?

```ts
optional PLUGIN_DESCRIBE?: string | PluginDescribe;
```

Manifest `PluginDescribe` projection (JSON binding).

##### PLUGIN\_OIDC?

```ts
optional PLUGIN_OIDC?: NamedStub;
```

##### PLUGIN\_REMOTE\_LIBRARY?

```ts
optional PLUGIN_REMOTE_LIBRARY?: NamedStub;
```

##### PLUGIN\_STORAGE?

```ts
optional PLUGIN_STORAGE?: NamedStub;
```

##### PLUGIN\_STOREFRONT?

```ts
optional PLUGIN_STOREFRONT?: NamedStub;
```

Named author entrypoints (`PLUGIN_STOREFRONT`, `PLUGIN_STORAGE`, …).

***

### AdapterExecuteRequest

Host -> adapter batch: proven statements plus isolation and receipt.

#### Properties

##### deadlineUnixMs

```ts
deadlineUnixMs: number;
```

Deadline hint; `0` means none.

##### isolation

```ts
isolation: "atomicBatch" | "nestedSavepoint" | "consistentSnapshot";
```

Transaction isolation the adapter must realize.

##### operationId

```ts
operationId: string;
```

Host-chosen idempotency key.

##### receipt

```ts
receipt: AdapterReceipt;
```

Replay receipt to persist; zero/empty when not required.

##### requestHash

```ts
requestHash: string;
```

SHA-256 hex of the idempotency-relevant request.

##### statements

```ts
statements: AdapterStatement[];
```

Statements in execution order.

***

### AdapterReceipt

Replay receipt the host asks the adapter to persist with the batch.

#### Properties

##### guestHash

```ts
guestHash: string;
```

Guest `requestHash` compared on replay.

##### guestLen

```ts
guestLen: number;
```

Guest statement count inside the receipt wrap (excluding host prune/select).

***

### AdapterStatement

One host-lowered statement with its proof.

#### Properties

##### kind

```ts
kind: "execute" | "select" | "returning";
```

Statement classification.

##### maxRows

```ts
maxRows: number;
```

Row cap for queries; `0` means adapter default.

##### parameters

```ts
parameters: DbValue[];
```

Positional bind values.

##### proof

```ts
proof: ResolvedStatement;
```

Typed proof bound to `sql`.

##### resultSelection

```ts
resultSelection: "discard" | "affectedRows" | "rows";
```

Which outcome parts to return.

##### sql

```ts
sql: string;
```

Canonical SQL text.

***

### AtomicTransport

Host `GuestDatabase.execute` projection.

#### Extended by

- [`GrantedDatabaseTransport`](#granteddatabasetransport)

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

### AuthenticateUserParams

Params of `Oidc.authenticateUser`.

#### Properties

##### password

```ts
password: string;
```

Integration password; never logged by the host.

##### username

```ts
username: string;
```

Integration username / login id.

***

### Bindings

Host-granted bindings for one `PluginWorker.open` (`env` in the Workers
idiom). OS paths, FDs, and sockets are transport-private. Capability fields
are null when the manifest does not declare (or the operator did not grant)
the binding.

#### Properties

##### adapter

```ts
adapter: DatabaseAdapterConfig;
```

Database-adapter bootstrap for the `databaseAdapter` entrypoint. First-party
host-managed adapters receive host-private connect params in `config`;
third-party adapters receive this typed bootstrap (and an empty `config`).

##### cancel?

```ts
optional cancel?: Cancellation | null;
```

Host cancellation for the whole invocation (fence / lease loss).
Null when the guest does not export this capability.

##### config

```ts
config: ExtensibleConfig;
```

`CONFIG`: granted plugin settings (operator `[vars]` plus the family
settings table) as `application/json`; at most `maxConfigPayloadBytes`.

##### databases

```ts
databases: NamedDatabase[];
```

Named plugin-owned `[[databases]]` bindings: each entry is an isolated
database provisioned by the active adapter, separate from the Bookclerk
library and from every other plugin. Empty when the manifest declares none.

##### events?

```ts
optional events?: EventPublisher | null;
```

`EVENTS`: outbox publisher; null unless `[[events.producers]]` is granted.
Null when the guest does not export this capability.

##### kv

```ts
kv: NamedKv[];
```

Named `[[kv_namespaces]]` bindings: one host-served durable namespace per
binding, scoped to this plugin. Empty when the manifest declares none.

##### queues

```ts
queues: NamedQueue[];
```

Named `[[queues.producers]]` bindings: one host-served producer per
binding, each sending into the queue the row names. Empty when the
manifest declares none.

##### secrets

```ts
secrets: ExtensibleConfig;
```

`SECRETS`: granted secret values as `application/json`; empty payload
when the manifest declares no `[secrets]`.

##### storage?

```ts
optional storage?: Destination | null;
```

`WORK_FS`: host-granted object storage for durable plugin files (work
filesystem); null unless `[work_fs]` is granted. Job input/output travel on
`JobRunner.job(controller)`, never here.
Null when the guest does not export this capability.

***

### BookclerkEnv

Guest `env` bindings for `api_version = 3` manifests. Every host-provided
facility is a binding here; there is no `HOST` object. Bindings appear only
when `plugin.toml` declares them and the operator has consented.

Run `bookclerk-plugin types` to generate a precise `Env` for one manifest
(`bookclerk-configuration.d.ts`), then `extends BookclerkEntrypoint<Env>`.

#### Indexable

```ts
[binding: string]: unknown
```

Named `[[databases]]` bindings, `[[queues.producers]]` bindings, and any
renamed `binding = "..."` entries.

#### Properties

##### CONFIG?

```ts
optional CONFIG?: JsonObject;
```

`[vars]` plus operator settings, decoded from the granted `CONFIG` payload.

##### EVENTS?

```ts
optional EVENTS?: EventPublisherBinding;
```

`[[events.producers]]` outbox publisher.

##### KV?

```ts
optional KV?: KvNamespaceBinding;
```

`[[kv_namespaces]]` durable KV binding (default name).

##### OAUTH?

```ts
optional OAUTH?: unknown;
```

`[oauth]` loopback helper; surface reserved.

##### SECRETS?

```ts
optional SECRETS?: JsonObject;
```

`[secrets]` values, decoded from the granted `SECRETS` payload.

##### WORK\_FS?

```ts
optional WORK_FS?: StorageBinding;
```

`[work_fs]` object storage.

***

### Brand

Portal brand crossing the RPC boundary. Distinct from `plugin.toml`
`logo`: `iconUrl` is the live URL or data URI the SPA renders.

#### Properties

##### accent

```ts
accent: string;
```

Accent CSS color for highlights / CTAs.

##### bg

```ts
bg: string;
```

Background CSS color (hex or named).

##### fg

```ts
fg: string;
```

Foreground CSS color for text on `bg`.

##### iconUrl?

```ts
optional iconUrl?: string;
```

Icon URL or data URI for the portal.
Omitted when absent (wire zero value).

##### id

```ts
id: string;
```

Brand id (often matches the plugin id); empty means "no brand".

##### name

```ts
name: string;
```

Display name shown next to the brand swatch.

***

### ByteRange

Half-open byte window `[offset, offset + length)`.

#### Properties

##### length

```ts
length: number;
```

Number of bytes; `0` reads to the end.

##### offset

```ts
offset: number;
```

First byte offset.

***

### ByteSource

Transferred readable byte stream. The capability *is* the stream; callers
pull bounded windows. Abort is capability drop / RPC cancel. A failed pull
MUST set `err` — never a successful empty EOF.

#### Methods

##### pull()

```ts
pull(maxBytes): Promise<PullReply>;
```

Pull the next window of bytes. `done = true` on the final chunk.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `maxBytes` | `number` | Upper bound for this window; at most `maxStreamWindowBytes`. |

###### Returns

`Promise`\<[`PullReply`](#pullreply)\>

[PullReply](#pullreply)

***

### Cancellation

Transport cancellation. SDKs project this into a locally created AbortSignal
(AbortSignal is not a serializable Workers RPC value).

#### Methods

##### poll()

```ts
poll(): Promise<boolean>;
```

Non-blocking check; `true` once the host has fenced the invocation.

###### Returns

`Promise`\<`boolean`\>

`cancelled` (boolean)

***

### CancelledOutcome

Job observed cancellation and stopped.

#### Properties

##### message

```ts
message: string;
```

Short human note.

***

### CancelWatchLike

Host cancellation watch delivered to a job controller (adapter-internal).

#### Methods

##### wait()

```ts
wait(): Promise<void>;
```

Resolves once the host cancels the invocation.

###### Returns

`Promise`\<`void`\>

***

### CatalogDetail

Success payload of `ContentSource.catalogDetail`.

#### Properties

##### found

```ts
found: boolean;
```

False when the product is unknown to the storefront (`hit` is empty).

##### hit

```ts
hit: CatalogHit;
```

Full catalog record when `found`.

***

### CatalogDetailParams

Params of `ContentSource.catalogDetail`.

#### Properties

##### isbn?

```ts
optional isbn?: string;
```

ISBN when it differs from `productId`.
Omitted when absent (wire zero value).

##### productId

```ts
productId: string;
```

Store product id (Libro ISBN or ISBN-slug).

***

### CatalogHit

One catalog / candidate hit returned by `searchCatalog`,
`expandCandidates`, `listDeals`, and `catalogDetail`.

#### Properties

##### abridgement

```ts
abridgement: "unknown" | "unabridged" | "abridged";
```

Whether the edition is abridged when the storefront says so.

##### asin?

```ts
optional asin?: string;
```

Amazon ASIN when known.
Omitted when absent (wire zero value).

##### authors?

```ts
optional authors?: string;
```

Authors string when known.
Omitted when absent (wire zero value).

##### categories?

```ts
optional categories?: string;
```

Category / genre labels as a single string when known.
Omitted when absent (wire zero value).

##### coverUrl?

```ts
optional coverUrl?: string;
```

Cover image URL.
Omitted when absent (wire zero value).

##### currency?

```ts
optional currency?: string;
```

ISO currency code for `priceCents`.
Omitted when absent (wire zero value).

##### description?

```ts
optional description?: string;
```

Long description / blurb when fetched.
Omitted when absent (wire zero value).

##### isbn?

```ts
optional isbn?: string;
```

ISBN when known.
Omitted when absent (wire zero value).

##### language?

```ts
optional language?: string;
```

Content language code when known.
Omitted when absent (wire zero value).

##### lengthMinutes?

```ts
optional lengthMinutes?: bigint;
```

Runtime in whole minutes.
Omitted when absent (wire zero value).

##### narrators?

```ts
optional narrators?: string;
```

Narrators string when known.
Omitted when absent (wire zero value).

##### origin

```ts
origin: string;
```

Hit origin label (plugin id or storefront name).

##### priceCents?

```ts
optional priceCents?: bigint;
```

Current price in minor units (cents).
Omitted when absent (wire zero value).

##### priceLabel?

```ts
optional priceLabel?: string;
```

Pre-formatted price for display.
Omitted when absent (wire zero value).

##### productId

```ts
productId: string;
```

Storefront product / SKU id.

##### publishedAt?

```ts
optional publishedAt?: string;
```

Publication date string as provided by the storefront.
Omitted when absent (wire zero value).

##### publisher?

```ts
optional publisher?: string;
```

Publisher name when known.
Omitted when absent (wire zero value).

##### ratingCount?

```ts
optional ratingCount?: bigint;
```

Number of ratings when known.
Omitted when absent (wire zero value).

##### ratingOverall?

```ts
optional ratingOverall?: number;
```

Aggregate rating when known.
Omitted when absent (wire zero value).

##### series?

```ts
optional series?: string;
```

Series name when applicable.
Omitted when absent (wire zero value).

##### seriesIndex?

```ts
optional seriesIndex?: string;
```

Series index / sequence label.
Omitted when absent (wire zero value).

##### subtitle?

```ts
optional subtitle?: string;
```

Subtitle when distinct from `title`.
Omitted when absent (wire zero value).

##### title

```ts
title: string;
```

Primary title.

##### url?

```ts
optional url?: string;
```

Storefront product page URL.
Omitted when absent (wire zero value).

***

### CatalogHits

Success payload of the catalog list methods.

#### Properties

##### hits

```ts
hits: CatalogHit[];
```

Hits in storefront order.

***

### ChapterMarker

One chapter marker of a fetched title.

#### Properties

##### startMs

```ts
startMs: number;
```

Chapter start offset in milliseconds from the beginning of the title.

##### title

```ts
title: string;
```

Chapter title.

***

### CliArg

One named argument value passed to `PluginCli.invoke`.

#### Properties

##### name

```ts
name: string;
```

Arg name matching a `CliArgSpec.name`.

##### value

```ts
value: string;
```

String form of the value (the guest parses per `CliArgSpec.kind`).

***

### CliArgSpec

One CLI argument or flag under a `CliCommandSpec`.

#### Properties

##### about?

```ts
optional about?: string;
```

Help text for this arg.
Omitted when absent (wire zero value).

##### default?

```ts
optional default?: string;
```

Default string form when the operator omits the arg.
Omitted when absent (wire zero value).

##### kind

```ts
kind: "string" | "bool" | "int" | "path";
```

Parsed value kind.

##### long?

```ts
optional long?: string;
```

Long flag without leading dashes (e.g. "message" -> `--message`).
Omitted when absent (wire zero value).

##### name

```ts
name: string;
```

Internal arg name used as `CliArg.name` on invoke.

##### positional

```ts
positional: boolean;
```

When true, the arg is positional rather than a flagged option.

##### required

```ts
required: boolean;
```

When true, the host rejects invoke if the arg is missing.

##### short?

```ts
optional short?: string;
```

Short flag character (e.g. "m" -> `-m`).
Omitted when absent (wire zero value).

***

### CliCommandSpec

One plugin CLI command under `CliSchema`.

#### Properties

##### about?

```ts
optional about?: string;
```

Short help text for `--help`.
Omitted when absent (wire zero value).

##### args

```ts
args: CliArgSpec[];
```

Argument / flag specs for this command.

##### name

```ts
name: string;
```

Command verb after the plugin id (for example "ping").

***

### CliInvokeParams

Params of `PluginCli.invoke`.

#### Properties

##### args

```ts
args: CliArg[];
```

Named argument values.

##### command

```ts
command: string;
```

Command name matching a `CliCommandSpec.name`.

***

### CliInvokeResult

Result of `PluginCli.invoke`.

#### Properties

##### exitCode

```ts
exitCode: number;
```

Process-style exit code (0 = success).

##### payload

```ts
payload: ExtensibleConfig;
```

Structured payload for machine consumers; empty `mediaType` when absent.

##### stderr

```ts
stderr: string;
```

Captured standard error text.

##### stdout

```ts
stdout: string;
```

Captured standard output text.

***

### CliSchema

Declared plugin CLI surface (`PluginCli.describe` / `describe().cli`).

#### Properties

##### commands

```ts
commands: CliCommandSpec[];
```

Commands exposed as `bookclerk plugins <id> <command> ...`.

***

### ColumnReference

`REFERENCES` target of one column.

#### Properties

##### refColumns

```ts
refColumns: string[];
```

Referenced columns.

##### refTable

```ts
refTable: string;
```

Referenced table.

***

### CompletedOutcome

Job finished successfully.

#### Properties

##### bytesCopied

```ts
bytesCopied: number;
```

Bytes produced, when meaningful.

##### message

```ts
message: string;
```

Short human summary.

***

### ConfigOption

One discoverable config option group advertised for sources.

#### Properties

##### key

```ts
key: string;
```

Config key under the plugin's `config.toml` table.

##### label

```ts
label: string;
```

Operator-facing label for the option group.

##### values

```ts
values: ConfigOptionValue[];
```

Allowed selectable values for this key.

***

### ConfigOptionValue

One selectable value under a `ConfigOption`.

#### Properties

##### id

```ts
id: string;
```

Value written to config when selected.

##### label

```ts
label: string;
```

Operator-facing label for this value.

***

### ContentSource

Storefront content source (not byte Source). Every method takes and
returns typed structs from the "Typed method payloads" section.

#### Methods

##### catalogDetail()

```ts
catalogDetail(params): Promise<CatalogDetailReply>;
```

Full catalog record for one product.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`CatalogDetailParams`](#catalogdetailparams) | Product id and optional ISBN. |

###### Returns

`Promise`\<[`CatalogDetailReply`](#catalogdetailreply)\>

[CatalogDetailReply](#catalogdetailreply)

##### diagnose()

```ts
diagnose(): Promise<DiagnoseReply>;
```

Human-readable diagnostic lines.

###### Returns

`Promise`\<[`DiagnoseReply`](#diagnosereply)\>

[DiagnoseReply](#diagnosereply)

##### expandCandidates()

```ts
expandCandidates(params): Promise<CatalogHitsReply>;
```

Related-title expansion from a seed title.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`ExpandCandidatesParams`](#expandcandidatesparams) | Seed identity fields and limit. |

###### Returns

`Promise`\<[`CatalogHitsReply`](#cataloghitsreply)\>

[CatalogHitsReply](#cataloghitsreply)

##### fetchTitle()

```ts
fetchTitle(params): Promise<FetchTitleReply>;
```

Download and decrypt one title into `cacheDir`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`FetchTitleParams`](#fetchtitleparams) | Title, credentials, and fetch options. |

###### Returns

`Promise`\<[`FetchTitleReply`](#fetchtitlereply)\>

[FetchTitleReply](#fetchtitlereply)

##### health()

```ts
health(): Promise<HealthReply>;
```

Liveness / readiness probe.

###### Returns

`Promise`\<[`HealthReply`](#healthreply)\>

[HealthReply](#healthreply)

##### listAccounts()

```ts
listAccounts(): Promise<SourceAccountsReply>;
```

Enumerate accounts the guest knows about.

###### Returns

`Promise`\<[`SourceAccountsReply`](#sourceaccountsreply)\>

[SourceAccountsReply](#sourceaccountsreply)

##### listDeals()

```ts
listDeals(params): Promise<CatalogHitsReply>;
```

Current storefront deals.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`ListDealsParams`](#listdealsparams) | Optional result cap. |

###### Returns

`Promise`\<[`CatalogHitsReply`](#cataloghitsreply)\>

[CatalogHitsReply](#cataloghitsreply)

##### login()

```ts
login(params): Promise<LoginReply>;
```

Connect an account (password or one-shot OAuth).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`LoginParams`](#loginparams) | Credentials, marketplace, callback wiring. |

###### Returns

`Promise`\<[`LoginReply`](#loginreply)\>

[LoginReply](#loginreply)

##### loginComplete()

```ts
loginComplete(params): Promise<LoginReply>;
```

Finish an interactive OAuth login started by `loginStart`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`LoginCompleteParams`](#logincompleteparams) | Session id from `loginStart`. |

###### Returns

`Promise`\<[`LoginReply`](#loginreply)\>

[LoginReply](#loginreply)

##### loginStart()

```ts
loginStart(params): Promise<LoginStartReply>;
```

Begin an interactive OAuth login; returns a session id.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`LoginParams`](#loginparams) | Same shape as `login`; host fills callback IPC. |

###### Returns

`Promise`\<[`LoginStartReply`](#loginstartreply)\>

[LoginStartReply](#loginstartreply)

##### purchaseHint()

```ts
purchaseHint(params): Promise<PurchaseHintReply>;
```

Purchase link / price hint for one title.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`PurchaseHintParams`](#purchasehintparams) | Identity fields and price flag. |

###### Returns

`Promise`\<[`PurchaseHintReply`](#purchasehintreply)\>

[PurchaseHintReply](#purchasehintreply)

##### scan()

```ts
scan(params): Promise<ScanReply>;
```

Sync library rows for one or more accounts.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`ScanParams`](#scanparams) | Accounts, paging, and host-sealed credentials. |

###### Returns

`Promise`\<[`ScanReply`](#scanreply)\>

[ScanReply](#scanreply)

##### searchCatalog()

```ts
searchCatalog(params): Promise<CatalogHitsReply>;
```

Free-text storefront catalog search.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`SearchCatalogParams`](#searchcatalogparams) | Query, region, paging, sort, facet. |

###### Returns

`Promise`\<[`CatalogHitsReply`](#cataloghitsreply)\>

[CatalogHitsReply](#cataloghitsreply)

***

### CopyResult

Result of a server-side copy.

#### Properties

##### bytesCopied

```ts
bytesCopied: number;
```

***

### CreateTableSchema

Parsed `CREATE TABLE` (canonical SQL v1). Per-column lists align with `columns`.

#### Properties

##### columnChecks

```ts
columnChecks: string[];
```

Per-column CHECK expression text; empty when none.

##### columnDefaults

```ts
columnDefaults: string[];
```

Per-column DEFAULT expression text; empty when none.

##### columnNotNull

```ts
columnNotNull: boolean[];
```

Per-column NOT NULL flags.

##### columnPrimaryKey

```ts
columnPrimaryKey: boolean[];
```

Per-column PRIMARY KEY flags.

##### columnReferences

```ts
columnReferences: OptionalColumnReference[];
```

Per-column REFERENCES targets.

##### columns

```ts
columns: NamedSqlType[];
```

Columns with resolved types, in order.

##### columnUnique

```ts
columnUnique: boolean[];
```

Per-column UNIQUE flags.

##### identityColumn

```ts
identityColumn: string;
```

`INTEGER PRIMARY KEY AUTOINCREMENT` column; empty when none.

##### table

```ts
table: string;
```

Table name.

##### tableConstraints

```ts
tableConstraints: TableConstraint[];
```

Table-level constraints, in order.

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
| `T` | `Record`\<`string`, [`DbValue`](#dbvalue)\> |

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

### Database

`databaseAdapter` entrypoint (`Entrypoints.databaseAdapter`).

#### Methods

##### dropUnit()

```ts
dropUnit(unitRef): Promise<EmptyReply>;
```

Physically drop one provisioned binding unit (`unitRef` is adapter-owned).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `unitRef` | `string` | Adapter-native unit (sqlite path, postgres database, D1 name). |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### openSession()

```ts
openSession(): Promise<AdapterSessionReply>;
```

Open one adapter session (capability negotiation + typed execute).

###### Returns

`Promise`\<[`AdapterSessionReply`](#adaptersessionreply)\>

[AdapterSessionReply](#adaptersessionreply)

***

### DatabaseAdapterConfig

Author-facing database adapter configuration carried in
`DatabaseContext.adapter`. This is the generic bootstrap mechanism for
third-party adapters: the operator's granted `[database.<id>]` table plus
the scoped writable data dir. First-party host-managed adapters receive
host-private connect params in `DatabaseContext.config` instead.

#### Properties

##### binding?

```ts
optional binding?: string;
```

Named plugin database binding this open serves; empty for the primary
library open. Adapters advertising `DbCapabilities.pluginDatabases` must
serve each binding from its own isolated database.
Omitted when absent (wire zero value).

##### instanceId?

```ts
optional instanceId?: string;
```

Host-issued opaque instance id for this (owner plugin, binding) pair.
Collision-resistant and stable across re-opens. Empty for the primary
library open. Third-party adapters must key isolated databases on this
value rather than `binding` alone (two plugins may both declare `DB`).
Omitted when absent (wire zero value).

##### openExisting

```ts
openExisting: boolean;
```

When true, open an existing binding unit and do not provision a missing
one (read-only backup capture). False lets the adapter create the unit.

##### pluginDataDir

```ts
pluginDataDir: string;
```

Scoped writable directory for this plugin (`.../plugins/<id>/data`).

##### settings

```ts
settings: ExtensibleConfig;
```

Granted plugin settings (operator `[database.<id>]` table) as
`application/json`; `{}` when the operator configured nothing.

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

`Promise`\<[`D1Result`](#d1result)\<`Record`\<`string`, [`DbValue`](#dbvalue)\>\>[]\>

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
optional deadlineUnixMs?: number;
```

Guest-visible deadline (unix ms).

##### maxRequestBytes?

```ts
optional maxRequestBytes?: number;
```

Negotiated `maxRequestBytes` (`0` = unlimited).

##### maxResultRows?

```ts
optional maxResultRows?: number;
```

Default `maxRows` for `all()` (`0` = host adapter cap).

##### operationId?

```ts
optional operationId?: string;
```

Idempotency key; generated when omitted.

##### requestHash?

```ts
optional requestHash?: string;
```

SHA-256 hex of the request; empty when omitted.

##### retry?

```ts
optional retry?: RetryToken;
```

Default retry token; omitted calls mint a UUID and leave the hash empty.

***

### DbBootstrap

Bootstrap-only diagnostic metadata (not a capability).

#### Properties

##### engine

```ts
engine: string;
```

Diagnostic physical engine name. Hosts must not admit or generate SQL
from this value. Any string is valid.

***

### DbCapabilities

Semantic SQL-contract advertisement. Diagnostic engine identity is not
part of the capability plane — see `DbBootstrap`.

#### Properties

##### affectedRows

```ts
affectedRows: boolean;
```

Guest reports `rowsAffected`.

##### atomicBatch

```ts
atomicBatch: boolean;
```

Guest can run a bounded statement list as one SQL transaction.

##### atomicUnitRestore

```ts
atomicUnitRestore: boolean;
```

Adapter can destructively replace one logical database unit so an
ordinary restore failure does not leave that unit partially replaced.

##### cancellation

```ts
cancellation: boolean;
```

Guest honors RPC/session cancellation.

##### consistentBackupRead

```ts
consistentBackupRead: boolean;
```

Adapter can expose one stable logical database state while the host
reads schema, rows, and identity.

##### maxAtomicResultBytes

```ts
maxAtomicResultBytes: number;
```

Maximum encoded bytes of one `ExecuteReply`.

##### maxBinds

```ts
maxBinds: number;
```

Maximum bound parameters per statement.

##### maxCellBytes

```ts
maxCellBytes: number;
```

Maximum UTF-8 / blob bytes of one result cell.

##### maxFunctionArgs

```ts
maxFunctionArgs: number;
```

Maximum arguments in one physical function call after adapter hiding
(nested min / max / coalesce). Portable json_object is already ≤ 32
arguments (16 pairs), matching D1. `0` is unspecified.

##### maxLoweredStatementBytes

```ts
maxLoweredStatementBytes: number;
```

Maximum UTF-8 bytes of sqlite-family lowered SQL the adapter can realize
for one statement (INTEGER overflow wraps, LIKE→GLOB, NULLIF, INSERT OR
IGNORE, query LIMIT wrap, bytes-placeholder expansion). `0` is
unspecified: the host does not enforce a lowered-size ceiling. First-party
D1 advertises 100000. Hosts compare a standardized Bookclerk lowering
upper bound against this number and must not branch on engine identity.

##### maxPatternBytes

```ts
maxPatternBytes: number;
```

Maximum UTF-8 bytes of a BookclerkSQL LIKE pattern value (literals and
TEXT binds). Adapters that expand LIKE into GLOB must advertise a
conservative value that still fits the physical pattern cap. `0` is
unspecified.

##### maxPayloadBytes

```ts
maxPayloadBytes: number;
```

Maximum UTF-8 bytes of SQL plus binds per statement.

##### maxRequestBytes

```ts
maxRequestBytes: number;
```

Maximum encoded bytes of one `ExecuteRequest`.

##### maxResultBytes

```ts
maxResultBytes: number;
```

Maximum encoded bytes of one statement's result rows.

##### maxResultRows

```ts
maxResultRows: number;
```

Maximum rows a query statement may return.

##### maxSchemaColumns

```ts
maxSchemaColumns: number;
```

Maximum columns in one CREATE TABLE / result row. `0` is unspecified.

##### maxStatements

```ts
maxStatements: number;
```

Maximum statements in one atomic batch.

##### pluginDatabases

```ts
pluginDatabases: boolean;
```

Append-only. Adapter can open additional isolated sessions
for plugin-owned database bindings (per-binding file / schema / database).

##### returning

```ts
returning: boolean;
```

Guest SQL supports `RETURNING`.

##### schemaMigrations

```ts
schemaMigrations: boolean;
```

Guest versions schema with a `bookclerk_schema_migrations` table.

##### sqlContractVersion

```ts
sqlContractVersion: number;
```

Bookclerk SQL contract version.

##### timing

```ts
timing: boolean;
```

Guest can fill `DbTiming.dbExecutionUs`. Not a host connect minimum.

***

### DbColumn

One column in a typed result set.

#### Properties

##### dbType

```ts
dbType: "unspecified" | "bool" | "int64" | "float64" | "text" | "bytes";
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

### DbStatement

One guest statement in an `ExecuteRequest`.

#### Properties

##### kind

```ts
kind: "execute" | "select" | "returning";
```

Statement classification.

##### maxRows

```ts
maxRows: number;
```

Row cap for queries; `0` means adapter default.

##### parameters

```ts
parameters: DbValue[];
```

Positional bind values.

##### resultSelection

```ts
resultSelection: "discard" | "affectedRows" | "rows";
```

Which outcome parts to return.

##### sql

```ts
sql: string;
```

BookclerkSQL text; at most `maxScalarBytes`.

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

### DeliveredCheckpoint

A prior checkpoint delivered on resume.

#### Properties

##### json

```ts
json: string;
```

Checkpoint JSON text as recorded by the earlier `suspend()`.

##### schemaVersion

```ts
schemaVersion: number;
```

Schema version the author attached to the checkpoint.

***

### DiagnoseResult

Operator-facing diagnostic lines printed by `bookclerk plugins diagnose`.

#### Properties

##### lines

```ts
lines: string[];
```

Human-readable probe lines.

***

### DomainEvent

Domain event (not a job). Outbox-produced, at-least-once, idempotent consume.

#### Properties

##### accountId

```ts
accountId: string;
```

Account scope; empty for host-wide events.

##### causationId

```ts
causationId: string;
```

Id of the command or event that caused this one; empty when none.

##### checkpointJson

```ts
checkpointJson: string;
```

Append-only. Resume a prior EventResult.suspended.

##### checkpointSchemaVersion

```ts
checkpointSchemaVersion: number;
```

Schema version of `checkpointJson`.

##### correlationId

```ts
correlationId: string;
```

Trace correlation id; empty when none.

##### deduplicationKey

```ts
deduplicationKey: string;
```

Consumer-side idempotency key; stable across redeliveries.

##### deliveryAttempt

```ts
deliveryAttempt: number;
```

Delivery counter, starting at 1.

##### eventId

```ts
eventId: string;
```

Unique event id (outbox row identity).

##### eventType

```ts
eventType: string;
```

Dotted event type (e.g. `library.title.added`).

##### invocationSequence

```ts
invocationSequence: number;
```

Resume ordinal; distinct from `deliveryAttempt`.

##### occurredAtUnixMs

```ts
occurredAtUnixMs: number;
```

When the producer observed the fact.

##### payload

```ts
payload: Uint8Array;
```

Encoded event payload; at most `maxEventPayloadBytes`.

##### resumePending

```ts
resumePending: boolean;
```

True when this delivery resumes a prior suspension.

##### schemaVersion

```ts
schemaVersion: number;
```

Schema version of `payload`, owned by the event type.

##### source

```ts
source: string;
```

Append-only. Producer plugin id; empty when unknown.

***

### Entrypoints

Exported entrypoints returned by `PluginWorker.open`. One capability per
`plugin.toml` `entrypoints` entry / trigger; null when not exported. The host
refuses an entrypoint the manifest or operator grant did not allow.

#### Properties

##### cli?

```ts
optional cli?: PluginCli | null;
```

`cli` entrypoint.
Null when the guest does not export this capability.

##### databaseAdapter?

```ts
optional databaseAdapter?: Database | null;
```

`databaseAdapter` entrypoint.
Null when the guest does not export this capability.

##### eventConsumer?

```ts
optional eventConsumer?: EventConsumer | null;
```

`[[events.consumers]]` trigger: `event(batch)` handler.
Null when the guest does not export this capability.

##### jobRunner?

```ts
optional jobRunner?: JobRunner | null;
```

`[triggers] jobs` trigger: `job(controller)` handler.
Null when the guest does not export this capability.

##### oidc?

```ts
optional oidc?: Oidc | null;
```

`oidc` entrypoint.
Null when the guest does not export this capability.

##### queueConsumer?

```ts
optional queueConsumer?: QueueConsumer | null;
```

`[[queues.consumers]]` trigger: `queue(batch)` handler.
Null when the guest does not export this capability.

##### remoteLibrary?

```ts
optional remoteLibrary?: RemoteLibrary | null;
```

`remoteLibrary` entrypoint.
Null when the guest does not export this capability.

##### storage?

```ts
optional storage?: Destination | null;
```

`storage` entrypoint.
Null when the guest does not export this capability.

##### storefront?

```ts
optional storefront?: ContentSource | null;
```

`storefront` entrypoint.
Null when the guest does not export this capability.

***

### EventAck

Event handled; the host marks it delivered.

#### Properties

##### dummy

```ts
dummy: void;
```

Placeholder; the struct carries no data.

***

### EventConsumer

`[[events.consumers]]` handler (`Entrypoints.eventConsumer`). Delivery is
at-least-once; consumers must be idempotent on `deduplicationKey`.

#### Methods

##### event()

```ts
event(batch): Promise<EventBatchReply>;
```

Deliver one ordered batch of domain events.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `batch` | `EventBatch` | Events in delivery order. |

###### Returns

`Promise`\<[`EventBatchReply`](#eventbatchreply)\>

[EventBatchReply](#eventbatchreply)

***

### EventConsumerSpec

One declared event consumer: a `[[events.consumers]]` row the default
entrypoint's `event(batch)` handler accepts.

#### Properties

##### eventType

```ts
eventType: string;
```

Versioned event type (snake_case, e.g. `book_acquired`).

##### schemaVersions

```ts
schemaVersions: number[];
```

Schema versions the guest can consume; never empty.

##### supportsSuspend

```ts
supportsSuspend: boolean;
```

Whether `EventResult.suspended` is supported for this type.

***

### EventDeadLetter

Event moved to the dead-letter queue for operator review.

#### Properties

##### reason

```ts
reason: string;
```

Short human reason.

***

### EventPollResult

Success payload of `RemoteLibrary.pollEvents`: signals for the host to kick
off workflows.

#### Properties

##### users

```ts
users: ExternalUser[];
```

Newly observed external users since the last poll.

***

### EventPublisher

`EVENTS` binding: host-served outbox publisher granted through
`Bindings.events`. Event types must be listed in `[[events.producers]]`.

#### Methods

##### publish()

```ts
publish(event): Promise<PublishReply>;
```

Append one event to the outbox (idempotent on `deduplicationKey`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `event` | [`PluginEvent`](#pluginevent) | Event to publish. |

###### Returns

`Promise`\<[`PublishReply`](#publishreply)\>

[PublishReply](#publishreply)

***

### EventPublisherBinding

`EVENTS` binding: outbox publisher, present when `[[events.producers]]` is
declared and granted.

#### Methods

##### publish()

```ts
publish(event): Promise<PublishOk>;
```

Publish one domain event through the host outbox.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `event` | [`PublishEvent`](#publishevent) | Event type, schema version, payload, dedup key. |

###### Returns

`Promise`\<[`PublishOk`](#publishok)\>

Outbox row identity (`duplicate` when the dedup key matched).

***

### EventReject

Event rejected; the host records the reason and stops delivering.

#### Properties

##### reason

```ts
reason: string;
```

Short human reason.

***

### EventRetry

Redeliver later.

#### Properties

##### reason

```ts
reason: string;
```

Short human reason.

##### retryAtUnixMs

```ts
retryAtUnixMs: number;
```

Earliest redelivery time; `0` lets the host choose.

***

### EventSuspended

Append-only. Mirrors job SuspendedOutcome; event handlers persist a
bounded checkpoint and release the process until wakeAtUnixMs. Optional
wake-on-matching-event fields (empty = timestamp-only).

#### Properties

##### checkpointJson

```ts
checkpointJson: string;
```

Bounded checkpoint to replay on resume; at most `maxCheckpointBytes`.

##### checkpointSchemaVersion

```ts
checkpointSchemaVersion: number;
```

Schema version of `checkpointJson`.

##### wakeAtUnixMs

```ts
wakeAtUnixMs: number;
```

Earliest resume time.

##### wakeOnEventType

```ts
wakeOnEventType: string;
```

Also wake when an event of this type arrives; empty disables.

##### wakeOnFilterJson

```ts
wakeOnFilterJson: string;
```

JSON filter applied to matching wake events; empty matches all.

***

### EventSuspendOptions

Options for [EventMessage.suspend](#suspend).

#### Properties

##### checkpoint?

```ts
optional checkpoint?: Checkpoint;
```

Bounded checkpoint replayed on resume (at most `maxCheckpointBytes`).

##### checkpointSchemaVersion?

```ts
optional checkpointSchemaVersion?: number;
```

Schema version of `checkpoint` (default `1`).

##### wakeAt?

```ts
optional wakeAt?: Instant;
```

Earliest resume instant.

##### wakeOnEventType?

```ts
optional wakeOnEventType?: string;
```

Also wake when an event of this type arrives.

##### wakeOnFilter?

```ts
optional wakeOnFilter?: string | JsonObject;
```

Host-owned payload filter for wake events (object or JSON text).

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

### ExpandCandidatesParams

Params of `ContentSource.expandCandidates`. Seed fields identify a known
title; the guest returns related catalog hits.

#### Properties

##### asin?

```ts
optional asin?: string;
```

Seed Amazon ASIN.
Omitted when absent (wire zero value).

##### authors?

```ts
optional authors?: string;
```

Seed authors string.
Omitted when absent (wire zero value).

##### isbn?

```ts
optional isbn?: string;
```

Seed ISBN.
Omitted when absent (wire zero value).

##### limit

```ts
limit: number;
```

Maximum candidates to return; the host always sends an explicit value.

##### narrators?

```ts
optional narrators?: string;
```

Seed narrators string.
Omitted when absent (wire zero value).

##### productId

```ts
productId: string;
```

Seed storefront product id.

##### region

```ts
region: string;
```

Storefront region / marketplace code.

##### series?

```ts
optional series?: string;
```

Seed series name.
Omitted when absent (wire zero value).

##### seriesAsin?

```ts
optional seriesAsin?: string;
```

Seed series ASIN when known.
Omitted when absent (wire zero value).

##### source

```ts
source: string;
```

Source plugin id hint when expanding across storefronts.

##### title

```ts
title: string;
```

Seed title text.

***

### ExtensibleConfig

Plugin-specific extensible config. Not a substitute for typed ABI fields.

#### Properties

##### mediaType

```ts
mediaType: string;
```

Media type of `payload` (e.g. `application/json`).

##### payload

```ts
payload: Uint8Array;
```

Bounded encoded payload; at most `maxConfigPayloadBytes`.

##### schemaVersion

```ts
schemaVersion: number;
```

Version of `payload`'s schema, owned by the plugin.

***

### ExternalUser

One external user observed by an integration. The host may mint claim
tickets without exposing portal details to the guest.

#### Properties

##### accessToken?

```ts
optional accessToken?: string;
```

Ephemeral remote token (e.g. ABS JWT). Guest-to-host only; never
persisted.
Omitted when absent (wire zero value).

##### displayName?

```ts
optional displayName?: string;
```

Display name for UI.
Omitted when absent (wire zero value).

##### externalUserId

```ts
externalUserId: string;
```

Provider-scoped user id.

##### provider

```ts
provider: string;
```

Integration provider id (often the plugin id).

***

### FetchOptions

Fetch-relevant acquire knobs the host forwards so external load matches
in-process. Packaging / naming knobs stay host-side.

#### Properties

##### chapterLayout

```ts
chapterLayout: string;
```

Preferred chapter API layout when fetching (`tree` or `flat`).

##### coverSize

```ts
coverSize: string;
```

Cover image size request (`500`, `1215`, or `native`).

##### downloadClipsBookmarks

```ts
downloadClipsBookmarks: boolean;
```

When true, download clips/bookmarks sidecars when offered.

##### downloadCover

```ts
downloadCover: boolean;
```

When true, download a cover image alongside audio.

##### downloadPdf

```ts
downloadPdf: boolean;
```

When true, download a companion PDF when the store exposes one.

##### downloadSpeedLimitKbps

```ts
downloadSpeedLimitKbps: number;
```

Fetch speed cap in KB/s (`0` = unlimited).

##### retainAaxFile

```ts
retainAaxFile: boolean;
```

When true, keep the encrypted download in storage.

##### saveMetadataJson

```ts
saveMetadataJson: boolean;
```

When true, persist raw catalog API JSON as `metadata.json`.

##### stripAudibleBrandAudio

```ts
stripAudibleBrandAudio: boolean;
```

When true, trim Audible brand intro/outro from the remux window.

##### widevine

```ts
widevine: boolean;
```

Prefer Widevine/CENC download when the store offers it.

##### widevineCdmPath?

```ts
optional widevineCdmPath?: string;
```

Local Widevine `.wvd` path granted to the guest.
Omitted when absent (wire zero value).

##### widevineCdmProvider?

```ts
optional widevineCdmProvider?: string;
```

Remote L3 CDM provider URL; empty means the classic default, `off`
disables remote provisioning.
Omitted when absent (wire zero value).

##### xheAac

```ts
xheAac: boolean;
```

Prefer xHE-AAC on the Widevine path when offered.

***

### FetchTitleParams

Params of `ContentSource.fetchTitle`. The plugin writes media under
`cacheDir` and returns plain (DRM-free) paths. The host injects
credentials; guests must not open `library.db` or `master.key`.

#### Properties

##### accountId

```ts
accountId: string;
```

Account whose credentials apply.

##### cacheDir

```ts
cacheDir: string;
```

Absolute path the guest should write media into (jail-granted TMPDIR).

##### credentials?

```ts
optional credentials?: Uint8Array<ArrayBufferLike>;
```

Host-loaded credential blob for this account; empty when unavailable.
Omitted when absent (wire zero value).

##### fetch

```ts
fetch: FetchOptions;
```

Fetch-relevant acquire options.

##### pluginDataDir

```ts
pluginDataDir: string;
```

Scoped plugin data directory.

##### sourceConfig

```ts
sourceConfig: ExtensibleConfig;
```

Granted `[sources.<id>]` table as `application/json`.

##### titleId

```ts
titleId: string;
```

Library / storefront title id to download.

***

### ForeignKeyConstraint

Table-level `FOREIGN KEY` constraint.

#### Properties

##### columns

```ts
columns: string[];
```

Local columns.

##### refColumns

```ts
refColumns: string[];
```

Referenced columns.

##### refTable

```ts
refTable: string;
```

Referenced table.

***

### GetOk

Success payload of `Destination.get`.

#### Properties

##### body?

```ts
optional body?: ByteSource | null;
```

Streamed object bytes.
Null when the guest does not export this capability.

##### meta

```ts
meta: ObjectMetadata;
```

Object metadata.

***

### GrantedContext

Granted `Bindings` for one invocation as the bridge routes carry them:
the `context` body field / `x-bookclerk-context` header decoded from
`PluginWorker.open(invocation, bindings)`. The adapter merges it onto the
author's `env` before dispatching; authors never receive it directly.

#### Extended by

- [`JobRunnerContext`](#jobrunnercontext)

#### Properties

##### config?

```ts
optional config?: ExtensibleConfig;
```

Operator settings for this plugin as `application/json`.

##### databases?

```ts
optional databases?: NamedDatabase[];
```

Named `[[databases]]` bindings.

##### events?

```ts
optional events?: EventPublisherBinding;
```

`EVENTS` publisher, when `[[events.producers]]` is granted.

##### invocation?

```ts
optional invocation?: Partial<Invocation>;
```

Invocation identity (`Invocation` struct).

##### kv?

```ts
optional kv?: NamedKv[];
```

`[[kv_namespaces]]` bindings, when granted.

##### queues?

```ts
optional queues?: NamedQueue[];
```

`[[queues.producers]]` bindings, when granted.

##### secrets?

```ts
optional secrets?: ExtensibleConfig;
```

Granted secret values as `application/json`.

##### storage?

```ts
optional storage?: Destination;
```

`WORK_FS` object store, when granted.

***

### GrantedDatabaseTransport

Granted-channel SQL transport the adapter hands to the author isolate for
one named `[[databases]]` binding. The author-side SDK wraps it as the
D1-shaped [DatabaseBinding](#databasebinding) on `env.<BINDING>`; Python authors call
[GrantedDatabaseTransport.executeBytes](#executebytes) with Cap'n bytes instead.

#### Extends

- [`AtomicTransport`](#atomictransport)

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

###### Inherited from

[`AtomicTransport`](#atomictransport).[`execute`](#execute-1)

##### executeBytes()

```ts
executeBytes(bytes): Promise<Uint8Array<ArrayBufferLike>>;
```

Same batch as [AtomicTransport.execute](#execute-1), as Cap'n Proto bytes
(`ExecuteRequest` in, `ExecuteResultReply` out).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bytes` | `Uint8Array` | Unpacked `ExecuteRequest` message. |

###### Returns

`Promise`\<`Uint8Array`\<`ArrayBufferLike`\>\>

Unpacked `ExecuteResultReply` message.

***

### GrantedFetcher

Reverse-channel fetcher bound as `GRANTED` on the adapter isolate.

#### Methods

##### fetch()

```ts
fetch(input, init?): Promise<Response>;
```

Call the host grant broker.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `input` | `string` | Request URL. |
| `init?` | `RequestInit` | Optional fetch init. |

###### Returns

`Promise`\<`Response`\>

Host response.

***

### GrantedJobCapabilities

Granted capabilities for one `job()` invocation (adapter-internal).

#### Properties

##### cancel?

```ts
optional cancel?: CancelWatchLike | null;
```

##### input?

```ts
optional input?: Source | null;
```

##### output?

```ts
optional output?: Destination | null;
```

##### progress?

```ts
optional progress?: ProgressSink | null;
```

***

### GuestDatabase

Host-granted SQL for job plugin authors. SDK `DatabaseBinding` mirrors the
Cloudflare Workers D1 surface (`prepare`/`bind`/`run`/`first`/`all`/`raw`,
`batch`, `exec`) over this typed `execute` transport; wire types stay Cap'n
`ExecuteRequest`/`ExecuteReply`.

#### Methods

##### close()

```ts
close(): Promise<EmptyReply>;
```

Release the session; further calls fail.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### execute()

```ts
execute(request): Promise<ExecuteResultReply>;
```

Run one typed statement batch.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | `ExecuteRequest` | Statements, binds, and deadline. |

###### Returns

`Promise`\<[`ExecuteResultReply`](#executeresultreply)\>

[ExecuteResultReply](#executeresultreply)

***

### HeadOk

Success payload of `Destination.head`.

#### Properties

##### found

```ts
found: boolean;
```

Whether the object exists.

##### meta

```ts
meta: ObjectMetadata;
```

Object metadata; meaningful only when `found`.

***

### HealthOk

Typed liveness report.

#### Properties

##### detail

```ts
detail: string;
```

Short human status line; empty when none.

##### ok

```ts
ok: boolean;
```

True when the guest is healthy enough for traffic.

***

### IdentityHighWater

Adapter-private identity high-water (sqlite_sequence / bookclerk_identity).
Column names live in the canonical backup schema, not this catalog.

#### Properties

##### last

```ts
last: bigint;
```

Highest generated or stored value that must not be reused.

##### table

```ts
table: string;
```

Table whose identity column the mark belongs to.

***

### IntegerArithSite

One INTEGER arithmetic expression that must not wrap or error on overflow.

#### Properties

##### full

```ts
full: SqlSpan;
```

Full expression span (`a + b` or `abs(n)`).

##### kind

```ts
kind: "add" | "sub" | "mul" | "abs";
```

Operator.

##### lhs

```ts
lhs: SqlSpan;
```

Left operand (or `abs` argument).

##### rhs

```ts
rhs: SqlSpan;
```

Right operand (`abs` repeats `lhs`).

***

### Invocation

Identity of one `PluginWorker.open` invocation. The host issues ids; guests
echo `correlationId` / `causationId` onto published events.

#### Properties

##### accountId

```ts
accountId: string;
```

Account scope; empty for operator / host-wide invocations.

##### causationId

```ts
causationId: string;
```

Id of the event or command that caused this invocation; empty when none.

##### correlationId

```ts
correlationId: string;
```

Trace correlation id; empty when none.

##### deadlineUnixMs

```ts
deadlineUnixMs: number;
```

UTC Unix milliseconds; zero when the invocation has no deadline. The host
fence is authoritative.

##### id

```ts
id: string;
```

Unique host-issued invocation id.

***

### JobCheckpoint

Bounded, versioned checkpoint.

#### Properties

##### json

```ts
json: string;
```

##### schemaVersion

```ts
schemaVersion: number;
```

***

### JobCompletion

Value a `job()` handler may return to annotate a `completed` outcome.

#### Properties

##### bytesCopied?

```ts
optional bytesCopied?: number;
```

Bytes copied by the job, when meaningful.

##### message?

```ts
optional message?: string;
```

Operator-facing summary.

***

### JobInvocation

Durable command envelope (not a domain event). Command payload schema
version and checkpoint schema versions are independent of the plugin ABI.
The envelope itself is not persisted as opaque bytes across ABI majors.
Idempotency keys are scoped to (account, plugin, commandType) until a
terminal fenced outcome is committed.

#### Properties

##### attempt

```ts
attempt: number;
```

Failure retry counter, starting at 1.

##### causationId

```ts
causationId: string;
```

Id of the event or command that caused this one; empty when none.

##### checkpointJson

```ts
checkpointJson: string;
```

Checkpoint persisted by a prior `SuspendedOutcome`; empty on first run.

##### checkpointSchemaVersion

```ts
checkpointSchemaVersion: number;
```

Schema version of `checkpointJson`.

##### commandType

```ts
commandType: string;
```

Command type the handler dispatches on.

##### correlationId

```ts
correlationId: string;
```

Trace correlation id; empty when none.

##### deadlineUnixMs

```ts
deadlineUnixMs: number;
```

UTC Unix milliseconds. Host fence/lease is authoritative; this hint must
not outlive the fence (clock skew across VPS nodes).

##### idempotencyKey

```ts
idempotencyKey: string;
```

Caller idempotency key scoped to (account, plugin, commandType).

##### invocationId

```ts
invocationId: string;
```

Unique id of this invocation attempt.

##### invocationSequence

```ts
invocationSequence: number;
```

Resume ordinal; distinct from failure `attempt`.

##### payloadJson

```ts
payloadJson: string;
```

Command payload (JSON); at most `maxScalarBytes`.

##### payloadSchemaVersion

```ts
payloadSchemaVersion: number;
```

Schema version of `payloadJson`, owned by the command type.

##### stepId

```ts
stepId: string;
```

Optional step identifier for multi-step commands.

***

### JobRetryOptions

Options for [JobController.retryLater](#retrylater).

#### Properties

##### reason?

```ts
optional reason?: string;
```

Short operator-facing reason.

##### retryAt?

```ts
optional retryAt?: Instant;
```

Earliest retry instant; omitted lets the host choose.

***

### JobRunner

`[triggers] jobs` handler (`Entrypoints.jobRunner`); runs one durable command.

#### Methods

##### job()

```ts
job(controller): Promise<HandleReply>;
```

Run one command invocation to a terminal or suspended outcome. Named
database bindings come from `Bindings.databases` at `open`, not per job.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `controller` | `JobController` | Envelope plus host-served capabilities. |

###### Returns

`Promise`\<[`HandleReply`](#handlereply)\>

[HandleReply](#handlereply)

***

### JobRunnerContext

Job-runner bridge context: the durable job id plus [GrantedContext](#grantedcontext).

#### Extends

- [`GrantedContext`](#grantedcontext)

#### Properties

##### config?

```ts
optional config?: ExtensibleConfig;
```

Operator settings for this plugin as `application/json`.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`config`](#config-2)

##### databases?

```ts
optional databases?: NamedDatabase[];
```

Named `[[databases]]` bindings.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`databases`](#databases-1)

##### events?

```ts
optional events?: EventPublisherBinding;
```

`EVENTS` publisher, when `[[events.producers]]` is granted.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`events`](#events-2)

##### invocation?

```ts
optional invocation?: Partial<Invocation>;
```

Invocation identity (`Invocation` struct).

###### Inherited from

[`GrantedContext`](#grantedcontext).[`invocation`](#invocation-10)

##### jobId?

```ts
optional jobId?: string;
```

Durable job id (`Invocation.id` of the job open).

##### kv?

```ts
optional kv?: NamedKv[];
```

`[[kv_namespaces]]` bindings, when granted.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`kv`](#kv-2)

##### queues?

```ts
optional queues?: NamedQueue[];
```

`[[queues.producers]]` bindings, when granted.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`queues`](#queues-1)

##### secrets?

```ts
optional secrets?: ExtensibleConfig;
```

Granted secret values as `application/json`.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`secrets`](#secrets-2)

##### storage?

```ts
optional storage?: Destination;
```

`WORK_FS` object store, when granted.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`storage`](#storage-2)

***

### JobSuspendOptions

Options for [JobController.suspend](#suspend-1).

#### Properties

##### checkpoint?

```ts
optional checkpoint?: Checkpoint;
```

Bounded checkpoint replayed on resume (at most `maxCheckpointBytes`).

##### checkpointSchemaVersion?

```ts
optional checkpointSchemaVersion?: number;
```

Schema version of `checkpoint` (default `1`).

##### wakeAt?

```ts
optional wakeAt?: Instant;
```

Earliest resume instant.

***

### KvDeleteOk

Success payload of `KvNamespace.delete`.

#### Properties

##### existed

```ts
existed: boolean;
```

True when a live value existed before the delete.

***

### KvGetOk

Success payload of `KvNamespace.get`.

#### Properties

##### expirationUnixMs

```ts
expirationUnixMs: number;
```

Absolute expiry (UTC Unix milliseconds); zero means none.

##### found

```ts
found: boolean;
```

True when the key exists and has not expired.

##### metadata

```ts
metadata: string;
```

Stored metadata JSON text; empty when none.

##### value

```ts
value: Uint8Array;
```

Stored value; empty when not found.

***

### KvListKey

One key on a `KvNamespace.list` page.

#### Properties

##### expirationUnixMs

```ts
expirationUnixMs: number;
```

Absolute expiry (UTC Unix milliseconds); zero means none.

##### key

```ts
key: string;
```

Key name.

##### metadata

```ts
metadata: string;
```

Stored metadata JSON text; empty when none.

***

### KvListKeyBinding

One key on a [KvNamespaceBinding.list](#list-3) page.

#### Properties

##### expiration?

```ts
optional expiration?: number;
```

Absolute expiry in seconds since the Unix epoch, when set.

##### metadata?

```ts
optional metadata?: unknown;
```

Stored metadata, when set.

##### name

```ts
name: string;
```

Key name.

***

### KvListOk

Success payload of `KvNamespace.list`.

#### Properties

##### cursor

```ts
cursor: string;
```

Cursor for the next page; empty when `listComplete`.

##### keys

```ts
keys: KvListKey[];
```

Keys in lexical order.

##### listComplete

```ts
listComplete: boolean;
```

True when no further page exists.

***

### KvListOptions

Page request for `KvNamespace.list`.

#### Properties

##### cursor

```ts
cursor: string;
```

Opaque cursor from a previous page; empty means the first page.

##### limit

```ts
limit: number;
```

Most keys to return; zero means the host default, at most `maxListPage`.

##### prefix

```ts
prefix: string;
```

Only keys starting with this prefix; empty means all.

***

### KvListOptionsBinding

Page request for [KvNamespaceBinding.list](#list-3).

#### Properties

##### cursor?

```ts
optional cursor?: string;
```

Cursor from a previous page.

##### limit?

```ts
optional limit?: number;
```

Most keys to return (host default when omitted, at most `maxListPage`).

##### prefix?

```ts
optional prefix?: string;
```

Only keys starting with this prefix.

***

### KvListResultBinding

One page of [KvNamespaceBinding.list](#list-3).

#### Properties

##### cursor?

```ts
optional cursor?: string;
```

Cursor for the next page; absent when `list_complete`.

##### keys

```ts
keys: KvListKeyBinding[];
```

Keys in lexical order.

##### list\_complete

```ts
list_complete: boolean;
```

True when no further page exists.

***

### KvNamespace

Durable key/value namespace: a Workers `KVNamespace`-shaped binding the host
serves from the library database, scoped to the plugin and the namespace a
`[[kv_namespaces]]` row names, granted through `Bindings.kv`. Values live in
the cluster, so an instance moved to another host reads its own keys.

#### Methods

##### delete()

```ts
delete(key): Promise<KvDeleteReply>;
```

Delete one key; a missing key is not an error.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to delete. |

###### Returns

`Promise`\<[`KvDeleteReply`](#kvdeletereply)\>

[KvDeleteReply](#kvdeletereply)

##### get()

```ts
get(key): Promise<KvGetReply>;
```

Read one key; `found = false` when absent or expired.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key, at most `maxKvKeyBytes` UTF-8 bytes. |

###### Returns

`Promise`\<[`KvGetReply`](#kvgetreply)\>

[KvGetReply](#kvgetreply)

##### list()

```ts
list(options): Promise<KvListReply>;
```

List keys in lexical order, one page at a time.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options` | [`KvListOptions`](#kvlistoptions) | Prefix, cursor, and page size. |

###### Returns

`Promise`\<[`KvListReply`](#kvlistreply)\>

[KvListReply](#kvlistreply)

##### put()

```ts
put(
   key, 
   value, 
   options
): Promise<KvPutReply>;
```

Write one key, replacing any previous value and metadata.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key, at most `maxKvKeyBytes` UTF-8 bytes. |
| `value` | `Uint8Array` | Value, at most the namespace's `max_value_bytes`. |
| `options` | [`KvPutOptions`](#kvputoptions) | Expiration and metadata. |

###### Returns

`Promise`\<[`KvPutReply`](#kvputreply)\>

[KvPutReply](#kvputreply)

***

### KvNamespaceBinding

`[[kv_namespaces]]` binding (`env.<BINDING>`, default `KV`): Workers
`KVNamespace`-shaped durable storage the host serves from the library
database, scoped to this plugin and the namespace the row names. Values
follow the instance across hosts.

#### Methods

##### delete()

```ts
delete(key): Promise<void>;
```

Delete one key; a missing key is not an error.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to delete. |

###### Returns

`Promise`\<`void`\>

##### get()

###### Call Signature

```ts
get(key, type?): Promise<string | null>;
```

Read one key.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key, at most `maxKvKeyBytes` UTF-8 bytes. |
| `type?` | `"text"` | Decoding (`text` by default). |

###### Returns

`Promise`\<`string` \| `null`\>

The value, or `null` when absent or expired.

###### Call Signature

```ts
get<T>(key, type): Promise<T | null>;
```

Read one key and parse it as JSON.

###### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `T` | `unknown` |

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key, at most `maxKvKeyBytes` UTF-8 bytes. |
| `type` | `"json"` | `json`. |

###### Returns

`Promise`\<`T` \| `null`\>

The parsed value, or `null` when absent or expired.

###### Call Signature

```ts
get(key, type): Promise<ArrayBuffer | null>;
```

Read one key as bytes.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key, at most `maxKvKeyBytes` UTF-8 bytes. |
| `type` | `"arrayBuffer"` | `arrayBuffer`. |

###### Returns

`Promise`\<`ArrayBuffer` \| `null`\>

The bytes, or `null` when absent or expired.

##### getWithMetadata()

###### Call Signature

```ts
getWithMetadata(key, type?): Promise<KvValueWithMetadata<string>>;
```

Read one key with its metadata.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to read. |
| `type?` | `"text"` | Decoding (`text` by default). |

###### Returns

`Promise`\<[`KvValueWithMetadata`](#kvvaluewithmetadata)\<`string`\>\>

Value and metadata; both `null` when absent.

###### Call Signature

```ts
getWithMetadata<T>(key, type): Promise<KvValueWithMetadata<T>>;
```

Read one key as JSON with its metadata.

###### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `T` | `unknown` |

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to read. |
| `type` | `"json"` | `json`. |

###### Returns

`Promise`\<[`KvValueWithMetadata`](#kvvaluewithmetadata)\<`T`\>\>

Parsed value and metadata; both `null` when absent.

###### Call Signature

```ts
getWithMetadata(key, type): Promise<KvValueWithMetadata<ArrayBuffer>>;
```

Read one key as bytes with its metadata.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to read. |
| `type` | `"arrayBuffer"` | `arrayBuffer`. |

###### Returns

`Promise`\<[`KvValueWithMetadata`](#kvvaluewithmetadata)\<`ArrayBuffer`\>\>

Bytes and metadata; both `null` when absent.

##### list()

```ts
list(options?): Promise<KvListResultBinding>;
```

List keys in lexical order, one page at a time.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`KvListOptionsBinding`](#kvlistoptionsbinding) | Prefix, cursor, and page size. |

###### Returns

`Promise`\<[`KvListResultBinding`](#kvlistresultbinding)\>

One page of keys.

##### put()

```ts
put(
   key, 
   value, 
   options?
): Promise<void>;
```

Write one key, replacing any previous value and metadata.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to write. |
| `value` | `string` \| `ArrayBuffer` \| `ArrayBufferView`\<`ArrayBufferLike`\> | String (stored as UTF-8) or bytes. |
| `options?` | [`KvPutOptionsBinding`](#kvputoptionsbinding) | Expiration and metadata. |

###### Returns

`Promise`\<`void`\>

***

### KvPutOk

Success payload of `KvNamespace.put`.

#### Properties

##### expirationUnixMs

```ts
expirationUnixMs: number;
```

Effective absolute expiry (UTC Unix milliseconds); zero means none.

***

### KvPutOptions

Expiration and metadata for `KvNamespace.put`.

#### Properties

##### expirationTtlSeconds

```ts
expirationTtlSeconds: number;
```

Relative expiry in seconds from the write; zero means none. Ignored when
`expirationUnixMs` is set.

##### expirationUnixMs

```ts
expirationUnixMs: number;
```

Absolute expiry (UTC Unix milliseconds); zero means none.

##### metadata

```ts
metadata: string;
```

Opaque JSON metadata text, at most `maxKvMetadataBytes`; empty means none.

***

### KvPutOptionsBinding

Options for [KvNamespaceBinding.put](#put-3) (Workers KV names).

#### Properties

##### expiration?

```ts
optional expiration?: number;
```

Absolute expiry in seconds since the Unix epoch.

##### expirationTtl?

```ts
optional expirationTtl?: number;
```

Relative expiry in seconds from the write; ignored when `expiration` is set.

##### metadata?

```ts
optional metadata?: unknown;
```

JSON-serializable metadata, at most `maxKvMetadataBytes` once encoded.

***

### KvValueWithMetadata

Value plus metadata from [KvNamespaceBinding.getWithMetadata](#getwithmetadata).

#### Type Parameters

| Type Parameter |
| ------ |
| `T` |

#### Properties

##### metadata

```ts
metadata: unknown;
```

Stored metadata, or `null` when none.

##### value

```ts
value: T | null;
```

Decoded value, or `null` when the key is absent or expired.

***

### ListDealsParams

Params of `ContentSource.listDeals`.

#### Properties

##### limit?

```ts
optional limit?: number;
```

Maximum number of deals to return; guest default when 0.
Omitted when absent (wire zero value).

***

### ListeningProgress

One listening-progress row. The host upserts into `listening_progress`
tagged with the plugin id; plugins never open the library DB.

#### Properties

##### asin?

```ts
optional asin?: string;
```

Amazon ASIN when known.
Omitted when absent (wire zero value).

##### authors?

```ts
optional authors?: string;
```

Authors string when known.
Omitted when absent (wire zero value).

##### currentTimeSeconds?

```ts
optional currentTimeSeconds?: number;
```

Current playback position in seconds.
Omitted when absent (wire zero value).

##### durationSeconds?

```ts
optional durationSeconds?: number;
```

Total duration in seconds when known.
Omitted when absent (wire zero value).

##### externalItemId

```ts
externalItemId: string;
```

Provider-scoped item / library id.

##### externalUserId

```ts
externalUserId: string;
```

Provider-scoped user id.

##### identityId?

```ts
optional identityId?: bigint;
```

Bookclerk identity row id when already linked.
Omitted when absent (wire zero value).

##### isbn?

```ts
optional isbn?: string;
```

ISBN when known.
Omitted when absent (wire zero value).

##### isFinished

```ts
isFinished: boolean;
```

When true, the provider marks the item finished.

##### lastListenedAtUnixMs?

```ts
optional lastListenedAtUnixMs?: number;
```

Last listen timestamp as unix milliseconds (UTC).
Omitted when absent (wire zero value).

##### progress?

```ts
optional progress?: number;
```

Fractional progress in `0.0..=1.0` when the provider reports it.
Omitted when absent (wire zero value).

##### title?

```ts
optional title?: string;
```

Title text when known.
Omitted when absent (wire zero value).

***

### ListOptions

Paginated list request.

#### Properties

##### cursor?

```ts
optional cursor?: string;
```

##### limit?

```ts
optional limit?: number;
```

##### prefix?

```ts
optional prefix?: string;
```

***

### ListPage

One page of keys.

#### Properties

##### nextCursor?

```ts
optional nextCursor?: string;
```

##### objects

```ts
objects: ObjectInfo[];
```

***

### LoginCompleteParams

Params of `ContentSource.loginComplete`.

#### Properties

##### sessionId

```ts
sessionId: string;
```

Session id previously returned by `loginStart`.

***

### LoginParams

Params of `ContentSource.login` and `ContentSource.loginStart`. Password
sources fill email/password; OAuth sources use callback / external fields.
There is no files-dir root or library DB path -- only `pluginDataDir`.

#### Properties

##### callbackBind?

```ts
optional callbackBind?: string;
```

Bind address for OAuth callback servers (`host:port`). Ignored when
`callbackIpc` is set (host owns the TCP listener).
Omitted when absent (wire zero value).

##### callbackIpc?

```ts
optional callbackIpc?: string;
```

Host-owned callback IPC endpoint the guest must connect to. When set
(with `callbackPublicBase`), the guest must not bind a TCP listener.
Omitted when absent (wire zero value).

##### callbackPublicBase?

```ts
optional callbackPublicBase?: string;
```

Public base URL for the host TCP listener, e.g. `http://127.0.0.1:12345`.
Omitted when absent (wire zero value).

##### email?

```ts
optional email?: string;
```

Account email / username for password logins; empty for pure OAuth.
Omitted when absent (wire zero value).

##### external

```ts
external: boolean;
```

When true, use external / paste-redirect OAuth instead of a local
callback server.

##### extra

```ts
extra: ExtensibleConfig;
```

Store-specific knobs as `application/json`; guests may ignore unknowns.

##### force

```ts
force: boolean;
```

When true, overwrite an existing sealed credential for this account.

##### label?

```ts
optional label?: string;
```

Operator label stored on the account row.
Omitted when absent (wire zero value).

##### marketplace

```ts
marketplace: string;
```

Marketplace / locale for the storefront; empty means the guest default.

##### password?

```ts
optional password?: string;
```

Account password for password logins; never logged; empty for OAuth.
Omitted when absent (wire zero value).

##### pluginDataDir

```ts
pluginDataDir: string;
```

Scoped writable directory for this plugin only (`.../plugins/<id>/data`).

##### responseUrl?

```ts
optional responseUrl?: string;
```

Pre-supplied OAuth redirect URL (paste flow).
Omitted when absent (wire zero value).

##### showQr

```ts
showQr: boolean;
```

Prefer QR output when the guest supports it.

##### timeoutSecs?

```ts
optional timeoutSecs?: number;
```

Seconds to wait for OAuth callback capture; guest default when 0.
Omitted when absent (wire zero value).

***

### LoginResult

Result of `ContentSource.login` / `loginComplete`: account metadata plus
opaque credentials for the host to seal into `encrypted_secrets`
(`provider = plugin id`). Guests never write secrets into the library DB.

#### Properties

##### account

```ts
account: SourceAccount;
```

Account row fields for the host to upsert.

##### credentials?

```ts
optional credentials?: Uint8Array<ArrayBufferLike>;
```

Opaque credential blob the host seals; empty when login only refreshed
metadata. Guests choose the encoding (typically JSON bytes).
Omitted when absent (wire zero value).

***

### LoginStartResult

Result of `ContentSource.loginStart` (interactive OAuth). The operator
opens `url`; `loginComplete` later uses `sessionId`.

#### Properties

##### sessionId

```ts
sessionId: string;
```

Opaque session id for `loginComplete`.

##### url

```ts
url: string;
```

Browser URL the operator should open to complete OAuth.

***

### NamedDatabase

One named `[[databases]]` binding on the granted context: either an
in-isolate [DatabaseBinding](#databasebinding) or the adapter's granted-channel
[GrantedDatabaseTransport](#granteddatabasetransport) the author-side SDK wraps.

#### Properties

##### database?

```ts
optional database?: DatabaseBinding;
```

Ready binding (same isolate).

##### name

```ts
name: string;
```

Binding name (`env.<name>`).

##### transport?

```ts
optional transport?: GrantedDatabaseTransport;
```

Granted transport (crosses the adapter → author RPC boundary).

***

### NamedKv

One granted `[[kv_namespaces]]` binding on the granted context.

#### Properties

##### kv

```ts
kv: KvNamespaceBinding;
```

Ready namespace (crosses the adapter → author RPC boundary).

##### name

```ts
name: string;
```

Binding name (`env.<name>`).

***

### NamedQueue

One granted `[[queues.producers]]` binding on the granted context.

#### Properties

##### name

```ts
name: string;
```

Binding name (`env.<name>`).

##### queue

```ts
queue: QueueBinding;
```

Ready producer (crosses the adapter → author RPC boundary).

***

### NamedSqlType

Column name paired with its resolved type.

#### Properties

##### name

```ts
name: string;
```

Column name.

##### sqlType

```ts
sqlType: "boolean" | "text" | "integer" | "real" | "blob" | "null";
```

Resolved type.

***

### ObjectInfo

Object listing entry.

#### Properties

##### key

```ts
key: string;
```

##### size

```ts
size: number;
```

***

### ObjectMetadata

Metadata without a body.

#### Properties

##### contentType?

```ts
optional contentType?: string;
```

##### etag?

```ts
optional etag?: string;
```

##### key

```ts
key: string;
```

##### sha256?

```ts
optional sha256?: Uint8Array<ArrayBufferLike>;
```

##### size

```ts
size: number;
```

***

### Oidc

`oidc` entrypoint: relying-party client templates and credential
verification on behalf of the host authorization server.

#### Methods

##### authenticateUser()

```ts
authenticateUser(params): Promise<ExternalUserReply>;
```

Verify remote credentials on behalf of the host.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`AuthenticateUserParams`](#authenticateuserparams) | Username and password. |

###### Returns

`Promise`\<[`ExternalUserReply`](#externaluserreply)\>

[ExternalUserReply](#externaluserreply)

##### clients()

```ts
clients(): Promise<OidcClientsReply>;
```

Plugin-provided OIDC AS client templates. Empty list when unused.

###### Returns

`Promise`\<[`OidcClientsReply`](#oidcclientsreply)\>

[OidcClientsReply](#oidcclientsreply)

***

### OidcClientsOk

Success payload of `Oidc.clients`.

#### Properties

##### clients

```ts
clients: OidcClientTemplate[];
```

Client templates; empty when the plugin is not a relying party.

***

### OidcClientTemplate

Bookclerk-as-IdP relying-party template. Plugins declare callback path and
client id; the host materializes `oidc_clients` rows and remains the AS.
`originConfigKey` is a dotted config path (e.g. integrations.audiobookshelf.base_url).

#### Properties

##### callbackPath

```ts
callbackPath: string;
```

Redirect URI path relative to the integration origin.

##### clientId

```ts
clientId: string;
```

OIDC client id the host materializes.

##### defaultScopes

```ts
defaultScopes: string[];
```

Scopes granted by default.

##### displayName

```ts
displayName: string;
```

Name shown on consent / admin screens.

##### issueRefreshToken

```ts
issueRefreshToken: boolean;
```

When true, the AS issues refresh tokens to this client.

##### originConfigKey

```ts
originConfigKey: string;
```

Dotted config path holding the client's origin URL.

##### publicClient

```ts
publicClient: boolean;
```

When true, the client uses PKCE without a client secret.

***

### OpenOk

Success payload of `Source.open`.

#### Properties

##### body?

```ts
optional body?: ByteSource | null;
```

Streamed object bytes.
Null when the guest does not export this capability.

##### meta

```ts
meta: ObjectMetadata;
```

Object metadata.

***

### PhysicalAccess

Physical table/column access used for authorization.

#### Properties

##### column

```ts
column: string;
```

Empty = table presence only; "*" = projection wildcard.

##### table

```ts
table: string;
```

Physical table name.

***

### PlainFetch

Plain (DRM-free) fetch result. Sources always return decrypted media; DRM
guests decrypt before responding.

#### Properties

##### chapters

```ts
chapters: ChapterMarker[];
```

Chapter markers; empty when unknown.

##### coverPath?

```ts
optional coverPath?: string;
```

Cover image path under the cache directory.
Omitted when absent (wire zero value).

##### m4bPath?

```ts
optional m4bPath?: string;
```

Single M4B path when the guest assembled one.
Omitted when absent (wire zero value).

##### parts

```ts
parts: PlainPart[];
```

Ordered audio part files written under the cache directory.

##### pdfUrl?

```ts
optional pdfUrl?: string;
```

Companion PDF download URL when the store exposes one.
Omitted when absent (wire zero value).

***

### PlainPart

One plain audio part written under the cache directory.

#### Properties

##### durationMs?

```ts
optional durationMs?: number;
```

Duration of this part in milliseconds when known.
Omitted when absent (wire zero value).

##### path

```ts
path: string;
```

Absolute path to the part file under `cacheDir`.

##### title?

```ts
optional title?: string;
```

Part title (disc / chapter label).
Omitted when absent (wire zero value).

***

### PluginCapabilities

Typed capability declaration returned by `describe()`. The host compares
it with `plugin.toml` and the operator grant; widening is rejected at spawn.

#### Properties

##### bindings

```ts
bindings: string[];
```

Other named bindings the guest expects on `env` (`CONFIG`, `SECRETS`,
`WORK_FS`, `OAUTH`, `KV`, `EVENTS`, `[[queues.producers]]` names, ...).

##### consumes

```ts
consumes: EventConsumerSpec[];
```

Event types the default entrypoint consumes (`event(batch)` trigger).

##### databases

```ts
databases: string[];
```

Plugin-owned database binding names (`[[databases]]`).

##### entrypoints

```ts
entrypoints: (
  | "storefront"
  | "storage"
  | "databaseAdapter"
  | "remoteLibrary"
  | "cli"
  | "oidc")[];
```

Named entrypoints the guest exports.

##### jobs

```ts
jobs: string[];
```

Command types the default entrypoint runs (`job(controller)` trigger).

##### produces

```ts
produces: string[];
```

Event types the guest may publish through its `EVENTS` binding.

##### queueConsumers

```ts
queueConsumers: string[];
```

Queues the default entrypoint consumes (`queue(batch)` trigger).

##### queueProducers

```ts
queueProducers: string[];
```

Queues the guest may send to through its `[[queues.producers]]` bindings.

***

### PluginCli

`cli` entrypoint: guest commands under `bookclerk plugins <id> <command>`.

#### Methods

##### describe()

```ts
describe(): Promise<CliSchemaReply>;
```

Declared CLI surface.

###### Returns

`Promise`\<[`CliSchemaReply`](#clischemareply)\>

[CliSchemaReply](#clischemareply)

##### invoke()

```ts
invoke(params): Promise<CliInvokeReply>;
```

Run one plugin CLI command.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`CliInvokeParams`](#cliinvokeparams) | Command name and argument values. |

###### Returns

`Promise`\<[`CliInvokeReply`](#cliinvokereply)\>

[CliInvokeReply](#cliinvokereply)

***

### PluginEvent

Guest-published domain event (`EventPublisher.publish`). The host stamps
`eventId`, `source` (the plugin id), and `accountId` from the invocation;
guests cannot forge either.

#### Properties

##### causationId

```ts
causationId: string;
```

Id of the event or command that caused this one; empty inherits
`Invocation.causationId`.

##### correlationId

```ts
correlationId: string;
```

Trace correlation id; empty inherits `Invocation.correlationId`.

##### deduplicationKey

```ts
deduplicationKey: string;
```

Producer idempotency key, unique per (account, source, eventType) in the
outbox; a repeat returns `PublishOk.duplicate = true` with the earlier id.
Empty publishes unconditionally.

##### eventType

```ts
eventType: string;
```

Snake_case event type; must be listed in `[[events.producers]]`.

##### occurredAtUnixMs

```ts
occurredAtUnixMs: number;
```

When the producer observed the fact; zero means "now" on the host clock.

##### payload

```ts
payload: Uint8Array;
```

Encoded event payload; at most `maxEventPayloadBytes`.

##### schemaVersion

```ts
schemaVersion: number;
```

Schema version of `payload`, owned by the event type.

***

### PluginMigration

One plugin-owned migration application. `id` is an opaque plugin-chosen
stable identity (name, UUID, timestamp-like string, or digits-as-text).
Bookclerk assigns no order, version, or predecessor meaning to `id`.
Registration order is the forward sequence.

#### Properties

##### id

```ts
id: string;
```

Opaque plugin-chosen stable identity.

##### operations

```ts
operations: PluginMigrationOp[];
```

Operations in forward order; at most `maxPluginMigrationOps`.

***

### PluginMigrationsOk

Success payload of `PluginWorker.databaseMigrations`.

#### Properties

##### migrations

```ts
migrations: PluginMigration[];
```

At most `maxListPage` entries; aggregate id+SQL bytes at most
`maxPluginMigrationRegistrationBytes`; total operations at most
`maxPluginMigrationTotalOps`.

***

### PluginWorker

Plugin bootstrap capability: the guest's root object (Workers `default`
export analogue). `describe()` first; `open()` once per invocation.

#### Methods

##### databaseMigrations()

```ts
databaseMigrations(binding): Promise<PluginMigrationsReply>;
```

Complete ordered plugin-owned migration sequence for one named binding.
The host calls this before `open` while provisioning `[[databases]]`
(a binding must be migrated before any `Bindings.databases` session is
handed out). Empty list means the binding has no plugin-owned migrations.
Bounded by `maxListPage` / `maxPluginMigrationOps` /
`maxPluginMigrationTotalOps` / `maxScalarBytes` /
`maxPluginMigrationRegistrationBytes`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `binding` | `string` | Binding name from `plugin.toml`. |

###### Returns

`Promise`\<[`PluginMigrationsReply`](#pluginmigrationsreply)\>

[PluginMigrationsReply](#pluginmigrationsreply)

##### describe()

```ts
describe(): Promise<DescribeReply>;
```

Identity, ABI version, negotiated features, and typed capabilities.

###### Returns

`Promise`\<[`DescribeReply`](#describereply)\>

[DescribeReply](#describereply)

##### open()

```ts
open(invocation, bindings): Promise<EntrypointsReply>;
```

Open the exported entrypoints for one invocation with host-granted
bindings. Entrypoints the manifest does not export are null.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `invocation` | [`Invocation`](#invocation-11) | Host-issued invocation identity. |
| `bindings` | [`Bindings`](#bindings) | Granted `env` bindings. |

###### Returns

`Promise`\<[`EntrypointsReply`](#entrypointsreply)\>

[EntrypointsReply](#entrypointsreply)

##### shutdown()

```ts
shutdown(): Promise<EmptyReply>;
```

Flush and release resources before the process exits.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

***

### PreparedStatement

Cloudflare-style prepared statement. Kind, bounds, and request hash are
derived by the trusted host. Only universal [DbValue](#dbvalue)s are public.

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

`Promise`\<[`D1Result`](#d1result)\<`Record`\<`string`, [`DbValue`](#dbvalue)\>\>\>

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
| ...`values` | [`DbValue`](#dbvalue)[] | Universal `DbValue`s only. |

###### Returns

[`PreparedStatement`](#preparedstatement)

##### first()

```ts
first(colName?, options?): Promise<DbValue | Record<string, DbValue> | null>;
```

First row as a name→value map, or one column when `colName` is set.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `colName?` | `string` | Optional column name (Cloudflare `first(colName)`). |
| `options?` | \{ `retry?`: [`RetryToken`](#retrytoken); \} | Optional retry token. |
| `options.retry?` | [`RetryToken`](#retrytoken) | - |

###### Returns

`Promise`\<[`DbValue`](#dbvalue) \| `Record`\<`string`, [`DbValue`](#dbvalue)\> \| `null`\>

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

`Promise`\<[`DbValue`](#dbvalue)[][]\>

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

`Promise`\<[`D1Result`](#d1result)\<`Record`\<`string`, [`DbValue`](#dbvalue)\>\>\>

***

### PublishOk

Success payload of `EventPublisher.publish`.

#### Properties

##### duplicate

```ts
duplicate: boolean;
```

True when `deduplicationKey` matched an existing outbox row.

##### eventId

```ts
eventId: string;
```

Outbox event id (new or the earlier row when `duplicate`).

***

### PullOk

Success payload of `ByteSource.pull`.

#### Properties

##### chunk

```ts
chunk: Uint8Array;
```

Next bytes; may be shorter than requested.

##### done

```ts
done: boolean;
```

True when the stream is exhausted after `chunk`.

***

### PurchaseHint

Purchase hint for SPA / CLI purchase deep-links.

#### Properties

##### currency?

```ts
optional currency?: string;
```

ISO currency code for price fields.
Omitted when absent (wire zero value).

##### listPriceCents?

```ts
optional listPriceCents?: bigint;
```

List / MSRP price in minor units.
Omitted when absent (wire zero value).

##### listPriceLabel?

```ts
optional listPriceLabel?: string;
```

Pre-formatted list price.
Omitted when absent (wire zero value).

##### memberPriceCents?

```ts
optional memberPriceCents?: bigint;
```

Member / Plus price in minor units.
Omitted when absent (wire zero value).

##### memberPriceLabel?

```ts
optional memberPriceLabel?: string;
```

Pre-formatted member price.
Omitted when absent (wire zero value).

##### priceCents?

```ts
optional priceCents?: bigint;
```

Current price in minor units.
Omitted when absent (wire zero value).

##### priceLabel?

```ts
optional priceLabel?: string;
```

Pre-formatted current price.
Omitted when absent (wire zero value).

##### productId

```ts
productId: string;
```

Storefront product id.

##### title?

```ts
optional title?: string;
```

Title when resolved.
Omitted when absent (wire zero value).

##### url?

```ts
optional url?: string;
```

Purchase or product-page URL.
Omitted when absent (wire zero value).

***

### PurchaseHintParams

Params of `ContentSource.purchaseHint`. At least one identity field
(`productId` / `asin` / `isbn` / title+authors) should be set; guests may
return `invalid_params` when none are usable.

#### Properties

##### asin?

```ts
optional asin?: string;
```

Amazon ASIN when known.
Omitted when absent (wire zero value).

##### authors?

```ts
optional authors?: string;
```

Authors string for fuzzy lookup.
Omitted when absent (wire zero value).

##### isbn?

```ts
optional isbn?: string;
```

ISBN when known.
Omitted when absent (wire zero value).

##### productId?

```ts
optional productId?: string;
```

Storefront product id when known.
Omitted when absent (wire zero value).

##### region

```ts
region: string;
```

Storefront region / marketplace code.

##### title?

```ts
optional title?: string;
```

Title text for fuzzy lookup.
Omitted when absent (wire zero value).

##### withPrice

```ts
withPrice: boolean;
```

When true, guests should include live price fields when available.

***

### PurchaseHintResult

Success payload of `ContentSource.purchaseHint`.

#### Properties

##### found

```ts
found: boolean;
```

False when the guest could not resolve the title (`hint` is empty).

##### hint

```ts
hint: PurchaseHint;
```

Purchase hint when `found`.

***

### PutResult

Result of a streamed put.

#### Properties

##### bytesWritten

```ts
bytesWritten: number;
```

##### etag?

```ts
optional etag?: string;
```

##### key

```ts
key: string;
```

##### sha256?

```ts
optional sha256?: Uint8Array<ArrayBufferLike>;
```

***

### QueueBatchOk

Success payload of `QueueConsumer.queue`: exactly one entry per
`QueueBatch.messages` entry, in order. A short list is a host-side error
and the missing tail is retried.

#### Properties

##### results

```ts
results: QueueResult[];
```

Per-message outcomes in delivery order.

***

### QueueBinding

`[[queues.producers]]` binding (`env.<BINDING>`): Workers `Queue`-shaped
producer into the queue the row names. The host assigns ids, stamps the
sending plugin as `source`, and bounds each body at `maxEventPayloadBytes`.

#### Methods

##### send()

```ts
send(body, options?): Promise<QueueSendOk>;
```

Send one message.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | `unknown` | JSON value (default), string, or bytes per `contentType`. |
| `options?` | [`QueueSendOptions`](#queuesendoptions) | Encoding and delay. |

###### Returns

`Promise`\<[`QueueSendOk`](#queuesendok)\>

The outbox id the host assigned.

##### sendBatch()

```ts
sendBatch(messages, options?): Promise<QueueSendOk>;
```

Send up to `maxListPage` messages in order.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `messages` | readonly [`QueueSendBatchEntry`](#queuesendbatchentry)[] | Bodies with per-message options. |
| `options?` | `Pick`\<[`QueueSendOptions`](#queuesendoptions), `"delaySeconds"`\> | Default delay for entries that carry none. |

###### Returns

`Promise`\<[`QueueSendOk`](#queuesendok)\>

The outbox ids in send order.

***

### QueueConsumer

`[[queues.consumers]]` handler (`Entrypoints.queueConsumer`). Delivery is
at-least-once; consumers must be idempotent on `QueueMessage.id`.

#### Methods

##### queue()

```ts
queue(batch): Promise<QueueBatchReply>;
```

Deliver one ordered batch of messages from one queue.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `batch` | `QueueBatch` | Messages in delivery order. |

###### Returns

`Promise`\<[`QueueBatchReply`](#queuebatchreply)\>

[QueueBatchReply](#queuebatchreply)

***

### QueueProducer

Queue producer binding: host-served send into the queue a
`[[queues.producers]]` row names, granted through `Bindings.queues`.

#### Methods

##### send()

```ts
send(message): Promise<QueueSendReply>;
```

Send one message.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `message` | [`QueueSendMessage`](#queuesendmessage) | Body, encoding, and delay. |

###### Returns

`Promise`\<[`QueueSendReply`](#queuesendreply)\>

[QueueSendReply](#queuesendreply)

##### sendBatch()

```ts
sendBatch(messages, delaySeconds): Promise<QueueSendReply>;
```

Send up to `maxListPage` messages in order; `delaySeconds` applies to
every message that carries no delay of its own.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `messages` | [`QueueSendMessage`](#queuesendmessage)[] | Messages in send order. |
| `delaySeconds` | `number` | Default delay for the batch. |

###### Returns

`Promise`\<[`QueueSendReply`](#queuesendreply)\>

[QueueSendReply](#queuesendreply)

***

### QueueRetry

Redeliver a queue message later.

#### Properties

##### delaySeconds

```ts
delaySeconds: number;
```

Seconds before redelivery; zero applies the consumer's `retry_delay`.

##### reason

```ts
reason: string;
```

Short human reason.

***

### QueueRetryOptions

Options for [QueueMessage.retry](#retry-1).

#### Properties

##### delaySeconds?

```ts
optional delaySeconds?: number;
```

Seconds before redelivery; `0` applies the consumer's `retry_delay`.

##### reason?

```ts
optional reason?: string;
```

Short operator-facing reason.

***

### QueueSendBatchEntry

One entry of [QueueBinding.sendBatch](#sendbatch).

#### Extends

- [`QueueSendOptions`](#queuesendoptions)

#### Properties

##### body

```ts
body: unknown;
```

Message body (JSON value, string, or bytes per `contentType`).

##### contentType?

```ts
optional contentType?: "text" | "bytes" | "json";
```

Body encoding: `json` (default; the body is JSON-encoded), `text`
(UTF-8 string), or `bytes` (raw `Uint8Array`).

###### Inherited from

[`QueueSendOptions`](#queuesendoptions).[`contentType`](#contenttype-4)

##### delaySeconds?

```ts
optional delaySeconds?: number;
```

Seconds before the message may be delivered; `0` means now.

###### Inherited from

[`QueueSendOptions`](#queuesendoptions).[`delaySeconds`](#delayseconds-4)

***

### QueueSendMessage

One message to send through a `[[queues.producers]]` binding.

#### Properties

##### body

```ts
body: Uint8Array;
```

Encoded body; at most `maxEventPayloadBytes`.

##### contentType

```ts
contentType: string;
```

Body encoding: `json` (default when empty), `text`, or `bytes`.

##### delaySeconds

```ts
delaySeconds: number;
```

Seconds before the message may be delivered; zero means now.

***

### QueueSendOk

Success payload of `QueueProducer.send` / `sendBatch`.

#### Properties

##### messageIds

```ts
messageIds: string[];
```

Outbox row ids in send order.

***

### QueueSendOptions

Per-send options for a [QueueBinding](#queuebinding).

#### Extended by

- [`QueueSendBatchEntry`](#queuesendbatchentry)

#### Properties

##### contentType?

```ts
optional contentType?: "text" | "bytes" | "json";
```

Body encoding: `json` (default; the body is JSON-encoded), `text`
(UTF-8 string), or `bytes` (raw `Uint8Array`).

##### delaySeconds?

```ts
optional delaySeconds?: number;
```

Seconds before the message may be delivered; `0` means now.

***

### ReadOptions

Read options for [StorageEntrypoint.get](#get-1).

#### Properties

##### range?

```ts
optional range?: ByteRange;
```

***

### ReadResult

Streamed read result. `body` is a transferred ReadableStream.

#### Properties

##### body

```ts
body: ReadableStream<Uint8Array<ArrayBufferLike>>;
```

##### meta

```ts
meta: ObjectMetadata;
```

***

### RejectedOutcome

Job failed permanently; no retry.

#### Properties

##### message

```ts
message: string;
```

Short human reason.

***

### RemoteLibrary

`remoteLibrary` entrypoint: long-running remote-library lifecycle (start /
stop, library rescan, listening sync, external-user polling). Event delivery
is `EventConsumer`; credential verification is `Oidc`.

#### Methods

##### diagnose()

```ts
diagnose(): Promise<DiagnoseReply>;
```

Human-readable diagnostic lines.

###### Returns

`Promise`\<[`DiagnoseReply`](#diagnosereply)\>

[DiagnoseReply](#diagnosereply)

##### health()

```ts
health(): Promise<HealthReply>;
```

Liveness / readiness probe.

###### Returns

`Promise`\<[`HealthReply`](#healthreply)\>

[HealthReply](#healthreply)

##### pollEvents()

```ts
pollEvents(): Promise<EventPollReply>;
```

Drain external users the remote side observed since the last poll.

###### Returns

`Promise`\<[`EventPollReply`](#eventpollreply)\>

[EventPollReply](#eventpollreply)

##### scanLibrary()

```ts
scanLibrary(params): Promise<EmptyReply>;
```

Re-sync the remote library.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`ScanLibraryParams`](#scanlibraryparams) | Full-rescan flag. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### start()

```ts
start(): Promise<EmptyReply>;
```

Start background work after the host has granted bindings.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### stop()

```ts
stop(): Promise<EmptyReply>;
```

Stop background work; the host may drop the capability afterwards.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### syncListening()

```ts
syncListening(): Promise<SyncListeningReply>;
```

Push / pull listening progress.

###### Returns

`Promise`\<[`SyncListeningReply`](#synclisteningreply)\>

[SyncListeningReply](#synclisteningreply)

***

### ResolvedAssignment

Destination assignment `lhs = rhs` (INSERT, UPDATE, ...).

#### Properties

##### column

```ts
column: string;
```

Target column.

##### dest

```ts
dest: "boolean" | "text" | "integer" | "real" | "blob" | "null";
```

Declared column type.

##### source

```ts
source: "boolean" | "text" | "integer" | "real" | "blob" | "null";
```

Resolved type of the assigned expression.

##### table

```ts
table: string;
```

Target table.

***

### ResolvedStatement

Host-produced typed proof bound to one exact canonical statement.

#### Properties

##### assignments

```ts
assignments: ResolvedAssignment[];
```

Mutation destination assignments.

##### functions

```ts
functions: string[];
```

Function names invoked (folded), for authorization.

##### integerArithSites

```ts
integerArithSites: IntegerArithSite[];
```

INTEGER overflow sites (`+` `-` `*` `abs`).

##### outputColumns

```ts
outputColumns: NamedSqlType[];
```

SELECT / RETURNING / VALUES output columns in order.

##### physicalAccesses

```ts
physicalAccesses: PhysicalAccess[];
```

Physical tables/columns referenced (authorization).

##### schemaAction

```ts
schemaAction: SchemaAction;
```

DDL action + fingerprint.

##### statementHash

```ts
statementHash: string;
```

SHA-256 hex of the exact canonical SQL this proof claims.

##### textCollateSites

```ts
textCollateSites: TextCollateSite[];
```

TEXT expression spans needing bytewise collation.

***

### RetryableOutcome

Job failed transiently; the host reschedules it.

#### Properties

##### message

```ts
message: string;
```

Short human reason.

##### retryAfterUnixMs

```ts
retryAfterUnixMs: number;
```

Earliest retry time; `0` lets the host choose.

***

### RetryOptions

Options for [EventMessage.retry](#retry).

#### Properties

##### delaySeconds?

```ts
optional delaySeconds?: number;
```

Delay from now, in seconds; ignored when `retryAt` is set.

##### reason?

```ts
optional reason?: string;
```

Short operator-facing reason.

##### retryAt?

```ts
optional retryAt?: Instant;
```

Explicit earliest redelivery instant.

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

### ScalarLimits

Guest-advertised caps for `rpc.scalarLimits`; never above the file constants.

#### Properties

##### maxListPage

```ts
maxListPage: number;
```

Largest list page; at most `maxListPage`.

##### maxScalarBytes

```ts
maxScalarBytes: number;
```

Largest scalar the guest accepts; at most `maxScalarBytes`.

##### maxStreamWindowBytes

```ts
maxStreamWindowBytes: number;
```

Largest `ByteSource.pull` window; at most `maxStreamWindowBytes`.

***

### ScanBook

One library title returned by `ContentSource.scan`. The host upserts these
rows and forces `source` to the plugin id.

#### Properties

##### accountId

```ts
accountId: string;
```

Account that owns this library entry.

##### asin?

```ts
optional asin?: string;
```

Amazon ASIN when the storefront exposes one.
Omitted when absent (wire zero value).

##### authors?

```ts
optional authors?: string;
```

Comma- or guest-formatted author list.
Omitted when absent (wire zero value).

##### contentKind?

```ts
optional contentKind?: string;
```

Content classification (e.g. `book` vs `episode`).
Omitted when absent (wire zero value).

##### isbn?

```ts
optional isbn?: string;
```

ISBN when the storefront exposes one.
Omitted when absent (wire zero value).

##### lengthMinutes?

```ts
optional lengthMinutes?: bigint;
```

Runtime in whole minutes when known.
Omitted when absent (wire zero value).

##### marketplace?

```ts
optional marketplace?: string;
```

Marketplace / region when known.
Omitted when absent (wire zero value).

##### narrators?

```ts
optional narrators?: string;
```

Comma- or guest-formatted narrator list.
Omitted when absent (wire zero value).

##### productId

```ts
productId: string;
```

Storefront product / SKU id.

##### publisher?

```ts
optional publisher?: string;
```

Publisher name when known.
Omitted when absent (wire zero value).

##### series?

```ts
optional series?: string;
```

Series name when applicable.
Omitted when absent (wire zero value).

##### seriesIndex?

```ts
optional seriesIndex?: string;
```

Series index / sequence label.
Omitted when absent (wire zero value).

##### subtitle?

```ts
optional subtitle?: string;
```

Subtitle when distinct from `title`.
Omitted when absent (wire zero value).

##### title

```ts
title: string;
```

Primary title string.

***

### ScanLibraryParams

Params of `RemoteLibrary.scanLibrary` (remote library sync).

#### Properties

##### force

```ts
force: boolean;
```

When true, force a full rescan even if the guest would otherwise
incremental-sync.

***

### ScanParams

Params of `ContentSource.scan`. The host injects sealed credentials so the
plugin does not need a private credential store under `pluginDataDir`.

#### Properties

##### accounts

```ts
accounts: string[];
```

Account ids to scan; empty means all scan-enabled accounts.

##### credentials

```ts
credentials: AccountCredential[];
```

Host-loaded credential blobs for the requested accounts.

##### importEpisodes

```ts
importEpisodes: boolean;
```

When true, import podcast/episode-style rows.

##### importPlusTitles

```ts
importPlusTitles: boolean;
```

When true, import Plus/catalog entitlement titles.

##### pageSize

```ts
pageSize: number;
```

Storefront page size; the host always sends an explicit value.

##### pluginDataDir

```ts
pluginDataDir: string;
```

Scoped plugin data directory.

***

### ScanSummary

Summary result of `ContentSource.scan`.

#### Properties

##### accounts

```ts
accounts: number;
```

Number of accounts touched during the scan.

##### books

```ts
books: ScanBook[];
```

Titles for the host to upsert. Prefer this over plugin-side DB writes.

##### booksUpserted

```ts
booksUpserted: number;
```

Count of titles the guest expects the host to upsert; may mirror
`books.length`.

##### pages

```ts
pages: number;
```

Number of storefront pages fetched.

##### skippedDisabled

```ts
skippedDisabled: number;
```

Accounts skipped because `scanEnabled` was false.

***

### SchemaCreate

`CREATE TABLE` action with its durable fingerprint.

#### Properties

##### fingerprint

```ts
fingerprint: string;
```

Structured schema fingerprint (hex SHA-256).

##### noop

```ts
noop: boolean;
```

True when the catalog already holds this exact fingerprint.

##### schema

```ts
schema: CreateTableSchema;
```

Parsed table schema.

***

### SearchCatalogParams

Params of `ContentSource.searchCatalog`.

#### Properties

##### field

```ts
field: "author" | "any" | "narrator" | "series" | "genre";
```

Facet restriction.

##### language?

```ts
optional language?: string;
```

Preferred content language (soft-prioritize; e.g. `en`).
Omitted when absent (wire zero value).

##### limit

```ts
limit: number;
```

Maximum hits to return; the host always sends an explicit value.

##### page

```ts
page: number;
```

1-based page for storefronts that page.

##### query

```ts
query: string;
```

Free-text search query.

##### region

```ts
region: string;
```

Storefront region / marketplace code; empty means the guest default.

##### sort

```ts
sort: "relevance" | "popularity" | "rating" | "title" | "author";
```

Sort order.

***

### SourceAccount

Source account metadata returned from login and stored by the host.

#### Properties

##### accountId

```ts
accountId: string;
```

Stable account id within this source plugin.

##### label?

```ts
optional label?: string;
```

Operator-facing label.
Omitted when absent (wire zero value).

##### marketplace

```ts
marketplace: string;
```

Storefront marketplace / region code (for example `us`, `uk`).

##### scanEnabled

```ts
scanEnabled: boolean;
```

When true, bare / scheduled scans include this account. Explicit CLI
`--account` bypasses this flag.

##### source

```ts
source: string;
```

Source plugin id (the host forces this to the guest's install id).

***

### SourceAccounts

Success payload of `ContentSource.listAccounts`.

#### Properties

##### accounts

```ts
accounts: SourceAccount[];
```

Accounts the guest knows about.

***

### SqlSpan

Byte span in the exact canonical SQL string a proof is bound to.

#### Properties

##### end

```ts
end: number;
```

Exclusive end byte offset.

##### start

```ts
start: number;
```

Inclusive start byte offset.

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

### StorageBinding

`WORK_FS` binding: host-granted object storage for durable plugin files.
Same surface as the `storage` entrypoint the host calls on storage plugins.

#### Methods

##### delete()

```ts
delete(key): Promise<void>;
```

Remove an object; missing keys are not an error.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key. |

###### Returns

`Promise`\<`void`\>

Resolves when the delete is durable.

##### get()

```ts
get(key, options?): Promise<{
  body: ReadableStream<Uint8Array<ArrayBufferLike>>;
  meta: {
     key: string;
     size: number;
  };
}>;
```

Stream an object (optionally a byte range).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key. |
| `options?` | \{ `range?`: \{ `length?`: `number`; `offset`: `number`; \}; \} | Optional byte range. |
| `options.range?` | \{ `length?`: `number`; `offset`: `number`; \} | - |
| `options.range.length?` | `number` | - |
| `options.range.offset?` | `number` | - |

###### Returns

`Promise`\<\{
  `body`: `ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\>;
  `meta`: \{
     `key`: `string`;
     `size`: `number`;
  \};
\}\>

Metadata and the body stream.

##### head()

```ts
head(key): Promise<
  | {
  contentType?: string;
  etag?: string;
  key: string;
  size: number;
}
| null>;
```

Object metadata without the body.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key. |

###### Returns

`Promise`\<
  \| \{
  `contentType?`: `string`;
  `etag?`: `string`;
  `key`: `string`;
  `size`: `number`;
\}
  \| `null`\>

Metadata, or `null` when the key does not exist.

##### list()

```ts
list(options): Promise<{
  nextCursor?: string;
  objects: object[];
}>;
```

One page of keys under `prefix`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options` | \{ `cursor?`: `string`; `limit?`: `number`; `prefix?`: `string`; \} | Prefix, continuation cursor, and page size. |
| `options.cursor?` | `string` | - |
| `options.limit?` | `number` | - |
| `options.prefix?` | `string` | - |

###### Returns

`Promise`\<\{
  `nextCursor?`: `string`;
  `objects`: `object`[];
\}\>

Objects plus the cursor for the next page, when any.

##### put()

```ts
put(
   key, 
   body, 
   options?
): Promise<{
  bytesWritten: number;
  key: string;
}>;
```

Write an object from a stream.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key. |
| `body` | `ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\> | Body bytes. |
| `options?` | \{ `contentLength?`: `number`; `contentType?`: `string`; \} | Content type and length hints. |
| `options.contentLength?` | `number` | - |
| `options.contentType?` | `string` | - |

###### Returns

`Promise`\<\{
  `bytesWritten`: `number`;
  `key`: `string`;
\}\>

Key and bytes written.

***

### SuspendedOutcome

Job released the process and asks to be resumed later.

#### Properties

##### checkpointJson

```ts
checkpointJson: string;
```

Bounded checkpoint to replay on resume; at most `maxCheckpointBytes`.

##### checkpointSchemaVersion

```ts
checkpointSchemaVersion: number;
```

Schema version of `checkpointJson`.

##### wakeAtUnixMs

```ts
wakeAtUnixMs: number;
```

Earliest resume time.

***

### SyncListeningResult

Success payload of `RemoteLibrary.syncListening`.

#### Properties

##### items

```ts
items: ListeningProgress[];
```

Progress snapshots to upsert.

***

### TextCollateSite

One TEXT expression the adapter must collate bytewise (`COLLATE "C"`).

#### Properties

##### span

```ts
span: SqlSpan;
```

Identifier or string-literal span in canonical SQL.

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

### WireEventBatch

Wire `EventBatch` as the adapter delivers it to `bookclerkEvent`.

#### Properties

##### events

```ts
events: Partial<DomainEvent>[];
```

***

### WireQueueBatch

Wire `QueueBatch` as the adapter delivers it to `bookclerkQueue`.

#### Properties

##### messages

```ts
messages: Partial<QueueMessage>[];
```

##### queue

```ts
queue: string;
```

***

### WriteOptions

Write options for [StorageEntrypoint.put](#put-1).

#### Properties

##### commitToken?

```ts
optional commitToken?: string;
```

##### contentLength?

```ts
optional contentLength?: number;
```

##### contentType?

```ts
optional contentType?: string;
```

##### sha256?

```ts
optional sha256?: Uint8Array<ArrayBufferLike>;
```

##### stageOnly?

```ts
optional stageOnly?: boolean;
```

## Type Aliases

### Abridgement

```ts
type Abridgement = typeof ABRIDGEMENTS[number];
```

Union of `ABRIDGEMENTS` wire names.

***

### AdapterSessionReply

```ts
type AdapterSessionReply = 
  | {
  kind: "ok";
  value: AdapterDatabaseSession;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Database.openSession`.

***

### AuthorStub

```ts
type AuthorStub = Pick<BookclerkEntrypoint, 
  | "bookclerkDescribe"
  | "bookclerkEvent"
  | "bookclerkQueue"
  | "bookclerkJob"
  | "bookclerkDatabaseMigrations"
| "bookclerkShutdown">;
```

Author default entrypoint as seen through the `PLUGIN` service binding.

***

### CatalogDetailReply

```ts
type CatalogDetailReply = 
  | {
  kind: "ok";
  value: CatalogDetail;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.catalogDetail`.

***

### CatalogField

```ts
type CatalogField = typeof CATALOG_FIELDS[number];
```

Union of `CATALOG_FIELDS` wire names.

***

### CatalogHitsReply

```ts
type CatalogHitsReply = 
  | {
  kind: "ok";
  value: CatalogHits;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `searchCatalog` / `expandCandidates` / `listDeals`.

***

### CatalogSort

```ts
type CatalogSort = typeof CATALOG_SORTS[number];
```

Union of `CATALOG_SORTS` wire names.

***

### Checkpoint

```ts
type Checkpoint = string | JsonObject | unknown[] | number | boolean | null;
```

JSON-serializable checkpoint accepted by `suspend()`.

***

### CliArgKind

```ts
type CliArgKind = typeof CLI_ARG_KINDS[number];
```

Union of `CLI_ARG_KINDS` wire names.

***

### CliInvokeReply

```ts
type CliInvokeReply = 
  | {
  kind: "ok";
  value: CliInvokeResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `PluginCli.invoke`.

***

### CliSchemaReply

```ts
type CliSchemaReply = 
  | {
  kind: "ok";
  value: CliSchema;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `PluginCli.describe`.

***

### CopyReply

```ts
type CopyReply = 
  | {
  kind: "ok";
  value: CopyResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.copy`.

***

### DbBootstrapReply

```ts
type DbBootstrapReply = 
  | {
  kind: "ok";
  value: DbBootstrap;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `AdapterDatabaseSession.bootstrap`.

***

### DbCapabilitiesReply

```ts
type DbCapabilitiesReply = 
  | {
  kind: "ok";
  value: DbCapabilities;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `AdapterDatabaseSession.capabilities`.

***

### DbResultSelection

```ts
type DbResultSelection = typeof DB_RESULT_SELECTIONS[number];
```

Union of `DB_RESULT_SELECTIONS` wire names.

***

### DbStatementKind

```ts
type DbStatementKind = typeof DB_STATEMENT_KINDS[number];
```

Union of `DB_STATEMENT_KINDS` wire names.

***

### DbType

```ts
type DbType = typeof DB_TYPES[number];
```

Union of `DB_TYPES` wire names.

***

### DbValue

```ts
type DbValue = 
  | {
  kind: "null";
  value: DbType;
}
  | {
  kind: "boolean";
  value: boolean;
}
  | {
  kind: "int64";
  value: bigint;
}
  | {
  kind: "float64";
  value: number;
}
  | {
  kind: "text";
  value: string;
}
  | {
  kind: "bytes";
  value: Uint8Array;
};
```

Closed Cap'n `DbValue` union.

Members are typed null, bool, int64 (`bigint`), finite float64, UTF-8 text
(U+0000 forbidden), and bytes (`Uint8Array`, 0x00 allowed). Unknown `kind`
values fail closed.

***

### DescribeReply

```ts
type DescribeReply = 
  | {
  kind: "ok";
  value: PluginDescribe;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `PluginWorker.describe`.

***

### DiagnoseReply

```ts
type DiagnoseReply = 
  | {
  kind: "ok";
  value: DiagnoseResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `diagnose`.

***

### EmptyReply

```ts
type EmptyReply = 
  | {
  kind: "ok";
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `methods without a success payload`.

***

### Entrypoint

```ts
type Entrypoint = typeof ENTRYPOINTS[number];
```

Union of `ENTRYPOINTS` wire names.

***

### EntrypointName

```ts
type EntrypointName = 
  | "storefront"
  | "storage"
  | "databaseAdapter"
  | "remoteLibrary"
  | "cli"
  | "oidc";
```

Wire name of each named entrypoint (`plugin.toml` `entrypoints`).

***

### EntrypointsReply

```ts
type EntrypointsReply = 
  | {
  kind: "ok";
  value: Entrypoints;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `PluginWorker.open`.

***

### EventBatchReply

```ts
type EventBatchReply = 
  | {
  kind: "ok";
  value: EventResult[];
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `EventConsumer.event`. `ok` has exactly one entry per
`EventBatch.events` entry, in order; a short list is a host-side error and
the missing tail is retried.

***

### EventOutcome

```ts
type EventOutcome = 
  | {
  kind: "ack";
}
  | {
  kind: "retry";
  reason: string;
  retryAtUnixMs: number;
}
  | {
  kind: "reject";
  reason: string;
}
  | {
  kind: "deadLetter";
  reason: string;
}
  | {
  checkpointJson: string;
  checkpointSchemaVersion: number;
  kind: "suspended";
  wakeAtUnixMs: number;
  wakeOnEventType: string;
  wakeOnFilterJson: string;
};
```

Recorded per-event outcome (`EventResult` union, bridge JSON projection).

***

### EventPollReply

```ts
type EventPollReply = 
  | {
  kind: "ok";
  value: EventPollResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `RemoteLibrary.pollEvents`.

***

### EventResult

```ts
type EventResult = 
  | {
  kind: "ack";
  value: EventAck;
}
  | {
  kind: "retry";
  value: EventRetry;
}
  | {
  kind: "reject";
  value: EventReject;
}
  | {
  kind: "deadLetter";
  value: EventDeadLetter;
}
  | {
  kind: "suspended";
  value: EventSuspended;
};
```

Per-event outcome inside an `EventConsumer.event` batch reply.

***

### ExecuteResultReply

```ts
type ExecuteResultReply = 
  | {
  kind: "ok";
  value: ExecuteReply;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `execute`.

***

### ExternalUserReply

```ts
type ExternalUserReply = 
  | {
  kind: "ok";
  value: ExternalUser;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Oidc.authenticateUser`.

***

### FetchTitleReply

```ts
type FetchTitleReply = 
  | {
  kind: "ok";
  value: PlainFetch;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.fetchTitle`.

***

### GetReply

```ts
type GetReply = 
  | {
  kind: "ok";
  value: GetOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.get`.

***

### GuestDatabaseReply

```ts
type GuestDatabaseReply = 
  | {
  kind: "ok";
  value: GuestDatabase;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `guest database opens`.

***

### HandleReply

```ts
type HandleReply = 
  | {
  kind: "ok";
  value: JobOutcome;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `JobRunner.job`.

***

### HeadReply

```ts
type HeadReply = 
  | {
  kind: "ok";
  value: HeadOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.head`.

***

### HealthReply

```ts
type HealthReply = 
  | {
  kind: "ok";
  value: HealthOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `health`.

***

### IdentityExportReply

```ts
type IdentityExportReply = 
  | {
  kind: "ok";
  value: IdentityHighWater[];
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `AdapterDatabaseSession.exportIdentity`.

***

### Instant

```ts
type Instant = Date | number;
```

Wall-clock instant accepted by `retry` / `suspend`: a `Date` or Unix ms.

***

### IntegerArithKind

```ts
type IntegerArithKind = typeof INTEGER_ARITH_KINDS[number];
```

Union of `INTEGER_ARITH_KINDS` wire names.

***

### IsolationReq

```ts
type IsolationReq = typeof ISOLATION_REQS[number];
```

Union of `ISOLATION_REQS` wire names.

***

### JobOutcome

```ts
type JobOutcome = 
  | {
  kind: "completed";
  value: CompletedOutcome;
}
  | {
  kind: "retryable";
  value: RetryableOutcome;
}
  | {
  kind: "rejected";
  value: RejectedOutcome;
}
  | {
  kind: "cancelled";
  value: CancelledOutcome;
}
  | {
  kind: "suspended";
  value: SuspendedOutcome;
};
```

Terminal or suspended result of `JobRunner.job`.

***

### JobOutcomeRecord

```ts
type JobOutcomeRecord = 
  | {
  bytesCopied: number;
  kind: "completed";
  message: string;
}
  | {
  kind: "retryable";
  message: string;
  retryAfterUnixMs: number;
}
  | {
  kind: "rejected";
  message: string;
}
  | {
  kind: "cancelled";
  message: string;
}
  | {
  checkpoint: JobCheckpoint;
  kind: "suspended";
  wakeAtUnixMs: number;
};
```

Recorded terminal / suspended job outcome (`JobOutcome` union, bridge JSON projection).

***

### JsonObject

```ts
type JsonObject = Record<string, unknown>;
```

Plain JSON object (string keys, JSON values).

***

### KvDeleteReply

```ts
type KvDeleteReply = 
  | {
  kind: "ok";
  value: KvDeleteOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `KvNamespace.delete`.

***

### KvGetReply

```ts
type KvGetReply = 
  | {
  kind: "ok";
  value: KvGetOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `KvNamespace.get`.

***

### KvListReply

```ts
type KvListReply = 
  | {
  kind: "ok";
  value: KvListOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `KvNamespace.list`.

***

### KvPutReply

```ts
type KvPutReply = 
  | {
  kind: "ok";
  value: KvPutOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `KvNamespace.put`.

***

### KvValueType

```ts
type KvValueType = "text" | "json" | "arrayBuffer";
```

How [KvNamespaceBinding.get](#get-3) decodes a stored value.

***

### ListReply

```ts
type ListReply = 
  | {
  kind: "ok";
  value: ListPage;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.list`.

***

### LoginReply

```ts
type LoginReply = 
  | {
  kind: "ok";
  value: LoginResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.login` / `loginComplete`.

***

### LoginStartReply

```ts
type LoginStartReply = 
  | {
  kind: "ok";
  value: LoginStartResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.loginStart`.

***

### NamedStub

```ts
type NamedStub = Pick<NamedEntrypoint, "bookclerkInvoke">;
```

Named author entrypoint as seen through a `PLUGIN_<ENTRYPOINT>` binding.

***

### OidcClientsReply

```ts
type OidcClientsReply = 
  | {
  kind: "ok";
  value: OidcClientsOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Oidc.clients`.

***

### OpenReply

```ts
type OpenReply = 
  | {
  kind: "ok";
  value: OpenOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Source.open`.

***

### OptionalColumnReference

```ts
type OptionalColumnReference = 
  | {
  kind: "none";
}
  | {
  kind: "some";
  value: ColumnReference;
};
```

Per-column `REFERENCES` slot in `CreateTableSchema`.

***

### PluginDescribe

```ts
type PluginDescribe = Partial<WirePluginDescribe>;
```

Optional identity refinement returned by `BookclerkEntrypoint.describe()`.

The launcher derives the full wire `PluginDescribe` from `plugin.toml`
(`apiVersion`, `id`, `capabilities`, `configOptions`, `cli`, …); an author
`describe()` may add or override presentation fields such as
`displayName`, `brand`, or `rpcFeatures`. Identity, capabilities, and the
`[[settings]]`-derived `configOptions` always come from the manifest: a
guest list that differs from the manifest fails the spawn on the host.

***

### PluginErrorCode

```ts
type PluginErrorCode = typeof PLUGIN_ERROR_CODES[number];
```

Union of `PLUGIN_ERROR_CODES` wire names.

***

### PluginMigrationOp

```ts
type PluginMigrationOp = 
  | {
  kind: "schema";
  value: string;
}
  | {
  kind: "data";
  value: string;
};
```

One already-separated BookclerkSQL operation in a plugin-owned migration.
`schema` is admitted DDL; `data` is admitted DML. There is no native SQL
escape hatch.

***

### PluginMigrationsReply

```ts
type PluginMigrationsReply = 
  | {
  kind: "ok";
  value: PluginMigrationsOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `PluginWorker.databaseMigrations`.

***

### PortalAuthMode

```ts
type PortalAuthMode = typeof PORTAL_AUTH_MODES[number];
```

Union of `PORTAL_AUTH_MODES` wire names.

***

### PublishEvent

```ts
type PublishEvent = Pick<PluginEvent, "eventType"> & Partial<Omit<PluginEvent, "eventType" | "payload">> & object;
```

Event a guest publishes; the host stamps id, `source`, and `accountId`.

#### Type Declaration

| Name | Type | Description |
| ------ | ------ | ------ |
| `payload?` | `Uint8Array` \| [`JsonObject`](#jsonobject) \| `unknown`[] \| `string` | Payload bytes or a JSON-serializable value encoded as UTF-8 JSON. |

***

### PublishReply

```ts
type PublishReply = 
  | {
  kind: "ok";
  value: PublishOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `EventPublisher.publish`.

***

### PullReply

```ts
type PullReply = 
  | {
  kind: "ok";
  value: PullOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ByteSource.pull`.

***

### PurchaseHintReply

```ts
type PurchaseHintReply = 
  | {
  kind: "ok";
  value: PurchaseHintResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.purchaseHint`.

***

### PutReply

```ts
type PutReply = 
  | {
  kind: "ok";
  value: PutResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.put` / `Destination.commit`.

***

### QueueBatchReply

```ts
type QueueBatchReply = 
  | {
  kind: "ok";
  value: QueueBatchOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `QueueConsumer.queue`.

***

### QueueOutcome

```ts
type QueueOutcome = 
  | {
  kind: "ack";
}
  | {
  delaySeconds: number;
  kind: "retry";
  reason: string;
};
```

Recorded per-message outcome of a `queue()` handler (bridge projection of `QueueResult`).

***

### QueueResult

```ts
type QueueResult = 
  | {
  kind: "ack";
}
  | {
  kind: "retry";
  value: QueueRetry;
};
```

Per-message outcome inside a `QueueConsumer.queue` batch reply.

***

### QueueSendReply

```ts
type QueueSendReply = 
  | {
  kind: "ok";
  value: QueueSendOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `QueueProducer.send` / `sendBatch`.

***

### ResolvedSqlType

```ts
type ResolvedSqlType = typeof RESOLVED_SQL_TYPES[number];
```

Union of `RESOLVED_SQL_TYPES` wire names.

***

### ScanReply

```ts
type ScanReply = 
  | {
  kind: "ok";
  value: ScanSummary;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.scan`.

***

### SchemaAction

```ts
type SchemaAction = 
  | {
  kind: "none";
}
  | {
  kind: "create";
  value: SchemaCreate;
}
  | {
  kind: "drop";
  value: string;
};
```

CREATE/DROP action recorded on a proof.

***

### SourceAccountsReply

```ts
type SourceAccountsReply = 
  | {
  kind: "ok";
  value: SourceAccounts;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.listAccounts`.

***

### SyncListeningReply

```ts
type SyncListeningReply = 
  | {
  kind: "ok";
  value: SyncListeningResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `RemoteLibrary.syncListening`.

***

### TableConstraint

```ts
type TableConstraint = 
  | {
  kind: "primaryKey";
  value: string[];
}
  | {
  kind: "unique";
  value: string[];
}
  | {
  kind: "check";
  value: string;
}
  | {
  kind: "foreignKey";
  value: ForeignKeyConstraint;
};
```

One table-level constraint.

***

### UserRelationsReply

```ts
type UserRelationsReply = 
  | {
  kind: "ok";
  value: string[];
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `AdapterDatabaseSession.listUserRelations`.

## Variables

### ENTRYPOINT\_BINDINGS

```ts
const ENTRYPOINT_BINDINGS: Readonly<Record<EntrypointName, string>>;
```

Adapter binding name for each named entrypoint.

***

### ENTRYPOINT\_CLASSES

```ts
const ENTRYPOINT_CLASSES: Readonly<Record<EntrypointName, string>>;
```

Exported class name the launcher binds for each named entrypoint.

***

### FEATURE\_SCALAR\_LIMITS

```ts
const FEATURE_SCALAR_LIMITS: "rpc.scalarLimits";
```

Guest honors scalar / stream-window / list-page caps (`rpc.scalarLimits`).

***

### FEATURE\_STORAGE\_COPY

```ts
const FEATURE_STORAGE_COPY: "storage.copy";
```

Guest implements server-side `Destination.copy` (`storage.copy`).

***

### FEATURE\_STREAMS

```ts
const FEATURE_STREAMS: "rpc.streams";
```

Media moves through transferred `ByteRange` / `ByteSource` streams (`rpc.streams`).

***

### MAX\_CHECKPOINT\_BYTES

```ts
const MAX_CHECKPOINT_BYTES: 65536;
```

Maximum job / event checkpoint payload size (bytes).

***

### MAX\_LIST\_PAGE

```ts
const MAX_LIST_PAGE: 256;
```

Maximum objects in one `Destination.list` page.

***

### MAX\_PLUGIN\_MIGRATION\_OPS

```ts
const MAX_PLUGIN_MIGRATION_OPS: 256;
```

Maximum already-separated operations in one plugin-owned migration.

***

### MAX\_PLUGIN\_MIGRATION\_REGISTRATION\_BYTES

```ts
const MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES: 262144;
```

Maximum aggregate UTF-8 bytes of plugin migration ids plus SQL in one `databaseMigrations` registration.

***

### MAX\_PLUGIN\_MIGRATION\_TOTAL\_OPS

```ts
const MAX_PLUGIN_MIGRATION_TOTAL_OPS: 2048;
```

Maximum already-separated operations across one `databaseMigrations` registration.

***

### MAX\_SCALAR\_BYTES

```ts
const MAX_SCALAR_BYTES: 262144;
```

Maximum decoded size of an ordinary RPC scalar value (not a stream window).

***

### MAX\_STREAM\_WINDOW\_BYTES

```ts
const MAX_STREAM_WINDOW_BYTES: 1048576;
```

Maximum bytes returned by one `ByteSource.pull` (flow-control window).

***

### PRODUCT\_API\_VERSION

```ts
const PRODUCT_API_VERSION: 3;
```

Product ABI version (`plugin.toml` `api_version` / `describe().apiVersion`).

## Functions

### allowedEntrypointFamilies()

```ts
function allowedEntrypointFamilies(capabilities, requested): string[];
```

Filters requested entrypoint / trigger family names against the manifest
capabilities: named entrypoints must be declared in `entrypoints`,
`eventConsumer` needs a non-empty `consumes`, `queueConsumer` needs a
non-empty `queueConsumers`, and `jobRunner` needs `jobs` or the `storage`
entrypoint (storage guests run the host `stream_copy` job). Unknown names
and a missing manifest fail closed.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `capabilities` | \| `Partial`\<[`PluginCapabilities`](#plugincapabilities)\> \| `null` \| `undefined` | Manifest capabilities from `PLUGIN_DESCRIBE`. |
| `requested` | readonly `string`[] | Family names the launcher wants to export. |

#### Returns

`string`[]

The authorized subset in request order.

***

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

### cliArgs()

```ts
function cliArgs(params): Record<string, string>;
```

Turns `CliInvokeParams.args` (`[{ name, value }]`) into a plain
`{ name: value }` object for CLI handlers.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | `Pick`\<[`CliInvokeParams`](#cliinvokeparams), `"args"`\> \| `undefined` | CLI invocation params from the host. |

#### Returns

`Record`\<`string`, `string`\>

Argument values keyed by name.

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
| `options` | [`DatabaseBindingOptions`](#databasebindingoptions) | Request-budget and idempotency knobs. |

#### Returns

[`DatabaseBinding`](#databasebinding)

Binding whose `execute(batch)` encodes Cap'n then calls the host.

***

### dataMigrationOp()

```ts
function dataMigrationOp(sql): object;
```

Data DML convenience for [PluginMigration](#pluginmigration) operations.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `sql` | `string` | Already-separated BookclerkSQL data statement. |

#### Returns

`object`

Data operation tagged for host registration.

| Name | Type |
| ------ | ------ |
| `data` | `string` |

***

### decodeDbValue()

```ts
function decodeDbValue(bytes): DbValue;
```

Decodes a standalone unpacked Cap'n `DbValue` message.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bytes` | `Uint8Array` | Unpacked Cap'n stream. |

#### Returns

[`DbValue`](#dbvalue)

Domain value.

#### Throws

When the buffer is not a valid `DbValue`.

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

### decodeExtensibleConfig()

```ts
function decodeExtensibleConfig(cfg): JsonObject;
```

Decode an `ExtensibleConfig` (`{ schemaVersion, mediaType, payload }`) into
the plain object authors read from `env.CONFIG` / `env.SECRETS`. Non-JSON
media types are surfaced verbatim so nothing is lost.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `cfg` | `unknown` | Wire config struct, a pre-decoded object, or `undefined`. |

#### Returns

[`JsonObject`](#jsonobject)

Plain JSON object (`{}` when empty or malformed).

***

### encodeDbValue()

```ts
function encodeDbValue(value): Uint8Array;
```

Encodes a standalone unpacked Cap'n `DbValue` message.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `value` | [`DbValue`](#dbvalue) | Domain value (`bigint` / `Uint8Array`, not JSON number / `b64:`). |

#### Returns

`Uint8Array`

Unpacked Cap'n stream bytes (same encoding as the Rust SDK).

#### Throws

When `int64` is out of range, `float64` is not finite, or TEXT
contains U+0000.

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

### eventBatchResults()

```ts
function eventBatchResults(batch, failed): EventOutcome[];
```

Translate recorded [EventMessage](#eventmessage) outcomes into the wire list.
`failed` is the error thrown by the author's `event()`; undecided messages
then retry instead of ack.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `batch` | [`EventBatch`](#eventbatch) | Delivered batch after the handler ran. |
| `failed` | `unknown` | Error thrown by the handler, if any. |

#### Returns

[`EventOutcome`](#eventoutcome)[]

One outcome per message, in order.

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

[`D1Result`](#d1result)\<`Record`\<`string`, [`DbValue`](#dbvalue)\>\>[]

One `D1Result` per statement, in plan order.

***

### invocationOf()

```ts
function invocationOf(context): Invocation;
```

Invocation envelope (`Invocation` struct) with zero values normalized.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) \| `undefined` | Granted context carrying `invocation`. |

#### Returns

[`Invocation`](#invocation-11)

Frozen invocation identity.

***

### jobOutcomeFor()

```ts
function jobOutcomeFor(
   job, 
   returned, 
   failed
): JobOutcomeRecord;
```

Translate a `job()` handler's return / throw into the wire outcome.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `job` | [`JobController`](#jobcontroller) | Controller after the handler ran. |
| `returned` | `void` \| [`JobCompletion`](#jobcompletion) \| `undefined` | Handler return value. |
| `failed` | `unknown` | Error thrown by the handler, if any. |

#### Returns

[`JobOutcomeRecord`](#joboutcomerecord)

Wire job outcome.

***

### jsonPayload()

```ts
function jsonPayload(value): ExtensibleConfig;
```

Wraps a JSON-serializable value as an `application/json` `ExtensibleConfig`
(schema version 1) — the shape of `CliInvokeResult.payload` and every other
extensible payload on the wire.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `value` | `unknown` | JSON-serializable value. |

#### Returns

[`ExtensibleConfig`](#extensibleconfig)

Wire-shaped extensible payload.

***

### parseDbValue()

```ts
function parseDbValue(raw): DbValue;
```

Parses a JSON `DbValue`. Unknown union members throw.

`int64` accepts a `bigint` or a finite integer `number` (converted to
`bigint`). `bytes` accepts a `Uint8Array` or a `b64:` JSON string.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `raw` | `unknown` | Parsed JSON object. |

#### Returns

[`DbValue`](#dbvalue)

Typed value.

#### Throws

When `raw` is not a closed `DbValue`.

***

### queueBatchResults()

```ts
function queueBatchResults(batch, failed): QueueOutcome[];
```

Translate recorded [QueueMessage](#queuemessage) outcomes into the wire list.
`failed` is the error thrown by the author's `queue()`; undecided messages
then retry instead of ack.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `batch` | [`QueueBatch`](#queuebatch) | Delivered batch after the handler ran. |
| `failed` | `unknown` | Error thrown by the handler, if any. |

#### Returns

[`QueueOutcome`](#queueoutcome)[]

One outcome per message, in order.

***

### requirePluginMigrationRegistration()

```ts
function requirePluginMigrationRegistration(migrations): PluginMigration[];
```

Rejects a plugin migration registration that exceeds ABI resource limits.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `migrations` | [`PluginMigration`](#pluginmigration)[] | Ordered `databaseMigrations` registration. |

#### Returns

[`PluginMigration`](#pluginmigration)[]

The same array when within limits.

#### Throws

Error with `code` / `wireCode` `payload_too_large` when a limit is exceeded.

***

### schemaMigrationOp()

```ts
function schemaMigrationOp(sql): object;
```

Schema DDL convenience for [PluginMigration](#pluginmigration) operations.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `sql` | `string` | Already-separated BookclerkSQL schema statement. |

#### Returns

`object`

Schema operation tagged for host registration.

| Name | Type |
| ------ | ------ |
| `schema` | `string` |

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

### toBridgeJson()

```ts
function toBridgeJson(value): unknown;
```

Typed ABI value → bridge JSON: Cap'n `Data` fields (`Uint8Array`) travel as
base64 text; `bookclerk-workerd` decodes with the same convention.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `value` | `unknown` | Plain typed struct value. |

#### Returns

`unknown`

JSON-safe projection.

***

### wrapPluginFromBinding()

```ts
function wrapPluginFromBinding(): typeof InvocationAdapter;
```

Generated adapter isolate: `env.PLUGIN` is the author's default entrypoint
and `env.PLUGIN_<ENTRYPOINT>` the named ones. Every `POST /invoke` call
(Cap'n Proto bytes) and every stream route becomes one `bookclerk*`
dispatch on the author class; authors cannot replace adapter behavior with
adapter-internal names.

#### Returns

*typeof* `InvocationAdapter`

Adapter entrypoint class bound to [AdapterEnv](#adapterenv).

***

### wrapPluginFromNative()

```ts
function wrapPluginFromNative(): typeof InvocationAdapter;
```

Native-behind-workerd generated adapter: control plane only. The launcher
forwards every entrypoint call to the native guest as typed Cap'n Proto;
this isolate decides `describe` (merged against `PLUGIN_DESCRIBE`) and
`open` policy (`openInvocation`) and receives `shutdown`. There is no
author `PLUGIN` binding.

#### Returns

*typeof* `InvocationAdapter`

Adapter entrypoint class bound to [AdapterEnv](#adapterenv).
