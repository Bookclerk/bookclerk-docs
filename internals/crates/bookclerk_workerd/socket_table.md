# bookclerk_workerd::socket_table

The socket broker's bookkeeping, without I/O.

[`SocketTable`](#sockettable) holds every socket one plugin instance has open through
the broker: its id, URL, negotiated subprotocol, attachment and declared
heartbeat pairs. The broker's network code (`socket_broker`) asks it what
to do with each frame the remote side sends and each request the guest
makes, so the decisions the fuzz target checks are the ones the broker
acts on.

A socket is *pending* from [`SocketTable::reserve`](#sockettablereserve) until the handshake
finishes ([`SocketTable::establish`](#sockettableestablish)) or fails
([`SocketTable::abandon`](#sockettableabandon)). Pending sockets count against
[`MAX_SOCKETS_PER_INSTANCE`](../bookclerk_plugin_abi/README.md#max_sockets_per_instance) but are invisible to the guest. Ids are never
reused within one table.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SocketTable`](#sockettable) | struct | Open and pending sockets for one plugin instance. |
| [`Inbound`](#inbound) | enum | What the broker does with one data frame the remote side sent. |
| [`CLOSE_TOO_BIG`](#close_too_big) | constant | WebSocket close code for a message larger than the broker accepts (RFC 6455 "Message Too Big"). |
| [`MAX_ID_PREFIX_BYTES`](#max_id_prefix_bytes) | constant | Longest id prefix `SocketTable::new` keeps, so ids stay within the 64-byte id limit with any counter value. |

## Structs

### SocketTable

```rust
pub struct SocketTable {
    /* private fields */
}
```

Open and pending sockets for one plugin instance.

#### SocketTable::new

```rust
pub fn new(prefix: &str) -> Self
```

An empty table whose ids start with `prefix`.

##### Arguments

* `prefix` - Id prefix; characters outside `[A-Za-z0-9_-]` are
  dropped and it is cut to [`MAX_ID_PREFIX_BYTES`](#max_id_prefix_bytes), so every id passes
  `check_socket_id`.

#### SocketTable::len

```rust
pub fn len(&self) -> usize
```

Sockets counted against the per-instance limit, pending included.

#### SocketTable::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when the table holds no socket, pending or open.

#### SocketTable::reserve

```rust
pub fn reserve(
    &mut self,
    request: &SocketConnectRequest,
) -> Result<(String, SocketTarget)>
```

Validates a connect request and reserves a pending socket for it.

##### Returns

The new socket id and the parsed target to dial.

##### Errors

The `validate_socket_connect` refusal, or `limit_exceeded` when the
instance already holds `MAX_SOCKETS_PER_INSTANCE` sockets.

#### SocketTable::establish

```rust
pub fn establish(&mut self, socket: &str, protocol: Option<String>) -> Option<SocketInfo>
```

Marks a pending socket open once its handshake finished.

##### Arguments

* `socket` - Id from [`Self::reserve`](#sockettablereserve).
* `protocol` - Subprotocol the server chose, if any.

##### Returns

The socket's description for the guest, or `None` when `socket` is
not pending (already open, abandoned, or unknown).

#### SocketTable::abandon

```rust
pub fn abandon(&mut self, socket: &str) -> bool
```

Drops a pending socket whose dial or handshake failed.

##### Returns

True when a pending socket was removed; open sockets are left alone
(they end through [`Self::closed`](#sockettableclosed)).

#### SocketTable::inbound

```rust
pub fn inbound(&self, socket: &str, frame: SocketFrame) -> Inbound
```

Decides what to do with a data frame the remote side sent.

#### SocketTable::outbound

```rust
pub fn outbound(&self, socket: &str, frame: &SocketFrame) -> Result<()>
```

Checks a frame the guest wants to send.

##### Errors

`not_found` when the socket is not open, or the frame check's
refusal.

#### SocketTable::close_request

```rust
pub fn close_request(&self, socket: &str, code: u16, reason: &str) -> Result<Option<u16>>
```

Checks a close the guest asked for.

##### Returns

The close code to send (zero becomes 1000), or `None` when the
socket is not open, which is not an error.

##### Errors

The close code or reason check's refusal.

#### SocketTable::set_attachment

```rust
pub fn set_attachment(&mut self, socket: &str, attachment: Vec<u8>) -> Result<()>
```

Replaces an open socket's attachment.

##### Errors

The attachment check's refusal, or `not_found` when the socket is
not open.

#### SocketTable::closed

```rust
pub fn closed(
    &mut self,
    socket: &str,
    code: u16,
    reason: &str,
    clean: bool,
) -> Option<SocketClosed>
```

Records that an open socket ended and removes it.

##### Arguments

* `socket` - The socket that ended.
* `code` - Close code; 1006 when the connection dropped without one.
* `reason` - Close reason the peer sent; empty for none.
* `clean` - True when the close handshake completed.

##### Returns

The event for the guest, carrying the last attachment, or `None`
when the socket was not open, so a socket is reported closed once.

#### SocketTable::list

```rust
pub fn list(&self) -> Vec<SocketInfo>
```

The open sockets, oldest first.

Implements: `Clone`, `Debug`

## Enums

### Inbound

```rust
pub enum Inbound {
    Reply(SocketFrame),
    Deliver(SocketMessage),
    TooBig,
    Ignore,
}
```

What the broker does with one data frame the remote side sent.

**Variants**

- `Reply`: A declared heartbeat: send this frame back and do not wake the guest.
- `Deliver`: Wake the guest and hand it this message.
- `TooBig`: The frame is over `MAX_SOCKET_FRAME_BYTES`: close the socket with
  [`CLOSE_TOO_BIG`](#close_too_big).
- `Ignore`: The socket is not open (closed, or never known): drop the frame.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Constants

### CLOSE_TOO_BIG

```rust
pub const CLOSE_TOO_BIG: u16 = 1009;
```

WebSocket close code for a message larger than the broker accepts (RFC
6455 "Message Too Big").

### MAX_ID_PREFIX_BYTES

```rust
pub const MAX_ID_PREFIX_BYTES: usize = 32;
```

Longest id prefix [`SocketTable::new`](#sockettablenew) keeps, so ids stay within the
64-byte id limit with any counter value.
