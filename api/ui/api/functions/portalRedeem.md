[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / portalRedeem

# Function: portalRedeem()

> **portalRedeem**(`ticket`, `password?`): `Promise`\<`void`\>

Defined in: [api.ts:2553](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2553)

Redeems a claim ticket into a portal user session.

## Parameters

### ticket

`string`

One-time claim ticket string.

### password?

`string`

Optional password when claiming invite/reset tickets.

## Returns

`Promise`\<`void`\>

Resolves when the session cookie is set.
