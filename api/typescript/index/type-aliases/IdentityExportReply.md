[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / IdentityExportReply

# Type Alias: IdentityExportReply

> **IdentityExportReply** = \{ `kind`: `"ok"`; `value`: [`IdentityHighWater`](../interfaces/IdentityHighWater.md)[]; \} \| \{ `kind`: `"err"`; `value`: `PluginError`; \}

Defined in: [generated.ts:2804](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2804)

Result union of `AdapterDatabaseSession.exportIdentity`.
