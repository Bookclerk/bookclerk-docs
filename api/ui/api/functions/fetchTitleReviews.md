[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchTitleReviews

# Function: fetchTitleReviews()

> **fetchTitleReviews**(`body`): `Promise`\<[`TitleReviewsPage`](../type-aliases/TitleReviewsPage.md)\>

Defined in: [api.ts:3568](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3568)

Loads a page of Audible customer reviews for title detail infinite scroll.

## Parameters

### body

[`TitleReviewsQuery`](../type-aliases/TitleReviewsQuery.md)

ASIN, region, page, and sort.

## Returns

`Promise`\<[`TitleReviewsPage`](../type-aliases/TitleReviewsPage.md)\>

Reviews page payload.
