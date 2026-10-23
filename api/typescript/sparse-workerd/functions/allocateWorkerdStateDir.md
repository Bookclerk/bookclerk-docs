[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / allocateWorkerdStateDir

# Function: allocateWorkerdStateDir()

> **allocateWorkerdStateDir**(`pluginRoot`): `string`

Defined in: [sparse-workerd/config.ts:242](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L242)

Allocate a unique writable session directory for workerd generated embeds.

Keys the leaf by a short hash of the plugin root plus a random nonce so
concurrent sessions cannot clobber Cap'n Proto. Prefer `$TMPDIR` / OS temp;
fall back to `.bookclerk-state` beside the plugin only when temp is unset.

## Parameters

### pluginRoot

`string`

Plugin install root (used only as an opaque id seed).

## Returns

`string`

Canonical absolute session directory.
