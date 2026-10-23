**bookclerk_discover > shelves**

# Module: shelves

## Contents

**Structs**

- [`DiscoverFeed`](#discoverfeed) - Full Discover page payload: ordered shelves of personalized candidates.
- [`DiscoverShelf`](#discovershelf) - One horizontal Discover row (series to finish, more by author, …).
- [`ShelfKindInfo`](#shelfkindinfo) - A shelf kind the operator can ignore (all offered by default).
- [`ShelfTaste`](#shelftaste) - Taste context used to title and fill personalized shelves.

**Functions**

- [`build_discover_feed`](#build_discover_feed) - Partition scored recommendations into Discover shelves (items may repeat).
- [`flatten_feed`](#flatten_feed) - Flatten shelves into a unique score-sorted list (CLI / legacy).
- [`shelf_is_disabled`](#shelf_is_disabled) - Whether `shelf_id` matches an ignore entry (exact, kind prefix, or `from_store`).
- [`shelf_kind_catalog`](#shelf_kind_catalog) - Shelf kinds Discover can emit (for config / UI ignore lists).

---

## bookclerk_discover::shelves::DiscoverFeed

*Struct*

Full Discover page payload: ordered shelves of personalized candidates.

**Fields:**
- `shelves: Vec<DiscoverShelf>` - Ordered horizontal shelves that make up the Discover page.
- `shelf_kinds: Vec<ShelfKindInfo>` - Catalog of shelf kinds for ignore prefs (empty `disabled_shelves` = all on).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DiscoverFeed`
- **Default**
  - `fn default() -> DiscoverFeed`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::shelves::DiscoverShelf

*Struct*

One horizontal Discover row (series to finish, more by author, …).

**Fields:**
- `id: String` - Stable identifier for this item.
- `title: String` - Display title as shown on the storefront or library card.
- `subtitle: Option<String>` - Optional subtitle when the catalog distinguishes it from the title.
- `items: Vec<crate::recommend::Recommendation>` - Ordered collection of child entries for this response.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DiscoverShelf`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::shelves::ShelfKindInfo

*Struct*

A shelf kind the operator can ignore (all offered by default).

**Fields:**
- `id: String` - Stable kind id (`finish_series`, `author`, `genre`, `from_store`, …).
- `label: String` - Operator-visible label for this account or item.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ShelfKindInfo`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::shelves::ShelfTaste

*Struct*

Taste context used to title and fill personalized shelves.

**Fields:**
- `authors: std::collections::HashMap<String, (String, f64)>` - Lowercase author → (display name, weight).
- `narrators: std::collections::HashMap<String, (String, f64)>` - Lowercase narrator → (display name, weight).
- `categories: std::collections::HashMap<String, (String, f64)>` - Lowercase category/subject → (display name, weight).
- `seed_authors_by_title: std::collections::HashMap<String, Vec<String>>` - Seed title (lowercase) → authors on that seed (display).
- `owned_sources: std::collections::HashSet<String>` - Storefronts represented in the owned library (`audible`, `libro`, …).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ShelfTaste`
- **Default**
  - `fn default() -> ShelfTaste`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::shelves::build_discover_feed

*Function*

Partition scored recommendations into Discover shelves (items may repeat).

# Arguments

* `recs` - Scored recommendations to partition into shelves.
* `taste` - Local taste context used to title personalized shelves.
* `per_shelf` - Max items per shelf (clamped).
* `disabled_shelves` - Shelf kind ids the operator has ignored.

# Returns

`DiscoverFeed` result.

```rust
fn build_discover_feed(recs: &[crate::recommend::Recommendation], taste: &ShelfTaste, per_shelf: usize, disabled_shelves: &[String]) -> DiscoverFeed
```



## bookclerk_discover::shelves::flatten_feed

*Function*

Flatten shelves into a unique score-sorted list (CLI / legacy).

# Arguments

* `feed` - Discover feed to flatten.
* `limit` - Maximum number of results to return.

# Returns

Collected results (may be empty).

```rust
fn flatten_feed(feed: &DiscoverFeed, limit: usize) -> Vec<crate::recommend::Recommendation>
```



## bookclerk_discover::shelves::shelf_is_disabled

*Function*

Whether `shelf_id` matches an ignore entry (exact, kind prefix, or `from_store`).

# Arguments

* `shelf_id` - Shelf id to test against the ignore list.
* `disabled` - Ignore-list entries from user preferences.

# Returns

`true` when the predicate holds.

```rust
fn shelf_is_disabled(shelf_id: &str, disabled: &[String]) -> bool
```



## bookclerk_discover::shelves::shelf_kind_catalog

*Function*

Shelf kinds Discover can emit (for config / UI ignore lists).

```rust
fn shelf_kind_catalog() -> Vec<ShelfKindInfo>
```



