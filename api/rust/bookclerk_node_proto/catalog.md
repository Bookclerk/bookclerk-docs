**bookclerk_node_proto > catalog**

# Module: catalog

## Contents

**Structs**

- [`CatalogFile`](#catalogfile) - One file of a [`CatalogTitle`](catalog.md#bookclerk_node_protocatalogcatalogtitle).
- [`CatalogMetadata`](#catalogmetadata) - Bibliographic metadata of a [`CatalogTitle`](catalog.md#bookclerk_node_protocatalogcatalogtitle).
- [`CatalogRef`](#catalogref) - A title named by store and product id.
- [`CatalogTitle`](#catalogtitle) - One title, as a branch shares it.

**Functions**

- [`is_ancestry`](#is_ancestry) - True when every id in `ancestry` is a plain cluster id and there are at
- [`is_cluster_id`](#is_cluster_id) - True for a plain cluster id in an ancestry.
- [`is_product_id`](#is_product_id) - True for a product id: 1 to 256 bytes, no whitespace at the ends and no
- [`is_store_id`](#is_store_id) - True for a store id: a short lowercase token.

**Constants**

- [`CATALOG_ITEM`](#catalog_item) - Item kind of the catalog fence (`lost.item`), and the job id a catalog
- [`MAX_ANCESTRY`](#max_ancestry) - Most cluster ids an ancestry may list.
- [`MAX_CATALOG_BATCH_TITLES`](#max_catalog_batch_titles) - Most titles one `crate::Body::CatalogSync` batch may carry.
- [`MAX_CATALOG_CURSOR_BYTES`](#max_catalog_cursor_bytes) - Longest cursor.
- [`MAX_CATALOG_TITLE_BYTES`](#max_catalog_title_bytes) - Largest serialized title in a batch.
- [`MAX_CATALOG_WANTED`](#max_catalog_wanted) - Most titles one answer may say are wanted.

---

## bookclerk_node_proto::catalog::CATALOG_ITEM

*Constant*: `&str`

Item kind of the catalog fence (`lost.item`), and the job id a catalog
upload's token is sealed for.



## bookclerk_node_proto::catalog::CatalogFile

*Struct*

One file of a [`CatalogTitle`](catalog.md#bookclerk_node_protocatalogcatalogtitle).

**Fields:**
- `key: String` - Relative storage key (the same on every destination of the
- `size: u64` - Size in bytes.
- `content_type: Option<String>` - MIME type.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CatalogFile`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &CatalogFile) -> $crate::option::Option<$crate::cmp::Ordering>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogFile) -> bool`
- **Ord**
  - `fn cmp(self: &Self, other: &CatalogFile) -> $crate::cmp::Ordering`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_node_proto::catalog::CatalogMetadata

*Struct*

Bibliographic metadata of a [`CatalogTitle`](catalog.md#bookclerk_node_protocatalogcatalogtitle).

**Fields:**
- `title: String` - Display title.
- `subtitle: Option<String>` - Subtitle.
- `authors: Option<String>` - Authors as the store lists them.
- `narrators: Option<String>` - Narrators.
- `series: Option<String>` - Series name.
- `series_index: Option<String>` - Position in the series.
- `asin: Option<String>` - Amazon ASIN.
- `isbn: Option<String>` - ISBN.
- `publisher: Option<String>` - Publisher.
- `length_minutes: Option<i64>` - Runtime in minutes.
- `content_kind: Option<String>` - `book`, `episode`, ...
- `language: Option<String>` - Language code.
- `published_at: Option<String>` - Publication date text.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CatalogMetadata`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogMetadata) -> bool`
- **Default**
  - `fn default() -> CatalogMetadata`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_node_proto::catalog::CatalogRef

*Struct*

A title named by store and product id.

**Fields:**
- `store: String` - Store id.
- `product_id: String` - Product id.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Ord**
  - `fn cmp(self: &Self, other: &CatalogRef) -> $crate::cmp::Ordering`
- **Clone**
  - `fn clone(self: &Self) -> CatalogRef`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogRef) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &CatalogRef) -> $crate::option::Option<$crate::cmp::Ordering>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::catalog::CatalogTitle

*Struct*

One title, as a branch shares it.

**Fields:**
- `store: String` - The store that sells it, as the store's plain id (`audible`,
- `product_id: String` - The store's product id (ASIN, ISBN, ...).
- `metadata: CatalogMetadata` - Bibliographic metadata.
- `primary_key: String` - Storage key of the primary audio file (one of `files`).
- `pdf_key: Option<String>` - Storage key of the companion PDF (one of `files`), when shared.
- `files: Vec<CatalogFile>` - Every file of the title.

**Methods:**

- `fn reference(self: &Self) -> CatalogRef` - The title's [`CatalogRef`](catalog.md#bookclerk_node_protocatalogcatalogref).
- `fn problem(self: &Self) -> Option<&'static str>` - Why the title is malformed, or `None`: the store and product id, a
- `fn manifest_digest(self: &Self) -> String` - Lowercase hex SHA-256 of the title's manifest (primary key, PDF key

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogTitle) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CatalogTitle`



## bookclerk_node_proto::catalog::MAX_ANCESTRY

*Constant*: `usize`

Most cluster ids an ancestry may list.



## bookclerk_node_proto::catalog::MAX_CATALOG_BATCH_TITLES

*Constant*: `usize`

Most titles one `crate::Body::CatalogSync` batch may carry.



## bookclerk_node_proto::catalog::MAX_CATALOG_CURSOR_BYTES

*Constant*: `usize`

Longest cursor.



## bookclerk_node_proto::catalog::MAX_CATALOG_TITLE_BYTES

*Constant*: `usize`

Largest serialized title in a batch.



## bookclerk_node_proto::catalog::MAX_CATALOG_WANTED

*Constant*: `usize`

Most titles one answer may say are wanted.



## bookclerk_node_proto::catalog::is_ancestry

*Function*

True when every id in `ancestry` is a plain cluster id and there are at
most [`MAX_ANCESTRY`](catalog.md#bookclerk_node_protocatalogmax_ancestry).

```rust
fn is_ancestry(ancestry: &[String]) -> bool
```



## bookclerk_node_proto::catalog::is_cluster_id

*Function*

True for a plain cluster id in an ancestry.

```rust
fn is_cluster_id(id: &str) -> bool
```



## bookclerk_node_proto::catalog::is_product_id

*Function*

True for a product id: 1 to 256 bytes, no whitespace at the ends and no
control characters.

```rust
fn is_product_id(product_id: &str) -> bool
```



## bookclerk_node_proto::catalog::is_store_id

*Function*

True for a store id: a short lowercase token.

```rust
fn is_store_id(store: &str) -> bool
```



