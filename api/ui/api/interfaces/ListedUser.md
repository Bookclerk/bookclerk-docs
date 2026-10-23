[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / ListedUser

# Interface: ListedUser

Defined in: [api.ts:1106](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1106)

User row from `GET /api/users` (administrator settings).

## Properties

### avatar\_source?

> `optional` **avatar\_source?**: `string` \| `null`

Defined in: [api.ts:1123](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1123)

`auto`, `monogram`, `gravatar`, `upload`, or `sso:{id}`.

***

### display\_name

> **display\_name**: `string` \| `null`

Defined in: [api.ts:1110](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1110)

***

### email?

> `optional` **email?**: `string` \| `null`

Defined in: [api.ts:1112](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1112)

***

### gravatar\_hash?

> `optional` **gravatar\_hash?**: `string` \| `null`

Defined in: [api.ts:1125](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1125)

SHA-256 hex of the contact email for Gravatar.

***

### has\_avatar?

> `optional` **has\_avatar?**: `boolean`

Defined in: [api.ts:1121](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1121)

True when a profile picture file is stored for this user.

***

### has\_password

> **has\_password**: `boolean`

Defined in: [api.ts:1113](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1113)

***

### id

> **id**: `number`

Defined in: [api.ts:1107](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1107)

***

### identities?

> `optional` **identities?**: `object`[]

Defined in: [api.ts:1146](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1146)

Linked login identities (local + OIDC).

#### external\_user\_id

> **external\_user\_id**: `string`

#### label

> **label**: `string` \| `null`

#### provider

> **provider**: `string`

***

### integrations?

> `optional` **integrations?**: `object`[]

Defined in: [api.ts:1140](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1140)

Linked storefront / integration accounts.

#### account\_id

> **account\_id**: `string`

#### label

> **label**: `string` \| `null`

#### source

> **source**: `string`

***

### last\_active\_at?

> `optional` **last\_active\_at?**: `string` \| `null`

Defined in: [api.ts:1117](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1117)

Most recent unexpired portal session activity (RFC 3339). Active vs idle is ~5 minutes.

***

### last\_seen\_at?

> `optional` **last\_seen\_at?**: `string` \| `null`

Defined in: [api.ts:1119](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1119)

Durable last portal use (RFC 3339); survives logout. Null means never signed in.

***

### listening?

> `optional` **listening?**: \{ `last_listened_at`: `string`; `provider`: `string`; `title`: `string` \| `null`; \} \| `null`

Defined in: [api.ts:1134](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1134)

Unfinished listen within ~30 minutes (waveform uses the last ~5 minutes).

***

### login\_name

> **login\_name**: `string` \| `null`

Defined in: [api.ts:1111](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1111)

***

### online?

> `optional` **online?**: `boolean`

Defined in: [api.ts:1115](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1115)

Non-expired portal session present (`true` even when idle).

***

### role

> **role**: `string`

Defined in: [api.ts:1108](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1108)

***

### sso\_pictures?

> `optional` **sso\_pictures?**: `object`[]

Defined in: [api.ts:1127](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1127)

IdP-supplied pictures the user can pick.

#### identity\_id

> **identity\_id**: `number`

#### last\_used\_at?

> `optional` **last\_used\_at?**: `string` \| `null`

#### picture\_url

> **picture\_url**: `string`

#### provider

> **provider**: `string`

***

### status

> **status**: `string`

Defined in: [api.ts:1109](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1109)
