# bookclerk_search

Tantivy full-text search over the library's works.

## Audience

Host library / GUI code that builds and queries the on-disk search index.
Guest plugins do not depend on this crate.

Style: `docs/contributing/code-documentation.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`WorkHit`](#workhit) | struct | One work a search found. |
| [`WorkMatches`](#workmatches) | struct | One window of a query's matches. |
| [`WorkSearchIndex`](#worksearchindex) | struct | On-disk Tantivy index of works. |
| [`SavedFilter`](#savedfilter) | struct | A saved filter expression (Bookclerk quick-filter parity). |
| [`SearchError`](#searcherror) | enum | Errors from opening, writing, or querying the Tantivy index. |
| [`Result`](#result) | type alias | Result alias for search index and query helpers in this crate. |

## Structs

### WorkHit

```rust
pub struct WorkHit {
    pub work_id: String,
    pub title: String,
    pub score: f32,
}
```

One work a search found.

**Fields**

- `work_id`: The top-level work's id.
- `title`: The work's title, as indexed.
- `score`: Tantivy relevance score; higher is a better match.

Implements: `Clone`, `Debug`, `PartialEq`

### WorkMatches

```rust
pub struct WorkMatches {
    pub ids: Vec<String>,
    pub total: usize,
}
```

One window of a query's matches.

**Fields**

- `ids`: Work ids in the window, best match first.
- `total`: How many documents the query matches in all.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WorkSearchIndex

```rust
pub struct WorkSearchIndex {
    /* private fields */
}
```

On-disk Tantivy index of works.

#### WorkSearchIndex::open

```rust
pub fn open(dir: &Path) -> Result<Self>
```

Opens or creates the works index in `dir`, replacing an index of
another layout.

##### Errors

Returns [`SearchError::Index`](#searcherror) when the directory or index cannot be
opened.

#### WorkSearchIndex::open_for_search

```rust
pub fn open_for_search(dir: &Path) -> Result<Self>
```

Opens the index in `dir` for searching only: nothing is created or
replaced, so a search never races a writer over the directory.

##### Errors

Returns [`SearchError::NotReady`](#searcherror) when `dir` holds no index of the
current layout (the rebuild has not run yet), or
[`SearchError::Index`](#searcherror) when it cannot be opened.

#### WorkSearchIndex::is_complete

```rust
pub fn is_complete(dir: &Path) -> bool
```

True when a full rebuild of the current layout has finished in `dir`.

#### WorkSearchIndex::mark_incomplete

```rust
pub fn mark_incomplete(&self) -> Result<()>
```

Forgets that the index was fully rebuilt, so the daemon queues a new
rebuild.

##### Errors

Returns [`SearchError::Index`](#searcherror) when the marker cannot be removed.

#### WorkSearchIndex::refresh_works

```rust
pub async fn refresh_works(
    &self,
    library: &LibraryStore,
    work_ids: &[String],
) -> Result<usize>
```

Replaces the documents of `work_ids` with what the library holds
now: a held work's document is written again, and an unheld or
missing work's is removed.

##### Returns

How many documents were written.

##### Errors

Returns [`SearchError::Busy`](#searcherror) when another writer holds the index,
or the library or index error.

#### WorkSearchIndex::rebuild

```rust
pub async fn rebuild(&self, library: &LibraryStore) -> Result<usize>
```

Rebuilds the whole index from the library, walking every top-level
work in pages, then leaves the rebuild marker.

##### Returns

How many documents were written.

##### Errors

Returns [`SearchError::Busy`](#searcherror) when another writer holds the index,
or the library or index error.

#### WorkSearchIndex::rebuild_waiting

```rust
pub async fn rebuild_waiting(
    &self,
    library: &LibraryStore,
    max_wait: Duration,
) -> Result<usize>
```

[`Self::rebuild`](#worksearchindexrebuild), waiting while another writer holds the index.

A running daemon refreshes this index from library events, and each
refresh holds the writer for a moment. A rebuild that finds the
index busy has written nothing, so it pauses (100 ms, doubling to
2 s) and tries again until `max_wait` has passed.

##### Arguments

* `library` - The library the works are read from.
* `max_wait` - How long to keep trying while the index is busy.

##### Returns

How many documents were written.

##### Errors

Returns [`SearchError::Busy`](#searcherror) when the index is still busy after
`max_wait`, and any other error of [`Self::rebuild`](#worksearchindexrebuild) at once.

#### WorkSearchIndex::open_and_match_ids

```rust
pub async fn open_and_match_ids(
    dir: PathBuf,
    query: String,
    offset: usize,
    limit: usize,
) -> Result<WorkMatches>
```

Opens the index in `dir` for searching and reads one window of a
query's matches without blocking the caller's task (see
[`Self::open_for_search`](#worksearchindexopen_for_search) and [`Self::match_ids`](#worksearchindexmatch_ids)).

##### Errors

Propagates open and query failures.

#### WorkSearchIndex::match_ids

```rust
pub fn match_ids(
    &self,
    raw_query: &str,
    offset: usize,
    limit: usize,
) -> Result<WorkMatches>
```

The work ids of the `limit` matches of `raw_query` after the first
`offset`, best first (ties in index order), and the total match
count. Only the window's documents are read. Blocking.

##### Errors

Returns [`SearchError::Query`](#searcherror) for a query that does not parse, or
[`SearchError::Index`](#searcherror) when the index cannot be read.

#### WorkSearchIndex::search

```rust
pub fn search(&self, raw_query: &str, limit: usize) -> Result<Vec<WorkHit>>
```

The best `limit` matches of `raw_query` with their titles, best
first; a `limit` of 0 returns every match. Blocking.

##### Errors

Returns [`SearchError::Query`](#searcherror) for a query that does not parse, or
[`SearchError::Index`](#searcherror) when the index cannot be read.

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
    NotReady,
    Query(String),
    Library(LibraryError),
}
```

Errors from opening, writing, or querying the Tantivy index.

**Variants**

- `Index`: Index open / writer / commit failure (operator-facing detail).
- `Busy`: Another writer (a rebuild, an event refresh, or the CLI) holds the
  index lock; the write can be retried shortly.
- `NotReady`: The index has not been built under the current layout yet; the
  daemon's rebuild job is building it.
- `Query`: Query parse failure after Lucene-style normalization.
- `Library`: Failure loading library rows while rebuilding the index.

Implements: `Debug`, `Display`, `Error`, `From<LibraryError>`

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, SearchError>;
```

Result alias for search index and query helpers in this crate.
