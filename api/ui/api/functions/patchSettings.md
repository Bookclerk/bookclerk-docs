[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / patchSettings

# Function: patchSettings()

> **patchSettings**(`body`): `Promise`\<[`SettingsResponse`](../interfaces/SettingsResponse.md)\>

Defined in: [api.ts:2532](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2532)

Writes bootstrap settings (`daemon`, `jail`, `[database]` keys) through
`PATCH /api/settings`. Plugin settings are not accepted here: they are
instance documents, written with [putPluginInstanceConfig](putPluginInstanceConfig.md).

## Parameters

### body

Settings updates to apply.

#### expected_output_revision?

`number`

`core.output` revision the caller read; a stale value is rejected with 409.

#### settings

[`SettingsUpdate`](../interfaces/SettingsUpdate.md)[]

## Returns

`Promise`\<[`SettingsResponse`](../interfaces/SettingsResponse.md)\>

The settings view after the write.
