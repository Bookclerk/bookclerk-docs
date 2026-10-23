# bookclerk_enrich

Source-agnostic Audible metadata enrichment.

Matches library rows that lack an Audible ASIN using public Audible catalog
search + Audnexus (AudioBookshelf-style confidence scoring). Intended for
Libro.fm, Chirp, GraphicAudio, and any future non-Audible sources.

A confident match stores the Audible ASIN and catalog fields on the row;
acquire then overlays Audible chapter trees (and related tags) onto plain
audio that lacks Audible brand intro/outro.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Enrichment`](#enrichment) | struct | Metadata pulled from Audible / Audnexus for a confident match. |
| [`ScoredMatch`](#scoredmatch) | struct | A scored Audible match candidate. |
| [`MatchQuery`](#matchquery) | struct | Owned-title (e.g. Libro) metadata used as the match query. |
| [`ScoreInput`](#scoreinput) | struct | Candidate metadata used for confidence scoring. |
| [`CatalogProduct`](#catalogproduct) | struct | One Audible catalog product (public search hit). |
| [`CatalogRating`](#catalograting) | struct | Community ratings from a single Audible catalog product (`response_groups=rating`). |
| [`CatalogReview`](#catalogreview) | struct | One customer review from Audible's public catalog reviews endpoint. |
| [`CatalogReviewsPage`](#catalogreviewspage) | struct | One page of helpful Audible customer reviews (public; no account). |
| [`EnrichError`](#enricherror) | enum | Errors from public-catalog enrichment and match scoring. |
| [`CatalogReviewsSort`](#catalogreviewssort) | enum | Audible catalog reviews sort (`MostHelpful` or `MostRecent`). |
| [`apply_enrichment_to_book`](#apply_enrichment_to_book) | function | Apply a confident Audible catalog match onto a non-Audible ownership row. |
| [`confidence_percent_to_fraction`](#confidence_percent_to_fraction) | function | Convert a 0–100 percent threshold to a 0.0–1.0 fraction. |
| [`enrich_books_from_audible`](#enrich_books_from_audible) | function | Enrich library rows that lack an ASIN via public Audible catalog + Audnexus. |
| [`enrichment_for_asin`](#enrichment_for_asin) | function | Fetch Audnexus metadata for a known ASIN (no confidence scoring). |
| [`enrichment_for_asin_with_client`](#enrichment_for_asin_with_client) | function | Same as `enrichment_for_asin` with a shared HTTP client. |
| [`lookup_by_metadata`](#lookup_by_metadata) | function | Look up the best Audible match for the given query metadata via public APIs. |
| [`lookup_by_metadata_with_client`](#lookup_by_metadata_with_client) | function | Catalog + Audnexus lookup against an already-built HTTP client. |
| [`calculate_match_confidence`](#calculate_match_confidence) | function | Calculate match confidence in `[0.0, 1.0]`. |
| [`canonicalize_isbn`](#canonicalize_isbn) | function | Normalize then convert ISBN-10 → ISBN-13 (`978…`) when possible. |
| [`clean_author_for_compares`](#clean_author_for_compares) | function | Clean an author string for fuzzy compares (AudioBookshelf `cleanAuthorForCompares`). |
| [`clean_title_for_compares`](#clean_title_for_compares) | function | Clean a title for fuzzy compares (AudioBookshelf `cleanTitleForCompares`). |
| [`is_valid_asin`](#is_valid_asin) | function | True when `s` looks like an Audible ASIN (`^[A-Z0-9]{10}$`, case-insensitive). |
| [`isbn_exact_match`](#isbn_exact_match) | function | Returns true when both sides normalize to the same ISBN-13 (or matching ISBN-10). |
| [`levenshtein_distance`](#levenshtein_distance) | function | Levenshtein distance (case-insensitive by default, matching ABS). |
| [`levenshtein_similarity`](#levenshtein_similarity) | function | Similarity in `[0.0, 1.0]` from Levenshtein distance. |
| [`normalize_isbn`](#normalize_isbn) | function | Digits-only ISBN for exact matching (strips hyphens / spaces / `ISBN` prefix). |
| [`fetch_audible_catalog_rating`](#fetch_audible_catalog_rating) | function | Fetch Audible community ratings for one ASIN (public catalog; no account). |
| [`fetch_audible_catalog_reviews`](#fetch_audible_catalog_reviews) | function | Fetch a short list of helpful Audible customer reviews (public; no account). |
| [`fetch_audible_catalog_reviews_page`](#fetch_audible_catalog_reviews_page) | function | Paginated Audible catalog reviews (`page` is 1-based). |
| [`fetch_audnexus_book`](#fetch_audnexus_book) | function | Fetch book metadata from Audnexus (no Audible account). |
| [`fetch_audnexus_chapters`](#fetch_audnexus_chapters) | function | Fetch chapter metadata from Audnexus and normalize to Audible `chapter_info` shape. |
| [`fetch_public_chapter_info`](#fetch_public_chapter_info) | function | Convenience: public chapter fetch with a fresh HTTP client. |
| [`is_audible_podcast_product`](#is_audible_podcast_product) | function | True for Audible podcast shows / episodes (`content_type` / delivery type). |
| [`normalize_region`](#normalize_region) | function | Normalize a marketplace string to an Audnexus `region` query value. |
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

## Re-exports

- [`bookclerk_library::decode_html_entities`](../bookclerk_library/index.md#decode_html_entities)

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

Metadata pulled from Audible / Audnexus for a confident match.

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
- `Library`: Error propagated from [`bookclerk_library`](../bookclerk_library/index.md).

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
    min_confidence_percent: u8,
) -> Result<usize>
```

Enrich library rows that lack an ASIN via public Audible catalog + Audnexus.

Source-agnostic: any non-`audible` row with title and/or ISBN is considered.
Uses title, author, narrator, subtitle, ISBN, and duration when present.
`min_confidence_percent` is 0–100 (default [`DEFAULT_ENRICH_MIN_CONFIDENCE`](#default_enrich_min_confidence)).

#### Arguments

* `library` - Open library store used for reads/writes.
* `min_confidence_percent` - `min_confidence_percent` input for this call.

#### Returns

On success, the inner `usize` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### enrichment_for_asin

```rust
pub async fn enrichment_for_asin(asin: &str, region: &str) -> Result<Option<Enrichment>>
```

Fetch Audnexus metadata for a known ASIN (no confidence scoring).

Used by Discover / Wishlist / Library detail views to surface description,
runtime, publisher, and related fields without requiring a library row.

#### Arguments

* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

On success, the inner `Option<Enrichment>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### enrichment_for_asin_with_client

```rust
pub async fn enrichment_for_asin_with_client(
    http: &Client,
    asin: &str,
    region: &str,
) -> Result<Option<Enrichment>>
```

Same as [`enrichment_for_asin`](#enrichment_for_asin) with a shared HTTP client.

#### Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

On success, the inner `Option<Enrichment>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### lookup_by_metadata

```rust
pub async fn lookup_by_metadata(
    query: &MatchQuery<'_>,
    region: &str,
    min_confidence: f64,
) -> Result<Option<ScoredMatch>>
```

Look up the best Audible match for the given query metadata via public APIs.

Returns `None` when no candidate meets `min_confidence` (0.0–1.0).

#### Arguments

* `query` - Query vector or free-text search string.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `min_confidence` - Minimum match confidence in `[0.0, 1.0]` to apply enrichment.

#### Returns

On success, the inner `Option<ScoredMatch>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### lookup_by_metadata_with_client

```rust
pub async fn lookup_by_metadata_with_client(
    http: &Client,
    query: &MatchQuery<'_>,
    region: &str,
    min_confidence: f64,
) -> Result<Option<ScoredMatch>>
```

Catalog + Audnexus lookup against an already-built HTTP client.

#### Arguments

* `http` - `http` input for this call.
* `query` - Query vector or free-text search string.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `min_confidence` - Minimum match confidence in `[0.0, 1.0]` to apply enrichment.

#### Returns

On success, the inner `Option<ScoredMatch>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

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

When the query title is an ASIN, returns `1.0`.

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

#### Arguments

* `author` - String `author` for this call.

#### Returns

String result for this operation.

### clean_title_for_compares

```rust
pub fn clean_title_for_compares(title: &str, keep_subtitle: bool) -> String
```

Clean a title for fuzzy compares (AudioBookshelf `cleanTitleForCompares`).

#### Arguments

* `title` - Display title.
* `keep_subtitle` - When true, retain subtitle text for soft compares.

#### Returns

String result for this operation.

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

### fetch_audnexus_book

```rust
pub async fn fetch_audnexus_book(
    http: &Client,
    asin: &str,
    region: &str,
) -> Result<Option<Value>>
```

Fetch book metadata from Audnexus (no Audible account).

#### Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

On success, the inner `Option<Value>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### fetch_audnexus_chapters

```rust
pub async fn fetch_audnexus_chapters(
    http: &Client,
    asin: &str,
    region: &str,
) -> Result<Option<Value>>
```

Fetch chapter metadata from Audnexus and normalize to Audible `chapter_info` shape.

Audnexus uses camelCase (`startOffsetMs`, `brandIntroDurationMs`). Callers that
already accept both casings can use the raw body; this helper also mirrors
snake_case keys for code that only reads Audible API field names.

#### Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

On success, the inner `Option<Value>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### fetch_public_chapter_info

```rust
pub async fn fetch_public_chapter_info(asin: &str, region: &str) -> Result<Option<Value>>
```

Convenience: public chapter fetch with a fresh HTTP client.

#### Arguments

* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

On success, the inner `Option<Value>` value.

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

Normalize a marketplace string to an Audnexus `region` query value.

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
