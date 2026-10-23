[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / ensureDirUnder

# Function: ensureDirUnder()

> **ensureDirUnder**(`root`, `rel`): `string`

Defined in: [sparse-workerd/ensure.ts:163](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L163)

Create `root` / `rel` as a directory after resolve + relative/`startsWith` barrier.

## Parameters

### root

`string`

Trusted directory.

### rel

`string`

Relative suffix (may be multi-segment when each segment is safe).

## Returns

`string`

Absolute directory path under `root`.
