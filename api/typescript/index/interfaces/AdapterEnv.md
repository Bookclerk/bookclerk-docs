[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / AdapterEnv

# Interface: AdapterEnv

Defined in: [plugin.ts:2205](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2205)

Trusted adapter env. Authors never see this type on their class.

## Properties

### BRIDGE\_TOKEN?

> `optional` **BRIDGE\_TOKEN?**: `string`

Defined in: [plugin.ts:2220](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2220)

Isolate-to-host bridge bearer.

***

### GRANTED?

> `optional` **GRANTED?**: [`GrantedFetcher`](GrantedFetcher.md)

Defined in: [plugin.ts:2218](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2218)

Per-invocation grant reverse channel.

***

### PLUGIN?

> `optional` **PLUGIN?**: [`AuthorStub`](../type-aliases/AuthorStub.md)

Defined in: [plugin.ts:2207](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2207)

Author default entrypoint.

***

### PLUGIN\_CLI?

> `optional` **PLUGIN\_CLI?**: [`NamedStub`](../type-aliases/NamedStub.md)

Defined in: [plugin.ts:2213](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2213)

***

### PLUGIN\_DATABASE\_ADAPTER?

> `optional` **PLUGIN\_DATABASE\_ADAPTER?**: [`NamedStub`](../type-aliases/NamedStub.md)

Defined in: [plugin.ts:2211](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2211)

***

### PLUGIN\_DESCRIBE?

> `optional` **PLUGIN\_DESCRIBE?**: `string` \| `PluginDescribe`

Defined in: [plugin.ts:2216](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2216)

Manifest `PluginDescribe` projection (JSON binding).

***

### PLUGIN\_OIDC?

> `optional` **PLUGIN\_OIDC?**: [`NamedStub`](../type-aliases/NamedStub.md)

Defined in: [plugin.ts:2214](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2214)

***

### PLUGIN\_REMOTE\_LIBRARY?

> `optional` **PLUGIN\_REMOTE\_LIBRARY?**: [`NamedStub`](../type-aliases/NamedStub.md)

Defined in: [plugin.ts:2212](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2212)

***

### PLUGIN\_STORAGE?

> `optional` **PLUGIN\_STORAGE?**: [`NamedStub`](../type-aliases/NamedStub.md)

Defined in: [plugin.ts:2210](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2210)

***

### PLUGIN\_STOREFRONT?

> `optional` **PLUGIN\_STOREFRONT?**: [`NamedStub`](../type-aliases/NamedStub.md)

Defined in: [plugin.ts:2209](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2209)

Named author entrypoints (`PLUGIN_STOREFRONT`, `PLUGIN_STORAGE`, …).
