[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchPluginInstanceConfig

# Function: fetchPluginInstanceConfig()

> **fetchPluginInstanceConfig**(`instanceId`): `Promise`\<[`PluginInstanceConfig`](../interfaces/PluginInstanceConfig.md)\>

Defined in: [api.ts:2033](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2033)

Reads a plugin instance's stored document.

## Parameters

### instanceId

`string`

Plugin instance id.

## Returns

`Promise`\<[`PluginInstanceConfig`](../interfaces/PluginInstanceConfig.md)\>

Stored settings, secret refs, and the document revision.
