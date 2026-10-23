[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / titleNeedsShelfMeta

# Function: titleNeedsShelfMeta()

> **titleNeedsShelfMeta**(`title`): `boolean`

Defined in: [catalogTitle.ts:1002](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L1002)

Returns whether a shelf card should pull title-meta (includes missing overall rating).

## Parameters

### title

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Catalog title.

## Returns

`boolean`

True when [titleNeedsMeta](titleNeedsMeta.md) or `rating_overall` is unknown.
