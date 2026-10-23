[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / Destination

# Class: Destination

Defined in: [plugin.ts:281](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L281)

Object store capability: `job.output` and the `WORK_FS` binding. The
runtime stub *is* the capability; abort is stream cancel.

## Extends

- `RpcTarget`

## Constructors

### Constructor

> **new Destination**(): `Destination`

#### Returns

`Destination`

#### Inherited from

`RpcTarget.constructor`

## Methods

### abortStage()

> **abortStage**(`_key`, `_commitToken`): `Promise`\<`void`\>

Defined in: [plugin.ts:368](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L368)

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

### commit()

> **commit**(`_key`, `_commitToken`): `Promise`\<[`PutResult`](../interfaces/PutResult.md)\>

Defined in: [plugin.ts:357](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L357)

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

Defined in: [plugin.ts:336](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L336)

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

Defined in: [plugin.ts:346](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L346)

Delete a key (no-op if missing).

#### Parameters

##### \_key

`string`

Object key.

#### Returns

`Promise`\<`void`\>

Resolves when the delete is complete.

***

### get()

> **get**(`_key`, `_options?`): `Promise`\<[`ReadResult`](../interfaces/ReadResult.md)\>

Defined in: [plugin.ts:309](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L309)

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

Defined in: [plugin.ts:288](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L288)

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

Defined in: [plugin.ts:298](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L298)

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

Defined in: [plugin.ts:321](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L321)

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
