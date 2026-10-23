[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / validateSpawnExecutable

# Function: validateSpawnExecutable()

> **validateSpawnExecutable**(`bin`, `trustedRoot?`): `string`

Defined in: [sparse-workerd/ensure.ts:402](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L402)

Validates a workerd (or helper) binary path before spawn.

Requires an absolute path with no NUL bytes. When `trustedRoot` is set,
resolves both paths and requires the binary to stay under that root.

## Parameters

### bin

`string`

Candidate executable path.

### trustedRoot?

`string`

Optional directory the binary must remain under.

## Returns

`string`

Absolute validated path.

## Throws

When the path is relative, contains NUL, or escapes `trustedRoot`.
