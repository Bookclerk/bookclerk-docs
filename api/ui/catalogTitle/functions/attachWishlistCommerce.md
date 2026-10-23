[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / attachWishlistCommerce

# Function: attachWishlistCommerce()

> **attachWishlistCommerce**(`title`, `wishlist`): [`CatalogTitle`](../type-aliases/CatalogTitle.md)

Defined in: [catalogTitle.ts:765](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L765)

Unions snapshotted wishlist store editions / hints onto a Discover (or other)
catalog title so Where-to-buy matches the Wishlist detail for the same work.

## Parameters

### title

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Catalog title from search or shelves.

### wishlist

readonly [`TitleRequest`](../../api/interfaces/TitleRequest.md)[]

Open personal wishlist rows.

## Returns

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Title with wishlist commerce merged when a row matches.
