[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / StorageEntrypoint

# Class: StorageEntrypoint\<Env\>

Defined in: [plugin.ts:1816](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1816)

`storage` entrypoint (`Destination` interface): an object store the host
writes acquired media into. Export it as
`export class Storage extends StorageEntrypoint`.

## Extends

- [`NamedEntrypoint`](NamedEntrypoint.md)\<`Env`\>

## Type Parameters

### Env

`Env` *extends* [`BookclerkEnv`](../interfaces/BookclerkEnv.md) = [`BookclerkEnv`](../interfaces/BookclerkEnv.md)

## Constructors

### Constructor

> **new StorageEntrypoint**\<`Env`\>(`ctx`, `env`): `StorageEntrypoint`\<`Env`\>

Defined in: [cloudflare-workers.d.ts:15](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L15)

#### Parameters

##### ctx

`ExecutionContext`

##### env

`Env`

#### Returns

`StorageEntrypoint`\<`Env`\>

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

> `static` **bookclerkMethods**: readonly `string`[] = `STORAGE_METHODS`

Defined in: [plugin.ts:1817](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1817)

Methods the adapter may dispatch on this entrypoint.

#### Overrides

[`NamedEntrypoint`](NamedEntrypoint.md).[`bookclerkMethods`](NamedEntrypoint.md#bookclerkmethods)

## Methods

### abortStage()

> **abortStage**(`_key`, `_commitToken`): `Promise`\<`void`\>

Defined in: [plugin.ts:1894](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1894)

Abort a destination-side staged object.

#### Parameters

##### \_key

`string`

Object key.

##### \_commitToken

`string`

Staging token to discard.

#### Returns

`Promise`\<`void`\>

Rejects with typed `unsupported` unless overridden.

***

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

### commit()

> **commit**(`_key`, `_commitToken`): `Promise`\<[`PutResult`](../interfaces/PutResult.md)\>

Defined in: [plugin.ts:1884](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1884)

Finalize a destination-side staged object.

#### Parameters

##### \_key

`string`

Object key.

##### \_commitToken

`string`

Idempotency / commit token.

#### Returns

`Promise`\<[`PutResult`](../interfaces/PutResult.md)\>

Published object metadata.

***

### copy()

> **copy**(`_from`, `_to`): `Promise`\<[`CopyResult`](../interfaces/CopyResult.md)\>

Defined in: [plugin.ts:1865](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1865)

Server-side copy when the backend supports it.

#### Parameters

##### \_from

`string`

Source key.

##### \_to

`string`

Destination key.

#### Returns

`Promise`\<[`CopyResult`](../interfaces/CopyResult.md)\>

Bytes copied.

***

### delete()

> **delete**(`_key`): `Promise`\<`void`\>

Defined in: [plugin.ts:1874](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1874)

Delete a key (no-op if missing).

#### Parameters

##### \_key

`string`

Object key.

#### Returns

`Promise`\<`void`\>

Resolves when the delete is complete.

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

### get()

> **get**(`_key`, `_options?`): `Promise`\<[`ReadResult`](../interfaces/ReadResult.md)\>

Defined in: [plugin.ts:1844](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1844)

Streamed read. The body is a transferred stream, not a scalar.

#### Parameters

##### \_key

`string`

Object key.

##### \_options?

[`ReadOptions`](../interfaces/ReadOptions.md)

Optional byte range.

#### Returns

`Promise`\<[`ReadResult`](../interfaces/ReadResult.md)\>

Metadata plus a transferred body stream.

***

### head()

> **head**(`_key`): `Promise`\<[`ObjectMetadata`](../interfaces/ObjectMetadata.md) \| `null`\>

Defined in: [plugin.ts:1825](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1825)

Metadata without a body; `null` when the key is missing.

#### Parameters

##### \_key

`string`

Object key.

#### Returns

`Promise`\<[`ObjectMetadata`](../interfaces/ObjectMetadata.md) \| `null`\>

Metadata or `null` when the key is missing.

***

### list()

> **list**(`_options`): `Promise`\<[`ListPage`](../interfaces/ListPage.md)\>

Defined in: [plugin.ts:1834](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1834)

One page of keys under `options.prefix`.

#### Parameters

##### \_options

[`ListOptions`](../interfaces/ListOptions.md)

Prefix, cursor, and limit.

#### Returns

`Promise`\<[`ListPage`](../interfaces/ListPage.md)\>

One page of object keys.

***

### put()

> **put**(`_key`, `_body`, `_options?`): `Promise`\<[`PutResult`](../interfaces/PutResult.md)\>

Defined in: [plugin.ts:1855](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1855)

Streamed write. `body` ownership is transferred to the destination.

#### Parameters

##### \_key

`string`

Object key.

##### \_body

`ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\>

Byte stream.

##### \_options?

[`WriteOptions`](../interfaces/WriteOptions.md)

Optional content type / length.

#### Returns

`Promise`\<[`PutResult`](../interfaces/PutResult.md)\>

Bytes written and optional etag / sha256.
