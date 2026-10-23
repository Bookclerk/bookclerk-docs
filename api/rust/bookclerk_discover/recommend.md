**bookclerk_discover > recommend**

# Module: recommend

## Contents

**Structs**

- [`RankedQueueEntry`](#rankedqueueentry) - Global wishlist queue entry ranked by overall/operator recommend taste +
- [`RecommendOptions`](#recommendoptions) - Tunables for [`recommend`](recommend.md#bookclerk_discoverrecommendrecommend).
- [`Recommendation`](#recommendation) - One ranked recommendation (typically an unowned storefront title).

**Functions**

- [`combine_wishlist_score`](#combine_wishlist_score) - Combine local recommend taste with a heavy multi-user wishlist boost.
- [`listening_engagement`](#listening_engagement) - Continuous engagement from a listening progress row (≈0.0–6.0).
- [`rank_global_request_queue`](#rank_global_request_queue) - Rank the global wishlist queue with **overall / operator** recommend taste
- [`recommend`](#recommend) - Build ranked recommendations for the operator (or a specific external user).
- [`recommend_feed`](#recommend_feed) - Personalized Discover feed (Netflix-style shelves) from the same candidate pool.

**Constants**

- [`WISH_COUNT_WEIGHT`](#wish_count_weight) - Per open wish on the global queue. Large enough that multi-user demand

---

## bookclerk_discover::recommend::RankedQueueEntry

*Struct*

Global wishlist queue entry ranked by overall/operator recommend taste +
heavy wish-count weight (shared order; not per-viewer personalized).

**Fields:**
- `work_key: String` - Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).
- `title: String` - Display title as shown on the storefront or library card.
- `authors: Option<String>` - Comma-separated author names when the storefront provides them.
- `asin: Option<String>` - Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn: Option<String>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url: Option<String>` - HTTPS URL for cover art when the catalog exposes one.
- `wish_count: i64` - Number of distinct users who wishlisted this title.
- `wishers: Vec<bookclerk_library::QueueWisher>` - Distinct people who have this title on an open wishlist.
- `sample_uuids: Vec<String>` - Sample Bookclerk user UUIDs who requested this title (privacy-capped).
- `first_requested_at: chrono::DateTime<chrono::Utc>` - UTC timestamp of the earliest wishlist request for this work.
- `last_requested_at: chrono::DateTime<chrono::Utc>` - UTC timestamp of the latest wishlist request for this work.
- `score: f64` - Final rank score (`taste_score + wish_count * `[`WISH_COUNT_WEIGHT`](recommend.md#bookclerk_discoverrecommendwish_count_weight)).
- `taste_score: f64` - Local taste / embedding component before the wish-count boost.
- `reasons: Vec<String>` - Short human-readable explanations for why this item was ranked here.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> RankedQueueEntry`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::recommend::RecommendOptions

*Struct*

Tunables for [`recommend`](recommend.md#bookclerk_discoverrecommendrecommend).

**Fields:**
- `limit: usize` - Maximum number of recommendations to return.
- `embedding_model: String` - Embedding model id used for similarity (`all-minilm-l6-v2-q`, …).
- `region: String` - Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `include_purchase_hints: bool` - When true, attach buy links to each recommendation.
- `external_user_id: Option<String>` - When set, only listening rows for this external user influence ranking.
- `include_listening: bool` - When false, ignore `listening_progress` entirely (owned-library taste only).
- `listening_providers: Vec<String>` - When non-empty, only use listening rows from these integration ids
- `fetch_storefront_candidates: bool` - Pull unowned titles from storefront catalogs (the primary path).
- `storefront_seed_limit: usize` - Max owned titles used as storefront related-search seeds.
- `storefront_max_remote_calls: usize` - Hard cap on outbound catalog calls during recommend.
- `exclude_graphicaudio_series_sets: bool` - Drop GraphicAudio Magento series-set SKUs from discovery candidates.
- `disabled_shelves: Vec<String>` - Shelf kinds / ids to hide (`finish_series`, `author`, `genre`, `from_store`, …).
- `models_dir: Option<std::path::PathBuf>` - Models dir for on-the-fly candidate embedding (optional; empty = skip).
- `embed_intra_threads: usize` - ONNX intra-op thread count for the embedder (clamped ≥ 1).
- `embeddings_enabled: bool` - When false, skip embedding similarity and use heuristic ranking only.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RecommendOptions`
- **Default**
  - `fn default() -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::recommend::Recommendation

*Struct*

One ranked recommendation (typically an unowned storefront title).

**Fields:**
- `work_id: Option<String>` - Library `works.id` UUID when the recommendation is tied to an owned work.
- `title: String` - Display title as shown on the storefront or library card.
- `authors: Option<String>` - Comma-separated author names when the storefront provides them.
- `narrators: Option<String>` - Comma-separated narrator names when known.
- `series: Option<String>` - Series name when the title belongs to a named series.
- `series_index: Option<String>` - Position within the series (e.g. `1`, `1.5`) when known.
- `asin: Option<String>` - Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn: Option<String>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `score: f64` - Ranking score (higher is better); units are algorithm-specific.
- `reasons: Vec<String>` - Short human-readable explanations for why this item was ranked here.
- `purchase_hints: Vec<crate::purchase::PurchaseHint>` - Buy / open-in-store links with optional member vs list pricing.
- `from_request: bool` - True when sourced from an open title request rather than storefront discovery.
- `request_uuid: Option<String>` - Wishlist / title-request UUID when this card came from a request.
- `candidate_source: Option<String>` - Storefront that proposed this title (`audible`, `libro`, …).
- `candidate_product_id: Option<String>` - Storefront product id of the edition that produced this card.
- `store_editions: Vec<crate::identity::StoreEdition>` - All known storefront editions (for multi-store purchase links).
- `seed_categories: Option<String>` - Categories/subjects copied from the taste seed that produced this hit.
- `categories: Vec<String>` - Stable shelf-kind tags for prefs (`finish_series`, `author`, `chirp_deals`, …).
- `work_key: String` - Stable bibliographic key for wishlist / merge (`isbn:` / `asin:` / `soft:`…).
- `cover_url: Option<String>` - Cover art URL when known from a storefront catalog hit or wishlist row.
- `subtitle: Option<String>` - Optional bibliographic extras from storefront catalog / Audnexus.
- `description: Option<String>` - Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher: Option<String>` - Publisher imprint as reported by the catalog.
- `length_minutes: Option<i64>` - Audiobook runtime in whole minutes when known.
- `published_at: Option<String>` - Publication date string from the catalog (ISO or storefront format).
- `genres: Option<String>` - Candidate genres (not taste-seed `seed_categories`).
- `language: Option<String>` - BCP-47 / storefront language code (`en`, `de`, …) when known.
- `wishers: Vec<bookclerk_library::QueueWisher>` - Distinct people who have this title on an open wishlist.
- `wish_count: i64` - Number of distinct users who wishlisted this title (may exceed `wishers.len()`).

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Recommendation`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> Recommendation`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::recommend::WISH_COUNT_WEIGHT

*Constant*: `f64`

Per open wish on the global queue. Large enough that multi-user demand
dominates typical local taste scores (~1–25) while recommend signals still
order titles that share the same wish count.



## bookclerk_discover::recommend::combine_wishlist_score

*Function*

Combine local recommend taste with a heavy multi-user wishlist boost.

# Arguments

* `taste_score` - `taste_score` input for this call.
* `wish_count` - Numeric `wish_count` value for this call.

# Returns

`f64` result.

```rust
fn combine_wishlist_score(taste_score: f64, wish_count: i64) -> f64
```



## bookclerk_discover::recommend::listening_engagement

*Function*

Continuous engagement from a listening progress row (≈0.0–6.0).

**Absolute hours heard** are the primary signal — 50% of a 30‑hour title
(15 h) outweighs 50% of a 3‑hour title (1.5 h). Percent complete is only a
secondary completion bonus, not the main weight.

# Arguments

* `row` - SeaORM query row to project into an RPC DTO.

# Returns

`f64` result.

```rust
fn listening_engagement(row: &bookclerk_library::ListeningProgressRecord) -> f64
```



## bookclerk_discover::recommend::rank_global_request_queue

*Function*

Rank the global wishlist queue with **overall / operator** recommend taste
plus a heavy wish-count weight.

This ranking is **not** personalized: portal `external_user_id` filters are
ignored so every viewer sees the same shared queue order (household library
+ all listening progress). Titles already in the library are omitted.

Wishlist rows keyed as ASIN vs ISBN for the same work are merged first.

# Arguments

* `library` - Open library store used for reads/writes.
* `registry` - Configured content-source or integration registry.
* `opts` - Options struct for this operation.

# Returns

On success, the inner `Vec<RankedQueueEntry>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn rank_global_request_queue(library: &bookclerk_library::LibraryStore, _registry: &bookclerk_source::SourceRegistry, opts: &RecommendOptions) -> std::result::Result<Vec<RankedQueueEntry>, DiscoverError>
```



## bookclerk_discover::recommend::recommend

*Function*

Build ranked recommendations for the operator (or a specific external user).

# Arguments

* `library` - Open library store used for reads/writes.
* `registry` - Configured content-source or integration registry.
* `opts` - Options struct for this operation.

# Returns

On success, the inner `Vec<Recommendation>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn recommend(library: &bookclerk_library::LibraryStore, registry: &bookclerk_source::SourceRegistry, opts: &RecommendOptions) -> std::result::Result<Vec<Recommendation>, DiscoverError>
```



## bookclerk_discover::recommend::recommend_feed

*Function*

Personalized Discover feed (Netflix-style shelves) from the same candidate pool.

# Arguments

* `library` - Open library store used for reads/writes.
* `registry` - Configured content-source or integration registry.
* `opts` - Options struct for this operation.

# Returns

On success, the inner `crate::shelves::DiscoverFeed` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn recommend_feed(library: &bookclerk_library::LibraryStore, registry: &bookclerk_source::SourceRegistry, opts: &RecommendOptions) -> std::result::Result<crate::shelves::DiscoverFeed, DiscoverError>
```



