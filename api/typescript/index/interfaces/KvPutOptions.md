[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvPutOptions

# Interface: KvPutOptions

Defined in: [generated.ts:2326](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2326)

Expiration and metadata for `KvNamespace.put`.

## Properties

### expirationTtlSeconds

> **expirationTtlSeconds**: `number`

Defined in: [generated.ts:2333](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2333)

Relative expiry in seconds from the write; zero means none. Ignored when
`expirationUnixMs` is set.

***

### expirationUnixMs

> **expirationUnixMs**: `number`

Defined in: [generated.ts:2328](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2328)

Absolute expiry (UTC Unix milliseconds); zero means none.

***

### metadata

> **metadata**: `string`

Defined in: [generated.ts:2335](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2335)

Opaque JSON metadata text, at most `maxKvMetadataBytes`; empty means none.
