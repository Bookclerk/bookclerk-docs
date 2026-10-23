# catalogTitle

## Type Aliases

### CatalogFilterKind

```ts
type CatalogFilterKind = "all" | "authors" | "series" | "narrators" | "genres" | "sources";
```

Client-side catalog filter categories (legacy / local lists).

***

### CatalogSearchField

```ts
type CatalogSearchField = "author" | "narrator" | "series" | "genre";
```

Storefront search facet passed as `?field=` on `/api/discover/search`.

***

### CatalogSortKey

```ts
type CatalogSortKey = CatalogSearchSort;
```

Alias of [CatalogSearchSort](api.md#catalogsearchsort) for Discover UI state.

***

### CatalogTitle

```ts
type CatalogTitle = object;
```

Unified title shape for Discover / Wishlist detail dialogs and result rows.

#### Properties

##### asin

```ts
asin: string | null;
```

##### authors

```ts
authors: string | null;
```

##### cover\_url

```ts
cover_url: string | null;
```

##### description?

```ts
optional description?: string | null;
```

##### genres?

```ts
optional genres?: string | null;
```

##### is\_abridged?

```ts
optional is_abridged?: boolean | null;
```

`true` abridged / `false` unabridged when known.

##### isbn

```ts
isbn: string | null;
```

##### language?

```ts
optional language?: string | null;
```

##### length\_minutes?

```ts
optional length_minutes?: number | null;
```

##### narrators

```ts
narrators: string | null;
```

##### notes?

```ts
optional notes?: string | null;
```

##### published\_at?

```ts
optional published_at?: string | null;
```

##### publisher?

```ts
optional publisher?: string | null;
```

##### purchase\_hints?

```ts
optional purchase_hints?: PurchaseHint[];
```

##### rating\_overall?

```ts
optional rating_overall?: number | null;
```

Audible community overall rating (0–5), when known.

##### reasons?

```ts
optional reasons?: string[];
```

##### series

```ts
series: string | null;
```

##### series\_index?

```ts
optional series_index?: string | null;
```

##### sources

```ts
sources: string[];
```

##### store\_editions

```ts
store_editions: StoreEdition[];
```

##### subtitle?

```ts
optional subtitle?: string | null;
```

##### title

```ts
title: string;
```

##### wish\_count?

```ts
optional wish_count?: number;
```

Distinct wisher count (may exceed `wishers.length`).

##### wishers?

```ts
optional wishers?: QueueWisher[];
```

People who have this title on an open wishlist.

##### wishlist\_uuid?

```ts
optional wishlist_uuid?: string | null;
```

Present when the title is already on the current user’s wishlist.

##### work\_key

```ts
work_key: string;
```

***

### RuntimeBucket

```ts
type RuntimeBucket = "any" | "under6" | "6to12" | "12to20" | "over20";
```

Discover runtime length buckets for server filters.

## Variables

### CATALOG\_FILTER\_KINDS

```ts
const CATALOG_FILTER_KINDS: object[];
```

UI labels for [CatalogFilterKind](#catalogfilterkind).

#### Type Declaration

| Name | Type |
| ------ | ------ |
| `label` | `string` |
| `value` | [`CatalogFilterKind`](#catalogfilterkind) |

***

### CATALOG\_LANGUAGE\_ALL

```ts
const CATALOG_LANGUAGE_ALL: "__all__" = "__all__";
```

Wire value for “no hard language filter” in the Discover language control.

***

### CATALOG\_LANGUAGE\_OPTIONS

```ts
const CATALOG_LANGUAGE_OPTIONS: object[];
```

Language select options for Discover (includes [CATALOG\_LANGUAGE\_ALL](#catalog_language_all)).

#### Type Declaration

| Name | Type |
| ------ | ------ |
| `label` | `string` |
| `value` | `string` |

***

### CATALOG\_SORT\_OPTIONS

```ts
const CATALOG_SORT_OPTIONS: object[];
```

UI labels for Discover catalog sort keys.

#### Type Declaration

| Name | Type |
| ------ | ------ |
| `label` | `string` |
| `value` | [`CatalogSortKey`](#catalogsortkey) |

***

### CATALOG\_SOURCE\_IDS

```ts
const CATALOG_SOURCE_IDS: readonly ["audible", "chirp", "libro", "graphicaudio"];
```

Known storefront ids for prefs / empty facet fallbacks.

***

### RUNTIME\_BUCKET\_OPTIONS

```ts
const RUNTIME_BUCKET_OPTIONS: object[];
```

UI options and minute bounds for [RuntimeBucket](#runtimebucket).

#### Type Declaration

| Name | Type |
| ------ | ------ |
| `label` | `string` |
| `max?` | `number` |
| `min?` | `number` |
| `value` | [`RuntimeBucket`](#runtimebucket) |

***

### VIRTUAL\_VOICE\_EXCLUDE

```ts
const VIRTUAL_VOICE_EXCLUDE: "Virtual Voice" = "Virtual Voice";
```

Narrator name excluded when “Hide Virtual Voice” is enabled.

## Functions

### applyTitleMeta()

```ts
function applyTitleMeta(title, meta): CatalogTitle;
```

Overlays public Audnexus / catalog metadata onto a sparse [CatalogTitle](#catalogtitle).

An Audnexus ASIN is a trusted Audible product id (same rule as purchase-hint
resolve), so a missing Audible store edition is filled in for Where-to-buy.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `title` | [`CatalogTitle`](#catalogtitle) | Existing title. |
| `meta` | [`TitleMeta`](api.md#titlemeta) \| `null` \| `undefined` | Fetched metadata (no-op when nullish). |

#### Returns

[`CatalogTitle`](#catalogtitle)

Merged title.

***

### attachWishersFromQueue()

```ts
function attachWishersFromQueue(title, queue): CatalogTitle;
```

Copies household queue identity onto a personal wishlist title.

Personal `TitleRequest` rows do not include household wishers or the merged
store editions from the global queue. Discover shelves use that merged row,
so Wishlist must union the same editions or Where-to-buy diverges.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `title` | [`CatalogTitle`](#catalogtitle) | Catalog title, typically from [catalogTitleFromRequest](#catalogtitlefromrequest). |
| `queue` | readonly [`GlobalQueueEntry`](api.md#globalqueueentry)[] | Global request-queue entries. |

#### Returns

[`CatalogTitle`](#catalogtitle)

Title with wishers, identifiers, and store editions from the queue.

***

### attachWishlistCommerce()

```ts
function attachWishlistCommerce(title, wishlist): CatalogTitle;
```

Unions snapshotted wishlist store editions / hints onto a Discover (or other)
catalog title so Where-to-buy matches the Wishlist detail for the same work.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `title` | [`CatalogTitle`](#catalogtitle) | Catalog title from search or shelves. |
| `wishlist` | readonly [`TitleRequest`](api.md#titlerequest)[] | Open personal wishlist rows. |

#### Returns

[`CatalogTitle`](#catalogtitle)

Title with wishlist commerce merged when a row matches.

***

### buildCatalogSearchFilters()

```ts
function buildCatalogSearchFilters(opts): CatalogSearchFilters;
```

Builds the server filter payload from UI multi-select state.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `opts` | \{ `authors`: `string`[]; `excludeSources?`: `string`[]; `genres`: `string`[]; `hideVirtualVoice`: `boolean`; `languages?`: `string`[]; `minRating?`: `number` \| `null`; `narrators`: `string`[]; `runtimeBucket?`: [`RuntimeBucket`](#runtimebucket); `series`: `string`[]; `sources?`: `string`[]; \} | Include lists, exclusions, rating, and runtime bucket. |
| `opts.authors` | `string`[] | - |
| `opts.excludeSources?` | `string`[] | - |
| `opts.genres` | `string`[] | - |
| `opts.hideVirtualVoice` | `boolean` | - |
| `opts.languages?` | `string`[] | - |
| `opts.minRating?` | `number` \| `null` | - |
| `opts.narrators` | `string`[] | - |
| `opts.runtimeBucket?` | [`RuntimeBucket`](#runtimebucket) | - |
| `opts.series` | `string`[] | - |
| `opts.sources?` | `string`[] | **Deprecated** prefer excludeSources |

#### Returns

[`CatalogSearchFilters`](api.md#catalogsearchfilters)

`CatalogSearchFilters` for `searchCatalog`.

***

### catalogFilterValues()

```ts
function catalogFilterValues(titles, kind): object[];
```

Builds distinct local filter options from a title list.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `titles` | [`CatalogTitle`](#catalogtitle)[] | Catalog titles. |
| `kind` | [`CatalogFilterKind`](#catalogfilterkind) | Filter category. |

#### Returns

`object`[]

Value/label pairs.

***

### catalogIdentityMatches()

```ts
function catalogIdentityMatches(a, b): boolean;
```

True when two catalog identities refer to the same work (work key, ASIN, ISBN, or title).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `a` | \{ `asin?`: `string` \| `null`; `isbn?`: `string` \| `null`; `title?`: `string` \| `null`; `work_key?`: `string` \| `null`; \} | First identity. |
| `a.asin?` | `string` \| `null` | - |
| `a.isbn?` | `string` \| `null` | - |
| `a.title?` | `string` \| `null` | - |
| `a.work_key?` | `string` \| `null` | - |
| `b` | \{ `asin?`: `string` \| `null`; `isbn?`: `string` \| `null`; `title?`: `string` \| `null`; `work_key?`: `string` \| `null`; \} | Second identity. |
| `b.asin?` | `string` \| `null` | - |
| `b.isbn?` | `string` \| `null` | - |
| `b.title?` | `string` \| `null` | - |
| `b.work_key?` | `string` \| `null` | - |

#### Returns

`boolean`

Whether the rows should share store editions and wishlist state.

***

### catalogLanguageRank()

```ts
function catalogLanguageRank(hitLanguage, preferred?): number;
```

Soft language rank for ordering search hits.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `hitLanguage` | `string` \| `null` \| `undefined` | Language on the hit. |
| `preferred` | `string` | Preferred language (default: browser). |

#### Returns

`number`

`0` preferred, `1` unknown, `2` other.

***

### catalogLanguageSearchOpts()

```ts
function catalogLanguageSearchOpts(filterLanguage): object;
```

Maps Discover language control → hard filter / all-languages flag.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `filterLanguage` | `string` | Selected language code or [CATALOG\_LANGUAGE\_ALL](#catalog_language_all). |

#### Returns

`object`

Server search language options.

| Name | Type |
| ------ | ------ |
| `allLanguages?` | `boolean` |
| `languages?` | `string`[] |

***

### catalogLanguageSelectOptions()

```ts
function catalogLanguageSelectOptions(preferred?): object[];
```

Options for the language `<select>`, ensuring the browser default appears.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `preferred` | `string` | Preferred BCP-47 primary tag (default: browser language). |

#### Returns

`object`[]

Value/label pairs for the control.

***

### catalogMatchesFilter()

```ts
function catalogMatchesFilter(
   title, 
   kind, 
   value
): boolean;
```

Returns whether a title matches a local catalog filter.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `title` | [`CatalogTitle`](#catalogtitle) | Catalog title. |
| `kind` | [`CatalogFilterKind`](#catalogfilterkind) | Filter category. |
| `value` | `string` | Selected value (`all` / empty always matches). |

#### Returns

`boolean`

True when visible under the filter.

***

### catalogTitleFromHit()

```ts
function catalogTitleFromHit(hit, wishlistUuid?): CatalogTitle;
```

Maps a catalog search hit into a [CatalogTitle](#catalogtitle).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `hit` | [`CatalogSearchHit`](api.md#catalogsearchhit) | Search hit. |
| `wishlistUuid?` | `string` \| `null` | Optional wishlist uuid when already wished. |

#### Returns

[`CatalogTitle`](#catalogtitle)

Unified catalog title.

***

### catalogTitleFromQueueEntry()

```ts
function catalogTitleFromQueueEntry(entry, wishlistUuid?): CatalogTitle;
```

Maps a global request-queue entry into a [CatalogTitle](#catalogtitle).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `entry` | [`GlobalQueueEntry`](api.md#globalqueueentry) | Queue entry. |
| `wishlistUuid?` | `string` \| `null` | Optional wishlist uuid when the current user also wished it. |

#### Returns

[`CatalogTitle`](#catalogtitle)

Unified catalog title.

***

### catalogTitleFromRec()

```ts
function catalogTitleFromRec(rec, wishlistUuid?): CatalogTitle;
```

Maps a Discover recommendation into a [CatalogTitle](#catalogtitle).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `rec` | [`Recommendation`](api.md#recommendation) | Recommendation row. |
| `wishlistUuid?` | `string` \| `null` | Optional wishlist uuid override. |

#### Returns

[`CatalogTitle`](#catalogtitle)

Unified catalog title.

***

### catalogTitleFromRequest()

```ts
function catalogTitleFromRequest(req): CatalogTitle;
```

Maps a wishlist title-request into a [CatalogTitle](#catalogtitle).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `req` | [`TitleRequest`](api.md#titlerequest) | Wishlist row. |

#### Returns

[`CatalogTitle`](#catalogtitle)

Unified catalog title (with `wishlist_uuid` set).

***

### defaultSortDirFor()

```ts
function defaultSortDirFor(sort): "desc" | "asc";
```

Default sort direction for a catalog sort key.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `sort` | [`CatalogSearchSort`](api.md#catalogsearchsort) | Catalog sort key. |

#### Returns

`"desc"` \| `"asc"`

`asc` for title/author/length; otherwise `desc`.

***

### descriptionLooksComplete()

```ts
function descriptionLooksComplete(raw): boolean;
```

Returns whether a blurb looks like a full store description, not a truncated teaser.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `raw` | `string` \| `null` \| `undefined` | Optional HTML/plain description. |

#### Returns

`boolean`

True when length and ending look complete.

***

### discoverSearchFromMeta()

```ts
function discoverSearchFromMeta(kind, value): object;
```

Composes a Discover catalog search + facet filter from a title-detail meta link.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `kind` | `"series"` \| `"authors"` \| `"narrators"` \| `"genres"` | Metadata link kind (authors / narrators / series / genres). |
| `value` | `string` | Linked label text. |

#### Returns

`object`

Search query, field, sort, and matching include filters.

| Name | Type |
| ------ | ------ |
| `authors?` | `string`[] |
| `field` | [`CatalogSearchField`](#catalogsearchfield) |
| `genres?` | `string`[] |
| `hideVirtualVoice` | `boolean` |
| `narrators?` | `string`[] |
| `q` | `string` |
| `series?` | `string`[] |
| `sort` | [`CatalogSearchSort`](api.md#catalogsearchsort) |

***

### formatPurchasePrices()

```ts
function formatPurchasePrices(hint): string | null;
```

Compact purchase price line; prefers dual member/list when both exist.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `hint` | \{ `list_price_label?`: `string` \| `null`; `member_price_label?`: `string` \| `null`; `price_label?`: `string` \| `null`; \} | Price label fields from a purchase hint. |
| `hint.list_price_label?` | `string` \| `null` | - |
| `hint.member_price_label?` | `string` \| `null` | - |
| `hint.price_label?` | `string` \| `null` | - |

#### Returns

`string` \| `null`

Display string, or `null` when no prices.

***

### formatSeriesLabel()

```ts
function formatSeriesLabel(series, seriesIndex?): string | null;
```

Formats series name + book number for list/card rows (`Sun Eater #3`).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `series` | `string` \| `null` \| `undefined` | Series name. |
| `seriesIndex?` | `string` \| `null` | Optional book number. |

#### Returns

`string` \| `null`

Label, or `null` when series is empty.

***

### formatShelfBestPrice()

```ts
function formatShelfBestPrice(hint): string | null;
```

Single best price for shelf cards (member / primary only — no dual list).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `hint` | \{ `list_price_label?`: `string` \| `null`; `member_price_label?`: `string` \| `null`; `price_label?`: `string` \| `null`; \} | Price label fields from a purchase hint. |
| `hint.list_price_label?` | `string` \| `null` | - |
| `hint.member_price_label?` | `string` \| `null` | - |
| `hint.price_label?` | `string` \| `null` | - |

#### Returns

`string` \| `null`

Display string, or `null` when no prices.

***

### googleFaviconUrl()

```ts
function googleFaviconUrl(domain): string;
```

Builds a Google S2 favicon URL for a domain.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `domain` | `string` | Hostname such as `audible.com`. |

#### Returns

`string`

Favicon image URL.

***

### pickBetterDescription()

```ts
function pickBetterDescription(a, b): string | null;
```

Prefers richer store/Audnexus blurbs over short plain catalog teasers.

HTML descriptions win over plain text; otherwise the longer string wins.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `a` | `string` \| `null` \| `undefined` | Current description. |
| `b` | `string` \| `null` \| `undefined` | Incoming description. |

#### Returns

`string` \| `null`

Preferred non-empty description, or `null`.

***

### preferCatalogLanguageOrder()

```ts
function preferCatalogLanguageOrder<T>(items, preferred?): T[];
```

Stable soft-sort: preferred language first, then unknown, then other.

#### Type Parameters

| Type Parameter |
| ------ |
| `T` *extends* `object` |

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `items` | `T`[] | Rows with optional `language`. |
| `preferred` | `string` | Preferred language code. |

#### Returns

`T`[]

New array sorted by [catalogLanguageRank](#cataloglanguagerank).

***

### preferredCatalogLanguage()

```ts
function preferredCatalogLanguage(): string;
```

Primary BCP-47 language tag from the browser (default `en`).

#### Returns

`string`

Two/three-letter language code.

***

### runtimeBucketBounds()

```ts
function runtimeBucketBounds(bucket): object;
```

Maps a runtime bucket to server min/max length filters.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bucket` | [`RuntimeBucket`](#runtimebucket) | Selected runtime bucket. |

#### Returns

`object`

Optional `min_length_minutes` / `max_length_minutes`.

| Name | Type |
| ------ | ------ |
| `max_length_minutes?` | `number` |
| `min_length_minutes?` | `number` |

***

### sortCatalogTitles()

```ts
function sortCatalogTitles(titles, sortKey): CatalogTitle[];
```

Client-sorts catalog titles for keys the server does not re-rank locally.

Relevance / popularity / rating keep input order (server ranking).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `titles` | [`CatalogTitle`](#catalogtitle)[] | Catalog titles. |
| `sortKey` | [`CatalogSearchSort`](api.md#catalogsearchsort) | Sort key. |

#### Returns

[`CatalogTitle`](#catalogtitle)[]

Sorted copy or original order for server-ranked keys.

***

### splitMetaList()

```ts
function splitMetaList(raw): string[];
```

Splits genre / category blobs (`Fantasy; Epic` / comma / pipe).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `raw` | `string` \| `null` \| `undefined` | Combined list string. |

#### Returns

`string`[]

Trimmed non-empty parts.

***

### storeFaviconUrl()

```ts
function storeFaviconUrl(source): string | undefined;
```

Favicon URL for a storefront id (`audible`, `libro`, …), if known.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `source` | `string` | Store id. |

#### Returns

`string` \| `undefined`

Favicon URL, or `undefined` when the domain is unknown.

***

### storeLabel()

```ts
function storeLabel(source): string;
```

Human-readable label for a storefront plugin id in Discover filters.

Falls back to the raw id when the store is unknown so future plugins still render.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `source` | `string` | Store id (`audible`, `libro`, …). |

#### Returns

`string`

Display name for UI chrome.

***

### titleNeedsMeta()

```ts
function titleNeedsMeta(title): boolean;
```

Returns whether a title still needs public metadata enrichment.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `title` | [`CatalogTitle`](#catalogtitle) | Catalog title. |

#### Returns

`boolean`

True when core fields (description, length, publisher, narrators, series index) are incomplete.

***

### titleNeedsShelfMeta()

```ts
function titleNeedsShelfMeta(title): boolean;
```

Returns whether a shelf card should pull title-meta (includes missing overall rating).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `title` | [`CatalogTitle`](#catalogtitle) | Catalog title. |

#### Returns

`boolean`

True when [titleNeedsMeta](#titleneedsmeta) or `rating_overall` is unknown.

***

### withAudibleEditionFromAsin()

```ts
function withAudibleEditionFromAsin<T>(row): T;
```

Adds an Audible store edition when an ASIN is known and no Audible row exists.

Used by Discover shelves, Wishlist enrichment, and the shared detail modal so
Where-to-buy does not depend on which surface fetched title-meta first.

#### Type Parameters

| Type Parameter |
| ------ |
| `T` *extends* `object` |

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `row` | `T` | Title or detail fields that carry editions. |

#### Returns

`T`

The same object, or a copy with an Audible edition appended.
