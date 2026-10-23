[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / DeliveredCheckpoint

# Interface: DeliveredCheckpoint

Defined in: [plugin.ts:751](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L751)

A prior checkpoint delivered on resume.

## Properties

### json

> **json**: `string`

Defined in: [plugin.ts:753](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L753)

Checkpoint JSON text as recorded by the earlier `suspend()`.

***

### schemaVersion

> **schemaVersion**: `number`

Defined in: [plugin.ts:755](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L755)

Schema version the author attached to the checkpoint.
