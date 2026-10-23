**bookclerk_discover > openlibrary**

# Module: openlibrary

## Contents

**Structs**

- [`OpenLibraryOptions`](#openlibraryoptions) - Options for a polite Open Library enrichment pass.

**Functions**

- [`enrich_books_from_openlibrary`](#enrich_books_from_openlibrary) - Fill missing description/subjects/cover/language from Open Library.
- [`enrich_books_from_openlibrary_with`](#enrich_books_from_openlibrary_with) - Same as [`enrich_books_from_openlibrary`](openlibrary.md#bookclerk_discoveropenlibraryenrich_books_from_openlibrary) with explicit politeness options.

---

## bookclerk_discover::openlibrary::OpenLibraryOptions

*Struct*

Options for a polite Open Library enrichment pass.

**Fields:**
- `contact_email: Option<String>` - Contact email embedded in User-Agent (required for identified rate limit).
- `min_interval: std::time::Duration` - Minimum delay between Open Library HTTP calls.
- `max_requests: usize` - Hard cap on Open Library HTTP requests for one enrich pass.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> OpenLibraryOptions`
- **Default**
  - `fn default() -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::openlibrary::enrich_books_from_openlibrary

*Function*

Fill missing description/subjects/cover/language from Open Library.

Skips rows already enriched from Open Library. Caps remote calls per run and
spaces requests to respect Open Library rate guidelines.

# Arguments

* `library` - Open library store used for reads/writes.

# Returns

On success, the inner `usize` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn enrich_books_from_openlibrary(library: &bookclerk_library::LibraryStore) -> std::result::Result<usize, DiscoverError>
```



## bookclerk_discover::openlibrary::enrich_books_from_openlibrary_with

*Function*

Same as [`enrich_books_from_openlibrary`](openlibrary.md#bookclerk_discoveropenlibraryenrich_books_from_openlibrary) with explicit politeness options.

# Arguments

* `library` - Open library store used for reads/writes.
* `opts` - Options struct for this operation.

# Returns

On success, the inner `usize` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn enrich_books_from_openlibrary_with(library: &bookclerk_library::LibraryStore, opts: &OpenLibraryOptions) -> std::result::Result<usize, DiscoverError>
```



