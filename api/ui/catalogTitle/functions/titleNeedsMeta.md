[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / titleNeedsMeta

# Function: titleNeedsMeta()

> **titleNeedsMeta**(`title`): `boolean`

Defined in: [catalogTitle.ts:981](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L981)

Returns whether a title still needs public metadata enrichment.

## Parameters

### title

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Catalog title.

## Returns

`boolean`

True when core fields (description, length, publisher, narrators, series index) are incomplete.
