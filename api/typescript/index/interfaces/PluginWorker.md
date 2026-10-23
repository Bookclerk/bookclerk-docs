[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PluginWorker

# Interface: PluginWorker

Defined in: [generated.ts:2943](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2943)

Plugin bootstrap capability: the guest's root object (Workers `default`
export analogue). `describe()` first; `open()` once per invocation.

## Methods

### databaseMigrations()

> **databaseMigrations**(`binding`): `Promise`\<[`PluginMigrationsReply`](../type-aliases/PluginMigrationsReply.md)\>

Defined in: [generated.ts:2977](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2977)

Complete ordered plugin-owned migration sequence for one named binding.
The host calls this before `open` while provisioning `[[databases]]`
(a binding must be migrated before any `Bindings.databases` session is
handed out). Empty list means the binding has no plugin-owned migrations.
Bounded by `maxListPage` / `maxPluginMigrationOps` /
`maxPluginMigrationTotalOps` / `maxScalarBytes` /
`maxPluginMigrationRegistrationBytes`.

#### Parameters

##### binding

`string`

Binding name from `plugin.toml`.

#### Returns

`Promise`\<[`PluginMigrationsReply`](../type-aliases/PluginMigrationsReply.md)\>

[PluginMigrationsReply](../type-aliases/PluginMigrationsReply.md)

***

### describe()

> **describe**(): `Promise`\<[`DescribeReply`](../type-aliases/DescribeReply.md)\>

Defined in: [generated.ts:2949](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2949)

Identity, ABI version, negotiated features, and typed capabilities.

#### Returns

`Promise`\<[`DescribeReply`](../type-aliases/DescribeReply.md)\>

[DescribeReply](../type-aliases/DescribeReply.md)

***

### open()

> **open**(`invocation`, `bindings`): `Promise`\<[`EntrypointsReply`](../type-aliases/EntrypointsReply.md)\>

Defined in: [generated.ts:2958](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2958)

Open the exported entrypoints for one invocation with host-granted
bindings. Entrypoints the manifest does not export are null.

#### Parameters

##### invocation

[`Invocation`](Invocation.md)

Host-issued invocation identity.

##### bindings

[`Bindings`](Bindings.md)

Granted `env` bindings.

#### Returns

`Promise`\<[`EntrypointsReply`](../type-aliases/EntrypointsReply.md)\>

[EntrypointsReply](../type-aliases/EntrypointsReply.md)

***

### shutdown()

> **shutdown**(): `Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

Defined in: [generated.ts:2964](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2964)

Flush and release resources before the process exits.

#### Returns

`Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

[EmptyReply](../type-aliases/EmptyReply.md)
