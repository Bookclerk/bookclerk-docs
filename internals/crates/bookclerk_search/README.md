# bookclerk_search

Tantivy search index with classic Libation field names and query normalization.

## Audience

Host library / GUI code that builds and queries the on-disk search index.
Guest plugins do not depend on this crate.

Style: `docs/contributing/code-documentation.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SearchEngine`](#searchengine) | struct | On-disk Tantivy search engine (classic Lucene `SearchEngine` parity). |
| [`SearchHit`](#searchhit) | struct | One search hit. |
| [`SearchPage`](#searchpage) | struct | One page of search hits in title order, and how many documents match. |
| [`SavedFilter`](#savedfilter) | struct | A saved filter expression (Bookclerk quick-filter parity). |
| [`SearchError`](#searcherror) | enum | Errors from opening, writing, or querying the Tantivy index. |
| [`normalize_lucene_query`](#normalize_lucene_query) | function | Normalize a classic Libation/Lucene query for Tantivy's parser. |
| [`Result`](#result) | type alias | Result alias for search index and query helpers in this crate. |

## Structs

### SearchEngine

```rust
pub struct SearchEngine {
    /* private fields */
}
```

On-disk Tantivy search engine (classic Lucene `SearchEngine` parity).

#### SearchEngine::open

```rust
pub fn open(dir: &Path) -> Result<Self>
```

Open or create a search index at `dir`.

##### Arguments

* `dir` - Absolute path to the Tantivy index directory (created if missing).

##### Errors

Returns [`crate::SearchError::Index`](#searcherror) when the directory or index cannot
be opened.

#### SearchEngine::is_complete

```rust
pub fn is_complete(dir: &Path) -> bool
```

True when a full rebuild of the current layout has finished in `dir`.

An index that was never rebuilt, was deleted, or was written under
an older layout is not complete; the daemon queues a rebuild for it.

#### SearchEngine::rebuild

```rust
pub async fn rebuild(&self, library: &LibraryStore) -> Result<usize>
```

Rebuild the entire index from the library DB.

Rows are read in pages that start at 64. A page of enriched `books`
rows can exceed the sqlite guest `maxResultBytes` (256 KiB). The reader
halves until a slice fits, then keeps that width, so the index still
receives every catalog row.

##### Errors

Returns an error when the operation fails.

#### SearchEngine::rebuild_waiting

```rust
pub async fn rebuild_waiting(
    &self,
    library: &LibraryStore,
    max_wait: Duration,
) -> Result<usize>
```

Rebuilds the index like [`Self::rebuild`](#searchenginerebuild), waiting while another
writer holds it.

A running daemon refreshes its index from library events, and each
refresh holds the writer for a moment (a daemon rebuild holds it for
longer). A rebuild that finds the index busy has written nothing, so
it pauses (100 ms, doubling to 2 s) and tries again until `max_wait`
has passed.

##### Arguments

* `library` - The library the titles are read from.
* `max_wait` - How long to keep trying while the index is busy.

##### Returns

How many titles the index now holds.

##### Errors

Returns [`SearchError::Busy`](#searcherror) when the index is still busy after
`max_wait`, and any other error of [`Self::rebuild`](#searchenginerebuild) at once.

#### SearchEngine::refresh_listings

```rust
pub async fn refresh_listings(
    &self,
    library: &LibraryStore,
    listings: &[ListingRef],
) -> Result<usize>
```

Replaces the documents of `listings` with the library titles that
sell them now.

Every document carrying a listing's term is deleted and each title
`LibraryStore::books_of_listing` still finds is added again, so a
title whose row is gone (a holding removed, an account revoked)
leaves the index, and one that changed is replaced.

##### Arguments

* `library` - The library the titles are read from.
* `listings` - The listings an event touched.

##### Returns

How many documents were written.

##### Errors

Returns [`SearchError::Busy`](#searcherror) when another writer holds the index,
[`SearchError::Library`](#searcherror) when a read fails, or
[`SearchError::Index`](#searcherror) when the write fails.

#### SearchEngine::refresh_titles

```rust
pub async fn refresh_titles(
    &self,
    library: &LibraryStore,
    uuids: &[String],
) -> Result<usize>
```

Replaces the documents of the titles `uuids` with their rows now, and
drops any whose row is gone.

##### Arguments

* `library` - The library the titles are read from.
* `uuids` - Library `books.uuid` values.

##### Returns

How many documents were written.

##### Errors

As [`Self::refresh_listings`](#searchenginerefresh_listings).

#### SearchEngine::mark_incomplete

```rust
pub fn mark_incomplete(&self) -> Result<()>
```

Forgets that the index was fully rebuilt, so the daemon queues a new
rebuild. For a change a refresh cannot place, such as an event whose
listing is already gone.

##### Errors

Returns [`SearchError::Index`](#searcherror) when the marker cannot be removed.

#### SearchEngine::open_and_search

```rust
pub async fn open_and_search(
    dir: PathBuf,
    query: String,
    limit: usize,
) -> Result<Vec<SearchHit>>
```

Open an index and search it without blocking the caller's task.

Both [`SearchEngine::open`](#searchengineopen) and [`SearchEngine::search`](#searchenginesearch) do synchronous
disk work — opening builds the schema and touches the index directory,
and querying reads and scores segments. Calling them directly from an
async handler stalls a runtime worker for the whole query, which on the
daemon's `/api/library/books?q=` path meant one search could hold up
unrelated requests.

##### Errors

Propagates index-open and query failures.

#### SearchEngine::search

```rust
pub fn search(&self, raw_query: &str, limit: usize) -> Result<Vec<SearchHit>>
```

Search the index. `limit` of 0 returns all matches (classic `-n 0`).

Blocking. Prefer [`SearchEngine::open_and_search`](#searchengineopen_and_search) from async code.

##### Errors

Returns an error when the operation fails.

#### SearchEngine::open_and_search_page

```rust
pub async fn open_and_search_page(
    dir: PathBuf,
    query: String,
    offset: usize,
    limit: usize,
) -> Result<SearchPage>
```

Opens an index and reads one title-ordered page of a query's hits
without blocking the caller's task (see [`Self::open_and_search`](#searchengineopen_and_search)).

##### Errors

Propagates index-open and query failures.

#### SearchEngine::open_and_match_uuids

```rust
pub async fn open_and_match_uuids(dir: PathBuf, query: String) -> Result<Vec<String>>
```

Opens an index and lists every hit of a query without blocking the
caller's task (see [`Self::open_and_search`](#searchengineopen_and_search)).

##### Errors

Propagates index-open and query failures.

#### SearchEngine::search_page

```rust
pub fn search_page(
    &self,
    raw_query: &str,
    offset: usize,
    limit: usize,
) -> Result<SearchPage>
```

One page of a query's hits in case-insensitive title order, with the
number of matching documents.

Sorting, paging and counting all happen in the index, so no hit cap
applies and only `limit` uuids leave it. Blocking.

##### Arguments

* `raw_query` - Lucene-style query (normalized before parse).
* `offset` - Hits to skip.
* `limit` - Most hits to return; 0 returns only the count.

##### Errors

Returns [`SearchError::Query`](#searcherror) for a query that does not parse, or
[`SearchError::Index`](#searcherror) when the index cannot be read.

#### SearchEngine::match_uuids

```rust
pub fn match_uuids(&self, raw_query: &str) -> Result<Vec<String>>
```

The uuid of every document a query matches, in title order.

For callers that must filter the hits by something the index does
not hold authoritatively; there is no cap. The count and the hits
come from one searcher, so a commit in between cannot cut the list
short. Blocking.

##### Errors

As [`Self::search_page`](#searchenginesearch_page).

### SearchHit

```rust
pub struct SearchHit {
    pub asin: String,
    pub uuid: String,
    pub account_id: String,
    pub title: String,
    pub score: f32,
}
```

One search hit.

**Fields**

- `asin`: Display / primary id (source `product_id`).
- `uuid`: Library public uuid (indexed lowercased; returned as stored).
- `account_id`: Library account id that owns this title.
- `title`: Display title stored with the hit.
- `score`: Tantivy relevance score (higher is better).

Implements: `Clone`, `Debug`, `PartialEq`

### SearchPage

```rust
pub struct SearchPage {
    pub uuids: Vec<String>,
    pub total: usize,
}
```

One page of search hits in title order, and how many documents match.

**Fields**

- `uuids`: Library uuids of the page's hits (lowercased, as indexed), in
  case-insensitive title order, then uuid.
- `total`: Documents that match the query, before paging.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### SavedFilter

```rust
pub struct SavedFilter {
    pub name: String,
    pub query: String,
}
```

A saved filter expression (Bookclerk quick-filter parity).

**Fields**

- `name`: Operator-facing display name for this saved filter.
- `query`: Lucene-style query string (normalized before parse).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

## Enums

### SearchError

```rust
pub enum SearchError {
    Index(String),
    Busy,
    Query(String),
    Library(LibraryError),
}
```

Errors from opening, writing, or querying the Tantivy index.

**Variants**

- `Index`: Index open / writer / commit failure (operator-facing detail).
- `Busy`: Another writer (a rebuild, an event refresh, or the CLI) holds the
  index lock; the write can be retried shortly.
- `Query`: Query parse failure after Lucene-style normalization.
- `Library`: Failure loading library rows while rebuilding the index.

Implements: `Debug`, `Display`, `Error`, `From<LibraryError>`

## Functions

### normalize_lucene_query

```rust
pub fn normalize_lucene_query(input: &str) -> String
```

Normalize a classic Libation/Lucene query for Tantivy's parser.

Handles common shortcuts: `-acquired`, `[tag]`, field synonyms.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, SearchError>;
```

Result alias for search index and query helpers in this crate.
