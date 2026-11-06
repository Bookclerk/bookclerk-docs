# bookclerk_cover::wire

Frames the host and the cover worker exchange over the worker's stdio.

The worker is long-lived and handles many covers at once, so each request
carries an id and replies may come back in any order. Every frame is a
four-byte big-endian length followed by that many bytes of body:

| Frame | Body |
| --- | --- |
| Request | `id: u64`, then the cover bytes (at most [`MAX_INPUT_BYTES`](README.md#max_input_bytes)) |
| Reply, ok | `id: u64`, `0u8`, `width: u32`, `height: u32`, then for each [`CoverSize`](README.md#coversize) in order a `u32` length and that many JPEG bytes |
| Reply, error | `id: u64`, `1u8`, then a UTF-8 message of at most [`MAX_ERROR_BYTES`](#max_error_bytes) |

All integers are big-endian. The host reads replies from a process that
decoded untrusted bytes, so [`decode_reply`](#decode_reply) checks every length, and
accepts an ok reply only when each image starts with the JPEG signature.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Request`](#request) | struct | One cover the host asks the worker to process. |
| [`WireError`](#wireerror) | enum | Why a frame body could not be read. |
| [`Reply`](#reply) | enum | The worker's answer to one `Request`. |
| [`body_len`](#body_len) | function | Reads a frame's length prefix. |
| [`encode_request`](#encode_request) | function | Encodes a request as a whole frame, length prefix included. |
| [`decode_request`](#decode_request) | function | Decodes a request body (the bytes after the length prefix). |
| [`encode_reply`](#encode_reply) | function | Encodes a reply as a whole frame, length prefix included. |
| [`decode_reply`](#decode_reply) | function | Decodes a reply body (the bytes after the length prefix). |
| [`MAX_ERROR_BYTES`](#max_error_bytes) | constant | Longest error message a reply carries, in bytes. |
| [`MAX_REQUEST_BODY`](#max_request_body) | constant | Largest request body: an id and a cover. |
| [`MAX_REPLY_BODY`](#max_reply_body) | constant | Largest reply body. A JPEG of a cover that fits the decode limits is far smaller than this; a longer frame means the worker is broken. |

## Structs

### Request

```rust
pub struct Request {
    pub id: u64,
    pub cover: Vec<u8>,
}
```

One cover the host asks the worker to process.

**Fields**

- `id`: Id the reply echoes.
- `cover`: The untrusted cover bytes.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### WireError

```rust
pub enum WireError {
    Truncated,
    Trailing,
    TooLarge,
    UnknownTag(u8),
    BadMessage,
    NotJpeg,
}
```

Why a frame body could not be read.

**Variants**

- `Truncated`: The body ended before a field it promised.
- `Trailing`: The body has bytes after its last field.
- `TooLarge`: The body is over the frame's size cap.
- `UnknownTag`: The reply tag is neither ok nor error.
- `BadMessage`: An error message is not UTF-8 or is too long.
- `NotJpeg`: An ok reply carries an image that is not a JPEG, or zero dimensions.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### Reply

```rust
pub enum Reply {
    Ok { id: u64, set: CoverSet },
    Err { id: u64, message: String },
}
```

The worker's answer to one [`Request`](#request).

**Variants**

- `Ok`: The cover was processed.
  - `id`: The request's id.
  - `set`: The JPEGs.
- `Err`: The cover was refused.
  - `id`: The request's id.
  - `message`: Why, for logs; at most [`MAX_ERROR_BYTES`](#max_error_bytes).

#### Reply::id

```rust
pub fn id(&self) -> u64
```

The id of the request this answers.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### body_len

```rust
pub fn body_len(prefix: [u8; 4]) -> usize
```

Reads a frame's length prefix.

#### Arguments

* `prefix` - The four bytes before the body.

#### Returns

The body length in bytes.

### encode_request

```rust
pub fn encode_request(request: &Request) -> Result<Vec<u8>, WireError>
```

Encodes a request as a whole frame, length prefix included.

#### Errors

[`WireError::TooLarge`](#wireerror) for a cover over [`MAX_INPUT_BYTES`](README.md#max_input_bytes).

### decode_request

```rust
pub fn decode_request(body: &[u8]) -> Result<Request, WireError>
```

Decodes a request body (the bytes after the length prefix).

#### Errors

[`WireError::Truncated`](#wireerror) without a full id, [`WireError::TooLarge`](#wireerror) over
[`MAX_REQUEST_BODY`](#max_request_body).

### encode_reply

```rust
pub fn encode_reply(reply: &Reply) -> Result<Vec<u8>, WireError>
```

Encodes a reply as a whole frame, length prefix included.

An error message is cut to [`MAX_ERROR_BYTES`](#max_error_bytes) at a character boundary.

#### Errors

[`WireError::TooLarge`](#wireerror) when an image does not fit a `u32` length or the
body is over [`MAX_REPLY_BODY`](#max_reply_body).

### decode_reply

```rust
pub fn decode_reply(body: &[u8]) -> Result<Reply, WireError>
```

Decodes a reply body (the bytes after the length prefix).

#### Errors

A [`WireError`](#wireerror) for a truncated, oversized or trailing body, an unknown
tag, a bad message, zero dimensions, or an image without the JPEG
signature.

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

Largest request body: an id and a cover.

### MAX_REPLY_BODY

```rust
pub const MAX_REPLY_BODY: usize = _;
```

Largest reply body. A JPEG of a cover that fits the decode limits is far
smaller than this; a longer frame means the worker is broken.
