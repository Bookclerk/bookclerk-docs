[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / AuthMeUser

# Interface: AuthMeUser

Defined in: [api.ts:35](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L35)

Signed-in administrator or member profile from `/api/auth/me`.

## Properties

### avatar\_source?

> `optional` **avatar\_source?**: `string` \| `null`

Defined in: [api.ts:46](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L46)

`auto`, `monogram`, `gravatar`, `upload`, or `sso:{id}`.

***

### display\_name

> **display\_name**: `string` \| `null`

Defined in: [api.ts:38](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L38)

***

### email?

> `optional` **email?**: `string` \| `null`

Defined in: [api.ts:41](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L41)

***

### gravatar\_hash?

> `optional` **gravatar\_hash?**: `string` \| `null`

Defined in: [api.ts:48](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L48)

SHA-256 hex of the contact email for Gravatar.

***

### has\_avatar?

> `optional` **has\_avatar?**: `boolean`

Defined in: [api.ts:44](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L44)

True when a profile picture file is stored for this user.

***

### has\_password?

> `optional` **has\_password?**: `boolean`

Defined in: [api.ts:42](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L42)

***

### id

> **id**: `number`

Defined in: [api.ts:36](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L36)

***

### login\_name?

> `optional` **login\_name?**: `string` \| `null`

Defined in: [api.ts:39](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L39)

***

### role

> **role**: `string`

Defined in: [api.ts:37](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L37)

***

### sso\_pictures?

> `optional` **sso\_pictures?**: `object`[]

Defined in: [api.ts:50](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L50)

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

### status?

> `optional` **status?**: `string`

Defined in: [api.ts:40](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L40)
