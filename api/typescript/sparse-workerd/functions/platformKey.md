[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / platformKey

# Function: platformKey()

> **platformKey**(`platform?`, `arch?`): `string` \| `undefined`

Defined in: [sparse-workerd/ensure.ts:69](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L69)

Maps Node `process.platform` / `process.arch` to a pin asset key.

## Parameters

### platform?

`Platform` = `process.platform`

Node platform string (default `process.platform`).

### arch?

`Architecture` = `process.arch`

Node arch string (default `process.arch`).

## Returns

`string` \| `undefined`

Pin asset key, or `undefined` when the host is unsupported.
