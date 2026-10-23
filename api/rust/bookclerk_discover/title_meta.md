**bookclerk_discover > title_meta**

# Module: title_meta

## Contents

**Structs**

- [`TitleMeta`](#titlemeta) - Bibliographic fields suitable for an Audible-style detail panel.
- [`TitleMetaQuery`](#titlemetaquery) - Query for public title metadata (Audnexus / Audible catalog).
- [`TitleReview`](#titlereview) - One Audible customer review for the title detail panel.
- [`TitleReviewsPage`](#titlereviewspage) - One page of title reviews for infinite scroll.
- [`TitleReviewsQuery`](#titlereviewsquery) - Query for a page of Audible customer reviews.

**Functions**

- [`resolve_title_meta`](#resolve_title_meta) - Resolve public metadata for a title detail dialog.
- [`resolve_title_meta_batch`](#resolve_title_meta_batch) - Resolve many title-meta queries with bounded concurrency (order preserved).
- [`resolve_title_reviews`](#resolve_title_reviews) - Resolve one page of Audible customer reviews for infinite scroll.

---

## bookclerk_discover::title_meta::TitleMeta

*Struct*

Bibliographic fields suitable for an Audible-style detail panel.

**Fields:**
- `asin: Option<String>` - Audible / Amazon ASIN when this edition is sold on Audible.
- `title: Option<String>` - Display title as shown on the storefront or library card.
- `subtitle: Option<String>` - Optional subtitle when the catalog distinguishes it from the title.
- `authors: Option<String>` - Comma-separated author names when the storefront provides them.
- `narrators: Option<String>` - Comma-separated narrator names when known.
- `series: Option<String>` - Series name when the title belongs to a named series.
- `series_index: Option<String>` - Position within the series (e.g. `1`, `1.5`) when known.
- `isbn: Option<String>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url: Option<String>` - HTTPS URL for cover art when the catalog exposes one.
- `description: Option<String>` - Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher: Option<String>` - Publisher imprint as reported by the catalog.
- `length_minutes: Option<i64>` - Audiobook runtime in whole minutes when known.
- `published_at: Option<String>` - Publication date string from the catalog (ISO or storefront format).
- `categories: Option<String>` - Genre / subject tags as a single delimited string when known.
- `language: Option<String>` - BCP-47 / storefront language code (`en`, `de`, …) when known.
- `is_abridged: Option<bool>` - `Some(true)` abridged / `Some(false)` unabridged when the storefront said so.
- `rating_overall: Option<f64>` - Audible community overall rating (0–5).
- `rating_performance: Option<f64>` - Narration / performance sub-score when the storefront splits ratings.
- `rating_story: Option<f64>` - Story / content sub-score when the storefront splits ratings.
- `rating_count: Option<i64>` - Count of star ratings (Audible `num_ratings`).
- `review_count: Option<i64>` - Count of written reviews (Audible `num_reviews`).
- `reviews: Vec<TitleReview>` - Deprecated: reviews load via `/api/discover/title-reviews`. Kept empty

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TitleMeta`



## bookclerk_discover::title_meta::TitleMetaQuery

*Struct*

Query for public title metadata (Audnexus / Audible catalog).

**Fields:**
- `title: String` - Display title as shown on the storefront or library card.
- `authors: Option<String>` - Comma-separated author names when the storefront provides them.
- `asin: Option<String>` - Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn: Option<String>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `narrators: Option<String>` - Comma-separated narrator names when known.
- `length_minutes: Option<i64>` - Audiobook runtime in whole minutes when known.
- `region: String` - Marketplace / region code (`us`, `uk`, …) for catalog lookups.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TitleMetaQuery`



## bookclerk_discover::title_meta::TitleReview

*Struct*

One Audible customer review for the title detail panel.

**Fields:**
- `id: Option<String>` - Stable identifier for this item.
- `title: Option<String>` - Display title as shown on the storefront or library card.
- `body: String` - Full review text from the storefront.
- `author_name: Option<String>` - Display name of the review author on the storefront.
- `overall_rating: Option<i64>` - Overall star rating for this review when present.
- `performance_rating: Option<i64>` - Narration sub-rating for this review when present.
- `story_rating: Option<i64>` - Story sub-rating for this review when present.
- `submitted_at: Option<String>` - When the review was submitted (storefront timestamp string).

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TitleReview`
- **From**
  - `fn from(r: CatalogReview) -> Self`



## bookclerk_discover::title_meta::TitleReviewsPage

*Struct*

One page of title reviews for infinite scroll.

**Fields:**
- `asin: String` - Audible / Amazon ASIN when this edition is sold on Audible.
- `page: u32` - 1-based page index for this reviews / list request.
- `page_size: u32` - Maximum number of items returned in this page.
- `has_more: bool` - When true, another page is available by incrementing `Self::page`.
- `sort_by: String` - Remote sort key for reviews (`most_helpful`, `most_recent`, …).
- `reviews: Vec<TitleReview>` - Review rows on this page.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TitleReviewsPage`



## bookclerk_discover::title_meta::TitleReviewsQuery

*Struct*

Query for a page of Audible customer reviews.

**Fields:**
- `asin: String` - Audible / Amazon ASIN when this edition is sold on Audible.
- `region: String` - Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `page: u32` - 1-based page index.
- `page_size: u32` - Page size (clamped server-side to 1..=20).
- `sort_by: String` - `MostHelpful` (default) or `MostRecent`.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TitleReviewsQuery`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::title_meta::resolve_title_meta

*Function*

Resolve public metadata for a title detail dialog.

When `sources` is provided, sparse Audible/Audnexus results are gap-filled
from storefront `bookclerk_source::ContentSource::catalog_detail` (Libro ISBN details).

# Arguments

* `query` - Query vector or free-text search string.
* `sources` - String `sources` for this call.

# Returns

On success, the inner `Option<TitleMeta>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn resolve_title_meta(query: &TitleMetaQuery, sources: Option<&bookclerk_source::SourceRegistry>) -> std::result::Result<Option<TitleMeta>, DiscoverError>
```



## bookclerk_discover::title_meta::resolve_title_meta_batch

*Function*

Resolve many title-meta queries with bounded concurrency (order preserved).

# Arguments

* `queries` - Batch of purchase-hint or title-meta queries.
* `max_concurrent` - Numeric `max_concurrent` value for this call.
* `sources` - String `sources` for this call.

# Returns

On success, `Vec<Result<Option<TitleMeta>>>`.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn resolve_title_meta_batch(queries: &[TitleMetaQuery], max_concurrent: usize, sources: Option<&bookclerk_source::SourceRegistry>) -> Vec<std::result::Result<Option<TitleMeta>, DiscoverError>>
```



## bookclerk_discover::title_meta::resolve_title_reviews

*Function*

Resolve one page of Audible customer reviews for infinite scroll.

# Arguments

* `query` - Query vector or free-text search string.

# Returns

On success, the inner `TitleReviewsPage` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn resolve_title_reviews(query: &TitleReviewsQuery) -> std::result::Result<TitleReviewsPage, DiscoverError>
```



