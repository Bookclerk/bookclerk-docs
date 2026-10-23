[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / allowedEntrypointFamilies

# Function: allowedEntrypointFamilies()

> **allowedEntrypointFamilies**(`capabilities`, `requested`): `string`[]

Defined in: [plugin.ts:2930](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2930)

Filters requested entrypoint / trigger family names against the manifest
capabilities: named entrypoints must be declared in `entrypoints`,
`eventConsumer` needs a non-empty `consumes`, `queueConsumer` needs a
non-empty `queueConsumers`, and `jobRunner` needs `jobs` or the `storage`
entrypoint (storage guests run the host `stream_copy` job). Unknown names
and a missing manifest fail closed.

## Parameters

### capabilities

`Partial`\<[`PluginCapabilities`](../interfaces/PluginCapabilities.md)\> \| `null` \| `undefined`

Manifest capabilities from `PLUGIN_DESCRIBE`.

### requested

readonly `string`[]

Family names the launcher wants to export.

## Returns

`string`[]

The authorized subset in request order.
