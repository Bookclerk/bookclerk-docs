[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / PluginConsentResponse

# Interface: PluginConsentResponse

Defined in: [api.ts:1829](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1829)

Network/capability consent status for a plugin.

## Properties

### brand?

> `optional` **brand?**: [`PluginConsentBrand`](PluginConsentBrand.md)

Defined in: [api.ts:1839](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1839)

***

### covered

> **covered**: `boolean`

Defined in: [api.ts:1836](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1836)

***

### existing?

> `optional` **existing?**: [`PluginGrant`](PluginGrant.md)

Defined in: [api.ts:1838](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1838)

***

### limits

> **limits**: [`PluginConsentLimits`](PluginConsentLimits.md)

Defined in: [api.ts:1840](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1840)

***

### plugin\_id

> **plugin\_id**: `string`

Defined in: [api.ts:1830](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1830)

***

### plugin\_key?

> `optional` **plugin\_key?**: `string`

Defined in: [api.ts:1831](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1831)

***

### provenance?

> `optional` **provenance?**: `string`

Defined in: [api.ts:1832](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1832)

***

### request

> **request**: [`PluginGrant`](PluginGrant.md)

Defined in: [api.ts:1835](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1835)

***

### runtime

> **runtime**: `string`

Defined in: [api.ts:1834](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1834)

Guest runtime: `native` or `workerd`.

***

### summary

> **summary**: `string`[]

Defined in: [api.ts:1837](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1837)
