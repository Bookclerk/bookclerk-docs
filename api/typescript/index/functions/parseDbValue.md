[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / parseDbValue

# Function: parseDbValue()

> **parseDbValue**(`raw`): [`DbValue`](../type-aliases/DbValue.md)

Defined in: [db-value.ts:69](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-value.ts#L69)

Parses a JSON `DbValue`. Unknown union members throw.

`int64` accepts a `bigint` or a finite integer `number` (converted to
`bigint`). `bytes` accepts a `Uint8Array` or a `b64:` JSON string.

## Parameters

### raw

`unknown`

Parsed JSON object.

## Returns

[`DbValue`](../type-aliases/DbValue.md)

Typed value.

## Throws

When `raw` is not a closed `DbValue`.
