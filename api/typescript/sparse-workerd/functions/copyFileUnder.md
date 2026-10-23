[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / copyFileUnder

# Function: copyFileUnder()

> **copyFileUnder**(`root`, `name`, `src`): `string`

Defined in: [sparse-workerd/ensure.ts:220](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L220)

Copy `src` to `root` / `name` after resolve + barrier on the destination.

## Parameters

### root

`string`

Trusted destination directory (must already exist).

### name

`string`

Single path component filename.

### src

`string`

Absolute source file (already validated by the caller).

## Returns

`string`

Absolute destination path under `root`.
