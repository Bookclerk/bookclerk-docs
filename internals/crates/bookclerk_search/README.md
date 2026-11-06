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
    Query(String),
    Library(LibraryError),
}
```

Errors from opening, writing, or querying the Tantivy index.

**Variants**

- `Index`: Index open / writer / commit failure (operator-facing detail).
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
