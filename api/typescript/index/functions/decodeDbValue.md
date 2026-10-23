[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / decodeDbValue

# Function: decodeDbValue()

> **decodeDbValue**(`bytes`): [`DbValue`](../type-aliases/DbValue.md)

Defined in: [db-value.ts:130](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-value.ts#L130)

Decodes a standalone unpacked Cap'n `DbValue` message.

## Parameters

### bytes

`Uint8Array`

Unpacked Cap'n stream.

## Returns

[`DbValue`](../type-aliases/DbValue.md)

Domain value.

## Throws

When the buffer is not a valid `DbValue`.
