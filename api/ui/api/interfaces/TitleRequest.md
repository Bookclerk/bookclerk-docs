[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / TitleRequest

# Interface: TitleRequest

Defined in: [api.ts:3060](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3060)

Wishlist / title-request row from `/api/wishlist`.

Represents a user ask for a title (and optional acquire queue linkage via
`resolved_book_uuid` / `work_key`).

## Properties

### asin

> **asin**: `string` \| `null`

Defined in: [api.ts:3066](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3066)

***

### authors

> **authors**: `string` \| `null`

Defined in: [api.ts:3065](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3065)

***

### cover\_url?

> `optional` **cover\_url?**: `string` \| `null`

Defined in: [api.ts:3073](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3073)

***

### created\_at

> **created\_at**: `string`

Defined in: [api.ts:3086](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3086)

***

### description?

> `optional` **description?**: `string` \| `null`

Defined in: [api.ts:3074](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3074)

***

### genres?

> `optional` **genres?**: `string` \| `null`

Defined in: [api.ts:3082](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3082)

***

### id

> **id**: `number`

Defined in: [api.ts:3061](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3061)

***

### identity\_id

> **identity\_id**: `number` \| `null`

Defined in: [api.ts:3063](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3063)

***

### isbn

> **isbn**: `string` \| `null`

Defined in: [api.ts:3067](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3067)

***

### language?

> `optional` **language?**: `string` \| `null`

Defined in: [api.ts:3083](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3083)

***

### length\_minutes?

> `optional` **length\_minutes?**: `number` \| `null`

Defined in: [api.ts:3080](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3080)

***

### narrators?

> `optional` **narrators?**: `string` \| `null`

Defined in: [api.ts:3076](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3076)

***

### notes

> **notes**: `string` \| `null`

Defined in: [api.ts:3068](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3068)

***

### published\_at?

> `optional` **published\_at?**: `string` \| `null`

Defined in: [api.ts:3081](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3081)

***

### publisher?

> `optional` **publisher?**: `string` \| `null`

Defined in: [api.ts:3079](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3079)

***

### purchase\_hints?

> `optional` **purchase\_hints?**: [`PurchaseHint`](PurchaseHint.md)[]

Defined in: [api.ts:3085](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3085)

***

### resolved\_book\_uuid

> **resolved\_book\_uuid**: `string` \| `null`

Defined in: [api.ts:3072](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3072)

***

### series?

> `optional` **series?**: `string` \| `null`

Defined in: [api.ts:3077](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3077)

***

### series\_index?

> `optional` **series\_index?**: `string` \| `null`

Defined in: [api.ts:3078](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3078)

***

### status

> **status**: `string`

Defined in: [api.ts:3069](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3069)

***

### store\_editions?

> `optional` **store\_editions?**: [`StoreEdition`](StoreEdition.md)[]

Defined in: [api.ts:3084](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3084)

***

### subtitle?

> `optional` **subtitle?**: `string` \| `null`

Defined in: [api.ts:3075](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3075)

***

### title

> **title**: `string`

Defined in: [api.ts:3064](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3064)

***

### updated\_at

> **updated\_at**: `string`

Defined in: [api.ts:3087](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3087)

***

### uuid

> **uuid**: `string`

Defined in: [api.ts:3062](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3062)

***

### work\_id

> **work\_id**: `string` \| `null`

Defined in: [api.ts:3071](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3071)

***

### work\_key

> **work\_key**: `string`

Defined in: [api.ts:3070](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3070)
