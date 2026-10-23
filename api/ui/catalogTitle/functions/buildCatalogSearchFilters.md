[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / buildCatalogSearchFilters

# Function: buildCatalogSearchFilters()

> **buildCatalogSearchFilters**(`opts`): [`CatalogSearchFilters`](../../api/type-aliases/CatalogSearchFilters.md)

Defined in: [catalogTitle.ts:272](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L272)

Builds the server filter payload from UI multi-select state.

## Parameters

### opts

Include lists, exclusions, rating, and runtime bucket.

#### authors

`string`[]

#### excludeSources?

`string`[]

#### genres

`string`[]

#### hideVirtualVoice

`boolean`

#### languages?

`string`[]

#### minRating?

`number` \| `null`

#### narrators

`string`[]

#### runtimeBucket?

[`RuntimeBucket`](../type-aliases/RuntimeBucket.md)

#### series

`string`[]

#### sources?

`string`[]

**Deprecated**

prefer excludeSources

## Returns

[`CatalogSearchFilters`](../../api/type-aliases/CatalogSearchFilters.md)

`CatalogSearchFilters` for `searchCatalog`.
