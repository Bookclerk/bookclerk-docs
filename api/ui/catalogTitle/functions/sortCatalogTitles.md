[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / sortCatalogTitles

# Function: sortCatalogTitles()

> **sortCatalogTitles**(`titles`, `sortKey`): [`CatalogTitle`](../type-aliases/CatalogTitle.md)[]

Defined in: [catalogTitle.ts:1139](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L1139)

Client-sorts catalog titles for keys the server does not re-rank locally.

Relevance / popularity / rating keep input order (server ranking).

## Parameters

### titles

[`CatalogTitle`](../type-aliases/CatalogTitle.md)[]

Catalog titles.

### sortKey

[`CatalogSearchSort`](../../api/type-aliases/CatalogSearchSort.md)

Sort key.

## Returns

[`CatalogTitle`](../type-aliases/CatalogTitle.md)[]

Sorted copy or original order for server-ranked keys.
