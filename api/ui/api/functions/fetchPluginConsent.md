[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchPluginConsent

# Function: fetchPluginConsent()

> **fetchPluginConsent**(`id`, `pluginKey?`): `Promise`\<[`PluginConsentResponse`](../interfaces/PluginConsentResponse.md)\>

Defined in: [api.ts:1859](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1859)

Loads network/capability consent status for a plugin before enable.

Call from Settings when an operator reviews domains/bindings that still need
approval; `covered` is false until approvePluginConsent succeeds.

## Parameters

### id

`string`

Display alias from Settings.

### pluginKey?

`string`

Provenance-qualified PluginKey when the alias is ambiguous.

## Returns

`Promise`\<[`PluginConsentResponse`](../interfaces/PluginConsentResponse.md)\>

Current consent request payload and whether it already covers the ask.
