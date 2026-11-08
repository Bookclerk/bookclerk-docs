# plugin

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

`RpcTarget.constructor`

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

`Promise`\<[`DbBootstrap`](generated.md#dbbootstrap)\>

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
| `_request` | [`ExecuteRequest`](db-execute.md#executerequest) | Cap'n `ExecuteRequest` (structured Workers RPC object). |

###### Returns

`Promise`\<[`ExecuteReply`](db-execute.md#executereply)\>

`ExecuteReply`.

##### exportIdentity()

```ts
exportIdentity(): Promise<IdentityHighWater[]>;
```

Identity high-water marks for a logical export; optional.

###### Returns

`Promise`\<[`IdentityHighWater`](generated.md#identityhighwater)[]\>

One row per identity column.

##### importIdentity()

```ts
importIdentity(_rows): Promise<void>;
```

Restore identity high-water marks after a logical import; optional.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_rows` | [`IdentityHighWater`](generated.md#identityhighwater)[] | Rows from an earlier [AdapterDatabaseSession.exportIdentity](#exportidentity). |

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
| <a id="type-parameter-env"></a> `Env` *extends* [`BookclerkEnv`](env.md#bookclerkenv) | [`BookclerkEnv`](env.md#bookclerkenv) |

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

`WorkerEntrypoint<Env>.constructor`

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

`WorkerEntrypoint.ctx`

##### env

```ts
readonly env: Env;
```

###### Inherited from

`WorkerEntrypoint.env`

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

`Promise`\<[`PluginMigration`](generated.md#pluginmigration)[]\>

Bounded registration (`[]` when not implemented).

##### bookclerkDescribe()

```ts
bookclerkDescribe(): Promise<Partial<PluginDescribe> | null>;
```

**`Internal`**

Adapter dispatch for `describe()`.

###### Returns

`Promise`\<`Partial`\<[`PluginDescribe`](generated.md#plugindescribe)\> \| `null`\>

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
| `invocation` | `Partial`\<[`JobInvocation`](generated.md#jobinvocation)\> | Durable command envelope. |
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

##### bookclerkScheduled()

```ts
bookclerkScheduled(context, wire): Promise<void>;
```

**`Internal`**

Adapter dispatch for the `scheduled(controller)` trigger.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `context` | [`GrantedContext`](#grantedcontext) | Granted bindings for this invocation. |
| `wire` | `Partial`\<[`ScheduledController`](generated.md#scheduledcontroller)\> | Wire `ScheduledController`. |

###### Returns

`Promise`\<`void`\>

Resolves when the author's handler completes.

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
databaseMigrations?(binding): 
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

  \| [`PluginMigration`](generated.md#pluginmigration)[]
  \| `Promise`\<[`PluginMigration`](generated.md#pluginmigration)[]\>

##### describe()?

```ts
describe?(): 
  | Partial<PluginDescribe>
| Promise<Partial<PluginDescribe>>;
```

Optional identity refinement merged over the manifest projection.

###### Returns

  \| `Partial`\<[`PluginDescribe`](generated.md#plugindescribe)\>
  \| `Promise`\<`Partial`\<[`PluginDescribe`](generated.md#plugindescribe)\>\>

##### event()?

```ts
event?(batch): void | Promise<void>;
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
job?(job): 
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
queue?(batch): void | Promise<void>;
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

##### scheduled()?

```ts
scheduled?(controller): void | Promise<void>;
```

`[triggers] crons` trigger, the Workers `scheduled(controller)`: run one
cron fire. Return to complete; throw to have the host retry the same
fire (same `scheduledTime` and `fireId`) with backoff. Fires are
delivered at least once, so dedupe on `controller.fireId`.

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `controller` | `Readonly`\<[`ScheduledController`](generated.md#scheduledcontroller)\> |

###### Returns

`void` \| `Promise`\<`void`\>

##### shutdown()?

```ts
shutdown?(): void | Promise<void>;
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
| <a id="type-parameter-env-1"></a> `Env` *extends* [`BookclerkEnv`](env.md#bookclerkenv) | [`BookclerkEnv`](env.md#bookclerkenv) |

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

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-10)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-5)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-5)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-6)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[];
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-4)

##### describe()

```ts
describe(): Promise<CliSchema>;
```

Guest CLI schema.

###### Returns

`Promise`\<[`CliSchema`](generated.md#clischema)\>

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

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-5)

##### invoke()

```ts
invoke(_params): Promise<CliInvokeResult>;
```

Invokes a guest CLI command.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`CliInvokeParams`](generated.md#cliinvokeparams) | Command name plus named argument values. |

###### Returns

`Promise`\<[`CliInvokeResult`](generated.md#cliinvokeresult)\>

Exit code, captured output, optional structured payload.

***

### CoverEntrypoint

`covers` entrypoint (`CoverFetcher`): downloads cover images for the host.
The host passes a `sink` [Destination](#destination); put the body for `urls[i]`
under the key `String(i)` and return one [CoverImage](generated.md#coverimage) per URL, in
request order. The host sniffs, decodes and re-encodes every body, so the
bytes stay untrusted. Export it as `export class Covers extends CoverEntrypoint`.

#### Extends

- [`NamedEntrypoint`](#namedentrypoint)\<`Env`\>

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| <a id="type-parameter-env-2"></a> `Env` *extends* [`BookclerkEnv`](env.md#bookclerkenv) | [`BookclerkEnv`](env.md#bookclerkenv) |

#### Constructors

##### Constructor

```ts
new CoverEntrypoint<Env>(ctx, env): CoverEntrypoint<Env>;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `ExecutionContext` |
| `env` | `Env` |

###### Returns

[`CoverEntrypoint`](#coverentrypoint)\<`Env`\>

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-10)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-5)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-5)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-6)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[];
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-5)

##### fetchCovers()

```ts
fetchCovers(_params, _sink): Promise<CoverImage[]>;
```

Fetches each URL and streams its body into `sink` under its index.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`CoverFetchParams`](generated.md#coverfetchparams) | URLs (absolute `https://`) and the per-body byte cap. |
| `_sink` | [`Destination`](#destination) | Host destination; accepts `put` for keys `0..urls.length`. |

###### Returns

`Promise`\<[`CoverImage`](generated.md#coverimage)[]\>

One entry per URL, in request order.

##### health()

```ts
health(): Promise<HealthOk>;
```

Reports whether the fetcher can answer.

###### Returns

`Promise`\<[`HealthOk`](generated.md#healthok)\>

Health flag plus detail.

***

### DatabaseAdapterEntrypoint

`databaseAdapter` entrypoint: opens typed SQL sessions for the host
library. Export it as `export class DatabaseAdapter extends DatabaseAdapterEntrypoint`.

The adapter isolate drives sessions through two dispatch verbs on
[DatabaseAdapterEntrypoint.bookclerkInvoke](#bookclerkinvoke-2): `openSession` returns a
fresh session id, and `session(id, method, ...params)` calls `method` on
the retained [AdapterDatabaseSession](#adapterdatabasesession); `close` releases the entry.

#### Extends

- [`NamedEntrypoint`](#namedentrypoint)\<`Env`\>

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| <a id="type-parameter-env-3"></a> `Env` *extends* [`BookclerkEnv`](env.md#bookclerkenv) | [`BookclerkEnv`](env.md#bookclerkenv) |

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

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-10)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-5)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-5)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-6)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[];
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-5)

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

`RpcTarget.constructor`

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
| `events` | \| readonly `Partial`\<[`DomainEvent`](generated.md#domainevent)\>[] \| `undefined` |
| `invocation?` | [`Invocation`](generated.md#invocation) |

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
| `event` | `Partial`\<[`DomainEvent`](generated.md#domainevent)\> \| `undefined` |

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
| <a id="type-parameter-t"></a> `T` | [`JsonObject`](env.md#jsonobject) |

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
| `invocation` | `Partial`\<[`JobInvocation`](generated.md#jobinvocation)\> \| `undefined` |
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
| <a id="type-parameter-t-1"></a> `T` | [`JsonObject`](env.md#jsonobject) |

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

### MetadataEntrypoint

`metadata` entrypoint (`MetadataProvider`): bibliographic lookups against a
public catalog. Records describe editions; the host decides identity.
Export it as `export class Metadata extends MetadataEntrypoint`.

#### Extends

- [`NamedEntrypoint`](#namedentrypoint)\<`Env`\>

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| <a id="type-parameter-env-4"></a> `Env` *extends* [`BookclerkEnv`](env.md#bookclerkenv) | [`BookclerkEnv`](env.md#bookclerkenv) |

#### Constructors

##### Constructor

```ts
new MetadataEntrypoint<Env>(ctx, env): MetadataEntrypoint<Env>;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `ExecutionContext` |
| `env` | `Env` |

###### Returns

[`MetadataEntrypoint`](#metadataentrypoint)\<`Env`\>

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-10)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-5)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-5)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-6)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[];
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-4)

##### chapters()

```ts
chapters(_params): Promise<MetadataChapters>;
```

Chapter layout of one edition.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`MetadataChaptersParams`](generated.md#metadatachaptersparams) | Identifiers of the edition. |

###### Returns

`Promise`\<[`MetadataChapters`](generated.md#metadatachapters)\>

The layout, or `{ found: false }` when the provider has none.

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

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-5)

##### health()

```ts
health(): Promise<HealthOk>;
```

Reports whether the provider can answer.

###### Returns

`Promise`\<[`HealthOk`](generated.md#healthok)\>

Health flag plus detail.

##### lookup()

```ts
lookup(_params): Promise<MetadataRecord[]>;
```

Records for the editions the identifiers name.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`MetadataLookupParams`](generated.md#metadatalookupparams) | Identifiers, marketplace and result cap. |

###### Returns

`Promise`\<[`MetadataRecord`](generated.md#metadatarecord)[]\>

Records in relevance order (`[]` when nothing matched).

##### search()

```ts
search(_params): Promise<MetadataRecord[]>;
```

Records matching a title and, optionally, an author.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`MetadataSearchParams`](generated.md#metadatasearchparams) | Title, author, language, marketplace and result cap. |

###### Returns

`Promise`\<[`MetadataRecord`](generated.md#metadatarecord)[]\>

Records in relevance order (`[]` when nothing matched).

***

### NamedEntrypoint

Base for named entrypoints. Subclasses list their RPC surface on the static
`bookclerkMethods`; the adapter reaches them only through `bookclerkInvoke`,
which installs the granted bindings on `env` before dispatching.

#### Extends

- `WorkerEntrypoint`\<`Env`\>

#### Extended by

- [`StorefrontEntrypoint`](#storefrontentrypoint)
- [`StorageEntrypoint`](#storageentrypoint)
- [`RemoteLibraryEntrypoint`](#remotelibraryentrypoint)
- [`DatabaseAdapterEntrypoint`](#databaseadapterentrypoint)
- [`CliEntrypoint`](#clientrypoint)
- [`OidcEntrypoint`](#oidcentrypoint)
- [`MetadataEntrypoint`](#metadataentrypoint)
- [`CoverEntrypoint`](#coverentrypoint)

#### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| <a id="type-parameter-env-5"></a> `Env` *extends* [`BookclerkEnv`](env.md#bookclerkenv) | [`BookclerkEnv`](env.md#bookclerkenv) |

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

`WorkerEntrypoint<Env>.constructor`

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

`WorkerEntrypoint.ctx`

##### env

```ts
readonly env: Env;
```

###### Inherited from

`WorkerEntrypoint.env`

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
| <a id="type-parameter-env-6"></a> `Env` *extends* [`BookclerkEnv`](env.md#bookclerkenv) | [`BookclerkEnv`](env.md#bookclerkenv) |

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

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-10)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-5)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-5)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-6)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[];
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-4)

#### Methods

##### authenticateUser()

```ts
authenticateUser(_params): Promise<ExternalUser>;
```

Verifies remote credentials on behalf of the host.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`AuthenticateUserParams`](generated.md#authenticateuserparams) | Username / password pair. |

###### Returns

`Promise`\<[`ExternalUser`](generated.md#externaluser)\>

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

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-4)

##### clients()

```ts
clients(): Promise<OidcClientTemplate[]>;
```

Plugin-provided OIDC authorization-server client templates. The host
materializes `oidc_clients` rows; plugins never mint tokens.

###### Returns

`Promise`\<[`OidcClientTemplate`](generated.md#oidcclienttemplate)[]\>

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

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-5)

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

`RpcTarget.constructor`

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
| `messages` | \| readonly `Partial`\<[`QueueMessage`](generated.md#queuemessage)\>[] \| `undefined` |
| `invocation?` | [`Invocation`](generated.md#invocation) |

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
| `message` | `Partial`\<[`QueueMessage`](generated.md#queuemessage)\> \| `undefined` |

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
readonly raw: Partial<QueueMessage>;
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
| <a id="type-parameter-t-2"></a> `T` | `unknown` |

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
| <a id="type-parameter-env-7"></a> `Env` *extends* [`BookclerkEnv`](env.md#bookclerkenv) | [`BookclerkEnv`](env.md#bookclerkenv) |

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

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-10)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-5)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-5)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-6)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[] = REMOTE_LIBRARY_METHODS;
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-5)

##### health()

```ts
health(): Promise<HealthOk>;
```

Reports whether the remote library session is usable.

###### Returns

`Promise`\<[`HealthOk`](generated.md#healthok)\>

Health flag plus detail.

##### pollEvents()

```ts
pollEvents(): Promise<ExternalUser[]>;
```

Drains external users observed since the last poll.

###### Returns

`Promise`\<[`ExternalUser`](generated.md#externaluser)[]\>

Newly observed users.

##### scanLibrary()

```ts
scanLibrary(_params): Promise<void>;
```

Re-syncs the remote library.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`ScanLibraryParams`](generated.md#scanlibraryparams) | Scan scope. |

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

`Promise`\<[`ListeningProgress`](generated.md#listeningprogress)[]\>

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

`RpcTarget.constructor`

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
| <a id="type-parameter-env-8"></a> `Env` *extends* [`BookclerkEnv`](env.md#bookclerkenv) | [`BookclerkEnv`](env.md#bookclerkenv) |

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

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-10)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-5)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-5)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-6)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[] = STORAGE_METHODS;
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-5)

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
| <a id="type-parameter-env-9"></a> `Env` *extends* [`BookclerkEnv`](env.md#bookclerkenv) | [`BookclerkEnv`](env.md#bookclerkenv) |

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

[`NamedEntrypoint`](#namedentrypoint).[`constructor`](#constructor-10)

#### Properties

##### ctx

```ts
readonly ctx: ExecutionContext;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`ctx`](#ctx-5)

##### env

```ts
readonly env: Env;
```

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`env`](#env-5)

##### invocation

```ts
invocation: Invocation;
```

Invocation identity of the current call (set by `bookclerkInvoke`).

###### Inherited from

[`NamedEntrypoint`](#namedentrypoint).[`invocation`](#invocation-6)

##### bookclerkMethods

```ts
static bookclerkMethods: readonly string[] = STOREFRONT_METHODS;
```

Methods the adapter may dispatch on this entrypoint.

###### Overrides

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkMethods`](#bookclerkmethods-4)

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

[`NamedEntrypoint`](#namedentrypoint).[`bookclerkInvoke`](#bookclerkinvoke-4)

##### catalogDetail()

```ts
catalogDetail(_params): Promise<CatalogHit | null>;
```

Full catalog record for one product; `null` when unknown.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`CatalogDetailParams`](generated.md#catalogdetailparams) | Product identifiers. |

###### Returns

`Promise`\<[`CatalogHit`](generated.md#cataloghit) \| `null`\>

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
| `_params` | [`ExpandCandidatesParams`](generated.md#expandcandidatesparams) | Seed title plus limits. |

###### Returns

`Promise`\<[`CatalogHit`](generated.md#cataloghit)[]\>

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

[`NamedEntrypoint`](#namedentrypoint).[`fetch`](#fetch-5)

##### fetchTitle()

```ts
fetchTitle(_params): Promise<PlainFetch>;
```

Fetches one title into `params.cacheDir` and returns plain media paths.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`FetchTitleParams`](generated.md#fetchtitleparams) | Title identifiers, credentials, fetch options. |

###### Returns

`Promise`\<[`PlainFetch`](generated.md#plainfetch)\>

Plain media parts plus metadata.

##### health()

```ts
health(): Promise<HealthOk>;
```

Reports whether the storefront session is usable.

###### Returns

`Promise`\<[`HealthOk`](generated.md#healthok)\>

Health flag plus detail.

##### listAccounts()

```ts
listAccounts(): Promise<SourceAccount[]>;
```

Accounts the guest knows about.

###### Returns

`Promise`\<[`SourceAccount`](generated.md#sourceaccount)[]\>

Account list.

##### listDeals()

```ts
listDeals(_params): Promise<CatalogHit[]>;
```

Current storefront deals.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`ListDealsParams`](generated.md#listdealsparams) | Optional filters. |

###### Returns

`Promise`\<[`CatalogHit`](generated.md#cataloghit)[]\>

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
| `_params` | [`LoginParams`](generated.md#loginparams) | Login parameters. |

###### Returns

`Promise`\<[`LoginResult`](generated.md#loginresult)\>

Account identity plus opaque credentials.

##### loginComplete()

```ts
loginComplete(_params): Promise<LoginResult>;
```

Finishes a login started by [StorefrontEntrypoint.loginStart](#loginstart).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`LoginCompleteParams`](generated.md#logincompleteparams) | Continuation plus user input. |

###### Returns

`Promise`\<[`LoginResult`](generated.md#loginresult)\>

Account identity plus opaque credentials.

##### loginStart()

```ts
loginStart(_params): Promise<LoginStartResult>;
```

Begins an interactive OAuth login.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`LoginParams`](generated.md#loginparams) | Login parameters. |

###### Returns

`Promise`\<[`LoginStartResult`](generated.md#loginstartresult)\>

Continuation for [StorefrontEntrypoint.loginComplete](#logincomplete).

##### purchaseHint()

```ts
purchaseHint(_params): Promise<PurchaseHint | null>;
```

Purchase link / price hint; `null` when the title is unknown.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`PurchaseHintParams`](generated.md#purchasehintparams) | Title identifiers. |

###### Returns

`Promise`\<[`PurchaseHint`](generated.md#purchasehint-1) \| `null`\>

Hint or `null`.

##### revokeAccount()

```ts
revokeAccount(_params): Promise<void>;
```

Undo store-side state for one account before the host deletes its
credentials, such as deregistering the device registered at login.
Best effort: the host revokes the account whatever this returns. The
default does nothing.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`RevokeAccountParams`](generated.md#revokeaccountparams) | Account id and its stored credentials. |

###### Returns

`Promise`\<`void`\>

Resolves when the store sign-out finished.

##### scan()

```ts
scan(_params): Promise<ScanSummary>;
```

Library scan; the host upserts `ScanSummary.books`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`ScanParams`](generated.md#scanparams) | Scan parameters (accounts + credentials). |

###### Returns

`Promise`\<[`ScanSummary`](generated.md#scansummary)\>

Books discovered plus per-account counters.

##### searchCatalog()

```ts
searchCatalog(_params): Promise<CatalogHit[]>;
```

Free-text storefront catalog search.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `_params` | [`SearchCatalogParams`](generated.md#searchcatalogparams) | Query parameters. |

###### Returns

`Promise`\<[`CatalogHit`](generated.md#cataloghit)[]\>

Catalog hits.

## Interfaces

### AdapterEnv

Trusted adapter env. Authors never see this type on their class.

#### Properties

##### BRIDGE\_TOKEN?

```ts
BRIDGE_TOKEN?: string;
```

Isolate-to-host bridge bearer.

##### GRANTED?

```ts
GRANTED?: GrantedFetcher;
```

Per-invocation grant reverse channel.

##### PLUGIN?

```ts
PLUGIN?: AuthorStub;
```

Author default entrypoint.

##### PLUGIN\_CLI?

```ts
PLUGIN_CLI?: NamedStub;
```

##### PLUGIN\_DATABASE\_ADAPTER?

```ts
PLUGIN_DATABASE_ADAPTER?: NamedStub;
```

##### PLUGIN\_DESCRIBE?

```ts
PLUGIN_DESCRIBE?: string | PluginDescribe;
```

Manifest `PluginDescribe` projection (JSON binding).

##### PLUGIN\_OIDC?

```ts
PLUGIN_OIDC?: NamedStub;
```

##### PLUGIN\_REMOTE\_LIBRARY?

```ts
PLUGIN_REMOTE_LIBRARY?: NamedStub;
```

##### PLUGIN\_STORAGE?

```ts
PLUGIN_STORAGE?: NamedStub;
```

##### PLUGIN\_STOREFRONT?

```ts
PLUGIN_STOREFRONT?: NamedStub;
```

Named author entrypoints (`PLUGIN_STOREFRONT`, `PLUGIN_STORAGE`, …).

***

### BridgeContext

Context as the launcher sends it on the bridge: [GrantedContext](#grantedcontext)
plus the grant tokens the adapter exchanges for the author's `EVENTS` and
`env.<BINDING>` stubs. Authors never receive this shape.

#### Extends

- `Omit`\<[`GrantedContext`](#grantedcontext), `"databases"` \| `"queues"` \| `"kv"`\>

#### Properties

##### config?

```ts
config?: ExtensibleConfig;
```

Operator settings for this plugin as `application/json`.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`config`](#config-1)

##### databases?

```ts
databases?: Record<string, string>;
```

Named `[[databases]]` binding → database-only bearer for `/db/execute`.

##### events?

```ts
events?: EventPublisherBinding;
```

`EVENTS` publisher, when `[[events.producers]]` is granted.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`events`](#events-1)

##### eventsToken?

```ts
eventsToken?: string;
```

Granted-channel bearer for `env.EVENTS.publish`; stripped before dispatch.

##### invocation?

```ts
invocation?: Partial<Invocation>;
```

Invocation identity (`Invocation` struct).

###### Inherited from

[`GrantedContext`](#grantedcontext).[`invocation`](#invocation-13)

##### kv?

```ts
kv?: Record<string, string>;
```

`[[kv_namespaces]]` binding → kv-only bearer for `/kv/*`.

##### queues?

```ts
queues?: Record<string, string>;
```

`[[queues.producers]]` binding → queue-only bearer for `/queues/send`.

##### secrets?

```ts
secrets?: ExtensibleConfig;
```

Granted secret values as `application/json`.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`secrets`](#secrets-1)

##### storage?

```ts
storage?: Destination;
```

`WORK_FS` object store, when granted.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`storage`](#storage-1)

***

### ByteRange

Optional byte range for a streamed read.

#### Properties

##### length?

```ts
length?: number;
```

##### offset

```ts
offset: number;
```

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

### CopyResult

Result of a server-side copy.

#### Properties

##### bytesCopied

```ts
bytesCopied: number;
```

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

### EventSuspendOptions

Options for [EventMessage.suspend](#suspend).

#### Properties

##### checkpoint?

```ts
checkpoint?: Checkpoint;
```

Bounded checkpoint replayed on resume (at most `maxCheckpointBytes`).

##### checkpointSchemaVersion?

```ts
checkpointSchemaVersion?: number;
```

Schema version of `checkpoint` (default `1`).

##### wakeAt?

```ts
wakeAt?: Instant;
```

Earliest resume instant.

##### wakeOnEventType?

```ts
wakeOnEventType?: string;
```

Also wake when an event of this type arrives.

##### wakeOnFilter?

```ts
wakeOnFilter?: string | JsonObject;
```

Host-owned payload filter for wake events (object or JSON text).

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
config?: ExtensibleConfig;
```

Operator settings for this plugin as `application/json`.

##### databases?

```ts
databases?: NamedDatabase[];
```

Named `[[databases]]` bindings.

##### events?

```ts
events?: EventPublisherBinding;
```

`EVENTS` publisher, when `[[events.producers]]` is granted.

##### invocation?

```ts
invocation?: Partial<Invocation>;
```

Invocation identity (`Invocation` struct).

##### kv?

```ts
kv?: NamedKv[];
```

`[[kv_namespaces]]` bindings, when granted.

##### queues?

```ts
queues?: NamedQueue[];
```

`[[queues.producers]]` bindings, when granted.

##### secrets?

```ts
secrets?: ExtensibleConfig;
```

Granted secret values as `application/json`.

##### storage?

```ts
storage?: Destination;
```

`WORK_FS` object store, when granted.

***

### GrantedDatabaseTransport

Granted-channel SQL transport the adapter hands to the author isolate for
one named `[[databases]]` binding. The author-side SDK wraps it as the
D1-shaped [DatabaseBinding](db-execute.md#databasebinding) on `env.<BINDING>`; Python authors call
[GrantedDatabaseTransport.executeBytes](#executebytes) with Cap'n bytes instead.

#### Extends

- [`AtomicTransport`](db-execute.md#atomictransport)

#### Methods

##### execute()

```ts
execute(request): Promise<ExecuteReply>;
```

Runs one typed atomic batch.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`ExecuteRequest`](db-execute.md#executerequest) | Structured Cap'n `ExecuteRequest`. |

###### Returns

`Promise`\<[`ExecuteReply`](db-execute.md#executereply)\>

Structured `ExecuteReply`.

###### Inherited from

[`AtomicTransport`](db-execute.md#atomictransport).[`execute`](db-execute.md#execute)

##### executeBytes()

```ts
executeBytes(bytes): Promise<Uint8Array<ArrayBufferLike>>;
```

Same batch as [AtomicTransport.execute](db-execute.md#execute), as Cap'n Proto bytes
(`ExecuteRequest` in, `ExecuteResultReply` out).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bytes` | `Uint8Array` | Unpacked `ExecuteRequest` message. |

###### Returns

`Promise`\<`Uint8Array`\<`ArrayBufferLike`\>\>

Unpacked `ExecuteResultReply` message.

##### limits()

```ts
limits(): Promise<DbCapabilities>;
```

Negotiated limits of the granted session (`GuestDatabase.limits`).

###### Returns

`Promise`\<[`DbCapabilities`](generated.md#dbcapabilities)\>

The adapter's `DbCapabilities` for this binding.

###### Overrides

[`AtomicTransport`](db-execute.md#atomictransport).[`limits`](db-execute.md#limits)

##### limitsBytes()

```ts
limitsBytes(): Promise<Uint8Array<ArrayBufferLike>>;
```

Same answer as [GrantedDatabaseTransport.limits](#limits), as Cap'n Proto
bytes (`DbCapabilitiesReply`) for Python authors.

###### Returns

`Promise`\<`Uint8Array`\<`ArrayBufferLike`\>\>

Unpacked `DbCapabilitiesReply` message.

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
cancel?: CancelWatchLike | null;
```

##### input?

```ts
input?: Source | null;
```

##### output?

```ts
output?: Destination | null;
```

##### progress?

```ts
progress?: ProgressSink | null;
```

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
bytesCopied?: number;
```

Bytes copied by the job, when meaningful.

##### message?

```ts
message?: string;
```

Operator-facing summary.

***

### JobRetryOptions

Options for [JobController.retryLater](#retrylater).

#### Properties

##### reason?

```ts
reason?: string;
```

Short operator-facing reason.

##### retryAt?

```ts
retryAt?: Instant;
```

Earliest retry instant; omitted lets the host choose.

***

### JobRunnerContext

Job-runner bridge context: the durable job id plus [GrantedContext](#grantedcontext).

#### Extends

- [`GrantedContext`](#grantedcontext)

#### Properties

##### config?

```ts
config?: ExtensibleConfig;
```

Operator settings for this plugin as `application/json`.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`config`](#config-1)

##### databases?

```ts
databases?: NamedDatabase[];
```

Named `[[databases]]` bindings.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`databases`](#databases-1)

##### events?

```ts
events?: EventPublisherBinding;
```

`EVENTS` publisher, when `[[events.producers]]` is granted.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`events`](#events-1)

##### invocation?

```ts
invocation?: Partial<Invocation>;
```

Invocation identity (`Invocation` struct).

###### Inherited from

[`GrantedContext`](#grantedcontext).[`invocation`](#invocation-13)

##### jobId?

```ts
jobId?: string;
```

Durable job id (`Invocation.id` of the job open).

##### kv?

```ts
kv?: NamedKv[];
```

`[[kv_namespaces]]` bindings, when granted.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`kv`](#kv-1)

##### queues?

```ts
queues?: NamedQueue[];
```

`[[queues.producers]]` bindings, when granted.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`queues`](#queues-1)

##### secrets?

```ts
secrets?: ExtensibleConfig;
```

Granted secret values as `application/json`.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`secrets`](#secrets-1)

##### storage?

```ts
storage?: Destination;
```

`WORK_FS` object store, when granted.

###### Inherited from

[`GrantedContext`](#grantedcontext).[`storage`](#storage-1)

***

### JobSuspendOptions

Options for [JobController.suspend](#suspend-1).

#### Properties

##### checkpoint?

```ts
checkpoint?: Checkpoint;
```

Bounded checkpoint replayed on resume (at most `maxCheckpointBytes`).

##### checkpointSchemaVersion?

```ts
checkpointSchemaVersion?: number;
```

Schema version of `checkpoint` (default `1`).

##### wakeAt?

```ts
wakeAt?: Instant;
```

Earliest resume instant.

***

### ListOptions

Paginated list request.

#### Properties

##### cursor?

```ts
cursor?: string;
```

##### limit?

```ts
limit?: number;
```

##### prefix?

```ts
prefix?: string;
```

***

### ListPage

One page of keys.

#### Properties

##### nextCursor?

```ts
nextCursor?: string;
```

##### objects

```ts
objects: ObjectInfo[];
```

***

### NamedDatabase

One named `[[databases]]` binding on the granted context: either an
in-isolate [DatabaseBinding](db-execute.md#databasebinding) or the adapter's granted-channel
[GrantedDatabaseTransport](#granteddatabasetransport) the author-side SDK wraps.

#### Properties

##### database?

```ts
database?: DatabaseBinding;
```

Ready binding (same isolate).

##### name

```ts
name: string;
```

Binding name (`env.<name>`).

##### transport?

```ts
transport?: GrantedDatabaseTransport;
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
contentType?: string;
```

##### etag?

```ts
etag?: string;
```

##### key

```ts
key: string;
```

##### sha256?

```ts
sha256?: Uint8Array<ArrayBufferLike>;
```

##### size

```ts
size: number;
```

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
etag?: string;
```

##### key

```ts
key: string;
```

##### sha256?

```ts
sha256?: Uint8Array<ArrayBufferLike>;
```

***

### QueueRetryOptions

Options for [QueueMessage.retry](#retry-1).

#### Properties

##### delaySeconds?

```ts
delaySeconds?: number;
```

Seconds before redelivery; `0` applies the consumer's `retry_delay`.

##### reason?

```ts
reason?: string;
```

Short operator-facing reason.

***

### ReadOptions

Read options for [StorageEntrypoint.get](#get-1).

#### Properties

##### range?

```ts
range?: ByteRange;
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

### RetryOptions

Options for [EventMessage.retry](#retry).

#### Properties

##### delaySeconds?

```ts
delaySeconds?: number;
```

Delay from now, in seconds; ignored when `retryAt` is set.

##### reason?

```ts
reason?: string;
```

Short operator-facing reason.

##### retryAt?

```ts
retryAt?: Instant;
```

Explicit earliest redelivery instant.

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
commitToken?: string;
```

##### contentLength?

```ts
contentLength?: number;
```

##### contentType?

```ts
contentType?: string;
```

##### sha256?

```ts
sha256?: Uint8Array<ArrayBufferLike>;
```

##### stageOnly?

```ts
stageOnly?: boolean;
```

## Type Aliases

### AuthorStub

```ts
type AuthorStub = Pick<BookclerkEntrypoint, 
  | "bookclerkDescribe"
  | "bookclerkEvent"
  | "bookclerkQueue"
  | "bookclerkScheduled"
  | "bookclerkJob"
  | "bookclerkDatabaseMigrations"
| "bookclerkShutdown">;
```

Author default entrypoint as seen through the `PLUGIN` service binding.

***

### Checkpoint

```ts
type Checkpoint = 
  | string
  | JsonObject
  | unknown[]
  | number
  | boolean
  | null;
```

JSON-serializable checkpoint accepted by `suspend()`.

***

### EntrypointName

```ts
type EntrypointName = 
  | "storefront"
  | "storage"
  | "databaseAdapter"
  | "remoteLibrary"
  | "cli"
  | "oidc"
  | "metadata"
  | "covers";
```

Wire name of each named entrypoint (`plugin.toml` `entrypoints`).

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

### Instant

```ts
type Instant = Date | number;
```

Wall-clock instant accepted by `retry` / `suspend`: a `Date` or Unix ms.

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

### NamedStub

```ts
type NamedStub = Pick<NamedEntrypoint, "bookclerkInvoke">;
```

Named author entrypoint as seen through a `PLUGIN_<ENTRYPOINT>` binding.

***

### PluginDescribe

```ts
type PluginDescribe = Partial<PluginDescribe>;
```

Optional identity refinement returned by `BookclerkEntrypoint.describe()`.

The launcher derives the full wire `PluginDescribe` from `plugin.toml`
(`apiVersion`, `id`, `capabilities`, `configOptions`, `cli`, …); an author
`describe()` may add or override presentation fields such as
`displayName`, `brand`, or `rpcFeatures`. Identity, capabilities, and the
`[[settings]]`-derived `configOptions` always come from the manifest: a
guest list that differs from the manifest fails the spawn on the host.

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

### TRIGGER\_FAMILIES

```ts
const TRIGGER_FAMILIES: Readonly<Record<"eventConsumer" | "queueConsumer" | "scheduledHandler" | "jobRunner", string>>;
```

Wire names of the trigger families the launcher may ask `openInvocation` about.

## Functions

### allowedEntrypointFamilies()

```ts
function allowedEntrypointFamilies(capabilities, requested): string[];
```

Filters requested entrypoint / trigger family names against the manifest
capabilities: named entrypoints must be declared in `entrypoints`,
`eventConsumer` needs a non-empty `consumes`, `queueConsumer` needs a
non-empty `queueConsumers`, `scheduledHandler` needs a non-empty `crons`,
and `jobRunner` needs `jobs` or the `storage`
entrypoint (storage guests run the host `stream_copy` job). Unknown names
and a missing manifest fail closed.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `capabilities` | \| `Partial`\<[`PluginCapabilities`](generated.md#plugincapabilities)\> \| `null` \| `undefined` | Manifest capabilities from `PLUGIN_DESCRIBE`. |
| `requested` | readonly `string`[] | Family names the launcher wants to export. |

#### Returns

`string`[]

The authorized subset in request order.

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
| `params` | \| `Pick`\<[`CliInvokeParams`](generated.md#cliinvokeparams), `"args"`\> \| `undefined` | CLI invocation params from the host. |

#### Returns

`Record`\<`string`, `string`\>

Argument values keyed by name.

***

### dataMigrationOp()

```ts
function dataMigrationOp(sql): object;
```

Data DML convenience for [PluginMigration](generated.md#pluginmigration) operations.

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

[`JsonObject`](env.md#jsonobject)

Plain JSON object (`{}` when empty or malformed).

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

### fromBridgeJson()

```ts
function fromBridgeJson(value): unknown;
```

Bridge JSON → typed ABI value: base64 `Data` fields (`payload`, `sha256`,
`credentials`) become `Uint8Array`; everything else is structurally
identical, so no per-struct codec is needed.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `value` | `unknown` | Bridge JSON value. |

#### Returns

`unknown`

Typed struct value.

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

[`Invocation`](generated.md#invocation)

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

[`ExtensibleConfig`](generated.md#extensibleconfig)

Wire-shaped extensible payload.

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

### scheduledControllerOf()

```ts
function scheduledControllerOf(wire): Readonly<ScheduledController>;
```

Frozen author view of one wire `ScheduledController`, with every field
present and typed.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `wire` | \| `Partial`\<[`ScheduledController`](generated.md#scheduledcontroller)\> \| `null` \| `undefined` | Decoded wire struct (fields may be missing). |

#### Returns

`Readonly`\<[`ScheduledController`](generated.md#scheduledcontroller)\>

The controller the author's `scheduled()` receives.

***

### schemaMigrationOp()

```ts
function schemaMigrationOp(sql): object;
```

Schema DDL convenience for [PluginMigration](generated.md#pluginmigration) operations.

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

## References

### CliInvokeParams

Re-exports [CliInvokeParams](generated.md#cliinvokeparams)

***

### CliInvokeResult

Re-exports [CliInvokeResult](generated.md#cliinvokeresult)

***

### CliSchema

Re-exports [CliSchema](generated.md#clischema)

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

### DomainEvent

Re-exports [DomainEvent](generated.md#domainevent)

***

### ExtensibleConfig

Re-exports [ExtensibleConfig](generated.md#extensibleconfig)

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

### HealthOk

Re-exports [HealthOk](generated.md#healthok)

***

### Invocation

Re-exports [Invocation](generated.md#invocation)

***

### JobInvocation

Re-exports [JobInvocation](generated.md#jobinvocation)

***

### MAX\_CHECKPOINT\_BYTES

Re-exports [MAX_CHECKPOINT_BYTES](abi.md#max_checkpoint_bytes)

***

### MAX\_EVENT\_PAYLOAD\_BYTES

Re-exports [MAX_EVENT_PAYLOAD_BYTES](abi.md#max_event_payload_bytes)

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

### MetadataChapter

Re-exports [MetadataChapter](generated.md#metadatachapter)

***

### MetadataChapters

Re-exports [MetadataChapters](generated.md#metadatachapters)

***

### MetadataChaptersParams

Re-exports [MetadataChaptersParams](generated.md#metadatachaptersparams)

***

### MetadataLookupParams

Re-exports [MetadataLookupParams](generated.md#metadatalookupparams)

***

### MetadataRecord

Re-exports [MetadataRecord](generated.md#metadatarecord)

***

### MetadataSearchParams

Re-exports [MetadataSearchParams](generated.md#metadatasearchparams)

***

### OidcClientTemplate

Re-exports [OidcClientTemplate](generated.md#oidcclienttemplate)

***

### PluginCapabilities

Re-exports [PluginCapabilities](generated.md#plugincapabilities)

***

### PluginError

Re-exports [PluginError](README.md#pluginerror)

***

### PluginMigration

Re-exports [PluginMigration](generated.md#pluginmigration)

***

### PluginMigrationOp

Re-exports [PluginMigrationOp](generated.md#pluginmigrationop)

***

### PRODUCT\_API\_VERSION

Re-exports [PRODUCT_API_VERSION](abi.md#product_api_version)

***

### requirePluginMigrationRegistration

Re-exports [requirePluginMigrationRegistration](README.md#requirepluginmigrationregistration)

***

### ScalarLimits

Re-exports [ScalarLimits](generated.md#scalarlimits-1)
