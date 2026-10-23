[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / writeFileUnder

# Function: writeFileUnder()

> **writeFileUnder**(`root`, `name`, `contents`): `string`

Defined in: [sparse-workerd/ensure.ts:192](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L192)

Write `contents` to `root` / `name` (single component) after resolve + barrier.

Assumes `root` already exists (session/cache dirs are created first). No
mkdir of the tainted root — only write the barriered child path.

## Parameters

### root

`string`

Trusted directory (must already exist).

### name

`string`

Single path component filename.

### contents

`string` \| `ArrayBufferView`\<`ArrayBufferLike`\>

Bytes or string to write.

## Returns

`string`

Absolute file path under `root`.
