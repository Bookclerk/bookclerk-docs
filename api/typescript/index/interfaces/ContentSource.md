[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ContentSource

# Interface: ContentSource

Defined in: [generated.ts:964](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L964)

Storefront content source (not byte Source). Every method takes and
returns typed structs from the "Typed method payloads" section.

## Methods

### catalogDetail()

> **catalogDetail**(`params`): `Promise`\<[`CatalogDetailReply`](../type-aliases/CatalogDetailReply.md)\>

Defined in: [generated.ts:1052](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1052)

Full catalog record for one product.

#### Parameters

##### params

[`CatalogDetailParams`](CatalogDetailParams.md)

Product id and optional ISBN.

#### Returns

`Promise`\<[`CatalogDetailReply`](../type-aliases/CatalogDetailReply.md)\>

[CatalogDetailReply](../type-aliases/CatalogDetailReply.md)

***

### diagnose()

> **diagnose**(): `Promise`\<[`DiagnoseReply`](../type-aliases/DiagnoseReply.md)\>

Defined in: [generated.ts:1045](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1045)

Human-readable diagnostic lines.

#### Returns

`Promise`\<[`DiagnoseReply`](../type-aliases/DiagnoseReply.md)\>

[DiagnoseReply](../type-aliases/DiagnoseReply.md)

***

### expandCandidates()

> **expandCandidates**(`params`): `Promise`\<[`CatalogHitsReply`](../type-aliases/CatalogHitsReply.md)\>

Defined in: [generated.ts:1019](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1019)

Related-title expansion from a seed title.

#### Parameters

##### params

[`ExpandCandidatesParams`](ExpandCandidatesParams.md)

Seed identity fields and limit.

#### Returns

`Promise`\<[`CatalogHitsReply`](../type-aliases/CatalogHitsReply.md)\>

[CatalogHitsReply](../type-aliases/CatalogHitsReply.md)

***

### fetchTitle()

> **fetchTitle**(`params`): `Promise`\<[`FetchTitleReply`](../type-aliases/FetchTitleReply.md)\>

Defined in: [generated.ts:985](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L985)

Download and decrypt one title into `cacheDir`.

#### Parameters

##### params

[`FetchTitleParams`](FetchTitleParams.md)

Title, credentials, and fetch options.

#### Returns

`Promise`\<[`FetchTitleReply`](../type-aliases/FetchTitleReply.md)\>

[FetchTitleReply](../type-aliases/FetchTitleReply.md)

***

### health()

> **health**(): `Promise`\<[`HealthReply`](../type-aliases/HealthReply.md)\>

Defined in: [generated.ts:1039](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1039)

Liveness / readiness probe.

#### Returns

`Promise`\<[`HealthReply`](../type-aliases/HealthReply.md)\>

[HealthReply](../type-aliases/HealthReply.md)

***

### listAccounts()

> **listAccounts**(): `Promise`\<[`SourceAccountsReply`](../type-aliases/SourceAccountsReply.md)\>

Defined in: [generated.ts:991](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L991)

Enumerate accounts the guest knows about.

#### Returns

`Promise`\<[`SourceAccountsReply`](../type-aliases/SourceAccountsReply.md)\>

[SourceAccountsReply](../type-aliases/SourceAccountsReply.md)

***

### listDeals()

> **listDeals**(`params`): `Promise`\<[`CatalogHitsReply`](../type-aliases/CatalogHitsReply.md)\>

Defined in: [generated.ts:1033](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1033)

Current storefront deals.

#### Parameters

##### params

[`ListDealsParams`](ListDealsParams.md)

Optional result cap.

#### Returns

`Promise`\<[`CatalogHitsReply`](../type-aliases/CatalogHitsReply.md)\>

[CatalogHitsReply](../type-aliases/CatalogHitsReply.md)

***

### login()

> **login**(`params`): `Promise`\<[`LoginReply`](../type-aliases/LoginReply.md)\>

Defined in: [generated.ts:971](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L971)

Connect an account (password or one-shot OAuth).

#### Parameters

##### params

[`LoginParams`](LoginParams.md)

Credentials, marketplace, callback wiring.

#### Returns

`Promise`\<[`LoginReply`](../type-aliases/LoginReply.md)\>

[LoginReply](../type-aliases/LoginReply.md)

***

### loginComplete()

> **loginComplete**(`params`): `Promise`\<[`LoginReply`](../type-aliases/LoginReply.md)\>

Defined in: [generated.ts:1005](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1005)

Finish an interactive OAuth login started by `loginStart`.

#### Parameters

##### params

[`LoginCompleteParams`](LoginCompleteParams.md)

Session id from `loginStart`.

#### Returns

`Promise`\<[`LoginReply`](../type-aliases/LoginReply.md)\>

[LoginReply](../type-aliases/LoginReply.md)

***

### loginStart()

> **loginStart**(`params`): `Promise`\<[`LoginStartReply`](../type-aliases/LoginStartReply.md)\>

Defined in: [generated.ts:998](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L998)

Begin an interactive OAuth login; returns a session id.

#### Parameters

##### params

[`LoginParams`](LoginParams.md)

Same shape as `login`; host fills callback IPC.

#### Returns

`Promise`\<[`LoginStartReply`](../type-aliases/LoginStartReply.md)\>

[LoginStartReply](../type-aliases/LoginStartReply.md)

***

### purchaseHint()

> **purchaseHint**(`params`): `Promise`\<[`PurchaseHintReply`](../type-aliases/PurchaseHintReply.md)\>

Defined in: [generated.ts:1026](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1026)

Purchase link / price hint for one title.

#### Parameters

##### params

[`PurchaseHintParams`](PurchaseHintParams.md)

Identity fields and price flag.

#### Returns

`Promise`\<[`PurchaseHintReply`](../type-aliases/PurchaseHintReply.md)\>

[PurchaseHintReply](../type-aliases/PurchaseHintReply.md)

***

### scan()

> **scan**(`params`): `Promise`\<[`ScanReply`](../type-aliases/ScanReply.md)\>

Defined in: [generated.ts:978](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L978)

Sync library rows for one or more accounts.

#### Parameters

##### params

[`ScanParams`](ScanParams.md)

Accounts, paging, and host-sealed credentials.

#### Returns

`Promise`\<[`ScanReply`](../type-aliases/ScanReply.md)\>

[ScanReply](../type-aliases/ScanReply.md)

***

### searchCatalog()

> **searchCatalog**(`params`): `Promise`\<[`CatalogHitsReply`](../type-aliases/CatalogHitsReply.md)\>

Defined in: [generated.ts:1012](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1012)

Free-text storefront catalog search.

#### Parameters

##### params

[`SearchCatalogParams`](SearchCatalogParams.md)

Query, region, paging, sort, facet.

#### Returns

`Promise`\<[`CatalogHitsReply`](../type-aliases/CatalogHitsReply.md)\>

[CatalogHitsReply](../type-aliases/CatalogHitsReply.md)
