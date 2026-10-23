[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / refuseSymlinkPath

# Function: refuseSymlinkPath()

> **refuseSymlinkPath**(`trustedRoot`, `candidate`): `string`

Defined in: [sparse-workerd/ensure.ts:310](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L310)

Require `candidate` under `trustedRoot` with no symlink suffix components.

Resolves the trusted root identity once (a symlinked operator/plugin root is
allowed). Only path components *below* that identity are refused when they
are symlinks.

## Parameters

### trustedRoot

`string`

Original operator/plugin root.

### candidate

`string`

Path previously produced by [assertPathInside](assertPathInside.md).

## Returns

`string`

The validated absolute path.

## Throws

When a suffix component is a symlink or escapes `trustedRoot`.
