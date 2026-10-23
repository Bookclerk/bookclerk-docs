[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / OidcClientTemplate

# Interface: OidcClientTemplate

Defined in: [generated.ts:201](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L201)

Bookclerk-as-IdP relying-party template. Plugins declare callback path and
client id; the host materializes `oidc_clients` rows and remains the AS.
`originConfigKey` is a dotted config path (e.g. integrations.audiobookshelf.base_url).

## Properties

### callbackPath

> **callbackPath**: `string`

Defined in: [generated.ts:207](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L207)

Redirect URI path relative to the integration origin.

***

### clientId

> **clientId**: `string`

Defined in: [generated.ts:203](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L203)

OIDC client id the host materializes.

***

### defaultScopes

> **defaultScopes**: `string`[]

Defined in: [generated.ts:211](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L211)

Scopes granted by default.

***

### displayName

> **displayName**: `string`

Defined in: [generated.ts:205](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L205)

Name shown on consent / admin screens.

***

### issueRefreshToken

> **issueRefreshToken**: `boolean`

Defined in: [generated.ts:213](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L213)

When true, the AS issues refresh tokens to this client.

***

### originConfigKey

> **originConfigKey**: `string`

Defined in: [generated.ts:215](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L215)

Dotted config path holding the client's origin URL.

***

### publicClient

> **publicClient**: `boolean`

Defined in: [generated.ts:209](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L209)

When true, the client uses PKCE without a client secret.
