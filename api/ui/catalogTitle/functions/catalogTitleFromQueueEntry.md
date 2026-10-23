[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / catalogTitleFromQueueEntry

# Function: catalogTitleFromQueueEntry()

> **catalogTitleFromQueueEntry**(`entry`, `wishlistUuid?`): [`CatalogTitle`](../type-aliases/CatalogTitle.md)

Defined in: [catalogTitle.ts:1013](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L1013)

Maps a global request-queue entry into a [CatalogTitle](../type-aliases/CatalogTitle.md).

## Parameters

### entry

[`GlobalQueueEntry`](../../api/interfaces/GlobalQueueEntry.md)

Queue entry.

### wishlistUuid?

`string` \| `null`

Optional wishlist uuid when the current user also wished it.

## Returns

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Unified catalog title.
