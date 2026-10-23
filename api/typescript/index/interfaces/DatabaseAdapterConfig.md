[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / DatabaseAdapterConfig

# Interface: DatabaseAdapterConfig

Defined in: [generated.ts:1318](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1318)

Author-facing database adapter configuration carried in
`DatabaseContext.adapter`. This is the generic bootstrap mechanism for
third-party adapters: the operator's granted `[database.<id>]` table plus
the scoped writable data dir. First-party host-managed adapters receive
host-private connect params in `DatabaseContext.config` instead.

## Properties

### binding?

> `optional` **binding?**: `string`

Defined in: [generated.ts:1332](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1332)

Named plugin database binding this open serves; empty for the primary
library open. Adapters advertising `DbCapabilities.pluginDatabases` must
serve each binding from its own isolated database.
Omitted when absent (wire zero value).

***

### instanceId?

> `optional` **instanceId?**: `string`

Defined in: [generated.ts:1340](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1340)

Host-issued opaque instance id for this (owner plugin, binding) pair.
Collision-resistant and stable across re-opens. Empty for the primary
library open. Third-party adapters must key isolated databases on this
value rather than `binding` alone (two plugins may both declare `DB`).
Omitted when absent (wire zero value).

***

### openExisting

> **openExisting**: `boolean`

Defined in: [generated.ts:1345](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1345)

When true, open an existing binding unit and do not provision a missing
one (read-only backup capture). False lets the adapter create the unit.

***

### pluginDataDir

> **pluginDataDir**: `string`

Defined in: [generated.ts:1320](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1320)

Scoped writable directory for this plugin (`.../plugins/<id>/data`).

***

### settings

> **settings**: [`ExtensibleConfig`](ExtensibleConfig.md)

Defined in: [generated.ts:1325](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1325)

Granted plugin settings (operator `[database.<id>]` table) as
`application/json`; `{}` when the operator configured nothing.
