[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / deleteOidcClient

# Function: deleteOidcClient()

> **deleteOidcClient**(`clientId`, `currentPassword?`): `Promise`\<`void`\>

Defined in: [api.ts:874](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L874)

Deletes a registered OIDC client.

## Parameters

### clientId

`string`

Client to remove.

### currentPassword?

`string`

Owner re-auth when the session is older than 15 minutes.

## Returns

`Promise`\<`void`\>

Resolves when the client is removed.
