[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvNamespaceBinding

# Interface: KvNamespaceBinding

Defined in: [env.ts:138](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L138)

`[[kv_namespaces]]` binding (`env.<BINDING>`, default `KV`): Workers
`KVNamespace`-shaped durable storage the host serves from the library
database, scoped to this plugin and the namespace the row names. Values
follow the instance across hosts.

## Methods

### delete()

> **delete**(`key`): `Promise`\<`void`\>

Defined in: [env.ts:200](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L200)

Delete one key; a missing key is not an error.

#### Parameters

##### key

`string`

Key to delete.

#### Returns

`Promise`\<`void`\>

***

### get()

#### Call Signature

> **get**(`key`, `type?`): `Promise`\<`string` \| `null`\>

Defined in: [env.ts:146](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L146)

Read one key.

##### Parameters

###### key

`string`

Key, at most `maxKvKeyBytes` UTF-8 bytes.

###### type?

`"text"`

Decoding (`text` by default).

##### Returns

`Promise`\<`string` \| `null`\>

The value, or `null` when absent or expired.

#### Call Signature

> **get**\<`T`\>(`key`, `type`): `Promise`\<`T` \| `null`\>

Defined in: [env.ts:154](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L154)

Read one key and parse it as JSON.

##### Type Parameters

###### T

`T` = `unknown`

##### Parameters

###### key

`string`

Key, at most `maxKvKeyBytes` UTF-8 bytes.

###### type

`"json"`

`json`.

##### Returns

`Promise`\<`T` \| `null`\>

The parsed value, or `null` when absent or expired.

#### Call Signature

> **get**(`key`, `type`): `Promise`\<`ArrayBuffer` \| `null`\>

Defined in: [env.ts:162](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L162)

Read one key as bytes.

##### Parameters

###### key

`string`

Key, at most `maxKvKeyBytes` UTF-8 bytes.

###### type

`"arrayBuffer"`

`arrayBuffer`.

##### Returns

`Promise`\<`ArrayBuffer` \| `null`\>

The bytes, or `null` when absent or expired.

***

### getWithMetadata()

#### Call Signature

> **getWithMetadata**(`key`, `type?`): `Promise`\<[`KvValueWithMetadata`](KvValueWithMetadata.md)\<`string`\>\>

Defined in: [env.ts:170](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L170)

Read one key with its metadata.

##### Parameters

###### key

`string`

Key to read.

###### type?

`"text"`

Decoding (`text` by default).

##### Returns

`Promise`\<[`KvValueWithMetadata`](KvValueWithMetadata.md)\<`string`\>\>

Value and metadata; both `null` when absent.

#### Call Signature

> **getWithMetadata**\<`T`\>(`key`, `type`): `Promise`\<[`KvValueWithMetadata`](KvValueWithMetadata.md)\<`T`\>\>

Defined in: [env.ts:178](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L178)

Read one key as JSON with its metadata.

##### Type Parameters

###### T

`T` = `unknown`

##### Parameters

###### key

`string`

Key to read.

###### type

`"json"`

`json`.

##### Returns

`Promise`\<[`KvValueWithMetadata`](KvValueWithMetadata.md)\<`T`\>\>

Parsed value and metadata; both `null` when absent.

#### Call Signature

> **getWithMetadata**(`key`, `type`): `Promise`\<[`KvValueWithMetadata`](KvValueWithMetadata.md)\<`ArrayBuffer`\>\>

Defined in: [env.ts:186](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L186)

Read one key as bytes with its metadata.

##### Parameters

###### key

`string`

Key to read.

###### type

`"arrayBuffer"`

`arrayBuffer`.

##### Returns

`Promise`\<[`KvValueWithMetadata`](KvValueWithMetadata.md)\<`ArrayBuffer`\>\>

Bytes and metadata; both `null` when absent.

***

### list()

> **list**(`options?`): `Promise`\<[`KvListResultBinding`](KvListResultBinding.md)\>

Defined in: [env.ts:207](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L207)

List keys in lexical order, one page at a time.

#### Parameters

##### options?

[`KvListOptionsBinding`](KvListOptionsBinding.md)

Prefix, cursor, and page size.

#### Returns

`Promise`\<[`KvListResultBinding`](KvListResultBinding.md)\>

One page of keys.

***

### put()

> **put**(`key`, `value`, `options?`): `Promise`\<`void`\>

Defined in: [env.ts:194](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L194)

Write one key, replacing any previous value and metadata.

#### Parameters

##### key

`string`

Key to write.

##### value

`string` \| `ArrayBuffer` \| `ArrayBufferView`\<`ArrayBufferLike`\>

String (stored as UTF-8) or bytes.

##### options?

[`KvPutOptionsBinding`](KvPutOptionsBinding.md)

Expiration and metadata.

#### Returns

`Promise`\<`void`\>
