[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / GlobalQueueEntry

# Interface: GlobalQueueEntry

Defined in: [api.ts:3117](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3117)

Aggregated global request-queue entry (shared wishlist demand).

## Properties

### asin

> **asin**: `string` \| `null`

Defined in: [api.ts:3121](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3121)

***

### authors

> **authors**: `string` \| `null`

Defined in: [api.ts:3120](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3120)

***

### cover\_url?

> `optional` **cover\_url?**: `string` \| `null`

Defined in: [api.ts:3123](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3123)

***

### description?

> `optional` **description?**: `string` \| `null`

Defined in: [api.ts:3124](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3124)

***

### first\_requested\_at

> **first\_requested\_at**: `string`

Defined in: [api.ts:3140](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3140)

***

### genres?

> `optional` **genres?**: `string` \| `null`

Defined in: [api.ts:3132](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3132)

***

### isbn

> **isbn**: `string` \| `null`

Defined in: [api.ts:3122](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3122)

***

### language?

> `optional` **language?**: `string` \| `null`

Defined in: [api.ts:3133](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3133)

***

### last\_requested\_at

> **last\_requested\_at**: `string`

Defined in: [api.ts:3141](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3141)

***

### length\_minutes?

> `optional` **length\_minutes?**: `number` \| `null`

Defined in: [api.ts:3130](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3130)

***

### narrators?

> `optional` **narrators?**: `string` \| `null`

Defined in: [api.ts:3126](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3126)

***

### published\_at?

> `optional` **published\_at?**: `string` \| `null`

Defined in: [api.ts:3131](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3131)

***

### publisher?

> `optional` **publisher?**: `string` \| `null`

Defined in: [api.ts:3129](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3129)

***

### purchase\_hints?

> `optional` **purchase\_hints?**: [`PurchaseHint`](PurchaseHint.md)[]

Defined in: [api.ts:3135](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3135)

***

### reasons?

> `optional` **reasons?**: `string`[]

Defined in: [api.ts:3145](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3145)

***

### sample\_uuids

> **sample\_uuids**: `string`[]

Defined in: [api.ts:3139](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3139)

***

### score?

> `optional` **score?**: `number`

Defined in: [api.ts:3143](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3143)

Final rank (`taste_score + wish_count` boost).

***

### series?

> `optional` **series?**: `string` \| `null`

Defined in: [api.ts:3127](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3127)

***

### series\_index?

> `optional` **series\_index?**: `string` \| `null`

Defined in: [api.ts:3128](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3128)

***

### store\_editions?

> `optional` **store\_editions?**: [`StoreEdition`](StoreEdition.md)[]

Defined in: [api.ts:3134](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3134)

***

### subtitle?

> `optional` **subtitle?**: `string` \| `null`

Defined in: [api.ts:3125](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3125)

***

### taste\_score?

> `optional` **taste\_score?**: `number`

Defined in: [api.ts:3144](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3144)

***

### title

> **title**: `string`

Defined in: [api.ts:3119](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3119)

***

### wish\_count

> **wish\_count**: `number`

Defined in: [api.ts:3136](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3136)

***

### wishers?

> `optional` **wishers?**: [`QueueWisher`](QueueWisher.md)[]

Defined in: [api.ts:3138](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3138)

Distinct people who have this title on an open wishlist.

***

### work\_key

> **work\_key**: `string`

Defined in: [api.ts:3118](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3118)
