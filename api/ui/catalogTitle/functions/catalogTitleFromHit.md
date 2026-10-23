[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / catalogTitleFromHit

# Function: catalogTitleFromHit()

> **catalogTitleFromHit**(`hit`, `wishlistUuid?`): [`CatalogTitle`](../type-aliases/CatalogTitle.md)

Defined in: [catalogTitle.ts:545](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L545)

Maps a catalog search hit into a [CatalogTitle](../type-aliases/CatalogTitle.md).

## Parameters

### hit

[`CatalogSearchHit`](../../api/interfaces/CatalogSearchHit.md)

Search hit.

### wishlistUuid?

`string` \| `null`

Optional wishlist uuid when already wished.

## Returns

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Unified catalog title.
