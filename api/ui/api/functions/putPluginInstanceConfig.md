[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / putPluginInstanceConfig

# Function: putPluginInstanceConfig()

> **putPluginInstanceConfig**(`instanceId`, `body`): `Promise`\<\{ `plugin_instance_id`: `string`; `revision`: `number`; \}\>

Defined in: [api.ts:2051](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2051)

Replaces a plugin instance's document (compare-and-swap on `expected_revision`).

Secrets in `body.secrets` are sealed server-side and referenced by name;
the response never carries them.

## Parameters

### instanceId

`string`

Plugin instance id.

### body

[`PluginInstanceConfigWrite`](../interfaces/PluginInstanceConfigWrite.md)

New document plus secrets to seal.

## Returns

`Promise`\<\{ `plugin_instance_id`: `string`; `revision`: `number`; \}\>

The committed revision.
