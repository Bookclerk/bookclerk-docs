# bookclerk_plugin_source_libro::catalog

Public Libro.fm explore catalog helpers for Discover.

Mirrors former `bookclerk-discover` explore HTTP (no account required).
This crate intentionally does **not** depend on `bookclerk-enrich`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`search_catalog`](#search_catalog) | function | Explore search → `CatalogHit`s keyed by ISBN. |
| [`catalog_detail`](#catalog_detail) | function | Public catalog detail for one ISBN/slug (title dialogs / page enrich). |
| [`expand_candidates`](#expand_candidates) | function | Expand via `explore/audiobook_details/{isbn}` → `related_audiobooks`. |
| [`purchase_hint`](#purchase_hint) | function | Resolve a Libro.fm purchase URL (ISBN or catalog search), optionally priced. |

## Functions

### search_catalog

```rust
pub async fn search_catalog(opts: &CatalogSearchOpts) -> Result<Vec<CatalogHit>>
```

Explore search → [`CatalogHit`](../bookclerk_source/index.md#cataloghit)s keyed by ISBN.

Prefers the explore JSON endpoint; when that fails (Libro often returns
500/403), falls back to scraping the public HTML search page.

Maps every bibliographic field the search payload already includes.

Lean HTML search hits stay sparse on purpose — Discover enriches the
**final page** via [`catalog_detail`](#catalog_detail) so we do not N+1 product fetches
inside each store search (that blew the daemon’s 12s search budget).

#### Errors

Returns an error when the operation fails.

### catalog_detail

```rust
pub async fn catalog_detail(product_id: &str) -> Result<Option<CatalogHit>>
```

Public catalog detail for one ISBN/slug (title dialogs / page enrich).

Prefers product HTML (JSON-LD + `.audiobook-genres`) — reliable under WAF —
and falls back to `explore/audiobook_details` when HTML misses. Explore is
tried second because it often 403s and wasted a round-trip before HTML.

#### Errors

Returns an error when the operation fails.

### expand_candidates

```rust
pub async fn expand_candidates(seed: &ExpandSeed, limit: usize) -> Result<Vec<CatalogHit>>
```

Expand via `explore/audiobook_details/{isbn}` → `related_audiobooks`.

#### Errors

Returns an error when the operation fails.

### purchase_hint

```rust
pub async fn purchase_hint(opts: &PurchaseHintOpts) -> Result<Option<SourcePurchaseHint>>
```

Resolve a Libro.fm purchase URL (ISBN or catalog search), optionally priced.

#### Errors

Returns an error when the operation fails.
