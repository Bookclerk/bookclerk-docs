[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / updateOidcClient

# Function: updateOidcClient()

> **updateOidcClient**(`clientId`, `body`): `Promise`\<[`OidcAsClient`](../interfaces/OidcAsClient.md)\>

Defined in: [api.ts:854](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L854)

Updates an OIDC client without echoing a secret unless flipping to confidential.

## Parameters

### clientId

`string`

Existing client_id.

### body

[`OidcAsClientWrite`](../interfaces/OidcAsClientWrite.md)

Redirects and token policy.

## Returns

`Promise`\<[`OidcAsClient`](../interfaces/OidcAsClient.md)\>

Updated client row (no plaintext secret unless flipping to confidential).
