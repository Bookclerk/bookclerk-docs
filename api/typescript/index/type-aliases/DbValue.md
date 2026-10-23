[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / DbValue

# Type Alias: DbValue

> **DbValue** = \{ `kind`: `"null"`; `value`: [`DbType`](DbType.md); \} \| \{ `kind`: `"boolean"`; `value`: `boolean`; \} \| \{ `kind`: `"int64"`; `value`: `bigint`; \} \| \{ `kind`: `"float64"`; `value`: `number`; \} \| \{ `kind`: `"text"`; `value`: `string`; \} \| \{ `kind`: `"bytes"`; `value`: `Uint8Array`; \}

Defined in: [db-value.ts:22](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-value.ts#L22)

Closed Cap'n `DbValue` union.

Members are typed null, bool, int64 (`bigint`), finite float64, UTF-8 text
(U+0000 forbidden), and bytes (`Uint8Array`, 0x00 allowed). Unknown `kind`
values fail closed.
