[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / GrantedContext

# Interface: GrantedContext

Defined in: [plugin.ts:506](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L506)

Granted `Bindings` for one invocation as the bridge routes carry them:
the `context` body field / `x-bookclerk-context` header decoded from
`PluginWorker.open(invocation, bindings)`. The adapter merges it onto the
author's `env` before dispatching; authors never receive it directly.

## Extended by

- [`JobRunnerContext`](JobRunnerContext.md)

## Properties

### config?

> `optional` **config?**: [`ExtensibleConfig`](ExtensibleConfig.md)

Defined in: [plugin.ts:510](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L510)

Operator settings for this plugin as `application/json`.

***

### databases?

> `optional` **databases?**: [`NamedDatabase`](NamedDatabase.md)[]

Defined in: [plugin.ts:518](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L518)

Named `[[databases]]` bindings.

***

### events?

> `optional` **events?**: [`EventPublisherBinding`](EventPublisherBinding.md)

Defined in: [plugin.ts:516](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L516)

`EVENTS` publisher, when `[[events.producers]]` is granted.

***

### invocation?

> `optional` **invocation?**: `Partial`\<[`Invocation`](Invocation.md)\>

Defined in: [plugin.ts:508](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L508)

Invocation identity (`Invocation` struct).

***

### kv?

> `optional` **kv?**: [`NamedKv`](NamedKv.md)[]

Defined in: [plugin.ts:522](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L522)

`[[kv_namespaces]]` bindings, when granted.

***

### queues?

> `optional` **queues?**: [`NamedQueue`](NamedQueue.md)[]

Defined in: [plugin.ts:520](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L520)

`[[queues.producers]]` bindings, when granted.

***

### secrets?

> `optional` **secrets?**: [`ExtensibleConfig`](ExtensibleConfig.md)

Defined in: [plugin.ts:512](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L512)

Granted secret values as `application/json`.

***

### storage?

> `optional` **storage?**: [`Destination`](../classes/Destination.md)

Defined in: [plugin.ts:514](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L514)

`WORK_FS` object store, when granted.
