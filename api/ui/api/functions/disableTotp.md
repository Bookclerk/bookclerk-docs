[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / disableTotp

# Function: disableTotp()

> **disableTotp**(`currentPassword?`): `Promise`\<`void`\>

Defined in: [api.ts:1528](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1528)

Disables TOTP for the current user.

## Parameters

### currentPassword?

`string`

Required when the portal session is older than the reauth window.

## Returns

`Promise`\<`void`\>

Resolves when TOTP is disabled.
