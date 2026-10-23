[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / attachWishersFromQueue

# Function: attachWishersFromQueue()

> **attachWishersFromQueue**(`title`, `queue`): [`CatalogTitle`](../type-aliases/CatalogTitle.md)

Defined in: [catalogTitle.ts:673](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L673)

Copies household queue identity onto a personal wishlist title.

Personal `TitleRequest` rows do not include household wishers or the merged
store editions from the global queue. Discover shelves use that merged row,
so Wishlist must union the same editions or Where-to-buy diverges.

## Parameters

### title

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Catalog title, typically from [catalogTitleFromRequest](catalogTitleFromRequest.md).

### queue

readonly [`GlobalQueueEntry`](../../api/interfaces/GlobalQueueEntry.md)[]

Global request-queue entries.

## Returns

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Title with wishers, identifiers, and store editions from the queue.
