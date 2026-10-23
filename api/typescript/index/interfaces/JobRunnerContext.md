[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / JobRunnerContext

# Interface: JobRunnerContext

Defined in: [plugin.ts:589](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L589)

Job-runner bridge context: the durable job id plus [GrantedContext](GrantedContext.md).

## Extends

- [`GrantedContext`](GrantedContext.md)

## Properties

### config?

> `optional` **config?**: [`ExtensibleConfig`](ExtensibleConfig.md)

Defined in: [plugin.ts:510](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L510)

Operator settings for this plugin as `application/json`.

#### Inherited from

[`GrantedContext`](GrantedContext.md).[`config`](GrantedContext.md#config)

***

### databases?

> `optional` **databases?**: [`NamedDatabase`](NamedDatabase.md)[]

Defined in: [plugin.ts:518](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L518)

Named `[[databases]]` bindings.

#### Inherited from

[`GrantedContext`](GrantedContext.md).[`databases`](GrantedContext.md#databases)

***

### events?

> `optional` **events?**: [`EventPublisherBinding`](EventPublisherBinding.md)

Defined in: [plugin.ts:516](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L516)

`EVENTS` publisher, when `[[events.producers]]` is granted.

#### Inherited from

[`GrantedContext`](GrantedContext.md).[`events`](GrantedContext.md#events)

***

### invocation?

> `optional` **invocation?**: `Partial`\<[`Invocation`](Invocation.md)\>

Defined in: [plugin.ts:508](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L508)

Invocation identity (`Invocation` struct).

#### Inherited from

[`GrantedContext`](GrantedContext.md).[`invocation`](GrantedContext.md#invocation)

***

### jobId?

> `optional` **jobId?**: `string`

Defined in: [plugin.ts:591](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L591)

Durable job id (`Invocation.id` of the job open).

***

### kv?

> `optional` **kv?**: [`NamedKv`](NamedKv.md)[]

Defined in: [plugin.ts:522](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L522)

`[[kv_namespaces]]` bindings, when granted.

#### Inherited from

[`GrantedContext`](GrantedContext.md).[`kv`](GrantedContext.md#kv)

***

### queues?

> `optional` **queues?**: [`NamedQueue`](NamedQueue.md)[]

Defined in: [plugin.ts:520](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L520)

`[[queues.producers]]` bindings, when granted.

#### Inherited from

[`GrantedContext`](GrantedContext.md).[`queues`](GrantedContext.md#queues)

***

### secrets?

> `optional` **secrets?**: [`ExtensibleConfig`](ExtensibleConfig.md)

Defined in: [plugin.ts:512](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L512)

Granted secret values as `application/json`.

#### Inherited from

[`GrantedContext`](GrantedContext.md).[`secrets`](GrantedContext.md#secrets)

***

### storage?

> `optional` **storage?**: [`Destination`](../classes/Destination.md)

Defined in: [plugin.ts:514](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L514)

`WORK_FS` object store, when granted.

#### Inherited from

[`GrantedContext`](GrantedContext.md).[`storage`](GrantedContext.md#storage)
