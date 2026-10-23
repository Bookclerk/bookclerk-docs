[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / FetchTitleParams

# Interface: FetchTitleParams

Defined in: [generated.ts:1649](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1649)

Params of `ContentSource.fetchTitle`. The plugin writes media under
`cacheDir` and returns plain (DRM-free) paths. The host injects
credentials; guests must not open `library.db` or `master.key`.

## Properties

### accountId

> **accountId**: `string`

Defined in: [generated.ts:1653](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1653)

Account whose credentials apply.

***

### cacheDir

> **cacheDir**: `string`

Defined in: [generated.ts:1657](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1657)

Absolute path the guest should write media into (jail-granted TMPDIR).

***

### credentials?

> `optional` **credentials?**: `Uint8Array`\<`ArrayBufferLike`\>

Defined in: [generated.ts:1662](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1662)

Host-loaded credential blob for this account; empty when unavailable.
Omitted when absent (wire zero value).

***

### fetch

> **fetch**: [`FetchOptions`](FetchOptions.md)

Defined in: [generated.ts:1666](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1666)

Fetch-relevant acquire options.

***

### pluginDataDir

> **pluginDataDir**: `string`

Defined in: [generated.ts:1651](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1651)

Scoped plugin data directory.

***

### sourceConfig

> **sourceConfig**: [`ExtensibleConfig`](ExtensibleConfig.md)

Defined in: [generated.ts:1664](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1664)

Granted `[sources.<id>]` table as `application/json`.

***

### titleId

> **titleId**: `string`

Defined in: [generated.ts:1655](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1655)

Library / storefront title id to download.
