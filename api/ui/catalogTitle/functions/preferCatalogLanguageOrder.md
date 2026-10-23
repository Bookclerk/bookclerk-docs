[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / preferCatalogLanguageOrder

# Function: preferCatalogLanguageOrder()

> **preferCatalogLanguageOrder**\<`T`\>(`items`, `preferred?`): `T`[]

Defined in: [catalogTitle.ts:439](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L439)

Stable soft-sort: preferred language first, then unknown, then other.

## Type Parameters

### T

`T` *extends* `object`

## Parameters

### items

`T`[]

Rows with optional `language`.

### preferred?

`string` = `...`

Preferred language code.

## Returns

`T`[]

New array sorted by [catalogLanguageRank](catalogLanguageRank.md).
