[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / OidcBrokerConfigView

# Interface: OidcBrokerConfigView

Defined in: [api.ts:694](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L694)

Owner/operator identity-broker settings payload.

## Properties

### allowed\_email\_domains

> **allowed\_email\_domains**: `string`[]

Defined in: [api.ts:696](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L696)

***

### authorization\_endpoint?

> `optional` **authorization\_endpoint?**: `string`

Defined in: [api.ts:705](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L705)

***

### callback\_url?

> `optional` **callback\_url?**: `string` \| `null`

Defined in: [api.ts:709](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L709)

***

### detected\_origin?

> `optional` **detected\_origin?**: `string`

Defined in: [api.ts:702](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L702)

Origin detected from this request when it is a bound loopback listen address.

***

### discovery\_url?

> `optional` **discovery\_url?**: `string`

Defined in: [api.ts:704](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L704)

Absolute OpenID discovery URL.

***

### enabled

> **enabled**: `boolean`

Defined in: [api.ts:695](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L695)

***

### issuer\_url?

> `optional` **issuer\_url?**: `string`

Defined in: [api.ts:700](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L700)

Effective issuer origin (`public_origin`, else bound loopback, else localhost).

***

### providers

> **providers**: [`OidcProviderConfigView`](OidcProviderConfigView.md)[]

Defined in: [api.ts:710](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L710)

***

### public\_origin?

> `optional` **public\_origin?**: `string` \| `null`

Defined in: [api.ts:698](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L698)

Configured `integrations.public_origin` (issuer / callback / invite base).

***

### revocation\_endpoint?

> `optional` **revocation\_endpoint?**: `string`

Defined in: [api.ts:708](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L708)

***

### token\_endpoint?

> `optional` **token\_endpoint?**: `string`

Defined in: [api.ts:706](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L706)

***

### userinfo\_endpoint?

> `optional` **userinfo\_endpoint?**: `string`

Defined in: [api.ts:707](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L707)
