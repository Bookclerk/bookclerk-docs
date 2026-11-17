# bookclerk_plugin_integration_audiobookshelf::push

Audiobookshelf's pushed events, read from its Socket.io channel.

Audiobookshelf pushes changes to signed-in admins over Socket.io (Engine.IO
v4 over a WebSocket). The host holds that socket while the guest sleeps
(`[sockets]`): it answers the server's `2` ping with `3` itself, and wakes
the guest for every other frame. This module is the sans-IO side of that
conversation: [`socket_url`](#socket_url) names the endpoint, and [`on_frame`](#on_frame) reads one
text frame and says what to send back, whether to close, and whether the
frame announced a new user.

The handshake is:

1. The server opens Engine.IO with `0{...}`; the guest joins the default
   namespace with `40`.
2. The server confirms with `40{...}`; the guest authenticates with
   `42["auth","<api key>"]`.
3. The server answers `42["init",...]` and from then on pushes events
   such as `42["user_added",{...}]`. A rejected key gets
   `42["invalid_token"]`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Reaction`](#reaction) | struct | What the guest does after one pushed frame. |
| [`socket_url`](#socket_url) | function | The WebSocket URL of the Socket.io endpoint under `base_url`. |
| [`on_frame`](#on_frame) | function | Reads one text frame from the server and says how to answer it. |
| [`PING`](#ping) | constant | The Engine.IO ping the server sends; the host answers it without waking the guest. |
| [`PONG`](#pong) | constant | The Engine.IO pong the host sends back for `PING`. |

## Structs

### Reaction

```rust
pub struct Reaction {
    pub reply: Option<String>,
    pub close: bool,
    pub new_user: bool,
}
```

What the guest does after one pushed frame.

**Fields**

- `reply`: A text frame to send back, if any.
- `close`: Whether to close the socket: the server rejected the key or left the
  namespace.
- `new_user`: Whether the frame announced a user created on the server, so the
  host should look for new users now.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

## Functions

### socket_url

```rust
pub fn socket_url(base_url: &str) -> Option<String>
```

The WebSocket URL of the Socket.io endpoint under `base_url`.

`http` becomes `ws` and `https` becomes `wss`; a path prefix on the base
URL (Audiobookshelf behind a sub-path) is kept.

#### Arguments

* `base_url` - The configured server origin, such as `https://abs.example`.

#### Returns

`None` when the base URL is not `http` or `https`.

### on_frame

```rust
pub fn on_frame(frame: &str, api_key: &str) -> Reaction
```

Reads one text frame from the server and says how to answer it.

Unknown or malformed frames are ignored, so a server that adds events or
sends noise never breaks the socket.

#### Arguments

* `frame` - The text frame as received.
* `api_key` - The key to authenticate with once the namespace is joined.

#### Returns

The reply to send, whether to close, and whether a user was added.

## Constants

### PING

```rust
pub const PING: &str = "2";
```

The Engine.IO ping the server sends; the host answers it without waking
the guest.

### PONG

```rust
pub const PONG: &str = "3";
```

The Engine.IO pong the host sends back for [`PING`](#ping).
