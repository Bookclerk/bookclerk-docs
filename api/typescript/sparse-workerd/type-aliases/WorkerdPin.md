[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / WorkerdPin

# Type Alias: WorkerdPin

> **WorkerdPin** = `object`

Defined in: [sparse-workerd/ensure.ts:26](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L26)

Contents of `workerd-pin.json` shipping with this package.

## Properties

### assets

> **assets**: `Record`\<`string`, [`WorkerdAsset`](WorkerdAsset.md)\>

Defined in: [sparse-workerd/ensure.ts:34](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L34)

Platform key → artifact map (`linux-x86_64`, `macos-aarch64`, `windows-x86_64`, …).

***

### bundled\_compat\_date

> **bundled\_compat\_date**: `string`

Defined in: [sparse-workerd/ensure.ts:30](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L30)

Compatibility date bundled with this pin for Cap'n Proto configs.

***

### release\_tag

> **release\_tag**: `string`

Defined in: [sparse-workerd/ensure.ts:28](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L28)

GitHub release tag (for example `v1.20250310.0`) that identifies this pin.

***

### version\_stamp

> **version\_stamp**: `string`

Defined in: [sparse-workerd/ensure.ts:32](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L32)

Stamp filename written beside the installed binary to record `release_tag`.
