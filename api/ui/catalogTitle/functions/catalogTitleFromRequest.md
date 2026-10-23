[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / catalogTitleFromRequest

# Function: catalogTitleFromRequest()

> **catalogTitleFromRequest**(`req`): [`CatalogTitle`](../type-aliases/CatalogTitle.md)

Defined in: [catalogTitle.ts:631](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L631)

Maps a wishlist title-request into a [CatalogTitle](../type-aliases/CatalogTitle.md).

## Parameters

### req

[`TitleRequest`](../../api/interfaces/TitleRequest.md)

Wishlist row.

## Returns

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Unified catalog title (with `wishlist_uuid` set).
