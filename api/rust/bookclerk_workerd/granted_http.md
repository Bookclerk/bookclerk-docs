# bookclerk_workerd::granted_http

Sans-IO HTTP/1.1 parsing for the granted reverse channel.

Every byte on the granted channel comes from a plugin isolate, so the
request head, the chunked body framing and the query string are parsed
here over plain byte slices, with no socket in sight. [`crate::granted`](granted.md)
feeds these from its reader; unit tests and the `granted_http` fuzz
target feed them directly, so all three drive the same code
(`docs/adr/untrusted-input-fuzzing.md`).

Limits enforced here are structural (head size, chunk-size digits,
trailer count). Body byte caps belong to the route and stay in
[`crate::granted`](granted.md).

Parsing is strict rather than lenient: the only client is the plugin
runtime's own `fetch`, so anything RFC 9112 does not require accepting
(bare LF, whitespace before a chunk size, a second transfer coding) is
rejected and answered with `400 Bad Request` instead of being guessed at.

| Item | Kind | Summary |
| --- | --- | --- |
| [`RequestHead`](#requesthead) | struct | Parsed request line and headers. |
| [`ChunkedProgress`](#chunkedprogress) | struct | Progress made by one `ChunkedDecoder::decode` call. |
| [`ChunkedDecoder`](#chunkeddecoder) | struct | Incremental `Transfer-Encoding: chunked` decoder. |
| [`BodyReader`](#bodyreader) | struct | Sans-IO reader for one request body under its declared framing. |
| [`HeadError`](#headerror) | enum | Why a request head was rejected. |
| [`HeadParse`](#headparse) | enum | Outcome of `parse_request_head` on the bytes buffered so far. |
| [`BodyFraming`](#bodyframing) | enum | How the request frames its body. |
| [`FramingError`](#framingerror) | enum | Why the body framing headers were rejected. |
| [`ChunkedError`](#chunkederror) | enum | Why a chunked body was rejected. |
| [`BodyError`](#bodyerror) | enum | Why a request body was rejected or cut short. |
| [`parse_request_head`](#parse_request_head) | function | Parses an HTTP/1.1 request head from the start of `buf`. |
| [`header`](#header) | function | The first header named `name` (ASCII case-insensitive). |
| [`transfer_encodings`](#transfer_encodings) | function | Every transfer coding named by the `Transfer-Encoding` headers, in wire order, lowercased and trimmed, with empty list elements dropped. |
| [`transfer_encoding_is_chunked`](#transfer_encoding_is_chunked) | function | True when a `Transfer-Encoding` header lists the `chunked` coding as a token (so `chunkedx` or `notchunked` do not count). |
| [`body_framing`](#body_framing) | function | Decides the body framing from the headers. |
| [`declared_content_length_over_cap`](#declared_content_length_over_cap) | function | True when a well-formed `Content-Length` declares more than `cap` bytes. A malformed one is left for `body_framing` to reject. |
| [`query_param`](#query_param) | function | The percent-decoded value of the first `name=` pair in `query`. |
| [`percent_decode`](#percent_decode) | function | Decodes `%HH` escapes and `+` (as space); malformed escapes are kept as written, and the result is UTF-8 with replacement characters. |
| [`MAX_HEAD_BYTES`](#max_head_bytes) | constant | Largest request head (request line plus headers plus the blank line) accepted before the connection is dropped. |
| [`MAX_CHUNK_SIZE_DIGITS`](#max_chunk_size_digits) | constant | Most hex digits a chunk-size field may carry. Sixteen digits already overflow `u64`; real clients send far fewer. |
| [`MAX_CHUNK_LINE_BYTES`](#max_chunk_line_bytes) | constant | Longest chunk-size line (digits, extensions and CRLF) before the body is rejected as malformed. |
| [`MAX_TRAILER_LINES`](#max_trailer_lines) | constant | Most trailer lines accepted after the terminating chunk. |

## Structs

### RequestHead

```rust
pub struct RequestHead {
    pub method: String,
    pub path: String,
    pub headers: Vec<(String, String)>,
}
```

Parsed request line and headers.

**Fields**

- `method`: HTTP method token as sent (case preserved).
- `path`: Request target as sent, including any query string.
- `headers`: Header fields in wire order, names and values trimmed, names with
  their original case.

#### RequestHead::split_query

```rust
pub fn split_query(&self) -> (&str, &str)
```

The path without its query string, and the query string (empty when
absent).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ChunkedProgress

```rust
pub struct ChunkedProgress {
    pub consumed: usize,
    pub done: bool,
}
```

Progress made by one [`ChunkedDecoder::decode`](#chunkeddecoderdecode) call.

**Fields**

- `consumed`: Input bytes consumed; the caller drops them from its buffer.
- `done`: True once the terminating chunk and its trailers are consumed.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ChunkedDecoder

```rust
pub struct ChunkedDecoder {
    /* private fields */
}
```

Incremental `Transfer-Encoding: chunked` decoder.

Decoding is incremental so a chunk of any declared size streams through
without being buffered whole: the caller's buffer only ever holds what the
peer has sent, never what it promised. The decoder itself keeps a few
bytes of state.

#### ChunkedDecoder::new

```rust
pub fn new() -> Self
```

A decoder positioned before the first chunk-size line.

#### ChunkedDecoder::is_done

```rust
pub fn is_done(&self) -> bool
```

True once the terminating chunk has been consumed.

#### ChunkedDecoder::decode

```rust
pub fn decode(
    &mut self,
    input: &[u8],
    out: &mut Vec<u8>,
) -> Result<ChunkedProgress, ChunkedError>
```

Decodes as much of `input` as is complete, appending chunk data to
`out`.

Stops at the first incomplete element (a size line without its CRLF,
data the peer has not sent yet) and reports how many bytes were
consumed; the caller keeps the rest and calls again with more input.

##### Errors

Returns [`ChunkedError`](#chunkederror) on malformed framing; the decoder is then
unusable and the connection should be dropped.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### BodyReader

```rust
pub struct BodyReader {
    /* private fields */
}
```

Sans-IO reader for one request body under its declared framing.

[`crate::granted`](granted.md) feeds it whatever bytes arrive on the socket and sends
or collects what it yields; the `granted_http` fuzz target and the corpus
replay feed it the same bytes in arbitrary splits. Framing dispatch, the
route's byte cap and end-of-stream handling therefore run through one
code path for all three (`docs/adr/untrusted-input-fuzzing.md`).

A `Content-Length` body yields its first `len` bytes and ignores anything
after them (the connection closes after one request). A chunked body is
decoded incrementally; a read-until-EOF body yields every byte and is
done only at [`Self::finish`](#bodyreaderfinish).

#### BodyReader::new

```rust
pub fn new(headers: &[(String, String)]) -> Result<Self, BodyError>
```

Starts a reader for the body `headers` describe.

##### Errors

Returns [`BodyError::Framing`](#bodyerror) when the framing headers are malformed
or ambiguous.

#### BodyReader::for_framing

```rust
pub fn for_framing(framing: BodyFraming) -> Self
```

Starts a reader for an already decided framing.

#### BodyReader::with_cap

```rust
pub fn with_cap(self, cap: usize) -> Self
```

Caps the decoded body at `cap` bytes; a body that declares or
delivers more fails with [`BodyError::TooLarge`](#bodyerror).

#### BodyReader::framing

```rust
pub fn framing(&self) -> BodyFraming
```

The framing this reader follows.

#### BodyReader::is_done

```rust
pub fn is_done(&self) -> bool
```

True once the body is complete (a read-until-EOF body only after
[`Self::finish`](#bodyreaderfinish)).

#### BodyReader::push

```rust
pub fn push(&mut self, input: &[u8], out: &mut Vec<u8>) -> Result<bool, BodyError>
```

Feeds the next bytes from the stream, appending decoded body bytes to
`out`.

##### Returns

`true` once the body is complete; later input is ignored.

##### Errors

[`BodyError::TooLarge`](#bodyerror) when the declared or received body passes the
cap, or [`BodyError::Chunked`](#bodyerror) on malformed chunked framing. Either
error is final.

#### BodyReader::finish

```rust
pub fn finish(&mut self) -> Result<(), BodyError>
```

Reports end of stream.

##### Errors

[`BodyError::Truncated`](#bodyerror) when a `Content-Length` or chunked body is
incomplete. A read-until-EOF body completes here.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### HeadError

```rust
pub enum HeadError {
    TooLarge,
    NotUtf8,
    BadRequestLine,
    BadHeaderLine,
}
```

Why a request head was rejected.

**Variants**

- `TooLarge`: The head grew past [`MAX_HEAD_BYTES`](#max_head_bytes) without a blank line.
- `NotUtf8`: The head is not UTF-8.
- `BadRequestLine`: The request line is missing its method or target.
- `BadHeaderLine`: A header line has no `:`.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### HeadParse

```rust
pub enum HeadParse {
    Incomplete,
    Complete { head: RequestHead, consumed: usize },
}
```

Outcome of [`parse_request_head`](#parse_request_head) on the bytes buffered so far.

**Variants**

- `Incomplete`: No blank line yet; read more.
- `Complete`: A complete head; `consumed` bytes belong to it and the rest of the
  buffer is the start of the body.
  - `head`: The parsed head.
  - `consumed`: Bytes of `buf` the head occupied, including the blank line.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### BodyFraming

```rust
pub enum BodyFraming {
    UntilEof,
    Length(usize),
    Chunked,
}
```

How the request frames its body.

**Variants**

- `UntilEof`: No `Content-Length` and no chunked encoding: the body runs to EOF.
- `Length`: `Content-Length` in bytes.
- `Chunked`: `Transfer-Encoding: chunked`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### FramingError

```rust
pub enum FramingError {
    BadContentLength,
    Ambiguous,
    UnsupportedTransferEncoding,
}
```

Why the body framing headers were rejected.

**Variants**

- `BadContentLength`: `Content-Length` is not a decimal `usize`, or two differ.
- `Ambiguous`: Both `Content-Length` and chunked encoding are present.
- `UnsupportedTransferEncoding`: `Transfer-Encoding` names a coding other than a single `chunked`
  (`gzip, chunked`, `identity`, `chunked, chunked`); the channel does
  not decode those, so the body length would be unknown.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `From<FramingError>`, `PartialEq`

### ChunkedError

```rust
pub enum ChunkedError {
    BadChunkSize,
    ChunkLineTooLong,
    MissingChunkTerminator,
    BadTrailer,
}
```

Why a chunked body was rejected.

**Variants**

- `BadChunkSize`: A chunk-size field is empty, not hex, or longer than
  [`MAX_CHUNK_SIZE_DIGITS`](#max_chunk_size_digits).
- `ChunkLineTooLong`: A chunk-size line ran past [`MAX_CHUNK_LINE_BYTES`](#max_chunk_line_bytes) without CRLF.
- `MissingChunkTerminator`: Chunk data was not followed by CRLF.
- `BadTrailer`: More than [`MAX_TRAILER_LINES`](#max_trailer_lines) trailer lines, or a trailer line
  longer than [`MAX_CHUNK_LINE_BYTES`](#max_chunk_line_bytes).

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `From<ChunkedError>`, `PartialEq`

### BodyError

```rust
pub enum BodyError {
    Framing(FramingError),
    Chunked(ChunkedError),
    TooLarge { cap: usize },
    Truncated,
}
```

Why a request body was rejected or cut short.

**Variants**

- `Framing`: The framing headers were malformed or ambiguous.
- `Chunked`: The chunked framing was malformed.
- `TooLarge`: The body (declared or received) exceeds the route's byte cap.
  - `cap`: The cap the body ran past, in bytes.
- `Truncated`: The stream ended before the declared `Content-Length` or the
  terminating chunk.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `From<ChunkedError>`, `From<FramingError>`, `PartialEq`

## Functions

### parse_request_head

```rust
pub fn parse_request_head(buf: &[u8]) -> Result<HeadParse, HeadError>
```

Parses an HTTP/1.1 request head from the start of `buf`.

#### Errors

Returns [`HeadError`](#headerror) when the head is oversized, not UTF-8, or has a
malformed request or header line. An incomplete head is not an error
until it exceeds [`MAX_HEAD_BYTES`](#max_head_bytes).

### header

```rust
pub fn header<'a>(headers: &'a [(String, String)], name: &str) -> Option<&'a str>
```

The first header named `name` (ASCII case-insensitive).

### transfer_encodings

```rust
pub fn transfer_encodings(headers: &[(String, String)]) -> Vec<String>
```

Every transfer coding named by the `Transfer-Encoding` headers, in wire
order, lowercased and trimmed, with empty list elements dropped.

### transfer_encoding_is_chunked

```rust
pub fn transfer_encoding_is_chunked(headers: &[(String, String)]) -> bool
```

True when a `Transfer-Encoding` header lists the `chunked` coding as a
token (so `chunkedx` or `notchunked` do not count).

### body_framing

```rust
pub fn body_framing(headers: &[(String, String)]) -> Result<BodyFraming, FramingError>
```

Decides the body framing from the headers.

A `Content-Length` that is not a decimal number, or that disagrees with a
second `Content-Length`, is rejected rather than ignored, so a malformed
header can never turn a bounded body into a read-until-EOF one. A
`Transfer-Encoding` is accepted only when it names exactly one coding and
that coding is `chunked`; any other coding list is rejected because the
channel cannot know where such a body ends.

#### Errors

Returns [`FramingError`](#framingerror) on a malformed or ambiguous framing.

### declared_content_length_over_cap

```rust
pub fn declared_content_length_over_cap(headers: &[(String, String)], cap: u32) -> bool
```

True when a well-formed `Content-Length` declares more than `cap` bytes.
A malformed one is left for [`body_framing`](#body_framing) to reject.

### query_param

```rust
pub fn query_param(query: &str, name: &str) -> Option<String>
```

The percent-decoded value of the first `name=` pair in `query`.

### percent_decode

```rust
pub fn percent_decode(s: &str) -> String
```

Decodes `%HH` escapes and `+` (as space); malformed escapes are kept as
written, and the result is UTF-8 with replacement characters.

Works on bytes throughout: a `%` escape may end inside a multi-byte
character, so slicing the `str` would panic.

## Constants

### MAX_HEAD_BYTES

```rust
pub const MAX_HEAD_BYTES: usize = _;
```

Largest request head (request line plus headers plus the blank line)
accepted before the connection is dropped.

### MAX_CHUNK_SIZE_DIGITS

```rust
pub const MAX_CHUNK_SIZE_DIGITS: usize = 16;
```

Most hex digits a chunk-size field may carry. Sixteen digits already
overflow `u64`; real clients send far fewer.

### MAX_CHUNK_LINE_BYTES

```rust
pub const MAX_CHUNK_LINE_BYTES: usize = 1024;
```

Longest chunk-size line (digits, extensions and CRLF) before the body is
rejected as malformed.

### MAX_TRAILER_LINES

```rust
pub const MAX_TRAILER_LINES: usize = 16;
```

Most trailer lines accepted after the terminating chunk.
