**bookclerk_discover > works**

# Module: works

## Contents

**Functions**

- [`rebuild_works_from_library`](#rebuild_works_from_library) - Link every book into a work keyed by ASIN and/or ISBN, else book uuid.

---

## bookclerk_discover::works::rebuild_works_from_library

*Function*

Link every book into a work keyed by ASIN and/or ISBN, else book uuid.

When a book has both identifiers, prefer an existing work that already
carries either alias so ISBN-only and ASIN-only rows consolidate.

# Arguments

* `library` - Open library store used for reads/writes.

# Returns

On success, the inner `usize` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn rebuild_works_from_library(library: &bookclerk_library::LibraryStore) -> std::result::Result<usize, DiscoverError>
```



