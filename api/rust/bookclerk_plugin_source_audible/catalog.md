# bookclerk_plugin_source_audible::catalog

Public Audible catalog helpers for Discover (`search_catalog` / expand / purchase).

Uses [`bookclerk_enrich`](../bookclerk_enrich/index.md) public catalog HTTP — no account required.

| Item | Kind | Summary |
| --- | --- | --- |
| [`search_catalog`](#search_catalog) | function | Search the public Audible catalog for Discover typeahead / paged browse. |
| [`expand_candidates`](#expand_candidates) | function | Expand author / series / narrator / series-ASIN candidates from a taste seed. |
| [`purchase_hint`](#purchase_hint) | function | Resolve a purchase / catalog URL (optionally with live price). |

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
