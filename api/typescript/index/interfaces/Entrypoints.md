[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / Entrypoints

# Interface: Entrypoints

Defined in: [generated.ts:303](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L303)

Exported entrypoints returned by `PluginWorker.open`. One capability per
`plugin.toml` `entrypoints` entry / trigger; null when not exported. The host
refuses an entrypoint the manifest or operator grant did not allow.

## Properties

### cli?

> `optional` **cli?**: [`PluginCli`](PluginCli.md) \| `null`

Defined in: [generated.ts:338](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L338)

`cli` entrypoint.
Null when the guest does not export this capability.

***

### databaseAdapter?

> `optional` **databaseAdapter?**: [`Database`](Database.md) \| `null`

Defined in: [generated.ts:328](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L328)

`databaseAdapter` entrypoint.
Null when the guest does not export this capability.

***

### eventConsumer?

> `optional` **eventConsumer?**: [`EventConsumer`](EventConsumer.md) \| `null`

Defined in: [generated.ts:308](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L308)

`[[events.consumers]]` trigger: `event(batch)` handler.
Null when the guest does not export this capability.

***

### jobRunner?

> `optional` **jobRunner?**: [`JobRunner`](JobRunner.md) \| `null`

Defined in: [generated.ts:313](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L313)

`[triggers] jobs` trigger: `job(controller)` handler.
Null when the guest does not export this capability.

***

### oidc?

> `optional` **oidc?**: [`Oidc`](Oidc.md) \| `null`

Defined in: [generated.ts:343](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L343)

`oidc` entrypoint.
Null when the guest does not export this capability.

***

### queueConsumer?

> `optional` **queueConsumer?**: [`QueueConsumer`](QueueConsumer.md) \| `null`

Defined in: [generated.ts:348](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L348)

`[[queues.consumers]]` trigger: `queue(batch)` handler.
Null when the guest does not export this capability.

***

### remoteLibrary?

> `optional` **remoteLibrary?**: [`RemoteLibrary`](RemoteLibrary.md) \| `null`

Defined in: [generated.ts:333](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L333)

`remoteLibrary` entrypoint.
Null when the guest does not export this capability.

***

### storage?

> `optional` **storage?**: `Destination` \| `null`

Defined in: [generated.ts:323](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L323)

`storage` entrypoint.
Null when the guest does not export this capability.

***

### storefront?

> `optional` **storefront?**: [`ContentSource`](ContentSource.md) \| `null`

Defined in: [generated.ts:318](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L318)

`storefront` entrypoint.
Null when the guest does not export this capability.
