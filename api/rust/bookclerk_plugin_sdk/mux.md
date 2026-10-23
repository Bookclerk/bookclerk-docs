**bookclerk_plugin_sdk > mux**

# Module: mux

## Contents

**Structs**

- [`Mux`](#mux) - One multiplexed endpoint. Cheap to clone (shared tasks + id allocator).
- [`MuxStream`](#muxstream) - One multiplexed logical connection implementing `AsyncRead` + `AsyncWrite`.

**Functions**

- [`live_mux_task_count`](#live_mux_task_count) - In-process mux reader and writer tasks that have not returned.

**Constants**

- [`INITIAL_WINDOW`](#initial_window) - Initial per-stream send credit, in bytes.
- [`MAX_AGGREGATE_BUFFERED`](#max_aggregate_buffered) - Unread bytes kept across every stream on one mux.
- [`MAX_BUFFERED_PER_STREAM`](#max_buffered_per_stream) - Unread bytes kept for one stream. Further Data is discarded and the stream
- [`MAX_FRAME_PAYLOAD`](#max_frame_payload) - Maximum Data-frame payload in bytes (1 MiB); larger writes are split.
- [`MAX_LIVE_STREAMS`](#max_live_streams) - Live local and peer streams, including those not yet accepted.
- [`TYPE_CLOSE`](#type_close) - Frame type byte for closing a multiplexed connection.
- [`TYPE_DATA`](#type_data) - Frame type byte for payload bytes on an open connection.
- [`TYPE_OPEN`](#type_open) - Frame type byte for opening a multiplexed connection.
- [`TYPE_WINDOW`](#type_window) - Frame type byte for a credit-window update.

---

## bookclerk_plugin_sdk::mux::INITIAL_WINDOW

*Constant*: `u32`

Initial per-stream send credit, in bytes.



## bookclerk_plugin_sdk::mux::MAX_AGGREGATE_BUFFERED

*Constant*: `usize`

Unread bytes kept across every stream on one mux.



## bookclerk_plugin_sdk::mux::MAX_BUFFERED_PER_STREAM

*Constant*: `usize`

Unread bytes kept for one stream. Further Data is discarded and the stream
is closed so the reader can still take `Window` and `Close` for others.



## bookclerk_plugin_sdk::mux::MAX_FRAME_PAYLOAD

*Constant*: `usize`

Maximum Data-frame payload in bytes (1 MiB); larger writes are split.



## bookclerk_plugin_sdk::mux::MAX_LIVE_STREAMS

*Constant*: `usize`

Live local and peer streams, including those not yet accepted.



## bookclerk_plugin_sdk::mux::Mux

*Struct*

One multiplexed endpoint. Cheap to clone (shared tasks + id allocator).

Clones share one task owner. Dropping a clone does not stop the reader or
writer while another clone remains. Dropping the last clone, or calling
`Mux::shutdown`, cancels both tasks even if the peer is still connected.

**Methods:**

- `fn open(self: &Self) -> Result<MuxStream>` - Open a logical stream toward the peer.
- `fn accept(self: &Self) -> Result<MuxStream>` - Wait for the next logical stream opened by the peer.
- `fn client<R, W>(reader: R, writer: W) -> Self` - Client endpoint (odd connection ids). Socket-proxy guests and OAuth hosts.
- `fn server<R, W>(reader: R, writer: W) -> Self` - Server endpoint (even connection ids). Socket-proxy gateway and OAuth guests.
- `fn shutdown(self: &Self)` - Stop the reader and writer even if other clones of this mux still exist.
- `fn closed(self: &Self) -> impl Trait` - Wait until the reader and writer tasks have finished.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Mux`



## bookclerk_plugin_sdk::mux::MuxStream

*Struct*

One multiplexed logical connection implementing `AsyncRead` + `AsyncWrite`.

**Methods:**

- `fn id(self: &Self) -> u32` - Stable connection id shared with the peer for this logical stream.

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)`
- **AsyncWrite**
  - `fn poll_write(self: Pin<& mut Self>, cx: & mut Context, buf: &[u8]) -> Poll<std::io::Result<usize>>`
  - `fn poll_flush(self: Pin<& mut Self>, cx: & mut Context) -> Poll<std::io::Result<()>>`
  - `fn poll_shutdown(self: Pin<& mut Self>, cx: & mut Context) -> Poll<std::io::Result<()>>`
- **AsyncRead**
  - `fn poll_read(self: Pin<& mut Self>, cx: & mut Context, buf: & mut ReadBuf) -> Poll<std::io::Result<()>>`



## bookclerk_plugin_sdk::mux::TYPE_CLOSE

*Constant*: `u8`

Frame type byte for closing a multiplexed connection.



## bookclerk_plugin_sdk::mux::TYPE_DATA

*Constant*: `u8`

Frame type byte for payload bytes on an open connection.



## bookclerk_plugin_sdk::mux::TYPE_OPEN

*Constant*: `u8`

Frame type byte for opening a multiplexed connection.



## bookclerk_plugin_sdk::mux::TYPE_WINDOW

*Constant*: `u8`

Frame type byte for a credit-window update.



## bookclerk_plugin_sdk::mux::live_mux_task_count

*Function*

In-process mux reader and writer tasks that have not returned.

```rust
fn live_mux_task_count() -> usize
```



