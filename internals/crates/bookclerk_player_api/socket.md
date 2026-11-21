# bookclerk_player_api::socket

Engine.IO (v4) and Socket.IO (v5) framing of `/player/socket.io`.

The endpoint speaks websocket only: an app opens
`/player/socket.io/?EIO=4&transport=websocket`, the host sends the
Engine.IO open packet, the app connects to the default namespace and
then sends `auth` with its access token, as apps written for an
Audiobookshelf server do. After that the host pushes events
(`user_item_progress_updated`, `user_updated`, `item_updated`, ...)
and pings; the app answers pings and may send its own.

Everything here is sans-IO: [`parse_client_frame`](#parse_client_frame) reads one text frame
an app sent and the `*_frame` functions build the frames the host
sends. Binary frames and polling are refused by the host before they
reach this module.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SocketError`](#socketerror) | enum | Why a frame an app sent was refused. The host closes the socket. |
| [`ClientFrame`](#clientframe) | enum | One text frame an app sent, read. |
| [`ClientEvent`](#clientevent) | enum | An event an app sends. |
| [`parse_client_frame`](#parse_client_frame) | function | Reads one text frame an app sent. |
| [`open_frame`](#open_frame) | function | The Engine.IO open packet (`0{...}`), the first frame the host sends. |
| [`connect_frame`](#connect_frame) | function | The Socket.IO connect answer (`40{"sid":...}`) on the default namespace. |
| [`connect_error_frame`](#connect_error_frame) | function | The Socket.IO connect error (`44{"message":...}`) for a namespace the endpoint does not serve. |
| [`event_frame`](#event_frame) | function | A Socket.IO event (`42["name",data]`) on the default namespace. |
| [`bare_event_frame`](#bare_event_frame) | function | A Socket.IO event with no argument (`42["name"]`), such as `pong`. |
| [`MAX_CLIENT_FRAME_BYTES`](#max_client_frame_bytes) | constant | The longest text frame an app may send, in bytes. An `auth` event with a token is under a hundred bytes; nothing an app sends is near this. |
| [`MAX_EVENT_NAME_BYTES`](#max_event_name_bytes) | constant | The longest event name read back, in bytes. |
| [`PING_INTERVAL_MS`](#ping_interval_ms) | constant | How often the host pings, in milliseconds (Engine.IO's default). |
| [`PING_TIMEOUT_MS`](#ping_timeout_ms) | constant | How long the host waits for a pong before closing, in milliseconds. |
| [`ENGINE_IO_REVISION`](#engine_io_revision) | constant | The Engine.IO protocol revision the endpoint speaks (`EIO=4`). |
| [`PING_FRAME`](#ping_frame) | constant | The Engine.IO ping (`2`) the host sends every `PING_INTERVAL_MS`. |
| [`PONG_FRAME`](#pong_frame) | constant | The Engine.IO pong (`3`) answering an app's ping. |
| [`CLOSE_FRAME`](#close_frame) | constant | The Engine.IO close (`1`). |

## Enums

### SocketError

```rust
pub enum SocketError {
    TooLong,
    UnknownPacket,
    Malformed,
}
```

Why a frame an app sent was refused. The host closes the socket.

**Variants**

- `TooLong`: The frame is longer than [`MAX_CLIENT_FRAME_BYTES`](#max_client_frame_bytes).
- `UnknownPacket`: The frame is empty or its packet type is not one an app sends.
- `Malformed`: A Socket.IO packet whose namespace, ack id or payload is malformed.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### ClientFrame

```rust
pub enum ClientFrame {
    Close,
    Ping,
    Pong,
    Noop,
    Connect { default_namespace: bool, token: Option<String> },
    Disconnect,
    Event { event: ClientEvent, ack: Option<u64> },
}
```

One text frame an app sent, read.

**Variants**

- `Close`: Engine.IO close (`1`): the app is leaving.
- `Ping`: Engine.IO ping (`2`), answered with a pong.
- `Pong`: Engine.IO pong (`3`), the answer to the host's ping.
- `Noop`: Engine.IO upgrade or noop (`5`, `6`): nothing to do.
- `Connect`: Socket.IO connect (`40`) to a namespace.
  - `default_namespace`: True when the namespace is the default one (`/`), the only one
    the endpoint serves.
  - `token`: The `token` field of the connect payload, when the app sent one.
- `Disconnect`: Socket.IO disconnect (`41`).
- `Event`: Socket.IO event (`42`) on the default namespace.
  - `event`: The event the app sent.
  - `ack`: The ack id the app asked to be answered on, if any.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ClientEvent

```rust
pub enum ClientEvent {
    Auth(String),
    Ping,
    Other(String),
}
```

An event an app sends.

**Variants**

- `Auth`: `auth` with an access token: ties the socket to a patron.
- `Ping`: The application-level `ping` event, answered with `pong`.
- `Other`: Any other event (scans, cover search, log listeners): ignored.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### parse_client_frame

```rust
pub fn parse_client_frame(frame: &str) -> Result<ClientFrame, SocketError>
```

Reads one text frame an app sent.

#### Arguments

* `frame` - The websocket text frame.

#### Returns

The frame.

#### Errors

[`SocketError`](#socketerror) when the frame is too long, empty, of a packet type an
app does not send, or malformed.

### open_frame

```rust
pub fn open_frame(sid: &str) -> String
```

The Engine.IO open packet (`0{...}`), the first frame the host sends.

#### Arguments

* `sid` - A fresh, unguessable session id for this socket.

### connect_frame

```rust
pub fn connect_frame(sid: &str) -> String
```

The Socket.IO connect answer (`40{"sid":...}`) on the default namespace.

### connect_error_frame

```rust
pub fn connect_error_frame(namespace_message: &str) -> String
```

The Socket.IO connect error (`44{"message":...}`) for a namespace the
endpoint does not serve.

### event_frame

```rust
pub fn event_frame<T: Serialize>(name: &str, data: &T) -> String
```

A Socket.IO event (`42["name",data]`) on the default namespace.

#### Arguments

* `name` - The event name, such as `user_item_progress_updated`.
* `data` - Its one argument.

### bare_event_frame

```rust
pub fn bare_event_frame(name: &str) -> String
```

A Socket.IO event with no argument (`42["name"]`), such as `pong`.

## Constants

### MAX_CLIENT_FRAME_BYTES

```rust
pub const MAX_CLIENT_FRAME_BYTES: usize = _;
```

The longest text frame an app may send, in bytes. An `auth` event with
a token is under a hundred bytes; nothing an app sends is near this.

### MAX_EVENT_NAME_BYTES

```rust
pub const MAX_EVENT_NAME_BYTES: usize = 64;
```

The longest event name read back, in bytes.

### PING_INTERVAL_MS

```rust
pub const PING_INTERVAL_MS: u64 = 25_000;
```

How often the host pings, in milliseconds (Engine.IO's default).

### PING_TIMEOUT_MS

```rust
pub const PING_TIMEOUT_MS: u64 = 20_000;
```

How long the host waits for a pong before closing, in milliseconds.

### ENGINE_IO_REVISION

```rust
pub const ENGINE_IO_REVISION: &str = "4";
```

The Engine.IO protocol revision the endpoint speaks (`EIO=4`).

### PING_FRAME

```rust
pub const PING_FRAME: &str = "2";
```

The Engine.IO ping (`2`) the host sends every [`PING_INTERVAL_MS`](#ping_interval_ms).

### PONG_FRAME

```rust
pub const PONG_FRAME: &str = "3";
```

The Engine.IO pong (`3`) answering an app's ping.

### CLOSE_FRAME

```rust
pub const CLOSE_FRAME: &str = "1";
```

The Engine.IO close (`1`).
