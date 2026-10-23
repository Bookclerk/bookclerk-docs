[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / RemoteLibraryEntrypoint

# Class: RemoteLibraryEntrypoint\<Env\>

Defined in: [plugin.ts:1914](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1914)

`remoteLibrary` entrypoint: lifecycle, rescan, listening sync, and user
polling for a remote library server. Export it as
`export class RemoteLibrary extends RemoteLibraryEntrypoint`.

## Extends

- [`NamedEntrypoint`](NamedEntrypoint.md)\<`Env`\>

## Type Parameters

### Env

`Env` *extends* [`BookclerkEnv`](../interfaces/BookclerkEnv.md) = [`BookclerkEnv`](../interfaces/BookclerkEnv.md)

## Constructors

### Constructor

> **new RemoteLibraryEntrypoint**\<`Env`\>(`ctx`, `env`): `RemoteLibraryEntrypoint`\<`Env`\>

Defined in: [cloudflare-workers.d.ts:15](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L15)

#### Parameters

##### ctx

`ExecutionContext`

##### env

`Env`

#### Returns

`RemoteLibraryEntrypoint`\<`Env`\>

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

> `static` **bookclerkMethods**: readonly `string`[] = `REMOTE_LIBRARY_METHODS`

Defined in: [plugin.ts:1915](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1915)

Methods the adapter may dispatch on this entrypoint.

#### Overrides

[`NamedEntrypoint`](NamedEntrypoint.md).[`bookclerkMethods`](NamedEntrypoint.md#bookclerkmethods)

## Methods

### bookclerkInvoke()

> **bookclerkInvoke**(`context`, `method`, ...`args`): `Promise`\<`unknown`\>

Defined in: [plugin.ts:1647](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1647)

**`Internal`**

Adapter dispatch: install granted bindings, then call `method`.

#### Parameters

##### context

[`GrantedContext`](../interfaces/GrantedContext.md)

Granted bindings for this invocation.

##### method

`string`

Method name from the static allowlist.

##### args

...`unknown`[]

Method arguments.

#### Returns

`Promise`\<`unknown`\>

Method result.

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`bookclerkInvoke`](NamedEntrypoint.md#bookclerkinvoke)

***

### diagnose()

> **diagnose**(): `Promise`\<`string`[]\>

Defined in: [plugin.ts:1930](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1930)

Operator-facing diagnostic lines.

#### Returns

`Promise`\<`string`[]\>

Probe lines.

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

### health()

> **health**(): `Promise`\<[`HealthOk`](../interfaces/HealthOk.md)\>

Defined in: [plugin.ts:1922](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1922)

Reports whether the remote library session is usable.

#### Returns

`Promise`\<[`HealthOk`](../interfaces/HealthOk.md)\>

Health flag plus detail.

***

### pollEvents()

> **pollEvents**(): `Promise`\<[`ExternalUser`](../interfaces/ExternalUser.md)[]\>

Defined in: [plugin.ts:1971](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1971)

Drains external users observed since the last poll.

#### Returns

`Promise`\<[`ExternalUser`](../interfaces/ExternalUser.md)[]\>

Newly observed users.

***

### scanLibrary()

> **scanLibrary**(`_params`): `Promise`\<`void`\>

Defined in: [plugin.ts:1955](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1955)

Re-syncs the remote library.

#### Parameters

##### \_params

[`ScanLibraryParams`](../interfaces/ScanLibraryParams.md)

Scan scope.

#### Returns

`Promise`\<`void`\>

Resolves when the scan has been accepted.

***

### start()

> **start**(): `Promise`\<`void`\>

Defined in: [plugin.ts:1938](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1938)

Starts long-running work for this invocation.

#### Returns

`Promise`\<`void`\>

Resolves when running.

***

### stop()

> **stop**(): `Promise`\<`void`\>

Defined in: [plugin.ts:1946](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1946)

Stops long-running work for this invocation.

#### Returns

`Promise`\<`void`\>

Resolves when stopped.

***

### syncListening()

> **syncListening**(): `Promise`\<[`ListeningProgress`](../interfaces/ListeningProgress.md)[]\>

Defined in: [plugin.ts:1963](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1963)

Push / pull listening progress; the host upserts the rows.

#### Returns

`Promise`\<[`ListeningProgress`](../interfaces/ListeningProgress.md)[]\>

Progress rows.
