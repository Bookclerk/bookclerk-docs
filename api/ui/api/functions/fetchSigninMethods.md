[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchSigninMethods

# Function: fetchSigninMethods()

> **fetchSigninMethods**(): `Promise`\<\{ `integrations`: `object`[]; `oidc`: [`OidcProvider`](../interfaces/OidcProvider.md)[]; `operator_token`: `boolean`; `require_second_factor`: `boolean`; \}\>

Defined in: [api.ts:580](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L580)

Public sign-in picker (operator-token availability, SSO, integration logins).

## Returns

`Promise`\<\{ `integrations`: `object`[]; `oidc`: [`OidcProvider`](../interfaces/OidcProvider.md)[]; `operator_token`: `boolean`; `require_second_factor`: `boolean`; \}\>

Whether SPA operator-token login is offered, plus IdP and integration buttons.
