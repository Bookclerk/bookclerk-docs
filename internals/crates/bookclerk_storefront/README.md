# bookclerk_storefront

Types storefront guests share with the host.

## Audience

Native storefront guests (Libro.fm, Chirp, GraphicAudio, …) and the host
crates that talk to them. A guest reaches the library only through the
plugin ABI, so this crate carries the storefront DTOs, the conversions to
and from the typed ABI ([`abi`](abi.md)), the listing / record helpers and the
text normalization both sides apply, and nothing that opens a database.

It deliberately has no dependency on `bookclerk-library`,
`bookclerk-config`, an ORM, a SQL driver or OpenAPI, so a guest build
stays small. The host-only half (the `ContentSource` trait, the source
registry, `DownloadOptions`, `FetchOptions` and the
conversions to library rows) lives in `bookclerk-source`, which
re-exports the storefront DTOs, ABI conversions and listing / record
helpers. Host code gets the text and content-kind helpers from
`bookclerk-library`, which re-exports them from here.

Product narrative: `docs/plugins/README.md`. Style: `docs/contributing/code-documentation.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SourceBrand`](#sourcebrand) | struct | Visual identity for a content source in the connect portal. |
| [`CatalogHit`](#cataloghit) | struct | Neutral catalog / candidate hit (no store crate types). |
| [`CatalogSearchOpts`](#catalogsearchopts) | struct | Options for the host's `ContentSource::search_catalog`. |
| [`ConfigOptionValue`](#configoptionvalue) | struct | One allowed value for a `SourceConfigOption`. |
| [`ExpandSeed`](#expandseed) | struct | Seed for related / series / author expansion. |
| [`ImportCredentialsOptions`](#importcredentialsoptions) | struct | Options for the host's `ContentSource::import_credentials`. |
| [`LoginOptions`](#loginoptions) | struct | Options for interactive / CLI login. |
| [`PlainAudioPart`](#plainaudiopart) | struct | One DRM-free audio part (chapter file or single book). |
| [`PlainFetch`](#plainfetch) | struct | DRM-free fetch result. Sources that use DRM decrypt inside the plugin and return clear media here — the host never sees ciphertext or keys. |
| [`PurchaseHintOpts`](#purchasehintopts) | struct | Options for the host's `ContentSource::purchase_hint`. |
| [`ScanOptions`](#scanoptions) | struct | Options for a library scan. |
| [`ScanSummary`](#scansummary) | struct | Summary of a scan run. |
| [`SourceAccount`](#sourceaccount) | struct | Account discovered or created by a content source. |
| [`SourceConfigOption`](#sourceconfigoption) | struct | One source-native setting a storefront declares for its plugin instance. |
| [`SourcePurchaseHint`](#sourcepurchasehint) | struct | URL + optional live price for one storefront edition. |
| [`SourceError`](#sourceerror) | enum | Errors from content-source auth, scan, or fetch. |
| [`CatalogSearchField`](#catalogsearchfield) | enum | Optional role / facet for the host's `ContentSource::search_catalog`. |
| [`CatalogSearchSort`](#catalogsearchsort) | enum | Sort mode for the host's `ContentSource::search_catalog`. |
| [`CatalogSortDir`](#catalogsortdir) | enum | Ascending / descending for catalog search host re-rank. |
| [`OAuthProgress`](#oauthprogress) | enum | Progress events for interactive OAuth login (portal / CLI). |
| [`default_preferred_language`](#default_preferred_language) | function | Preferred language for Discover when the client omits one. |
| [`language_rank`](#language_rank) | function | Rank a hit's language relative to the preferred code. |
| [`normalize_language`](#normalize_language) | function | Normalize a language tag or display name to a primary ISO-ish code. |
| [`audio_extension`](#audio_extension) | function | Prefer byte sniffing, then `Content-Type`, then URL path; `default` when all fail. |
| [`extension_from_bytes`](#extension_from_bytes) | function | Guess an audio extension from magic bytes (`infer`), with AAC/MP3 fallbacks. |
| [`extension_from_content_type`](#extension_from_content_type) | function | Map a `Content-Type` header (parameters stripped) to an audio extension. |
| [`extension_from_url`](#extension_from_url) | function | Guess an audio file extension from a URL path (query/fragment stripped). |
| [`Result`](#result) | type alias | Result alias for content-source operations. |
| [`SourceFetch`](#sourcefetch) | type alias | Result of fetching a title for acquire (always clear media). |
| [`DEFAULT_CATALOG_SEARCH_LIMIT`](#default_catalog_search_limit) | constant | Hits a storefront returns from one catalog search when the caller leaves the limit at `0`. |

## Re-exports

- [`content_kind::is_episode`](content_kind.md#is_episode)
- [`content_kind::is_podcast_parent`](content_kind.md#is_podcast_parent)
- [`text::decode_html_entities`](text.md#decode_html_entities)
- [`text::decode_html_entities_cow`](text.md#decode_html_entities_cow)
- [`text::decode_html_entities_in_place`](text.md#decode_html_entities_in_place)
- [`text::decode_html_entities_opt`](text.md#decode_html_entities_opt)
- [`text::decode_html_entities_opt_in_place`](text.md#decode_html_entities_opt_in_place)
- [`text::str_maybe_html_entity`](text.md#str_maybe_html_entity)

## Modules

- [`abi`](abi.md): Conversions between host storefront types and the typed Cap'n Proto plugin ABI (`bookclerk_plugin_abi`).
- [`content_kind`](content_kind.md): Content-kind labels storefronts report for podcast shows and episodes.
- [`listing`](listing.md): Plugin ABI records as resolver listing drafts.
- [`records`](records.md): Builders and flat views for the structured bibliographic records that `ScanBook` and `CatalogHit` carry on the plugin ABI.
- [`text`](text.md): Text normalization for storefront / catalog metadata.

## Structs

### SourceBrand

```rust
pub struct SourceBrand {
    pub id: &'static str,
    pub name: &'static str,
    pub bg: &'static str,
    pub fg: &'static str,
    pub accent: &'static str,
    pub icon_url: &'static str,
}
```

Visual identity for a content source in the connect portal.

**Fields**

- `id`: Stable plugin id matching the host's `ContentSource::id` (`audible`, …).
- `name`: Human-facing store name shown on the connect button.
- `bg`: CSS background color (hex) for the portal button.
- `fg`: CSS foreground / text color (hex) for the portal button.
- `accent`: CSS accent color (hex) for focus / highlights.
- `icon_url`: Remote favicon / app-icon URL (not stored in-repo).

#### SourceBrand::logo_href

```rust
pub fn logo_href(&self) -> &'static str
```

Favicon / app-icon URL used as the portal button logo (`href`).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### CatalogHit

```rust
pub struct CatalogHit {
    pub product_id: String,
    pub title: String,
    pub url: Option<String>,
    pub cover_url: Option<String>,
    pub origin: String,
    pub subtitle: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub rating_overall: Option<f64>,
    pub rating_count: Option<i64>,
    pub is_abridged: Option<bool>,
    pub work_kind: WorkKind,
    pub expression_kind: ExpressionKind,
    pub media_kind: MediaKind,
    pub identifiers: Vec<Identifier>,
    pub credits: Vec<Credit>,
    pub series_entries: Vec<SeriesEntry>,
    pub parent_product_id: Option<String>,
    pub rank_hint: Option<u32>,
}
```

Neutral catalog / candidate hit (no store crate types).

Identity fields are required; bibliographic extras and list price are filled
when a storefront’s search/expand payload already includes them (avoids a
second Audnexus / purchase-hint round-trip for Discover detail).

**Fields**

- `product_id`: Store-native product / title id (ASIN, ISBN, UUID, …).
- `title`: Display title for the work.
- `url`: Canonical storefront product URL when available.
- `cover_url`: Public cover image URL when the storefront provides one.
- `origin`: How this was found (`related`, `series`, `author`, `search`, `top_deals`, …).
- `subtitle`: Subtitle / secondary title line when the storefront provides one.
- `description`: Marketing / synopsis text from the storefront.
- `publisher`: Publisher imprint name when known.
- `length_minutes`: Runtime in whole minutes when the storefront reports duration.
- `published_at`: ISO-8601 / store release date string when known.
- `categories`: Genre / subject labels (`;`-separated).
- `language`: Content language (BCP-47 or storefront display name).
- `price_cents`: List / deal price from the same catalog payload (optional).
- `currency`: ISO 4217 currency code for [`Self::price_cents`](#cataloghit) when priced.
- `price_label`: Pre-formatted price string for UI (e.g. `$14.95`).
- `rating_overall`: Community overall rating (0–5) when the storefront returned it.
- `rating_count`: Number of star ratings backing [`Self::rating_overall`](#cataloghit) when known.
- `is_abridged`: `Some(true)` when the storefront marks the title abridged; `Some(false)`
  for unabridged; `None` when unknown.
- `work_kind`: Kind of work the hit is.
- `expression_kind`: How the work is realised in this edition.
- `media_kind`: Form of this edition; print and ebook hits are never acquired.
- `identifiers`: External identifiers (ISBN, ASIN, OpenLibrary ids).
- `credits`: Credited contributors in billing order.
- `series_entries`: Series the work belongs to.
- `parent_product_id`: Product id of the parent podcast show, for a podcast episode.
- `rank_hint`: The storefront's own 1-based rank for a catalog search hit across all
  result pages (see [`CatalogSearchOpts::rank_hint`](#catalogsearchoptsrank_hint)); `None` when the
  store does not rank its results. Catalog search merges stores by it.

#### CatalogHit::decode_html_entities

```rust
pub fn decode_html_entities(self) -> Self
```

Decode HTML entities in human-readable metadata fields.

Storefronts (notably Libro.fm) sometimes return `Memory&#39;s Blade`-style
titles; normalize before discover merge / UI display.

No-op (no allocations) when no field contains `&`.

#### CatalogHit::authors

```rust
pub fn authors(&self) -> Option<String>
```

Author names joined for the flat view; `None` when none is credited.

#### CatalogHit::narrators

```rust
pub fn narrators(&self) -> Option<String>
```

Narrator names joined for the flat view; `None` when none is credited.

#### CatalogHit::asin

```rust
pub fn asin(&self) -> Option<String>
```

Amazon ASIN when the hit carries one.

#### CatalogHit::asin_in

```rust
pub fn asin_in(&self, marketplace: &str) -> Option<String>
```

ASIN for `marketplace`, falling back to an unscoped ASIN and then to
any ASIN the hit carries.

#### CatalogHit::isbn

```rust
pub fn isbn(&self) -> Option<String>
```

ISBN when the hit carries one.

#### CatalogHit::series

```rust
pub fn series(&self) -> Option<String>
```

Name of the first series the hit belongs to.

#### CatalogHit::series_index

```rust
pub fn series_index(&self) -> Option<String>
```

Position label in the first series the hit belongs to.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `From<CatalogHit>`, `Serialize`

### CatalogSearchOpts

```rust
pub struct CatalogSearchOpts {
    pub query: String,
    pub region: String,
    pub limit: usize,
    pub page: u32,
    pub sort: CatalogSearchSort,
    pub field: Option<CatalogSearchField>,
    pub language: Option<String>,
}
```

Options for the host's `ContentSource::search_catalog`.

**Fields**

- `query`: Free-text or facet search string.
- `region`: Marketplace / catalog region (`us`, `uk`, …); empty = plugin default.
- `limit`: Maximum hits to return; `0` means [`DEFAULT_CATALOG_SEARCH_LIMIT`](#default_catalog_search_limit)
  (read it through [`CatalogSearchOpts::effective_limit`](#catalogsearchoptseffective_limit)).
- `page`: 1-based page index for storefronts that support paging (default 1).
- `sort`: Preferred upstream / merge sort.
- `field`: When set, prefer storefront-native search for this facet.
- `language`: Preferred content language (BCP-47 primary or display name); soft-prioritize.

#### CatalogSearchOpts::effective_limit

```rust
pub fn effective_limit(&self) -> usize
```

The number of hits to return: [`CatalogSearchOpts::limit`](#catalogsearchopts), or
[`DEFAULT_CATALOG_SEARCH_LIMIT`](#default_catalog_search_limit) when the caller left it at `0`.

##### Returns

A hit count of at least 1.

#### CatalogSearchOpts::rank_hint

```rust
pub fn rank_hint(&self, index: usize) -> u32
```

1-based rank of the hit at `index` on this search's page, counted
across every page of [`Self::effective_limit`](#catalogsearchoptseffective_limit) hits (page 2 of 20-hit
pages starts at 21).

A storefront whose search order is meaningful for the requested sort
stores it in [`CatalogHit::rank_hint`](#cataloghit), so catalog search can merge
stores without naming any of them. Saturates at `u32::MAX`.

##### Arguments

* `index` - 0-based position of the hit within this page.

##### Returns

The rank, never 0 (0 means unranked on the wire).

#### CatalogSearchOpts::stamp_rank_hints

```rust
pub fn stamp_rank_hints(&self, hits: &mut [CatalogHit])
```

Stamps [`Self::rank_hint`](#catalogsearchoptsrank_hint) on each hit of one search page, in order.

##### Arguments

* `hits` - The page's hits in the storefront's own order.

Implements: `Clone`, `Debug`, `Default`, `From<&CatalogSearchOpts>`, `From<SearchCatalogParams>`

### ConfigOptionValue

```rust
pub struct ConfigOptionValue {
    pub id: &'static str,
    pub label: &'static str,
}
```

One allowed value for a [`SourceConfigOption`](#sourceconfigoption).

**Fields**

- `id`: Stored setting value (`high`, `m4b`, `web`, …).
- `label`: Operator-facing display name for this choice in Settings / CLI help.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ExpandSeed

```rust
pub struct ExpandSeed {
    pub source: String,
    pub product_id: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_asin: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub region: String,
}
```

Seed for related / series / author expansion.

**Fields**

- `source`: Source id of the seed title (`chirp`, `audible`, …).
- `product_id`: Store-native product id of the owned / seed title.
- `title`: Display title used for related/series/author expansion queries.
- `authors`: Author names from the seed library row.
- `narrators`: Narrator names from the seed library row.
- `series`: Series name from the seed library row.
- `series_asin`: Parent Audible series ASIN when known (from library metadata).
- `asin`: Amazon ASIN when known for Audible-related expansion.
- `isbn`: ISBN when known for cross-store matching.
- `region`: Marketplace / catalog region (`us`, `uk`, …).

Implements: `Clone`, `Debug`

### ImportCredentialsOptions

```rust
pub struct ImportCredentialsOptions {
    pub libation_accounts: bool,
    pub mkb79: bool,
    pub label: Option<String>,
    pub force: bool,
}
```

Options for the host's `ContentSource::import_credentials`.

**Fields**

- `libation_accounts`: Treat input as classic Libation `AccountsSettings.json`.
- `mkb79`: Import mkb79 / audible-cli legacy auth JSON.
- `label`: Destination display label / filename stem.
- `force`: Overwrite an existing credential.

Implements: `Clone`, `Debug`, `Default`

### LoginOptions

```rust
pub struct LoginOptions {
    pub marketplace: String,
    pub label: Option<String>,
    pub email: Option<String>,
    pub password: Option<String>,
    pub force: bool,
    pub callback_bind: Option<String>,
    pub external: bool,
    pub response_url: Option<String>,
    pub show_qr: bool,
    pub timeout_secs: Option<u64>,
    pub extra: Value,
}
```

Options for interactive / CLI login.

**Fields**

- `marketplace`: Target marketplace / region for login (`us`, `uk`, …); empty = plugin default.
- `label`: Optional nickname stored with the new account credentials.
- `email`: Email/password sources; ignored for OAuth.
- `password`: Email/password sources; ignored for OAuth.
- `force`: When true, overwrite an existing credential blob for the same account.
- `callback_bind`: Optional OAuth callback bind (`host:port`) for portal / reverse-proxy use.
- `external`: External / paste-redirect OAuth instead of a local callback server.
- `response_url`: Pre-supplied OAuth redirect URL (with [`Self::external`](#loginoptions)).
- `show_qr`: Emit a terminal QR for the authorize URL when the source supports it.
- `timeout_secs`: Seconds to wait for OAuth callback capture (source-defined default when `None`).
- `extra`: Store-specific knobs (`audible_username`, `ascii_qr`, …). Guests may ignore unknowns.

Implements: `Clone`, `Debug`, `Default`, `From<LoginParams>`

### PlainAudioPart

```rust
pub struct PlainAudioPart {
    pub path: PathBuf,
    pub title: Option<String>,
    pub duration_ms: Option<u64>,
}
```

One DRM-free audio part (chapter file or single book).

**Fields**

- `path`: Absolute path to a clear (DRM-free) audio file on disk.
- `title`: Optional chapter / part title for packaging and metadata.
- `duration_ms`: Duration in milliseconds when known.

Implements: `Clone`, `Debug`, `From<PlainAudioPart>`, `From<PlainPart>`

### PlainFetch

```rust
pub struct PlainFetch {
    pub parts: Vec<PlainAudioPart>,
    pub m4b_path: Option<PathBuf>,
    pub cover_path: Option<PathBuf>,
    pub chapters: Vec<(String, u64)>,
    pub pdf_url: Option<String>,
}
```

DRM-free fetch result. Sources that use DRM decrypt inside the plugin and
return clear media here — the host never sees ciphertext or keys.

**Fields**

- `parts`: Ordered clear audio parts (single file or per-chapter downloads).
- `m4b_path`: Pre-built M4B from the store / plugin when available.
- `cover_path`: Absolute path to a downloaded cover image when available.
- `chapters`: Chapter titles paired with start offsets in milliseconds.
- `pdf_url`: Companion PDF download URL when the store exposes one.

Implements: `Clone`, `Debug`, `From<PlainFetch>`

### PurchaseHintOpts

```rust
pub struct PurchaseHintOpts {
    pub product_id: Option<String>,
    pub title: Option<String>,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub region: String,
    pub with_price: bool,
}
```

Options for the host's `ContentSource::purchase_hint`.

**Fields**

- `product_id`: Preferred store-native product id when already known.
- `title`: Title used to disambiguate when `product_id` is absent.
- `authors`: Author names used to disambiguate catalog lookups.
- `asin`: Amazon ASIN used for Audible / cross-store hints.
- `isbn`: ISBN used for cross-store hints.
- `region`: Marketplace / catalog region (`us`, `uk`, …); empty = plugin default.
- `with_price`: When true, resolve live price if the source can.

Implements: `Clone`, `Debug`, `Default`, `From<&PurchaseHintOpts>`, `From<PurchaseHintParams>`

### ScanOptions

```rust
pub struct ScanOptions {
    pub accounts: Vec<String>,
    pub page_size: u32,
    pub import_episodes: bool,
    pub import_plus_titles: bool,
    pub cancel: Option<Arc<AtomicBool>>,
}
```

Options for a library scan.

**Fields**

- `accounts`: Limit to specific account nicknames / ids.
- `page_size`: Upstream library page size (titles per HTTP page; default 50).
- `import_episodes`: Import podcast episodes — consumed by plugins that support it.
- `import_plus_titles`: Import catalog Plus / non-owned titles — consumed by plugins that support it.
- `cancel`: Cooperative cancel checked between sources and pages.

#### ScanOptions::is_cancelled

```rust
pub fn is_cancelled(&self) -> bool
```

True when a job fence asked this scan to stop.

Implements: `Clone`, `Debug`, `Default`

### ScanSummary

```rust
pub struct ScanSummary {
    pub accounts: usize,
    pub books_upserted: usize,
    pub pages: u32,
    pub skipped_disabled: usize,
    pub removed: usize,
    pub warnings: Vec<String>,
}
```

Summary of a scan run.

**Fields**

- `accounts`: Number of accounts successfully scanned in this run.
- `books_upserted`: Library rows inserted or updated.
- `pages`: Upstream library pages fetched across all accounts.
- `skipped_disabled`: Accounts skipped because [`crate::SourceAccount::scan_enabled`](#sourceaccount) was false.
- `removed`: Titles the storefronts reported gone from their accounts (delta
  passes). Counted only; holdings are not removed yet.
- `warnings`: Plugin keys skipped because more than one instance is registered.
  
  Other sources in the same scan still run. Accounts stay keyed by
  storefront id, so those keys are not scanned twice.

#### ScanSummary::merge

```rust
pub fn merge(&mut self, other: &Self)
```

Merge another source's summary into this one.

Implements: `Clone`, `Debug`, `Default`

### SourceAccount

```rust
pub struct SourceAccount {
    pub account_id: String,
    pub source: String,
    pub marketplace: String,
    pub label: Option<String>,
    pub scan_enabled: bool,
}
```

Account discovered or created by a content source.

**Fields**

- `account_id`: Stable account id stored in the library (often email or store user id).
- `source`: Canonical plugin id (`audible`, `libro`, …).
- `marketplace`: Store marketplace / region code (`us`, `uk`, …).
- `label`: Optional operator-facing nickname for CLI / UI selection.
- `scan_enabled`: When false, scheduled / bare scans skip this account (explicit
  `--account` still includes it).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `From<SourceAccount>`, `Serialize`

### SourceConfigOption

```rust
pub struct SourceConfigOption {
    pub key: &'static str,
    pub label: &'static str,
    pub values: &'static [ConfigOptionValue],
}
```

One source-native setting a storefront declares for its plugin instance.

**Fields**

- `key`: Setting key in the plugin instance document (`bitrate`, `container`, `access`).
- `label`: Operator-facing display name for this knob in Settings.
- `values`: Closed set of allowed wire values for [`Self::key`](#sourceconfigoption).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### SourcePurchaseHint

```rust
pub struct SourcePurchaseHint {
    pub product_id: String,
    pub title: Option<String>,
    pub url: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub list_price_cents: Option<i64>,
    pub list_price_label: Option<String>,
    pub member_price_cents: Option<i64>,
    pub member_price_label: Option<String>,
}
```

URL + optional live price for one storefront edition.

**Fields**

- `product_id`: Store-native product id for the hinted edition.
- `title`: Display title when the storefront returns one with the hint.
- `url`: Deep link to buy / open the title on the storefront.
- `price_cents`: Primary / best known sell price in minor units (prefer member when dual).
- `currency`: ISO 4217 currency code for the primary price.
- `price_label`: Pre-formatted primary price string for UI.
- `list_price_cents`: Non-member / list / retail price when the storefront shows dual pricing.
- `list_price_label`: Pre-formatted non-member / list price for dual-price storefronts.
- `member_price_cents`: Member / subscriber price when distinct from list.
- `member_price_label`: Pre-formatted member / subscriber price when distinct from list.

#### SourcePurchaseHint::decode_html_entities

```rust
pub fn decode_html_entities(self) -> Self
```

Decode HTML entities in human-readable fields (no-op when no `&`).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `From<PurchaseHint>`, `From<SourcePurchaseHint>`, `Serialize`

## Enums

### SourceError

```rust
pub enum SourceError {
    NoAccounts(String),
    Auth(String),
    Api(String),
    RateLimited(Option<Duration>),
    Io(Error),
    Library(Box<dyn Error + Send + Sync + 'static>),
    Other(Error),
}
```

Errors from content-source auth, scan, or fetch.

**Variants**

- `NoAccounts`: No accounts are configured for this source (safe to skip in multi-source scan).
- `Auth`: Authentication or credential failure (operator-facing message).
- `Api`: Upstream store HTTP / protocol failure (operator-facing message).
- `RateLimited`: The store asked to be left alone for a while (HTTP 429 or its
  equivalent); the duration is its `Retry-After`, when it sent one.
- `Io`: Local filesystem failure while caching or writing downloads.
- `Library`: Library database / secret-store failure on the host.
  
  Guests never open the library; the host converts its store errors
  into this variant (`bookclerk-library` implements the `From`).
- `Other`: Catch-all for plugin-internal failures wrapped as [`anyhow::Error`](https://docs.rs/anyhow/latest/anyhow/struct.Error.html).

#### SourceError::from_rate_limit

```rust
pub fn from_rate_limit(status: u16, retry_after: Option<&str>) -> Option<Self>
```

[`Self::RateLimited`](#sourceerror) from an HTTP status and `Retry-After` header,
when the status is 429; otherwise `None`.

##### Arguments

* `status` - The response status code.
* `retry_after` - The `Retry-After` header value: whole seconds, as
  stores send it (an HTTP date is ignored).

#### SourceError::no_accounts

```rust
pub fn no_accounts(msg: impl Into<String>) -> Self
```

Construct [`Self::NoAccounts`](#sourceerror) with an operator-facing message.

#### SourceError::auth

```rust
pub fn auth(msg: impl Into<String>) -> Self
```

Construct [`Self::Auth`](#sourceerror) with an operator-facing message.

#### SourceError::api

```rust
pub fn api(msg: impl Into<String>) -> Self
```

Construct [`Self::Api`](#sourceerror) with an operator-facing message.

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<SourceError>`

### CatalogSearchField

```rust
pub enum CatalogSearchField {
    Author,
    Narrator,
    Series,
    Genre,
}
```

Optional role / facet for the host's `ContentSource::search_catalog`.

Free-text Discover typeahead leaves this unset. Meta links (author, narrator,
series, genre) set it so storefronts that support scoped params (e.g. Audible
`author=` / `narrator=`) can return the right catalog slice.

**Variants**

- `Author`: Scope search to titles by this author.
- `Narrator`: Scope search to titles narrated by this person.
- `Series`: Scope search to titles in this series.
- `Genre`: Scope search to this genre / category facet.

#### CatalogSearchField::from_wire

```rust
pub fn from_wire(raw: &str) -> Option<Self>
```

Parse a wire / query-string value (`author`, `narrators`, …).

#### CatalogSearchField::as_wire

```rust
pub fn as_wire(self) -> &'static str
```

Serialize to the portal / query-string wire id.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CatalogSearchSort

```rust
pub enum CatalogSearchSort {
    Relevance,
    Popularity,
    Rating,
    Title,
    Author,
    Price,
    Length,
}
```

Sort mode for the host's `ContentSource::search_catalog`.

Storefronts that lack a matching upstream sort ignore this (caller merges).

**Variants**

- `Relevance`: Storefront relevance ranking (default).
- `Popularity`: Audible `BestSellers` when available.
- `Rating`: Audible `AvgRating` when available.
- `Title`: Alphabetical by title (host or upstream).
- `Author`: Alphabetical by primary author (often host re-rank).
- `Price`: Catalog list/deal `price_cents` (host re-rank; sparse on some stores).
- `Length`: `length_minutes` (host re-rank).

#### CatalogSearchSort::from_wire

```rust
pub fn from_wire(raw: &str) -> Self
```

Parse a wire / query-string sort id (unknown → [`Self::Relevance`](#catalogsearchsort)).

#### CatalogSearchSort::as_wire

```rust
pub fn as_wire(self) -> &'static str
```

Serialize to the portal / query-string wire id.

#### CatalogSearchSort::audible_products_sort_by

```rust
pub fn audible_products_sort_by(self) -> &'static str
```

Audible `/1.0/catalog/products` `products_sort_by` value.

#### CatalogSearchSort::audible_catalog_search_sort

```rust
pub fn audible_catalog_search_sort(self) -> &'static str
```

Audible `/1.0/catalog/search` `sort` (`sort_option_id`) value.

Prefer this endpoint for Discover keyword browse — `/catalog/products`
keyword/title filters often omit major titles that still resolve by ASIN.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<CatalogSearchSort>`, `From<CatalogSort>`, `PartialEq`, `Serialize`

### CatalogSortDir

```rust
pub enum CatalogSortDir {
    Asc,
    Desc,
}
```

Ascending / descending for catalog search host re-rank.

**Variants**

- `Asc`: Ascending order.
- `Desc`: Descending order (default for popularity / rating / price).

#### CatalogSortDir::from_wire

```rust
pub fn from_wire(raw: &str) -> Self
```

Parse a wire / query-string direction (`asc` / `desc`; unknown → [`Self::Desc`](#catalogsortdir)).

#### CatalogSortDir::as_wire

```rust
pub fn as_wire(self) -> &'static str
```

Serialize to the portal / query-string wire id.

#### CatalogSortDir::default_for_sort

```rust
pub fn default_for_sort(sort: CatalogSearchSort) -> Self
```

Default direction when the client omits `sort_dir`.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OAuthProgress

```rust
pub enum OAuthProgress {
    LoginUrl { url: String, qr: Option<String> },
    CallbackListening { addr: String },
    WaitingForCallback,
    Completed { account_id: String },
}
```

Progress events for interactive OAuth login (portal / CLI).

**Variants**

- `LoginUrl`: Browser URL the operator should open (optional pre-rendered QR text).
  - `url`: Absolute HTTPS URL for the browser / QR payload.
  - `qr`: Optional pre-rendered QR text for terminal UIs.
- `CallbackListening`: Local callback server is listening (SSH port-forward hint).
  - `addr`: Listen address (host:port) of the local redirect receiver.
- `WaitingForCallback`: Waiting for the OAuth redirect / callback.
- `Completed`: Login finished; credentials are persisted for `account_id`.
  - `account_id`: Account id stored after a successful login.

Implements: `Clone`, `Debug`

## Functions

### default_preferred_language

```rust
pub fn default_preferred_language() -> &'static str
```

Preferred language for Discover when the client omits one.

### language_rank

```rust
pub fn language_rank(hit_language: Option<&str>, preferred: &str) -> u8
```

Rank a hit's language relative to the preferred code.

- `0` — matches preferred
- `1` — unknown / missing
- `2` — other language

### normalize_language

```rust
pub fn normalize_language(raw: &str) -> Option<String>
```

Normalize a language tag or display name to a primary ISO-ish code.

Accepts BCP-47 (`en-US`), bare codes (`en`), and common Audible display
names (`english`). Returns `None` for empty / unrecognized input.

### audio_extension

```rust
pub fn audio_extension(
    url: &str,
    bytes: Option<&[u8]>,
    content_type: Option<&str>,
    default: &'static str,
) -> &'static str
```

Prefer byte sniffing, then `Content-Type`, then URL path; `default` when all fail.

### extension_from_bytes

```rust
pub fn extension_from_bytes(bytes: &[u8]) -> Option<&'static str>
```

Guess an audio extension from magic bytes (`infer`), with AAC/MP3 fallbacks.

### extension_from_content_type

```rust
pub fn extension_from_content_type(content_type: &str) -> Option<&'static str>
```

Map a `Content-Type` header (parameters stripped) to an audio extension.

### extension_from_url

```rust
pub fn extension_from_url(url: &str) -> Option<&'static str>
```

Guess an audio file extension from a URL path (query/fragment stripped).

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, SourceError>;
```

Result alias for content-source operations.

### SourceFetch

```rust
pub type SourceFetch = PlainFetch;
```

Result of fetching a title for acquire (always clear media).

Historically a dual Encrypted/Plain enum; DRM is plugin-owned now, so this
is an alias of [`PlainFetch`](#plainfetch). Prefer `PlainFetch` in new code.

## Constants

### DEFAULT_CATALOG_SEARCH_LIMIT

```rust
pub const DEFAULT_CATALOG_SEARCH_LIMIT: usize = 20;
```

Hits a storefront returns from one catalog search when the caller leaves
the limit at `0`.
