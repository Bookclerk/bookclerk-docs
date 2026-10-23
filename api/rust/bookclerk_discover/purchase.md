**bookclerk_discover > purchase**

# Module: purchase

## Contents

**Structs**

- [`PurchaseHint`](#purchasehint) - A purchase / catalog availability hint (optionally priced at view time).
- [`PurchaseHintsQuery`](#purchasehintsquery) - Inputs for view-time catalog + pricing lookup.
- [`PurchaseHintsResponse`](#purchasehintsresponse) - Priced catalog matches for one title, sorted best-first.

**Functions**

- [`best_purchase_hint`](#best_purchase_hint) - Pick the lowest-priced hint (ties keep earlier order). Unpriced sort after priced.
- [`best_purchase_hint_preferring`](#best_purchase_hint_preferring) - Pick the offer the caller would actually pay least for.
- [`format_money_label`](#format_money_label) - Formats integer cents into a storefront-style money label for `currency`.
- [`parse_money_label_to_cents`](#parse_money_label_to_cents) - Parse `$12.34` / `12.34` / `FREE` into cents.
- [`purchase_hints_for`](#purchase_hints_for) - Look up purchase links via registered
- [`resolve_purchase_hints`](#resolve_purchase_hints) - Resolve every catalog match and attach live prices (view-time).
- [`resolve_purchase_hints_batch`](#resolve_purchase_hints_batch) - Resolve many purchase-hint queries with bounded concurrency (order preserved).
- [`seed_purchase_hint`](#seed_purchase_hint) - Seed a deterministic storefront URL from a known candidate (no remote I/O).

---

## bookclerk_discover::purchase::PurchaseHint

*Struct*

A purchase / catalog availability hint (optionally priced at view time).

**Fields:**
- `source: String` - Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).
- `product_id: String` - Storefront-native product id (ASIN, ISBN, UUID, …).
- `title: Option<String>` - Display title as shown on the storefront or library card.
- `url: Option<String>` - Absolute HTTPS URL.
- `price_cents: Option<i64>` - Primary / best known sell price in minor units (prefer member when dual).
- `currency: Option<String>` - ISO 4217 currency code for price fields (`USD`, `EUR`, …).
- `price_label: Option<String>` - Display string from the store (`$2.99`, `FREE`, …).
- `list_price_cents: Option<i64>` - Non-member / list price in integer cents when known.
- `list_price_label: Option<String>` - Human-readable list price from the storefront.
- `member_price_cents: Option<i64>` - Member / credit price in integer cents when known.
- `member_price_label: Option<String>` - Human-readable member price from the storefront.

**Methods:**

- `fn link<impl Into<String>, impl Into<String>>(source: impl Trait, product_id: impl Trait, title: Option<String>, url: Option<String>) -> Self` - URL-only catalog link (no price yet).

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PurchaseHint) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PurchaseHint`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> PurchaseHint`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::purchase::PurchaseHintsQuery

*Struct*

Inputs for view-time catalog + pricing lookup.

**Fields:**
- `title: String` - Display title as shown on the storefront or library card.
- `authors: Option<String>` - Comma-separated author names when the storefront provides them.
- `asin: Option<String>` - Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn: Option<String>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `candidate_source: Option<String>` - Storefront id of the edition that produced this card.
- `candidate_product_id: Option<String>` - Storefront product id of the edition that produced this card.
- `store_editions: Vec<crate::identity::StoreEdition>` - Known storefront editions already on the recommendation card.
- `region: Option<String>` - Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `preferred_sources: Vec<String>` - Storefronts the caller has linked accounts for. Treated as **member**

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PurchaseHintsQuery`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> PurchaseHintsQuery`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::purchase::PurchaseHintsResponse

*Struct*

Priced catalog matches for one title, sorted best-first.

**Fields:**
- `hints: Vec<PurchaseHint>` - Resolved purchase hints for this query.
- `best: Option<PurchaseHint>` - Lowest-priced hint (or first catalog hit when no prices resolved).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PurchaseHintsResponse`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::purchase::best_purchase_hint

*Function*

Pick the lowest-priced hint (ties keep earlier order). Unpriced sort after priced.

# Arguments

* `hints` - Purchase hints to pick from.

# Returns

`Some(...)` when found / applicable; otherwise `None`.

```rust
fn best_purchase_hint(hints: &[PurchaseHint]) -> Option<&PurchaseHint>
```



## bookclerk_discover::purchase::best_purchase_hint_preferring

*Function*

Pick the offer the caller would actually pay least for.

Linked storefronts (`preferred`) are compared at **member** price; every
other store is compared at **list / non-member** price. That way a Libro-only
member still sees Audible on the shelf when Audible’s non-member price beats
Libro’s member price. When nothing is linked, all stores are compared as
non-members. Hints with no usable price for that role sort last.

# Arguments

* `hints` - Purchase hints to pick from.
* `preferred` - String `preferred` for this call.

# Returns

`Some(...)` when found / applicable; otherwise `None`.

```rust
fn best_purchase_hint_preferring<'a>(hints: &'a [PurchaseHint], preferred: &std::collections::HashSet<String>) -> Option<&'a PurchaseHint>
```



## bookclerk_discover::purchase::format_money_label

*Function*

Formats integer cents into a storefront-style money label for `currency`.

# Arguments

* `cents` - Integer cents to format.
* `currency` - ISO 4217 currency code.

# Returns

String result for this operation.

```rust
fn format_money_label(cents: i64, currency: &str) -> String
```



## bookclerk_discover::purchase::parse_money_label_to_cents

*Function*

Parse `$12.34` / `12.34` / `FREE` into cents.

# Arguments

* `raw` - String `raw` for this call.

# Returns

`Some(...)` when found / applicable; otherwise `None`.

```rust
fn parse_money_label_to_cents(raw: &str) -> Option<i64>
```



## bookclerk_discover::purchase::purchase_hints_for

*Function*

Look up purchase links via registered
`bookclerk_source::ContentSource::purchase_hint` (no prices).

Call [`resolve_purchase_hints`](purchase.md#bookclerk_discoverpurchaseresolve_purchase_hints) for multi-store + live pricing.

# Arguments

* `registry` - Configured content-source or integration registry.
* `title` - Display title.
* `author` - String `author` for this call.
* `asin` - Optional Audible / Amazon ASIN.
* `isbn` - Optional ISBN (any punctuation; normalized internally).
* `region` - Marketplace / region code (`us`, `uk`, …).

# Returns

On success, the inner `Vec<PurchaseHint>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn purchase_hints_for(registry: &bookclerk_source::SourceRegistry, title: &str, author: Option<&str>, asin: Option<&str>, isbn: Option<&str>, region: &str) -> std::result::Result<Vec<PurchaseHint>, DiscoverError>
```



## bookclerk_discover::purchase::resolve_purchase_hints

*Function*

Resolve every catalog match and attach live prices (view-time).

# Arguments

* `registry` - Configured content-source or integration registry.
* `query` - Query vector or free-text search string.

# Returns

On success, the inner `PurchaseHintsResponse` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn resolve_purchase_hints(registry: &bookclerk_source::SourceRegistry, query: &PurchaseHintsQuery) -> std::result::Result<PurchaseHintsResponse, DiscoverError>
```



## bookclerk_discover::purchase::resolve_purchase_hints_batch

*Function*

Resolve many purchase-hint queries with bounded concurrency (order preserved).

# Arguments

* `registry` - Configured content-source or integration registry.
* `queries` - Batch of purchase-hint or title-meta queries.
* `max_concurrent` - Numeric `max_concurrent` value for this call.

# Returns

On success, `Vec<Result<PurchaseHintsResponse>>`.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn resolve_purchase_hints_batch(registry: &bookclerk_source::SourceRegistry, queries: &[PurchaseHintsQuery], max_concurrent: usize) -> Vec<std::result::Result<PurchaseHintsResponse, DiscoverError>>
```



## bookclerk_discover::purchase::seed_purchase_hint

*Function*

Seed a deterministic storefront URL from a known candidate (no remote I/O).

# Arguments

* `source` - Storefront id or filesystem source path, depending on call site.
* `product_id` - Storefront-native product id.
* `title` - Display title.
* `region` - Marketplace / region code (`us`, `uk`, …).

# Returns

`Some(...)` when found / applicable; otherwise `None`.

```rust
fn seed_purchase_hint(source: &str, product_id: &str, title: Option<String>, region: &str) -> Option<PurchaseHint>
```



