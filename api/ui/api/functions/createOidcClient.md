[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / createOidcClient

# Function: createOidcClient()

> **createOidcClient**(`body`): `Promise`\<[`OidcAsClient`](../interfaces/OidcAsClient.md)\>

Defined in: [api.ts:837](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L837)

Creates an OIDC client. Confidential clients return `client_secret` once.

## Parameters

### body

[`OidcAsClientWrite`](../interfaces/OidcAsClientWrite.md)

Client id, redirects, token policy.

## Returns

`Promise`\<[`OidcAsClient`](../interfaces/OidcAsClient.md)\>

Created client; confidential clients include `client_secret` once.
