# bookclerk_plugin_source_libro::catalog

Libro.fm catalog search, detail, related titles and purchase links.

Every call goes to the Android app's signed-out catalog API
(`/api/vN/explore/…`, the same prefix as [`LIBRARY_PATH`](README.md#library_path)). Requests carry
an honest Bookclerk user agent and the app's `X-LibroFm-AppVer` header,
which is part of the app's API contract (the signed-in client sends the
same value) and without which `explore/audiobook_details` answers `404`.
No `Authorization`, device or OS headers are sent. Libro.fm's load
balancer refuses automated requests to its website pages, so this module
never fetches storefront HTML and never presents itself as a browser.

Search rows are lean (ISBN, title, authors, narrators, cover);
`explore/audiobook_details` adds description, genres, runtime, publisher,
series, abridged, language, list price and related titles.

This crate intentionally does **not** depend on `bookclerk-enrich`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`LibroDetails`](#librodetails) | struct | One title from `explore/audiobook_details`. |
| [`explore_search_path`](#explore_search_path) | function | Path of the app's catalog search (`GET …/explore/search?q=&page=`). |
| [`search_catalog`](#search_catalog) | function | Catalog search → `CatalogHit`s keyed by ISBN. |
| [`catalog_detail`](#catalog_detail) | function | Catalog record for one ISBN (or ISBN-leading slug). |
| [`expand_candidates`](#expand_candidates) | function | Related candidates for Discover. |
| [`purchase_hint`](#purchase_hint) | function | Resolve a Libro.fm product link for a title, by ISBN first and then by title and author. |
| [`parse_details`](#parse_details) | function | Parses one `explore/audiobook_details` body. |
| [`parse_search_page`](#parse_search_page) | function | Parses one catalog search response body into at most `limit` hits. |
| [`CATALOG_USER_AGENT`](#catalog_user_agent) | constant | User agent for signed-out catalog calls: names Bookclerk and its site so Libro.fm can identify (and contact) the client. |

## Structs

### LibroDetails

```rust
pub struct LibroDetails {
    pub audiobook: Option<CatalogHit>,
    pub related: Vec<CatalogHit>,
    pub for_sale: bool,
}
```

One title from `explore/audiobook_details`.

**Fields**

- `audiobook`: The title itself, with its list price; `None` when the payload had no
  usable audiobook object.
- `related`: Related titles the app shows under it (lean rows, like search).
- `for_sale`: Whether Libro.fm sells the title (`purchase_info.for_sale`); `true`
  when the payload does not say.

Implements: `Debug`, `Default`

## Functions

### explore_search_path

```rust
pub fn explore_search_path() -> String
```

Path of the app's catalog search (`GET …/explore/search?q=&page=`).

### search_catalog

```rust
pub async fn search_catalog(
    base_url: &str,
    opts: &CatalogSearchOpts,
) -> Result<Vec<CatalogHit>>
```

Catalog search → [`CatalogHit`](../bookclerk_source/README.md#cataloghit)s keyed by ISBN.

#### Arguments

* `base_url` - Libro.fm API origin (the plugin's `base_url` setting).
* `opts` - Query, page and hit limit.

#### Errors

Returns an error when Libro.fm answers with a non-success status or a body
that is not the search envelope.

### catalog_detail

```rust
pub async fn catalog_detail(
    base_url: &str,
    product_id: &str,
) -> Result<Option<CatalogHit>>
```

Catalog record for one ISBN (or ISBN-leading slug).

Prefers the app's `explore/audiobook_details` (description, genres,
runtime, publisher, series, abridged, language, list price). When that
misses or fails, falls back to the catalog search row whose ISBN matches
exactly, so a fuzzy hit never stands in for the requested book.

#### Arguments

* `base_url` - Libro.fm API origin (the plugin's `base_url` setting).
* `product_id` - ISBN-10/13 or `{isbn}-{slug}`; anything else returns `None`.

#### Errors

Returns an error when the details call misses and the fallback search fails.

### expand_candidates

```rust
pub async fn expand_candidates(
    base_url: &str,
    seed: &ExpandSeed,
    limit: usize,
) -> Result<Vec<CatalogHit>>
```

Related candidates for Discover.

A Libro.fm seed with an ISBN gets the app's related titles from
`explore/audiobook_details`. Otherwise, or when that returns none, a
Libro.fm seed gets more titles by its first author and a seed from
another store gets a title-and-author search. The seed itself is never
returned.

#### Arguments

* `base_url` - Libro.fm API origin.
* `seed` - Title the candidates relate to.
* `limit` - Most candidates to return.

#### Errors

Returns an error when the fallback catalog search fails.

### purchase_hint

```rust
pub async fn purchase_hint(
    base_url: &str,
    opts: &PurchaseHintOpts,
) -> Result<Option<SourcePurchaseHint>>
```

Resolve a Libro.fm product link for a title, by ISBN first and then by
title and author.

With `opts.with_price`, the list price comes from
`explore/audiobook_details` (`purchase_info.price`, USD). An edition that
details report as not for sale is never offered, priced or not: the title
match then moves on to the next search row.

#### Arguments

* `base_url` - Libro.fm API origin.
* `opts` - ISBN, product id, title and authors to match.

#### Errors

Currently never: a failed ISBN lookup or title search is logged and
treated as no match. The `Result` keeps the `ContentSource` signature.

### parse_details

```rust
pub fn parse_details(body: &[u8]) -> Result<LibroDetails>
```

Parses one `explore/audiobook_details` body.

Sans-IO so the unit tests drive the same code as the network path. The
list price (`purchase_info.price`) is copied onto the audiobook hit.

#### Errors

Returns an error when `body` is not JSON shaped like the details envelope.

### parse_search_page

```rust
pub fn parse_search_page(body: &[u8], limit: usize) -> Result<Vec<CatalogHit>>
```

Parses one catalog search response body into at most `limit` hits.

Sans-IO so the replay test and fuzz target drive the same code as the
network path.

#### Errors

Returns an error when `body` is not JSON shaped like the search envelope.

## Constants

### CATALOG_USER_AGENT

```rust
pub const CATALOG_USER_AGENT: &str = "bookclerk/0.1.0 (+https://bookclerk.app)";
```

User agent for signed-out catalog calls: names Bookclerk and its site so
Libro.fm can identify (and contact) the client.
