[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / pluginGlobalOutbound

# Function: pluginGlobalOutbound()

> **pluginGlobalOutbound**(`mode`): `"blocked"` \| `"egress"`

Defined in: [sparse-workerd/config.ts:330](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L330)

Maps a manifest network mode to the plugin worker's `globalOutbound`.

## Parameters

### mode

`string`

Manifest `capabilities.network.mode` (`outbound` or deny-like).

## Returns

`"blocked"` \| `"egress"`

Cap'n Proto outbound service name (`egress` or `blocked`).
