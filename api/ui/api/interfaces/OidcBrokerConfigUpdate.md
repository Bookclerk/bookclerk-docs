[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / OidcBrokerConfigUpdate

# Interface: OidcBrokerConfigUpdate

Defined in: [api.ts:737](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L737)

Body for `PUT /api/auth/oidc/config`.

## Properties

### allowed\_email\_domains

> **allowed\_email\_domains**: `string`[]

Defined in: [api.ts:739](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L739)

***

### current\_password?

> `optional` **current\_password?**: `string`

Defined in: [api.ts:743](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L743)

***

### enabled

> **enabled**: `boolean`

Defined in: [api.ts:738](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L738)

***

### providers

> **providers**: [`OidcProviderConfigUpdate`](OidcProviderConfigUpdate.md)[]

Defined in: [api.ts:740](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L740)

***

### public\_origin?

> `optional` **public\_origin?**: `string` \| `null`

Defined in: [api.ts:742](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L742)

Omit to leave unchanged; empty string clears `integrations.public_origin`.
