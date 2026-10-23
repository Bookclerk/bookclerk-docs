[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / DatabaseAdapterEntrypoint

# Class: DatabaseAdapterEntrypoint\<Env\>

Defined in: [plugin.ts:2018](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2018)

`databaseAdapter` entrypoint: opens typed SQL sessions for the host
library. Export it as `export class DatabaseAdapter extends DatabaseAdapterEntrypoint`.

The adapter isolate drives sessions through two dispatch verbs on
[DatabaseAdapterEntrypoint.bookclerkInvoke](#bookclerkinvoke): `openSession` returns a
fresh session id, and `session(id, method, ...params)` calls `method` on
the retained [AdapterDatabaseSession](AdapterDatabaseSession.md); `close` releases the entry.

## Extends

- [`NamedEntrypoint`](NamedEntrypoint.md)\<`Env`\>

## Type Parameters

### Env

`Env` *extends* [`BookclerkEnv`](../interfaces/BookclerkEnv.md) = [`BookclerkEnv`](../interfaces/BookclerkEnv.md)

## Constructors

### Constructor

> **new DatabaseAdapterEntrypoint**\<`Env`\>(`ctx`, `env`): `DatabaseAdapterEntrypoint`\<`Env`\>

Defined in: [cloudflare-workers.d.ts:15](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L15)

#### Parameters

##### ctx

`ExecutionContext`

##### env

`Env`

#### Returns

`DatabaseAdapterEntrypoint`\<`Env`\>

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`constructor`](NamedEntrypoint.md#constructor)

## Properties

### ctx

> `readonly` **ctx**: `ExecutionContext`

Defined in: [cloudflare-workers.d.ts:16](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L16)

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`ctx`](NamedEntrypoint.md#ctx)

***

### env

> `readonly` **env**: `Env`

Defined in: [cloudflare-workers.d.ts:17](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L17)

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`env`](NamedEntrypoint.md#env-1)

***

### invocation

> **invocation**: [`Invocation`](../interfaces/Invocation.md)

Defined in: [plugin.ts:1626](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1626)

Invocation identity of the current call (set by `bookclerkInvoke`).

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`invocation`](NamedEntrypoint.md#invocation)

***

### bookclerkMethods

> `static` **bookclerkMethods**: readonly `string`[]

Defined in: [plugin.ts:2019](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2019)

Methods the adapter may dispatch on this entrypoint.

#### Overrides

[`NamedEntrypoint`](NamedEntrypoint.md).[`bookclerkMethods`](NamedEntrypoint.md#bookclerkmethods)

## Methods

### bookclerkInvoke()

> **bookclerkInvoke**(`context`, `method`, ...`args`): `Promise`\<`unknown`\>

Defined in: [plugin.ts:2050](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2050)

**`Internal`**

Adapter dispatch: `openSession` → session id; `dropUnit`; `session` → call
on a retained session (`args` = `[id, method, ...params]`).

#### Parameters

##### context

[`GrantedContext`](../interfaces/GrantedContext.md)

Granted bindings for this invocation.

##### method

`string`

`openSession`, `dropUnit`, or `session`.

##### args

...`unknown`[]

Method arguments.

#### Returns

`Promise`\<`unknown`\>

Session id for `openSession`, else the session method result.

#### Overrides

[`NamedEntrypoint`](NamedEntrypoint.md).[`bookclerkInvoke`](NamedEntrypoint.md#bookclerkinvoke)

***

### dropUnit()

> **dropUnit**(`_unitRef`): `Promise`\<`void`\>

Defined in: [plugin.ts:2036](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2036)

Physically drops one provisioned binding unit.

#### Parameters

##### \_unitRef

`string`

Adapter-native unit identity.

#### Returns

`Promise`\<`void`\>

Resolves when the unit is dropped.

***

### fetch()

> **fetch**(`_request?`): `Promise`\<`Response`\>

Defined in: [plugin.ts:1634](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1634)

Rejects HTTP fetch — workerd guests are Workers-RPC only.

#### Parameters

##### \_request?

`Request`

Incoming HTTP request when the entrypoint is fetch-facing.

#### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`fetch`](NamedEntrypoint.md#fetch)

***

### openSession()

> **openSession**(): `Promise`\<[`AdapterDatabaseSession`](AdapterDatabaseSession.md)\>

Defined in: [plugin.ts:2026](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2026)

Opens a session. Sessions cannot survive suspension.

#### Returns

`Promise`\<[`AdapterDatabaseSession`](AdapterDatabaseSession.md)\>

Adapter session capability.
