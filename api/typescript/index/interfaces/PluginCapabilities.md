[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PluginCapabilities

# Interface: PluginCapabilities

Defined in: [generated.ts:1160](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1160)

Typed capability declaration returned by `describe()`. The host compares
it with `plugin.toml` and the operator grant; widening is rejected at spawn.

## Properties

### bindings

> **bindings**: `string`[]

Defined in: [generated.ts:1175](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1175)

Other named bindings the guest expects on `env` (`CONFIG`, `SECRETS`,
`WORK_FS`, `OAUTH`, `KV`, `EVENTS`, `[[queues.producers]]` names, ...).

***

### consumes

> **consumes**: [`EventConsumerSpec`](EventConsumerSpec.md)[]

Defined in: [generated.ts:1164](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1164)

Event types the default entrypoint consumes (`event(batch)` trigger).

***

### databases

> **databases**: `string`[]

Defined in: [generated.ts:1170](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1170)

Plugin-owned database binding names (`[[databases]]`).

***

### entrypoints

> **entrypoints**: (`"storefront"` \| `"storage"` \| `"databaseAdapter"` \| `"remoteLibrary"` \| `"cli"` \| `"oidc"`)[]

Defined in: [generated.ts:1162](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1162)

Named entrypoints the guest exports.

***

### jobs

> **jobs**: `string`[]

Defined in: [generated.ts:1168](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1168)

Command types the default entrypoint runs (`job(controller)` trigger).

***

### produces

> **produces**: `string`[]

Defined in: [generated.ts:1166](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1166)

Event types the guest may publish through its `EVENTS` binding.

***

### queueConsumers

> **queueConsumers**: `string`[]

Defined in: [generated.ts:1177](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1177)

Queues the default entrypoint consumes (`queue(batch)` trigger).

***

### queueProducers

> **queueProducers**: `string`[]

Defined in: [generated.ts:1179](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1179)

Queues the guest may send to through its `[[queues.producers]]` bindings.
