# bookclerk_plugin_source_audible::catalog

Public Audible catalog helpers for Discover (`search_catalog` / expand / purchase).

Uses [`bookclerk_audible_catalog`](../bookclerk_audible_catalog/README.md) public catalog HTTP — no account required.

| Item | Kind | Summary |
| --- | --- | --- |
| [`search_catalog`](#search_catalog) | function | Search the public Audible catalog for Discover typeahead / paged browse. |
| [`expand_candidates`](#expand_candidates) | function | Expand author / series / narrator / series-ASIN candidates from a taste seed. |
| [`purchase_hint`](#purchase_hint) | function | Resolve a purchase / catalog URL (optionally with live price). |
| [`purchase_answer`](#purchase_answer) | function | Resolve a title's Audible product and price, with every listing a title search found. |
| [`purchase_hints`](#purchase_hints) | function | Resolves purchase hints for several titles, pricing them together. |
| [`purchase_answers`](#purchase_answers) | function | Resolves several titles, pricing them together, with each title search's records. |
| [`catalog_detail`](#catalog_detail) | function | The public catalog record for one ASIN (`catalogDetail` by product id): its bibliographic fields, cover and audio sample. |
| [`TITLE_RECORDS`](#title_records) | constant | Most listings one title search answers as records. A page of cards stays within a couple of `PRICE_BATCH_ASINS` price requests. |
| [`PRICE_BATCH_ASINS`](#price_batch_asins) | constant | Most ASINs the public catalog prices in one `products` request. |
| [`RESOLVE_BUDGET`](#resolve_budget) | constant | How long `purchase_hints` keeps starting catalog searches for titles without an ASIN. Well under the host's 10 s batch deadline, so the price request for every resolved title still lands in time. |

## Functions

### search_catalog

```rust
pub async fn search_catalog(opts: &CatalogSearchOpts) -> Result<Vec<CatalogHit>>
```

Search the public Audible catalog for Discover typeahead / paged browse.

When `CatalogSearchOpts::field` is set, uses storefront-native filters
(`author=` / `narrator=` / series-name filter / Genres `category_id`) so facet
links return the right catalog slice. General queries use `keywords=` only
and preserve Audible relevance order (host merge soft-prefers language).

#### Errors

Returns an error when the operation fails.

### expand_candidates

```rust
pub async fn expand_candidates(seed: &ExpandSeed, limit: usize) -> Result<Vec<CatalogHit>>
```

Expand author / series / narrator / series-ASIN candidates from a taste seed.

Soft-fails individual HTTP calls; budgets returned hits by `limit`.

#### Errors

Returns an error when the operation fails.

### purchase_hint

```rust
pub async fn purchase_hint(opts: &PurchaseHintOpts) -> Result<Option<SourcePurchaseHint>>
```

Resolve a purchase / catalog URL (optionally with live price).

The bare-hint form of [`purchase_answer`]: its pick, without the
title-search records.

#### Errors

As [`purchase_answer`](#purchase_answer).

### purchase_answer

```rust
pub async fn purchase_answer(opts: &PurchaseHintOpts) -> Result<PricedAnswer>
```

Resolve a title's Audible product and price, with every listing a title
search found.

- A known ASIN (the query's `asin`, else an ASIN-shaped `product_id`) is
  priced as it is, with no search. This is the exact ASIN lookup the
  store declares (`bookclerk_storefront::PriceLookup::Asin`).
- Otherwise the title and author are searched (the ISBN as keywords
  when that misses or there is no title), and up to [`TITLE_RECORDS`](#title_records)
  listings are returned as priced records, each with its own ASIN,
  credits and series, for the host to place. The first is also the
  store's pick, so a host that reads only the hint still gets one.

#### Errors

Returns `SourceError::RateLimited` when Audible asks to slow down
while pricing; a failed search answers nothing.

### purchase_hints

```rust
pub async fn purchase_hints(
    opts: &[PurchaseHintOpts],
) -> Result<Vec<Option<SourcePurchaseHint>>>
```

Resolves purchase hints for several titles, pricing them together.

The bare-hint form of [`purchase_answers`](#purchase_answers).

#### Errors

As [`purchase_answers`](#purchase_answers).

### purchase_answers

```rust
pub async fn purchase_answers(opts: &[PurchaseHintOpts]) -> Result<Vec<PricedAnswer>>
```

Resolves several titles, pricing them together, with each title
search's records.

Titles whose ASIN is known resolve first, without a search. The rest
are searched one by one (as [`purchase_answer`](#purchase_answer) does) until
[`RESOLVE_BUDGET`](#resolve_budget) runs out; a title not searched by then answers
nothing, which the host does not take as an answer. Then every ASIN
found (picks and records) is priced per marketplace, each query in its
own region, one catalog request per [`PRICE_BATCH_ASINS`](#price_batch_asins), so a page of
cards costs one or two price requests per region instead of one per
card.

#### Errors

Returns `SourceError::RateLimited` when Audible asks to slow down;
a search that fails otherwise answers nothing for that title.

### catalog_detail

```rust
pub async fn catalog_detail(product_id: &str) -> Result<Option<CatalogHit>>
```

The public catalog record for one ASIN (`catalogDetail` by product id):
its bibliographic fields, cover and audio sample.

#### Arguments

* `product_id` - The ASIN.

#### Returns

`None` when `product_id` is not an ASIN, or every marketplace answered
within eight seconds and none lists it.

#### Errors

Returns an error when no HTTP client can be built, or when no
marketplace listed the product and one of them failed or the eight
seconds ran out first, so the product's absence is not settled.

## Constants

### TITLE_RECORDS

```rust
pub const TITLE_RECORDS: usize = 10;
```

Most listings one title search answers as records. A page of cards
stays within a couple of [`PRICE_BATCH_ASINS`](#price_batch_asins) price requests.

### PRICE_BATCH_ASINS

```rust
pub const PRICE_BATCH_ASINS: usize = 50;
```

Most ASINs the public catalog prices in one `products` request.

### RESOLVE_BUDGET

```rust
pub const RESOLVE_BUDGET: Duration = _;
```

How long [`purchase_hints`](#purchase_hints) keeps starting catalog searches for titles
without an ASIN. Well under the host's 10 s batch deadline, so the price
request for every resolved title still lands in time.
