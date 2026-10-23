[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / GetOk

# Interface: GetOk

Defined in: [generated.ts:593](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L593)

Success payload of `Destination.get`.

## Properties

### body?

> `optional` **body?**: [`ByteSource`](ByteSource.md) \| `null`

Defined in: [generated.ts:600](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L600)

Streamed object bytes.
Null when the guest does not export this capability.

***

### meta

> **meta**: `ObjectMetadata`

Defined in: [generated.ts:595](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L595)

Object metadata.
