**bookclerk_search > engine**

# Module: engine

## Contents

**Structs**

- [`SearchEngine`](#searchengine) - On-disk Tantivy search engine (classic Lucene `SearchEngine` parity).
- [`SearchHit`](#searchhit) - One search hit.

---

## bookclerk_search::engine::SearchEngine

*Struct*

On-disk Tantivy search engine (classic Lucene `SearchEngine` parity).

**Methods:**

- `fn open(dir: &Path) -> Result<Self>` - Open or create a search index at `dir`.
- `fn rebuild(self: &Self, library: &LibraryStore) -> Result<usize>` - Rebuild the entire index from the library DB.
- `fn open_and_search(dir: PathBuf, query: String, limit: usize) -> Result<Vec<SearchHit>>` - Open an index and search it without blocking the caller's task.
- `fn search(self: &Self, raw_query: &str, limit: usize) -> Result<Vec<SearchHit>>` - Search the index. `limit` of 0 returns all matches (classic `-n 0`).



## bookclerk_search::engine::SearchHit

*Struct*

One search hit.

**Fields:**
- `asin: String` - Display / primary id (source `product_id`).
- `uuid: String` - Library public uuid (indexed lowercased; returned as stored).
- `account_id: String` - Library account id that owns this title.
- `title: String` - Display title stored with the hit.
- `score: f32` - Tantivy relevance score (higher is better).

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &SearchHit) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> SearchHit`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



