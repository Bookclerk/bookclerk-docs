[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / SearchCatalogParams

# Interface: SearchCatalogParams

Defined in: [generated.ts:1736](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1736)

Params of `ContentSource.searchCatalog`.

## Properties

### field

> **field**: `"author"` \| `"any"` \| `"narrator"` \| `"series"` \| `"genre"`

Defined in: [generated.ts:1748](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1748)

Facet restriction.

***

### language?

> `optional` **language?**: `string`

Defined in: [generated.ts:1753](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1753)

Preferred content language (soft-prioritize; e.g. `en`).
Omitted when absent (wire zero value).

***

### limit

> **limit**: `number`

Defined in: [generated.ts:1742](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1742)

Maximum hits to return; the host always sends an explicit value.

***

### page

> **page**: `number`

Defined in: [generated.ts:1744](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1744)

1-based page for storefronts that page.

***

### query

> **query**: `string`

Defined in: [generated.ts:1738](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1738)

Free-text search query.

***

### region

> **region**: `string`

Defined in: [generated.ts:1740](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1740)

Storefront region / marketplace code; empty means the guest default.

***

### sort

> **sort**: `"relevance"` \| `"popularity"` \| `"rating"` \| `"title"` \| `"author"`

Defined in: [generated.ts:1746](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1746)

Sort order.
