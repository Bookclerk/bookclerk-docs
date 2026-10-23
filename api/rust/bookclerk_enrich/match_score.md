**bookclerk_enrich > match_score**

# Module: match_score

## Contents

**Structs**

- [`MatchQuery`](#matchquery) - Owned-title (e.g. Libro) metadata used as the match query.
- [`ScoreInput`](#scoreinput) - Candidate metadata used for confidence scoring.

**Functions**

- [`calculate_match_confidence`](#calculate_match_confidence) - Calculate match confidence in `[0.0, 1.0]`.
- [`canonicalize_isbn`](#canonicalize_isbn) - Normalize then convert ISBN-10 → ISBN-13 (`978…`) when possible.
- [`clean_author_for_compares`](#clean_author_for_compares) - Clean an author string for fuzzy compares (AudioBookshelf `cleanAuthorForCompares`).
- [`clean_title_for_compares`](#clean_title_for_compares) - Clean a title for fuzzy compares (AudioBookshelf `cleanTitleForCompares`).
- [`is_valid_asin`](#is_valid_asin) - True when `s` looks like an Audible ASIN (`^[A-Z0-9]{10}$`, case-insensitive).
- [`isbn_exact_match`](#isbn_exact_match) - Returns true when both sides normalize to the same ISBN-13 (or matching ISBN-10).
- [`levenshtein_distance`](#levenshtein_distance) - Levenshtein distance (case-insensitive by default, matching ABS).
- [`levenshtein_similarity`](#levenshtein_similarity) - Similarity in `[0.0, 1.0]` from Levenshtein distance.
- [`normalize_isbn`](#normalize_isbn) - Digits-only ISBN for exact matching (strips hyphens / spaces / `ISBN` prefix).

---

## bookclerk_enrich::match_score::MatchQuery

*Struct*

Owned-title (e.g. Libro) metadata used as the match query.

**Generic Parameters:**
- 'a

**Fields:**
- `title: &'a str` - Display title as shown on the storefront or library card.
- `subtitle: Option<&'a str>` - Optional subtitle when the catalog distinguishes it from the title.
- `author: Option<&'a str>` - Primary author name used for fuzzy matching.
- `narrator: Option<&'a str>` - Primary narrator name used for fuzzy matching.
- `isbn: Option<&'a str>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `duration_minutes: Option<f64>` - Candidate runtime in minutes for scoring against the seed.

**Trait Implementations:**

- **Default**
  - `fn default() -> MatchQuery<'a>`
- **Clone**
  - `fn clone(self: &Self) -> MatchQuery<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_enrich::match_score::ScoreInput

*Struct*

Candidate metadata used for confidence scoring.

**Generic Parameters:**
- 'a

**Fields:**
- `title: &'a str` - Display title as shown on the storefront or library card.
- `subtitle: Option<&'a str>` - Optional subtitle when the catalog distinguishes it from the title.
- `author: Option<&'a str>` - Primary author name used for fuzzy matching.
- `narrator: Option<&'a str>` - Primary narrator name used for fuzzy matching.
- `isbn: Option<&'a str>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `duration_minutes: Option<f64>` - Runtime in minutes (Audible / Audnexus `runtimeLengthMin`).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ScoreInput<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_enrich::match_score::calculate_match_confidence

*Function*

Calculate match confidence in `[0.0, 1.0]`.

Base score follows AudioBookshelf (duration / title / author). When Libro
(or other) query metadata is richer:
- **Narrator** (both sides present): small extra weight in the blend
- **Exact ISBN** (normalized digits): boost by closing
  `ISBN_MATCH_GAP_CLOSE` of the remaining gap to 1.0 (not a forced accept)

When the query title is an ASIN, returns `1.0`.

# Arguments

* `book` - Library book row to update.
* `query` - Query vector or free-text search string.

# Returns

`f64` result.

```rust
fn calculate_match_confidence(book: &ScoreInput, query: &MatchQuery) -> f64
```



## bookclerk_enrich::match_score::canonicalize_isbn

*Function*

Normalize then convert ISBN-10 → ISBN-13 (`978…`) when possible.

ISBN is **not** available from every storefront (Chirp / GraphicAudio / Audible
public catalog often omit it). When present, prefer this canonical form so
10- and 13-digit variants of the same book share one key.

# Arguments

* `raw` - String `raw` for this call.

# Returns

String result for this operation.

```rust
fn canonicalize_isbn(raw: &str) -> String
```



## bookclerk_enrich::match_score::clean_author_for_compares

*Function*

Clean an author string for fuzzy compares (AudioBookshelf `cleanAuthorForCompares`).

# Arguments

* `author` - String `author` for this call.

# Returns

String result for this operation.

```rust
fn clean_author_for_compares(author: &str) -> String
```



## bookclerk_enrich::match_score::clean_title_for_compares

*Function*

Clean a title for fuzzy compares (AudioBookshelf `cleanTitleForCompares`).

# Arguments

* `title` - Display title.
* `keep_subtitle` - When true, retain subtitle text for soft compares.

# Returns

String result for this operation.

```rust
fn clean_title_for_compares(title: &str, keep_subtitle: bool) -> String
```



## bookclerk_enrich::match_score::is_valid_asin

*Function*

True when `s` looks like an Audible ASIN (`^[A-Z0-9]{10}$`, case-insensitive).

Includes all-numeric catalog ids (e.g. `1094100765`); Audnexus accepts these
and older Audible storefront ASINs are often numeric.

# Arguments

* `s` - Input string to validate or normalize.

# Returns

`true` when the predicate holds.

```rust
fn is_valid_asin(s: &str) -> bool
```



## bookclerk_enrich::match_score::isbn_exact_match

*Function*

Returns true when both sides normalize to the same ISBN-13 (or matching ISBN-10).

# Arguments

* `query_isbn` - String `query_isbn` for this call.
* `book_isbn` - String `book_isbn` for this call.

# Returns

`true` when the predicate holds.

```rust
fn isbn_exact_match(query_isbn: Option<&str>, book_isbn: Option<&str>) -> bool
```



## bookclerk_enrich::match_score::levenshtein_distance

*Function*

Levenshtein distance (case-insensitive by default, matching ABS).

# Arguments

* `a` - Left-hand vector.
* `b` - Right-hand vector.

# Returns

`usize` result.

```rust
fn levenshtein_distance(a: &str, b: &str) -> usize
```



## bookclerk_enrich::match_score::levenshtein_similarity

*Function*

Similarity in `[0.0, 1.0]` from Levenshtein distance.

# Arguments

* `a` - Left-hand vector.
* `b` - Right-hand vector.

# Returns

`f64` result.

```rust
fn levenshtein_similarity(a: &str, b: &str) -> f64
```



## bookclerk_enrich::match_score::normalize_isbn

*Function*

Digits-only ISBN for exact matching (strips hyphens / spaces / `ISBN` prefix).

# Arguments

* `raw` - String `raw` for this call.

# Returns

String result for this operation.

```rust
fn normalize_isbn(raw: &str) -> String
```



