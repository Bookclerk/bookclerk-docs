# bookclerk_cover::http_cache

HTTP cache validators for the images the daemon serves.

Every image the cover service answers with carries a strong
[`EntityTag`](#entitytag) derived from its bytes, so a browser that already holds a
copy can ask again with `If-None-Match` and get a `304 Not Modified`
instead of the body. [`if_none_match`](#if_none_match) reads that request header.

The header comes from whoever sends the request, so it is untrusted
input: the reader works on a byte slice with no I/O, never allocates in
proportion to the input, and treats anything it cannot read as "no
match", which only ever costs a full `200` answer. The `cache_headers`
fuzz target drives it against an independent oracle.

| Item | Kind | Summary |
| --- | --- | --- |
| [`EntityTag`](#entitytag) | struct | A strong entity tag naming one exact image body. |
| [`if_none_match`](#if_none_match) | function | Whether an `If-None-Match` header matches `tag`, so the answer may be `304 Not Modified`. |
| [`MAX_IF_NONE_MATCH_BYTES`](#max_if_none_match_bytes) | constant | Longest `If-None-Match` header read, in bytes. A browser sends one tag; a longer header is treated as no match rather than scanned. |

## Structs

### EntityTag

```rust
pub struct EntityTag {
    /* private fields */
}
```

A strong entity tag naming one exact image body.

The opaque part is the first 32 lowercase hex digits of the body's
SHA-256, so equal bodies always get equal tags and the tag never needs
storing beside the image.

#### EntityTag::of

```rust
pub fn of(body: &[u8]) -> Self
```

The tag for `body`.

##### Arguments

* `body` - The exact bytes the response carries.

#### EntityTag::opaque

```rust
pub fn opaque(&self) -> &str
```

The opaque part, without quotes.

#### EntityTag::header_value

```rust
pub fn header_value(&self) -> String
```

The `ETag` header value: the opaque part in double quotes.

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `PartialEq`

## Functions

### if_none_match

```rust
pub fn if_none_match(header: &[u8], tag: &EntityTag) -> bool
```

Whether an `If-None-Match` header matches `tag`, so the answer may be
`304 Not Modified`.

Follows RFC 9110 section 13.1.2: the header is `*` or a comma-separated
list of entity tags, each `"opaque"` or weak `W/"opaque"`, compared with
the weak comparison function (the `W/` prefix is ignored). Optional
whitespace may surround each element and empty elements are skipped. A
header that does not follow this grammar, or is longer than
[`MAX_IF_NONE_MATCH_BYTES`](#max_if_none_match_bytes), matches nothing.

#### Arguments

* `header` - The raw header value; several header lines joined with
  `,` read the same as one.
* `tag` - The current tag of the representation.

#### Returns

`true` when the client's copy is current.

## Constants

### MAX_IF_NONE_MATCH_BYTES

```rust
pub const MAX_IF_NONE_MATCH_BYTES: usize = _;
```

Longest `If-None-Match` header read, in bytes. A browser sends one tag;
a longer header is treated as no match rather than scanned.
