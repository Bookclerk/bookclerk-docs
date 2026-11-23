# bookclerk_media::pcm_wire

Frames the host and `bookclerk-media-worker pcm` exchange over the
worker's stdio.

One worker process decodes one audio file, from a start position to its
end, for a player; or cuts a short sample of it, re-encoded as MP3, for a
book's details. The worker opens nothing: it asks the host for the byte
ranges its demuxer reads and answers with decoded PCM, or with the
sample's MP3 bytes. Each frame is a four-byte big-endian length, then
that many bytes of body, which start with a one-byte tag:

| Direction | Frame | Body after the tag |
| --- | --- | --- |
| host → worker | `Open` (0) | `size: u64`, `start_ms: u64` |
| host → worker | `Data` (1) | `offset: u64`, then the span's bytes |
| host → worker | `Sample` (2) | `size: u64`, `start_ms: u64`, `length_ms: u64` |
| worker → host | `Need` (0) | `offset: u64`, `len: u32` |
| worker → host | `Format` (1) | `sample_rate: u32`, `channels: u8`, `start_ms: u64`, `duration_ms: u64` |
| worker → host | `Pcm` (2) | interleaved signed 16-bit samples, little-endian |
| worker → host | `End` (3) | nothing |
| worker → host | `Failed` (4) | a UTF-8 message of at most [`MAX_ERROR_BYTES`](#max_error_bytes) |
| worker → host | `Encoded` (5) | 1 to [`MAX_ENCODED_BYTES`](#max_encoded_bytes) bytes of MP3 |

The first request is `Open` or `Sample`, and only once. After `Open` the
worker streams `Pcm` frames; after `Sample` it streams `Encoded` frames,
whose `Format` states the encoded layout (mono, at most
[`MAX_SAMPLE_OUTPUT_HZ`](#max_sample_output_hz)).

Integers in headers are big-endian; samples are little-endian, the
byte order every platform the worker ships on uses natively. A
`duration_ms` of [`UNKNOWN_DURATION`](#unknown_duration) means the container states none.

The worker parses untrusted files, so the host treats it as compromised:
[`ReplyReader`](#replyreader) checks every length and field of each frame, and
[`ReplyOrder`](#replyorder) checks the stream as a whole (one `Format` before any
`Pcm` or `Encoded`, whole frames of samples, only the kind of payload the
request asked for, nothing after `End` or `Failed`). The
host still checks each `Need` against the file it is reading.

| Item | Kind | Summary |
| --- | --- | --- |
| [`PcmHeader`](#pcmheader) | struct | The layout of the PCM that follows, and where it starts. |
| [`ReplyReader`](#replyreader) | struct | Splits the worker's output into checked replies, fed bytes as they arrive (sans IO: the host's reader task and the fuzz target drive the same code). |
| [`PcmWireError`](#pcmwireerror) | enum | Why a frame, or the stream of frames, could not be read. |
| [`PcmRequest`](#pcmrequest) | enum | A frame the host sends the worker. |
| [`PcmReply`](#pcmreply) | enum | A frame the worker sends the host. |
| [`ReplyOrder`](#replyorder) | enum | Where a reply stream is: the rules `ReplyOrder::check` enforces on top of each frame's own checks. |
| [`body_len`](#body_len) | function | Reads a frame's length prefix. |
| [`encode_request`](#encode_request) | function | Encodes a request as a whole frame, length prefix included. |
| [`decode_request`](#decode_request) | function | Decodes a request body (the frame without its length prefix). |
| [`encode_reply`](#encode_reply) | function | Encodes a reply as a whole frame, length prefix included. |
| [`decode_reply`](#decode_reply) | function | Decodes a reply body (the frame without its length prefix) and checks each field against its limit. |
| [`PCM_WORKER_ARG`](#pcm_worker_arg) | constant | First worker argument that selects the PCM decoder. |
| [`MAX_ERROR_BYTES`](#max_error_bytes) | constant | Longest error message a reply carries, in bytes. |
| [`MAX_NEED_BYTES`](#max_need_bytes) | constant | Most bytes one `Need` asks for (1 MiB). |
| [`MAX_PCM_SAMPLES`](#max_pcm_samples) | constant | Most samples (not frames) one `Pcm` frame carries: a FLAC block at its largest (65 535 frames) in stereo fits. |
| [`MAX_REQUEST_BODY`](#max_request_body) | constant | Largest request body: a tag, an offset and one span's bytes. |
| [`MAX_REPLY_BODY`](#max_reply_body) | constant | Largest reply body: a tag and a `Pcm` frame at its largest. |
| [`MAX_ENCODED_BYTES`](#max_encoded_bytes) | constant | Most MP3 bytes one `Encoded` frame carries (128 KiB). |
| [`MAX_SAMPLE_MS`](#max_sample_ms) | constant | Longest sample a `Sample` request may ask for, in milliseconds (ten minutes); the worker cuts a longer request to this. |
| [`MAX_SAMPLE_OUTPUT_HZ`](#max_sample_output_hz) | constant | Highest sample rate a sample is encoded at, in Hz; a faster source is resampled down to it (MP3 tops out at 48 kHz). |
| [`UNKNOWN_DURATION`](#unknown_duration) | constant | The `duration_ms` a `Format` frame carries when the container states no duration. |
| [`MAX_POSITION_MS`](#max_position_ms) | constant | Longest position or duration a `Format` frame may state, in milliseconds: sixty days, the playback core's own limit. |

## Structs

### PcmHeader

```rust
pub struct PcmHeader {
    pub sample_rate: u32,
    pub channels: u8,
    pub start_ms: u64,
    pub duration_ms: Option<u64>,
}
```

The layout of the PCM that follows, and where it starts.

**Fields**

- `sample_rate`: Samples per second per channel, in Hz; 1 to [`MAX_SAMPLE_RATE_HZ`](README.md#max_sample_rate_hz).
- `channels`: Interleaved channels in every `Pcm` frame: 1 or 2.
- `start_ms`: Where the first sample sits in the file, in milliseconds: the
  asked-for start, or the packet boundary before it.
- `duration_ms`: The file's length in milliseconds, when the container states one.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ReplyReader

```rust
pub struct ReplyReader {
    /* private fields */
}
```

Splits the worker's output into checked replies, fed bytes as they
arrive (sans IO: the host's reader task and the fuzz target drive the
same code).

#### ReplyReader::push

```rust
pub fn push(&mut self, bytes: &[u8])
```

Appends bytes read from the worker.

#### ReplyReader::buffered

```rust
pub fn buffered(&self) -> usize
```

Bytes held that are not yet a whole frame.

#### ReplyReader::next_reply

```rust
pub fn next_reply(&mut self) -> Result<Option<PcmReply>, PcmWireError>
```

Takes the next whole reply, if one has arrived.

##### Returns

`None` until a whole frame is buffered.

##### Errors

[`PcmWireError::TooLarge`](#pcmwireerror) as soon as a length prefix is over
[`MAX_REPLY_BODY`](#max_reply_body), or what [`decode_reply`](#decode_reply) returns. The reader is
then broken and returns that error from every later call.

Implements: `Debug`, `Default`

## Enums

### PcmWireError

```rust
pub enum PcmWireError {
    Truncated,
    Trailing,
    TooLarge,
    UnknownTag(u8),
    BadSpan,
    BadFormat,
    BadSamples,
    BadMessage,
    OutOfOrder(&'static str),
}
```

Why a frame, or the stream of frames, could not be read.

**Variants**

- `Truncated`: The body ended before a field it promised.
- `Trailing`: The body has bytes after its last field.
- `TooLarge`: The body is over the frame's size cap.
- `UnknownTag`: The tag byte is not one this direction uses.
- `BadSpan`: A `Need` for no bytes, more than [`MAX_NEED_BYTES`](#max_need_bytes), or a span that
  overflows.
- `BadFormat`: A `Format` with a rate outside 1 to [`MAX_SAMPLE_RATE_HZ`](README.md#max_sample_rate_hz), a channel
  count other than 1 or 2, or a position or duration over
  [`MAX_POSITION_MS`](#max_position_ms).
- `BadSamples`: A `Pcm` frame with no samples, more than [`MAX_PCM_SAMPLES`](#max_pcm_samples), or an
  odd byte count.
- `BadMessage`: An error message that is not UTF-8 or is too long.
- `OutOfOrder`: A frame out of order: `Pcm` before `Format`, a second `Format`, a
  partial frame of samples, or anything after `End` or `Failed`.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### PcmRequest

```rust
pub enum PcmRequest {
    Open { size: u64, start_ms: u64 },
    Data { offset: u64, bytes: Vec<u8> },
    Sample { size: u64, start_ms: u64, length_ms: u64 },
}
```

A frame the host sends the worker.

**Variants**

- `Open`: Start decoding a file of `size` bytes from `start_ms`. The first
  frame on the worker's stdin, and only once.
  - `size`: The file's size in bytes.
  - `start_ms`: Where to start, in milliseconds from the start of the file.
- `Data`: The bytes of the span the worker last asked for.
  - `offset`: The span's offset.
  - `bytes`: The span's bytes.
- `Sample`: Cut `length_ms` of a file of `size` bytes from `start_ms` and encode
  it as MP3. The first frame on the worker's stdin, instead of `Open`.
  - `size`: The file's size in bytes.
  - `start_ms`: Where the sample starts, in milliseconds from the start of the
    file.
  - `length_ms`: How long it runs, in milliseconds; at most [`MAX_SAMPLE_MS`](#max_sample_ms) is
    cut, and less when the file ends first.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PcmReply

```rust
pub enum PcmReply {
    Need { offset: u64, len: u32 },
    Format(PcmHeader),
    Pcm(Vec<i16>),
    End,
    Failed { message: String },
    Encoded(Vec<u8>),
}
```

A frame the worker sends the host.

**Variants**

- `Need`: Send this span of the file.
  - `offset`: The span's offset.
  - `len`: Its length in bytes: 1 to [`MAX_NEED_BYTES`](#max_need_bytes).
- `Format`: The PCM layout, or for a sample the encoded layout; sent once,
  before the first `Pcm` or `Encoded`.
- `Pcm`: Decoded samples, interleaved, a whole number of frames.
- `End`: The file decoded to its end; the worker exits.
- `Failed`: The file could not be decoded; the worker exits.
  - `message`: Why, for logs; at most [`MAX_ERROR_BYTES`](#max_error_bytes).
- `Encoded`: The next bytes of a sample's MP3 stream: 1 to
  [`MAX_ENCODED_BYTES`](#max_encoded_bytes).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ReplyOrder

```rust
pub enum ReplyOrder {
    Opening,
    OpeningSample,
    Streaming(PcmHeader),
    Encoding(PcmHeader),
    Done,
}
```

Where a reply stream is: the rules [`ReplyOrder::check`](#replyordercheck) enforces on top
of each frame's own checks.

**Variants**

- `Opening`: No `Format` yet after `Open`: only `Need`, `Format` or `Failed` may
  come.
- `OpeningSample`: No `Format` yet after `Sample`: only `Need`, `Format` or `Failed`
  may come.
- `Streaming`: Streaming samples in the stated layout.
- `Encoding`: Streaming a sample's MP3 bytes, encoded in the stated layout.
- `Done`: `End` or `Failed` came; nothing more may.

#### ReplyOrder::for_request

```rust
pub fn for_request(request: &PcmRequest) -> Self
```

The order a stream opened by `request` starts in: [`Self::Opening`](#replyorder)
for `Open`, [`Self::OpeningSample`](#replyorder) for `Sample`, and
[`Self::Done`](#replyorder) (nothing may come) for `Data`, which opens nothing.

#### ReplyOrder::check

```rust
pub fn check(&mut self, reply: &PcmReply) -> Result<(), PcmWireError>
```

Checks that `reply` may come next and moves past it.

##### Errors

[`PcmWireError::OutOfOrder`](#pcmwireerror) for `Pcm`, `Encoded` or `End` before
`Format`, a second `Format`, samples that are not a whole number of
frames, `Encoded` in a PCM stream or `Pcm` in a sample's, a sample
format that is not mono or over [`MAX_SAMPLE_OUTPUT_HZ`](#max_sample_output_hz), or any
frame after `End` or `Failed`. The order is then
[`ReplyOrder::Done`](#replyorder).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

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
pub fn encode_request(request: &PcmRequest) -> Result<Vec<u8>, PcmWireError>
```

Encodes a request as a whole frame, length prefix included.

#### Errors

[`PcmWireError::TooLarge`](#pcmwireerror) when a `Data` span is over
[`MAX_NEED_BYTES`](#max_need_bytes).

### decode_request

```rust
pub fn decode_request(body: &[u8]) -> Result<PcmRequest, PcmWireError>
```

Decodes a request body (the frame without its length prefix).

#### Errors

A [`PcmWireError`](#pcmwireerror) for a body that is too large, truncated, has
trailing bytes or an unknown tag.

### encode_reply

```rust
pub fn encode_reply(reply: &PcmReply) -> Result<Vec<u8>, PcmWireError>
```

Encodes a reply as a whole frame, length prefix included.

An error message is cut to [`MAX_ERROR_BYTES`](#max_error_bytes) at a character boundary.

#### Errors

[`PcmWireError::BadSamples`](#pcmwireerror) for a `Pcm` frame with no samples or more
than [`MAX_PCM_SAMPLES`](#max_pcm_samples), or an `Encoded` frame with no bytes or more
than [`MAX_ENCODED_BYTES`](#max_encoded_bytes).

### decode_reply

```rust
pub fn decode_reply(body: &[u8]) -> Result<PcmReply, PcmWireError>
```

Decodes a reply body (the frame without its length prefix) and checks
each field against its limit.

#### Errors

A [`PcmWireError`](#pcmwireerror) for a body that is too large, truncated, has
trailing bytes or an unknown tag, or carries a field out of bounds.

## Constants

### PCM_WORKER_ARG

```rust
pub const PCM_WORKER_ARG: &str = "pcm";
```

First worker argument that selects the PCM decoder.

### MAX_ERROR_BYTES

```rust
pub const MAX_ERROR_BYTES: usize = 1024;
```

Longest error message a reply carries, in bytes.

### MAX_NEED_BYTES

```rust
pub const MAX_NEED_BYTES: u32 = _;
```

Most bytes one `Need` asks for (1 MiB).

### MAX_PCM_SAMPLES

```rust
pub const MAX_PCM_SAMPLES: usize = _;
```

Most samples (not frames) one `Pcm` frame carries: a FLAC block at its
largest (65 535 frames) in stereo fits.

### MAX_REQUEST_BODY

```rust
pub const MAX_REQUEST_BODY: usize = _;
```

Largest request body: a tag, an offset and one span's bytes.

### MAX_REPLY_BODY

```rust
pub const MAX_REPLY_BODY: usize = _;
```

Largest reply body: a tag and a `Pcm` frame at its largest.

### MAX_ENCODED_BYTES

```rust
pub const MAX_ENCODED_BYTES: usize = _;
```

Most MP3 bytes one `Encoded` frame carries (128 KiB).

### MAX_SAMPLE_MS

```rust
pub const MAX_SAMPLE_MS: u64 = _;
```

Longest sample a `Sample` request may ask for, in milliseconds (ten
minutes); the worker cuts a longer request to this.

### MAX_SAMPLE_OUTPUT_HZ

```rust
pub const MAX_SAMPLE_OUTPUT_HZ: u32 = 48_000;
```

Highest sample rate a sample is encoded at, in Hz; a faster source is
resampled down to it (MP3 tops out at 48 kHz).

### UNKNOWN_DURATION

```rust
pub const UNKNOWN_DURATION: u64 = u64::MAX;
```

The `duration_ms` a `Format` frame carries when the container states
no duration.

### MAX_POSITION_MS

```rust
pub const MAX_POSITION_MS: u64 = _;
```

Longest position or duration a `Format` frame may state, in
milliseconds: sixty days, the playback core's own limit.
