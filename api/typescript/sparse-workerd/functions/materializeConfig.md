[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / materializeConfig

# Function: materializeConfig()

> **materializeConfig**(`pluginRoot`, `manifest`, `options`): [`GeneratedConfig`](../type-aliases/GeneratedConfig.md)

Defined in: [sparse-workerd/config.ts:373](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L373)

Materializes bridge assets + Cap'n Proto under a host session directory.

Copies bridge scripts into `stateDir/.bookclerk/`, embeds plugin modules via
Cap'n Proto `/modules/…` + `--import-path` (read-only install root), and
writes the Cap'n Proto config beside those embeds — never under the tainted
plugin install path.

## Parameters

### pluginRoot

`string`

Plugin directory containing `plugin.toml` and modules.

### manifest

`Manifest`

Validated workerd manifest.

### options

[`MaterializeOptions`](../type-aliases/MaterializeOptions.md)

Listen port and bridge token.

## Returns

[`GeneratedConfig`](../type-aliases/GeneratedConfig.md)

Generated config path, listen address, session dir, and import path.

## Throws

When `[workerd]` is missing, modules are absent, or the
  bridge token is empty.
