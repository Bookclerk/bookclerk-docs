[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / putOidcConfig

# Function: putOidcConfig()

> **putOidcConfig**(`body`): `Promise`\<[`OidcBrokerConfigView`](../interfaces/OidcBrokerConfigView.md)\>

Defined in: [api.ts:782](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L782)

Replaces `[auth.oidc]` providers. Omit `client_secret` to keep the stored value.

## Parameters

### body

[`OidcBrokerConfigUpdate`](../interfaces/OidcBrokerConfigUpdate.md)

Broker enablement and provider list.

## Returns

`Promise`\<[`OidcBrokerConfigView`](../interfaces/OidcBrokerConfigView.md)\>

Saved configuration with redacted secrets.
