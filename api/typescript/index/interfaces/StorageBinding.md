[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / StorageBinding

# Interface: StorageBinding

Defined in: [env.ts:214](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L214)

`WORK_FS` binding: host-granted object storage for durable plugin files.
Same surface as the `storage` entrypoint the host calls on storage plugins.

## Methods

### delete()

> **delete**(`key`): `Promise`\<`void`\>

Defined in: [env.ts:262](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L262)

Remove an object; missing keys are not an error.

#### Parameters

##### key

`string`

Object key.

#### Returns

`Promise`\<`void`\>

Resolves when the delete is durable.

***

### get()

> **get**(`key`, `options?`): `Promise`\<\{ `body`: `ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\>; `meta`: \{ `key`: `string`; `size`: `number`; \}; \}\>

Defined in: [env.ts:239](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L239)

Stream an object (optionally a byte range).

#### Parameters

##### key

`string`

Object key.

##### options?

Optional byte range.

###### range?

\{ `length?`: `number`; `offset`: `number`; \}

###### range.length?

`number`

###### range.offset

`number`

#### Returns

`Promise`\<\{ `body`: `ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\>; `meta`: \{ `key`: `string`; `size`: `number`; \}; \}\>

Metadata and the body stream.

***

### head()

> **head**(`key`): `Promise`\<\{ `contentType?`: `string`; `etag?`: `string`; `key`: `string`; `size`: `number`; \} \| `null`\>

Defined in: [env.ts:221](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L221)

Object metadata without the body.

#### Parameters

##### key

`string`

Object key.

#### Returns

`Promise`\<\{ `contentType?`: `string`; `etag?`: `string`; `key`: `string`; `size`: `number`; \} \| `null`\>

Metadata, or `null` when the key does not exist.

***

### list()

> **list**(`options`): `Promise`\<\{ `nextCursor?`: `string`; `objects`: `object`[]; \}\>

Defined in: [env.ts:228](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L228)

One page of keys under `prefix`.

#### Parameters

##### options

Prefix, continuation cursor, and page size.

###### cursor?

`string`

###### limit?

`number`

###### prefix?

`string`

#### Returns

`Promise`\<\{ `nextCursor?`: `string`; `objects`: `object`[]; \}\>

Objects plus the cursor for the next page, when any.

***

### put()

> **put**(`key`, `body`, `options?`): `Promise`\<\{ `bytesWritten`: `number`; `key`: `string`; \}\>

Defined in: [env.ts:251](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L251)

Write an object from a stream.

#### Parameters

##### key

`string`

Object key.

##### body

`ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\>

Body bytes.

##### options?

Content type and length hints.

###### contentLength?

`number`

###### contentType?

`string`

#### Returns

`Promise`\<\{ `bytesWritten`: `number`; `key`: `string`; \}\>

Key and bytes written.
