[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [enabledSources](../README.md) / loadEnabledSources

# Function: loadEnabledSources()

> **loadEnabledSources**(): `Promise`\<[`PortalSource`](../../api/interfaces/PortalSource.md)[]\>

Defined in: [enabledSources.ts:11](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/enabledSources.ts#L11)

Loads enabled storefront sources from the portal API.

Used by Discover / Preferences so store lists track the storefront plugins
deployed on this host instead of a hardcoded product catalog.

## Returns

`Promise`\<[`PortalSource`](../../api/interfaces/PortalSource.md)[]\>

Enabled sources (may be empty when none are enabled).
