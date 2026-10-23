[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / OidcProviderConfigUpdate

# Interface: OidcProviderConfigUpdate

Defined in: [api.ts:714](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L714)

Provider fields accepted by `PUT /api/auth/oidc/config`.

## Properties

### allowed\_email\_domains

> **allowed\_email\_domains**: `string`[]

Defined in: [api.ts:728](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L728)

***

### allowed\_emails

> **allowed\_emails**: `string`[]

Defined in: [api.ts:729](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L729)

***

### allowed\_subjects

> **allowed\_subjects**: `string`[]

Defined in: [api.ts:730](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L730)

***

### apple\_key\_id?

> `optional` **apple\_key\_id?**: `string` \| `null`

Defined in: [api.ts:732](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L732)

***

### apple\_private\_key?

> `optional` **apple\_private\_key?**: `string`

Defined in: [api.ts:733](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L733)

***

### apple\_team\_id?

> `optional` **apple\_team\_id?**: `string` \| `null`

Defined in: [api.ts:731](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L731)

***

### client\_id

> **client\_id**: `string`

Defined in: [api.ts:719](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L719)

***

### client\_secret?

> `optional` **client\_secret?**: `string`

Defined in: [api.ts:721](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L721)

Omit to keep; empty string to clear; non-empty to store.

***

### default\_role

> **default\_role**: `string`

Defined in: [api.ts:724](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L724)

***

### id

> **id**: `string`

Defined in: [api.ts:715](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L715)

***

### issuer?

> `optional` **issuer?**: `string` \| `null`

Defined in: [api.ts:718](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L718)

***

### link\_by\_email

> **link\_by\_email**: `boolean`

Defined in: [api.ts:727](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L727)

***

### name

> **name**: `string`

Defined in: [api.ts:716](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L716)

***

### preset?

> `optional` **preset?**: `string` \| `null`

Defined in: [api.ts:717](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L717)

***

### provision

> **provision**: [`OidcProvisionMode`](../type-aliases/OidcProvisionMode.md)

Defined in: [api.ts:723](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L723)

***

### role\_claim

> **role\_claim**: `string`

Defined in: [api.ts:725](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L725)

***

### role\_map

> **role\_map**: `Record`\<`string`, `string`\>

Defined in: [api.ts:726](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L726)

***

### scopes

> **scopes**: `string`[]

Defined in: [api.ts:722](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L722)
