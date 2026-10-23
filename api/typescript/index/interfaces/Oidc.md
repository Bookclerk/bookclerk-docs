[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / Oidc

# Interface: Oidc

Defined in: [generated.ts:1127](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1127)

`oidc` entrypoint: relying-party client templates and credential
verification on behalf of the host authorization server.

## Methods

### authenticateUser()

> **authenticateUser**(`params`): `Promise`\<[`ExternalUserReply`](../type-aliases/ExternalUserReply.md)\>

Defined in: [generated.ts:1140](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1140)

Verify remote credentials on behalf of the host.

#### Parameters

##### params

[`AuthenticateUserParams`](AuthenticateUserParams.md)

Username and password.

#### Returns

`Promise`\<[`ExternalUserReply`](../type-aliases/ExternalUserReply.md)\>

[ExternalUserReply](../type-aliases/ExternalUserReply.md)

***

### clients()

> **clients**(): `Promise`\<[`OidcClientsReply`](../type-aliases/OidcClientsReply.md)\>

Defined in: [generated.ts:1133](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1133)

Plugin-provided OIDC AS client templates. Empty list when unused.

#### Returns

`Promise`\<[`OidcClientsReply`](../type-aliases/OidcClientsReply.md)\>

[OidcClientsReply](../type-aliases/OidcClientsReply.md)
