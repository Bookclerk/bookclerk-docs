[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / approvePluginConsent

# Function: approvePluginConsent()

> **approvePluginConsent**(`id`, `grant?`, `pluginKey?`): `Promise`\<[`PluginConsentResponse`](../interfaces/PluginConsentResponse.md)\>

Defined in: [api.ts:1880](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1880)

Approves network consent for a plugin.

## Parameters

### id

`string`

Display alias from Settings.

### grant?

Operator-approved grant overlay; omitted fields keep defaults.

#### addressCidrs?

`string`[]

#### allowUndeclaredPublicRedirects?

`boolean`

#### bindings

`string`[]

#### compatibilityFlags

`string`[]

#### cpuCores?

`number`

#### cpuMs?

`number`

#### diskMib?

`number`

#### domains

`string`[]

#### extraProcesses?

`number`

#### memoryMib?

`number`

#### networkMode

`string`

#### subrequests?

`number`

#### tcp?

`object`[]

### pluginKey?

`string`

Provenance-qualified PluginKey when the alias is ambiguous.

## Returns

`Promise`\<[`PluginConsentResponse`](../interfaces/PluginConsentResponse.md)\>

Updated consent response after approval.
