[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / Bindings

# Interface: Bindings

Defined in: [generated.ts:245](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L245)

Host-granted bindings for one `PluginWorker.open` (`env` in the Workers
idiom). OS paths, FDs, and sockets are transport-private. Capability fields
are null when the manifest does not declare (or the operator did not grant)
the binding.

## Properties

### adapter

> **adapter**: [`DatabaseAdapterConfig`](DatabaseAdapterConfig.md)

Defined in: [generated.ts:261](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L261)

Database-adapter bootstrap for the `databaseAdapter` entrypoint. First-party
host-managed adapters receive host-private connect params in `config`;
third-party adapters receive this typed bootstrap (and an empty `config`).

***

### cancel?

> `optional` **cancel?**: [`Cancellation`](Cancellation.md) \| `null`

Defined in: [generated.ts:277](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L277)

Host cancellation for the whole invocation (fence / lease loss).
Null when the guest does not export this capability.

***

### config

> **config**: [`ExtensibleConfig`](ExtensibleConfig.md)

Defined in: [generated.ts:250](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L250)

`CONFIG`: granted plugin settings (operator `[vars]` plus the family
settings table) as `application/json`; at most `maxConfigPayloadBytes`.

***

### databases

> **databases**: `NamedDatabase`[]

Defined in: [generated.ts:272](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L272)

Named plugin-owned `[[databases]]` bindings: each entry is an isolated
database provisioned by the active adapter, separate from the Bookclerk
library and from every other plugin. Empty when the manifest declares none.

***

### events?

> `optional` **events?**: [`EventPublisher`](EventPublisher.md) \| `null`

Defined in: [generated.ts:266](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L266)

`EVENTS`: outbox publisher; null unless `[[events.producers]]` is granted.
Null when the guest does not export this capability.

***

### kv

> **kv**: `NamedKv`[]

Defined in: [generated.ts:295](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L295)

Named `[[kv_namespaces]]` bindings: one host-served durable namespace per
binding, scoped to this plugin. Empty when the manifest declares none.

***

### queues

> **queues**: `NamedQueue`[]

Defined in: [generated.ts:290](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L290)

Named `[[queues.producers]]` bindings: one host-served producer per
binding, each sending into the queue the row names. Empty when the
manifest declares none.

***

### secrets

> **secrets**: [`ExtensibleConfig`](ExtensibleConfig.md)

Defined in: [generated.ts:255](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L255)

`SECRETS`: granted secret values as `application/json`; empty payload
when the manifest declares no `[secrets]`.

***

### storage?

> `optional` **storage?**: `Destination` \| `null`

Defined in: [generated.ts:284](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L284)

`WORK_FS`: host-granted object storage for durable plugin files (work
filesystem); null unless `[work_fs]` is granted. Job input/output travel on
`JobRunner.job(controller)`, never here.
Null when the guest does not export this capability.
