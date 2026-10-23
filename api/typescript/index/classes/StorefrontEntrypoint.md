[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / StorefrontEntrypoint

# Class: StorefrontEntrypoint\<Env\>

Defined in: [plugin.ts:1680](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1680)

`storefront` entrypoint (`ContentSource` interface). Export it as
`export class Storefront extends StorefrontEntrypoint`. Every method takes
and returns the typed ABI structs generated from `schema/plugin.capnp`.

## Extends

- [`NamedEntrypoint`](NamedEntrypoint.md)\<`Env`\>

## Type Parameters

### Env

`Env` *extends* [`BookclerkEnv`](../interfaces/BookclerkEnv.md) = [`BookclerkEnv`](../interfaces/BookclerkEnv.md)

## Constructors

### Constructor

> **new StorefrontEntrypoint**\<`Env`\>(`ctx`, `env`): `StorefrontEntrypoint`\<`Env`\>

Defined in: [cloudflare-workers.d.ts:15](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L15)

#### Parameters

##### ctx

`ExecutionContext`

##### env

`Env`

#### Returns

`StorefrontEntrypoint`\<`Env`\>

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`constructor`](NamedEntrypoint.md#constructor)

## Properties

### ctx

> `readonly` **ctx**: `ExecutionContext`

Defined in: [cloudflare-workers.d.ts:16](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L16)

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`ctx`](NamedEntrypoint.md#ctx)

***

### env

> `readonly` **env**: `Env`

Defined in: [cloudflare-workers.d.ts:17](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L17)

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`env`](NamedEntrypoint.md#env-1)

***

### invocation

> **invocation**: [`Invocation`](../interfaces/Invocation.md)

Defined in: [plugin.ts:1626](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1626)

Invocation identity of the current call (set by `bookclerkInvoke`).

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`invocation`](NamedEntrypoint.md#invocation)

***

### bookclerkMethods

> `static` **bookclerkMethods**: readonly `string`[] = `STOREFRONT_METHODS`

Defined in: [plugin.ts:1681](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1681)

Methods the adapter may dispatch on this entrypoint.

#### Overrides

[`NamedEntrypoint`](NamedEntrypoint.md).[`bookclerkMethods`](NamedEntrypoint.md#bookclerkmethods)

## Methods

### bookclerkInvoke()

> **bookclerkInvoke**(`context`, `method`, ...`args`): `Promise`\<`unknown`\>

Defined in: [plugin.ts:1647](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1647)

**`Internal`**

Adapter dispatch: install granted bindings, then call `method`.

#### Parameters

##### context

[`GrantedContext`](../interfaces/GrantedContext.md)

Granted bindings for this invocation.

##### method

`string`

Method name from the static allowlist.

##### args

...`unknown`[]

Method arguments.

#### Returns

`Promise`\<`unknown`\>

Method result.

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`bookclerkInvoke`](NamedEntrypoint.md#bookclerkinvoke)

***

### catalogDetail()

> **catalogDetail**(`_params`): `Promise`\<[`CatalogHit`](../interfaces/CatalogHit.md) \| `null`\>

Defined in: [plugin.ts:1779](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1779)

Full catalog record for one product; `null` when unknown.

#### Parameters

##### \_params

[`CatalogDetailParams`](../interfaces/CatalogDetailParams.md)

Product identifiers.

#### Returns

`Promise`\<[`CatalogHit`](../interfaces/CatalogHit.md) \| `null`\>

Catalog hit or `null`.

***

### diagnose()

> **diagnose**(): `Promise`\<`string`[]\>

Defined in: [plugin.ts:1787](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1787)

Operator-facing diagnostic lines.

#### Returns

`Promise`\<`string`[]\>

Probe lines.

***

### expandCandidates()

> **expandCandidates**(`_params`): `Promise`\<[`CatalogHit`](../interfaces/CatalogHit.md)[]\>

Defined in: [plugin.ts:1752](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1752)

Related-title expansion from a seed title.

#### Parameters

##### \_params

[`ExpandCandidatesParams`](../interfaces/ExpandCandidatesParams.md)

Seed title plus limits.

#### Returns

`Promise`\<[`CatalogHit`](../interfaces/CatalogHit.md)[]\>

Candidate hits.

***

### fetch()

> **fetch**(`_request?`): `Promise`\<`Response`\>

Defined in: [plugin.ts:1634](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1634)

Rejects HTTP fetch — workerd guests are Workers-RPC only.

#### Parameters

##### \_request?

`Request`

Incoming HTTP request when the entrypoint is fetch-facing.

#### Returns

`Promise`\<`Response`\>

Always a 404 empty response.

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`fetch`](NamedEntrypoint.md#fetch)

***

### fetchTitle()

> **fetchTitle**(`_params`): `Promise`\<[`PlainFetch`](../interfaces/PlainFetch.md)\>

Defined in: [plugin.ts:1708](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1708)

Fetches one title into `params.cacheDir` and returns plain media paths.

#### Parameters

##### \_params

[`FetchTitleParams`](../interfaces/FetchTitleParams.md)

Title identifiers, credentials, fetch options.

#### Returns

`Promise`\<[`PlainFetch`](../interfaces/PlainFetch.md)\>

Plain media parts plus metadata.

***

### health()

> **health**(): `Promise`\<[`HealthOk`](../interfaces/HealthOk.md)\>

Defined in: [plugin.ts:1795](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1795)

Reports whether the storefront session is usable.

#### Returns

`Promise`\<[`HealthOk`](../interfaces/HealthOk.md)\>

Health flag plus detail.

***

### listAccounts()

> **listAccounts**(): `Promise`\<[`SourceAccount`](../interfaces/SourceAccount.md)[]\>

Defined in: [plugin.ts:1716](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1716)

Accounts the guest knows about.

#### Returns

`Promise`\<[`SourceAccount`](../interfaces/SourceAccount.md)[]\>

Account list.

***

### listDeals()

> **listDeals**(`_params`): `Promise`\<[`CatalogHit`](../interfaces/CatalogHit.md)[]\>

Defined in: [plugin.ts:1770](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1770)

Current storefront deals.

#### Parameters

##### \_params

[`ListDealsParams`](../interfaces/ListDealsParams.md)

Optional filters.

#### Returns

`Promise`\<[`CatalogHit`](../interfaces/CatalogHit.md)[]\>

Deal hits.

***

### login()

> **login**(`_params`): `Promise`\<[`LoginResult`](../interfaces/LoginResult.md)\>

Defined in: [plugin.ts:1690](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1690)

Password or one-shot OAuth login. The host seals
`LoginResult.credentials` into `encrypted_secrets`.

#### Parameters

##### \_params

[`LoginParams`](../interfaces/LoginParams.md)

Login parameters.

#### Returns

`Promise`\<[`LoginResult`](../interfaces/LoginResult.md)\>

Account identity plus opaque credentials.

***

### loginComplete()

> **loginComplete**(`_params`): `Promise`\<[`LoginResult`](../interfaces/LoginResult.md)\>

Defined in: [plugin.ts:1734](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1734)

Finishes a login started by [StorefrontEntrypoint.loginStart](#loginstart).

#### Parameters

##### \_params

[`LoginCompleteParams`](../interfaces/LoginCompleteParams.md)

Continuation plus user input.

#### Returns

`Promise`\<[`LoginResult`](../interfaces/LoginResult.md)\>

Account identity plus opaque credentials.

***

### loginStart()

> **loginStart**(`_params`): `Promise`\<[`LoginStartResult`](../interfaces/LoginStartResult.md)\>

Defined in: [plugin.ts:1725](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1725)

Begins an interactive OAuth login.

#### Parameters

##### \_params

[`LoginParams`](../interfaces/LoginParams.md)

Login parameters.

#### Returns

`Promise`\<[`LoginStartResult`](../interfaces/LoginStartResult.md)\>

Continuation for [StorefrontEntrypoint.loginComplete](#logincomplete).

***

### purchaseHint()

> **purchaseHint**(`_params`): `Promise`\<[`PurchaseHint`](../interfaces/PurchaseHint.md) \| `null`\>

Defined in: [plugin.ts:1761](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1761)

Purchase link / price hint; `null` when the title is unknown.

#### Parameters

##### \_params

[`PurchaseHintParams`](../interfaces/PurchaseHintParams.md)

Title identifiers.

#### Returns

`Promise`\<[`PurchaseHint`](../interfaces/PurchaseHint.md) \| `null`\>

Hint or `null`.

***

### scan()

> **scan**(`_params`): `Promise`\<[`ScanSummary`](../interfaces/ScanSummary.md)\>

Defined in: [plugin.ts:1699](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1699)

Library scan; the host upserts `ScanSummary.books`.

#### Parameters

##### \_params

[`ScanParams`](../interfaces/ScanParams.md)

Scan parameters (accounts + credentials).

#### Returns

`Promise`\<[`ScanSummary`](../interfaces/ScanSummary.md)\>

Books discovered plus per-account counters.

***

### searchCatalog()

> **searchCatalog**(`_params`): `Promise`\<[`CatalogHit`](../interfaces/CatalogHit.md)[]\>

Defined in: [plugin.ts:1743](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1743)

Free-text storefront catalog search.

#### Parameters

##### \_params

[`SearchCatalogParams`](../interfaces/SearchCatalogParams.md)

Query parameters.

#### Returns

`Promise`\<[`CatalogHit`](../interfaces/CatalogHit.md)[]\>

Catalog hits.
