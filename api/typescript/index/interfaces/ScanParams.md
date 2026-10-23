[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ScanParams

# Interface: ScanParams

Defined in: [generated.ts:1499](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1499)

Params of `ContentSource.scan`. The host injects sealed credentials so the
plugin does not need a private credential store under `pluginDataDir`.

## Properties

### accounts

> **accounts**: `string`[]

Defined in: [generated.ts:1503](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1503)

Account ids to scan; empty means all scan-enabled accounts.

***

### credentials

> **credentials**: [`AccountCredential`](AccountCredential.md)[]

Defined in: [generated.ts:1511](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1511)

Host-loaded credential blobs for the requested accounts.

***

### importEpisodes

> **importEpisodes**: `boolean`

Defined in: [generated.ts:1507](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1507)

When true, import podcast/episode-style rows.

***

### importPlusTitles

> **importPlusTitles**: `boolean`

Defined in: [generated.ts:1509](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1509)

When true, import Plus/catalog entitlement titles.

***

### pageSize

> **pageSize**: `number`

Defined in: [generated.ts:1505](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1505)

Storefront page size; the host always sends an explicit value.

***

### pluginDataDir

> **pluginDataDir**: `string`

Defined in: [generated.ts:1501](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1501)

Scoped plugin data directory.
