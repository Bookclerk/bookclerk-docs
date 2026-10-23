**bookclerk_library > text**

# Module: text

## Contents

**Functions**

- [`decode_html_entities`](#decode_html_entities) - Owned decode (allocates even when unchanged — prefer [`decode_html_entities_cow`](text.md#bookclerk_librarytextdecode_html_entities_cow)
- [`decode_html_entities_cow`](#decode_html_entities_cow) - Decode HTML entities in storefront metadata (`Memory&#39;s Blade` → `Memory's Blade`).
- [`decode_html_entities_in_place`](#decode_html_entities_in_place) - Decode in place when the value contains `&`; otherwise leave untouched.
- [`decode_html_entities_opt`](#decode_html_entities_opt) - Decode when present; leave `None` unchanged.
- [`decode_html_entities_opt_in_place`](#decode_html_entities_opt_in_place) - Decode an optional string in place.
- [`str_maybe_html_entity`](#str_maybe_html_entity) - True when any human-readable field may contain an HTML entity.

---

## bookclerk_library::text::decode_html_entities

*Function*

Owned decode (allocates even when unchanged — prefer [`decode_html_entities_cow`](text.md#bookclerk_librarytextdecode_html_entities_cow)
or [`decode_html_entities_in_place`](text.md#bookclerk_librarytextdecode_html_entities_in_place) on hot paths).

```rust
fn decode_html_entities(raw: &str) -> String
```



## bookclerk_library::text::decode_html_entities_cow

*Function*

Decode HTML entities in storefront metadata (`Memory&#39;s Blade` → `Memory's Blade`).

Libro.fm (and occasionally other catalogs) emit entity-encoded titles and
descriptions. Decode before display, persistence, and soft identity matching.

Fast path: strings without `&` are returned borrowed (no allocation).

```rust
fn decode_html_entities_cow(raw: &str) -> std::borrow::Cow<str>
```



## bookclerk_library::text::decode_html_entities_in_place

*Function*

Decode in place when the value contains `&`; otherwise leave untouched.

```rust
fn decode_html_entities_in_place(value: & mut String)
```



## bookclerk_library::text::decode_html_entities_opt

*Function*

Decode when present; leave `None` unchanged.

```rust
fn decode_html_entities_opt(raw: Option<&str>) -> Option<String>
```



## bookclerk_library::text::decode_html_entities_opt_in_place

*Function*

Decode an optional string in place.

```rust
fn decode_html_entities_opt_in_place(value: & mut Option<String>)
```



## bookclerk_library::text::str_maybe_html_entity

*Function*

True when any human-readable field may contain an HTML entity.

```rust
fn str_maybe_html_entity(raw: &str) -> bool
```



