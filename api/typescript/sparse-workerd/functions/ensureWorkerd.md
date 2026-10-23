[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / ensureWorkerd

# Function: ensureWorkerd()

> **ensureWorkerd**(`cacheDir?`, `root?`): `Promise`\<`string`\>

Defined in: [sparse-workerd/ensure.ts:504](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L504)

Ensures `cacheDir/workerd` matches the pin, downloading if needed.

Honors `BOOKCLERK_WORKERD_BIN` when that absolute binary exists and matches
the pin (stamp or `--version`). The selected cache directory is the trusted
install root; children stay under it.

## Parameters

### cacheDir?

`string` = `...`

Directory that will hold the binary (default [defaultCacheDir](defaultCacheDir.md)).

### root?

`string` = `...`

Package root for loading the pin (default [packageRoot](packageRoot.md)).

## Returns

`Promise`\<`string`\>

Absolute path to a current `workerd` binary.

## Throws

When the platform has no pin, download fails, or the digest mismatches.
