[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvNamespace

# Interface: KvNamespace

Defined in: [generated.ts:914](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L914)

Durable key/value namespace: a Workers `KVNamespace`-shaped binding the host
serves from the library database, scoped to the plugin and the namespace a
`[[kv_namespaces]]` row names, granted through `Bindings.kv`. Values live in
the cluster, so an instance moved to another host reads its own keys.

## Methods

### delete()

> **delete**(`key`): `Promise`\<[`KvDeleteReply`](../type-aliases/KvDeleteReply.md)\>

Defined in: [generated.ts:937](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L937)

Delete one key; a missing key is not an error.

#### Parameters

##### key

`string`

Key to delete.

#### Returns

`Promise`\<[`KvDeleteReply`](../type-aliases/KvDeleteReply.md)\>

[KvDeleteReply](../type-aliases/KvDeleteReply.md)

***

### get()

> **get**(`key`): `Promise`\<[`KvGetReply`](../type-aliases/KvGetReply.md)\>

Defined in: [generated.ts:921](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L921)

Read one key; `found = false` when absent or expired.

#### Parameters

##### key

`string`

Key, at most `maxKvKeyBytes` UTF-8 bytes.

#### Returns

`Promise`\<[`KvGetReply`](../type-aliases/KvGetReply.md)\>

[KvGetReply](../type-aliases/KvGetReply.md)

***

### list()

> **list**(`options`): `Promise`\<[`KvListReply`](../type-aliases/KvListReply.md)\>

Defined in: [generated.ts:944](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L944)

List keys in lexical order, one page at a time.

#### Parameters

##### options

[`KvListOptions`](KvListOptions.md)

Prefix, cursor, and page size.

#### Returns

`Promise`\<[`KvListReply`](../type-aliases/KvListReply.md)\>

[KvListReply](../type-aliases/KvListReply.md)

***

### put()

> **put**(`key`, `value`, `options`): `Promise`\<[`KvPutReply`](../type-aliases/KvPutReply.md)\>

Defined in: [generated.ts:930](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L930)

Write one key, replacing any previous value and metadata.

#### Parameters

##### key

`string`

Key, at most `maxKvKeyBytes` UTF-8 bytes.

##### value

`Uint8Array`

Value, at most the namespace's `max_value_bytes`.

##### options

[`KvPutOptions`](KvPutOptions.md)

Expiration and metadata.

#### Returns

`Promise`\<[`KvPutReply`](../type-aliases/KvPutReply.md)\>

[KvPutReply](../type-aliases/KvPutReply.md)
