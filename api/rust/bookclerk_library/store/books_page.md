**bookclerk_library > store > books_page**

# Module: store::books_page

## Contents

**Structs**

- [`BookPage`](#bookpage) - One page of books plus the unpaged match count.

**Constants**

- [`BOOK_PAGE_DEFAULT_LIMIT`](#book_page_default_limit) - Default page size for `LibraryStore::list_books_filtered_page`.
- [`BOOK_PAGE_MAX_LIMIT`](#book_page_max_limit) - Upper clamp for a library book page. Matches the HTTP handler.

---

## bookclerk_library::store::books_page::BOOK_PAGE_DEFAULT_LIMIT

*Constant*: `u64`

Default page size for `LibraryStore::list_books_filtered_page`.



## bookclerk_library::store::books_page::BOOK_PAGE_MAX_LIMIT

*Constant*: `u64`

Upper clamp for a library book page. Matches the HTTP handler.



## bookclerk_library::store::books_page::BookPage

*Struct*

One page of books plus the unpaged match count.

**Fields:**
- `books: Vec<crate::models::BookRecord>` - Rows after `ORDER BY` and `LIMIT`/`OFFSET`.
- `total: usize` - Rows matching the filter before pagination.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BookPage`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



