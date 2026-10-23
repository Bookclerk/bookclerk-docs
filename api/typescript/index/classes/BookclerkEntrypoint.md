[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / BookclerkEntrypoint

# Class: BookclerkEntrypoint\<Env\>

Defined in: [plugin.ts:1466](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1466)

Default-export base. Optional trigger methods: [BookclerkEntrypoint.event](#event)
and [BookclerkEntrypoint.job](#job); optional [BookclerkEntrypoint.describe](#describe)
(identity beyond `plugin.toml`), [BookclerkEntrypoint.databaseMigrations](#databasemigrations),
and [BookclerkEntrypoint.shutdown](#shutdown).

The `bookclerk*` methods are the adapter-facing dispatch surface; authors
neither call nor override them.

## Example

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

## Extends

- `WorkerEntrypoint`\<`Env`\>

## Type Parameters

### Env

`Env` *extends* [`BookclerkEnv`](../interfaces/BookclerkEnv.md) = [`BookclerkEnv`](../interfaces/BookclerkEnv.md)

## Constructors

### Constructor

> **new BookclerkEntrypoint**\<`Env`\>(`ctx`, `env`): `BookclerkEntrypoint`\<`Env`\>

Defined in: [cloudflare-workers.d.ts:15](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L15)

#### Parameters

##### ctx

`ExecutionContext`

##### env

`Env`

#### Returns

`BookclerkEntrypoint`\<`Env`\>

#### Inherited from

`WorkerEntrypoint<Env>.constructor`

## Properties

### ctx

> `readonly` **ctx**: `ExecutionContext`

Defined in: [cloudflare-workers.d.ts:16](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L16)

#### Inherited from

`WorkerEntrypoint.ctx`

***

### env

> `readonly` **env**: `Env`

Defined in: [cloudflare-workers.d.ts:17](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L17)

#### Inherited from

`WorkerEntrypoint.env`

## Methods

### bookclerkDatabaseMigrations()

> **bookclerkDatabaseMigrations**(`binding`): `Promise`\<[`PluginMigration`](../interfaces/PluginMigration.md)[]\>

Defined in: [plugin.ts:1599](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1599)

**`Internal`**

Adapter dispatch for `databaseMigrations(binding)`.

#### Parameters

##### binding

`string`

Binding name from `[[databases]]`.

#### Returns

`Promise`\<[`PluginMigration`](../interfaces/PluginMigration.md)[]\>

Bounded registration (`[]` when not implemented).

***

### bookclerkDescribe()

> **bookclerkDescribe**(): `Promise`\<`Partial`\<`PluginDescribe`\> \| `null`\>

Defined in: [plugin.ts:1514](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1514)

**`Internal`**

Adapter dispatch for `describe()`.

#### Returns

`Promise`\<`Partial`\<`PluginDescribe`\> \| `null`\>

Author refinement or `null` when not implemented.

***

### bookclerkEvent()

> **bookclerkEvent**(`context`, `wireBatch`): `Promise`\<[`EventOutcome`](../type-aliases/EventOutcome.md)[]\>

Defined in: [plugin.ts:1527](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1527)

**`Internal`**

Adapter dispatch for the `event(batch)` trigger.

#### Parameters

##### context

[`GrantedContext`](../interfaces/GrantedContext.md)

Granted bindings for this invocation.

##### wireBatch

[`WireEventBatch`](../interfaces/WireEventBatch.md)

Wire `EventBatch`.

#### Returns

`Promise`\<[`EventOutcome`](../type-aliases/EventOutcome.md)[]\>

One outcome per event, in order.

***

### bookclerkJob()

> **bookclerkJob**(`context`, `invocation`, `granted`): `Promise`\<[`JobOutcomeRecord`](../type-aliases/JobOutcomeRecord.md)\>

Defined in: [plugin.ts:1574](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1574)

**`Internal`**

Adapter dispatch for the `job(controller)` trigger.

#### Parameters

##### context

[`GrantedContext`](../interfaces/GrantedContext.md)

Granted bindings for this invocation.

##### invocation

`Partial`\<[`JobInvocation`](../interfaces/JobInvocation.md)\>

Durable command envelope.

##### granted

[`GrantedJobCapabilities`](../interfaces/GrantedJobCapabilities.md)

Granted input / output / progress / cancel stubs.

#### Returns

`Promise`\<[`JobOutcomeRecord`](../type-aliases/JobOutcomeRecord.md)\>

Wire job outcome.

***

### bookclerkQueue()

> **bookclerkQueue**(`context`, `wireBatch`): `Promise`\<[`QueueOutcome`](../type-aliases/QueueOutcome.md)[]\>

Defined in: [plugin.ts:1550](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1550)

**`Internal`**

Adapter dispatch for the `queue(batch)` trigger.

#### Parameters

##### context

[`GrantedContext`](../interfaces/GrantedContext.md)

Granted bindings for this invocation.

##### wireBatch

[`WireQueueBatch`](../interfaces/WireQueueBatch.md)

Wire `QueueBatch`.

#### Returns

`Promise`\<[`QueueOutcome`](../type-aliases/QueueOutcome.md)[]\>

One outcome per message, in order.

***

### bookclerkShutdown()

> **bookclerkShutdown**(): `Promise`\<`void`\>

Defined in: [plugin.ts:1611](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1611)

**`Internal`**

Adapter dispatch for `shutdown()`.

#### Returns

`Promise`\<`void`\>

Resolves when the author hook has run.

***

### databaseMigrations()?

> `optional` **databaseMigrations**(`binding`): [`PluginMigration`](../interfaces/PluginMigration.md)[] \| `Promise`\<[`PluginMigration`](../interfaces/PluginMigration.md)[]\>

Defined in: [plugin.ts:1503](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1503)

Complete ordered plugin-owned migration sequence for one named binding.
The host calls this at binding initialization, before ordinary execute.

#### Parameters

##### binding

`string`

#### Returns

[`PluginMigration`](../interfaces/PluginMigration.md)[] \| `Promise`\<[`PluginMigration`](../interfaces/PluginMigration.md)[]\>

***

### describe()?

> `optional` **describe**(): `Partial`\<`PluginDescribe`\> \| `Promise`\<`Partial`\<`PluginDescribe`\>\>

Defined in: [plugin.ts:1478](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1478)

Optional identity refinement merged over the manifest projection.

#### Returns

`Partial`\<`PluginDescribe`\> \| `Promise`\<`Partial`\<`PluginDescribe`\>\>

***

### event()?

> `optional` **event**(`batch`): `void` \| `Promise`\<`void`\>

Defined in: [plugin.ts:1484](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1484)

`[[events.consumers]]` trigger: handle one delivered batch. Record an
outcome per message; untouched messages ack on return and retry on throw.

#### Parameters

##### batch

[`EventBatch`](EventBatch.md)

#### Returns

`void` \| `Promise`\<`void`\>

***

### fetch()

> **fetch**(`_request?`): `Promise`\<`Response`\>

Defined in: [plugin.ts:1473](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1473)

Rejects HTTP fetch — workerd guests are Workers-RPC only.

#### Parameters

##### \_request?

`Request`

Incoming HTTP request when the entrypoint is fetch-facing.

#### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

***

### job()?

> `optional` **job**(`job`): `void` \| [`JobCompletion`](../interfaces/JobCompletion.md) \| `Promise`\<`void` \| [`JobCompletion`](../interfaces/JobCompletion.md)\>

Defined in: [plugin.ts:1497](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1497)

`[triggers] jobs` trigger: run one durable job. Return to complete,
throw to reject, or call `job.suspend()` / `job.retryLater()`.

#### Parameters

##### job

[`JobController`](JobController.md)

#### Returns

`void` \| [`JobCompletion`](../interfaces/JobCompletion.md) \| `Promise`\<`void` \| [`JobCompletion`](../interfaces/JobCompletion.md)\>

***

### queue()?

> `optional` **queue**(`batch`): `void` \| `Promise`\<`void`\>

Defined in: [plugin.ts:1491](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1491)

`[[queues.consumers]]` trigger: handle one delivered batch from one
queue. Record an outcome per message; untouched messages ack on return
and retry on throw.

#### Parameters

##### batch

[`QueueBatch`](QueueBatch.md)

#### Returns

`void` \| `Promise`\<`void`\>

***

### shutdown()?

> `optional` **shutdown**(): `void` \| `Promise`\<`void`\>

Defined in: [plugin.ts:1506](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1506)

Releases guest resources.

#### Returns

`void` \| `Promise`\<`void`\>
