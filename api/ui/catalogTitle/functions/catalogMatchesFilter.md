[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / catalogMatchesFilter

# Function: catalogMatchesFilter()

> **catalogMatchesFilter**(`title`, `kind`, `value`): `boolean`

Defined in: [catalogTitle.ts:1101](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L1101)

Returns whether a title matches a local catalog filter.

## Parameters

### title

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Catalog title.

### kind

[`CatalogFilterKind`](../type-aliases/CatalogFilterKind.md)

Filter category.

### value

`string`

Selected value (`all` / empty always matches).

## Returns

`boolean`

True when visible under the filter.
