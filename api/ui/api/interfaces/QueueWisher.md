[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / QueueWisher

# Interface: QueueWisher

Defined in: [api.ts:3093](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3093)

Person who has a title on an open wishlist (global queue avatars).

## Properties

### avatar\_source?

> `optional` **avatar\_source?**: `string` \| `null`

Defined in: [api.ts:3107](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3107)

Explicit picture choice (`auto` / `monogram` / `gravatar` / `upload` / `sso:{id}`).

***

### display\_name?

> `optional` **display\_name?**: `string` \| `null`

Defined in: [api.ts:3099](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3099)

Display name, identity label, or Operator.

***

### gravatar\_hash?

> `optional` **gravatar\_hash?**: `string` \| `null`

Defined in: [api.ts:3109](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3109)

SHA-256 hex of the contact email for Gravatar.

***

### has\_avatar?

> `optional` **has\_avatar?**: `boolean`

Defined in: [api.ts:3105](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3105)

True when an uploaded avatar file exists.

***

### identity\_id?

> `optional` **identity\_id?**: `number` \| `null`

Defined in: [api.ts:3097](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3097)

Portal identity that created the wish.

***

### login\_name?

> `optional` **login\_name?**: `string` \| `null`

Defined in: [api.ts:3101](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3101)

Local login name when set.

***

### operator?

> `optional` **operator?**: `boolean`

Defined in: [api.ts:3103](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3103)

True when the wish came from the operator token.

***

### picture\_url?

> `optional` **picture\_url?**: `string` \| `null`

Defined in: [api.ts:3111](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3111)

HTTPS picture from the wishing identity, when any.

***

### user\_id?

> `optional` **user\_id?**: `number` \| `null`

Defined in: [api.ts:3095](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3095)

First-party user id when the portal identity is linked.
