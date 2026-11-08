# bookclerk_audible_catalog

Unauthenticated Audible catalog HTTP: keyword, author, narrator, series
and genre search, plus ratings and reviews, from the public
`api.audible{tld}` catalog. No Audible login is involved.

The Audible storefront guest uses it for Discover search, expand and
purchase hints; host enrichment (`bookclerk-enrich`) uses it to match
titles from other stores to an ASIN. It links only the plugin SDK's HTTP
client and `bookclerk-storefront`, so the guest does not pull in the host's
library stack.

| Item | Kind | Summary |
| --- | --- | --- |
| [`CatalogProduct`](#catalogproduct) | struct | One Audible catalog product (public search hit). |
| [`CatalogRating`](#catalograting) | struct | Community ratings from a single Audible catalog product (`response_groups=rating`). |
| [`CatalogReview`](#catalogreview) | struct | One customer review from Audible's public catalog reviews endpoint. |
| [`CatalogReviewsPage`](#catalogreviewspage) | struct | One page of helpful Audible customer reviews (public; no account). |
| [`CatalogError`](#catalogerror) | enum | Failures talking to the public Audible catalog. |
| [`CatalogReviewsSort`](#catalogreviewssort) | enum | Audible catalog reviews sort (`MostHelpful` or `MostRecent`). |
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
| [`is_valid_asin`](#is_valid_asin) | function | True when `s` (trimmed) has the shape of an Audible ASIN: ten ASCII letters or digits. |
| [`Result`](#result) | type alias | Result alias for catalog lookups. |

## Structs

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

### CatalogError

```rust
pub enum CatalogError {
    Http(String),
}
```

Failures talking to the public Audible catalog.

**Variants**

- `Http`: The HTTP client could not be built, the request failed, or the
  response did not decode; the message says which.

Implements: `Debug`, `Display`, `Error`

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

### is_valid_asin

```rust
pub fn is_valid_asin(s: &str) -> bool
```

True when `s` (trimmed) has the shape of an Audible ASIN: ten ASCII
letters or digits.

Any ten-letter word also passes; callers that must tell a title from an
ASIN need a stricter check (see `bookclerk_enrich::is_asin_title`).

#### Arguments

* `s` - Candidate ASIN, possibly with surrounding whitespace.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, CatalogError>;
```

Result alias for catalog lookups.
