**bookclerk_discover > identity**

# Module: identity

## Contents

**Structs**

- [`StoreEdition`](#storeedition) - One storefront edition of a work (for multi-store purchase links).
- [`WorkIdentity`](#workidentity) - Bibliographic identity slice used for merge decisions.

**Functions**

- [`candidate_map_key`](#candidate_map_key) - Stable HashMap key for a storefront candidate (hard id, soft title+author, or source:product).
- [`hard_work_key`](#hard_work_key) - Hard bibliographic key: canonical ISBN first, then ASIN.
- [`identities_match`](#identities_match) - Whether two bibliographic identities refer to the same work.
- [`merge_candidate_metadata`](#merge_candidate_metadata) - Merge storefront metadata into `into`, preferring filled fields / richer text.
- [`merge_global_queue_entries`](#merge_global_queue_entries) - Merge wishlist queue rows that share ISBN/ASIN or soft title+author identity.
- [`merge_recommendation`](#merge_recommendation) - Merge a scored recommendation into an existing card.
- [`parse_series_index`](#parse_series_index) - Parse Audible/Chirp-style series indexes (`"1"`, `"1.5"`, `"Book 2"`, `"02"`).
- [`push_edition`](#push_edition) - Appends a store edition when not already present (dedupes on source+product_id).
- [`push_shelf_category`](#push_shelf_category) - Append a stable shelf-kind tag (`finish_series`, `author`, …) if missing.
- [`recommendation_map_key`](#recommendation_map_key) - Stable HashMap key for a recommendation card (same rules as [`work_map_key`](identity.md#bookclerk_discoveridentitywork_map_key)).
- [`soft_work_key`](#soft_work_key) - Soft key from cleaned title + primary author (exact string match only).
- [`work_map_key`](#work_map_key) - Best stable map key for a recommendation / candidate.
- [`works_match`](#works_match) - Whether two titles should be treated as the same work.

---

## bookclerk_discover::identity::StoreEdition

*Struct*

One storefront edition of a work (for multi-store purchase links).

**Fields:**
- `source: String` - Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).
- `product_id: String` - Storefront-native product id (ASIN, ISBN, UUID, …).

**Methods:**

- `fn new<impl Into<String>, impl Into<String>>(source: impl Trait, product_id: impl Trait) -> Self` - Creates a store edition from a storefront id and native product id.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **PartialEq**
  - `fn eq(self: &Self, other: &StoreEdition) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> StoreEdition`



## bookclerk_discover::identity::WorkIdentity

*Struct*

Bibliographic identity slice used for merge decisions.

**Generic Parameters:**
- 'a

**Fields:**
- `asin: Option<&'a str>` - Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn: Option<&'a str>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `title: &'a str` - Display title as shown on the storefront or library card.
- `authors: Option<&'a str>` - Comma-separated author names when the storefront provides them.
- `series: Option<&'a str>` - Series name when the title belongs to a named series.
- `series_index: Option<&'a str>` - Position within the series (e.g. `1`, `1.5`) when known.

**Methods:**

- `fn new(asin: Option<&'a str>, isbn: Option<&'a str>, title: &'a str, authors: Option<&'a str>) -> Self` - Creates a work identity from hard ids plus title and authors (no series yet).
- `fn with_series(self: Self, series: Option<&'a str>) -> Self` - Sets the optional series name used in soft identity matching.
- `fn with_series_index(self: Self, series_index: Option<&'a str>) -> Self` - Sets the optional series position used to avoid merging distinct volumes.

**Traits:** Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> WorkIdentity<'a>`



## bookclerk_discover::identity::candidate_map_key

*Function*

Stable HashMap key for a storefront candidate (hard id, soft title+author, or source:product).

# Arguments

* `c` - `c` input for this call.

# Returns

String result for this operation.

```rust
fn candidate_map_key(c: &crate::candidates::StorefrontCandidate) -> String
```



## bookclerk_discover::identity::hard_work_key

*Function*

Hard bibliographic key: canonical ISBN first, then ASIN.

Note: ISBN is not published by every storefront (Chirp / GraphicAudio /
Audible public search often omit it). Soft title+author matching then applies.

# Arguments

* `asin` - Optional Audible / Amazon ASIN.
* `isbn` - Optional ISBN (any punctuation; normalized internally).

# Returns

`Some(...)` when found / applicable; otherwise `None`.

```rust
fn hard_work_key(asin: Option<&str>, isbn: Option<&str>) -> Option<String>
```



## bookclerk_discover::identity::identities_match

*Function*

Whether two bibliographic identities refer to the same work.

# Arguments

* `a` - Left-hand vector.
* `b` - Right-hand vector.

# Returns

`true` when the predicate holds.

```rust
fn identities_match(a: WorkIdentity, b: WorkIdentity) -> bool
```



## bookclerk_discover::identity::merge_candidate_metadata

*Function*

Merge storefront metadata into `into`, preferring filled fields / richer text.

# Arguments

* `into` - `into` input for this call.
* `from` - `from` input for this call.

```rust
fn merge_candidate_metadata(into: & mut crate::candidates::StorefrontCandidate, from: &crate::candidates::StorefrontCandidate)
```



## bookclerk_discover::identity::merge_global_queue_entries

*Function*

Merge wishlist queue rows that share ISBN/ASIN or soft title+author identity.

Sums `wish_count`, unions store editions / purchase hints, and prefers
richer bibliographic metadata (longer strings / filled optionals). ISBN-keyed
`work_key` wins when available.

# Arguments

* `entries` - `entries` input for this call.

# Returns

Collected results (may be empty).

```rust
fn merge_global_queue_entries(entries: Vec<bookclerk_library::GlobalQueueEntry>) -> Vec<bookclerk_library::GlobalQueueEntry>
```



## bookclerk_discover::identity::merge_recommendation

*Function*

Merge a scored recommendation into an existing card.

# Arguments

* `into` - `into` input for this call.
* `from` - `from` input for this call.

```rust
fn merge_recommendation(into: & mut crate::recommend::Recommendation, from: crate::recommend::Recommendation)
```



## bookclerk_discover::identity::parse_series_index

*Function*

Parse Audible/Chirp-style series indexes (`"1"`, `"1.5"`, `"Book 2"`, `"02"`).

# Arguments

* `raw` - String `raw` for this call.

# Returns

`Some(...)` when found / applicable; otherwise `None`.

```rust
fn parse_series_index(raw: Option<&str>) -> Option<f64>
```



## bookclerk_discover::identity::push_edition

*Function*

Appends a store edition when not already present (dedupes on source+product_id).

# Arguments

* `editions` - `editions` input for this call.
* `edition` - `edition` input for this call.

```rust
fn push_edition(editions: & mut Vec<StoreEdition>, edition: StoreEdition)
```



## bookclerk_discover::identity::push_shelf_category

*Function*

Append a stable shelf-kind tag (`finish_series`, `author`, …) if missing.

# Arguments

* `categories` - String `categories` for this call.
* `kind` - Plugin kind string.

```rust
fn push_shelf_category(categories: & mut Vec<String>, kind: &str)
```



## bookclerk_discover::identity::recommendation_map_key

*Function*

Stable HashMap key for a recommendation card (same rules as [`work_map_key`](identity.md#bookclerk_discoveridentitywork_map_key)).

# Arguments

* `r` - `r` input for this call.

# Returns

String result for this operation.

```rust
fn recommendation_map_key(r: &crate::recommend::Recommendation) -> String
```



## bookclerk_discover::identity::soft_work_key

*Function*

Soft key from cleaned title + primary author (exact string match only).

Keeps subtitles so series volumes (`Infinity Blade: Awakening` vs
`…: Redemption`) do not share a HashMap bucket when hard ids are absent.
Soft [`identities_match`](identity.md#bookclerk_discoveridentityidentities_match) still joins edition variants (e.g. `A Novel`).

# Arguments

* `title` - Display title.
* `authors` - Optional author string (comma-separated).

# Returns

String result for this operation.

```rust
fn soft_work_key(title: &str, authors: Option<&str>) -> String
```



## bookclerk_discover::identity::work_map_key

*Function*

Best stable map key for a recommendation / candidate.

# Arguments

* `asin` - Optional Audible / Amazon ASIN.
* `isbn` - Optional ISBN (any punctuation; normalized internally).
* `title` - Display title.
* `authors` - Optional author string (comma-separated).
* `source` - Storefront id or filesystem source path, depending on call site.
* `product_id` - Storefront-native product id.

# Returns

String result for this operation.

```rust
fn work_map_key(asin: Option<&str>, isbn: Option<&str>, title: &str, authors: Option<&str>, source: Option<&str>, product_id: Option<&str>) -> String
```



## bookclerk_discover::identity::works_match

*Function*

Whether two titles should be treated as the same work.

# Arguments

* `title_a` - String `title_a` for this call.
* `authors_a` - String `authors_a` for this call.
* `title_b` - String `title_b` for this call.
* `authors_b` - String `authors_b` for this call.

# Returns

`true` when the predicate holds.

```rust
fn works_match(title_a: &str, authors_a: Option<&str>, title_b: &str, authors_b: Option<&str>) -> bool
```



