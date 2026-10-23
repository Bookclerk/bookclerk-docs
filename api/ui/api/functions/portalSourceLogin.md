[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / portalSourceLogin

# Function: portalSourceLogin()

> **portalSourceLogin**(`id`, `body`): `Promise`\<`void`\>

Defined in: [api.ts:2675](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2675)

Password-connects a storefront account.

## Parameters

### id

`string`

Source plugin id.

### body

Store email and password.

#### email

`string`

#### password

`string`

## Returns

`Promise`\<`void`\>

Resolves when the connection is stored.
