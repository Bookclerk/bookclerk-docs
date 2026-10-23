[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PluginMigrationsOk

# Interface: PluginMigrationsOk

Defined in: [generated.ts:2925](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2925)

Success payload of `PluginWorker.databaseMigrations`.

## Properties

### migrations

> **migrations**: [`PluginMigration`](PluginMigration.md)[]

Defined in: [generated.ts:2931](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2931)

At most `maxListPage` entries; aggregate id+SQL bytes at most
`maxPluginMigrationRegistrationBytes`; total operations at most
`maxPluginMigrationTotalOps`.
