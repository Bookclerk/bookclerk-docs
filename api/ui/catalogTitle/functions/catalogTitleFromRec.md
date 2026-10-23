[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / catalogTitleFromRec

# Function: catalogTitleFromRec()

> **catalogTitleFromRec**(`rec`, `wishlistUuid?`): [`CatalogTitle`](../type-aliases/CatalogTitle.md)

Defined in: [catalogTitle.ts:582](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L582)

Maps a Discover recommendation into a [CatalogTitle](../type-aliases/CatalogTitle.md).

## Parameters

### rec

[`Recommendation`](../../api/interfaces/Recommendation.md)

Recommendation row.

### wishlistUuid?

`string` \| `null`

Optional wishlist uuid override.

## Returns

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Unified catalog title.
