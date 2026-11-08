# bookclerk_storefront::text

Text normalization for storefront / catalog metadata.

| Item | Kind | Summary |
| --- | --- | --- |
| [`decode_html_entities_cow`](#decode_html_entities_cow) | function | Decode HTML entities in storefront metadata (`Memory&#39;s Blade` → `Memory's Blade`). |
| [`decode_html_entities`](#decode_html_entities) | function | Owned decode (allocates even when unchanged — prefer `decode_html_entities_cow` or `decode_html_entities_in_place` on hot paths). |
| [`decode_html_entities_in_place`](#decode_html_entities_in_place) | function | Decode in place when the value contains `&`; otherwise leave untouched. |
| [`decode_html_entities_opt_in_place`](#decode_html_entities_opt_in_place) | function | Decode an optional string in place. |
| [`decode_html_entities_opt`](#decode_html_entities_opt) | function | Decode when present; leave `None` unchanged. |
| [`str_maybe_html_entity`](#str_maybe_html_entity) | function | True when any human-readable field may contain an HTML entity. |

## Functions

### decode_html_entities_cow

```rust
pub fn decode_html_entities_cow(raw: &str) -> Cow<'_, str>
```

Decode HTML entities in storefront metadata (`Memory&#39;s Blade` → `Memory's Blade`).

Libro.fm (and occasionally other catalogs) emit entity-encoded titles and
descriptions. Decode before display, persistence, and soft identity matching.

Fast path: strings without `&` are returned borrowed (no allocation).

### decode_html_entities

```rust
pub fn decode_html_entities(raw: &str) -> String
```

Owned decode (allocates even when unchanged — prefer [`decode_html_entities_cow`](#decode_html_entities_cow)
or [`decode_html_entities_in_place`](#decode_html_entities_in_place) on hot paths).

### decode_html_entities_in_place

```rust
pub fn decode_html_entities_in_place(value: &mut String)
```

Decode in place when the value contains `&`; otherwise leave untouched.

### decode_html_entities_opt_in_place

```rust
pub fn decode_html_entities_opt_in_place(value: &mut Option<String>)
```

Decode an optional string in place.

### decode_html_entities_opt

```rust
pub fn decode_html_entities_opt(raw: Option<&str>) -> Option<String>
```

Decode when present; leave `None` unchanged.

### str_maybe_html_entity

```rust
pub fn str_maybe_html_entity(raw: &str) -> bool
```

True when any human-readable field may contain an HTML entity.
