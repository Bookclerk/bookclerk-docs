[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / OidcProviderConfigView

# Interface: OidcProviderConfigView

Defined in: [api.ts:671](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L671)

Owner/operator view of one `[auth.oidc]` provider (secrets redacted).

## Properties

### allowed\_email\_domains

> **allowed\_email\_domains**: `string`[]

Defined in: [api.ts:683](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L683)

***

### allowed\_emails

> **allowed\_emails**: `string`[]

Defined in: [api.ts:684](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L684)

***

### allowed\_subjects

> **allowed\_subjects**: `string`[]

Defined in: [api.ts:685](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L685)

***

### apple\_key\_id?

> `optional` **apple\_key\_id?**: `string` \| `null`

Defined in: [api.ts:687](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L687)

***

### apple\_team\_id?

> `optional` **apple\_team\_id?**: `string` \| `null`

Defined in: [api.ts:686](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L686)

***

### client\_id

> **client\_id**: `string`

Defined in: [api.ts:676](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L676)

***

### default\_role

> **default\_role**: `string`

Defined in: [api.ts:679](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L679)

***

### has\_apple\_private\_key?

> `optional` **has\_apple\_private\_key?**: `boolean`

Defined in: [api.ts:689](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L689)

***

### has\_client\_secret

> **has\_client\_secret**: `boolean`

Defined in: [api.ts:688](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L688)

***

### id

> **id**: `string`

Defined in: [api.ts:672](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L672)

***

### issuer?

> `optional` **issuer?**: `string` \| `null`

Defined in: [api.ts:675](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L675)

***

### link\_by\_email

> **link\_by\_email**: `boolean`

Defined in: [api.ts:682](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L682)

***

### name

> **name**: `string`

Defined in: [api.ts:673](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L673)

***

### preset?

> `optional` **preset?**: `string` \| `null`

Defined in: [api.ts:674](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L674)

***

### provision

> **provision**: [`OidcProvisionMode`](../type-aliases/OidcProvisionMode.md)

Defined in: [api.ts:678](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L678)

***

### role\_claim

> **role\_claim**: `string`

Defined in: [api.ts:680](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L680)

***

### role\_map

> **role\_map**: `Record`\<`string`, `string`\>

Defined in: [api.ts:681](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L681)

***

### scopes

> **scopes**: `string`[]

Defined in: [api.ts:677](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L677)

***

### secret\_source

> **secret\_source**: [`OidcSecretSource`](../type-aliases/OidcSecretSource.md)

Defined in: [api.ts:690](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L690)
