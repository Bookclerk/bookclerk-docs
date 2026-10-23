[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / TitleMeta

# Interface: TitleMeta

Defined in: [api.ts:3333](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3333)

Public Audnexus / Audible catalog fields for detail dialogs.

## Properties

### asin?

> `optional` **asin?**: `string` \| `null`

Defined in: [api.ts:3334](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3334)

***

### authors?

> `optional` **authors?**: `string` \| `null`

Defined in: [api.ts:3337](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3337)

***

### categories?

> `optional` **categories?**: `string` \| `null`

Defined in: [api.ts:3347](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3347)

***

### cover\_url?

> `optional` **cover\_url?**: `string` \| `null`

Defined in: [api.ts:3342](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3342)

***

### description?

> `optional` **description?**: `string` \| `null`

Defined in: [api.ts:3343](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3343)

***

### is\_abridged?

> `optional` **is\_abridged?**: `boolean` \| `null`

Defined in: [api.ts:3350](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3350)

`true` abridged / `false` unabridged when the storefront said so.

***

### isbn?

> `optional` **isbn?**: `string` \| `null`

Defined in: [api.ts:3341](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3341)

***

### language?

> `optional` **language?**: `string` \| `null`

Defined in: [api.ts:3348](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3348)

***

### length\_minutes?

> `optional` **length\_minutes?**: `number` \| `null`

Defined in: [api.ts:3345](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3345)

***

### narrators?

> `optional` **narrators?**: `string` \| `null`

Defined in: [api.ts:3338](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3338)

***

### published\_at?

> `optional` **published\_at?**: `string` \| `null`

Defined in: [api.ts:3346](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3346)

***

### publisher?

> `optional` **publisher?**: `string` \| `null`

Defined in: [api.ts:3344](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3344)

***

### rating\_count?

> `optional` **rating\_count?**: `number` \| `null`

Defined in: [api.ts:3355](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3355)

***

### rating\_overall?

> `optional` **rating\_overall?**: `number` \| `null`

Defined in: [api.ts:3352](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3352)

Audible community ratings (detail fetch only).

***

### rating\_performance?

> `optional` **rating\_performance?**: `number` \| `null`

Defined in: [api.ts:3353](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3353)

***

### rating\_story?

> `optional` **rating\_story?**: `number` \| `null`

Defined in: [api.ts:3354](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3354)

***

### review\_count?

> `optional` **review\_count?**: `number` \| `null`

Defined in: [api.ts:3356](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3356)

***

### ~~reviews?~~

> `optional` **reviews?**: [`TitleReview`](TitleReview.md)[]

Defined in: [api.ts:3358](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3358)

#### Deprecated

Prefer [`fetchTitleReviews`]. Title-meta no longer embeds reviews.

***

### series?

> `optional` **series?**: `string` \| `null`

Defined in: [api.ts:3339](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3339)

***

### series\_index?

> `optional` **series\_index?**: `string` \| `null`

Defined in: [api.ts:3340](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3340)

***

### subtitle?

> `optional` **subtitle?**: `string` \| `null`

Defined in: [api.ts:3336](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3336)

***

### title?

> `optional` **title?**: `string` \| `null`

Defined in: [api.ts:3335](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3335)
