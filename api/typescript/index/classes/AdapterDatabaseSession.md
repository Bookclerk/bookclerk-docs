[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / AdapterDatabaseSession

# Class: AdapterDatabaseSession

Defined in: [plugin.ts:401](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L401)

Host ↔ database adapter session returned by [DatabaseAdapterEntrypoint.openSession](DatabaseAdapterEntrypoint.md#opensession).

## Extends

- `RpcTarget`

## Constructors

### Constructor

> **new AdapterDatabaseSession**(): `AdapterDatabaseSession`

#### Returns

`AdapterDatabaseSession`

#### Inherited from

`RpcTarget.constructor`

## Methods

### assertRestoreConstraints()

> **assertRestoreConstraints**(): `Promise`\<`void`\>

Defined in: [plugin.ts:491](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L491)

Re-check constraints after a unit restore; optional.

#### Returns

`Promise`\<`void`\>

Resolves when every constraint holds.

***

### bootstrap()

> **bootstrap**(): `Promise`\<[`DbBootstrap`](../interfaces/DbBootstrap.md)\>

Defined in: [plugin.ts:435](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L435)

Bootstrap-only diagnostic metadata (`DbBootstrap`); optional.

#### Returns

`Promise`\<[`DbBootstrap`](../interfaces/DbBootstrap.md)\>

Engine identity for diagnostics.

***

### capabilities()

> **capabilities**(): `Promise`\<`unknown`\>

Defined in: [plugin.ts:407](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L407)

Typed SQL-contract advertisement.

#### Returns

`Promise`\<`unknown`\>

Guest `DbCapabilities`.

***

### close()

> **close**(): `Promise`\<`void`\>

Defined in: [plugin.ts:426](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L426)

Close the adapter session.

#### Returns

`Promise`\<`void`\>

Resolves when the session is closed.

***

### dropUserRelations()

> **dropUserRelations**(`_names`): `Promise`\<`void`\>

Defined in: [plugin.ts:482](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L482)

Drop the named user relations; optional.

#### Parameters

##### \_names

`string`[]

Relation names from [AdapterDatabaseSession.listUserRelations](#listuserrelations).

#### Returns

`Promise`\<`void`\>

Resolves when the relations are gone.

***

### execute()

> **execute**(`_request`): `Promise`\<[`ExecuteReply`](../interfaces/ExecuteReply.md)\>

Defined in: [plugin.ts:417](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L417)

Typed atomic batch (`ExecuteRequest` → `ExecuteReply`).

#### Parameters

##### \_request

[`ExecuteRequest`](../interfaces/ExecuteRequest.md)

Cap'n `ExecuteRequest` (structured Workers RPC object).

#### Returns

`Promise`\<[`ExecuteReply`](../interfaces/ExecuteReply.md)\>

`ExecuteReply`.

***

### exportIdentity()

> **exportIdentity**(): `Promise`\<[`IdentityHighWater`](../interfaces/IdentityHighWater.md)[]\>

Defined in: [plugin.ts:444](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L444)

Identity high-water marks for a logical export; optional.

#### Returns

`Promise`\<[`IdentityHighWater`](../interfaces/IdentityHighWater.md)[]\>

One row per identity column.

***

### importIdentity()

> **importIdentity**(`_rows`): `Promise`\<`void`\>

Defined in: [plugin.ts:454](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L454)

Restore identity high-water marks after a logical import; optional.

#### Parameters

##### \_rows

[`IdentityHighWater`](../interfaces/IdentityHighWater.md)[]

Rows from an earlier [AdapterDatabaseSession.exportIdentity](#exportidentity).

#### Returns

`Promise`\<`void`\>

Resolves when the marks are applied.

***

### listUserRelations()

> **listUserRelations**(): `Promise`\<`string`[]\>

Defined in: [plugin.ts:463](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L463)

Names of user relations the host may drop before a restore; optional.

#### Returns

`Promise`\<`string`[]\>

Relation names.

***

### prepareUnitRestore()

> **prepareUnitRestore**(): `Promise`\<`void`\>

Defined in: [plugin.ts:472](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L472)

Prepare the engine for a unit restore (disable constraints, …); optional.

#### Returns

`Promise`\<`void`\>

Resolves when the engine is ready for the restore.
