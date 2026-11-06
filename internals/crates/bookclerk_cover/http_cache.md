# bookclerk_cover::http_cache

HTTP caching for the images the daemon serves and the ones it fetches.

**Serving.** Every image the cover service answers with carries a strong
[`EntityTag`](#entitytag) derived from its bytes, so a browser that already holds a
copy can ask again with `If-None-Match` and get a `304 Not Modified`
instead of the body. [`if_none_match`](#if_none_match) reads that request header.

**Fetching.** When the covers plugin fetches an image, it passes the
upstream server's caching headers back to the host. [`upstream_policy`](#upstream_policy)
reads them into an [`UpstreamPolicy`]: whether the image may be stored
and for how long it stays fresh, clamped between [`FRESHNESS_FLOOR`](#freshness_floor) and
[`FRESHNESS_CEILING`](#freshness_ceiling), plus the [`Validators`](#validators) to send on the next
fetch so an unchanged image costs a `304`.

Both kinds of header come from outside (any client, any image host), so
they are untrusted input: the readers work on byte slices with no I/O,
never allocate in proportion to the input, and treat anything they
cannot read as absent, which only ever costs a full answer or an earlier
refresh. The `cache_headers` fuzz target drives them against an
independent oracle.

| Item | Kind | Summary |
| --- | --- | --- |
| [`EntityTag`](#entitytag) | struct | A strong entity tag naming one exact image body. |
| [`UpstreamHeaders`](#upstreamheaders) | struct | The caching headers of one upstream image response, as raw bytes; `None` for a header the response did not carry. |
| [`Validators`](#validators) | struct | Validators to send when fetching the image again, so an unchanged image costs a `304 Not Modified`. |
| [`UpstreamPolicy`](#upstreampolicy) | struct | What the host does with one fetched image, read from upstream's headers. |
| [`Storage`](#storage) | enum | Whether a fetched image may be kept, and for how long it is fresh. |
| [`if_none_match`](#if_none_match) | function | Whether an `If-None-Match` header matches `tag`, so the answer may be `304 Not Modified`. |
| [`upstream_policy`](#upstream_policy) | function | Reads upstream's caching headers into the host's policy for one image. |
| [`MAX_IF_NONE_MATCH_BYTES`](#max_if_none_match_bytes) | constant | Longest `If-None-Match` header read, in bytes. A browser sends one tag; a longer header is treated as no match rather than scanned. |
| [`FRESHNESS_FLOOR`](#freshness_floor) | constant | Shortest time a fetched image stays fresh, whatever upstream says: an image host asking for `no-cache` or `max-age=0` is asked again after an hour, not on every view. |
| [`FRESHNESS_DEFAULT`](#freshness_default) | constant | How long a fetched image stays fresh when upstream sends no `max-age`, `s-maxage` or `Expires`. |
| [`FRESHNESS_CEILING`](#freshness_ceiling) | constant | Longest time a fetched image stays fresh, whatever upstream says, so a changed profile picture or logo shows within a week. |
| [`MAX_RESPONSE_HEADER_BYTES`](#max_response_header_bytes) | constant | Longest upstream header value read, in bytes (the plugin ABI's `maxCoverHeaderBytes`); a longer value is treated as absent. |

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

### UpstreamHeaders

```rust
pub struct UpstreamHeaders<'a> {
    pub cache_control: Option<&'a [u8]>,
    pub expires: Option<&'a [u8]>,
    pub date: Option<&'a [u8]>,
    pub age: Option<&'a [u8]>,
    pub etag: Option<&'a [u8]>,
    pub last_modified: Option<&'a [u8]>,
}
```

The caching headers of one upstream image response, as raw bytes; `None`
for a header the response did not carry.

**Fields**

- `cache_control`: `Cache-Control`.
- `expires`: `Expires`.
- `date`: `Date`.
- `age`: `Age`.
- `etag`: `ETag`.
- `last_modified`: `Last-Modified`.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### Validators

```rust
pub struct Validators {
    /* private fields */
}
```

Validators to send when fetching the image again, so an unchanged image
costs a `304 Not Modified`.

#### Validators::new

```rust
pub fn new(etag: Option<&str>, last_modified: Option<SystemTime>) -> Self
```

Validators from stored values, as a cache index kept them.

##### Arguments

* `etag` - A stored entity tag; dropped unless it is one well-formed
  tag no longer than [`MAX_RESPONSE_HEADER_BYTES`](#max_response_header_bytes).
* `last_modified` - A stored modification time.

#### Validators::if_none_match

```rust
pub fn if_none_match(&self) -> Option<&str>
```

The `If-None-Match` value to send, when upstream sent an `ETag`.

#### Validators::last_modified

```rust
pub fn last_modified(&self) -> Option<SystemTime>
```

When upstream says the image last changed.

#### Validators::if_modified_since

```rust
pub fn if_modified_since(&self) -> Option<String>
```

The `If-Modified-Since` value to send, when upstream sent a
`Last-Modified`.

#### Validators::is_empty

```rust
pub fn is_empty(&self) -> bool
```

Whether there is anything to revalidate with.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### UpstreamPolicy

```rust
pub struct UpstreamPolicy {
    pub storage: Storage,
    pub validators: Validators,
}
```

What the host does with one fetched image, read from upstream's headers.

**Fields**

- `storage`: Whether to keep it and for how long it is fresh.
- `validators`: What to revalidate it with.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### Storage

```rust
pub enum Storage {
    DoNotStore,
    Store { fresh_for: Duration },
}
```

Whether a fetched image may be kept, and for how long it is fresh.

**Variants**

- `DoNotStore`: Upstream sent `no-store`: serve this answer, keep nothing.
- `Store`: Keep the image; it is fresh for this long from when it was received,
  always between [`FRESHNESS_FLOOR`](#freshness_floor) and [`FRESHNESS_CEILING`](#freshness_ceiling).
  - `fresh_for`: Freshness lifetime left at receipt.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

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

### upstream_policy

```rust
pub fn upstream_policy(
    headers: &UpstreamHeaders<'_>,
    received: SystemTime,
) -> UpstreamPolicy
```

Reads upstream's caching headers into the host's policy for one image.

Follows RFC 9111 for a shared cache, with Bookclerk's limits:

- `no-store` means [`Storage::DoNotStore`](#storage). `private` is ignored: the
  host keeps images for its own users, and a picture marked `private`
  is still the one the user asked for.
- The lifetime is the first `s-maxage`, else the first `max-age`, else
  `Expires` minus `Date` (or minus `received` when `Date` is absent or
  unreadable; an unreadable `Expires` means already stale), else
  [`FRESHNESS_DEFAULT`](#freshness_default). `no-cache` means a lifetime of zero.
- An explicit lifetime loses the response's age: the larger of `Age`
  and `received` minus `Date`.
- The result is clamped to [`FRESHNESS_FLOOR`](#freshness_floor) ..= [`FRESHNESS_CEILING`](#freshness_ceiling).

A header longer than [`MAX_RESPONSE_HEADER_BYTES`](#max_response_header_bytes) is read as absent.

#### Arguments

* `headers` - Upstream's caching headers.
* `received` - When the host received the response.

#### Returns

The storage decision and the validators.

## Constants

### MAX_IF_NONE_MATCH_BYTES

```rust
pub const MAX_IF_NONE_MATCH_BYTES: usize = _;
```

Longest `If-None-Match` header read, in bytes. A browser sends one tag;
a longer header is treated as no match rather than scanned.

### FRESHNESS_FLOOR

```rust
pub const FRESHNESS_FLOOR: Duration = _;
```

Shortest time a fetched image stays fresh, whatever upstream says: an
image host asking for `no-cache` or `max-age=0` is asked again after an
hour, not on every view.

### FRESHNESS_DEFAULT

```rust
pub const FRESHNESS_DEFAULT: Duration = _;
```

How long a fetched image stays fresh when upstream sends no `max-age`,
`s-maxage` or `Expires`.

### FRESHNESS_CEILING

```rust
pub const FRESHNESS_CEILING: Duration = _;
```

Longest time a fetched image stays fresh, whatever upstream says, so a
changed profile picture or logo shows within a week.

### MAX_RESPONSE_HEADER_BYTES

```rust
pub const MAX_RESPONSE_HEADER_BYTES: usize = 1024;
```

Longest upstream header value read, in bytes (the plugin ABI's
`maxCoverHeaderBytes`); a longer value is treated as absent.
