# bookclerk_tags::wire

Frames the host and `bookclerk-media-worker tags` exchange over the
worker's stdio.

The worker holds a [`crate::TagReader`](README.md#tagreader) per open file and handles many
files at once; every frame names its file by an id the host picks. Each
frame is a four-byte big-endian length, then that many bytes of body,
which start with a one-byte tag:

| Direction | Frame | Body after the tag |
| --- | --- | --- |
| host → worker | `Open` (0) | `id: u64`, `size: u64` |
| host → worker | `Data` (1) | `id: u64`, `offset: u64`, then the span's bytes |
| host → worker | `Close` (2) | `id: u64` |
| worker → host | `Need` (0) | `id: u64`, `count: u16`, then `count` × (`offset: u64`, `len: u64`) |
| worker → host | `Done` (1) | `id: u64`, then the [`TagRecord`](README.md#tagrecord) as JSON |
| worker → host | `Failed` (2) | `id: u64`, then a UTF-8 message of at most [`MAX_ERROR_BYTES`](#max_error_bytes) |

All integers are big-endian. The worker reads untrusted files, so the
host treats it as compromised: [`decode_reply`](#decode_reply) checks every length and
count, and the host still checks each span against the file and each
record with [`TagRecord::validate`](README.md#tagrecordvalidate).

| Item | Kind | Summary |
| --- | --- | --- |
| [`WireError`](#wireerror) | enum | Why a frame body could not be read. |
| [`Request`](#request) | enum | A frame the host sends the worker. |
| [`Reply`](#reply) | enum | A frame the worker sends the host. |
| [`body_len`](#body_len) | function | Reads a frame's length prefix. |
| [`encode_request`](#encode_request) | function | Encodes a request as a whole frame, length prefix included. |
| [`decode_request`](#decode_request) | function | Decodes a request body (the bytes after the length prefix). |
| [`encode_reply`](#encode_reply) | function | Encodes a reply as a whole frame, length prefix included. |
| [`decode_reply`](#decode_reply) | function | Decodes a reply body (the bytes after the length prefix). |
| [`MAX_ERROR_BYTES`](#max_error_bytes) | constant | Longest error message a reply carries, in bytes. |
| [`MAX_REQUEST_BODY`](#max_request_body) | constant | Largest request body: a tag, an id, an offset and one span's bytes (at most `MAX_FETCH_BYTES`). |
| [`MAX_REPLY_BODY`](#max_reply_body) | constant | Largest reply body. A record at every limit is under 40 MiB of JSON. |

## Enums

### WireError

```rust
pub enum WireError {
    Truncated,
    Trailing,
    TooLarge,
    UnknownTag(u8),
    BadSpans,
    BadMessage,
    BadRecord(String),
}
```

Why a frame body could not be read.

**Variants**

- `Truncated`: The body ended before a field it promised.
- `Trailing`: The body has bytes after its last field.
- `TooLarge`: The body is over the frame's size cap.
- `UnknownTag`: The tag byte is not one this direction uses.
- `BadSpans`: A `Need` with no spans or more than [`MAX_SPANS_PER_ROUND`](README.md#max_spans_per_round), or an
  empty or overflowing span.
- `BadMessage`: An error message that is not UTF-8 or is too long.
- `BadRecord`: A `Done` whose record is not valid JSON for a [`TagRecord`](README.md#tagrecord).

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### Request

```rust
pub enum Request {
    Open { id: u64, size: u64 },
    Data { id: u64, offset: u64, bytes: Vec<u8> },
    Close { id: u64 },
}
```

A frame the host sends the worker.

**Variants**

- `Open`: Start reading a file of `size` bytes as `id`.
  - `id`: The file's id; unique among open files.
  - `size`: The file's size in bytes.
- `Data`: The bytes of one span the worker asked for.
  - `id`: The file's id.
  - `offset`: The span's offset.
  - `bytes`: The span's bytes.
- `Close`: Forget a file (the host gave up on it).
  - `id`: The file's id.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Reply

```rust
pub enum Reply {
    Need { id: u64, spans: Vec<ByteSpan> },
    Done { id: u64, record: Box<TagRecord> },
    Failed { id: u64, message: String },
}
```

A frame the worker sends the host.

**Variants**

- `Need`: Send these spans of the file.
  - `id`: The file's id.
  - `spans`: One to [`MAX_SPANS_PER_ROUND`](README.md#max_spans_per_round) non-empty spans.
- `Done`: The file's record; the worker has forgotten the file.
  - `id`: The file's id.
  - `record`: What the file holds.
- `Failed`: The file could not be read; the worker has forgotten it.
  - `id`: The file's id.
  - `message`: Why, for logs; at most [`MAX_ERROR_BYTES`](#max_error_bytes).

#### Reply::id

```rust
pub fn id(&self) -> u64
```

The id of the file this reply is about.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### body_len

```rust
pub fn body_len(prefix: [u8; 4]) -> usize
```

Reads a frame's length prefix.

#### Returns

The body length in bytes.

### encode_request

```rust
pub fn encode_request(request: &Request) -> Result<Vec<u8>, WireError>
```

Encodes a request as a whole frame, length prefix included.

#### Errors

[`WireError::TooLarge`](#wireerror) when the body is over [`MAX_REQUEST_BODY`](#max_request_body).

### decode_request

```rust
pub fn decode_request(body: &[u8]) -> Result<Request, WireError>
```

Decodes a request body (the bytes after the length prefix).

#### Errors

A [`WireError`](#wireerror) for a truncated, oversized or trailing body or an
unknown tag.

### encode_reply

```rust
pub fn encode_reply(reply: &Reply) -> Result<Vec<u8>, WireError>
```

Encodes a reply as a whole frame, length prefix included.

An error message is cut to [`MAX_ERROR_BYTES`](#max_error_bytes) at a character boundary.

#### Errors

[`WireError::BadSpans`](#wireerror) for a span list the host would refuse,
[`WireError::BadRecord`](#wireerror) when the record cannot be serialized, and
[`WireError::TooLarge`](#wireerror) over [`MAX_REPLY_BODY`](#max_reply_body).

### decode_reply

```rust
pub fn decode_reply(body: &[u8]) -> Result<Reply, WireError>
```

Decodes a reply body (the bytes after the length prefix).

#### Errors

A [`WireError`](#wireerror) for a truncated, oversized or trailing body, an unknown
tag, a bad span list, a bad message or a record that does not parse.

## Constants

### MAX_ERROR_BYTES

```rust
pub const MAX_ERROR_BYTES: usize = 1024;
```

Longest error message a reply carries, in bytes.

### MAX_REQUEST_BODY

```rust
pub const MAX_REQUEST_BODY: usize = _;
```

Largest request body: a tag, an id, an offset and one span's bytes
(at most [`MAX_FETCH_BYTES`](README.md#max_fetch_bytes)).

### MAX_REPLY_BODY

```rust
pub const MAX_REPLY_BODY: usize = _;
```

Largest reply body. A record at every limit is under 40 MiB of JSON.
