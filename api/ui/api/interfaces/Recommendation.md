[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / Recommendation

# Interface: Recommendation

Defined in: [api.ts:2978](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2978)

Discover shelf recommendation item.

## Properties

### asin

> **asin**: `string` \| `null`

Defined in: [api.ts:2985](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2985)

***

### authors

> **authors**: `string` \| `null`

Defined in: [api.ts:2981](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2981)

***

### candidate\_product\_id

> **candidate\_product\_id**: `string` \| `null`

Defined in: [api.ts:2995](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2995)

***

### candidate\_source

> **candidate\_source**: `string` \| `null`

Defined in: [api.ts:2994](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2994)

***

### categories?

> `optional` **categories?**: `string`[]

Defined in: [api.ts:2990](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2990)

Stable shelf-kind tags (`finish_series`, `author`, `requests`, …).

***

### cover\_url?

> `optional` **cover\_url?**: `string` \| `null`

Defined in: [api.ts:3000](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3000)

***

### description?

> `optional` **description?**: `string` \| `null`

Defined in: [api.ts:3002](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3002)

***

### from\_request

> **from\_request**: `boolean`

Defined in: [api.ts:2992](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2992)

***

### genres?

> `optional` **genres?**: `string` \| `null`

Defined in: [api.ts:3006](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3006)

***

### isbn

> **isbn**: `string` \| `null`

Defined in: [api.ts:2986](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2986)

***

### language?

> `optional` **language?**: `string` \| `null`

Defined in: [api.ts:3007](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3007)

***

### length\_minutes?

> `optional` **length\_minutes?**: `number` \| `null`

Defined in: [api.ts:3004](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3004)

***

### narrators

> **narrators**: `string` \| `null`

Defined in: [api.ts:2982](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2982)

***

### published\_at?

> `optional` **published\_at?**: `string` \| `null`

Defined in: [api.ts:3005](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3005)

***

### publisher?

> `optional` **publisher?**: `string` \| `null`

Defined in: [api.ts:3003](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3003)

***

### purchase\_hints

> **purchase\_hints**: [`PurchaseHint`](PurchaseHint.md)[]

Defined in: [api.ts:2991](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2991)

***

### reasons

> **reasons**: `string`[]

Defined in: [api.ts:2988](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2988)

***

### request\_uuid

> **request\_uuid**: `string` \| `null`

Defined in: [api.ts:2993](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2993)

***

### score

> **score**: `number`

Defined in: [api.ts:2987](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2987)

***

### seed\_categories?

> `optional` **seed\_categories?**: `string` \| `null`

Defined in: [api.ts:2997](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2997)

***

### series

> **series**: `string` \| `null`

Defined in: [api.ts:2983](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2983)

***

### series\_index

> **series\_index**: `string` \| `null`

Defined in: [api.ts:2984](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2984)

***

### store\_editions?

> `optional` **store\_editions?**: [`StoreEdition`](StoreEdition.md)[]

Defined in: [api.ts:2996](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2996)

***

### subtitle?

> `optional` **subtitle?**: `string` \| `null`

Defined in: [api.ts:3001](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3001)

***

### title

> **title**: `string`

Defined in: [api.ts:2980](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2980)

***

### wish\_count?

> `optional` **wish\_count?**: `number`

Defined in: [api.ts:3011](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3011)

Distinct wisher count (may exceed `wishers.length`).

***

### wishers?

> `optional` **wishers?**: [`QueueWisher`](QueueWisher.md)[]

Defined in: [api.ts:3009](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3009)

People who have this title on an open wishlist.

***

### work\_id

> **work\_id**: `string` \| `null`

Defined in: [api.ts:2979](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2979)

***

### work\_key?

> `optional` **work\_key?**: `string`

Defined in: [api.ts:2999](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2999)

Stable bibliographic key (`isbn:` / `asin:` / `soft:`…).
