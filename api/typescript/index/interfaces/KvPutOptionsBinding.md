[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvPutOptionsBinding

# Interface: KvPutOptionsBinding

Defined in: [env.ts:85](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L85)

Options for [KvNamespaceBinding.put](KvNamespaceBinding.md#put) (Workers KV names).

## Properties

### expiration?

> `optional` **expiration?**: `number`

Defined in: [env.ts:87](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L87)

Absolute expiry in seconds since the Unix epoch.

***

### expirationTtl?

> `optional` **expirationTtl?**: `number`

Defined in: [env.ts:89](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L89)

Relative expiry in seconds from the write; ignored when `expiration` is set.

***

### metadata?

> `optional` **metadata?**: `unknown`

Defined in: [env.ts:91](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L91)

JSON-serializable metadata, at most `maxKvMetadataBytes` once encoded.
