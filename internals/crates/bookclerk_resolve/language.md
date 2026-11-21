# bookclerk_resolve::language

Language codes: one spelling for the many ways stores write a language.

Stores name the same language differently (`en`, `en-US`, `eng`,
`English`), and the resolver compares expressions by language, so every
listing's language goes through [`normalize_language`](#normalize_language) before it is
compared or stored. Discover's language filters use the same function
(re-exported by `bookclerk-storefront`).

## Functions

### normalize_language

```rust
pub fn normalize_language(raw: &str) -> Option<String>
```

Normalize a language tag or display name to a primary ISO-ish code.

Accepts BCP-47 (`en-US`), bare codes (`en`), ISO 639-2 codes in either
their terminology or bibliographic (MARC) form (`fra` / `fre`, as
Libro.fm sends them), and common Audible display names (`english`).
Returns `None` for empty / unrecognized input.
