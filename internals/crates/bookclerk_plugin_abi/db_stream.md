# bookclerk_plugin_abi::db_stream

Streamed execute replies (`executeStreamed`).

An adapter that advertises `maxStreamedResultBytes` answers
`executeStreamed` with the same `ExecuteReply` it would have answered
`execute` with, cut into `ExecuteStreamFrame` messages: a head, then for
each statement its columns followed by its rows in bounded slices, then
the engine timing. The frames travel inline when they fit one scalar and
as a `ByteSource` otherwise, so a result past the inline caps arrives
whole without any caller paging by hand.

[`ExecuteStreamDecoder`](#executestreamdecoder) is the sans-IO receiver every transport feeds:
the host vat, the granted HTTP channel and the fuzz target run the same
state machine over the same bytes.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ExecuteStreamLimits`](#executestreamlimits) | struct | What a receiver lets a streamed reply grow to. |
| [`ExecuteStreamDecoder`](#executestreamdecoder) | struct | Sans-IO decoder of a streamed execute reply: feed it bytes in any split, then take the reassembled `ExecuteReply`. |
| [`ExecuteStream`](#executestream) | struct | A streamed execute reply in flight: the frames, either inline or behind a reader, before the receiver reassembles them. |
| [`ExecuteStreamFrame`](#executestreamframe) | enum | One frame of a streamed execute reply, in Rust terms. |
| [`ExecuteStreamBody`](#executestreambody) | enum | How the frames of an `ExecuteStream` are held. |
| [`encode_execute_stream`](#encode_execute_stream) | function | Frames `reply` as the byte sequence `executeStreamed` carries. |
| [`encode_execute_stream_error`](#encode_execute_stream_error) | function | Frames a failure as the one `Err` frame a receiver accepts in place of a reply (the granted HTTP channel answers every outcome as frames). |
| [`decode_execute_stream`](#decode_execute_stream) | function | Decodes a whole streamed reply held in memory. |
| [`STREAM_FRAME_TARGET_BYTES`](#stream_frame_target_bytes) | constant | Encoded size an encoder aims for per `rows` frame. A frame may exceed it only when one row alone does. |

## Structs

### ExecuteStreamLimits

```rust
pub struct ExecuteStreamLimits {
    pub max_bytes: u32,
    pub bytes_limit: &'static str,
    pub max_rows: u32,
    pub rows_limit: &'static str,
}
```

What a receiver lets a streamed reply grow to.

**Fields**

- `max_bytes`: Encoded reply budget in bytes (`maxStreamedResultBytes`, or the inline
  `maxAtomicResultBytes` when the session does not stream).
- `bytes_limit`: Capability field `max_bytes` came from, for the error it raises.
- `max_rows`: Rows one statement may return.
- `rows_limit`: Capability field `max_rows` came from, for the error it raises.

#### ExecuteStreamLimits::from_capabilities

```rust
pub fn from_capabilities(caps: &DbCapabilities) -> Self
```

The streamed budget of `caps`, or its inline caps when it does not
stream.

#### ExecuteStreamLimits::raw_ceiling

```rust
pub fn raw_ceiling(&self) -> u64
```

Raw frame bytes a receiver accepts before failing closed: the reply
budget plus one eighth for frame headers and one window of slack. A
`0` budget accepts one window.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ExecuteStreamDecoder

```rust
pub struct ExecuteStreamDecoder {
    /* private fields */
}
```

Sans-IO decoder of a streamed execute reply: feed it bytes in any split,
then take the reassembled [`ExecuteReply`](db_execute.md#executereply).

It fails closed on more raw bytes than the budget allows, on a statement
with more rows than the row cap, on duplicate column names or a row
whose width disagrees with its statement's columns (when the statement
closes, or as soon as it arrives on a [`Self::streaming`](#executestreamdecoderstreaming) decoder), and
on any frame out of sequence.

A decoder made with [`Self::streaming`](#executestreamdecoderstreaming) counts rows against the budget
but does not keep them: [`Self::push_frames`](#executestreamdecoderpush_frames) hands each row out once,
as it arrives, so a reader can walk a result without holding it.

#### ExecuteStreamDecoder::new

```rust
pub fn new(limits: ExecuteStreamLimits) -> Self
```

A decoder that holds the stream to `limits`.

#### ExecuteStreamDecoder::streaming

```rust
pub fn streaming(limits: ExecuteStreamLimits) -> Self
```

A decoder that holds the stream to `limits` but keeps no rows: each
row is handed out once by [`Self::push_frames`](#executestreamdecoderpush_frames) and counted against
the row cap, and the reply [`Self::finish`](#executestreamdecoderfinish) returns carries every
statement's columns and `rows_affected` with no rows.

#### ExecuteStreamDecoder::is_finished

```rust
pub fn is_finished(&self) -> bool
```

True once the final frame (`End` or `Err`) has been decoded.

#### ExecuteStreamDecoder::push

```rust
pub fn push(&mut self, bytes: &[u8]) -> Result<()>
```

Feeds the next bytes of the stream.

##### Errors

Returns a typed `limit_exceeded` error when the stream exceeds the
byte or row budget, [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a frame is
malformed or out of sequence or bytes follow the final frame, and a
decode failure from a frame's codec.

#### ExecuteStreamDecoder::push_frames

```rust
pub fn push_frames(
    &mut self,
    bytes: &[u8],
    out: &mut Vec<ExecuteStreamFrame>,
) -> Result<()>
```

Feeds the next bytes of the stream and appends the frames they
completed to `out`, in order.

A `Rows` frame carries the rows that arrived; on a
[`Self::streaming`](#executestreamdecoderstreaming) decoder those rows are handed out here and not
kept. A frame is appended only once the decoder has accepted it, so
a row past the budget or out of sequence is never handed out, and
the frames accepted before a failure stay in `out`.

##### Errors

See [`Self::push`](#executestreamdecoderpush).

#### ExecuteStreamDecoder::finish

```rust
pub fn finish(self) -> Result<ExecuteReply>
```

Takes the reassembled reply once the stream has ended.

##### Errors

Returns the adapter's error when the stream ended with `Err`, and
[`PluginError::unavailable`](error.md#pluginerrorunavailable) when it ended before its final frame.

Implements: `Debug`

### ExecuteStream

```rust
pub struct ExecuteStream {
    /* private fields */
}
```

A streamed execute reply in flight: the frames, either inline or behind
a reader, before the receiver reassembles them.

#### ExecuteStream::from_frames

```rust
pub fn from_frames(frames: Vec<u8>) -> Self
```

Frames already in memory: inline when they fit one scalar, otherwise
behind a reader so the wire moves them in windows.

#### ExecuteStream::from_reply

```rust
pub fn from_reply(reply: &ExecuteReply) -> Result<Self>
```

Frames `reply`, as an adapter answering `executeStreamed` does.

##### Errors

See [`encode_execute_stream`](#encode_execute_stream).

#### ExecuteStream::inline

```rust
pub fn inline(frames: Vec<u8>) -> Self
```

Frames held inline (the `inline` member of `ExecuteStreamReply`).

#### ExecuteStream::reader

```rust
pub fn reader(reader: Pin<Box<dyn AsyncRead + Send>>) -> Self
```

Frames behind a reader (the `stream` member of `ExecuteStreamReply`).

#### ExecuteStream::into_body

```rust
pub fn into_body(self) -> ExecuteStreamBody
```

The frames as the transport holds them.

#### ExecuteStream::collect

```rust
pub async fn collect(self, limits: ExecuteStreamLimits) -> Result<ExecuteReply>
```

Reads every frame and reassembles the reply, holding the stream to
`limits`.

##### Errors

Returns the decoder's errors (see [`ExecuteStreamDecoder::push`](#executestreamdecoderpush)) and
[`PluginError::unavailable`](error.md#pluginerrorunavailable) when the reader fails.

#### ExecuteStream::into_bytes

```rust
pub async fn into_bytes(self, limits: ExecuteStreamLimits) -> Result<Vec<u8>>
```

Reads every frame into memory without decoding, refusing more than
`limits` allows (the granted HTTP channel relays the bytes as they are).

##### Errors

Returns a typed `limit_exceeded` error past the raw ceiling and
[`PluginError::unavailable`](error.md#pluginerrorunavailable) when the reader fails.

Implements: `Debug`

## Enums

### ExecuteStreamFrame

```rust
pub enum ExecuteStreamFrame {
    Head { operation_id: String, statement_count: u32 },
    Statement { index: u32, columns: Vec<DbColumn>, rows_affected: u64 },
    Rows(Vec<DbRow>),
    End(DbTiming),
    Err(PluginError),
}
```

One frame of a streamed execute reply, in Rust terms.

**Variants**

- `Head`: First frame.
  - `operation_id`: Echo of the request `operationId`.
  - `statement_count`: Number of `Statement` frames that follow.
- `Statement`: Opens the next statement result; its rows follow.
  - `index`: Request statement index, increasing by one per frame from `0`.
  - `columns`: Result column descriptors.
  - `rows_affected`: Rows changed by a mutation.
- `Rows`: Rows of the open statement, in order.
- `End`: Last frame: the batch committed.
- `Err`: Last frame: the batch failed.

Implements: `Clone`, `Debug`, `PartialEq`

### ExecuteStreamBody

```rust
pub enum ExecuteStreamBody {
    Inline(Vec<u8>),
    Reader(Pin<Box<dyn AsyncRead + Send>>),
}
```

How the frames of an [`ExecuteStream`](#executestream) are held.

**Variants**

- `Inline`: Every frame, in memory; at most one scalar when produced by
  [`ExecuteStream::from_frames`](#executestreamfrom_frames).
- `Reader`: Frames read on demand.

Implements: `Debug`

## Functions

### encode_execute_stream

```rust
pub fn encode_execute_stream(reply: &ExecuteReply) -> Result<Vec<u8>>
```

Frames `reply` as the byte sequence `executeStreamed` carries.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a TEXT cell contains U+0000,
or [`PluginError::internal`](error.md#pluginerrorinternal) when a frame cannot be serialized.

### encode_execute_stream_error

```rust
pub fn encode_execute_stream_error(err: &PluginError) -> Result<Vec<u8>>
```

Frames a failure as the one `Err` frame a receiver accepts in place of a
reply (the granted HTTP channel answers every outcome as frames).

#### Errors

Returns [`PluginError::internal`](error.md#pluginerrorinternal) when the frame cannot be serialized.

### decode_execute_stream

```rust
pub fn decode_execute_stream(
    bytes: &[u8],
    limits: ExecuteStreamLimits,
) -> Result<ExecuteReply>
```

Decodes a whole streamed reply held in memory.

#### Errors

See [`ExecuteStreamDecoder::push`](#executestreamdecoderpush) and [`ExecuteStreamDecoder::finish`](#executestreamdecoderfinish).

## Constants

### STREAM_FRAME_TARGET_BYTES

```rust
pub const STREAM_FRAME_TARGET_BYTES: u32 = _;
```

Encoded size an encoder aims for per `rows` frame. A frame may exceed it
only when one row alone does.
