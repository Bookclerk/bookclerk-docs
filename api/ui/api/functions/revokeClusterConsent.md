[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / revokeClusterConsent

# Function: revokeClusterConsent()

> **revokeClusterConsent**(`clusterId`, `pluginKey`): `Promise`\<\{ `consent`: [`RemoteConsentView`](../interfaces/RemoteConsentView.md); \}\>

Defined in: [api.ts:2511](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2511)

Withdraws an approval: held jobs of the plugin are cancelled and given
back, and nothing of it is claimed until approved again.

## Parameters

### clusterId

`string`

Owner cluster.

### pluginKey

`string`

Canonical plugin key.

## Returns

`Promise`\<\{ `consent`: [`RemoteConsentView`](../interfaces/RemoteConsentView.md); \}\>

The consent row after revoking.
