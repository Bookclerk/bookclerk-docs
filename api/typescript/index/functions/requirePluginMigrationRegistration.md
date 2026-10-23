[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / requirePluginMigrationRegistration

# Function: requirePluginMigrationRegistration()

> **requirePluginMigrationRegistration**(`migrations`): [`PluginMigration`](../interfaces/PluginMigration.md)[]

Defined in: [plugin-migrations.ts:42](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin-migrations.ts#L42)

Rejects a plugin migration registration that exceeds ABI resource limits.

## Parameters

### migrations

[`PluginMigration`](../interfaces/PluginMigration.md)[]

Ordered `databaseMigrations` registration.

## Returns

[`PluginMigration`](../interfaces/PluginMigration.md)[]

The same array when within limits.

## Throws

Error with `code` / `wireCode` `payload_too_large` when a limit is exceeded.
