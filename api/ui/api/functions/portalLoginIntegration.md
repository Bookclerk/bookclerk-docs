[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / portalLoginIntegration

# Function: portalLoginIntegration()

> **portalLoginIntegration**(`body`): `Promise`\<`void`\>

Defined in: [api.ts:2619](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2619)

Signs in via an integration provider username/password.

## Parameters

### body

Provider id plus credentials.

#### password

`string`

#### provider

`string`

#### username

`string`

## Returns

`Promise`\<`void`\>

Resolves when the portal session is established.
