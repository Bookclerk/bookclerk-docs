# bookclerk_plugin_sdk::mux

Bidirectional multiplexed byte streams with per-stream credit windows.

Used for the native-behind-workerd socket proxy (guest [`Mux::open`](#muxopen),
gateway [`Mux::accept`](#muxaccept)) and the OAuth callback tunnel (host opens, guest
accepts). Either side may initiate; connection ids from a client are odd
and from a server are even so they cannot collide.

The peer is untrusted. Frame length is rejected before a payload buffer is
allocated. Each stream buffers at most [`MAX_BUFFERED_PER_STREAM`](#max_buffered_per_stream) unread
bytes, the connection buffers at most [`MAX_AGGREGATE_BUFFERED`](#max_aggregate_buffered), and at
most [`MAX_LIVE_STREAMS`](#max_live_streams) stream objects exist. That cap covers streams
waiting in the accept queue and streams the peer has already closed: a
`Close` frame does not free the slot while the [`MuxStream`](#muxstream) is still
alive. Credit and close records for a generation stay reserved until the
writer drops both, so a blocked writer cannot accumulate them after the
map slot is released. A new Open is refused while either record vector or
that reservation table is already at the cap. Each slot has a generation
so dropping an old stream cannot remove a replacement that reused the id.
Window updates saturate at [`INITIAL_WINDOW`](#initial_window) and are coalesced until the
writer accepts them, so a full control queue cannot drop receive credit.
`Close` is written only after Data already accepted for that same stream.
Window updates and other streams' control frames are not stuck behind one
unread stream. A stream flush or shutdown waits until that stream's
accepted output is written. [`Mux::shutdown`](#muxshutdown) and dropping the last mux
cancel both tasks without draining queued frames.

Frame layout (big-endian):
```text
u32 length_of_rest | u8 type | u32 conn_id | payload
```
Types: `Open=1`, `Data=2`, `Close=3`, `Window=4`. Data payloads are capped
at 1 MiB. A `Window` frame carries a `u32` credit (bytes the peer may
send). Each stream starts with [`INITIAL_WINDOW`](#initial_window) bytes of send credit.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Mux`](#mux) | struct | One multiplexed endpoint. Cheap to clone (shared tasks + id allocator). |
| [`MuxStream`](#muxstream) | struct | One multiplexed logical connection implementing `AsyncRead` + `AsyncWrite`. |
| [`live_mux_task_count`](#live_mux_task_count) | function | In-process mux reader and writer tasks that have not returned. |
| [`TYPE_OPEN`](#type_open) | constant | Frame type byte for opening a multiplexed connection. |
| [`TYPE_DATA`](#type_data) | constant | Frame type byte for payload bytes on an open connection. |
| [`TYPE_CLOSE`](#type_close) | constant | Frame type byte for closing a multiplexed connection. |
| [`TYPE_WINDOW`](#type_window) | constant | Frame type byte for a credit-window update. |
| [`MAX_FRAME_PAYLOAD`](#max_frame_payload) | constant | Maximum Data-frame payload in bytes (1 MiB); larger writes are split. |
| [`INITIAL_WINDOW`](#initial_window) | constant | Initial per-stream send credit, in bytes. |
| [`MAX_BUFFERED_PER_STREAM`](#max_buffered_per_stream) | constant | Unread bytes kept for one stream. Further Data is discarded and the stream is closed so the reader can still take `Window` and `Close` for others. |
| [`MAX_AGGREGATE_BUFFERED`](#max_aggregate_buffered) | constant | Unread bytes kept across every stream on one mux. |
| [`MAX_LIVE_STREAMS`](#max_live_streams) | constant | Live local and peer streams, including those not yet accepted. |

## Structs

### Mux

```rust
pub struct Mux {
    /* private fields */
}
```

One multiplexed endpoint. Cheap to clone (shared tasks + id allocator).

Clones share one task owner. Dropping a clone does not stop the reader or
writer while another clone remains. Dropping the last clone, or calling
[`Mux::shutdown`](#muxshutdown), cancels both tasks even if the peer is still connected.

#### Mux::client

```rust
pub fn client<R, W>(reader: R, writer: W) -> Self
where
    R: AsyncRead + Send + Unpin + 'static,
    W: AsyncWrite + Send + Unpin + 'static,
```

Client endpoint (odd connection ids). Socket-proxy guests and OAuth hosts.

#### Mux::server

```rust
pub fn server<R, W>(reader: R, writer: W) -> Self
where
    R: AsyncRead + Send + Unpin + 'static,
    W: AsyncWrite + Send + Unpin + 'static,
```

Server endpoint (even connection ids). Socket-proxy gateway and OAuth guests.

#### Mux::shutdown

```rust
pub fn shutdown(&self)
```

Stop the reader and writer even if other clones of this mux still exist.

The peer does not have to hang up. [`Self::closed`](#muxclosed) resolves when both
tasks have left their loops and dropped the transport.

#### Mux::closed

```rust
pub fn closed(&self) -> impl Future<Output = ()> + Send + 'static
```

Wait until the reader and writer tasks have finished.

The returned future does not hold this mux. Dropping the last owner
while it is pending still cancels tasks that have not been polled.

#### Mux::open

```rust
pub async fn open(&self) -> Result<MuxStream>
```

Open a logical stream toward the peer.

##### Errors

Returns [`SdkError`](index.md#sdkerror) when the writer task has exited or the live-stream
cap is already full.

#### Mux::accept

```rust
pub async fn accept(&self) -> Result<MuxStream>
```

Wait for the next logical stream opened by the peer.

##### Errors

Returns [`SdkError`](index.md#sdkerror) when the reader task exits (peer hangup or bad frame).

Implements: `Clone`

### MuxStream

```rust
pub struct MuxStream {
    /* private fields */
}
```

One multiplexed logical connection implementing `AsyncRead` + `AsyncWrite`.

#### MuxStream::id

```rust
pub fn id(&self) -> u32
```

Stable connection id shared with the peer for this logical stream.

Implements: `AsyncRead`, `AsyncWrite`, `Drop`

## Functions

### live_mux_task_count

```rust
pub fn live_mux_task_count() -> usize
```

In-process mux reader and writer tasks that have not returned.

## Constants

### TYPE_OPEN

```rust
pub const TYPE_OPEN: u8 = 1;
```

Frame type byte for opening a multiplexed connection.

### TYPE_DATA

```rust
pub const TYPE_DATA: u8 = 2;
```

Frame type byte for payload bytes on an open connection.

### TYPE_CLOSE

```rust
pub const TYPE_CLOSE: u8 = 3;
```

Frame type byte for closing a multiplexed connection.

### TYPE_WINDOW

```rust
pub const TYPE_WINDOW: u8 = 4;
```

Frame type byte for a credit-window update.

### MAX_FRAME_PAYLOAD

```rust
pub const MAX_FRAME_PAYLOAD: usize = _;
```

Maximum Data-frame payload in bytes (1 MiB); larger writes are split.

### INITIAL_WINDOW

```rust
pub const INITIAL_WINDOW: u32 = _;
```

Initial per-stream send credit, in bytes.

### MAX_BUFFERED_PER_STREAM

```rust
pub const MAX_BUFFERED_PER_STREAM: usize = _;
```

Unread bytes kept for one stream. Further Data is discarded and the stream
is closed so the reader can still take `Window` and `Close` for others.

### MAX_AGGREGATE_BUFFERED

```rust
pub const MAX_AGGREGATE_BUFFERED: usize = _;
```

Unread bytes kept across every stream on one mux.

### MAX_LIVE_STREAMS

```rust
pub const MAX_LIVE_STREAMS: usize = 32;
```

Live local and peer streams, including those not yet accepted.
