# bookclerk_plugin_source_audible::catalog

Public Audible catalog helpers for Discover (`search_catalog` / expand / purchase).

Uses [`bookclerk_audible_catalog`](../bookclerk_audible_catalog/README.md) public catalog HTTP — no account required.

| Item | Kind | Summary |
| --- | --- | --- |
| [`search_catalog`](#search_catalog) | function | Search the public Audible catalog for Discover typeahead / paged browse. |
| [`expand_candidates`](#expand_candidates) | function | Expand author / series / narrator / series-ASIN candidates from a taste seed. |
| [`purchase_hint`](#purchase_hint) | function | Resolve a purchase / catalog URL (optionally with live price). |
| [`purchase_hints`](#purchase_hints) | function | Resolves purchase hints for several titles, pricing them together. |
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

#### Errors

Returns an error when the operation fails.

### purchase_hints

```rust
pub async fn purchase_hints(
    opts: &[PurchaseHintOpts],
) -> Result<Vec<Option<SourcePurchaseHint>>>
```

Resolves purchase hints for several titles, pricing them together.

Titles whose ASIN is known resolve first, without a search. The rest
are searched one by one (as [`purchase_hint`](#purchase_hint) does) until
[`RESOLVE_BUDGET`](#resolve_budget) runs out; a title not searched by then answers `None`,
which the host does not take as an answer. Then the resolved ASINs are
priced per marketplace, each query in its own region, one catalog
request per [`PRICE_BATCH_ASINS`](#price_batch_asins), so a page of cards costs one price
request per region instead of one per card.

#### Errors

Returns `SourceError::RateLimited` when Audible asks to slow down;
a resolution that fails otherwise answers `None` for that title.

## Constants

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
