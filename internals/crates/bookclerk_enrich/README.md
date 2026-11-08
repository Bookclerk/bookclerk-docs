# bookclerk_enrich

Source-agnostic Audible metadata enrichment.

Matches library rows that lack an Audible ASIN using public Audible catalog
search and the deployed metadata-provider plugins (AudioBookshelf-style
confidence scoring). Intended for Libro.fm, Chirp, GraphicAudio, and any
future non-Audible sources.

A confident match stores the Audible ASIN and catalog fields on the row;
acquire then overlays the provider's chapter layout (and related tags) onto
plain audio that lacks Audible brand intro/outro.

| Item | Kind | Summary |
| --- | --- | --- |
| [`MetadataProviders`](#metadataproviders) | trait | Deployed metadata providers the host asks for bibliographic records. |
| [`Enrichment`](#enrichment) | struct | Metadata pulled from the Audible catalog and a metadata provider for a confident match. |
| [`ScoredMatch`](#scoredmatch) | struct | A scored Audible match candidate. |
| [`MatchQuery`](#matchquery) | struct | Owned-title (e.g. Libro) metadata used as the match query. |
| [`ScoreInput`](#scoreinput) | struct | Candidate metadata used for confidence scoring. |
| [`ChaptersAnswer`](#chaptersanswer) | struct | The first chapter layout any deployed provider had for one edition. |
| [`MetadataAnswer`](#metadataanswer) | struct | What every deployed provider answered for one lookup or search. |
| [`NoMetadataProviders`](#nometadataproviders) | struct | No providers: every answer is empty. For callers with no plugin host. |
| [`ProvidedChapters`](#providedchapters) | struct | A chapter layout and the provider plugin that returned it. |
| [`ProvidedRecord`](#providedrecord) | struct | One record and the provider plugin that returned it. |
| [`ProviderFailure`](#providerfailure) | struct | A provider call that failed, so its records are missing from the answer. |
| [`CatalogProduct`](#catalogproduct) | struct | One Audible catalog product (public search hit). |
| [`CatalogRating`](#catalograting) | struct | Community ratings from a single Audible catalog product (`response_groups=rating`). |
| [`CatalogReview`](#catalogreview) | struct | One customer review from Audible's public catalog reviews endpoint. |
| [`CatalogReviewsPage`](#catalogreviewspage) | struct | One page of helpful Audible customer reviews (public; no account). |
| [`EnrichError`](#enricherror) | enum | Errors from public-catalog enrichment and match scoring. |
| [`CatalogReviewsSort`](#catalogreviewssort) | enum | Audible catalog reviews sort (`MostHelpful` or `MostRecent`). |
| [`apply_enrichment_to_book`](#apply_enrichment_to_book) | function | Apply a confident Audible catalog match onto a non-Audible ownership row. |
| [`confidence_percent_to_fraction`](#confidence_percent_to_fraction) | function | Convert a 0–100 percent threshold to a 0.0–1.0 fraction. |
| [`enrich_books_from_audible`](#enrich_books_from_audible) | function | Enrich library rows that lack an ASIN via the public Audible catalog and the deployed metadata providers. |
| [`enrichment_for_asin`](#enrichment_for_asin) | function | Provider metadata for a known ASIN (no confidence scoring). |
| [`enrichment_from_record`](#enrichment_from_record) | function | Maps a provider record into enrichment fields. |
| [`lookup_by_metadata`](#lookup_by_metadata) | function | Look up the best Audible match for the given query metadata. |
| [`lookup_by_metadata_with_client`](#lookup_by_metadata_with_client) | function | Catalog search + provider lookup against an already-built HTTP client. |
| [`calculate_match_confidence`](#calculate_match_confidence) | function | Calculate match confidence in `[0.0, 1.0]`. |
| [`canonicalize_isbn`](#canonicalize_isbn) | function | Normalize then convert ISBN-10 → ISBN-13 (`978…`) when possible. |
| [`clean_author_for_compares`](#clean_author_for_compares) | function | Clean an author string for fuzzy compares (AudioBookshelf `cleanAuthorForCompares`). |
| [`clean_title_for_compares`](#clean_title_for_compares) | function | Clean a title for fuzzy compares (AudioBookshelf `cleanTitleForCompares`). |
| [`is_asin_title`](#is_asin_title) | function | True when a title field holds an Audible ASIN rather than a real title. |
| [`is_valid_asin`](#is_valid_asin) | function | True when `s` looks like an Audible ASIN (`^[A-Z0-9]{10}$`, case-insensitive). |
| [`isbn_exact_match`](#isbn_exact_match) | function | Returns true when both sides normalize to the same ISBN-13 (or matching ISBN-10). |
| [`levenshtein_distance`](#levenshtein_distance) | function | Levenshtein distance (case-insensitive by default, matching ABS). |
| [`levenshtein_similarity`](#levenshtein_similarity) | function | Similarity in `[0.0, 1.0]` from Levenshtein distance. |
| [`normalize_isbn`](#normalize_isbn) | function | Digits-only ISBN for exact matching (strips hyphens / spaces / `ISBN` prefix). |
| [`asin_identifier`](#asin_identifier) | function | An ASIN identifier scoped to `marketplace`. |
| [`chapter_info_from_layout`](#chapter_info_from_layout) | function | Rebuilds a provider's flat pre-order chapter list as the nested Audible `chapter_info` object the acquire pipeline reads. |
| [`isbn_identifier`](#isbn_identifier) | function | An ISBN identifier. |
| [`record_identifier`](#record_identifier) | function | The first value of `scheme` among a record's identifiers. |
| [`fetch_audible_catalog_rating`](#fetch_audible_catalog_rating) | function | Fetch Audible community ratings for one ASIN (public catalog; no account). |
| [`fetch_audible_catalog_reviews`](#fetch_audible_catalog_reviews) | function | Fetch a short list of helpful Audible customer reviews (public; no account). |
| [`fetch_audible_catalog_reviews_page`](#fetch_audible_catalog_reviews_page) | function | Paginated Audible catalog reviews (`page` is 1-based). |
| [`is_audible_podcast_product`](#is_audible_podcast_product) | function | True for Audible podcast shows / episodes (`content_type` / delivery type). |
| [`normalize_region`](#normalize_region) | function | Normalize a marketplace string to an Audible / Audnexus `region` value. |
| [`normalize_review_body`](#normalize_review_body) | function | Normalize review text for clients: decode HTML entities and preserve Audible guided questionnaire JSON (`[{ type, question, id, answer }, …]`) so the UI can render Q&A sections. Unrelated JSON arrays are left unchanged. |
| [`public_http_client`](#public_http_client) | function | Shared HTTP client for public metadata calls. |
| [`region_tld`](#region_tld) | function | Map marketplace / region code to Audible API TLD (AudioBookshelf `regionMap`). |
| [`resolve_genre_category_id`](#resolve_genre_category_id) | function | Map a genre display name to an Audible `category_id` under the Genres root. |
| [`search_catalog_asins`](#search_catalog_asins) | function | Search the public Audible catalog by title/author; returns ASINs (relevance order). |
| [`search_catalog_by_genre_name`](#search_catalog_by_genre_name) | function | Resolve an Audible Genres browse-node id for a category display name, then list products. |
| [`search_catalog_by_narrator`](#search_catalog_by_narrator) | function | Narrator-focused public catalog search. |
| [`search_catalog_by_series_asin`](#search_catalog_by_series_asin) | function | List products in an Audible series by parent series ASIN (public catalog). |
| [`search_catalog_by_series_name`](#search_catalog_by_series_name) | function | Keyword/title search, then keep products whose series name matches `series`. |
| [`search_catalog_keywords`](#search_catalog_keywords) | function | Keyword catalog search (e.g. ISBN) — useful when title search misses a hit. |
| [`search_catalog_products`](#search_catalog_products) | function | Public Audible catalog search returning product metadata (not just ASINs). |
| [`search_catalog_products_ex`](#search_catalog_products_ex) | function | Public catalog search with optional narrator, series ASIN, and/or category filters. |
| [`search_catalog_products_ex2`](#search_catalog_products_ex2) | function | Like `search_catalog_products_ex` with optional `category_id` and result cap. |
| [`search_catalog_products_paged`](#search_catalog_products_paged) | function | Paged public catalog search with explicit `products_sort_by` and optional rating. |
| [`search_catalog_products_rich`](#search_catalog_products_rich) | function | Same as `search_catalog_products` but requests list price / category ladders. |
| [`search_catalog_storefront`](#search_catalog_storefront) | function | Storefront catalog search (`GET /1.0/catalog/search`) — the same index the Storefront catalog search (`GET /1.0/catalog/search`) — the same index the Audible website uses. |
| [`Result`](#result) | type alias | Result alias for this crate's error type. |
| [`DEFAULT_ENRICH_MIN_CONFIDENCE`](#default_enrich_min_confidence) | constant | Default minimum confidence percent (AudioBookshelf-style 0–100 scale). |
| [`MAX_CHAPTER_DEPTH`](#max_chapter_depth) | constant | Deepest chapter nesting `chapter_info_from_layout` builds. A provider's layout is plugin output and the tree is later walked recursively, so a deeper `depth` is clamped to this level. Audible itself nests at most a few levels. |

## Re-exports

- [`bookclerk_library::decode_html_entities`](../bookclerk_storefront/text.md#decode_html_entities)

## Traits

### MetadataProviders

```rust
pub trait MetadataProviders: Send + Sync {
    fn lookup<'life0, 'life1, 'async_trait>(
        &'life0 self,
        params: &'life1 MetadataLookupParams,
    ) -> Pin<Box<dyn Future<Output = MetadataAnswer> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn search<'life0, 'life1, 'async_trait>(
        &'life0 self,
        params: &'life1 MetadataSearchParams,
    ) -> Pin<Box<dyn Future<Output = MetadataAnswer> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn chapters<'life0, 'life1, 'async_trait>(
        &'life0 self,
        params: &'life1 MetadataChaptersParams,
    ) -> Pin<Box<dyn Future<Output = ChaptersAnswer> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
}
```

Deployed metadata providers the host asks for bibliographic records.

Implementations ask every provider in a stable order. A provider that
fails is reported in the answer's `failures` and does not stop the others.

#### MetadataProviders::lookup

```rust
fn lookup<'life0, 'life1, 'async_trait>(
    &'life0 self,
    params: &'life1 MetadataLookupParams,
) -> Pin<Box<dyn Future<Output = MetadataAnswer> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Records for a set of identifiers.

##### Arguments

* `params` - Identifiers (never empty), marketplace and limit.

##### Returns

Every provider's records plus the providers that failed.

#### MetadataProviders::search

```rust
fn search<'life0, 'life1, 'async_trait>(
    &'life0 self,
    params: &'life1 MetadataSearchParams,
) -> Pin<Box<dyn Future<Output = MetadataAnswer> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Records matching a title and author.

##### Arguments

* `params` - Title (never empty), author, language, marketplace and limit.

##### Returns

Every provider's records plus the providers that failed.

#### MetadataProviders::chapters

```rust
fn chapters<'life0, 'life1, 'async_trait>(
    &'life0 self,
    params: &'life1 MetadataChaptersParams,
) -> Pin<Box<dyn Future<Output = ChaptersAnswer> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

The chapter layout of one edition.

##### Arguments

* `params` - Identifiers of the edition (never empty) and marketplace.

##### Returns

The first layout a provider found, plus the providers that failed
before it.

Implemented by: `NoMetadataProviders`

## Structs

### Enrichment

```rust
pub struct Enrichment {
    pub asin: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub series_asin: Option<String>,
    pub length_minutes: Option<i64>,
    pub publisher: Option<String>,
    pub subtitle: Option<String>,
    pub cover_url: Option<String>,
    pub isbn: Option<String>,
    pub published_at: Option<String>,
    pub categories: Option<String>,
    pub description: Option<String>,
    pub language: Option<String>,
    pub confidence: Option<f64>,
}
```

Metadata pulled from the Audible catalog and a metadata provider for a confident match.

**Fields**

- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `title`: Display title as shown on the storefront or library card.
- `authors`: Comma-separated author names when the storefront provides them.
- `narrators`: Comma-separated narrator names when known.
- `series`: Series name when the title belongs to a named series.
- `series_index`: Position within the series (e.g. `1`, `1.5`) when known.
- `series_asin`: Audible series ASIN when the catalog associates one.
- `length_minutes`: Audiobook runtime in whole minutes when known.
- `publisher`: Publisher imprint as reported by the catalog.
- `subtitle`: Optional subtitle when the catalog distinguishes it from the title.
- `cover_url`: HTTPS URL for cover art when the catalog exposes one.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `published_at`: ISO-8601 / RFC3339 timestamp when known (`releaseDate`).
- `categories`: ABS-style genre string (`;`-separated).
- `description`: Publisher or storefront synopsis; may be truncated for embeddings.
- `language`: BCP-47 / storefront language code (`en`, `de`, …) when known.
- `confidence`: Match confidence in `[0.0, 1.0]` (omit/`None` for legacy callers).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### ScoredMatch

```rust
pub struct ScoredMatch {
    pub enrichment: Enrichment,
    pub confidence: f64,
}
```

A scored Audible match candidate.

**Fields**

- `enrichment`: Filled metadata fields from the public catalog match.
- `confidence`: Match confidence in `[0.0, 1.0]`; higher means a stronger ASIN match.

Implements: `Clone`, `Debug`

### MatchQuery

```rust
pub struct MatchQuery<'a> {
    pub title: &'a str,
    pub subtitle: Option<&'a str>,
    pub author: Option<&'a str>,
    pub narrator: Option<&'a str>,
    pub isbn: Option<&'a str>,
    pub duration_minutes: Option<f64>,
}
```

Owned-title (e.g. Libro) metadata used as the match query.

**Fields**

- `title`: Display title as shown on the storefront or library card.
- `subtitle`: Optional subtitle when the catalog distinguishes it from the title.
- `author`: Primary author name used for fuzzy matching.
- `narrator`: Primary narrator name used for fuzzy matching.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `duration_minutes`: Candidate runtime in minutes for scoring against the seed.

Implements: `Clone`, `Debug`, `Default`

### ScoreInput

```rust
pub struct ScoreInput<'a> {
    pub title: &'a str,
    pub subtitle: Option<&'a str>,
    pub author: Option<&'a str>,
    pub narrator: Option<&'a str>,
    pub isbn: Option<&'a str>,
    pub duration_minutes: Option<f64>,
}
```

Candidate metadata used for confidence scoring.

**Fields**

- `title`: Display title as shown on the storefront or library card.
- `subtitle`: Optional subtitle when the catalog distinguishes it from the title.
- `author`: Primary author name used for fuzzy matching.
- `narrator`: Primary narrator name used for fuzzy matching.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `duration_minutes`: Runtime in minutes (Audible / Audnexus `runtimeLengthMin`).

Implements: `Clone`, `Debug`

### ChaptersAnswer

```rust
pub struct ChaptersAnswer {
    pub found: Option<ProvidedChapters>,
    pub failures: Vec<ProviderFailure>,
}
```

The first chapter layout any deployed provider had for one edition.

**Fields**

- `found`: The first `found` layout in provider order, if any.
- `failures`: Providers whose call failed before a layout was found.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### MetadataAnswer

```rust
pub struct MetadataAnswer {
    pub records: Vec<ProvidedRecord>,
    pub failures: Vec<ProviderFailure>,
}
```

What every deployed provider answered for one lookup or search.

**Fields**

- `records`: Records in provider order, each provider's own relevance order kept.
- `failures`: Providers whose call failed. A caller that caches "nothing found"
  should not do so while this is non-empty, since the missing provider
  may have had a record.

#### MetadataAnswer::incomplete

```rust
pub fn incomplete(&self) -> bool
```

True when at least one provider failed.

Implements: `Clone`, `Debug`, `Default`, `PartialEq`

### NoMetadataProviders

```rust
pub struct NoMetadataProviders;
```

No providers: every answer is empty. For callers with no plugin host.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `MetadataProviders`

### ProvidedChapters

```rust
pub struct ProvidedChapters {
    pub provider: String,
    pub chapters: MetadataChapters,
}
```

A chapter layout and the provider plugin that returned it.

**Fields**

- `provider`: PluginKey of the provider that answered.
- `chapters`: The layout; always `found`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ProvidedRecord

```rust
pub struct ProvidedRecord {
    pub provider: String,
    pub record: MetadataRecord,
}
```

One record and the provider plugin that returned it.

**Fields**

- `provider`: PluginKey of the provider that answered.
- `record`: The record as the provider described it.

Implements: `Clone`, `Debug`, `PartialEq`

### ProviderFailure

```rust
pub struct ProviderFailure {
    pub provider: String,
    pub message: String,
}
```

A provider call that failed, so its records are missing from the answer.

**Fields**

- `provider`: PluginKey of the provider that failed.
- `message`: Error text from the plugin call.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CatalogProduct

```rust
pub struct CatalogProduct {
    pub asin: String,
    pub title: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_asin: Option<String>,
    pub series_sequence: Option<String>,
    pub cover_url: Option<String>,
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
}
```

One Audible catalog product (public search hit).

**Fields**

- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `title`: Display title as shown on the storefront or library card.
- `authors`: Comma-separated author names when the storefront provides them.
- `narrators`: Comma-separated narrator names when known.
- `series`: Series name when the title belongs to a named series.
- `series_asin`: Parent series ASIN when the catalog returns series metadata.
- `series_sequence`: Series sequence / position when present (string form from Audible).
- `cover_url`: Cover URL from `product_images` when the `media` response group is present.
- `subtitle`: Optional subtitle when the catalog distinguishes it from the title.
- `description`: Publisher summary / blurb (`product_extended_attrs`).
- `publisher`: Publisher imprint as reported by the catalog.
- `length_minutes`: Audiobook runtime in whole minutes when known.
- `published_at`: Publication date string from the catalog (ISO or storefront format).
- `categories`: Genre / subject labels (`;`-separated).
- `language`: BCP-47 / storefront language code (`en`, `de`, …) when known.
- `price_cents`: List or deal price in integer cents of [`Self::currency`](#catalogproduct).
- `currency`: ISO 4217 currency code for price fields (`USD`, `EUR`, …).
- `price_label`: Human-readable price string from the storefront (e.g. `$14.95`).
- `rating_overall`: Overall community rating when the `rating` response group is present.
- `rating_count`: Count of star ratings (`rating.num_ratings`) when present.

Implements: `Clone`, `Debug`, `PartialEq`

### CatalogRating

```rust
pub struct CatalogRating {
    pub overall: Option<f64>,
    pub performance: Option<f64>,
    pub story: Option<f64>,
    pub num_ratings: Option<i64>,
    pub num_reviews: Option<i64>,
}
```

Community ratings from a single Audible catalog product (`response_groups=rating`).

Kept off bulk Discover catalog calls (see `CATALOG_RESPONSE_GROUPS`); use from
title-detail paths only.

**Fields**

- `overall`: Aggregate star rating across listeners.
- `performance`: Narration / performance sub-score.
- `story`: Story / content sub-score.
- `num_ratings`: Total number of ratings contributing to the averages.
- `num_reviews`: Total number of written reviews when distinct from ratings.

Implements: `Clone`, `Debug`, `PartialEq`

### CatalogReview

```rust
pub struct CatalogReview {
    pub id: Option<String>,
    pub title: Option<String>,
    pub body: String,
    pub author_name: Option<String>,
    pub overall_rating: Option<i64>,
    pub performance_rating: Option<i64>,
    pub story_rating: Option<i64>,
    pub submitted_at: Option<String>,
}
```

One customer review from Audible's public catalog reviews endpoint.

**Fields**

- `id`: Stable identifier for this item.
- `title`: Display title as shown on the storefront or library card.
- `body`: Full review text from the storefront.
- `author_name`: Display name of the review author on the storefront.
- `overall_rating`: Overall star rating for this review when present.
- `performance_rating`: Narration sub-rating for this review when present.
- `story_rating`: Story sub-rating for this review when present.
- `submitted_at`: When the review was submitted (storefront timestamp string).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CatalogReviewsPage

```rust
pub struct CatalogReviewsPage {
    pub reviews: Vec<CatalogReview>,
    pub page: u32,
    pub page_size: usize,
    pub has_more: bool,
    pub sort_by: CatalogReviewsSort,
}
```

One page of helpful Audible customer reviews (public; no account).

**Fields**

- `reviews`: Review rows on this page.
- `page`: 1-based page index for this reviews / list request.
- `page_size`: Maximum number of items returned in this page.
- `has_more`: When true, another page is available by incrementing [`Self::page`](#catalogreviewspage).
- `sort_by`: Remote sort key for reviews (`most_helpful`, `most_recent`, …).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### EnrichError

```rust
pub enum EnrichError {
    Sync(String),
    Io(Error),
    Library(LibraryError),
}
```

Errors from public-catalog enrichment and match scoring.

**Variants**

- `Sync`: Failure during a blocking / sync enrich step.
- `Io`: Underlying filesystem or stream I/O failure.
- `Library`: Error propagated from [`bookclerk_library`](../bookclerk_library/README.md).

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<LibraryError>`

### CatalogReviewsSort

```rust
pub enum CatalogReviewsSort {
    MostHelpful,
    MostRecent,
}
```

Audible catalog reviews sort (`MostHelpful` or `MostRecent`).

**Variants**

- `MostHelpful`: Sort reviews by helpfulness (storefront default).
- `MostRecent`: Sort reviews by newest submission first.

#### CatalogReviewsSort::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the wire / FourCC string form of this value.

#### CatalogReviewsSort::parse

```rust
pub fn parse(raw: &str) -> Self
```

Parses this value from its wire string form.

##### Arguments

* `raw` - String `raw` for this call.

##### Returns

Updated `Self` for chaining.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### apply_enrichment_to_book

```rust
pub async fn apply_enrichment_to_book(
    library: &LibraryStore,
    book_uuid: &str,
    enrichment: &Enrichment,
) -> Result<BookRecord>
```

Apply a confident Audible catalog match onto a non-Audible ownership row.

Preserves `source`, `product_id`, `uuid`, `account_id`, and an existing
`asin` (sets ASIN only when the row does not already have one). Catalog
fields win for display metadata (title, contributors, series, ISBN, genres,
publish date). Store-native `length_minutes` is kept when present — Audible
runtimes include brand intro/outro that plain Acquire files omit.

#### Arguments

* `library` - Open library store used for reads/writes.
* `book_uuid` - String `book_uuid` for this call.
* `enrichment` - Catalog fields to merge onto the book row.

#### Returns

On success, the inner `bookclerk_library::BookRecord` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### confidence_percent_to_fraction

```rust
pub fn confidence_percent_to_fraction(percent: u8) -> f64
```

Convert a 0–100 percent threshold to a 0.0–1.0 fraction.

#### Arguments

* `percent` - Confidence threshold expressed as 0–100.

#### Returns

`f64` result.

### enrich_books_from_audible

```rust
pub async fn enrich_books_from_audible(
    library: &LibraryStore,
    providers: &dyn MetadataProviders,
    min_confidence_percent: u8,
) -> Result<usize>
```

Enrich library rows that lack an ASIN via the public Audible catalog and
the deployed metadata providers.

Source-agnostic: any non-`audible` row with title and/or ISBN is considered.
Uses title, author, narrator, subtitle, ISBN, and duration when present.
`min_confidence_percent` is 0–100 (default [`DEFAULT_ENRICH_MIN_CONFIDENCE`](#default_enrich_min_confidence)).
With no provider that answers ASIN lookups, nothing matches.

#### Arguments

* `library` - Open library store used for reads/writes.
* `providers` - Deployed metadata providers asked for each candidate ASIN.
* `min_confidence_percent` - Minimum match confidence, 0–100.

#### Returns

On success, the inner `usize` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### enrichment_for_asin

```rust
pub async fn enrichment_for_asin(
    providers: &dyn MetadataProviders,
    asin: &str,
    region: &str,
) -> Option<Enrichment>
```

Provider metadata for a known ASIN (no confidence scoring).

Used by Discover / Wishlist / Library detail views to surface description,
runtime, publisher, and related fields without requiring a library row.
The first provider record that carries a title wins.

#### Arguments

* `providers` - Deployed metadata providers.
* `asin` - Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

The record's fields, or `None` when the ASIN is malformed or no provider
has a record. A provider failure is logged and treated as no record.

### enrichment_from_record

```rust
pub fn enrichment_from_record(record: &MetadataRecord, asin: &str) -> Option<Enrichment>
```

Maps a provider record into enrichment fields.

#### Arguments

* `record` - The provider's record.
* `asin` - ASIN asked for; used when the record does not name one.

#### Returns

`None` when the record has no title.

### lookup_by_metadata

```rust
pub async fn lookup_by_metadata(
    providers: &dyn MetadataProviders,
    query: &MatchQuery<'_>,
    region: &str,
    min_confidence: f64,
) -> Result<Option<ScoredMatch>>
```

Look up the best Audible match for the given query metadata.

Candidate ASINs come from the public Audible catalog search; each one's
record comes from the deployed metadata providers. Returns `None` when
no candidate meets `min_confidence` (0.0–1.0), or when no provider has a
record for any candidate.

#### Arguments

* `providers` - Deployed metadata providers asked for each candidate.
* `query` - Title, author, ISBN and runtime of the row to match.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `min_confidence` - Minimum match confidence in `[0.0, 1.0]` to apply enrichment.

#### Returns

The highest-scoring candidate at or above `min_confidence`.

#### Errors

Returns an error when the HTTP client cannot be built or a catalog search fails.

### lookup_by_metadata_with_client

```rust
pub async fn lookup_by_metadata_with_client(
    http: &Client,
    providers: &dyn MetadataProviders,
    query: &MatchQuery<'_>,
    region: &str,
    min_confidence: f64,
) -> Result<Option<ScoredMatch>>
```

Catalog search + provider lookup against an already-built HTTP client.

#### Arguments

* `http` - Client for the public Audible catalog search.
* `providers` - Deployed metadata providers asked for each candidate.
* `query` - Title, author, ISBN and runtime of the row to match.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `min_confidence` - Minimum match confidence in `[0.0, 1.0]` to apply enrichment.

#### Returns

The highest-scoring candidate at or above `min_confidence`.

#### Errors

Returns an error when a catalog search fails.

### calculate_match_confidence

```rust
pub fn calculate_match_confidence(book: &ScoreInput<'_>, query: &MatchQuery<'_>) -> f64
```

Calculate match confidence in `[0.0, 1.0]`.

Base score follows AudioBookshelf (duration / title / author). When Libro
(or other) query metadata is richer:
- **Narrator** (both sides present): small extra weight in the blend
- **Exact ISBN** (normalized digits): boost by closing
  `ISBN_MATCH_GAP_CLOSE` of the remaining gap to 1.0 (not a forced accept)

When the query title (without its subtitle) is an ASIN (see
[`is_asin_title`](#is_asin_title)), returns `1.0`: the lookup fetched the candidate by
that ASIN. A plain ten-letter title such as `Foundation` is not an ASIN
and is scored like any other title.

#### Arguments

* `book` - Library book row to update.
* `query` - Query vector or free-text search string.

#### Returns

`f64` result.

### canonicalize_isbn

```rust
pub fn canonicalize_isbn(raw: &str) -> String
```

Normalize then convert ISBN-10 → ISBN-13 (`978…`) when possible.

ISBN is **not** available from every storefront (Chirp / GraphicAudio / Audible
public catalog often omit it). When present, prefer this canonical form so
10- and 13-digit variants of the same book share one key.

#### Arguments

* `raw` - String `raw` for this call.

#### Returns

String result for this operation.

### clean_author_for_compares

```rust
pub fn clean_author_for_compares(author: &str) -> String
```

Clean an author string for fuzzy compares (AudioBookshelf `cleanAuthorForCompares`).

Decodes HTML entities, then cleans with
[`bookclerk_resolve::normalize::clean_author`](../bookclerk_resolve/normalize.md#clean_author).

#### Arguments

* `author` - Author name or comma-separated names.

#### Returns

The cleaned, lower-cased name.

### clean_title_for_compares

```rust
pub fn clean_title_for_compares(title: &str, keep_subtitle: bool) -> String
```

Clean a title for fuzzy compares (AudioBookshelf `cleanTitleForCompares`).

Decodes HTML entities (Libro.fm and others ship entity-encoded titles),
then cleans with [`bookclerk_resolve::normalize::clean_title`](../bookclerk_resolve/normalize.md#clean_title).

#### Arguments

* `title` - Display title.
* `keep_subtitle` - When true, retain subtitle text for soft compares.

#### Returns

The cleaned, lower-cased title.

### is_asin_title

```rust
pub fn is_asin_title(title: &str) -> bool
```

True when a title field holds an Audible ASIN rather than a real title.

[`is_valid_asin`](#is_valid_asin) accepts any ten ASCII alphanumerics, which every
ten-letter title (`Foundation`, `Persuasion`) also satisfies. A title is
only treated as an ASIN when it also has an ASIN's shape: `B0` followed by
eight letters or digits (Audible's own ids), or an ISBN-10 (nine digits
and a digit or `X` check character), which older numeric ASINs use.
Case and surrounding whitespace are ignored.

#### Arguments

* `title` - Title text from a match query, possibly an ASIN typed or
  stored in place of a title.

#### Returns

`true` when the title should be looked up and scored as an ASIN.

### is_valid_asin

```rust
pub fn is_valid_asin(s: &str) -> bool
```

True when `s` looks like an Audible ASIN (`^[A-Z0-9]{10}$`, case-insensitive).

Includes all-numeric catalog ids (e.g. `1094100765`); Audnexus accepts these
and older Audible storefront ASINs are often numeric.

#### Arguments

* `s` - Input string to validate or normalize.

#### Returns

`true` when the predicate holds.

### isbn_exact_match

```rust
pub fn isbn_exact_match(query_isbn: Option<&str>, book_isbn: Option<&str>) -> bool
```

Returns true when both sides normalize to the same ISBN-13 (or matching ISBN-10).

#### Arguments

* `query_isbn` - String `query_isbn` for this call.
* `book_isbn` - String `book_isbn` for this call.

#### Returns

`true` when the predicate holds.

### levenshtein_distance

```rust
pub fn levenshtein_distance(a: &str, b: &str) -> usize
```

Levenshtein distance (case-insensitive by default, matching ABS).

#### Arguments

* `a` - Left-hand vector.
* `b` - Right-hand vector.

#### Returns

`usize` result.

### levenshtein_similarity

```rust
pub fn levenshtein_similarity(a: &str, b: &str) -> f64
```

Similarity in `[0.0, 1.0]` from Levenshtein distance.

#### Arguments

* `a` - Left-hand vector.
* `b` - Right-hand vector.

#### Returns

`f64` result.

### normalize_isbn

```rust
pub fn normalize_isbn(raw: &str) -> String
```

Digits-only ISBN for exact matching (strips hyphens / spaces / `ISBN` prefix).

#### Arguments

* `raw` - String `raw` for this call.

#### Returns

String result for this operation.

### asin_identifier

```rust
pub fn asin_identifier(asin: &str, marketplace: &str) -> Identifier
```

An ASIN identifier scoped to `marketplace`.

#### Arguments

* `asin` - The ASIN as the library stores it.
* `marketplace` - Marketplace code (`us`, `uk`, …); blank means none.

### chapter_info_from_layout

```rust
pub fn chapter_info_from_layout(layout: &MetadataChapters) -> Value
```

Rebuilds a provider's flat pre-order chapter list as the nested Audible
`chapter_info` object the acquire pipeline reads.

Each node carries both the camelCase keys Audnexus uses and the
snake_case keys of Audible's own API, as the pipeline accepts either.
A chapter whose `depth` skips a level is attached to the deepest open
shallower chapter, so a malformed list still nests without losing a
chapter, chapters of equal depth stay siblings, and
nesting stops at [`MAX_CHAPTER_DEPTH`](#max_chapter_depth) levels.

#### Arguments

* `layout` - The provider's chapter layout.

#### Returns

An object with `chapters`, the brand intro/outro and runtime lengths when
known, and `isAccurate`.

### isbn_identifier

```rust
pub fn isbn_identifier(isbn: &str) -> Identifier
```

An ISBN identifier.

#### Arguments

* `isbn` - ISBN-10 or ISBN-13, already normalized by the caller.

### record_identifier

```rust
pub fn record_identifier(
    record: &MetadataRecord,
    scheme: IdentifierScheme,
) -> Option<&str>
```

The first value of `scheme` among a record's identifiers.

#### Arguments

* `record` - The provider record.
* `scheme` - The scheme wanted.

### fetch_audible_catalog_rating

```rust
pub async fn fetch_audible_catalog_rating(
    http: &Client,
    asin: &str,
    region: &str,
) -> Result<Option<CatalogRating>>
```

Fetch Audible community ratings for one ASIN (public catalog; no account).

#### Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

On success, the inner `Option<CatalogRating>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### fetch_audible_catalog_reviews

```rust
pub async fn fetch_audible_catalog_reviews(
    http: &Client,
    asin: &str,
    region: &str,
    limit: usize,
) -> Result<Vec<CatalogReview>>
```

Fetch a short list of helpful Audible customer reviews (public; no account).

Convenience wrapper around [`fetch_audible_catalog_reviews_page`](#fetch_audible_catalog_reviews_page) (page 1).
`limit` is clamped to `1..=50` (Audible's reviews endpoint max).

#### Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `limit` - Maximum number of results to return.

#### Returns

On success, the inner `Vec<CatalogReview>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### fetch_audible_catalog_reviews_page

```rust
pub async fn fetch_audible_catalog_reviews_page(
    http: &Client,
    asin: &str,
    region: &str,
    page: u32,
    page_size: usize,
    sort_by: CatalogReviewsSort,
) -> Result<CatalogReviewsPage>
```

Paginated Audible catalog reviews (`page` is 1-based).

#### Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `page` - Numeric `page` value for this call.
* `page_size` - Maximum items to return in this page.
* `sort_by` - `sort_by` input for this call.

#### Returns

On success, the inner `CatalogReviewsPage` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### is_audible_podcast_product

```rust
pub fn is_audible_podcast_product(p: &Value) -> bool
```

True for Audible podcast shows / episodes (`content_type` / delivery type).

#### Arguments

* `p` - `p` input for this call.

#### Returns

`true` when the predicate holds.

### normalize_region

```rust
pub fn normalize_region(region: &str) -> String
```

Normalize a marketplace string to an Audible / Audnexus `region` value.

#### Arguments

* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

String result for this operation.

### normalize_review_body

```rust
pub fn normalize_review_body(body: &str) -> String
```

Normalize review text for clients: decode HTML entities and preserve Audible
guided questionnaire JSON (`[{ type, question, id, answer }, …]`) so the UI
can render Q&A sections. Unrelated JSON arrays are left unchanged.

#### Arguments

* `body` - HTTP request body bytes.

#### Returns

String result for this operation.

### public_http_client

```rust
pub fn public_http_client() -> Result<Client>
```

Shared HTTP client for public metadata calls.

#### Returns

On success, the inner `Client` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### region_tld

```rust
pub fn region_tld(region: &str) -> &'static str
```

Map marketplace / region code to Audible API TLD (AudioBookshelf `regionMap`).

#### Arguments

* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

`&'static str` result.

### resolve_genre_category_id

```rust
pub async fn resolve_genre_category_id(
    http: &Client,
    region: &str,
    genre: &str,
) -> Result<Option<String>>
```

Map a genre display name to an Audible `category_id` under the Genres root.

Fetches `GET …/catalog/categories?categories_num_levels=3&root=Genres`, flattens
the tree, and picks the best case-insensitive name match (deeper nodes and
Science Fiction & Fantasy paths preferred; Children’s / Teen deprioritized).

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `genre` - String `genre` for this call.

#### Returns

On success, the inner `Option<String>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_asins

```rust
pub async fn search_catalog_asins(
    http: &Client,
    region: &str,
    title: &str,
    author: Option<&str>,
) -> Result<Vec<String>>
```

Search the public Audible catalog by title/author; returns ASINs (relevance order).

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `title` - Display title.
* `author` - String `author` for this call.

#### Returns

On success, the inner `Vec<String>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_by_genre_name

```rust
pub async fn search_catalog_by_genre_name(
    http: &Client,
    region: &str,
    genre: &str,
    page: u32,
    products_sort_by: &str,
    limit: usize,
    with_rating: bool,
) -> Result<Vec<CatalogProduct>>
```

Resolve an Audible Genres browse-node id for a category display name, then list products.

`page` is 1-based (Audible catalog convention), matching
[`search_catalog_products_paged`](#search_catalog_products_paged).

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `genre` - String `genre` for this call.
* `page` - Numeric `page` value for this call.
* `products_sort_by` - String `products_sort_by` for this call.
* `limit` - Maximum number of results to return.
* `with_rating` - Boolean flag `with_rating`.

#### Returns

On success, the inner `Vec<CatalogProduct>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_by_narrator

```rust
pub async fn search_catalog_by_narrator(
    http: &Client,
    region: &str,
    narrator: &str,
) -> Result<Vec<CatalogProduct>>
```

Narrator-focused public catalog search.

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `narrator` - String `narrator` for this call.

#### Returns

On success, the inner `Vec<CatalogProduct>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_by_series_asin

```rust
pub async fn search_catalog_by_series_asin(
    http: &Client,
    region: &str,
    series_asin: &str,
) -> Result<Vec<CatalogProduct>>
```

List products in an Audible series by parent series ASIN (public catalog).

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `series_asin` - String `series_asin` for this call.

#### Returns

On success, the inner `Vec<CatalogProduct>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_by_series_name

```rust
pub async fn search_catalog_by_series_name(
    http: &Client,
    region: &str,
    series: &str,
    limit: usize,
) -> Result<Vec<CatalogProduct>>
```

Keyword/title search, then keep products whose series name matches `series`.

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `series` - Optional series name.
* `limit` - Maximum number of results to return.

#### Returns

On success, the inner `Vec<CatalogProduct>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_keywords

```rust
pub async fn search_catalog_keywords(
    http: &Client,
    region: &str,
    keywords: &str,
) -> Result<Vec<String>>
```

Keyword catalog search (e.g. ISBN) — useful when title search misses a hit.

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `keywords` - String `keywords` for this call.

#### Returns

On success, the inner `Vec<String>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_products

```rust
pub async fn search_catalog_products(
    http: &Client,
    region: &str,
    title: &str,
    author: Option<&str>,
    keywords: Option<&str>,
) -> Result<Vec<CatalogProduct>>
```

Public Audible catalog search returning product metadata (not just ASINs).

Pass `title` and/or `author` and/or `keywords`. Empty title is allowed when
keywords or author alone are set.

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `title` - Display title.
* `author` - String `author` for this call.
* `keywords` - String `keywords` for this call.

#### Returns

On success, the inner `Vec<CatalogProduct>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_products_ex

```rust
pub async fn search_catalog_products_ex(
    http: &Client,
    region: &str,
    title: &str,
    author: Option<&str>,
    keywords: Option<&str>,
    narrator: Option<&str>,
    series_asin: Option<&str>,
    rich: bool,
) -> Result<Vec<CatalogProduct>>
```

Public catalog search with optional narrator, series ASIN, and/or category filters.

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `title` - Display title.
* `author` - String `author` for this call.
* `keywords` - String `keywords` for this call.
* `narrator` - String `narrator` for this call.
* `series_asin` - String `series_asin` for this call.
* `rich` - Boolean flag `rich`.

#### Returns

On success, the inner `Vec<CatalogProduct>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_products_ex2

```rust
pub async fn search_catalog_products_ex2(
    http: &Client,
    region: &str,
    title: &str,
    author: Option<&str>,
    keywords: Option<&str>,
    narrator: Option<&str>,
    series_asin: Option<&str>,
    category_id: Option<&str>,
    rich: bool,
    num_results: usize,
) -> Result<Vec<CatalogProduct>>
```

Like [`search_catalog_products_ex`](#search_catalog_products_ex) with optional `category_id` and result cap.

Note: Audible's public catalog ignores `language=` query params; prefer
local re-rank on [`CatalogProduct::language`](#catalogproduct) after fetch.

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `title` - Display title.
* `author` - String `author` for this call.
* `keywords` - String `keywords` for this call.
* `narrator` - String `narrator` for this call.
* `series_asin` - String `series_asin` for this call.
* `category_id` - String `category_id` for this call.
* `rich` - Boolean flag `rich`.
* `num_results` - Numeric `num_results` value for this call.

#### Returns

On success, the inner `Vec<CatalogProduct>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_products_paged

```rust
pub async fn search_catalog_products_paged(
    http: &Client,
    region: &str,
    title: &str,
    author: Option<&str>,
    keywords: Option<&str>,
    narrator: Option<&str>,
    series_asin: Option<&str>,
    category_id: Option<&str>,
    page: u32,
    products_sort_by: &str,
    rich: bool,
    num_results: usize,
    with_rating: bool,
) -> Result<Vec<CatalogProduct>>
```

Paged public catalog search with explicit `products_sort_by` and optional rating.

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `title` - Display title.
* `author` - String `author` for this call.
* `keywords` - String `keywords` for this call.
* `narrator` - String `narrator` for this call.
* `series_asin` - String `series_asin` for this call.
* `category_id` - String `category_id` for this call.
* `page` - Numeric `page` value for this call.
* `products_sort_by` - String `products_sort_by` for this call.
* `rich` - Boolean flag `rich`.
* `num_results` - Numeric `num_results` value for this call.
* `with_rating` - Boolean flag `with_rating`.

#### Returns

On success, the inner `Vec<CatalogProduct>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_products_rich

```rust
pub async fn search_catalog_products_rich(
    http: &Client,
    region: &str,
    title: &str,
    author: Option<&str>,
    keywords: Option<&str>,
) -> Result<Vec<CatalogProduct>>
```

Same as [`search_catalog_products`](#search_catalog_products) but requests list price / category ladders.

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `title` - Display title.
* `author` - String `author` for this call.
* `keywords` - String `keywords` for this call.

#### Returns

On success, the inner `Vec<CatalogProduct>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### search_catalog_storefront

```rust
pub async fn search_catalog_storefront(
    http: &Client,
    region: &str,
    keywords: &str,
    page: u32,
    sort: &str,
    rich: bool,
    num_results: usize,
    with_rating: bool,
) -> Result<Vec<CatalogProduct>>
```

Storefront catalog search (`GET /1.0/catalog/search`) — the same index the
Storefront catalog search (`GET /1.0/catalog/search`) — the same index the
Audible website uses.

Prefer this over [`search_catalog_products_paged`](#search_catalog_products_paged) for Discover keyword browse:
`/catalog/products?keywords=` frequently omits titles that still resolve via
`/catalog/products/{asin}` (e.g. English *A Game of Thrones* `B002UZZ93G`).

#### Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `keywords` - String `keywords` for this call.
* `page` - Numeric `page` value for this call.
* `sort` - Sort field for the merged result set.
* `rich` - Boolean flag `rich`.
* `num_results` - Numeric `num_results` value for this call.
* `with_rating` - Boolean flag `with_rating`.

#### Returns

On success, the inner `Vec<CatalogProduct>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, EnrichError>;
```

Result alias for this crate's error type.

## Constants

### DEFAULT_ENRICH_MIN_CONFIDENCE

```rust
pub const DEFAULT_ENRICH_MIN_CONFIDENCE: u8 = 90;
```

Default minimum confidence percent (AudioBookshelf-style 0–100 scale).

### MAX_CHAPTER_DEPTH

```rust
pub const MAX_CHAPTER_DEPTH: usize = 8;
```

Deepest chapter nesting [`chapter_info_from_layout`](#chapter_info_from_layout) builds. A provider's
layout is plugin output and the tree is later walked recursively, so a
deeper `depth` is clamped to this level. Audible itself nests at most a few
levels.
