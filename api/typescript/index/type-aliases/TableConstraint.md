[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / TableConstraint

# Type Alias: TableConstraint

> **TableConstraint** = \{ `kind`: `"primaryKey"`; `value`: `string`[]; \} \| \{ `kind`: `"unique"`; `value`: `string`[]; \} \| \{ `kind`: `"check"`; `value`: `string`; \} \| \{ `kind`: `"foreignKey"`; `value`: [`ForeignKeyConstraint`](../interfaces/ForeignKeyConstraint.md); \}

Defined in: [generated.ts:2539](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2539)

One table-level constraint.
