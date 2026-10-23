**bookclerk_discover > catalog_search**

# Module: catalog_search

## Contents

**Structs**

- [`CatalogSearchFilters`](#catalogsearchfilters) - Include / exclude filters applied after identity merge.
- [`CatalogSearchHit`](#catalogsearchhit) - One autocomplete / results-page hit (possibly spanning multiple storefronts).
- [`CatalogSearchPage`](#catalogsearchpage) - Paged catalog-search response.
- [`CatalogSearchPageOpts`](#catalogsearchpageopts) - Options for [`catalog_search_page`](catalog_search.md#bookclerk_discovercatalog_searchcatalog_search_page).

**Functions**

- [`catalog_search`](#catalog_search) - Search every configured storefront catalog and merge by work identity.
- [`catalog_search_page`](#catalog_search_page) - Paged multi-store catalog browse (filters + sort + cursor).
- [`catalog_search_with_field`](#catalog_search_with_field) - Like [`catalog_search`](catalog_search.md#bookclerk_discovercatalog_searchcatalog_search), optionally scoping storefront queries to a facet

---

## bookclerk_discover::catalog_search::CatalogSearchFilters

*Struct*

Include / exclude filters applied after identity merge.

**Fields:**
- `authors: Vec<String>` - Comma-separated author names when the storefront provides them.
- `narrators: Vec<String>` - Comma-separated narrator names when known.
- `series: Vec<String>` - Series name when the title belongs to a named series.
- `genres: Vec<String>` - Genre labels from the catalog (comma-separated when multiple).
- `sources: Vec<String>` - Deduped storefront ids that contributed editions to this hit.
- `exclude_sources: Vec<String>` - Store ids to drop (any edition matching). Empty = no exclude.
- `languages: Vec<String>` - Normalized language codes (`en`, `zh`, …). When non-empty, only matching
- `exclude_narrators: Vec<String>` - Substring match on narrator strings (case-insensitive), e.g. `Virtual Voice`.
- `min_rating: Option<f64>` - Keep hits with `rating_overall >= min` or missing rating.
- `min_length_minutes: Option<i64>` - Inclusive lower bound on `length_minutes`; missing length passes.
- `max_length_minutes: Option<i64>` - Inclusive upper bound on `length_minutes`; missing length passes.

**Methods:**

- `fn is_empty(self: &Self) -> bool` - Returns true when no include/exclude filters are set.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> CatalogSearchFilters`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSearchFilters`



## bookclerk_discover::catalog_search::CatalogSearchHit

*Struct*

One autocomplete / results-page hit (possibly spanning multiple storefronts).

**Fields:**
- `work_key: String` - Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).
- `title: String` - Display title as shown on the storefront or library card.
- `authors: Option<String>` - Comma-separated author names when the storefront provides them.
- `narrators: Option<String>` - Comma-separated narrator names when known.
- `series: Option<String>` - Series name when the title belongs to a named series.
- `series_index: Option<String>` - Position within the series (e.g. `1`, `1.5`) when known.
- `asin: Option<String>` - Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn: Option<String>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url: Option<String>` - HTTPS URL for cover art when the catalog exposes one.
- `store_editions: Vec<crate::identity::StoreEdition>` - Per-storefront edition ids that collapsed into this work card.
- `sources: Vec<String>` - Storefronts that matched (deduped source ids).
- `subtitle: Option<String>` - Optional subtitle when the catalog distinguishes it from the title.
- `description: Option<String>` - Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher: Option<String>` - Publisher imprint as reported by the catalog.
- `length_minutes: Option<i64>` - Audiobook runtime in whole minutes when known.
- `published_at: Option<String>` - Publication date string from the catalog (ISO or storefront format).
- `genres: Option<String>` - Genre labels from the catalog (comma-separated when multiple).
- `language: Option<String>` - BCP-47 / storefront language code (`en`, `de`, …) when known.
- `is_abridged: Option<bool>` - When `Some(true)`, the edition is marked abridged by the storefront.
- `rating_overall: Option<f64>` - Aggregate listener rating on the storefront's scale when known.
- `rating_count: Option<i64>` - Number of ratings backing `Self::rating_overall` when known.
- `price_cents: Option<i64>` - Catalog list/deal price in cents when the storefront provided it.
- `purchase_hints: Vec<crate::purchase::PurchaseHint>` - Buy / open-in-store links with optional member vs list pricing.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSearchHit`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::catalog_search::CatalogSearchPage

*Struct*

Paged catalog-search response.

**Fields:**
- `items: Vec<CatalogSearchHit>` - Ordered collection of child entries for this response.
- `page_size: usize` - Maximum number of items returned in this page.
- `has_more: bool` - When true, another page is available via `Self::next_cursor`.
- `next_cursor: Option<String>` - Opaque cursor for the next page; omit or `None` when exhausted.
- `sort: String` - Sort field applied to the merged result set.
- `sort_dir: String` - Ascending or descending order for `Self::sort`.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSearchPage`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::catalog_search::CatalogSearchPageOpts

*Struct*

Options for [`catalog_search_page`](catalog_search.md#bookclerk_discovercatalog_searchcatalog_search_page).

**Generic Parameters:**
- 'a

**Fields:**
- `query: &'a str` - Free-text search string entered by the operator or SPA.
- `region: &'a str` - Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `page_size: usize` - Maximum number of items returned in this page.
- `cursor: Option<&'a str>` - Opaque pagination token from a previous page response.
- `sort: bookclerk_source::CatalogSearchSort` - Sort field applied to the merged result set.
- `sort_dir: bookclerk_source::CatalogSortDir` - Ascending or descending order for `Self::sort`.
- `field: Option<bookclerk_source::CatalogSearchField>` - Optional catalog field scope (`title`, `author`, …) when searching one facet.
- `language: Option<&'a str>` - BCP-47 / storefront language code (`en`, `de`, …) when known.
- `all_languages: bool` - When true, do not default hard language filter from `Self::language`.
- `filters: CatalogSearchFilters` - Server-side include/exclude filters applied after identity merge.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSearchPageOpts<'a>`



## bookclerk_discover::catalog_search::catalog_search

*Function*

Search every configured storefront catalog and merge by work identity.

# Arguments

* `registry` - Configured content-source or integration registry.
* `query` - Query vector or free-text search string.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `limit` - Maximum number of results to return.

# Returns

On success, the inner `Vec<CatalogSearchHit>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn catalog_search(registry: &bookclerk_source::SourceRegistry, query: &str, region: &str, limit: usize) -> std::result::Result<Vec<CatalogSearchHit>, DiscoverError>
```



## bookclerk_discover::catalog_search::catalog_search_page

*Function*

Paged multi-store catalog browse (filters + sort + cursor).

# Arguments

* `registry` - Configured content-source or integration registry.
* `opts` - Options struct for this operation.

# Returns

On success, the inner `CatalogSearchPage` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn catalog_search_page(registry: &bookclerk_source::SourceRegistry, opts: CatalogSearchPageOpts) -> std::result::Result<CatalogSearchPage, DiscoverError>
```



## bookclerk_discover::catalog_search::catalog_search_with_field

*Function*

Like [`catalog_search`](catalog_search.md#bookclerk_discovercatalog_searchcatalog_search), optionally scoping storefront queries to a facet
(author / narrator / series / genre). When `language` is set it is applied
as a hard include filter (unknown/missing language still passes).

# Arguments

* `registry` - Configured content-source or integration registry.
* `query` - Query vector or free-text search string.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `limit` - Maximum number of results to return.
* `field` - Optional catalog search field scope.
* `language` - Optional preferred language code.

# Returns

On success, the inner `Vec<CatalogSearchHit>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn catalog_search_with_field(registry: &bookclerk_source::SourceRegistry, query: &str, region: &str, limit: usize, field: Option<bookclerk_source::CatalogSearchField>, language: Option<&str>) -> std::result::Result<Vec<CatalogSearchHit>, DiscoverError>
```



