[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / rotateOidcClientSecret

# Function: rotateOidcClientSecret()

> **rotateOidcClientSecret**(`clientId`, `currentPassword?`): `Promise`\<[`OidcAsClient`](../interfaces/OidcAsClient.md)\>

Defined in: [api.ts:896](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L896)

Rotates a confidential client's secret; plaintext is returned once.

## Parameters

### clientId

`string`

Confidential client to rotate.

### currentPassword?

`string`

Owner re-auth when required.

## Returns

`Promise`\<[`OidcAsClient`](../interfaces/OidcAsClient.md)\>

Client row including the new plaintext secret once.
