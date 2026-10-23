**bookclerk_workerd > granted_http**

# Module: granted_http

## Contents

**Structs**

- [`BodyReader`](#bodyreader) - Sans-IO reader for one request body under its declared framing.
- [`ChunkedDecoder`](#chunkeddecoder) - Incremental `Transfer-Encoding: chunked` decoder.
- [`ChunkedProgress`](#chunkedprogress) - Progress made by one `ChunkedDecoder::decode` call.
- [`RequestHead`](#requesthead) - Parsed request line and headers.

**Enums**

- [`BodyError`](#bodyerror) - Why a request body was rejected or cut short.
- [`BodyFraming`](#bodyframing) - How the request frames its body.
- [`ChunkedError`](#chunkederror) - Why a chunked body was rejected.
- [`FramingError`](#framingerror) - Why the body framing headers were rejected.
- [`HeadError`](#headerror) - Why a request head was rejected.
- [`HeadParse`](#headparse) - Outcome of [`parse_request_head`](granted_http.md#bookclerk_workerdgranted_httpparse_request_head) on the bytes buffered so far.

**Functions**

- [`body_framing`](#body_framing) - Decides the body framing from the headers.
- [`declared_content_length_over_cap`](#declared_content_length_over_cap) - True when a well-formed `Content-Length` declares more than `cap` bytes.
- [`header`](#header) - The first header named `name` (ASCII case-insensitive).
- [`parse_request_head`](#parse_request_head) - Parses an HTTP/1.1 request head from the start of `buf`.
- [`percent_decode`](#percent_decode) - Decodes `%HH` escapes and `+` (as space); malformed escapes are kept as
- [`query_param`](#query_param) - The percent-decoded value of the first `name=` pair in `query`.
- [`transfer_encoding_is_chunked`](#transfer_encoding_is_chunked) - True when a `Transfer-Encoding` header lists the `chunked` coding as a
- [`transfer_encodings`](#transfer_encodings) - Every transfer coding named by the `Transfer-Encoding` headers, in wire

**Constants**

- [`MAX_CHUNK_LINE_BYTES`](#max_chunk_line_bytes) - Longest chunk-size line (digits, extensions and CRLF) before the body is
- [`MAX_CHUNK_SIZE_DIGITS`](#max_chunk_size_digits) - Most hex digits a chunk-size field may carry. Sixteen digits already
- [`MAX_HEAD_BYTES`](#max_head_bytes) - Largest request head (request line plus headers plus the blank line)
- [`MAX_TRAILER_LINES`](#max_trailer_lines) - Most trailer lines accepted after the terminating chunk.

---

## bookclerk_workerd::granted_http::BodyError

*Enum*

Why a request body was rejected or cut short.

**Variants:**
- `Framing(FramingError)` - The framing headers were malformed or ambiguous.
- `Chunked(ChunkedError)` - The chunked framing was malformed.
- `TooLarge{ cap: usize }` - The body (declared or received) exceeds the route's byte cap.
- `Truncated` - The stream ended before the declared `Content-Length` or the

**Traits:** Eq, Copy, Error

**Trait Implementations:**

- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **From**
  - `fn from(err: ChunkedError) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(err: FramingError) -> Self`
- **PartialEq**
  - `fn eq(self: &Self, other: &BodyError) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> BodyError`



## bookclerk_workerd::granted_http::BodyFraming

*Enum*

How the request frames its body.

**Variants:**
- `UntilEof` - No `Content-Length` and no chunked encoding: the body runs to EOF.
- `Length(usize)` - `Content-Length` in bytes.
- `Chunked` - `Transfer-Encoding: chunked`.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &BodyFraming) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> BodyFraming`



## bookclerk_workerd::granted_http::BodyReader

*Struct*

Sans-IO reader for one request body under its declared framing.

[`crate::granted`](granted.md) feeds it whatever bytes arrive on the socket and sends
or collects what it yields; the `granted_http` fuzz target and the corpus
replay feed it the same bytes in arbitrary splits. Framing dispatch, the
route's byte cap and end-of-stream handling therefore run through one
code path for all three (`docs/adr/untrusted-input-fuzzing.md`).

A `Content-Length` body yields its first `len` bytes and ignores anything
after them (the connection closes after one request). A chunked body is
decoded incrementally; a read-until-EOF body yields every byte and is
done only at `Self::finish`.

**Methods:**

- `fn new(headers: &[(String, String)]) -> Result<Self, BodyError>` - Starts a reader for the body `headers` describe.
- `fn for_framing(framing: BodyFraming) -> Self` - Starts a reader for an already decided framing.
- `fn with_cap(self: Self, cap: usize) -> Self` - Caps the decoded body at `cap` bytes; a body that declares or
- `fn framing(self: &Self) -> BodyFraming` - The framing this reader follows.
- `fn is_done(self: &Self) -> bool` - True once the body is complete (a read-until-EOF body only after
- `fn push(self: & mut Self, input: &[u8], out: & mut Vec<u8>) -> Result<bool, BodyError>` - Feeds the next bytes from the stream, appending decoded body bytes to
- `fn finish(self: & mut Self) -> Result<(), BodyError>` - Reports end of stream.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BodyReader`
- **PartialEq**
  - `fn eq(self: &Self, other: &BodyReader) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_workerd::granted_http::ChunkedDecoder

*Struct*

Incremental `Transfer-Encoding: chunked` decoder.

Decoding is incremental so a chunk of any declared size streams through
without being buffered whole: the caller's buffer only ever holds what the
peer has sent, never what it promised. The decoder itself keeps a few
bytes of state.

**Methods:**

- `fn new() -> Self` - A decoder positioned before the first chunk-size line.
- `fn is_done(self: &Self) -> bool` - True once the terminating chunk has been consumed.
- `fn decode(self: & mut Self, input: &[u8], out: & mut Vec<u8>) -> Result<ChunkedProgress, ChunkedError>` - Decodes as much of `input` as is complete, appending chunk data to

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ChunkedDecoder) -> bool`
- **Default**
  - `fn default() -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> ChunkedDecoder`



## bookclerk_workerd::granted_http::ChunkedError

*Enum*

Why a chunked body was rejected.

**Variants:**
- `BadChunkSize` - A chunk-size field is empty, not hex, or longer than
- `ChunkLineTooLong` - A chunk-size line ran past [`MAX_CHUNK_LINE_BYTES`](granted_http.md#bookclerk_workerdgranted_httpmax_chunk_line_bytes) without CRLF.
- `MissingChunkTerminator` - Chunk data was not followed by CRLF.
- `BadTrailer` - More than [`MAX_TRAILER_LINES`](granted_http.md#bookclerk_workerdgranted_httpmax_trailer_lines) trailer lines, or a trailer line

**Traits:** Eq, Copy, Error

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ChunkedError) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ChunkedError`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`



## bookclerk_workerd::granted_http::ChunkedProgress

*Struct*

Progress made by one `ChunkedDecoder::decode` call.

**Fields:**
- `consumed: usize` - Input bytes consumed; the caller drops them from its buffer.
- `done: bool` - True once the terminating chunk and its trailers are consumed.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ChunkedProgress) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ChunkedProgress`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_workerd::granted_http::FramingError

*Enum*

Why the body framing headers were rejected.

**Variants:**
- `BadContentLength` - `Content-Length` is not a decimal `usize`, or two differ.
- `Ambiguous` - Both `Content-Length` and chunked encoding are present.
- `UnsupportedTransferEncoding` - `Transfer-Encoding` names a coding other than a single `chunked`

**Traits:** Eq, Copy, Error

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &FramingError) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> FramingError`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`



## bookclerk_workerd::granted_http::HeadError

*Enum*

Why a request head was rejected.

**Variants:**
- `TooLarge` - The head grew past [`MAX_HEAD_BYTES`](granted_http.md#bookclerk_workerdgranted_httpmax_head_bytes) without a blank line.
- `NotUtf8` - The head is not UTF-8.
- `BadRequestLine` - The request line is missing its method or target.
- `BadHeaderLine` - A header line has no `:`.

**Traits:** Eq, Error, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &HeadError) -> bool`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> HeadError`



## bookclerk_workerd::granted_http::HeadParse

*Enum*

Outcome of [`parse_request_head`](granted_http.md#bookclerk_workerdgranted_httpparse_request_head) on the bytes buffered so far.

**Variants:**
- `Incomplete` - No blank line yet; read more.
- `Complete{ head: RequestHead, consumed: usize }` - A complete head; `consumed` bytes belong to it and the rest of the

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &HeadParse) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> HeadParse`



## bookclerk_workerd::granted_http::MAX_CHUNK_LINE_BYTES

*Constant*: `usize`

Longest chunk-size line (digits, extensions and CRLF) before the body is
rejected as malformed.



## bookclerk_workerd::granted_http::MAX_CHUNK_SIZE_DIGITS

*Constant*: `usize`

Most hex digits a chunk-size field may carry. Sixteen digits already
overflow `u64`; real clients send far fewer.



## bookclerk_workerd::granted_http::MAX_HEAD_BYTES

*Constant*: `usize`

Largest request head (request line plus headers plus the blank line)
accepted before the connection is dropped.



## bookclerk_workerd::granted_http::MAX_TRAILER_LINES

*Constant*: `usize`

Most trailer lines accepted after the terminating chunk.



## bookclerk_workerd::granted_http::RequestHead

*Struct*

Parsed request line and headers.

**Fields:**
- `method: String` - HTTP method token as sent (case preserved).
- `path: String` - Request target as sent, including any query string.
- `headers: Vec<(String, String)>` - Header fields in wire order, names and values trimmed, names with

**Methods:**

- `fn split_query(self: &Self) -> (&str, &str)` - The path without its query string, and the query string (empty when

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &RequestHead) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> RequestHead`



## bookclerk_workerd::granted_http::body_framing

*Function*

Decides the body framing from the headers.

A `Content-Length` that is not a decimal number, or that disagrees with a
second `Content-Length`, is rejected rather than ignored, so a malformed
header can never turn a bounded body into a read-until-EOF one. A
`Transfer-Encoding` is accepted only when it names exactly one coding and
that coding is `chunked`; any other coding list is rejected because the
channel cannot know where such a body ends.

# Errors

Returns [`FramingError`](granted_http.md#bookclerk_workerdgranted_httpframingerror) on a malformed or ambiguous framing.

```rust
fn body_framing(headers: &[(String, String)]) -> Result<BodyFraming, FramingError>
```



## bookclerk_workerd::granted_http::declared_content_length_over_cap

*Function*

True when a well-formed `Content-Length` declares more than `cap` bytes.
A malformed one is left for [`body_framing`](granted_http.md#bookclerk_workerdgranted_httpbody_framing) to reject.

```rust
fn declared_content_length_over_cap(headers: &[(String, String)], cap: u32) -> bool
```



## bookclerk_workerd::granted_http::header

*Function*

The first header named `name` (ASCII case-insensitive).

```rust
fn header<'a>(headers: &'a [(String, String)], name: &str) -> Option<&'a str>
```



## bookclerk_workerd::granted_http::parse_request_head

*Function*

Parses an HTTP/1.1 request head from the start of `buf`.

# Errors

Returns [`HeadError`](granted_http.md#bookclerk_workerdgranted_httpheaderror) when the head is oversized, not UTF-8, or has a
malformed request or header line. An incomplete head is not an error
until it exceeds [`MAX_HEAD_BYTES`](granted_http.md#bookclerk_workerdgranted_httpmax_head_bytes).

```rust
fn parse_request_head(buf: &[u8]) -> Result<HeadParse, HeadError>
```



## bookclerk_workerd::granted_http::percent_decode

*Function*

Decodes `%HH` escapes and `+` (as space); malformed escapes are kept as
written, and the result is UTF-8 with replacement characters.

Works on bytes throughout: a `%` escape may end inside a multi-byte
character, so slicing the `str` would panic.

```rust
fn percent_decode(s: &str) -> String
```



## bookclerk_workerd::granted_http::query_param

*Function*

The percent-decoded value of the first `name=` pair in `query`.

```rust
fn query_param(query: &str, name: &str) -> Option<String>
```



## bookclerk_workerd::granted_http::transfer_encoding_is_chunked

*Function*

True when a `Transfer-Encoding` header lists the `chunked` coding as a
token (so `chunkedx` or `notchunked` do not count).

```rust
fn transfer_encoding_is_chunked(headers: &[(String, String)]) -> bool
```



## bookclerk_workerd::granted_http::transfer_encodings

*Function*

Every transfer coding named by the `Transfer-Encoding` headers, in wire
order, lowercased and trimmed, with empty list elements dropped.

```rust
fn transfer_encodings(headers: &[(String, String)]) -> Vec<String>
```



