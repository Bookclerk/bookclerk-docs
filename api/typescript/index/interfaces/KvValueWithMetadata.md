[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvValueWithMetadata

# Interface: KvValueWithMetadata\<T\>

Defined in: [env.ts:125](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L125)

Value plus metadata from [KvNamespaceBinding.getWithMetadata](KvNamespaceBinding.md#getwithmetadata).

## Type Parameters

### T

`T`

## Properties

### metadata

> **metadata**: `unknown`

Defined in: [env.ts:129](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L129)

Stored metadata, or `null` when none.

***

### value

> **value**: `T` \| `null`

Defined in: [env.ts:127](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L127)

Decoded value, or `null` when the key is absent or expired.
