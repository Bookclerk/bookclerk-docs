[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / OidcAsClient

# Interface: OidcAsClient

Defined in: [api.ts:793](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L793)

Bookclerk-as-IdP OAuth client (secrets never listed after create/rotate).

## Properties

### allowed\_scopes

> **allowed\_scopes**: `string`[]

Defined in: [api.ts:799](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L799)

***

### client\_id

> **client\_id**: `string`

Defined in: [api.ts:794](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L794)

***

### client\_secret?

> `optional` **client\_secret?**: `string` \| `null`

Defined in: [api.ts:805](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L805)

Plaintext secret, returned only on create or rotate.

***

### confidential

> **confidential**: `boolean`

Defined in: [api.ts:797](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L797)

***

### enabled

> **enabled**: `boolean`

Defined in: [api.ts:801](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L801)

***

### has\_secret

> **has\_secret**: `boolean`

Defined in: [api.ts:800](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L800)

***

### issue\_refresh\_token

> **issue\_refresh\_token**: `boolean`

Defined in: [api.ts:798](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L798)

***

### name?

> `optional` **name?**: `string` \| `null`

Defined in: [api.ts:795](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L795)

***

### plugin\_id?

> `optional` **plugin\_id?**: `string` \| `null`

Defined in: [api.ts:803](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L803)

Set when a plugin owns this client (redirects are read-only).

***

### redirect\_uris

> **redirect\_uris**: `string`[]

Defined in: [api.ts:796](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L796)
