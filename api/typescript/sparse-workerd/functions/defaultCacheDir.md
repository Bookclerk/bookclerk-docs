[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / defaultCacheDir

# Function: defaultCacheDir()

> **defaultCacheDir**(): `string`

Defined in: [sparse-workerd/ensure.ts:112](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L112)

Resolve the workerd cache directory.

Honors `BOOKCLERK_WORKERD_CACHE` as the selected install root (workspace
`target/`, `/opt`, external volumes). Otherwise uses
`~/.cache/bookclerk/workerd`. Derived files stay under that root.

## Returns

`string`

Absolute cache directory path.
