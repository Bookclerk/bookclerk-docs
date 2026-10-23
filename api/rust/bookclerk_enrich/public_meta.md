**bookclerk_enrich > public_meta**

# Module: public_meta

## Contents

**Structs**

- [`CatalogProduct`](#catalogproduct) - One Audible catalog product (public search hit).
- [`CatalogRating`](#catalograting) - Community ratings from a single Audible catalog product (`response_groups=rating`).
- [`CatalogReview`](#catalogreview) - One customer review from Audible's public catalog reviews endpoint.
- [`CatalogReviewsPage`](#catalogreviewspage) - One page of helpful Audible customer reviews (public; no account).

**Enums**

- [`CatalogReviewsSort`](#catalogreviewssort) - Audible catalog reviews sort (`MostHelpful` or `MostRecent`).

**Functions**

- [`fetch_audible_catalog_rating`](#fetch_audible_catalog_rating) - Fetch Audible community ratings for one ASIN (public catalog; no account).
- [`fetch_audible_catalog_reviews`](#fetch_audible_catalog_reviews) - Fetch a short list of helpful Audible customer reviews (public; no account).
- [`fetch_audible_catalog_reviews_page`](#fetch_audible_catalog_reviews_page) - Paginated Audible catalog reviews (`page` is 1-based).
- [`fetch_audnexus_book`](#fetch_audnexus_book) - Fetch book metadata from Audnexus (no Audible account).
- [`fetch_audnexus_chapters`](#fetch_audnexus_chapters) - Fetch chapter metadata from Audnexus and normalize to Audible `chapter_info` shape.
- [`fetch_public_chapter_info`](#fetch_public_chapter_info) - Convenience: public chapter fetch with a fresh HTTP client.
- [`is_audible_podcast_product`](#is_audible_podcast_product) - True for Audible podcast shows / episodes (`content_type` / delivery type).
- [`normalize_region`](#normalize_region) - Normalize a marketplace string to an Audnexus `region` query value.
- [`normalize_review_body`](#normalize_review_body) - Normalize review text for clients: decode HTML entities and preserve Audible
- [`public_http_client`](#public_http_client) - Shared HTTP client for public metadata calls.
- [`region_tld`](#region_tld) - Map marketplace / region code to Audible API TLD (AudioBookshelf `regionMap`).
- [`resolve_genre_category_id`](#resolve_genre_category_id) - Map a genre display name to an Audible `category_id` under the Genres root.
- [`search_catalog_asins`](#search_catalog_asins) - Search the public Audible catalog by title/author; returns ASINs (relevance order).
- [`search_catalog_by_genre_name`](#search_catalog_by_genre_name) - Resolve an Audible Genres browse-node id for a category display name, then list products.
- [`search_catalog_by_narrator`](#search_catalog_by_narrator) - Narrator-focused public catalog search.
- [`search_catalog_by_series_asin`](#search_catalog_by_series_asin) - List products in an Audible series by parent series ASIN (public catalog).
- [`search_catalog_by_series_name`](#search_catalog_by_series_name) - Keyword/title search, then keep products whose series name matches `series`.
- [`search_catalog_keywords`](#search_catalog_keywords) - Keyword catalog search (e.g. ISBN) — useful when title search misses a hit.
- [`search_catalog_products`](#search_catalog_products) - Public Audible catalog search returning product metadata (not just ASINs).
- [`search_catalog_products_ex`](#search_catalog_products_ex) - Public catalog search with optional narrator, series ASIN, and/or category filters.
- [`search_catalog_products_ex2`](#search_catalog_products_ex2) - Like [`search_catalog_products_ex`](public_meta.md#bookclerk_enrichpublic_metasearch_catalog_products_ex) with optional `category_id` and result cap.
- [`search_catalog_products_paged`](#search_catalog_products_paged) - Paged public catalog search with explicit `products_sort_by` and optional rating.
- [`search_catalog_products_rich`](#search_catalog_products_rich) - Same as [`search_catalog_products`](public_meta.md#bookclerk_enrichpublic_metasearch_catalog_products) but requests list price / category ladders.
- [`search_catalog_storefront`](#search_catalog_storefront) - Storefront catalog search (`GET /1.0/catalog/search`) — the same index the

---

## bookclerk_enrich::public_meta::CatalogProduct

*Struct*

One Audible catalog product (public search hit).

**Fields:**
- `asin: String` - Audible / Amazon ASIN when this edition is sold on Audible.
- `title: Option<String>` - Display title as shown on the storefront or library card.
- `authors: Option<String>` - Comma-separated author names when the storefront provides them.
- `narrators: Option<String>` - Comma-separated narrator names when known.
- `series: Option<String>` - Series name when the title belongs to a named series.
- `series_asin: Option<String>` - Parent series ASIN when the catalog returns series metadata.
- `series_sequence: Option<String>` - Series sequence / position when present (string form from Audible).
- `cover_url: Option<String>` - Cover URL from `product_images` when the `media` response group is present.
- `subtitle: Option<String>` - Optional subtitle when the catalog distinguishes it from the title.
- `description: Option<String>` - Publisher summary / blurb (`product_extended_attrs`).
- `publisher: Option<String>` - Publisher imprint as reported by the catalog.
- `length_minutes: Option<i64>` - Audiobook runtime in whole minutes when known.
- `published_at: Option<String>` - Publication date string from the catalog (ISO or storefront format).
- `categories: Option<String>` - Genre / subject labels (`;`-separated).
- `language: Option<String>` - BCP-47 / storefront language code (`en`, `de`, …) when known.
- `price_cents: Option<i64>` - List or deal price in integer cents of `Self::currency`.
- `currency: Option<String>` - ISO 4217 currency code for price fields (`USD`, `EUR`, …).
- `price_label: Option<String>` - Human-readable price string from the storefront (e.g. `$14.95`).
- `rating_overall: Option<f64>` - Overall community rating when the `rating` response group is present.
- `rating_count: Option<i64>` - Count of star ratings (`rating.num_ratings`) when present.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CatalogProduct`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogProduct) -> bool`



## bookclerk_enrich::public_meta::CatalogRating

*Struct*

Community ratings from a single Audible catalog product (`response_groups=rating`).

Kept off bulk Discover catalog calls (see `CATALOG_RESPONSE_GROUPS`); use from
title-detail paths only.

**Fields:**
- `overall: Option<f64>` - Aggregate star rating across listeners.
- `performance: Option<f64>` - Narration / performance sub-score.
- `story: Option<f64>` - Story / content sub-score.
- `num_ratings: Option<i64>` - Total number of ratings contributing to the averages.
- `num_reviews: Option<i64>` - Total number of written reviews when distinct from ratings.

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogRating) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CatalogRating`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_enrich::public_meta::CatalogReview

*Struct*

One customer review from Audible's public catalog reviews endpoint.

**Fields:**
- `id: Option<String>` - Stable identifier for this item.
- `title: Option<String>` - Display title as shown on the storefront or library card.
- `body: String` - Full review text from the storefront.
- `author_name: Option<String>` - Display name of the review author on the storefront.
- `overall_rating: Option<i64>` - Overall star rating for this review when present.
- `performance_rating: Option<i64>` - Narration sub-rating for this review when present.
- `story_rating: Option<i64>` - Story sub-rating for this review when present.
- `submitted_at: Option<String>` - When the review was submitted (storefront timestamp string).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CatalogReview`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogReview) -> bool`



## bookclerk_enrich::public_meta::CatalogReviewsPage

*Struct*

One page of helpful Audible customer reviews (public; no account).

**Fields:**
- `reviews: Vec<CatalogReview>` - Review rows on this page.
- `page: u32` - 1-based page index for this reviews / list request.
- `page_size: usize` - Maximum number of items returned in this page.
- `has_more: bool` - When true, another page is available by incrementing `Self::page`.
- `sort_by: CatalogReviewsSort` - Remote sort key for reviews (`most_helpful`, `most_recent`, …).

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogReviewsPage) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CatalogReviewsPage`



## bookclerk_enrich::public_meta::CatalogReviewsSort

*Enum*

Audible catalog reviews sort (`MostHelpful` or `MostRecent`).

**Variants:**
- `MostHelpful` - Sort reviews by helpfulness (storefront default).
- `MostRecent` - Sort reviews by newest submission first.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the wire / FourCC string form of this value.
- `fn parse(raw: &str) -> Self` - Parses this value from its wire string form.

**Traits:** Copy, Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogReviewsSort) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CatalogReviewsSort`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_enrich::public_meta::fetch_audible_catalog_rating

*Function*

Fetch Audible community ratings for one ASIN (public catalog; no account).

# Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

# Returns

On success, the inner `Option<CatalogRating>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn fetch_audible_catalog_rating(http: &bookclerk_plugin_sdk::http::Client, asin: &str, region: &str) -> crate::error::Result<Option<CatalogRating>>
```



## bookclerk_enrich::public_meta::fetch_audible_catalog_reviews

*Function*

Fetch a short list of helpful Audible customer reviews (public; no account).

Convenience wrapper around [`fetch_audible_catalog_reviews_page`](public_meta.md#bookclerk_enrichpublic_metafetch_audible_catalog_reviews_page) (page 1).
`limit` is clamped to `1..=50` (Audible's reviews endpoint max).

# Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `limit` - Maximum number of results to return.

# Returns

On success, the inner `Vec<CatalogReview>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn fetch_audible_catalog_reviews(http: &bookclerk_plugin_sdk::http::Client, asin: &str, region: &str, limit: usize) -> crate::error::Result<Vec<CatalogReview>>
```



## bookclerk_enrich::public_meta::fetch_audible_catalog_reviews_page

*Function*

Paginated Audible catalog reviews (`page` is 1-based).

# Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `page` - Numeric `page` value for this call.
* `page_size` - Maximum items to return in this page.
* `sort_by` - `sort_by` input for this call.

# Returns

On success, the inner `CatalogReviewsPage` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn fetch_audible_catalog_reviews_page(http: &bookclerk_plugin_sdk::http::Client, asin: &str, region: &str, page: u32, page_size: usize, sort_by: CatalogReviewsSort) -> crate::error::Result<CatalogReviewsPage>
```



## bookclerk_enrich::public_meta::fetch_audnexus_book

*Function*

Fetch book metadata from Audnexus (no Audible account).

# Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

# Returns

On success, the inner `Option<Value>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn fetch_audnexus_book(http: &bookclerk_plugin_sdk::http::Client, asin: &str, region: &str) -> crate::error::Result<Option<serde_json::Value>>
```



## bookclerk_enrich::public_meta::fetch_audnexus_chapters

*Function*

Fetch chapter metadata from Audnexus and normalize to Audible `chapter_info` shape.

Audnexus uses camelCase (`startOffsetMs`, `brandIntroDurationMs`). Callers that
already accept both casings can use the raw body; this helper also mirrors
snake_case keys for code that only reads Audible API field names.

# Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

# Returns

On success, the inner `Option<Value>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn fetch_audnexus_chapters(http: &bookclerk_plugin_sdk::http::Client, asin: &str, region: &str) -> crate::error::Result<Option<serde_json::Value>>
```



## bookclerk_enrich::public_meta::fetch_public_chapter_info

*Function*

Convenience: public chapter fetch with a fresh HTTP client.

# Arguments

* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

# Returns

On success, the inner `Option<Value>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn fetch_public_chapter_info(asin: &str, region: &str) -> crate::error::Result<Option<serde_json::Value>>
```



## bookclerk_enrich::public_meta::is_audible_podcast_product

*Function*

True for Audible podcast shows / episodes (`content_type` / delivery type).

# Arguments

* `p` - `p` input for this call.

# Returns

`true` when the predicate holds.

```rust
fn is_audible_podcast_product(p: &serde_json::Value) -> bool
```



## bookclerk_enrich::public_meta::normalize_region

*Function*

Normalize a marketplace string to an Audnexus `region` query value.

# Arguments

* `region` - Marketplace / region code (`us`, `uk`, …).

# Returns

String result for this operation.

```rust
fn normalize_region(region: &str) -> String
```



## bookclerk_enrich::public_meta::normalize_review_body

*Function*

Normalize review text for clients: decode HTML entities and preserve Audible
guided questionnaire JSON (`[{ type, question, id, answer }, …]`) so the UI
can render Q&A sections. Unrelated JSON arrays are left unchanged.

# Arguments

* `body` - HTTP request body bytes.

# Returns

String result for this operation.

```rust
fn normalize_review_body(body: &str) -> String
```



## bookclerk_enrich::public_meta::public_http_client

*Function*

Shared HTTP client for public metadata calls.

# Returns

On success, the inner `Client` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn public_http_client() -> crate::error::Result<bookclerk_plugin_sdk::http::Client>
```



## bookclerk_enrich::public_meta::region_tld

*Function*

Map marketplace / region code to Audible API TLD (AudioBookshelf `regionMap`).

# Arguments

* `region` - Marketplace / region code (`us`, `uk`, …).

# Returns

`&'static str` result.

```rust
fn region_tld(region: &str) -> &'static str
```



## bookclerk_enrich::public_meta::resolve_genre_category_id

*Function*

Map a genre display name to an Audible `category_id` under the Genres root.

Fetches `GET …/catalog/categories?categories_num_levels=3&root=Genres`, flattens
the tree, and picks the best case-insensitive name match (deeper nodes and
Science Fiction & Fantasy paths preferred; Children’s / Teen deprioritized).

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `genre` - String `genre` for this call.

# Returns

On success, the inner `Option<String>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn resolve_genre_category_id(http: &bookclerk_plugin_sdk::http::Client, region: &str, genre: &str) -> crate::error::Result<Option<String>>
```



## bookclerk_enrich::public_meta::search_catalog_asins

*Function*

Search the public Audible catalog by title/author; returns ASINs (relevance order).

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `title` - Display title.
* `author` - String `author` for this call.

# Returns

On success, the inner `Vec<String>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_asins(http: &bookclerk_plugin_sdk::http::Client, region: &str, title: &str, author: Option<&str>) -> crate::error::Result<Vec<String>>
```



## bookclerk_enrich::public_meta::search_catalog_by_genre_name

*Function*

Resolve an Audible Genres browse-node id for a category display name, then list products.

`page` is 1-based (Audible catalog convention), matching
[`search_catalog_products_paged`](public_meta.md#bookclerk_enrichpublic_metasearch_catalog_products_paged).

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `genre` - String `genre` for this call.
* `page` - Numeric `page` value for this call.
* `products_sort_by` - String `products_sort_by` for this call.
* `limit` - Maximum number of results to return.
* `with_rating` - Boolean flag `with_rating`.

# Returns

On success, the inner `Vec<CatalogProduct>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_by_genre_name(http: &bookclerk_plugin_sdk::http::Client, region: &str, genre: &str, page: u32, products_sort_by: &str, limit: usize, with_rating: bool) -> crate::error::Result<Vec<CatalogProduct>>
```



## bookclerk_enrich::public_meta::search_catalog_by_narrator

*Function*

Narrator-focused public catalog search.

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `narrator` - String `narrator` for this call.

# Returns

On success, the inner `Vec<CatalogProduct>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_by_narrator(http: &bookclerk_plugin_sdk::http::Client, region: &str, narrator: &str) -> crate::error::Result<Vec<CatalogProduct>>
```



## bookclerk_enrich::public_meta::search_catalog_by_series_asin

*Function*

List products in an Audible series by parent series ASIN (public catalog).

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `series_asin` - String `series_asin` for this call.

# Returns

On success, the inner `Vec<CatalogProduct>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_by_series_asin(http: &bookclerk_plugin_sdk::http::Client, region: &str, series_asin: &str) -> crate::error::Result<Vec<CatalogProduct>>
```



## bookclerk_enrich::public_meta::search_catalog_by_series_name

*Function*

Keyword/title search, then keep products whose series name matches `series`.

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `series` - Optional series name.
* `limit` - Maximum number of results to return.

# Returns

On success, the inner `Vec<CatalogProduct>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_by_series_name(http: &bookclerk_plugin_sdk::http::Client, region: &str, series: &str, limit: usize) -> crate::error::Result<Vec<CatalogProduct>>
```



## bookclerk_enrich::public_meta::search_catalog_keywords

*Function*

Keyword catalog search (e.g. ISBN) — useful when title search misses a hit.

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `keywords` - String `keywords` for this call.

# Returns

On success, the inner `Vec<String>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_keywords(http: &bookclerk_plugin_sdk::http::Client, region: &str, keywords: &str) -> crate::error::Result<Vec<String>>
```



## bookclerk_enrich::public_meta::search_catalog_products

*Function*

Public Audible catalog search returning product metadata (not just ASINs).

Pass `title` and/or `author` and/or `keywords`. Empty title is allowed when
keywords or author alone are set.

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `title` - Display title.
* `author` - String `author` for this call.
* `keywords` - String `keywords` for this call.

# Returns

On success, the inner `Vec<CatalogProduct>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_products(http: &bookclerk_plugin_sdk::http::Client, region: &str, title: &str, author: Option<&str>, keywords: Option<&str>) -> crate::error::Result<Vec<CatalogProduct>>
```



## bookclerk_enrich::public_meta::search_catalog_products_ex

*Function*

Public catalog search with optional narrator, series ASIN, and/or category filters.

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `title` - Display title.
* `author` - String `author` for this call.
* `keywords` - String `keywords` for this call.
* `narrator` - String `narrator` for this call.
* `series_asin` - String `series_asin` for this call.
* `rich` - Boolean flag `rich`.

# Returns

On success, the inner `Vec<CatalogProduct>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_products_ex(http: &bookclerk_plugin_sdk::http::Client, region: &str, title: &str, author: Option<&str>, keywords: Option<&str>, narrator: Option<&str>, series_asin: Option<&str>, rich: bool) -> crate::error::Result<Vec<CatalogProduct>>
```



## bookclerk_enrich::public_meta::search_catalog_products_ex2

*Function*

Like [`search_catalog_products_ex`](public_meta.md#bookclerk_enrichpublic_metasearch_catalog_products_ex) with optional `category_id` and result cap.

Note: Audible's public catalog ignores `language=` query params; prefer
local re-rank on `CatalogProduct::language` after fetch.

# Arguments

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

# Returns

On success, the inner `Vec<CatalogProduct>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_products_ex2(http: &bookclerk_plugin_sdk::http::Client, region: &str, title: &str, author: Option<&str>, keywords: Option<&str>, narrator: Option<&str>, series_asin: Option<&str>, category_id: Option<&str>, rich: bool, num_results: usize) -> crate::error::Result<Vec<CatalogProduct>>
```



## bookclerk_enrich::public_meta::search_catalog_products_paged

*Function*

Paged public catalog search with explicit `products_sort_by` and optional rating.

# Arguments

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

# Returns

On success, the inner `Vec<CatalogProduct>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_products_paged(http: &bookclerk_plugin_sdk::http::Client, region: &str, title: &str, author: Option<&str>, keywords: Option<&str>, narrator: Option<&str>, series_asin: Option<&str>, category_id: Option<&str>, page: u32, products_sort_by: &str, rich: bool, num_results: usize, with_rating: bool) -> crate::error::Result<Vec<CatalogProduct>>
```



## bookclerk_enrich::public_meta::search_catalog_products_rich

*Function*

Same as [`search_catalog_products`](public_meta.md#bookclerk_enrichpublic_metasearch_catalog_products) but requests list price / category ladders.

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `title` - Display title.
* `author` - String `author` for this call.
* `keywords` - String `keywords` for this call.

# Returns

On success, the inner `Vec<CatalogProduct>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_products_rich(http: &bookclerk_plugin_sdk::http::Client, region: &str, title: &str, author: Option<&str>, keywords: Option<&str>) -> crate::error::Result<Vec<CatalogProduct>>
```



## bookclerk_enrich::public_meta::search_catalog_storefront

*Function*

Storefront catalog search (`GET /1.0/catalog/search`) — the same index the
Storefront catalog search (`GET /1.0/catalog/search`) — the same index the
Audible website uses.

Prefer this over [`search_catalog_products_paged`](public_meta.md#bookclerk_enrichpublic_metasearch_catalog_products_paged) for Discover keyword browse:
`/catalog/products?keywords=` frequently omits titles that still resolve via
`/catalog/products/{asin}` (e.g. English *A Game of Thrones* `B002UZZ93G`).

# Arguments

* `http` - `http` input for this call.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `keywords` - String `keywords` for this call.
* `page` - Numeric `page` value for this call.
* `sort` - Sort field for the merged result set.
* `rich` - Boolean flag `rich`.
* `num_results` - Numeric `num_results` value for this call.
* `with_rating` - Boolean flag `with_rating`.

# Returns

On success, the inner `Vec<CatalogProduct>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn search_catalog_storefront(http: &bookclerk_plugin_sdk::http::Client, region: &str, keywords: &str, page: u32, sort: &str, rich: bool, num_results: usize, with_rating: bool) -> crate::error::Result<Vec<CatalogProduct>>
```



