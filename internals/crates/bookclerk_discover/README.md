# bookclerk_discover

Discovery: works graph, embeddings, recommendations, purchase hints, and requests.

## Audience

Host binaries (`bookclerkd`, CLI) that build Discover shelves, catalog search,
and wishlist ranking. Storefront plugins are not expected to depend on this
crate directly.

See `docs/` Discover / library product docs for operator-facing behavior.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Embedder`](#embedder) | trait | Produce dense vectors for recommendation / similarity search. |
| [`CandidateFetchOptions`](#candidatefetchoptions) | struct | Options for storefront candidate expansion. |
| [`SeedTitle`](#seedtitle) | struct | One taste seed for storefront expansion: a book the subject has, or a title on their wishlist. |
| [`StorefrontCandidate`](#storefrontcandidate) | struct | A purchase candidate discovered from a storefront catalog (not owned locally). |
| [`CatalogSearchFilters`](#catalogsearchfilters) | struct | Include / exclude filters applied after identity merge. |
| [`CatalogSearchHit`](#catalogsearchhit) | struct | One autocomplete / results-page hit (possibly spanning multiple storefronts). |
| [`CatalogSearchPage`](#catalogsearchpage) | struct | Paged catalog-search response. |
| [`CatalogSearchPageOpts`](#catalogsearchpageopts) | struct | Options for `catalog_search_page`. |
| [`OnnxEmbedder`](#onnxembedder) | struct | ONNX MiniLM embedder (fastembed). Loaded on demand; drop to free RAM. |
| [`CosineHit`](#cosinehit) | struct | One cosine-similarity neighbor from an embedding query. |
| [`HashEmbedder`](#hashembedder) | struct | Deterministic offline embedder for discovery similarity. |
| [`StoreEdition`](#storeedition) | struct | One storefront edition of a work (for multi-store purchase links). |
| [`WorkIdentity`](#workidentity) | struct | Bibliographic identity slice used for merge decisions. |
| [`MetadataEnrichOptions`](#metadataenrichoptions) | struct | Pacing for one enrichment run. |
| [`FetchReport`](#fetchreport) | struct | What one candidate fetch did. |
| [`OwnedIdentities`](#ownedidentities) | struct | Identifiers of every title the library holds, used to keep titles the library already has out of a subject's candidates and feed. |
| [`RankReport`](#rankreport) | struct | What one ranking did. |
| [`StoredSubjectFeed`](#storedsubjectfeed) | struct | One subject's stored feed, ready for the page. |
| [`SubjectTaste`](#subjecttaste) | struct | One subject's taste: whose seeds Discover uses and what they read. |
| [`PurchaseHint`](#purchasehint) | struct | A purchase / catalog availability hint (optionally priced at view time). |
| [`PurchaseHintsQuery`](#purchasehintsquery) | struct | Inputs for view-time catalog + pricing lookup. |
| [`PurchaseHintsResponse`](#purchasehintsresponse) | struct | Priced catalog matches for one title, sorted best-first. |
| [`RankedQueueEntry`](#rankedqueueentry) | struct | Global wishlist queue entry ranked by overall/operator recommend taste + heavy wish-count weight (shared order; not per-viewer personalized). |
| [`RecommendOptions`](#recommendoptions) | struct | Tunables for `recommend`. |
| [`Recommendation`](#recommendation) | struct | One ranked recommendation (typically an unowned storefront title). |
| [`StoredRequestQueue`](#storedrequestqueue) | struct | The shared request queue as a read builds it. |
| [`DiscoverFeed`](#discoverfeed) | struct | Full Discover page payload: ordered shelves of personalized candidates. |
| [`DiscoverShelf`](#discovershelf) | struct | One horizontal Discover row (series to finish, more by author, …). |
| [`ShelfKindInfo`](#shelfkindinfo) | struct | A shelf kind the operator can ignore (all offered by default). |
| [`ShelfTaste`](#shelftaste) | struct | Taste context used to title and fill personalized shelves. |
| [`TitleMeta`](#titlemeta) | struct | Bibliographic fields suitable for an Audible-style detail panel. |
| [`TitleMetaQuery`](#titlemetaquery) | struct | Query for public title metadata (Audnexus / Audible catalog). |
| [`TitleReview`](#titlereview) | struct | One Audible customer review for the title detail panel. |
| [`TitleReviewsPage`](#titlereviewspage) | struct | One page of title reviews for infinite scroll. |
| [`TitleReviewsQuery`](#titlereviewsquery) | struct | Query for a page of Audible customer reviews. |
| [`PooledEdition`](#poolededition) | struct | One store edition a wish names, as placed in the shared pool. |
| [`FeedState`](#feedstate) | enum | How current a stored Discover feed is. |
| [`gather_storefront_candidates`](#gather_storefront_candidates) | function | Expand storefront catalogs from local taste seeds; drop already-owned ids. |
| [`select_taste_seeds`](#select_taste_seeds) | function | Pick local seed books for storefront expansion (finished / high-rated first). |
| [`catalog_search`](#catalog_search) | function | Search every configured storefront catalog and merge by work identity. |
| [`catalog_search_page`](#catalog_search_page) | function | Paged multi-store catalog browse (filters + sort + cursor). |
| [`catalog_search_with_field`](#catalog_search_with_field) | function | Like `catalog_search`, optionally scoping storefront queries to a facet (author / narrator / series / genre). When `language` is set it is applied as a hard include filter (unknown/missing language still passes). |
| [`default_embedding_model_id`](#default_embedding_model_id) | function | Preferred model id for this build (ONNX MiniLM; runtime may fall back). |
| [`embed_dirty_works`](#embed_dirty_works) | function | Embed works whose text hash changed (or are missing), and drop the embeddings of works that no longer exist. |
| [`embedding_model_id`](#embedding_model_id) | function | Resolve the configured model id. |
| [`open_embedder`](#open_embedder) | function | Open an embedder for the current build / config. |
| [`text_for_work`](#text_for_work) | function | Build the text blob embedded for a work. |
| [`candidate_map_key`](#candidate_map_key) | function | Stable HashMap key for a storefront candidate (hard id, soft title+author, or source:product). |
| [`hard_work_key`](#hard_work_key) | function | Hard bibliographic key: canonical ISBN first, then ASIN. |
| [`identities_match`](#identities_match) | function | Whether two bibliographic identities refer to the same work. |
| [`merge_candidate_metadata`](#merge_candidate_metadata) | function | Merge storefront metadata into `into`, preferring filled fields / richer text. |
| [`merge_recommendation`](#merge_recommendation) | function | Merge a scored recommendation into an existing card. |
| [`parse_series_index`](#parse_series_index) | function | Parse Audible/Chirp-style series indexes (`"1"`, `"1.5"`, `"Book 2"`, `"02"`). |
| [`push_edition`](#push_edition) | function | Appends a store edition when not already present (dedupes on source+product_id). |
| [`push_shelf_category`](#push_shelf_category) | function | Append a stable shelf-kind tag (`finish_series`, `author`, …) if missing. |
| [`recommendation_map_key`](#recommendation_map_key) | function | Stable HashMap key for a recommendation card (same rules as `work_map_key`). |
| [`soft_work_key`](#soft_work_key) | function | Soft key from cleaned title + primary author (exact string match only). |
| [`work_map_key`](#work_map_key) | function | Best stable map key for a recommendation / candidate. |
| [`works_match`](#works_match) | function | Whether two titles should be treated as the same work. |
| [`enrich_books_from_metadata`](#enrich_books_from_metadata) | function | Fills missing description, subjects, cover and language from the deployed metadata providers. |
| [`fetch_subject_candidates`](#fetch_subject_candidates) | function | Fetches one subject's storefront candidates and stores them store by store. |
| [`load_subject_taste`](#load_subject_taste) | function | Loads one subject's taste. |
| [`rank_subject_feed`](#rank_subject_feed) | function | Ranks one subject's stored candidates into shelves and stores the feed. |
| [`stored_subject_feed`](#stored_subject_feed) | function | Reads one subject's stored feed, drops their hidden shelves and joins each card's catalog metadata from the candidate pool. |
| [`stored_subject_feed_state`](#stored_subject_feed_state) | function | Reads one subject's stored feed without joining its catalog metadata, for callers that need only whether it exists and when it was ranked. |
| [`best_purchase_hint`](#best_purchase_hint) | function | Pick the lowest-priced hint (ties keep earlier order). Unpriced sort after priced. |
| [`best_purchase_hint_preferring`](#best_purchase_hint_preferring) | function | Pick the offer the caller would actually pay least for. |
| [`format_money_label`](#format_money_label) | function | Formats integer cents into a storefront-style money label for `currency`. |
| [`parse_money_label_to_cents`](#parse_money_label_to_cents) | function | Parse `$12.34` / `12.34` / `FREE` into cents. |
| [`purchase_hints_for`](#purchase_hints_for) | function | Look up purchase links via registered `bookclerk_source::ContentSource::purchase_hint` (no prices). |
| [`resolve_purchase_hints`](#resolve_purchase_hints) | function | Resolve every catalog match and attach live prices (view-time). |
| [`resolve_purchase_hints_batch`](#resolve_purchase_hints_batch) | function | Resolve many purchase-hint queries at once (order preserved). |
| [`seed_purchase_hint`](#seed_purchase_hint) | function | Seed a deterministic storefront URL from a known candidate (no remote I/O). |
| [`combine_wishlist_score`](#combine_wishlist_score) | function | Combine local recommend taste with a heavy multi-user wishlist boost. |
| [`listening_engagement`](#listening_engagement) | function | Continuous engagement from a listening progress row (≈0.0–6.0). |
| [`rank_request_queue_taste`](#rank_request_queue_taste) | function | Ranks every entry of the shared request queue against **household** taste, for the `discover_request_queue` job to store. |
| [`recommend`](#recommend) | function | Build ranked recommendations for the operator (or a specific external user). |
| [`recommend_feed`](#recommend_feed) | function | Personalized Discover feed (Netflix-style shelves) from the same candidate pool. |
| [`queue_entry_key`](#queue_entry_key) | function | The key a queue entry's stored ranking is filed under: the library work its wishes are for. Every wish names a work, so one work is one entry whatever identifiers each wisher's store gave. |
| [`read_request_queue`](#read_request_queue) | function | Builds the ranked queue from the open requests and the stored ranking. |
| [`build_discover_feed`](#build_discover_feed) | function | Partitions scored recommendations into Discover shelves. |
| [`flatten_feed`](#flatten_feed) | function | Flatten shelves into a unique score-sorted list (CLI / legacy). |
| [`shelf_is_disabled`](#shelf_is_disabled) | function | Whether `shelf_id` matches an ignore entry (exact, kind prefix, or `from_store`). |
| [`shelf_kind_catalog`](#shelf_kind_catalog) | function | Shelf kinds Discover can emit (for config / UI ignore lists). |
| [`resolve_title_meta`](#resolve_title_meta) | function | Resolve public metadata for a title detail dialog. |
| [`resolve_title_meta_batch`](#resolve_title_meta_batch) | function | Resolve many title-meta queries with bounded concurrency (order preserved). |
| [`resolve_title_reviews`](#resolve_title_reviews) | function | Resolve one page of Audible customer reviews for infinite scroll. |
| [`pool_wish_editions`](#pool_wish_editions) | function | Asks each store about the editions a wish names and records the answers in the shared pool. |
| [`EMBED_PAGE_WORKS`](#embed_page_works) | constant | Works read per page by `embed_dirty_works`; also the most texts one embedder call receives. |
| [`MODEL_ALL_MINILM_L6_V2_Q`](#model_all_minilm_l6_v2_q) | constant | Quantized MiniLM-L6-v2 — ~22 MB ONNX, ~50 MB RAM, 384 dims. |
| [`MODEL_LOCAL_HASH_V1`](#model_local_hash_v1) | constant | Local hash embedder id used when ONNX is unavailable / disabled. |
| [`DEFAULT_MAX_REQUESTS_PER_RUN`](#default_max_requests_per_run) | constant | Default cap on provider asks in one run. |
| [`METADATA_ENRICH_SOURCE_PREFIX`](#metadata_enrich_source_prefix) | constant | `enrich_source` prefix marking a title filled by a metadata provider; the provider's plugin key follows it. |
| [`HOUSEHOLD_BLEND_WEIGHT`](#household_blend_weight) | constant | How much a household book counts in a member's taste while it blends in, relative to one of their own. |
| [`OWN_TASTE_THRESHOLD`](#own_taste_threshold) | constant | Books and listens of their own a subject needs before the household's books stop blending into their taste. |
| [`STARTER_GENRES`](#starter_genres) | constant | Genres browsed for a thin taste, in the order the stores are asked (the call budget may stop before the end). |
| [`STARTER_GENRE_MIN_SEEDS`](#starter_genre_min_seeds) | constant | Seeds below which a fetch also browses each store's `STARTER_GENRES`, so a new library still gets shelves. |
| [`STORE_CALL_MAX_AGE`](#store_call_max_age) | constant | How long a cached storefront answer is reused (the daily refresh). |
| [`STORE_CALL_TIMEOUT`](#store_call_timeout) | constant | Longest one storefront call may take before its store moves on. |
| [`STORE_FETCH_DEADLINE`](#store_fetch_deadline) | constant | How long one store may keep starting new calls in a fetch. |
| [`BATCH_DEADLINE`](#batch_deadline) | constant | Longest a batch waits for its lookups before answering with what has resolved. Under the daemon's 25 s route budget, so a slow store never turns a page of cards into one failed request. |
| [`BREAKER_OPEN_FOR`](#breaker_open_for) | constant | How long a tripped store is skipped. After that, lookups ask it again, and `BREAKER_TRIP_TIMEOUTS` more timeouts in a row pause it again. |
| [`BREAKER_TRIP_TIMEOUTS`](#breaker_trip_timeouts) | constant | Timeouts in a row after which a store's price lookups are skipped. |
| [`RATE_LIMIT_FIRST_PAUSE`](#rate_limit_first_pause) | constant | How long a store that asked to slow down without saying for how long is left alone the first time; each repeat doubles it. |
| [`RATE_LIMIT_MAX_PAUSE`](#rate_limit_max_pause) | constant | Longest pause a rate limit can grow to by doubling, and the most a store's own `Retry-After` is honored for. |
| [`WISH_COUNT_WEIGHT`](#wish_count_weight) | constant | Per open wish on the global queue. Large enough that multi-user demand dominates typical local taste scores (~1–25) while recommend signals still order titles that share the same wish count. |
| [`REQUEST_QUEUE_MAX_AGE`](#request_queue_max_age) | constant | How old the stored ranking may get before a read asks for a new pass, so taste changes no event announces (a book removed, a rating edited) still reach the queue within a day. |
| [`SERIES_CAP`](#series_cap) | constant | Most titles of one series on a shelf that is not about series, so one long series cannot crowd the others out. |
| [`MAX_WISH_EDITIONS`](#max_wish_editions) | constant | Most store editions looked up for one wish. A wish names a few; the cap bounds what a crafted request can make the server fetch. |

## Re-exports

- [`bookclerk_source::default_preferred_language`](../bookclerk_storefront/README.md#default_preferred_language)
- [`bookclerk_source::language_rank`](../bookclerk_storefront/README.md#language_rank)
- [`bookclerk_source::normalize_language`](../bookclerk_storefront/README.md#normalize_language)
- [`bookclerk_source::CatalogSearchField`](../bookclerk_storefront/README.md#catalogsearchfield)
- [`bookclerk_source::CatalogSearchSort`](../bookclerk_storefront/README.md#catalogsearchsort)
- [`bookclerk_source::CatalogSortDir`](../bookclerk_storefront/README.md#catalogsortdir)

## Traits

### Embedder

```rust
pub trait Embedder: Send {
    fn model_id(&self) -> &str;
    fn dimensions(&self) -> usize;
    fn embed(&mut self, texts: &[String]) -> Result<Vec<Vec<f32>>, DiscoverError>;
}
```

Produce dense vectors for recommendation / similarity search.

#### Embedder::model_id

```rust
fn model_id(&self) -> &str
```

Stable embedding model id stored alongside vectors in SQLite.

#### Embedder::dimensions

```rust
fn dimensions(&self) -> usize
```

Length of each embedding vector produced by this backend.

#### Embedder::embed

```rust
fn embed(&mut self, texts: &[String]) -> Result<Vec<Vec<f32>>, DiscoverError>
```

Embeds each input text into a dense float vector of [`Self::dimensions`](#embedderdimensions) length.

##### Errors

Returns an error when the operation fails.

Implemented by: `HashEmbedder`, `OnnxEmbedder`

## Structs

### CandidateFetchOptions

```rust
pub struct CandidateFetchOptions {
    pub region: String,
    pub seed_limit: usize,
    pub max_remote_calls: usize,
    pub include_audible: bool,
    pub include_libro: bool,
    pub include_chirp: bool,
    pub include_graphicaudio: bool,
    pub include_deals: bool,
    pub exclude_graphicaudio_series_sets: bool,
}
```

Options for storefront candidate expansion.

**Fields**

- `region`: Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `seed_limit`: Max local seed titles to expand from (finished / rated first).
- `max_remote_calls`: Cap remote HTTP calls across all storefronts.
- `include_audible`: Call `bookclerk_source::ContentSource::expand_candidates` on registered Audible.
- `include_libro`: Call `bookclerk_source::ContentSource::expand_candidates` on registered Libro.fm.
- `include_chirp`: Call `bookclerk_source::ContentSource::expand_candidates` on registered Chirp.
- `include_graphicaudio`: Call `bookclerk_source::ContentSource::expand_candidates` on registered GraphicAudio.
- `include_deals`: Fetch deals via `bookclerk_source::ContentSource::list_deals` on all registered sources.
- `exclude_graphicaudio_series_sets`: When true, drop GraphicAudio Magento series-set SKUs from candidates.
  Default is false (sets are kept).

Implements: `Clone`, `Debug`, `Default`

### SeedTitle

```rust
pub struct SeedTitle {
    pub seed: ExpandSeed,
    pub categories: Option<String>,
}
```

One taste seed for storefront expansion: a book the subject has, or a
title on their wishlist.

**Fields**

- `seed`: What the storefront is asked about.
- `categories`: The seed's genres or subjects, copied onto candidates that lack their own.

#### SeedTitle::from_book

```rust
pub fn from_book(book: &BookRecord, region: &str) -> Self
```

A seed from a library book.

#### SeedTitle::from_wish

```rust
pub fn from_wish(
    title: &str,
    authors: Option<String>,
    asin: Option<String>,
    isbn: Option<String>,
) -> Self
```

A seed from an open wishlist title (no store of its own; stores
expand it by title, author and identifiers).

Implements: `Clone`, `Debug`

### StorefrontCandidate

```rust
pub struct StorefrontCandidate {
    pub source: String,
    pub product_id: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub cover_url: Option<String>,
    pub seed_categories: Option<String>,
    pub origin: String,
    pub seed_title: Option<String>,
    pub starter_genre: Option<String>,
    pub store_editions: Vec<StoreEdition>,
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
    pub is_abridged: Option<bool>,
    pub store_rank: Option<u32>,
    pub work_id: Option<String>,
}
```

A purchase candidate discovered from a storefront catalog (not owned locally).

**Fields**

- `source`: Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).
- `product_id`: Storefront-native product id (ASIN, ISBN, UUID, …).
- `title`: Display title as shown on the storefront or library card.
- `authors`: Comma-separated author names when the storefront provides them.
- `narrators`: Comma-separated narrator names when known.
- `series`: Series name when the title belongs to a named series.
- `series_index`: Position within the series (e.g. `1`, `1.5`) when known.
- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url`: Public cover image URL when a storefront provided one.
- `seed_categories`: Categories/subjects copied from the taste seed that produced this hit.
- `origin`: How this candidate was found (related-to seed, author search, …).
- `seed_title`: Owned-library title that seeded this related / similar candidate.
- `starter_genre`: Starter genre this candidate was browsed from (display name), set
  only on hits of a starter-genre search for a thin taste.
- `store_editions`: Known storefront editions of this work (including the primary source).
- `subtitle`: Bibliographic extras from the storefront catalog payload (optional).
- `description`: Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher`: Publisher imprint as reported by the catalog.
- `length_minutes`: Audiobook runtime in whole minutes when known.
- `published_at`: Publication date string from the catalog (ISO or storefront format).
- `categories`: Genre / subject tags as a single delimited string when known.
- `language`: BCP-47 / storefront language code (`en`, `de`, …) when known.
- `price_cents`: List or deal price in integer cents of [`Self::currency`](#storefrontcandidate).
- `currency`: ISO 4217 currency code for price fields (`USD`, `EUR`, …).
- `price_label`: Human-readable price string from the storefront (e.g. `$14.95`).
- `rating_overall`: Community overall rating when a storefront provided one.
- `rating_count`: Number of ratings backing [`Self::rating_overall`](#storefrontcandidate) when known.
- `is_abridged`: Abridged flag when the storefront provided one.
- `store_rank`: The storefront's own 1-based search rank for this hit (its declared
  `CatalogHit.rankHint`); catalog search merges stores by it for
  relevance and popularity sorts. `None` when no store ranked it.
- `work_id`: The work the pool resolved this listing into, when the candidate was
  read from the shared listing pool. Lets the ranking drop titles the
  library holds by work identity without resolving the title again.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### CatalogSearchFilters

```rust
pub struct CatalogSearchFilters {
    pub authors: Vec<String>,
    pub narrators: Vec<String>,
    pub series: Vec<String>,
    pub genres: Vec<String>,
    pub sources: Vec<String>,
    pub exclude_sources: Vec<String>,
    pub languages: Vec<String>,
    pub exclude_narrators: Vec<String>,
    pub min_rating: Option<f64>,
    pub min_length_minutes: Option<i64>,
    pub max_length_minutes: Option<i64>,
}
```

Include / exclude filters applied after identity merge.

**Fields**

- `authors`: Comma-separated author names when the storefront provides them.
- `narrators`: Comma-separated narrator names when known.
- `series`: Series name when the title belongs to a named series.
- `genres`: Genre labels from the catalog (comma-separated when multiple).
- `sources`: Deduped storefront ids that contributed editions to this hit.
- `exclude_sources`: Store ids to drop (any edition matching). Empty = no exclude.
- `languages`: Normalized language codes (`en`, `zh`, …). When non-empty, only matching
  (or unknown/missing) languages pass.
- `exclude_narrators`: Substring match on narrator strings (case-insensitive), e.g. `Virtual Voice`.
- `min_rating`: Keep hits with `rating_overall >= min` or missing rating.
- `min_length_minutes`: Inclusive lower bound on `length_minutes`; missing length passes.
- `max_length_minutes`: Inclusive upper bound on `length_minutes`; missing length passes.

#### CatalogSearchFilters::is_empty

```rust
pub fn is_empty(&self) -> bool
```

Returns true when no include/exclude filters are set.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### CatalogSearchHit

```rust
pub struct CatalogSearchHit {
    pub work_key: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub cover_url: Option<String>,
    pub cover_path: Option<String>,
    pub store_editions: Vec<StoreEdition>,
    pub sources: Vec<String>,
    pub subtitle: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub genres: Option<String>,
    pub language: Option<String>,
    pub is_abridged: Option<bool>,
    pub rating_overall: Option<f64>,
    pub rating_count: Option<i64>,
    pub price_cents: Option<i64>,
    pub purchase_hints: Vec<PurchaseHint>,
}
```

One autocomplete / results-page hit (possibly spanning multiple storefronts).

**Fields**

- `work_key`: Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).
- `title`: Display title as shown on the storefront or library card.
- `authors`: Comma-separated author names when the storefront provides them.
- `narrators`: Comma-separated narrator names when known.
- `series`: Series name when the title belongs to a named series.
- `series_index`: Position within the series (e.g. `1`, `1.5`) when known.
- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url`: HTTPS URL for cover art when the catalog exposes one.
- `cover_path`: Same-origin path of this cover's processed images, set by the daemon
  when `cover_url` is an `https` URL. Append `/128`
  (thumbnail), `/360` (card) or `/full`, so a client can show covers
  without loading `cover_url` from another site.
- `store_editions`: Per-storefront edition ids that collapsed into this work card.
- `sources`: Storefronts that matched (deduped source ids).
- `subtitle`: Optional subtitle when the catalog distinguishes it from the title.
- `description`: Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher`: Publisher imprint as reported by the catalog.
- `length_minutes`: Audiobook runtime in whole minutes when known.
- `published_at`: Publication date string from the catalog (ISO or storefront format).
- `genres`: Genre labels from the catalog (comma-separated when multiple).
- `language`: BCP-47 / storefront language code (`en`, `de`, …) when known.
- `is_abridged`: When `Some(true)`, the edition is marked abridged by the storefront.
- `rating_overall`: Aggregate listener rating on the storefront's scale when known.
- `rating_count`: Number of ratings backing [`Self::rating_overall`](#catalogsearchhit) when known.
- `price_cents`: Catalog list/deal price in cents when the storefront provided it.
- `purchase_hints`: Buy / open-in-store links with optional member vs list pricing.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### CatalogSearchPage

```rust
pub struct CatalogSearchPage {
    pub items: Vec<CatalogSearchHit>,
    pub page_size: usize,
    pub has_more: bool,
    pub next_cursor: Option<String>,
    pub sort: String,
    pub sort_dir: String,
}
```

Paged catalog-search response.

**Fields**

- `items`: Ordered collection of child entries for this response.
- `page_size`: Maximum number of items returned in this page.
- `has_more`: When true, another page is available via [`Self::next_cursor`](#catalogsearchpage).
- `next_cursor`: Opaque cursor for the next page; omit or `None` when exhausted.
- `sort`: Sort field applied to the merged result set.
- `sort_dir`: Ascending or descending order for [`Self::sort`](#catalogsearchpage).

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### CatalogSearchPageOpts

```rust
pub struct CatalogSearchPageOpts<'a> {
    pub query: &'a str,
    pub region: &'a str,
    pub page_size: usize,
    pub cursor: Option<&'a str>,
    pub sort: CatalogSearchSort,
    pub sort_dir: CatalogSortDir,
    pub field: Option<CatalogSearchField>,
    pub language: Option<&'a str>,
    pub all_languages: bool,
    pub filters: CatalogSearchFilters,
}
```

Options for [`catalog_search_page`](#catalog_search_page).

**Fields**

- `query`: Free-text search string entered by the operator or SPA.
- `region`: Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `page_size`: Maximum number of items returned in this page.
- `cursor`: Opaque pagination token from a previous page response.
- `sort`: Sort field applied to the merged result set.
- `sort_dir`: Ascending or descending order for [`Self::sort`](#catalogsearchpageopts).
- `field`: Optional catalog field scope (`title`, `author`, …) when searching one facet.
- `language`: BCP-47 / storefront language code (`en`, `de`, …) when known.
- `all_languages`: When true, do not default hard language filter from [`Self::language`](#catalogsearchpageopts).
- `filters`: Server-side include/exclude filters applied after identity merge.

Implements: `Clone`, `Debug`

### OnnxEmbedder

```rust
pub struct OnnxEmbedder {
    /* private fields */
}
```

ONNX MiniLM embedder (fastembed). Loaded on demand; drop to free RAM.

Only built with the `onnx` feature (the hosts' `onnx-embeddings`).

#### OnnxEmbedder::open

```rust
pub fn open(cache_dir: &Path, intra_threads: usize) -> Result<Self, DiscoverError>
```

Download (if needed) and load quantized MiniLM into `cache_dir`.

##### Arguments

* `cache_dir` - Directory used to cache downloaded model weights.
* `intra_threads` - ONNX intra-op thread count (clamped to at least 1).

##### Returns

On success, the inner `Self` value.

##### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

Implements: `Embedder`

### CosineHit

```rust
pub struct CosineHit {
    pub target_id: String,
    pub score: f32,
}
```

One cosine-similarity neighbor from an embedding query.

**Fields**

- `target_id`: Library entity id of the similar work (`works.id`).
- `score`: Ranking score (higher is better); units are algorithm-specific.

Implements: `Clone`, `Debug`

### HashEmbedder

```rust
pub struct HashEmbedder {
    /* private fields */
}
```

Deterministic offline embedder for discovery similarity.

Not semantic like MiniLM — used when ONNX fails to load or when
`prefer_onnx` is false. Still enables the full pipeline (hash → store →
cosine) for CI and constrained hosts.

#### HashEmbedder::new

```rust
pub fn new(dims: usize) -> Self
```

Creates a hash embedder with the given vector length (clamped to 8..=384).

##### Arguments

* `dims` - Desired embedding length before clamping.

##### Returns

Embedder that maps text to deterministic unit vectors of length `dims`.

Implements: `Debug`, `Default`, `Embedder`

### StoreEdition

```rust
pub struct StoreEdition {
    pub source: String,
    pub product_id: String,
}
```

One storefront edition of a work (for multi-store purchase links).

**Fields**

- `source`: Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).
- `product_id`: Storefront-native product id (ASIN, ISBN, UUID, …).

#### StoreEdition::new

```rust
pub fn new(source: impl Into<String>, product_id: impl Into<String>) -> Self
```

Creates a store edition from a storefront id and native product id.

##### Arguments

* `source` - Storefront id or filesystem source path, depending on call site.
* `product_id` - Storefront-native product id.

##### Returns

Newly constructed `new` value.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`, `ToSchema`

### WorkIdentity

```rust
pub struct WorkIdentity<'a> {
    pub asin: Option<&'a str>,
    pub isbn: Option<&'a str>,
    pub title: &'a str,
    pub authors: Option<&'a str>,
    pub series: Option<&'a str>,
    pub series_index: Option<&'a str>,
}
```

Bibliographic identity slice used for merge decisions.

**Fields**

- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `title`: Display title as shown on the storefront or library card.
- `authors`: Comma-separated author names when the storefront provides them.
- `series`: Series name when the title belongs to a named series.
- `series_index`: Position within the series (e.g. `1`, `1.5`) when known.

#### WorkIdentity::new

```rust
pub fn new(
    asin: Option<&'a str>,
    isbn: Option<&'a str>,
    title: &'a str,
    authors: Option<&'a str>,
) -> Self
```

Creates a work identity from hard ids plus title and authors (no series yet).

##### Arguments

* `asin` - Optional Audible / Amazon ASIN.
* `isbn` - Optional ISBN (any punctuation; normalized internally).
* `title` - Display title.
* `authors` - Optional author string (comma-separated).

##### Returns

Newly constructed `new` value.

#### WorkIdentity::with_series

```rust
pub fn with_series(self, series: Option<&'a str>) -> Self
```

Sets the optional series name used in soft identity matching.

##### Arguments

* `series` - Optional series name.

##### Returns

Updated `Self` for chaining.

#### WorkIdentity::with_series_index

```rust
pub fn with_series_index(self, series_index: Option<&'a str>) -> Self
```

Sets the optional series position used to avoid merging distinct volumes.

##### Arguments

* `series_index` - Optional series position string.

##### Returns

Updated `Self` for chaining.

Implements: `Clone`, `Copy`, `Debug`

### MetadataEnrichOptions

```rust
pub struct MetadataEnrichOptions {
    pub min_interval: Duration,
    pub max_requests: usize,
}
```

Pacing for one enrichment run.

**Fields**

- `min_interval`: Pause before each provider ask.
- `max_requests`: Most provider asks in one run; titles past the cap wait for the next run.

Implements: `Clone`, `Debug`, `Default`

### FetchReport

```rust
pub struct FetchReport {
    pub seeds: usize,
    pub stores_saved: usize,
    pub candidates: usize,
    pub stores_failed: Vec<String>,
    pub priced: usize,
}
```

What one candidate fetch did.

**Fields**

- `seeds`: Seeds the subject's taste offered (books, then wishlist titles).
- `stores_saved`: Stores whose answers were stored.
- `candidates`: Candidates stored across those stores.
- `stores_failed`: Stores whose fetch failed outright, by plugin id.
- `priced`: Candidates that carry a price from the fetch, across stores.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### OwnedIdentities

```rust
pub struct OwnedIdentities {
    pub asins: HashSet<String>,
    pub isbns: HashSet<String>,
    pub products: HashSet<String>,
}
```

Identifiers of every title the library holds, used to keep titles the
library already has out of a subject's candidates and feed.

**Fields**

- `asins`: Upper-cased ASINs.
- `isbns`: Canonical ISBN-13s.
- `products`: `source:product_id` keys and bare product ids.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### RankReport

```rust
pub struct RankReport {
    pub scope: SeedScope,
    pub shelves: usize,
    pub candidates: usize,
    pub cards: usize,
    pub stored_bytes: usize,
}
```

What one ranking did.

**Fields**

- `scope`: Seeds the feed was ranked against.
- `shelves`: Shelves stored.
- `candidates`: Stored candidates read.
- `cards`: Cards on the stored shelves, after any trimming to fit the budget.
- `stored_bytes`: Bytes of the stored feed as the database counts it (an escaped JSON
  string), at most the 192 KiB stored-feed budget.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### StoredSubjectFeed

```rust
pub struct StoredSubjectFeed {
    pub feed: DiscoverFeed,
    pub scope: SeedScope,
    pub computed_at: DateTime<Utc>,
    pub candidates_fetched_at: Option<DateTime<Utc>>,
}
```

One subject's stored feed, ready for the page.

**Fields**

- `feed`: The shelves, without the subject's hidden ones.
- `scope`: Seeds the feed was ranked from.
- `computed_at`: When it was ranked.
- `candidates_fetched_at`: Newest storefront fetch it used, when any.

Implements: `Clone`, `Debug`

### SubjectTaste

```rust
pub struct SubjectTaste {
    pub scope: SeedScope,
    pub books: Vec<BookRecord>,
    pub weights: HashMap<String, f64>,
    pub listening: Vec<ListeningSignal>,
    pub wishlist: Vec<SeedTitle>,
    pub wish_titles: Vec<String>,
    pub linked_sources: HashSet<String>,
    pub owned: OwnedIdentities,
}
```

One subject's taste: whose seeds Discover uses and what they read.

**Fields**

- `scope`: Whether the taste is the subject's own or the household's.
- `books`: Books whose finishes, ratings, series and authors shape the taste:
  the subject's own, then the household's blended in while the
  subject's own are fewer than [`OWN_TASTE_THRESHOLD`](#own_taste_threshold).
- `weights`: How much each book in `books` counts; a blended household book is
  listed at [`HOUSEHOLD_BLEND_WEIGHT`](#household_blend_weight), the subject's own count in full.
- `listening`: The subject's listening rows (all rows for the household). While
  household books blend in, the household's rows for those books are
  added too, and count at their book's weight.
- `wishlist`: The subject's open wishlist titles, newest first, used as extra seeds.
- `wish_titles`: Display titles of `wishlist`, newest first.
- `linked_sources`: Lowercase storefront ids the subject has an account with.
- `owned`: Identifiers of every title the library holds, read with the taste so
  a subject job reads the library once.

Implements: `Clone`, `Debug`

### PurchaseHint

```rust
pub struct PurchaseHint {
    pub source: String,
    pub product_id: String,
    pub title: Option<String>,
    pub url: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub list_price_cents: Option<i64>,
    pub list_price_label: Option<String>,
    pub member_price_cents: Option<i64>,
    pub member_price_label: Option<String>,
}
```

A purchase / catalog availability hint (optionally priced at view time).

**Fields**

- `source`: Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).
- `product_id`: Storefront-native product id (ASIN, ISBN, UUID, …).
- `title`: Display title as shown on the storefront or library card.
- `url`: Absolute HTTPS URL.
- `price_cents`: Primary / best known sell price in minor units (prefer member when dual).
  `0` = free.
- `currency`: ISO 4217 currency code for price fields (`USD`, `EUR`, …).
- `price_label`: Display string from the store (`$2.99`, `FREE`, …).
- `list_price_cents`: Non-member / list price in integer cents when known.
- `list_price_label`: Human-readable list price from the storefront.
- `member_price_cents`: Member / credit price in integer cents when known.
- `member_price_label`: Human-readable member price from the storefront.

#### PurchaseHint::link

```rust
pub fn link(
    source: impl Into<String>,
    product_id: impl Into<String>,
    title: Option<String>,
    url: Option<String>,
) -> Self
```

URL-only catalog link (no price yet).

##### Arguments

* `source` - Storefront id or filesystem source path, depending on call site.
* `product_id` - Storefront-native product id.
* `title` - Display title.
* `url` - Absolute URL being checked or opened.

##### Returns

Updated `Self` for chaining.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### PurchaseHintsQuery

```rust
pub struct PurchaseHintsQuery {
    pub title: String,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub candidate_source: Option<String>,
    pub candidate_product_id: Option<String>,
    pub store_editions: Vec<StoreEdition>,
    pub region: Option<String>,
    pub preferred_sources: Vec<String>,
}
```

Inputs for view-time catalog + pricing lookup.

**Fields**

- `title`: Display title as shown on the storefront or library card.
- `authors`: Comma-separated author names when the storefront provides them.
- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `candidate_source`: Storefront id of the edition that produced this card.
- `candidate_product_id`: Storefront product id of the edition that produced this card.
- `store_editions`: Known storefront editions already on the recommendation card.
- `region`: Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `preferred_sources`: Storefronts the caller has linked accounts for. Treated as **member**
  pricing when picking `best`; other stores are compared at list /
  non-member price. Every catalog match is still returned in `hints`.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### PurchaseHintsResponse

```rust
pub struct PurchaseHintsResponse {
    pub hints: Vec<PurchaseHint>,
    pub best: Option<PurchaseHint>,
    pub incomplete: bool,
}
```

Priced catalog matches for one title, sorted best-first.

**Fields**

- `hints`: Resolved purchase hints for this query.
- `best`: Lowest-priced hint (or first catalog hit when no prices resolved).
- `incomplete`: True when the batch deadline cut this lookup short: the hints are the
  query's trusted seeds alone, so a client should not keep the answer
  and asks again later. Omitted (false) for a finished lookup.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### RankedQueueEntry

```rust
pub struct RankedQueueEntry {
    pub work_key: String,
    pub work_id: String,
    pub title: String,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub cover_url: Option<String>,
    pub cover_path: Option<String>,
    pub wish_count: i64,
    pub wishers: Vec<QueueWisher>,
    pub sample_uuids: Vec<String>,
    pub first_requested_at: DateTime<Utc>,
    pub last_requested_at: DateTime<Utc>,
    pub score: f64,
    pub taste_score: f64,
    pub reasons: Vec<String>,
}
```

Global wishlist queue entry ranked by overall/operator recommend taste +
heavy wish-count weight (shared order; not per-viewer personalized).

**Fields**

- `work_key`: Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).
- `work_id`: The work every wish of this entry is for. Wishes for one work are
  one entry even when their `work_key`s differ, so a client matches
  its own wishes on this id first and on `work_key` otherwise.
- `title`: Display title as shown on the storefront or library card.
- `authors`: Comma-separated author names when the storefront provides them.
- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url`: HTTPS URL for cover art when the catalog exposes one.
- `cover_path`: Same-origin path of this cover's processed images, set by the daemon
  when `cover_url` is an `https` URL. Append `/128`
  (thumbnail), `/360` (card) or `/full`, so a client can show covers
  without loading `cover_url` from another site.
- `wish_count`: Number of distinct users who wishlisted this title.
- `wishers`: Distinct people who have this title on an open wishlist.
- `sample_uuids`: Sample Bookclerk user UUIDs who requested this title (privacy-capped).
- `first_requested_at`: UTC timestamp of the earliest wishlist request for this work.
- `last_requested_at`: UTC timestamp of the latest wishlist request for this work.
- `score`: Final rank score (`taste_score + wish_count * `[`WISH_COUNT_WEIGHT`](#wish_count_weight)).
- `taste_score`: Local taste / embedding component before the wish-count boost.
- `reasons`: Short human-readable explanations for why this item was ranked here.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### RecommendOptions

```rust
pub struct RecommendOptions {
    pub limit: usize,
    pub embedding_model: String,
    pub region: String,
    pub include_purchase_hints: bool,
    pub external_user_id: Option<String>,
    pub include_listening: bool,
    pub listening_providers: Vec<String>,
    pub fetch_storefront_candidates: bool,
    pub storefront_seed_limit: usize,
    pub storefront_max_remote_calls: usize,
    pub exclude_graphicaudio_series_sets: bool,
    pub disabled_shelves: Vec<String>,
    pub models_dir: Option<PathBuf>,
    pub embed_intra_threads: usize,
    pub embeddings_enabled: bool,
}
```

Tunables for [`recommend`](#recommend).

**Fields**

- `limit`: Maximum number of recommendations to return.
- `embedding_model`: Embedding model id used for similarity (`all-minilm-l6-v2-q`, …).
- `region`: Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `include_purchase_hints`: When true, attach buy links to each recommendation.
- `external_user_id`: When set, only listening rows for this external user influence ranking.
  Provider-agnostic — any integration that synced that user id.
- `include_listening`: When false, ignore `listening_progress` entirely (owned-library taste only).
  Listening is always optional: empty/missing progress simply adds no signal.
- `listening_providers`: When non-empty, only use listening rows from these integration ids
  (`audiobookshelf`, plugin ids, …). Empty = all providers.
- `fetch_storefront_candidates`: Pull unowned titles from storefront catalogs (the primary path).
- `storefront_seed_limit`: Max owned titles used as storefront related-search seeds.
- `storefront_max_remote_calls`: Hard cap on outbound catalog calls during recommend.
- `exclude_graphicaudio_series_sets`: Drop GraphicAudio Magento series-set SKUs from discovery candidates.
- `disabled_shelves`: Shelf kinds / ids to hide (`finish_series`, `author`, `genre`, `from_store`, …).
  Empty = offer every shelf Discover can build.
- `models_dir`: Where the ONNX model lives. `Some` means candidates are embedded with
  ONNX MiniLM when it loads (local-hash when it doesn't); `None` means
  local-hash only, without trying ONNX.
- `embed_intra_threads`: ONNX intra-op thread count for the embedder (clamped ≥ 1).
- `embeddings_enabled`: When false, skip embedding similarity and use heuristic ranking only.
  Which embedder scores similarity is chosen by [`Self::models_dir`](#recommendoptions).

Implements: `Clone`, `Debug`, `Default`

### Recommendation

```rust
pub struct Recommendation {
    pub work_id: Option<String>,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub score: f64,
    pub reasons: Vec<String>,
    pub purchase_hints: Vec<PurchaseHint>,
    pub from_request: bool,
    pub request_uuid: Option<String>,
    pub candidate_source: Option<String>,
    pub candidate_product_id: Option<String>,
    pub store_editions: Vec<StoreEdition>,
    pub seed_categories: Option<String>,
    pub categories: Vec<String>,
    pub work_key: String,
    pub cover_url: Option<String>,
    pub cover_path: Option<String>,
    pub subtitle: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub genres: Option<String>,
    pub language: Option<String>,
    pub wishers: Vec<QueueWisher>,
    pub wish_count: i64,
    pub seed_title: Option<String>,
    pub starter_genre: Option<String>,
}
```

One ranked recommendation (typically an unowned storefront title).

**Fields**

- `work_id`: Library `works.id` UUID when the recommendation is tied to an owned work.
- `title`: Display title as shown on the storefront or library card.
- `authors`: Comma-separated author names when the storefront provides them.
- `narrators`: Comma-separated narrator names when known.
- `series`: Series name when the title belongs to a named series.
- `series_index`: Position within the series (e.g. `1`, `1.5`) when known.
- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `score`: Ranking score (higher is better); units are algorithm-specific.
- `reasons`: Short human-readable explanations for why this item was ranked here.
- `purchase_hints`: Buy / open-in-store links with optional member vs list pricing.
- `from_request`: True when sourced from an open title request rather than storefront discovery.
- `request_uuid`: Wishlist / title-request UUID when this card came from a request.
- `candidate_source`: Storefront that proposed this title (`audible`, `libro`, …).
- `candidate_product_id`: Storefront product id of the edition that produced this card.
- `store_editions`: All known storefront editions (for multi-store purchase links).
- `seed_categories`: Categories/subjects copied from the taste seed that produced this hit.
- `categories`: Stable shelf-kind tags for prefs (`finish_series`, `author`, `chirp_deals`, …).
  Dynamic shelf titles stay in `build_discover_feed`; membership uses these.
- `work_key`: Stable bibliographic key for wishlist / merge (`isbn:` / `asin:` / `soft:`…).
- `cover_url`: Cover art URL when known from a storefront catalog hit or wishlist row.
- `cover_path`: Same-origin path of this cover's processed images, set by the daemon
  when `cover_url` is an `https` URL. Append `/128`
  (thumbnail), `/360` (card) or `/full`, so a client can show covers
  without loading `cover_url` from another site.
- `subtitle`: Optional bibliographic extras from storefront catalog / Audnexus.
- `description`: Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher`: Publisher imprint as reported by the catalog.
- `length_minutes`: Audiobook runtime in whole minutes when known.
- `published_at`: Publication date string from the catalog (ISO or storefront format).
- `genres`: Candidate genres (not taste-seed `seed_categories`).
- `language`: BCP-47 / storefront language code (`en`, `de`, …) when known.
- `wishers`: Distinct people who have this title on an open wishlist.
- `wish_count`: Number of distinct users who wishlisted this title (may exceed `wishers.len()`).
- `seed_title`: Title of the book or wish the storefront was asked about to find this
  card, when it came from such an expansion.
- `starter_genre`: Starter genre this card was browsed from while the taste was thin.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### StoredRequestQueue

```rust
pub struct StoredRequestQueue {
    pub entries: Vec<RankedQueueEntry>,
    pub stale: bool,
}
```

The shared request queue as a read builds it.

**Fields**

- `entries`: Open entries the library does not hold, best first.
- `stale`: True when the stored ranking is missing an entry or older than
  [`REQUEST_QUEUE_MAX_AGE`](#request_queue_max_age), so the caller should queue a ranking pass.

Implements: `Clone`, `Debug`, `Default`

### DiscoverFeed

```rust
pub struct DiscoverFeed {
    pub shelves: Vec<DiscoverShelf>,
    pub shelf_kinds: Vec<ShelfKindInfo>,
    pub state: Option<FeedState>,
    pub computed_at: Option<String>,
}
```

Full Discover page payload: ordered shelves of personalized candidates.

**Fields**

- `shelves`: Ordered horizontal shelves that make up the Discover page.
- `shelf_kinds`: Catalog of shelf kinds for ignore prefs (empty `disabled_shelves` = all on).
- `state`: Whether these are the caller's current shelves, set by the daemon's
  read of the stored feed. Absent in a stored feed and from the CLI.
- `computed_at`: RFC 3339 time the shelves were ranked; absent while `state` is `preparing`.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### DiscoverShelf

```rust
pub struct DiscoverShelf {
    pub id: String,
    pub title: String,
    pub subtitle: Option<String>,
    pub items: Vec<Recommendation>,
}
```

One horizontal Discover row (series to finish, more by author, …).

**Fields**

- `id`: Stable identifier for this item.
- `title`: Display title as shown on the storefront or library card.
- `subtitle`: Optional subtitle when the catalog distinguishes it from the title.
- `items`: Ordered collection of child entries for this response.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### ShelfKindInfo

```rust
pub struct ShelfKindInfo {
    pub id: String,
    pub label: String,
}
```

A shelf kind the operator can ignore (all offered by default).

**Fields**

- `id`: Stable kind id (`finish_series`, `author`, `genre`, `from_store`, …).
- `label`: Operator-visible label for this account or item.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### ShelfTaste

```rust
pub struct ShelfTaste {
    pub authors: HashMap<String, (String, f64)>,
    pub narrators: HashMap<String, (String, f64)>,
    pub categories: HashMap<String, (String, f64)>,
    pub seed_authors_by_title: HashMap<String, Vec<String>>,
    pub owned_sources: HashSet<String>,
    pub wish_titles: Vec<String>,
    pub linked_sources: HashSet<String>,
}
```

Taste context used to title and fill personalized shelves.

**Fields**

- `authors`: Lowercase author → (display name, weight).
- `narrators`: Lowercase narrator → (display name, weight).
- `categories`: Lowercase category/subject → (display name, weight).
- `seed_authors_by_title`: Seed title (lowercase) → authors on that seed (display).
- `owned_sources`: Storefronts represented in the owned library (`audible`, `libro`, …).
- `wish_titles`: Titles on the subject's own open wishlist, newest first, that name
  the "Because you wishlisted" shelves.
- `linked_sources`: Lowercase storefront ids the subject has an account with, compared
  at member price when a card is placed on a store shelf.

Implements: `Clone`, `Debug`, `Default`

### TitleMeta

```rust
pub struct TitleMeta {
    pub asin: Option<String>,
    pub title: Option<String>,
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub isbn: Option<String>,
    pub cover_url: Option<String>,
    pub cover_path: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub is_abridged: Option<bool>,
    pub rating_overall: Option<f64>,
    pub rating_performance: Option<f64>,
    pub rating_story: Option<f64>,
    pub rating_count: Option<i64>,
    pub review_count: Option<i64>,
    pub reviews: Vec<TitleReview>,
}
```

Bibliographic fields suitable for an Audible-style detail panel.

**Fields**

- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `title`: Display title as shown on the storefront or library card.
- `subtitle`: Optional subtitle when the catalog distinguishes it from the title.
- `authors`: Comma-separated author names when the storefront provides them.
- `narrators`: Comma-separated narrator names when known.
- `series`: Series name when the title belongs to a named series.
- `series_index`: Position within the series (e.g. `1`, `1.5`) when known.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url`: HTTPS URL for cover art when the catalog exposes one.
- `cover_path`: Same-origin path of this cover's processed images, set by the daemon
  when `cover_url` is an `https` URL. Append `/128`
  (thumbnail), `/360` (card) or `/full`, so a client can show covers
  without loading `cover_url` from another site.
- `description`: Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher`: Publisher imprint as reported by the catalog.
- `length_minutes`: Audiobook runtime in whole minutes when known.
- `published_at`: Publication date string from the catalog (ISO or storefront format).
- `categories`: Genre / subject tags as a single delimited string when known.
- `language`: BCP-47 / storefront language code (`en`, `de`, …) when known.
- `is_abridged`: `Some(true)` abridged / `Some(false)` unabridged when the storefront said so.
- `rating_overall`: Audible community overall rating (0–5).
- `rating_performance`: Narration / performance sub-score when the storefront splits ratings.
- `rating_story`: Story / content sub-score when the storefront splits ratings.
- `rating_count`: Count of star ratings (Audible `num_ratings`).
- `review_count`: Count of written reviews (Audible `num_reviews`).
- `reviews`: Deprecated: reviews load via `/api/discover/title-reviews`. Kept empty
  for older clients; prefer the paginated reviews endpoint.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### TitleMetaQuery

```rust
pub struct TitleMetaQuery {
    pub title: String,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub narrators: Option<String>,
    pub length_minutes: Option<i64>,
    pub region: String,
}
```

Query for public title metadata (Audnexus / Audible catalog).

**Fields**

- `title`: Display title as shown on the storefront or library card.
- `authors`: Comma-separated author names when the storefront provides them.
- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `narrators`: Comma-separated narrator names when known.
- `length_minutes`: Audiobook runtime in whole minutes when known.
- `region`: Marketplace / region code (`us`, `uk`, …) for catalog lookups.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### TitleReview

```rust
pub struct TitleReview {
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

One Audible customer review for the title detail panel.

**Fields**

- `id`: Stable identifier for this item.
- `title`: Display title as shown on the storefront or library card.
- `body`: Full review text from the storefront.
- `author_name`: Display name of the review author on the storefront.
- `overall_rating`: Overall star rating for this review when present.
- `performance_rating`: Narration sub-rating for this review when present.
- `story_rating`: Story sub-rating for this review when present.
- `submitted_at`: When the review was submitted (storefront timestamp string).

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `From<CatalogReview>`, `Serialize`, `ToSchema`

### TitleReviewsPage

```rust
pub struct TitleReviewsPage {
    pub asin: String,
    pub page: u32,
    pub page_size: u32,
    pub has_more: bool,
    pub sort_by: String,
    pub reviews: Vec<TitleReview>,
}
```

One page of title reviews for infinite scroll.

**Fields**

- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `page`: 1-based page index for this reviews / list request.
- `page_size`: Maximum number of items returned in this page.
- `has_more`: When true, another page is available by incrementing [`Self::page`](#titlereviewspage).
- `sort_by`: Remote sort key for reviews (`MostHelpful` or `MostRecent`, as applied).
- `reviews`: Review rows on this page.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### TitleReviewsQuery

```rust
pub struct TitleReviewsQuery {
    pub asin: String,
    pub region: String,
    pub page: u32,
    pub page_size: u32,
    pub sort_by: String,
}
```

Query for a page of Audible customer reviews.

**Fields**

- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `region`: Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `page`: 1-based page index.
- `page_size`: Page size (clamped server-side to 1..=20).
- `sort_by`: `MostHelpful` (default) or `MostRecent`.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### PooledEdition

```rust
pub struct PooledEdition {
    pub source: String,
    pub product_id: String,
    pub listing_id: String,
    pub work: Option<WishWork>,
}
```

One store edition a wish names, as placed in the shared pool.

**Fields**

- `source`: Storefront plugin id the wish named (`libro`), lower-cased.
- `product_id`: The store's product id the wish named.
- `listing_id`: The `store_listings` row the store's answer resolved to.
- `work`: The work and edition that listing resolved to, when it has a work.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### FeedState

```rust
pub enum FeedState {
    Ready,
    Refreshing,
    Preparing,
}
```

How current a stored Discover feed is.

**Variants**

- `Ready`: The stored shelves are current.
- `Refreshing`: Stored shelves are shown while a newer fetch or ranking is queued.
- `Preparing`: No shelves are stored yet; a fetch is queued and the page shows placeholders.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

## Functions

### gather_storefront_candidates

```rust
pub async fn gather_storefront_candidates(
    _library: &LibraryStore,
    registry: &SourceRegistry,
    seeds: &[BookRecord],
    owned_asins: &HashSet<String>,
    owned_isbns: &HashSet<String>,
    owned_product_keys: &HashSet<String>,
    opts: &CandidateFetchOptions,
) -> Result<Vec<StorefrontCandidate>, DiscoverError>
```

Expand storefront catalogs from local taste seeds; drop already-owned ids.

#### Arguments

* `library` - Open library store used for reads/writes.
* `registry` - Configured content-source or integration registry.
* `seeds` - `seeds` input for this call.
* `owned_asins` - String `owned_asins` for this call.
* `owned_isbns` - String `owned_isbns` for this call.
* `owned_product_keys` - String `owned_product_keys` for this call.
* `opts` - Options struct for this operation.

#### Returns

On success, the inner `Vec<StorefrontCandidate>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### select_taste_seeds

```rust
pub fn select_taste_seeds(
    books: &[BookRecord],
    listening_engagement_by_uuid: &HashMap<String, f64>,
) -> Vec<BookRecord>
```

Pick local seed books for storefront expansion (finished / high-rated first).

#### Arguments

* `books` - `books` input for this call.
* `listening_engagement_by_uuid` - String `listening_engagement_by_uuid` for this call.

#### Returns

Collected results (may be empty).

### catalog_search

```rust
pub async fn catalog_search(
    registry: &SourceRegistry,
    query: &str,
    region: &str,
    limit: usize,
) -> Result<Vec<CatalogSearchHit>, DiscoverError>
```

Search every configured storefront catalog and merge by work identity.

#### Arguments

* `registry` - Configured content-source or integration registry.
* `query` - Query vector or free-text search string.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `limit` - Maximum number of results to return.

#### Returns

On success, the inner `Vec<CatalogSearchHit>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### catalog_search_page

```rust
pub async fn catalog_search_page(
    registry: &SourceRegistry,
    opts: CatalogSearchPageOpts<'_>,
) -> Result<CatalogSearchPage, DiscoverError>
```

Paged multi-store catalog browse (filters + sort + cursor).

#### Arguments

* `registry` - Configured content-source or integration registry.
* `opts` - Options struct for this operation.

#### Returns

On success, the inner `CatalogSearchPage` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### catalog_search_with_field

```rust
pub async fn catalog_search_with_field(
    registry: &SourceRegistry,
    query: &str,
    region: &str,
    limit: usize,
    field: Option<CatalogSearchField>,
    language: Option<&str>,
) -> Result<Vec<CatalogSearchHit>, DiscoverError>
```

Like [`catalog_search`](#catalog_search), optionally scoping storefront queries to a facet
(author / narrator / series / genre). When `language` is set it is applied
as a hard include filter (unknown/missing language still passes).

#### Arguments

* `registry` - Configured content-source or integration registry.
* `query` - Query vector or free-text search string.
* `region` - Marketplace / region code (`us`, `uk`, …).
* `limit` - Maximum number of results to return.
* `field` - Optional catalog search field scope.
* `language` - Optional preferred language code.

#### Returns

On success, the inner `Vec<CatalogSearchHit>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### default_embedding_model_id

```rust
pub fn default_embedding_model_id() -> &'static str
```

Preferred model id for this build (ONNX MiniLM; runtime may fall back).

### embed_dirty_works

```rust
pub async fn embed_dirty_works(
    library: &LibraryStore,
    embedder: &mut dyn Embedder,
) -> Result<usize, DiscoverError>
```

Embed works whose text hash changed (or are missing), and drop the
embeddings of works that no longer exist.

Walks `works` in keyset pages of [`EMBED_PAGE_WORKS`]: each page costs
one read of the works, one chunked read of their stored hashes, and one
embedder call for the dirty ones, so memory stays bounded by the page
and the store sees no per-work lookup.

#### Arguments

* `library` - Open library store used for reads/writes.
* `embedder` - Embedding backend that produces dense vectors.

#### Returns

How many embeddings were written.

#### Errors

Returns an error when a store read or write fails, or when the
embedder fails or returns a different number of vectors than texts.

### embedding_model_id

```rust
pub fn embedding_model_id(configured: &str) -> &str
```

Resolve the configured model id.

#### Arguments

* `configured` - Configured model id from config/env; empty selects the build default.

#### Returns

String result for this operation.

### open_embedder

```rust
pub fn open_embedder(
    models_dir: &Path,
    intra_threads: usize,
    prefer_onnx: bool,
) -> Result<Box<dyn Embedder>, DiscoverError>
```

Open an embedder for the current build / config.

When `prefer_onnx` is true, tries quantized MiniLM via fastembed/ort and
**warns + falls back** to the local-hash embedder if load/download fails
(missing glibc symbols, offline host, corrupt cache, …).

#### Arguments

* `models_dir` - Directory that holds (or will hold) embedding model files.
* `intra_threads` - ONNX intra-op thread count (clamped to at least 1).
* `prefer_onnx` - When true, try MiniLM ONNX first and fall back to hash embeddings on failure.

#### Returns

On success, the inner `Box<dyn Embedder>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### text_for_work

```rust
pub fn text_for_work(work: &WorkRecord) -> String
```

Build the text blob embedded for a work.

#### Arguments

* `work` - Library work row whose metadata is turned into embedding text.

#### Returns

String result for this operation.

### candidate_map_key

```rust
pub fn candidate_map_key(c: &StorefrontCandidate) -> String
```

Stable HashMap key for a storefront candidate (hard id, soft title+author, or source:product).

#### Arguments

* `c` - `c` input for this call.

#### Returns

String result for this operation.

### hard_work_key

```rust
pub fn hard_work_key(asin: Option<&str>, isbn: Option<&str>) -> Option<String>
```

Hard bibliographic key: canonical ISBN first, then ASIN.

Note: ISBN is not published by every storefront (Chirp / GraphicAudio /
Audible public search often omit it). Soft title+author matching then applies.

#### Arguments

* `asin` - Optional Audible / Amazon ASIN.
* `isbn` - Optional ISBN (any punctuation; normalized internally).

#### Returns

`Some(...)` when found / applicable; otherwise `None`.

### identities_match

```rust
pub fn identities_match(a: WorkIdentity<'_>, b: WorkIdentity<'_>) -> bool
```

Whether two bibliographic identities refer to the same work.

#### Arguments

* `a` - Left-hand vector.
* `b` - Right-hand vector.

#### Returns

`true` when the predicate holds.

### merge_candidate_metadata

```rust
pub fn merge_candidate_metadata(
    into: &mut StorefrontCandidate,
    from: &StorefrontCandidate,
)
```

Merge storefront metadata into `into`, preferring filled fields / richer text.

#### Arguments

* `into` - `into` input for this call.
* `from` - `from` input for this call.

### merge_recommendation

```rust
pub fn merge_recommendation(into: &mut Recommendation, from: Recommendation)
```

Merge a scored recommendation into an existing card.

#### Arguments

* `into` - `into` input for this call.
* `from` - `from` input for this call.

### parse_series_index

```rust
pub fn parse_series_index(raw: Option<&str>) -> Option<f64>
```

Parse Audible/Chirp-style series indexes (`"1"`, `"1.5"`, `"Book 2"`, `"02"`).

#### Arguments

* `raw` - String `raw` for this call.

#### Returns

`Some(...)` when found / applicable; otherwise `None`.

### push_edition

```rust
pub fn push_edition(editions: &mut Vec<StoreEdition>, edition: StoreEdition)
```

Appends a store edition when not already present (dedupes on source+product_id).

#### Arguments

* `editions` - `editions` input for this call.
* `edition` - `edition` input for this call.

### push_shelf_category

```rust
pub fn push_shelf_category(categories: &mut Vec<String>, kind: &str)
```

Append a stable shelf-kind tag (`finish_series`, `author`, …) if missing.

#### Arguments

* `categories` - String `categories` for this call.
* `kind` - Plugin kind string.

### recommendation_map_key

```rust
pub fn recommendation_map_key(r: &Recommendation) -> String
```

Stable HashMap key for a recommendation card (same rules as [`work_map_key`](#work_map_key)).

#### Arguments

* `r` - `r` input for this call.

#### Returns

String result for this operation.

### soft_work_key

```rust
pub fn soft_work_key(title: &str, authors: Option<&str>) -> String
```

Soft key from cleaned title + primary author (exact string match only).

Keeps subtitles so series volumes (`Infinity Blade: Awakening` vs
`…: Redemption`) do not share a HashMap bucket when hard ids are absent.
Soft [`identities_match`](#identities_match) still joins edition variants (e.g. `A Novel`).

#### Arguments

* `title` - Display title.
* `authors` - Optional author string (comma-separated).

#### Returns

String result for this operation.

### work_map_key

```rust
pub fn work_map_key(
    asin: Option<&str>,
    isbn: Option<&str>,
    title: &str,
    authors: Option<&str>,
    source: Option<&str>,
    product_id: Option<&str>,
) -> String
```

Best stable map key for a recommendation / candidate.

#### Arguments

* `asin` - Optional Audible / Amazon ASIN.
* `isbn` - Optional ISBN (any punctuation; normalized internally).
* `title` - Display title.
* `authors` - Optional author string (comma-separated).
* `source` - Storefront id or filesystem source path, depending on call site.
* `product_id` - Storefront-native product id.

#### Returns

String result for this operation.

### works_match

```rust
pub fn works_match(
    title_a: &str,
    authors_a: Option<&str>,
    title_b: &str,
    authors_b: Option<&str>,
) -> bool
```

Whether two titles should be treated as the same work.

#### Arguments

* `title_a` - String `title_a` for this call.
* `authors_a` - String `authors_a` for this call.
* `title_b` - String `title_b` for this call.
* `authors_b` - String `authors_b` for this call.

#### Returns

`true` when the predicate holds.

### enrich_books_from_metadata

```rust
pub async fn enrich_books_from_metadata(
    library: &LibraryStore,
    providers: &dyn MetadataProviders,
    opts: &MetadataEnrichOptions,
) -> Result<usize, DiscoverError>
```

Fills missing description, subjects, cover and language from the
deployed metadata providers.

Skips titles a provider already filled. With no provider deployed, every
answer is empty and nothing changes.

#### Arguments

* `library` - Library store read and written.
* `providers` - Deployed metadata providers.
* `opts` - Pacing for this run.

#### Returns

The number of titles filled.

#### Errors

Returns an error when a library read or write fails.

### fetch_subject_candidates

```rust
pub async fn fetch_subject_candidates(
    library: &LibraryStore,
    registry: &SourceRegistry,
    subject: &DiscoverSubject,
    opts: &RecommendOptions,
) -> Result<FetchReport, DiscoverError>
```

Fetches one subject's storefront candidates and stores them store by store.

Every deployed storefront is asked, whether or not the subject has an
account there. Stores run concurrently; within a store each seed's call
has [`STORE_CALL_TIMEOUT`](#store_call_timeout), each store stops starting calls after
[`STORE_FETCH_DEADLINE`](#store_fetch_deadline), and the store's share of
`opts.storefront_max_remote_calls` bounds how many calls it makes. A
store's rows replace its previous rows only when it finishes, so a store
that fails keeps what it returned last time.

#### Arguments

* `library` - Library store (taste, cache, candidate rows).
* `registry` - The deployed storefronts.
* `subject` - Whose candidates to fetch.
* `opts` - Seed limit, call budget, region and series-set filter.

#### Errors

Returns a library error from loading the taste. Store failures are
reported in [`FetchReport::stores_failed`](#fetchreport), not as errors.

### load_subject_taste

```rust
pub async fn load_subject_taste(
    library: &LibraryStore,
    subject: &DiscoverSubject,
    opts: &RecommendOptions,
) -> Result<SubjectTaste, DiscoverError>
```

Loads one subject's taste.

A user or portal identity uses the books held by store accounts linked to
them plus the household books they have listened to, the listening rows
recorded for them (by identity id, or by provider and external user id on
rows recorded before the identity was known), and their open wishlist.
While they have fewer than [`OWN_TASTE_THRESHOLD`](#own_taste_threshold) books and listens of
their own, the rest of the household's books blend in at
[`HOUSEHOLD_BLEND_WEIGHT`](#household_blend_weight), so a first wish or book adds to the shelves
they already had instead of replacing them. With no signal of their own
at all (a new member, or a visitor who has connected no store yet), and
always for the operator, the household library is used as it stands.

#### Arguments

* `library` - Library store.
* `subject` - Whose taste to load.
* `opts` - `include_listening` and `listening_providers` apply as on the page.

#### Errors

Returns the library read error.

### rank_subject_feed

```rust
pub async fn rank_subject_feed(
    library: &LibraryStore,
    registry: &SourceRegistry,
    subject: &DiscoverSubject,
    opts: &RecommendOptions,
) -> Result<RankReport, DiscoverError>
```

Ranks one subject's stored candidates into shelves and stores the feed.

No storefront is called (`include_purchase_hints` is ignored: cards fetch
live prices when they scroll into view). Owned titles are filtered again,
so a title bought since the fetch drops out.

#### Arguments

* `library` - Library store.
* `registry` - Unused for network; passed through to ranking.
* `subject` - Whose feed to rank.
* `opts` - Ranking options; `disabled_shelves` is ignored (applied on read).

#### Errors

Returns the library read or write error.

### stored_subject_feed

```rust
pub async fn stored_subject_feed(
    library: &LibraryStore,
    subject: &DiscoverSubject,
    disabled_shelves: &[String],
) -> Result<Option<StoredSubjectFeed>, DiscoverError>
```

Reads one subject's stored feed, drops their hidden shelves and joins
each card's catalog metadata from the candidate pool.

The stored feed keeps card references and what is personal to the
subject; the description, cover and other metadata of the subject's
listings are read from the pool here, in chunks that stay under the
database plugin's result limit.

#### Returns

`None` when the subject has no stored feed yet, or it does not decode
(written by another release); the caller then shows placeholders and
queues a ranking.

#### Errors

Returns the library read error.

### stored_subject_feed_state

```rust
pub async fn stored_subject_feed_state(
    library: &LibraryStore,
    subject: &DiscoverSubject,
) -> Result<Option<StoredSubjectFeed>, DiscoverError>
```

Reads one subject's stored feed without joining its catalog metadata,
for callers that need only whether it exists and when it was ranked.

The cards come back as stored: references and the subject's own fields,
without the description, cover and other metadata
[`stored_subject_feed`](#stored_subject_feed) joins from the candidate pool. Hidden shelves
are kept. No candidate row is read.

#### Returns

`None` when the subject has no stored feed yet, or it does not decode.

#### Errors

Returns the library read error.

### best_purchase_hint

```rust
pub fn best_purchase_hint(hints: &[PurchaseHint]) -> Option<&PurchaseHint>
```

Pick the lowest-priced hint (ties keep earlier order). Unpriced sort after priced.

#### Arguments

* `hints` - Purchase hints to pick from.

#### Returns

`Some(...)` when found / applicable; otherwise `None`.

### best_purchase_hint_preferring

```rust
pub fn best_purchase_hint_preferring<'a>(
    hints: &'a [PurchaseHint],
    preferred: &HashSet<String>,
) -> Option<&'a PurchaseHint>
```

Pick the offer the caller would actually pay least for.

Linked storefronts (`preferred`) are compared at **member** price; every
other store is compared at **list / non-member** price. That way a Libro-only
member still sees Audible on the shelf when Audible’s non-member price beats
Libro’s member price. When nothing is linked, all stores are compared as
non-members. Hints with no usable price for that role sort last.

#### Arguments

* `hints` - Purchase hints to pick from.
* `preferred` - String `preferred` for this call.

#### Returns

`Some(...)` when found / applicable; otherwise `None`.

### format_money_label

```rust
pub fn format_money_label(cents: i64, currency: &str) -> String
```

Formats integer cents into a storefront-style money label for `currency`.

#### Arguments

* `cents` - Integer cents to format.
* `currency` - ISO 4217 currency code.

#### Returns

String result for this operation.

### parse_money_label_to_cents

```rust
pub fn parse_money_label_to_cents(raw: &str) -> Option<i64>
```

Parse `$12.34` / `12.34` / `FREE` into cents.

#### Arguments

* `raw` - String `raw` for this call.

#### Returns

`Some(...)` when found / applicable; otherwise `None`.

### purchase_hints_for

```rust
pub async fn purchase_hints_for(
    registry: &SourceRegistry,
    title: &str,
    author: Option<&str>,
    asin: Option<&str>,
    isbn: Option<&str>,
    region: &str,
) -> Result<Vec<PurchaseHint>, DiscoverError>
```

Look up purchase links via registered
`bookclerk_source::ContentSource::purchase_hint` (no prices).

Call [`resolve_purchase_hints`](#resolve_purchase_hints) for multi-store + live pricing.

#### Arguments

* `registry` - Configured content-source or integration registry.
* `title` - Display title.
* `author` - String `author` for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `isbn` - Optional ISBN (any punctuation; normalized internally).
* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

On success, the inner `Vec<PurchaseHint>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### resolve_purchase_hints

```rust
pub async fn resolve_purchase_hints(
    registry: &SourceRegistry,
    query: &PurchaseHintsQuery,
) -> Result<PurchaseHintsResponse, DiscoverError>
```

Resolve every catalog match and attach live prices (view-time).

#### Arguments

* `registry` - Configured content-source or integration registry.
* `query` - Query vector or free-text search string.

#### Returns

On success, the inner `PurchaseHintsResponse` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### resolve_purchase_hints_batch

```rust
pub async fn resolve_purchase_hints_batch(
    registry: &SourceRegistry,
    queries: &[PurchaseHintsQuery],
    max_concurrent: usize,
) -> Vec<Result<PurchaseHintsResponse, DiscoverError>>
```

Resolve many purchase-hint queries at once (order preserved).

Queries answered from the cache are done at once. For the rest, every
store is asked **once**, through `bookclerk_source::ContentSource::purchase_hints`,
for all of them together, so a store whose catalog prices many products
per request makes one request for a page of cards. The whole batch
answers within [`BATCH_DEADLINE`]: a store still working then counts as
timed out. A query no store answered with a hint (timed out, paused, or
never tried within a guest's own budget) carries its trusted seeds alone
(no prices) and `incomplete` set: it is not cached, so the page shows a
link rather than nothing, keeps no copy, and asks again later. A store
that asks to slow down
(`bookclerk_source::SourceError::RateLimited`) is left alone for the
time it said (see [`RATE_LIMIT_FIRST_PAUSE`](#rate_limit_first_pause)).

#### Arguments

* `registry` - Configured content-source or integration registry.
* `queries` - Batch of purchase-hint queries.
* `max_concurrent` - Unused since each store is asked once; kept for
  the route's call shape.

#### Returns

One result per query, in request order.

### seed_purchase_hint

```rust
pub fn seed_purchase_hint(
    source: &str,
    product_id: &str,
    title: Option<String>,
    region: &str,
) -> Option<PurchaseHint>
```

Seed a deterministic storefront URL from a known candidate (no remote I/O).

#### Arguments

* `source` - Storefront id or filesystem source path, depending on call site.
* `product_id` - Storefront-native product id.
* `title` - Display title.
* `region` - Marketplace / region code (`us`, `uk`, …).

#### Returns

`Some(...)` when found / applicable; otherwise `None`.

### combine_wishlist_score

```rust
pub fn combine_wishlist_score(taste_score: f64, wish_count: i64) -> f64
```

Combine local recommend taste with a heavy multi-user wishlist boost.

#### Arguments

* `taste_score` - `taste_score` input for this call.
* `wish_count` - Numeric `wish_count` value for this call.

#### Returns

`f64` result.

### listening_engagement

```rust
pub fn listening_engagement(row: &ListeningSignal) -> f64
```

Continuous engagement from a listening progress row (≈0.0–6.0).

**Absolute hours heard** are the primary signal — 50% of a 30‑hour title
(15 h) outweighs 50% of a 3‑hour title (1.5 h). Percent complete is only a
secondary completion bonus, not the main weight.

#### Arguments

* `row` - SeaORM query row to project into an RPC DTO.

#### Returns

`f64` result.

### rank_request_queue_taste

```rust
pub async fn rank_request_queue_taste(
    library: &LibraryStore,
    opts: &RecommendOptions,
) -> Result<Vec<QueueRank>, DiscoverError>
```

Ranks every entry of the shared request queue against **household**
taste, for the `discover_request_queue` job to store.

This ranking is **not** personalized: portal `external_user_id` filters are
ignored so every viewer sees the same shared queue order (the household
library and all listening progress). Wishlist rows keyed as ASIN vs ISBN
for the same work are merged first, as the read merges them, so each
ranking's [`crate::queue_entry_key`](#queue_entry_key) matches the entry the read builds.

Only taste is ranked here: the wish-count boost and the wish reasons are
added when the queue is read, from the live counts, so a new wish moves an
entry before the next pass. Entries the library already holds are kept
with `owned` set, so the read can hide them.

#### Arguments

* `library` - Open library store.
* `opts` - Ranking options (embedder, listening filters).

#### Returns

One ranking per merged queue entry (empty when the queue is).

#### Errors

Returns the library read error, or the embedder's open error.

### recommend

```rust
pub async fn recommend(
    library: &LibraryStore,
    registry: &SourceRegistry,
    opts: &RecommendOptions,
) -> Result<Vec<Recommendation>, DiscoverError>
```

Build ranked recommendations for the operator (or a specific external user).

#### Arguments

* `library` - Open library store used for reads/writes.
* `registry` - Configured content-source or integration registry.
* `opts` - Options struct for this operation.

#### Returns

On success, the inner `Vec<Recommendation>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### recommend_feed

```rust
pub async fn recommend_feed(
    library: &LibraryStore,
    registry: &SourceRegistry,
    opts: &RecommendOptions,
) -> Result<DiscoverFeed, DiscoverError>
```

Personalized Discover feed (Netflix-style shelves) from the same candidate pool.

#### Arguments

* `library` - Open library store used for reads/writes.
* `registry` - Configured content-source or integration registry.
* `opts` - Options struct for this operation.

#### Returns

On success, the inner `crate::shelves::DiscoverFeed` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### queue_entry_key

```rust
pub fn queue_entry_key(entry: &GlobalQueueEntry) -> String
```

The key a queue entry's stored ranking is filed under: the library work
its wishes are for. Every wish names a work, so one work is one entry
whatever identifiers each wisher's store gave.

### read_request_queue

```rust
pub async fn read_request_queue(
    library: &LibraryStore,
) -> Result<StoredRequestQueue, DiscoverError>
```

Builds the ranked queue from the open requests and the stored ranking.

An entry with no stored ranking yet (wished since the last pass) is shown
with no taste score, so wish count alone places it until the next pass;
an entry the last pass found in the library is hidden.

#### Arguments

* `library` - Open library store.

#### Errors

Returns the library read error.

### build_discover_feed

```rust
pub fn build_discover_feed(
    recs: &[Recommendation],
    taste: &ShelfTaste,
    per_shelf: usize,
    disabled_shelves: &[String],
) -> DiscoverFeed
```

Partitions scored recommendations into Discover shelves.

Every title lands on one shelf. Shelves claim titles in the order they
are built: the wishlist first, then the series shelves, the
subject's wishes, authors, narrators, genres, starter genres, similar
taste, Chirp deals, then one shelf per store for whatever is left, each
title under the store where it costs the subject least (member price on
stores they have an account with, list price elsewhere, its own store
when no price is known). A wished title shows on the wishlist shelf alone.
Shelves other than the series shelves and the wishlist keep at most
[`SERIES_CAP`](#series_cap) titles of one series; those rotate between series instead.

#### Arguments

* `recs` - Scored recommendations to partition into shelves.
* `taste` - Local taste context used to title personalized shelves.
* `per_shelf` - Max items per shelf (clamped).
* `disabled_shelves` - Shelf kind ids the operator has ignored.

#### Returns

`DiscoverFeed` result.

### flatten_feed

```rust
pub fn flatten_feed(feed: &DiscoverFeed, limit: usize) -> Vec<Recommendation>
```

Flatten shelves into a unique score-sorted list (CLI / legacy).

#### Arguments

* `feed` - Discover feed to flatten.
* `limit` - Maximum number of results to return.

#### Returns

Collected results (may be empty).

### shelf_is_disabled

```rust
pub fn shelf_is_disabled(shelf_id: &str, disabled: &[String]) -> bool
```

Whether `shelf_id` matches an ignore entry (exact, kind prefix, or `from_store`).

#### Arguments

* `shelf_id` - Shelf id to test against the ignore list.
* `disabled` - Ignore-list entries from user preferences.

#### Returns

`true` when the predicate holds.

### shelf_kind_catalog

```rust
pub fn shelf_kind_catalog() -> Vec<ShelfKindInfo>
```

Shelf kinds Discover can emit (for config / UI ignore lists).

### resolve_title_meta

```rust
pub async fn resolve_title_meta(
    query: &TitleMetaQuery,
    providers: Arc<dyn MetadataProviders>,
    sources: Option<&SourceRegistry>,
) -> Result<Option<TitleMeta>, DiscoverError>
```

Resolve public metadata for a title detail dialog.

When `sources` is provided, sparse Audible/Audnexus results are gap-filled
from storefront `bookclerk_source::ContentSource::catalog_detail` (Libro ISBN details).

#### Arguments

* `query` - Query vector or free-text search string.
* `providers` - Deployed metadata providers asked for records.
* `sources` - String `sources` for this call.

#### Returns

On success, the inner `Option<TitleMeta>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### resolve_title_meta_batch

```rust
pub async fn resolve_title_meta_batch(
    queries: &[TitleMetaQuery],
    max_concurrent: usize,
    providers: Arc<dyn MetadataProviders>,
    sources: Option<&SourceRegistry>,
) -> Vec<Result<Option<TitleMeta>, DiscoverError>>
```

Resolve many title-meta queries with bounded concurrency (order preserved).

#### Arguments

* `queries` - Batch of purchase-hint or title-meta queries.
* `max_concurrent` - Numeric `max_concurrent` value for this call.
* `providers` - Deployed metadata providers asked for records.
* `sources` - String `sources` for this call.

#### Returns

On success, `Vec<Result<Option<TitleMeta>>>`.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### resolve_title_reviews

```rust
pub async fn resolve_title_reviews(
    query: &TitleReviewsQuery,
) -> Result<TitleReviewsPage, DiscoverError>
```

Resolve one page of Audible customer reviews for infinite scroll.

#### Arguments

* `query` - Query vector or free-text search string.

#### Returns

On success, the inner `TitleReviewsPage` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### pool_wish_editions

```rust
pub async fn pool_wish_editions(
    library: &LibraryStore,
    registry: &SourceRegistry,
    editions: &[(String, String)],
    hints: &[PurchaseHint],
    region: &str,
) -> Vec<PooledEdition>
```

Asks each store about the editions a wish names and records the answers
in the shared pool.

Editions are looked up concurrently, each within
`EDITION_LOOKUP_TIMEOUT`. An edition whose store is not deployed, does
not answer in time, fails or does not know the product is left out: the
wish is still saved, with that edition's row naming no listing. Server
side purchase hints for the same editions add the member price they
quote to the pooled listing, since they also come from the store.

#### Arguments

* `library` - The library whose pool records the listings.
* `registry` - The deployed storefronts.
* `editions` - `(source, product_id)` pairs, in the wish's order; blank
  and repeated pairs are skipped, and at most [`MAX_WISH_EDITIONS`](#max_wish_editions) are
  asked about.
* `hints` - Purchase hints the server resolved for the wish.
* `region` - The marketplace the stores are asked about.

#### Returns

One entry per edition the pool now holds, in the order given.

## Constants

### EMBED_PAGE_WORKS

```rust
pub const EMBED_PAGE_WORKS: u64 = 128;
```

Works read per page by [`embed_dirty_works`](#embed_dirty_works); also the most texts one
embedder call receives.

### MODEL_ALL_MINILM_L6_V2_Q

```rust
pub const MODEL_ALL_MINILM_L6_V2_Q: &str = "all-minilm-l6-v2-q";
```

Quantized MiniLM-L6-v2 — ~22 MB ONNX, ~50 MB RAM, 384 dims.

### MODEL_LOCAL_HASH_V1

```rust
pub const MODEL_LOCAL_HASH_V1: &str = "local-hash-v1";
```

Local hash embedder id used when ONNX is unavailable / disabled.

### DEFAULT_MAX_REQUESTS_PER_RUN

```rust
pub const DEFAULT_MAX_REQUESTS_PER_RUN: usize = 25;
```

Default cap on provider asks in one run.

### METADATA_ENRICH_SOURCE_PREFIX

```rust
pub const METADATA_ENRICH_SOURCE_PREFIX: &str = "metadata:";
```

`enrich_source` prefix marking a title filled by a metadata provider; the
provider's plugin key follows it.

### HOUSEHOLD_BLEND_WEIGHT

```rust
pub const HOUSEHOLD_BLEND_WEIGHT: f64 = 0.5;
```

How much a household book counts in a member's taste while it blends in,
relative to one of their own.

### OWN_TASTE_THRESHOLD

```rust
pub const OWN_TASTE_THRESHOLD: usize = 5;
```

Books and listens of their own a subject needs before the household's
books stop blending into their taste.

### STARTER_GENRES

```rust
pub const STARTER_GENRES: [&str; 6] = _;
```

Genres browsed for a thin taste, in the order the stores are asked (the
call budget may stop before the end).

### STARTER_GENRE_MIN_SEEDS

```rust
pub const STARTER_GENRE_MIN_SEEDS: usize = 5;
```

Seeds below which a fetch also browses each store's [`STARTER_GENRES`](#starter_genres),
so a new library still gets shelves.

### STORE_CALL_MAX_AGE

```rust
pub const STORE_CALL_MAX_AGE: Duration = _;
```

How long a cached storefront answer is reused (the daily refresh).

### STORE_CALL_TIMEOUT

```rust
pub const STORE_CALL_TIMEOUT: Duration = _;
```

Longest one storefront call may take before its store moves on.

### STORE_FETCH_DEADLINE

```rust
pub const STORE_FETCH_DEADLINE: Duration = _;
```

How long one store may keep starting new calls in a fetch.

A store that is slow on every call stops asking once this has passed
and saves what it has, so it delays the subject's ranking by at most
this plus one [`STORE_CALL_TIMEOUT`](#store_call_timeout).

### BATCH_DEADLINE

```rust
pub const BATCH_DEADLINE: Duration = _;
```

Longest a batch waits for its lookups before answering with what has
resolved. Under the daemon's 25 s route budget, so a slow store never
turns a page of cards into one failed request.

### BREAKER_OPEN_FOR

```rust
pub const BREAKER_OPEN_FOR: Duration = _;
```

How long a tripped store is skipped. After that, lookups ask it again,
and [`BREAKER_TRIP_TIMEOUTS`](#breaker_trip_timeouts) more timeouts in a row pause it again.

### BREAKER_TRIP_TIMEOUTS

```rust
pub const BREAKER_TRIP_TIMEOUTS: u32 = 3;
```

Timeouts in a row after which a store's price lookups are skipped.

### RATE_LIMIT_FIRST_PAUSE

```rust
pub const RATE_LIMIT_FIRST_PAUSE: Duration = _;
```

How long a store that asked to slow down without saying for how long
is left alone the first time; each repeat doubles it.

### RATE_LIMIT_MAX_PAUSE

```rust
pub const RATE_LIMIT_MAX_PAUSE: Duration = _;
```

Longest pause a rate limit can grow to by doubling, and the most a
store's own `Retry-After` is honored for.

### WISH_COUNT_WEIGHT

```rust
pub const WISH_COUNT_WEIGHT: f64 = 40.0;
```

Per open wish on the global queue. Large enough that multi-user demand
dominates typical local taste scores (~1–25) while recommend signals still
order titles that share the same wish count.

### REQUEST_QUEUE_MAX_AGE

```rust
pub const REQUEST_QUEUE_MAX_AGE: Duration = _;
```

How old the stored ranking may get before a read asks for a new pass,
so taste changes no event announces (a book removed, a rating edited)
still reach the queue within a day.

### SERIES_CAP

```rust
pub const SERIES_CAP: usize = 2;
```

Most titles of one series on a shelf that is not about series, so one
long series cannot crowd the others out.

### MAX_WISH_EDITIONS

```rust
pub const MAX_WISH_EDITIONS: usize = 8;
```

Most store editions looked up for one wish. A wish names a few; the cap
bounds what a crafted request can make the server fetch.
