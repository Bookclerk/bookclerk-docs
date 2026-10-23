[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / createWishlistItem

# Function: createWishlistItem()

> **createWishlistItem**(`body`): `Promise`\<[`TitleRequest`](../interfaces/TitleRequest.md)\>

Defined in: [api.ts:3770](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3770)

Adds a title to the current user's wishlist.

## Parameters

### body

Title fields and optional catalog enrichment.

#### asin?

`string`

#### authors?

`string`

#### cover_url?

`string` \| `null`

#### description?

`string` \| `null`

#### genres?

`string` \| `null`

#### isbn?

`string`

#### language?

`string` \| `null`

#### length_minutes?

`number` \| `null`

#### narrators?

`string` \| `null`

#### notes?

`string`

#### published_at?

`string` \| `null`

#### publisher?

`string` \| `null`

#### purchase_hints?

[`PurchaseHint`](../interfaces/PurchaseHint.md)[]

#### series?

`string` \| `null`

#### series_index?

`string` \| `null`

#### store_editions?

[`StoreEdition`](../interfaces/StoreEdition.md)[]

#### subtitle?

`string` \| `null`

#### title

`string`

#### work_key?

`string`

## Returns

`Promise`\<[`TitleRequest`](../interfaces/TitleRequest.md)\>

Created title-request row.
