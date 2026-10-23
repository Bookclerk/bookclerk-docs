**bookclerk_plugin_source_libro > catalog**

# Module: catalog

## Contents

**Functions**

- [`catalog_detail`](#catalog_detail) - Public catalog detail for one ISBN/slug (title dialogs / page enrich).
- [`expand_candidates`](#expand_candidates) - Expand via `explore/audiobook_details/{isbn}` → `related_audiobooks`.
- [`purchase_hint`](#purchase_hint) - Resolve a Libro.fm purchase URL (ISBN or catalog search), optionally priced.
- [`search_catalog`](#search_catalog) - Explore search → [`CatalogHit`](../bookclerk_source/types.md#bookclerk_sourcetypescataloghit)s keyed by ISBN.

---

## bookclerk_plugin_source_libro::catalog::catalog_detail

*Function*

Public catalog detail for one ISBN/slug (title dialogs / page enrich).

Prefers product HTML (JSON-LD + `.audiobook-genres`) — reliable under WAF —
and falls back to `explore/audiobook_details` when HTML misses. Explore is
tried second because it often 403s and wasted a round-trip before HTML.

# Errors

Returns an error when the operation fails.

```rust
fn catalog_detail(product_id: &str) -> bookclerk_source::Result<Option<bookclerk_source::CatalogHit>>
```



## bookclerk_plugin_source_libro::catalog::expand_candidates

*Function*

Expand via `explore/audiobook_details/{isbn}` → `related_audiobooks`.

# Errors

Returns an error when the operation fails.

```rust
fn expand_candidates(seed: &bookclerk_source::ExpandSeed, limit: usize) -> bookclerk_source::Result<Vec<bookclerk_source::CatalogHit>>
```



## bookclerk_plugin_source_libro::catalog::purchase_hint

*Function*

Resolve a Libro.fm purchase URL (ISBN or catalog search), optionally priced.

# Errors

Returns an error when the operation fails.

```rust
fn purchase_hint(opts: &bookclerk_source::PurchaseHintOpts) -> bookclerk_source::Result<Option<bookclerk_source::SourcePurchaseHint>>
```



## bookclerk_plugin_source_libro::catalog::search_catalog

*Function*

Explore search → [`CatalogHit`](../bookclerk_source/types.md#bookclerk_sourcetypescataloghit)s keyed by ISBN.

Prefers the explore JSON endpoint; when that fails (Libro often returns
500/403), falls back to scraping the public HTML search page.

Maps every bibliographic field the search payload already includes.

Lean HTML search hits stay sparse on purpose — Discover enriches the
**final page** via [`catalog_detail`](catalog.md#bookclerk_plugin_source_librocatalogcatalog_detail) so we do not N+1 product fetches
inside each store search (that blew the daemon’s 12s search budget).

# Errors

Returns an error when the operation fails.

```rust
fn search_catalog(opts: &bookclerk_source::CatalogSearchOpts) -> bookclerk_source::Result<Vec<bookclerk_source::CatalogHit>>
```



