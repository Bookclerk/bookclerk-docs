**bookclerk_integrations > listening**

# Module: listening

## Contents

**Functions**

- [`match_book_uuid`](#match_book_uuid) - Best-effort match to an owned library book via ASIN, ISBN, or exact title.
- [`upsert_listening_snapshots`](#upsert_listening_snapshots) - Upsert provider-agnostic listening snapshots into `listening_progress`.

---

## bookclerk_integrations::listening::match_book_uuid

*Function*

Best-effort match to an owned library book via ASIN, ISBN, or exact title.

# Arguments

* `library` - Open library store used for reads/writes.
* `asin` - Optional Audible / Amazon ASIN.
* `isbn` - Optional ISBN (any punctuation; normalized internally).
* `title` - Display title.

# Returns

On success, the inner `Option<String>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn match_book_uuid(library: &bookclerk_library::LibraryStore, asin: Option<&str>, isbn: Option<&str>, title: Option<&str>) -> crate::error::Result<Option<String>>
```



## bookclerk_integrations::listening::upsert_listening_snapshots

*Function*

Upsert provider-agnostic listening snapshots into `listening_progress`.

`provider` should be the integration id (`audiobookshelf`, plugin id, …).

# Arguments

* `library` - Open library store used for reads/writes.
* `provider` - Integration id.
* `items` - `items` input for this call.

# Returns

On success, the inner `usize` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn upsert_listening_snapshots(library: &bookclerk_library::LibraryStore, provider: &str, items: &[crate::types::ListeningProgressSnapshot]) -> crate::error::Result<usize>
```



