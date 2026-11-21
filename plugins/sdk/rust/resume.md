# bookclerk_plugin_sdk::resume

Sans-IO decisions for resuming an interrupted HTTP download.

Audience: guests that download large files (store audio, archives).
`crate::download` (feature `http`) drives these functions over the
SDK's HTTP client; they are separate and IO-free so the fuzz target
`download_resume` and the unit tests exercise exactly the code a guest
runs.

A download keeps the bytes it already has and asks the server for the
rest with `Range: bytes=<offset>-`. The server's answer is untrusted:
[`parse_content_range`](#parse_content_range) reads its `Content-Range` header and
[`next_step`](#next_step) decides whether the body continues the partial file,
replaces it, finishes it, or ends the download. A body is only ever
appended where it provably starts at the partial's end, so a server
that answers a different range, changes the file's size, or ignores
the range never splices unrelated bytes into the file.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Partial`](#partial) | struct | What a download already has when it reads a response. |
| [`ResponseHead`](#responsehead) | struct | The parts of a response's head `next_step` reads. |
| [`ContentRange`](#contentrange) | enum | A `Content-Range` header the server sent, as `parse_content_range` reads it. |
| [`Step`](#step) | enum | What to do with a response, as `next_step` decides it. |
| [`Validator`](#validator) | enum | A validator that identifies one version of the remote file, sent as `If-Range` so a server whose file changed answers the whole new file instead of a range of it. |
| [`parse_content_range`](#parse_content_range) | function | Parses a `Content-Range` value (RFC 9110 §14.4) for the `bytes` unit. |
| [`next_step`](#next_step) | function | Decides what a response means for a partial download. |
| [`MAX_CONTENT_RANGE_BYTES`](#max_content_range_bytes) | constant | Longest `Content-Range` value read, in bytes. A valid one for a file under 2^64 bytes is at most `bytes ` plus three 20-digit numbers. |
| [`MAX_VALIDATOR_BYTES`](#max_validator_bytes) | constant | Longest validator (`ETag` or `Last-Modified` value) kept for `If-Range`, in bytes. Real ones are well under this; a longer value is not used. |

## Structs

### Partial

```rust
pub struct Partial {
    pub offset: u64,
    pub total: Option<u64>,
}
```

What a download already has when it reads a response.

**Fields**

- `offset`: Bytes already written, from the start of the file; the request
  asked for `Range: bytes=<offset>-` when this is above zero.
- `total`: The file's length as an earlier response stated it, if any; a
  response stating another length means the file changed.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ResponseHead

```rust
pub struct ResponseHead<'a> {
    pub status: u16,
    pub content_range: Option<&'a [u8]>,
    pub content_length: Option<u64>,
}
```

The parts of a response's head [`next_step`](#next_step) reads.

**Fields**

- `status`: HTTP status code.
- `content_range`: The raw `Content-Range` value, when the header was sent.
- `content_length`: `Content-Length`, when the header was sent and is a number.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Enums

### ContentRange

```rust
pub enum ContentRange {
    Bytes { first: u64, last: u64, total: Option<u64> },
    Unsatisfied { total: u64 },
}
```

A `Content-Range` header the server sent, as [`parse_content_range`](#parse_content_range)
reads it.

**Variants**

- `Bytes`: `bytes <first>-<last>/<total>`: the body holds bytes `first` to
  `last` (inclusive) of a file `total` bytes long, or of a file whose
  length the server did not state (`/*`).
  - `first`: Offset of the body's first byte in the file.
  - `last`: Offset of the body's last byte in the file; at least `first`.
  - `total`: The whole file's length in bytes, greater than `last`; `None`
    for `/*`.
- `Unsatisfied`: `bytes */<total>`: the range asked for is past the end of a file
  `total` bytes long (the form a `416` carries).
  - `total`: The whole file's length in bytes.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Step

```rust
pub enum Step {
    Append { total: Option<u64>, end: Option<u64> },
    Replace { total: Option<u64>, end: Option<u64> },
    Complete,
    StartOver,
    Retry,
    Fail,
}
```

What to do with a response, as [`next_step`](#next_step) decides it.

**Variants**

- `Append`: The body continues the partial at its current end (`206` starting
  exactly at the offset): append it.
  - `total`: The file's length, when the server stated it.
  - `end`: Offset just past the body's last byte, for a `206` (its
    range's last byte plus one): a body that runs past it does not
    match its own range, and one that stops there leaves the rest
    of the file to ask for. `None` for a `200`, whose body runs to
    the end of the file.
- `Replace`: The body is the whole file from byte 0 (`200`, or a `206` from 0
  asked without a range): drop what the partial holds and write the
  body in its place.
  - `total`: The file's length, when the server stated it.
  - `end`: Offset just past the body's last byte, for a `206` (its
    range's last byte plus one): a body that runs past it does not
    match its own range, and one that stops there leaves the rest
    of the file to ask for. `None` for a `200`, whose body runs to
    the end of the file.
- `Complete`: The partial already holds the whole file (`416` stating a length
  equal to the offset): nothing more to read.
- `StartOver`: The answer does not fit the partial (another range, a changed
  length, an unsatisfiable range at another length): drop the
  partial and ask again for the whole file.
- `Retry`: A transient refusal (`408`, `429`, `5xx`): wait and ask again for
  the same range.
- `Fail`: Any other status: the download fails with it.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Validator

```rust
pub enum Validator {
    ETag(String),
    LastModified(String),
}
```

A validator that identifies one version of the remote file, sent as
`If-Range` so a server whose file changed answers the whole new file
instead of a range of it.

**Variants**

- `ETag`: A strong `ETag` (never one starting with `W/`, which `If-Range`
  may not carry).
- `LastModified`: A `Last-Modified` date, as sent.

#### Validator::from_headers

```rust
pub fn from_headers(etag: Option<&[u8]>, last_modified: Option<&[u8]>) -> Option<Self>
```

Picks the validator to resume with from a response's headers.

A strong `ETag` wins over `Last-Modified`. A value is used only
when it is visible ASCII (and spaces, for a date) of at most
[`MAX_VALIDATOR_BYTES`](#max_validator_bytes) bytes, so it can be sent back unchanged.

##### Arguments

* `etag` - The raw `ETag` value, when sent.
* `last_modified` - The raw `Last-Modified` value, when sent.

##### Returns

The validator, or `None` when neither header is usable.

#### Validator::as_str

```rust
pub fn as_str(&self) -> &str
```

The value to send as `If-Range`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### parse_content_range

```rust
pub fn parse_content_range(value: &[u8]) -> Option<ContentRange>
```

Parses a `Content-Range` value (RFC 9110 §14.4) for the `bytes` unit.

Accepts exactly `bytes <first>-<last>/<total|*>` or `bytes */<total>`,
with one space after the unit and decimal numbers only. Optional
whitespace around the value is ignored; anything else (another unit,
signs, a range whose last byte is before its first or at or past the
stated length, numbers that overflow 64 bits, a value longer than
[`MAX_CONTENT_RANGE_BYTES`](#max_content_range_bytes)) is refused.

#### Arguments

* `value` - The header value as received, not necessarily UTF-8.

#### Returns

The parsed range, or `None` when the value is not a valid `bytes`
content range.

### next_step

```rust
pub fn next_step(partial: Partial, head: ResponseHead<'_>) -> Step
```

Decides what a response means for a partial download.

Only a `206` whose `Content-Range` starts exactly at `partial.offset`
and states the same length as `partial.total` (when both are known),
with a `Content-Length` (when sent) matching the range, is appended. A
`200` replaces the partial. A `416` finishes the download only when it
states `bytes */<offset>` for a non-empty partial and agrees with the
known length. Everything else that names a range the partial cannot
use starts over; `408`, `429` and `5xx` are retried; any other status
fails.

#### Arguments

* `partial` - What the download held when it sent the request.
* `head` - The response's status and range headers.

#### Returns

The [`Step`](#step) to take.

## Constants

### MAX_CONTENT_RANGE_BYTES

```rust
pub const MAX_CONTENT_RANGE_BYTES: usize = 80;
```

Longest `Content-Range` value read, in bytes. A valid one for a file
under 2^64 bytes is at most `bytes ` plus three 20-digit numbers.

### MAX_VALIDATOR_BYTES

```rust
pub const MAX_VALIDATOR_BYTES: usize = 256;
```

Longest validator (`ETag` or `Last-Modified` value) kept for `If-Range`,
in bytes. Real ones are well under this; a longer value is not used.
