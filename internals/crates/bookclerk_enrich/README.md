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
| [`EnrichError`](#enricherror) | enum | Errors from public-catalog enrichment and match scoring. |
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
| [`isbn_exact_match`](#isbn_exact_match) | function | Returns true when both sides normalize to the same ISBN-13 (or matching ISBN-10). |
| [`levenshtein_distance`](#levenshtein_distance) | function | Levenshtein distance (case-insensitive by default, matching ABS). |
| [`levenshtein_similarity`](#levenshtein_similarity) | function | Similarity in `[0.0, 1.0]` from Levenshtein distance. |
| [`normalize_isbn`](#normalize_isbn) | function | Digits-only ISBN for exact matching (strips hyphens / spaces / `ISBN` prefix). |
| [`asin_identifier`](#asin_identifier) | function | An ASIN identifier scoped to `marketplace`. |
| [`chapter_info_from_layout`](#chapter_info_from_layout) | function | Rebuilds a provider's flat pre-order chapter list as the nested Audible `chapter_info` object the acquire pipeline reads. |
| [`isbn_identifier`](#isbn_identifier) | function | An ISBN identifier. |
| [`record_identifier`](#record_identifier) | function | The first value of `scheme` among a record's identifiers. |
| [`Result`](#result) | type alias | Result alias for this crate's error type. |
| [`DEFAULT_ENRICH_MIN_CONFIDENCE`](#default_enrich_min_confidence) | constant | Default minimum confidence percent (AudioBookshelf-style 0–100 scale). |
| [`MAX_CHAPTER_DEPTH`](#max_chapter_depth) | constant | Deepest chapter nesting `chapter_info_from_layout` builds. A provider's layout is plugin output and the tree is later walked recursively, so a deeper `depth` is clamped to this level. Audible itself nests at most a few levels. |

## Re-exports

- [`bookclerk_audible_catalog::fetch_audible_catalog_rating`](../bookclerk_audible_catalog/README.md#fetch_audible_catalog_rating)
- [`bookclerk_audible_catalog::fetch_audible_catalog_reviews`](../bookclerk_audible_catalog/README.md#fetch_audible_catalog_reviews)
- [`bookclerk_audible_catalog::fetch_audible_catalog_reviews_page`](../bookclerk_audible_catalog/README.md#fetch_audible_catalog_reviews_page)
- [`bookclerk_audible_catalog::is_audible_podcast_product`](../bookclerk_audible_catalog/README.md#is_audible_podcast_product)
- [`bookclerk_audible_catalog::normalize_region`](../bookclerk_audible_catalog/README.md#normalize_region)
- [`bookclerk_audible_catalog::normalize_review_body`](../bookclerk_audible_catalog/README.md#normalize_review_body)
- [`bookclerk_audible_catalog::public_http_client`](../bookclerk_audible_catalog/README.md#public_http_client)
- [`bookclerk_audible_catalog::region_tld`](../bookclerk_audible_catalog/README.md#region_tld)
- [`bookclerk_audible_catalog::resolve_genre_category_id`](../bookclerk_audible_catalog/README.md#resolve_genre_category_id)
- [`bookclerk_audible_catalog::search_catalog_asins`](../bookclerk_audible_catalog/README.md#search_catalog_asins)
- [`bookclerk_audible_catalog::search_catalog_by_genre_name`](../bookclerk_audible_catalog/README.md#search_catalog_by_genre_name)
- [`bookclerk_audible_catalog::search_catalog_by_narrator`](../bookclerk_audible_catalog/README.md#search_catalog_by_narrator)
- [`bookclerk_audible_catalog::search_catalog_by_series_asin`](../bookclerk_audible_catalog/README.md#search_catalog_by_series_asin)
- [`bookclerk_audible_catalog::search_catalog_by_series_name`](../bookclerk_audible_catalog/README.md#search_catalog_by_series_name)
- [`bookclerk_audible_catalog::search_catalog_keywords`](../bookclerk_audible_catalog/README.md#search_catalog_keywords)
- [`bookclerk_audible_catalog::search_catalog_products`](../bookclerk_audible_catalog/README.md#search_catalog_products)
- [`bookclerk_audible_catalog::search_catalog_products_ex`](../bookclerk_audible_catalog/README.md#search_catalog_products_ex)
- [`bookclerk_audible_catalog::search_catalog_products_ex2`](../bookclerk_audible_catalog/README.md#search_catalog_products_ex2)
- [`bookclerk_audible_catalog::search_catalog_products_paged`](../bookclerk_audible_catalog/README.md#search_catalog_products_paged)
- [`bookclerk_audible_catalog::search_catalog_products_rich`](../bookclerk_audible_catalog/README.md#search_catalog_products_rich)
- [`bookclerk_audible_catalog::search_catalog_storefront`](../bookclerk_audible_catalog/README.md#search_catalog_storefront)
- [`bookclerk_audible_catalog::CatalogError`](../bookclerk_audible_catalog/README.md#catalogerror)
- [`bookclerk_audible_catalog::CatalogProduct`](../bookclerk_audible_catalog/README.md#catalogproduct)
- [`bookclerk_audible_catalog::CatalogRating`](../bookclerk_audible_catalog/README.md#catalograting)
- [`bookclerk_audible_catalog::CatalogReview`](../bookclerk_audible_catalog/README.md#catalogreview)
- [`bookclerk_audible_catalog::CatalogReviewsPage`](../bookclerk_audible_catalog/README.md#catalogreviewspage)
- [`bookclerk_audible_catalog::CatalogReviewsSort`](../bookclerk_audible_catalog/README.md#catalogreviewssort)
- [`bookclerk_library::decode_html_entities`](../bookclerk_storefront/text.md#decode_html_entities)
- [`match_score::is_valid_asin`](../bookclerk_audible_catalog/README.md#is_valid_asin)

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

## Enums

### EnrichError

```rust
pub enum EnrichError {
    Sync(String),
    Io(Error),
    Catalog(CatalogError),
    Library(LibraryError),
}
```

Errors from public-catalog enrichment and match scoring.

**Variants**

- `Sync`: Failure during a blocking / sync enrich step.
- `Io`: Underlying filesystem or stream I/O failure.
- `Catalog`: The public Audible catalog request failed.
- `Library`: Error propagated from [`bookclerk_library`](../bookclerk_library/README.md).

Implements: `Debug`, `Display`, `Error`, `From<CatalogError>`, `From<Error>`, `From<LibraryError>`

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

[`is_valid_asin`](../bookclerk_audible_catalog/README.md#is_valid_asin) accepts any ten ASCII alphanumerics, which every
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
