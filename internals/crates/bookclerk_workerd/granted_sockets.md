# bookclerk_workerd::granted_sockets

`POST /sockets/<op>` on the granted reverse channel: a workerd isolate's
`env.SOCKETS` calls, parsed and answered over plain byte slices.

The isolate holds a sockets-only grant token minted at `open`; each call
names its operation in the path (`connect`, `send`, `close`,
`attachment`, `list`) and carries that operation's fields as JSON. Binary
frames and attachments travel as base64. Every request is checked with the
same `bookclerk_plugin_abi::sockets` rules the host applies again when it
serves `Sockets`, so a malformed call fails here with a typed error and
never reaches the host. Keeping parsing sans-IO lets the `socket_request`
fuzz target drive the code the bridge runs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`RouteAutoResponse`](#routeautoresponse) | struct | One heartbeat pair as the route carries it. |
| [`RouteConnect`](#routeconnect) | struct | `POST /sockets/connect` body. |
| [`RouteSend`](#routesend) | struct | `POST /sockets/send` body. |
| [`RouteClose`](#routeclose) | struct | `POST /sockets/close` body. |
| [`RouteAttachment`](#routeattachment) | struct | `POST /sockets/attachment` body. |
| [`RouteList`](#routelist) | struct | `POST /sockets/list` body: an empty object. |
| [`RouteFrame`](#routeframe) | enum | One message as the route carries it: `{"text": "..."}` or `{"binary": "<base64>"}`. |
| [`SocketCall`](#socketcall) | enum | One checked `env.SOCKETS` call, ready for the host's `Sockets` stub. |
| [`SocketReply`](#socketreply) | enum | What the host answered one `SocketCall` with. |
| [`parse_socket_call`](#parse_socket_call) | function | Parses and checks one `POST /sockets/<op>` call. |
| [`socket_reply_json`](#socket_reply_json) | function | Encodes a host answer as the route's JSON reply body. |
| [`MAX_SOCKET_ROUTE_BODY_BYTES`](#max_socket_route_body_bytes) | constant | Largest `/sockets/*` request body: one frame at `maxSocketFrameBytes` in its worst JSON form, plus room for a connect request's URL, headers and heartbeat pairs. A text frame of control characters escapes to six bytes per byte (`\u0000`), which outgrows base64's four per three. The ABI checks bound each field once decoded; this only bounds the read. |
| [`SOCKET_ROUTE_OPS`](#socket_route_ops) | constant | Operations the route serves, in path spelling. |

## Structs

### RouteAutoResponse

```rust
pub struct RouteAutoResponse {
    pub request: RouteFrame,
    pub response: RouteFrame,
}
```

One heartbeat pair as the route carries it.

**Fields**

- `request`: Frame to match.
- `response`: Frame to send back.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RouteConnect

```rust
pub struct RouteConnect {
    pub url: String,
    pub protocols: Vec<String>,
    pub headers: Vec<SocketHeader>,
    pub auto_responses: Vec<RouteAutoResponse>,
    pub attachment: Vec<u8>,
}
```

`POST /sockets/connect` body.

**Fields**

- `url`: `wss://` or `ws://` URL on the plugin's network grant.
- `protocols`: Requested subprotocols.
- `headers`: Extra request headers. Values may be credentials.
- `auto_responses`: Heartbeats the broker answers without waking the guest.
- `attachment`: Initial attachment, base64.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RouteSend

```rust
pub struct RouteSend {
    pub socket: String,
    pub frame: RouteFrame,
}
```

`POST /sockets/send` body.

**Fields**

- `socket`: Socket id from `connect` or `list`.
- `frame`: The message.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RouteClose

```rust
pub struct RouteClose {
    pub socket: String,
    pub code: u16,
    pub reason: String,
}
```

`POST /sockets/close` body.

**Fields**

- `socket`: Socket id.
- `code`: WebSocket close code; zero or absent sends 1000.
- `reason`: Close reason, at most 123 UTF-8 bytes.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RouteAttachment

```rust
pub struct RouteAttachment {
    pub socket: String,
    pub attachment: Vec<u8>,
}
```

`POST /sockets/attachment` body.

**Fields**

- `socket`: Socket id.
- `attachment`: New attachment, base64.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RouteList

```rust
pub struct RouteList {}
```

`POST /sockets/list` body: an empty object.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### RouteFrame

```rust
pub enum RouteFrame {
    Text(String),
    Binary(Vec<u8>),
}
```

One message as the route carries it: `{"text": "..."}` or
`{"binary": "<base64>"}`.

**Variants**

- `Text`: Text message, UTF-8.
- `Binary`: Binary message, base64 on the wire.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `From<RouteFrame>`, `From<SocketFrame>`, `PartialEq`, `Serialize`

### SocketCall

```rust
pub enum SocketCall {
    Connect(SocketConnectRequest),
    Send { socket: String, frame: SocketFrame },
    Close { socket: String, code: u16, reason: String },
    SetAttachment { socket: String, attachment: Vec<u8> },
    List,
}
```

One checked `env.SOCKETS` call, ready for the host's `Sockets` stub.

**Variants**

- `Connect`: `Sockets.connect`.
- `Send`: `Sockets.send`.
  - `socket`: Socket id.
  - `frame`: The message.
- `Close`: `Sockets.close`.
  - `socket`: Socket id.
  - `code`: Close code to send (zero already mapped to 1000).
  - `reason`: Close reason.
- `SetAttachment`: `Sockets.setAttachment`.
  - `socket`: Socket id.
  - `attachment`: New attachment.
- `List`: `Sockets.list`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### SocketReply

```rust
pub enum SocketReply {
    Connected(SocketInfo),
    Done,
    Listed(Vec<SocketInfo>),
}
```

What the host answered one [`SocketCall`](#socketcall) with.

**Variants**

- `Connected`: `connect` opened this socket.
- `Done`: `send`, `close` or `attachment` finished.
- `Listed`: `list` found these open sockets, oldest first.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### parse_socket_call

```rust
pub fn parse_socket_call(op: &str, body: &[u8]) -> Result<SocketCall, PluginError>
```

Parses and checks one `POST /sockets/<op>` call.

#### Arguments

* `op` - Path segment after `/sockets/`.
* `body` - Request body bytes, already bounded by
  [`MAX_SOCKET_ROUTE_BODY_BYTES`](#max_socket_route_body_bytes).

#### Returns

The call, with every field within the ABI limits.

#### Errors

Returns `not_found` for an operation the route does not serve,
`invalid_params` for malformed JSON, an unknown field, a malformed socket
id, URL, header, subprotocol, heartbeat or close code, and
`payload_too_large` for an oversized frame or attachment.

### socket_reply_json

```rust
pub fn socket_reply_json(reply: &SocketReply) -> Result<Vec<u8>>
```

Encodes a host answer as the route's JSON reply body.

`connect` answers the [`SocketInfo`](../bookclerk_plugin_abi/generated.md#socketinfo) (attachment as base64), `list`
answers `{"sockets": [...]}`, and the rest answer `{}`.

#### Errors

Returns the serializer's error, which a well-formed [`SocketInfo`](../bookclerk_plugin_abi/generated.md#socketinfo) never
produces.

## Constants

### MAX_SOCKET_ROUTE_BODY_BYTES

```rust
pub const MAX_SOCKET_ROUTE_BODY_BYTES: u32 = _;
```

Largest `/sockets/*` request body: one frame at `maxSocketFrameBytes` in
its worst JSON form, plus room for a connect request's URL, headers and
heartbeat pairs. A text frame of control characters escapes to six bytes
per byte (`\u0000`), which outgrows base64's four per three. The ABI checks
bound each field once decoded; this only bounds the read.

### SOCKET_ROUTE_OPS

```rust
pub const SOCKET_ROUTE_OPS: [&str; 5] = _;
```

Operations the route serves, in path spelling.
