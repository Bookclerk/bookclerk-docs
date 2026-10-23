[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PluginMigration

# Interface: PluginMigration

Defined in: [generated.ts:2917](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2917)

One plugin-owned migration application. `id` is an opaque plugin-chosen
stable identity (name, UUID, timestamp-like string, or digits-as-text).
Bookclerk assigns no order, version, or predecessor meaning to `id`.
Registration order is the forward sequence.

## Properties

### id

> **id**: `string`

Defined in: [generated.ts:2919](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2919)

Opaque plugin-chosen stable identity.

***

### operations

> **operations**: [`PluginMigrationOp`](../type-aliases/PluginMigrationOp.md)[]

Defined in: [generated.ts:2921](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2921)

Operations in forward order; at most `maxPluginMigrationOps`.
