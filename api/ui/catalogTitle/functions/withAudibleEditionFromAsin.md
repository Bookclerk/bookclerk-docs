[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / withAudibleEditionFromAsin

# Function: withAudibleEditionFromAsin()

> **withAudibleEditionFromAsin**\<`T`\>(`row`): `T`

Defined in: [catalogTitle.ts:955](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L955)

Adds an Audible store edition when an ASIN is known and no Audible row exists.

Used by Discover shelves, Wishlist enrichment, and the shared detail modal so
Where-to-buy does not depend on which surface fetched title-meta first.

## Type Parameters

### T

`T` *extends* `object`

## Parameters

### row

`T`

Title or detail fields that carry editions.

## Returns

`T`

The same object, or a copy with an Audible edition appended.
