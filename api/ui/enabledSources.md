# enabledSources

## Variables

### DISCOVER\_SEARCH\_PLACEHOLDER

```ts
const DISCOVER_SEARCH_PLACEHOLDER: "Search Book Stores... (Enter for results)" = "Search Book Stores... (Enter for results)";
```

Placeholder for the Discover search field when store names are unknown.

## Functions

### loadEnabledSources()

```ts
function loadEnabledSources(): Promise<PortalSource[]>;
```

Loads enabled storefront sources from the portal API.

Used by Discover / Preferences so store lists track the storefront plugins
deployed on this host instead of a hardcoded product catalog.

#### Returns

`Promise`\<[`PortalSource`](api.md#portalsource)[]\>

Enabled sources (may be empty when none are enabled).
