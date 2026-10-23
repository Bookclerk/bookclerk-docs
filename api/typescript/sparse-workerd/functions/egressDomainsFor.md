[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / egressDomainsFor

# Function: egressDomainsFor()

> **egressDomainsFor**(`needsPython`, `mode`, `base`): `string`[]

Defined in: [sparse-workerd/config.ts:342](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L342)

Builds the egress allowlist, appending Pyodide hosts when needed.

## Parameters

### needsPython

`boolean`

Whether the plugin embeds Python modules.

### mode

`string`

Manifest network mode (`outbound` enables Pyodide hosts).

### base

`string`[]

Domains declared in `capabilities.network.domains`.

## Returns

`string`[]

Deduplicated domain list for the egress policy JSON.
