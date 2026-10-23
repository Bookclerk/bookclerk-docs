[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / deletePluginPlacement

# Function: deletePluginPlacement()

> **deletePluginPlacement**(`instanceId`): `Promise`\<\{ `deleted`: `boolean`; `plugin_instance_id`: `string`; \}\>

Defined in: [api.ts:2125](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2125)

Drops an instance's placement: its deployments stay as they are and the
Enabled switch manages them again.

## Parameters

### instanceId

`string`

Plugin instance id.

## Returns

`Promise`\<\{ `deleted`: `boolean`; `plugin_instance_id`: `string`; \}\>

Whether a placement was removed.
