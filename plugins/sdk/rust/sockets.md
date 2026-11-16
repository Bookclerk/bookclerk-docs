# bookclerk_plugin_sdk::sockets

Host-brokered WebSockets (`[sockets]`) for native guests.

A plugin that declares `[sockets]` (with `capabilities.network.mode =
"outbound"`) receives [`Sockets`] on [`Bindings::sockets`](crate::Bindings)
and exports a [`SocketHandler`](../../../internals/crates/bookclerk_plugin_abi/README.md#sockethandler) on
[`Entrypoints::socket_handler`](../../../internals/crates/bookclerk_plugin_abi/README.md#entrypoints). The host's broker holds
each connection, answers declared heartbeats itself and wakes the handler
once per message, so the plugin process may sleep between messages: the
model of hibernatable Workers WebSockets.

[`HostWebSocket`](#hostwebsocket) wraps one socket id with the same surface the
TypeScript and Python SDKs give their handlers (`send`, `close`,
`serialize_attachment`, `deserialize_attachment`). Attachments are stored
as UTF-8 JSON so a socket opened by one SDK reads the same in another.

| Item | Kind | Summary |
| --- | --- | --- |
| [`HostWebSocket`](#hostwebsocket) | struct | One host-held socket, named by its id, over the plugin's `Sockets` grant. |
| [`text_heartbeat`](#text_heartbeat) | function | A text heartbeat pair: when `request` arrives the broker sends `response` without waking the guest (for example Socket.io's `"2"` ping and `"3"` pong). |
| [`encode_attachment`](#encode_attachment) | function | Encodes an attachment value as the host stores it: UTF-8 JSON. |
| [`decode_attachment`](#decode_attachment) | function | Decodes stored attachment bytes. |

## Re-exports

- [`bookclerk_plugin_abi::SocketAutoResponse`](../../../internals/crates/bookclerk_plugin_abi/generated.md#socketautoresponse)
- [`bookclerk_plugin_abi::SocketClosed`](../../../internals/crates/bookclerk_plugin_abi/generated.md#socketclosed)
- [`bookclerk_plugin_abi::SocketConnectRequest`](../../../internals/crates/bookclerk_plugin_abi/generated.md#socketconnectrequest)
- [`bookclerk_plugin_abi::SocketFrame`](../../../internals/crates/bookclerk_plugin_abi/generated.md#socketframe)
- [`bookclerk_plugin_abi::SocketHandler`](../../../internals/crates/bookclerk_plugin_abi/README.md#sockethandler)
- [`bookclerk_plugin_abi::SocketHeader`](../../../internals/crates/bookclerk_plugin_abi/generated.md#socketheader)
- [`bookclerk_plugin_abi::SocketInfo`](../../../internals/crates/bookclerk_plugin_abi/generated.md#socketinfo)
- [`bookclerk_plugin_abi::SocketMessage`](../../../internals/crates/bookclerk_plugin_abi/generated.md#socketmessage)
- [`bookclerk_plugin_abi::Sockets`](../../../internals/crates/bookclerk_plugin_abi/README.md#sockets)
- [`bookclerk_plugin_abi::MAX_SOCKETS_PER_INSTANCE`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_sockets_per_instance)
- [`bookclerk_plugin_abi::MAX_SOCKET_ATTACHMENT_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_socket_attachment_bytes)
- [`bookclerk_plugin_abi::MAX_SOCKET_AUTO_RESPONSES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_socket_auto_responses)
- [`bookclerk_plugin_abi::MAX_SOCKET_AUTO_RESPONSE_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_socket_auto_response_bytes)
- [`bookclerk_plugin_abi::MAX_SOCKET_FRAME_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_socket_frame_bytes)
- [`bookclerk_plugin_abi::MAX_SOCKET_HEADERS`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_socket_headers)
- [`bookclerk_plugin_abi::MAX_SOCKET_URL_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_socket_url_bytes)

## Structs

### HostWebSocket

```rust
pub struct HostWebSocket<'a> {
    /* private fields */
}
```

One host-held socket, named by its id, over the plugin's [`Sockets`](../../../internals/crates/bookclerk_plugin_abi/README.md#sockets) grant.

The handle holds no connection: it is cheap to build from a
[`SocketMessage`](../../../internals/crates/bookclerk_plugin_abi/generated.md#socketmessage) or [`SocketClosed`](../../../internals/crates/bookclerk_plugin_abi/generated.md#socketclosed) in the handler, and stays valid
across invocations while the socket is open.

#### HostWebSocket::connect

```rust
pub async fn connect(
    sockets: &'a dyn Sockets,
    request: SocketConnectRequest,
) -> Result<(Self, SocketInfo), PluginError>
```

Opens a socket and wraps it.

##### Arguments

* `sockets` - The plugin's `SOCKETS` binding.
* `request` - URL, subprotocols, headers, heartbeats and attachment.

##### Returns

The handle and the host's [`SocketInfo`](../../../internals/crates/bookclerk_plugin_abi/generated.md#socketinfo) for it.

##### Errors

Returns the host's error: `invalid_params` for a request outside the
ABI rules, `forbidden` for a URL off the network grant,
`limit_exceeded` past [`MAX_SOCKETS_PER_INSTANCE`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_sockets_per_instance).

#### HostWebSocket::from_info

```rust
pub fn from_info(sockets: &'a dyn Sockets, info: &SocketInfo) -> Self
```

Wraps a socket `connect` or `list` returned.

##### Arguments

* `sockets` - The plugin's `SOCKETS` binding.
* `info` - The socket.

##### Returns

The handle.

#### HostWebSocket::from_message

```rust
pub fn from_message(sockets: &'a dyn Sockets, event: &SocketMessage) -> Self
```

Wraps the socket a message arrived on.

##### Arguments

* `sockets` - The plugin's `SOCKETS` binding.
* `event` - The message.

##### Returns

The handle, carrying the attachment the message arrived with.

#### HostWebSocket::new

```rust
pub fn new(sockets: &'a dyn Sockets, id: String, attachment: Vec<u8>) -> Self
```

Wraps a socket by id.

##### Arguments

* `sockets` - The plugin's `SOCKETS` binding.
* `id` - Host-minted socket id.
* `attachment` - Attachment last seen for it.

##### Returns

The handle.

#### HostWebSocket::id

```rust
pub fn id(&self) -> &str
```

Host-minted socket id.

#### HostWebSocket::send

```rust
pub async fn send(&self, frame: SocketFrame) -> Result<(), PluginError>
```

Sends one frame.

##### Errors

Returns `payload_too_large` past [`MAX_SOCKET_FRAME_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_socket_frame_bytes) and
`not_found` when the socket is closed.

#### HostWebSocket::send_text

```rust
pub async fn send_text(&self, text: impl Into<String>) -> Result<(), PluginError>
```

Sends one text frame.

##### Errors

As [`HostWebSocket::send`](#hostwebsocketsend).

#### HostWebSocket::close

```rust
pub async fn close(&self, code: u16, reason: impl Into<String>) -> Result<(), PluginError>
```

Closes the socket.

##### Arguments

* `code` - Close code; `0` sends 1000.
* `reason` - Close reason, at most 123 UTF-8 bytes.

##### Errors

Returns `invalid_params` for a code a peer may not send.

#### HostWebSocket::serialize_attachment

```rust
pub async fn serialize_attachment<T: Serialize + ?Sized>(
    &mut self,
    value: &T,
) -> Result<(), PluginError>
```

Stores a JSON value with the socket; every later message carries it.

##### Errors

As [`encode_attachment`](#encode_attachment), plus the host's `not_found` for a closed
socket.

#### HostWebSocket::deserialize_attachment

```rust
pub fn deserialize_attachment<T: DeserializeOwned>(
    &self,
) -> Result<Option<T>, PluginError>
```

The stored attachment, decoded.

##### Errors

As [`decode_attachment`](#decode_attachment).

## Functions

### text_heartbeat

```rust
pub fn text_heartbeat(
    request: impl Into<String>,
    response: impl Into<String>,
) -> SocketAutoResponse
```

A text heartbeat pair: when `request` arrives the broker sends `response`
without waking the guest (for example Socket.io's `"2"` ping and `"3"` pong).

#### Arguments

* `request` - Text frame to match byte for byte.
* `response` - Text frame to send back.

#### Returns

The pair for `SocketConnectRequest::auto_responses`.

### encode_attachment

```rust
pub fn encode_attachment<T: Serialize + ?Sized>(value: &T) -> Result<Vec<u8>, PluginError>
```

Encodes an attachment value as the host stores it: UTF-8 JSON.

#### Arguments

* `value` - Value to keep with the socket.

#### Returns

At most [`MAX_SOCKET_ATTACHMENT_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_socket_attachment_bytes) bytes.

#### Errors

Returns `invalid_params` when the value does not serialize as JSON and
`payload_too_large` when it does not fit.

### decode_attachment

```rust
pub fn decode_attachment<T: DeserializeOwned>(
    bytes: &[u8],
) -> Result<Option<T>, PluginError>
```

Decodes stored attachment bytes.

#### Arguments

* `bytes` - Attachment the host kept.

#### Returns

`None` when no attachment is set.

#### Errors

Returns `invalid_params` when the bytes are not JSON of type `T`.
