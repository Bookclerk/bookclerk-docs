**bookclerk_search**

# Module: bookclerk_search

## Contents

**Structs**

- [`SavedFilter`](#savedfilter) - A saved filter expression (Bookclerk quick-filter parity).

**Enums**

- [`SearchError`](#searcherror) - Errors from opening, writing, or querying the Tantivy index.

**Type Aliases**

- [`Result`](#result) - Result alias for search index and query helpers in this crate.

---

## bookclerk_search::Result

*Type Alias*: `std::result::Result<T, SearchError>`

Result alias for search index and query helpers in this crate.



## bookclerk_search::SavedFilter

*Struct*

A saved filter expression (Bookclerk quick-filter parity).

**Fields:**
- `name: String` - Operator-facing display name for this saved filter.
- `query: String` - Lucene-style query string (normalized before parse).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SavedFilter`



## bookclerk_search::SearchError

*Enum*

Errors from opening, writing, or querying the Tantivy index.

**Variants:**
- `Index(String)` - Index open / writer / commit failure (operator-facing detail).
- `Query(String)` - Query parse failure after Lucene-style normalization.
- `Library(bookclerk_library::LibraryError)` - Failure loading library rows while rebuilding the index.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(source: bookclerk_library::LibraryError) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`



