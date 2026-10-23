**bookclerk_source > types**

# Module: types

## Contents

**Structs**

- [`CatalogHit`](#cataloghit) - Neutral catalog / candidate hit (no store crate types).
- [`CatalogSearchOpts`](#catalogsearchopts) - Options for `crate::ContentSource::search_catalog`.
- [`ConfigOptionValue`](#configoptionvalue) - One allowed value for a [`SourceConfigOption`](types.md#bookclerk_sourcetypessourceconfigoption).
- [`ExpandSeed`](#expandseed) - Seed for related / series / author expansion.
- [`FetchOptions`](#fetchoptions) - Options passed to `crate::ContentSource::fetch_title`.
- [`ImportCredentialsOptions`](#importcredentialsoptions) - Options for `crate::ContentSource::import_credentials`.
- [`LoginOptions`](#loginoptions) - Options for interactive / CLI login.
- [`PlainAudioPart`](#plainaudiopart) - One DRM-free audio part (chapter file or single book).
- [`PlainFetch`](#plainfetch) - DRM-free fetch result. Sources that use DRM decrypt inside the plugin and
- [`PurchaseHintOpts`](#purchasehintopts) - Options for `crate::ContentSource::purchase_hint`.
- [`ScanOptions`](#scanoptions) - Options for a library scan.
- [`ScanSummary`](#scansummary) - Summary of a scan run.
- [`SourceAccount`](#sourceaccount) - Account discovered or created by a content source.
- [`SourceConfigOption`](#sourceconfigoption) - One source-native config knob under `[sources.<id>]`.
- [`SourcePurchaseHint`](#sourcepurchasehint) - URL + optional live price for one storefront edition.

**Enums**

- [`CatalogSearchField`](#catalogsearchfield) - Optional role / facet for `crate::ContentSource::search_catalog`.
- [`CatalogSearchSort`](#catalogsearchsort) - Sort mode for `crate::ContentSource::search_catalog`.
- [`CatalogSortDir`](#catalogsortdir) - Ascending / descending for catalog search host re-rank.
- [`OAuthProgress`](#oauthprogress) - Progress events for interactive OAuth login (portal / CLI).

**Type Aliases**

- [`SourceFetch`](#sourcefetch) - Result of fetching a title for acquire (always clear media).

---

## bookclerk_source::types::CatalogHit

*Struct*

Neutral catalog / candidate hit (no store crate types).

Identity fields are required; bibliographic extras and list price are filled
when a storefront’s search/expand payload already includes them (avoids a
second Audnexus / purchase-hint round-trip for Discover detail).

**Fields:**
- `product_id: String` - Store-native product / title id (ASIN, ISBN, UUID, …).
- `title: String` - Display title for the work.
- `authors: Option<String>` - Author names (`;`-separated when multiple).
- `narrators: Option<String>` - Narrator names (`;`-separated when multiple).
- `series: Option<String>` - Series name when the title belongs to a series.
- `series_index: Option<String>` - Position within the series (e.g. `"1"`, `"1-5"`).
- `asin: Option<String>` - Amazon ASIN when known (may match `Self::product_id` for Audible).
- `isbn: Option<String>` - ISBN-10/13 when known.
- `url: Option<String>` - Canonical storefront product URL when available.
- `cover_url: Option<String>` - Public cover image URL when the storefront provides one.
- `origin: String` - How this was found (`related`, `series`, `author`, `search`, `top_deals`, …).
- `subtitle: Option<String>` - Subtitle / secondary title line when the storefront provides one.
- `description: Option<String>` - Marketing / synopsis text from the storefront.
- `publisher: Option<String>` - Publisher imprint name when known.
- `length_minutes: Option<i64>` - Runtime in whole minutes when the storefront reports duration.
- `published_at: Option<String>` - ISO-8601 / store release date string when known.
- `categories: Option<String>` - Genre / subject labels (`;`-separated).
- `language: Option<String>` - Content language (BCP-47 or storefront display name).
- `price_cents: Option<i64>` - List / deal price from the same catalog payload (optional).
- `currency: Option<String>` - ISO 4217 currency code for `Self::price_cents` when priced.
- `price_label: Option<String>` - Pre-formatted price string for UI (e.g. `$14.95`).
- `rating_overall: Option<f64>` - Community overall rating (0–5) when the storefront returned it.
- `rating_count: Option<i64>` - Number of star ratings backing `Self::rating_overall` when known.
- `is_abridged: Option<bool>` - `Some(true)` when the storefront marks the title abridged; `Some(false)`

**Methods:**

- `fn decode_html_entities(self: Self) -> Self` - Decode HTML entities in human-readable metadata fields.

**Trait Implementations:**

- **Default**
  - `fn default() -> CatalogHit`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogHit`
- **From**
  - `fn from(hit: abi::CatalogHit) -> Self`



## bookclerk_source::types::CatalogSearchField

*Enum*

Optional role / facet for `crate::ContentSource::search_catalog`.

Free-text Discover typeahead leaves this unset. Meta links (author, narrator,
series, genre) set it so storefronts that support scoped params (e.g. Audible
`author=` / `narrator=`) can return the right catalog slice.

**Variants:**
- `Author` - Scope search to titles by this author.
- `Narrator` - Scope search to titles narrated by this person.
- `Series` - Scope search to titles in this series.
- `Genre` - Scope search to this genre / category facet.

**Methods:**

- `fn from_wire(raw: &str) -> Option<Self>` - Parse a wire / query-string value (`author`, `narrators`, …).
- `fn as_wire(self: Self) -> &'static str` - Serialize to the portal / query-string wire id.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogSearchField) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSearchField`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_source::types::CatalogSearchOpts

*Struct*

Options for `crate::ContentSource::search_catalog`.

**Fields:**
- `query: String` - Free-text or facet search string.
- `region: String` - Marketplace / catalog region (`us`, `uk`, …); empty = plugin default.
- `limit: usize` - Maximum hits to return (`0` = plugin default).
- `page: u32` - 1-based page index for storefronts that support paging (default 1).
- `sort: CatalogSearchSort` - Preferred upstream / merge sort.
- `field: Option<CatalogSearchField>` - When set, prefer storefront-native search for this facet.
- `language: Option<String>` - Preferred content language (BCP-47 primary or display name); soft-prioritize.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CatalogSearchOpts`
- **From**
  - `fn from(params: abi::SearchCatalogParams) -> Self`
- **Default**
  - `fn default() -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_source::types::CatalogSearchSort

*Enum*

Sort mode for `crate::ContentSource::search_catalog`.

Storefronts that lack a matching upstream sort ignore this (caller merges).

**Variants:**
- `Relevance` - Storefront relevance ranking (default).
- `Popularity` - Audible `BestSellers` when available.
- `Rating` - Audible `AvgRating` when available.
- `Title` - Alphabetical by title (host or upstream).
- `Author` - Alphabetical by primary author (often host re-rank).
- `Price` - Catalog list/deal `price_cents` (host re-rank; sparse on some stores).
- `Length` - `length_minutes` (host re-rank).

**Methods:**

- `fn from_wire(raw: &str) -> Self` - Parse a wire / query-string sort id (unknown → `Self::Relevance`).
- `fn as_wire(self: Self) -> &'static str` - Serialize to the portal / query-string wire id.
- `fn audible_products_sort_by(self: Self) -> &'static str` - Audible `/1.0/catalog/products` `products_sort_by` value.
- `fn audible_catalog_search_sort(self: Self) -> &'static str` - Audible `/1.0/catalog/search` `sort` (`sort_option_id`) value.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogSearchSort) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSearchSort`
- **From**
  - `fn from(sort: abi::CatalogSort) -> Self`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> CatalogSearchSort`



## bookclerk_source::types::CatalogSortDir

*Enum*

Ascending / descending for catalog search host re-rank.

**Variants:**
- `Asc` - Ascending order.
- `Desc` - Descending order (default for popularity / rating / price).

**Methods:**

- `fn from_wire(raw: &str) -> Self` - Parse a wire / query-string direction (`asc` / `desc`; unknown → `Self::Desc`).
- `fn as_wire(self: Self) -> &'static str` - Serialize to the portal / query-string wire id.
- `fn default_for_sort(sort: CatalogSearchSort) -> Self` - Default direction when the client omits `sort_dir`.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogSortDir) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSortDir`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> CatalogSortDir`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_source::types::ConfigOptionValue

*Struct*

One allowed value for a [`SourceConfigOption`](types.md#bookclerk_sourcetypessourceconfigoption).

**Fields:**
- `id: &'static str` - Wire / TOML id (`high`, `m4b`, `web`, …).
- `label: &'static str` - Operator-facing display name for this choice in Settings / CLI help.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ConfigOptionValue) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ConfigOptionValue`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_source::types::ExpandSeed

*Struct*

Seed for related / series / author expansion.

**Fields:**
- `source: String` - Source id of the seed title (`chirp`, `audible`, …).
- `product_id: String` - Store-native product id of the owned / seed title.
- `title: String` - Display title used for related/series/author expansion queries.
- `authors: Option<String>` - Author names from the seed library row.
- `narrators: Option<String>` - Narrator names from the seed library row.
- `series: Option<String>` - Series name from the seed library row.
- `series_asin: Option<String>` - Parent Audible series ASIN when known (from library metadata).
- `asin: Option<String>` - Amazon ASIN when known for Audible-related expansion.
- `isbn: Option<String>` - ISBN when known for cross-store matching.
- `region: String` - Marketplace / catalog region (`us`, `uk`, …).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ExpandSeed`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_source::types::FetchOptions

*Struct*

Options passed to `crate::ContentSource::fetch_title`.

**Fields:**
- `download: crate::options::DownloadOptions` - Packaging / naming knobs for this acquire (from `[output]` + destination).
- `cache_dir: std::path::PathBuf` - Absolute directory for scratch downloads for this fetch.
- `files_dir: std::path::PathBuf` - Root files directory (`BOOKCLERK_FILES_DIR`). Used for CDM / Widevine path
- `max_cache_bytes: Option<u64>` - Optional on-disk budget for `cache_dir` (bytes). Sources should call
- `cancel: Option<std::sync::Arc<std::sync::atomic::AtomicBool>>` - Cooperative cancel for mid-fetch quota or a lost job fence.

**Methods:**

- `fn enforce_cache_budget(self: &Self) -> crate::error::Result<()>` - Fails when cancel is set or `cache_dir` exceeds `Self::max_cache_bytes`.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> FetchOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_source::types::ImportCredentialsOptions

*Struct*

Options for `crate::ContentSource::import_credentials`.

**Fields:**
- `libation_accounts: bool` - Treat input as classic Libation `AccountsSettings.json`.
- `mkb79: bool` - Import mkb79 / audible-cli legacy auth JSON.
- `label: Option<String>` - Destination display label / filename stem.
- `force: bool` - Overwrite an existing credential.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ImportCredentialsOptions`
- **Default**
  - `fn default() -> ImportCredentialsOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_source::types::LoginOptions

*Struct*

Options for interactive / CLI login.

**Fields:**
- `marketplace: String` - Target marketplace / region for login (`us`, `uk`, …); empty = plugin default.
- `label: Option<String>` - Optional nickname stored with the new account credentials.
- `email: Option<String>` - Email/password sources; ignored for OAuth.
- `password: Option<String>` - Email/password sources; ignored for OAuth.
- `force: bool` - When true, overwrite an existing credential blob for the same account.
- `callback_bind: Option<String>` - Optional OAuth callback bind (`host:port`) for portal / reverse-proxy use.
- `external: bool` - External / paste-redirect OAuth instead of a local callback server.
- `response_url: Option<String>` - Pre-supplied OAuth redirect URL (with `Self::external`).
- `show_qr: bool` - Emit a terminal QR for the authorize URL when the source supports it.
- `timeout_secs: Option<u64>` - Seconds to wait for OAuth callback capture (source-defined default when `None`).
- `extra: serde_json::Value` - Store-specific knobs (`audible_username`, `ascii_qr`, …). Guests may ignore unknowns.

**Trait Implementations:**

- **Default**
  - `fn default() -> LoginOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> LoginOptions`
- **From**
  - `fn from(params: abi::LoginParams) -> Self` - Guest view of host login params; the callback IPC / public base are



## bookclerk_source::types::OAuthProgress

*Enum*

Progress events for interactive OAuth login (portal / CLI).

**Variants:**
- `LoginUrl{ url: String, qr: Option<String> }` - Browser URL the operator should open (optional pre-rendered QR text).
- `CallbackListening{ addr: String }` - Local callback server is listening (SSH port-forward hint).
- `WaitingForCallback` - Waiting for the OAuth redirect / callback.
- `Completed{ account_id: String }` - Login finished; credentials are persisted for `account_id`.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> OAuthProgress`



## bookclerk_source::types::PlainAudioPart

*Struct*

One DRM-free audio part (chapter file or single book).

**Fields:**
- `path: std::path::PathBuf` - Absolute path to a clear (DRM-free) audio file on disk.
- `title: Option<String>` - Optional chapter / part title for packaging and metadata.
- `duration_ms: Option<u64>` - Duration in milliseconds when known.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> PlainAudioPart`
- **From**
  - `fn from(part: abi::PlainPart) -> Self`



## bookclerk_source::types::PlainFetch

*Struct*

DRM-free fetch result. Sources that use DRM decrypt inside the plugin and
return clear media here — the host never sees ciphertext or keys.

**Fields:**
- `parts: Vec<PlainAudioPart>` - Ordered clear audio parts (single file or per-chapter downloads).
- `m4b_path: Option<std::path::PathBuf>` - Pre-built M4B from the store / plugin when available.
- `cover_path: Option<std::path::PathBuf>` - Absolute path to a downloaded cover image when available.
- `chapters: Vec<(String, u64)>` - Chapter titles paired with start offsets in milliseconds.
- `pdf_url: Option<String>` - Companion PDF download URL when the store exposes one.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> PlainFetch`
- **From**
  - `fn from(plain: abi::PlainFetch) -> Self`



## bookclerk_source::types::PurchaseHintOpts

*Struct*

Options for `crate::ContentSource::purchase_hint`.

**Fields:**
- `product_id: Option<String>` - Preferred store-native product id when already known.
- `title: Option<String>` - Title used to disambiguate when `product_id` is absent.
- `authors: Option<String>` - Author names used to disambiguate catalog lookups.
- `asin: Option<String>` - Amazon ASIN used for Audible / cross-store hints.
- `isbn: Option<String>` - ISBN used for cross-store hints.
- `region: String` - Marketplace / catalog region (`us`, `uk`, …); empty = plugin default.
- `with_price: bool` - When true, resolve live price if the source can.

**Trait Implementations:**

- **Default**
  - `fn default() -> PurchaseHintOpts`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> PurchaseHintOpts`
- **From**
  - `fn from(params: abi::PurchaseHintParams) -> Self`



## bookclerk_source::types::ScanOptions

*Struct*

Options for a library scan.

**Fields:**
- `accounts: Vec<String>` - Limit to specific account nicknames / ids.
- `page_size: u32` - Upstream library page size (titles per HTTP page; default 50).
- `import_episodes: bool` - Import podcast episodes — consumed by plugins that support it.
- `import_plus_titles: bool` - Import catalog Plus / non-owned titles — consumed by plugins that support it.
- `cancel: Option<std::sync::Arc<std::sync::atomic::AtomicBool>>` - Cooperative cancel checked between sources and pages.

**Methods:**

- `fn is_cancelled(self: &Self) -> bool` - True when a job fence asked this scan to stop.

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> ScanOptions`



## bookclerk_source::types::ScanSummary

*Struct*

Summary of a scan run.

**Fields:**
- `accounts: usize` - Number of accounts successfully scanned in this run.
- `books_upserted: usize` - Library rows inserted or updated.
- `pages: u32` - Upstream library pages fetched across all accounts.
- `skipped_disabled: usize` - Accounts skipped because `crate::SourceAccount::scan_enabled` was false.
- `warnings: Vec<String>` - Plugin keys skipped because more than one instance is registered.

**Methods:**

- `fn merge(self: & mut Self, other: &Self)` - Merge another source's summary into this one.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> ScanSummary`
- **Default**
  - `fn default() -> ScanSummary`



## bookclerk_source::types::SourceAccount

*Struct*

Account discovered or created by a content source.

**Fields:**
- `account_id: String` - Stable account id stored in the library (often email or store user id).
- `source: String` - Canonical plugin id (`audible`, `libro`, …).
- `marketplace: String` - Store marketplace / region code (`us`, `uk`, …).
- `label: Option<String>` - Optional operator-facing nickname for CLI / UI selection.
- `scan_enabled: bool` - When false, scheduled / bare scans skip this account (explicit

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SourceAccount`
- **From**
  - `fn from(account: abi::SourceAccount) -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_source::types::SourceConfigOption

*Struct*

One source-native config knob under `[sources.<id>]`.

**Fields:**
- `key: &'static str` - TOML key (`bitrate`, `container`, `access`).
- `label: &'static str` - Operator-facing display name for this knob in Settings.
- `values: &'static [ConfigOptionValue]` - Closed set of allowed wire values for `Self::key`.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SourceConfigOption) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> SourceConfigOption`



## bookclerk_source::types::SourceFetch

*Type Alias*: `PlainFetch`

Result of fetching a title for acquire (always clear media).

Historically a dual Encrypted/Plain enum; DRM is plugin-owned now, so this
is an alias of [`PlainFetch`](types.md#bookclerk_sourcetypesplainfetch). Prefer `PlainFetch` in new code.



## bookclerk_source::types::SourcePurchaseHint

*Struct*

URL + optional live price for one storefront edition.

**Fields:**
- `product_id: String` - Store-native product id for the hinted edition.
- `title: Option<String>` - Display title when the storefront returns one with the hint.
- `url: Option<String>` - Deep link to buy / open the title on the storefront.
- `price_cents: Option<i64>` - Primary / best known sell price in minor units (prefer member when dual).
- `currency: Option<String>` - ISO 4217 currency code for the primary price.
- `price_label: Option<String>` - Pre-formatted primary price string for UI.
- `list_price_cents: Option<i64>` - Non-member / list / retail price when the storefront shows dual pricing.
- `list_price_label: Option<String>` - Pre-formatted non-member / list price for dual-price storefronts.
- `member_price_cents: Option<i64>` - Member / subscriber price when distinct from list.
- `member_price_label: Option<String>` - Pre-formatted member / subscriber price when distinct from list.

**Methods:**

- `fn decode_html_entities(self: Self) -> Self` - Decode HTML entities in human-readable fields (no-op when no `&`).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> SourcePurchaseHint`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SourcePurchaseHint`
- **From**
  - `fn from(hint: abi::PurchaseHint) -> Self`



