[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / WorkerdAsset

# Type Alias: WorkerdAsset

> **WorkerdAsset** = `object`

Defined in: [sparse-workerd/ensure.ts:16](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L16)

One platform asset entry inside [WorkerdPin](WorkerdPin.md).

## Properties

### artifact

> **artifact**: `string`

Defined in: [sparse-workerd/ensure.ts:18](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L18)

Gzip-compressed release artifact filename on the Cloudflare workerd GitHub release.

***

### sha256\_hex

> **sha256\_hex**: `string`

Defined in: [sparse-workerd/ensure.ts:20](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L20)

Expected SHA-256 hex digest of the compressed artifact bytes (not the gunzipped binary).
