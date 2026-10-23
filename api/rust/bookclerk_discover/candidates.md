**bookclerk_discover > candidates**

# Module: candidates

## Contents

**Structs**

- [`CandidateFetchOptions`](#candidatefetchoptions) - Options for storefront candidate expansion.
- [`StorefrontCandidate`](#storefrontcandidate) - A purchase candidate discovered from a storefront catalog (not owned locally).

**Functions**

- [`gather_storefront_candidates`](#gather_storefront_candidates) - Expand storefront catalogs from local taste seeds; drop already-owned ids.
- [`select_taste_seeds`](#select_taste_seeds) - Pick local seed books for storefront expansion (finished / high-rated first).

---

## bookclerk_discover::candidates::CandidateFetchOptions

*Struct*

Options for storefront candidate expansion.

**Fields:**
- `region: String` - Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `seed_limit: usize` - Max local seed titles to expand from (finished / rated first).
- `max_remote_calls: usize` - Cap remote HTTP calls across all storefronts.
- `include_audible: bool` - Call `bookclerk_source::ContentSource::expand_candidates` on registered Audible.
- `include_libro: bool` - Call `bookclerk_source::ContentSource::expand_candidates` on registered Libro.fm.
- `include_chirp: bool` - Call `bookclerk_source::ContentSource::expand_candidates` on registered Chirp.
- `include_graphicaudio: bool` - Call `bookclerk_source::ContentSource::expand_candidates` on registered GraphicAudio.
- `include_deals: bool` - Fetch deals via `bookclerk_source::ContentSource::list_deals` on all registered sources.
- `exclude_graphicaudio_series_sets: bool` - When true, drop GraphicAudio Magento series-set SKUs from candidates.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> CandidateFetchOptions`
- **Default**
  - `fn default() -> Self`



## bookclerk_discover::candidates::StorefrontCandidate

*Struct*

A purchase candidate discovered from a storefront catalog (not owned locally).

**Fields:**
- `source: String` - Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).
- `product_id: String` - Storefront-native product id (ASIN, ISBN, UUID, …).
- `title: String` - Display title as shown on the storefront or library card.
- `authors: Option<String>` - Comma-separated author names when the storefront provides them.
- `narrators: Option<String>` - Comma-separated narrator names when known.
- `series: Option<String>` - Series name when the title belongs to a named series.
- `series_index: Option<String>` - Position within the series (e.g. `1`, `1.5`) when known.
- `asin: Option<String>` - Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn: Option<String>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url: Option<String>` - Public cover image URL when a storefront provided one.
- `seed_categories: Option<String>` - Categories/subjects copied from the taste seed that produced this hit.
- `origin: String` - How this candidate was found (related-to seed, author search, …).
- `seed_title: Option<String>` - Owned-library title that seeded this related / similar candidate.
- `store_editions: Vec<crate::identity::StoreEdition>` - Known storefront editions of this work (including the primary source).
- `subtitle: Option<String>` - Bibliographic extras from the storefront catalog payload (optional).
- `description: Option<String>` - Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher: Option<String>` - Publisher imprint as reported by the catalog.
- `length_minutes: Option<i64>` - Audiobook runtime in whole minutes when known.
- `published_at: Option<String>` - Publication date string from the catalog (ISO or storefront format).
- `categories: Option<String>` - Genre / subject tags as a single delimited string when known.
- `language: Option<String>` - BCP-47 / storefront language code (`en`, `de`, …) when known.
- `price_cents: Option<i64>` - List or deal price in integer cents of `Self::currency`.
- `currency: Option<String>` - ISO 4217 currency code for price fields (`USD`, `EUR`, …).
- `price_label: Option<String>` - Human-readable price string from the storefront (e.g. `$14.95`).
- `rating_overall: Option<f64>` - Community overall rating when a storefront provided one.
- `rating_count: Option<i64>` - Number of ratings backing `Self::rating_overall` when known.
- `is_abridged: Option<bool>` - Abridged flag when the storefront provided one.
- `audible_rank: Option<u32>` - Position within the Audible page that produced this hit (for merge rank).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> StorefrontCandidate`



## bookclerk_discover::candidates::gather_storefront_candidates

*Function*

Expand storefront catalogs from local taste seeds; drop already-owned ids.

# Arguments

* `library` - Open library store used for reads/writes.
* `registry` - Configured content-source or integration registry.
* `seeds` - `seeds` input for this call.
* `owned_asins` - String `owned_asins` for this call.
* `owned_isbns` - String `owned_isbns` for this call.
* `owned_product_keys` - String `owned_product_keys` for this call.
* `opts` - Options struct for this operation.

# Returns

On success, the inner `Vec<StorefrontCandidate>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn gather_storefront_candidates(_library: &bookclerk_library::LibraryStore, registry: &bookclerk_source::SourceRegistry, seeds: &[bookclerk_library::BookRecord], owned_asins: &std::collections::HashSet<String>, owned_isbns: &std::collections::HashSet<String>, owned_product_keys: &std::collections::HashSet<String>, opts: &CandidateFetchOptions) -> std::result::Result<Vec<StorefrontCandidate>, DiscoverError>
```



## bookclerk_discover::candidates::select_taste_seeds

*Function*

Pick local seed books for storefront expansion (finished / high-rated first).

# Arguments

* `books` - `books` input for this call.
* `listening_engagement_by_uuid` - String `listening_engagement_by_uuid` for this call.

# Returns

Collected results (may be empty).

```rust
fn select_taste_seeds(books: &[bookclerk_library::BookRecord], listening_engagement_by_uuid: &std::collections::HashMap<String, f64>) -> Vec<bookclerk_library::BookRecord>
```



