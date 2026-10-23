**bookclerk_plugin_source_audible > catalog**

# Module: catalog

## Contents

**Functions**

- [`expand_candidates`](#expand_candidates) - Expand author / series / narrator / series-ASIN candidates from a taste seed.
- [`purchase_hint`](#purchase_hint) - Resolve a purchase / catalog URL (optionally with live price).
- [`search_catalog`](#search_catalog) - Search the public Audible catalog for Discover typeahead / paged browse.

---

## bookclerk_plugin_source_audible::catalog::expand_candidates

*Function*

Expand author / series / narrator / series-ASIN candidates from a taste seed.

Soft-fails individual HTTP calls; budgets returned hits by `limit`.

# Errors

Returns an error when the operation fails.

```rust
fn expand_candidates(seed: &bookclerk_source::ExpandSeed, limit: usize) -> bookclerk_source::Result<Vec<bookclerk_source::CatalogHit>>
```



## bookclerk_plugin_source_audible::catalog::purchase_hint

*Function*

Resolve a purchase / catalog URL (optionally with live price).

# Errors

Returns an error when the operation fails.

```rust
fn purchase_hint(opts: &bookclerk_source::PurchaseHintOpts) -> bookclerk_source::Result<Option<bookclerk_source::SourcePurchaseHint>>
```



## bookclerk_plugin_source_audible::catalog::search_catalog

*Function*

Search the public Audible catalog for Discover typeahead / paged browse.

When `CatalogSearchOpts::field` is set, uses storefront-native filters
(`author=` / `narrator=` / series-name filter / Genres `category_id`) so facet
links return the right catalog slice. General queries use `keywords=` only
and preserve Audible relevance order (host merge soft-prefers language).

# Errors

Returns an error when the operation fails.

```rust
fn search_catalog(opts: &bookclerk_source::CatalogSearchOpts) -> bookclerk_source::Result<Vec<bookclerk_source::CatalogHit>>
```



