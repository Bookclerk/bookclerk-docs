[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / removeWishlistItem

# Function: removeWishlistItem()

> **removeWishlistItem**(`uuid`): `Promise`\<[`TitleRequest`](../interfaces/TitleRequest.md)\>

Defined in: [api.ts:3806](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3806)

Deletes a wishlist title-request owned by the current user.

## Parameters

### uuid

`string`

Wishlist title-request uuid from [fetchWishlist](fetchWishlist.md).

## Returns

`Promise`\<[`TitleRequest`](../interfaces/TitleRequest.md)\>

The removed title-request row as echoed by the API.
