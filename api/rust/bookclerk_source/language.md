**bookclerk_source > language**

# Module: language

## Contents

**Functions**

- [`default_preferred_language`](#default_preferred_language) - Preferred language for Discover when the client omits one.
- [`language_rank`](#language_rank) - Rank a hit's language relative to the preferred code.
- [`normalize_language`](#normalize_language) - Normalize a language tag or display name to a primary ISO-ish code.

---

## bookclerk_source::language::default_preferred_language

*Function*

Preferred language for Discover when the client omits one.

```rust
fn default_preferred_language() -> &'static str
```



## bookclerk_source::language::language_rank

*Function*

Rank a hit's language relative to the preferred code.

- `0` — matches preferred
- `1` — unknown / missing
- `2` — other language

```rust
fn language_rank(hit_language: Option<&str>, preferred: &str) -> u8
```



## bookclerk_source::language::normalize_language

*Function*

Normalize a language tag or display name to a primary ISO-ish code.

Accepts BCP-47 (`en-US`), bare codes (`en`), and common Audible display
names (`english`). Returns `None` for empty / unrecognized input.

```rust
fn normalize_language(raw: &str) -> Option<String>
```



