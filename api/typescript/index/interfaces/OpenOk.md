[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / OpenOk

# Interface: OpenOk

Defined in: [generated.ts:637](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L637)

Success payload of `Source.open`.

## Properties

### body?

> `optional` **body?**: [`ByteSource`](ByteSource.md) \| `null`

Defined in: [generated.ts:644](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L644)

Streamed object bytes.
Null when the guest does not export this capability.

***

### meta

> **meta**: `ObjectMetadata`

Defined in: [generated.ts:639](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L639)

Object metadata.
