**bookclerk_enrich > enrich**

# Module: enrich

## Contents

**Structs**

- [`Enrichment`](#enrichment) - Metadata pulled from Audible / Audnexus for a confident match.
- [`ScoredMatch`](#scoredmatch) - A scored Audible match candidate.

**Functions**

- [`apply_enrichment_to_book`](#apply_enrichment_to_book) - Apply a confident Audible catalog match onto a non-Audible ownership row.
- [`confidence_percent_to_fraction`](#confidence_percent_to_fraction) - Convert a 0–100 percent threshold to a 0.0–1.0 fraction.
- [`enrich_books_from_audible`](#enrich_books_from_audible) - Enrich library rows that lack an ASIN via public Audible catalog + Audnexus.
- [`enrichment_for_asin`](#enrichment_for_asin) - Fetch Audnexus metadata for a known ASIN (no confidence scoring).
- [`enrichment_for_asin_with_client`](#enrichment_for_asin_with_client) - Same as [`enrichment_for_asin`](enrich.md#bookclerk_enrichenrichenrichment_for_asin) with a shared HTTP client.
- [`lookup_by_metadata`](#lookup_by_metadata) - Look up the best Audible match for the given query metadata via public APIs.
- [`lookup_by_metadata_with_client`](#lookup_by_metadata_with_client) - Catalog + Audnexus lookup against an already-built HTTP client.

**Constants**

- [`DEFAULT_ENRICH_MIN_CONFIDENCE`](#default_enrich_min_confidence) - Default minimum confidence percent (AudioBookshelf-style 0–100 scale).

---

## bookclerk_enrich::enrich::DEFAULT_ENRICH_MIN_CONFIDENCE

*Constant*: `u8`

Default minimum confidence percent (AudioBookshelf-style 0–100 scale).



## bookclerk_enrich::enrich::Enrichment

*Struct*

Metadata pulled from Audible / Audnexus for a confident match.

**Fields:**
- `asin: String` - Audible / Amazon ASIN when this edition is sold on Audible.
- `title: String` - Display title as shown on the storefront or library card.
- `authors: Option<String>` - Comma-separated author names when the storefront provides them.
- `narrators: Option<String>` - Comma-separated narrator names when known.
- `series: Option<String>` - Series name when the title belongs to a named series.
- `series_index: Option<String>` - Position within the series (e.g. `1`, `1.5`) when known.
- `series_asin: Option<String>` - Audible series ASIN when the catalog associates one.
- `length_minutes: Option<i64>` - Audiobook runtime in whole minutes when known.
- `publisher: Option<String>` - Publisher imprint as reported by the catalog.
- `subtitle: Option<String>` - Optional subtitle when the catalog distinguishes it from the title.
- `cover_url: Option<String>` - HTTPS URL for cover art when the catalog exposes one.
- `isbn: Option<String>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `published_at: Option<String>` - ISO-8601 / RFC3339 timestamp when known (`releaseDate`).
- `categories: Option<String>` - ABS-style genre string (`;`-separated).
- `description: Option<String>` - Publisher or storefront synopsis; may be truncated for embeddings.
- `language: Option<String>` - BCP-47 / storefront language code (`en`, `de`, …) when known.
- `confidence: Option<f64>` - Match confidence in `[0.0, 1.0]` (omit/`None` for legacy callers).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Enrichment`
- **PartialEq**
  - `fn eq(self: &Self, other: &Enrichment) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_enrich::enrich::ScoredMatch

*Struct*

A scored Audible match candidate.

**Fields:**
- `enrichment: Enrichment` - Filled metadata fields from the public catalog match.
- `confidence: f64` - Match confidence in `[0.0, 1.0]`; higher means a stronger ASIN match.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ScoredMatch`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_enrich::enrich::apply_enrichment_to_book

*Function*

Apply a confident Audible catalog match onto a non-Audible ownership row.

Preserves `source`, `product_id`, `uuid`, `account_id`, and an existing
`asin` (sets ASIN only when the row does not already have one). Catalog
fields win for display metadata (title, contributors, series, ISBN, genres,
publish date). Store-native `length_minutes` is kept when present — Audible
runtimes include brand intro/outro that plain Acquire files omit.

# Arguments

* `library` - Open library store used for reads/writes.
* `book_uuid` - String `book_uuid` for this call.
* `enrichment` - Catalog fields to merge onto the book row.

# Returns

On success, the inner `bookclerk_library::BookRecord` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn apply_enrichment_to_book(library: &bookclerk_library::LibraryStore, book_uuid: &str, enrichment: &Enrichment) -> crate::error::Result<bookclerk_library::BookRecord>
```



## bookclerk_enrich::enrich::confidence_percent_to_fraction

*Function*

Convert a 0–100 percent threshold to a 0.0–1.0 fraction.

# Arguments

* `percent` - Confidence threshold expressed as 0–100.

# Returns

`f64` result.

```rust
fn confidence_percent_to_fraction(percent: u8) -> f64
```



## bookclerk_enrich::enrich::enrich_books_from_audible

*Function*

Enrich library rows that lack an ASIN via public Audible catalog + Audnexus.

Source-agnostic: any non-`audible` row with title and/or ISBN is considered.
Uses title, author, narrator, subtitle, ISBN, and duration when present.
`min_confidence_percent` is 0–100 (default [`DEFAULT_ENRICH_MIN_CONFIDENCE`](enrich.md#bookclerk_enrichenrichdefault_enrich_min_confidence)).

# Arguments

* `library` - Open library store used for reads/writes.
* `min_confidence_percent` - `min_confidence_percent` input for this call.

# Returns

On success, the inner `usize` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn enrich_books_from_audible(library: &bookclerk_library::LibraryStore, min_confidence_percent: u8) -> crate::error::Result<usize>
```



## bookclerk_enrich::enrich::enrichment_for_asin

*Function*

Fetch Audnexus metadata for a known ASIN (no confidence scoring).

Used by Discover / Wishlist / Library detail views to surface description,
runtime, publisher, and related fields without requiring a library row.

# Arguments

* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

# Returns

On success, the inner `Option<Enrichment>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn enrichment_for_asin(asin: &str, region: &str) -> crate::error::Result<Option<Enrichment>>
```



## bookclerk_enrich::enrich::enrichment_for_asin_with_client

*Function*

Same as [`enrichment_for_asin`](enrich.md#bookclerk_enrichenrichenrichment_for_asin) with a shared HTTP client.

# Arguments

* `http` - `http` input for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `region` - Marketplace / region code (`us`, `uk`, …).

# Returns

On success, the inner `Option<Enrichment>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn enrichment_for_asin_with_client(http: &bookclerk_plugin_sdk::http::Client, asin: &str, region: &str) -> crate::error::Result<Option<Enrichment>>
```



## bookclerk_enrich::enrich::lookup_by_metadata

*Function*

Look up the best Audible match for the given query metadata via public APIs.

Returns `None` when no candidate meets `min_confidence` (0.0–1.0).

# Arguments

* `query` - Query vector or free-text search string.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `min_confidence` - Minimum match confidence in `[0.0, 1.0]` to apply enrichment.

# Returns

On success, the inner `Option<ScoredMatch>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn lookup_by_metadata(query: &crate::match_score::MatchQuery, region: &str, min_confidence: f64) -> crate::error::Result<Option<ScoredMatch>>
```



## bookclerk_enrich::enrich::lookup_by_metadata_with_client

*Function*

Catalog + Audnexus lookup against an already-built HTTP client.

# Arguments

* `http` - `http` input for this call.
* `query` - Query vector or free-text search string.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `min_confidence` - Minimum match confidence in `[0.0, 1.0]` to apply enrichment.

# Returns

On success, the inner `Option<ScoredMatch>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn lookup_by_metadata_with_client(http: &bookclerk_plugin_sdk::http::Client, query: &crate::match_score::MatchQuery, region: &str, min_confidence: f64) -> crate::error::Result<Option<ScoredMatch>>
```



