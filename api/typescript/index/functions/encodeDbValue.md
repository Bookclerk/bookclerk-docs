[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / encodeDbValue

# Function: encodeDbValue()

> **encodeDbValue**(`value`): `Uint8Array`

Defined in: [db-value.ts:116](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-value.ts#L116)

Encodes a standalone unpacked Cap'n `DbValue` message.

## Parameters

### value

[`DbValue`](../type-aliases/DbValue.md)

Domain value (`bigint` / `Uint8Array`, not JSON number / `b64:`).

## Returns

`Uint8Array`

Unpacked Cap'n stream bytes (same encoding as the Rust SDK).

## Throws

When `int64` is out of range, `float64` is not finite, or TEXT
contains U+0000.
