[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PluginMigrationOp

# Type Alias: PluginMigrationOp

> **PluginMigrationOp** = \{ `kind`: `"schema"`; `value`: `string`; \} \| \{ `kind`: `"data"`; `value`: `string`; \}

Defined in: [generated.ts:2907](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2907)

One already-separated BookclerkSQL operation in a plugin-owned migration.
`schema` is admitted DDL; `data` is admitted DML. There is no native SQL
escape hatch.
