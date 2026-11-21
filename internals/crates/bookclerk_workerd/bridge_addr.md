# bookclerk_workerd::bridge_addr

Where the launcher reaches workerd's bridge RPC socket, and the stream it
gets.

On Unix the bridge is a Unix socket the launcher binds and hands to
workerd with `--socket-fd`, so neither process needs an IP socket and the
jail can deny them all. Windows workerd has no Unix sockets, so there the
bridge stays on in-container loopback TCP.

| Item | Kind | Summary |
| --- | --- | --- |
| [`BridgeAddr`](#bridgeaddr) | enum | Address of the bridge RPC socket. |
| [`BridgeStream`](#bridgestream) | enum | A connection to the bridge. |

## Enums

### BridgeAddr

```rust
pub enum BridgeAddr {
    Tcp(u16),
    Abstract(String),
    Path(PathBuf),
}
```

Address of the bridge RPC socket.

**Variants**

- `Tcp`: `127.0.0.1:port`: Windows, and unconfined smoke runs.
- `Abstract`: A Linux abstract-namespace Unix socket name (no leading NUL).
- `Path`: A pathname Unix socket.

#### BridgeAddr::host_header

```rust
pub fn host_header(&self) -> String
```

`Host` header value for requests on this socket.

#### BridgeAddr::connect

```rust
pub async fn connect(&self) -> Result<BridgeStream>
```

Opens a fresh connection to the bridge.

##### Errors

When the connection is refused or the address can't be formed.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

### BridgeStream

```rust
pub enum BridgeStream {
    Tcp(TcpStream),
    Unix(UnixStream),
}
```

A connection to the bridge.

**Variants**

- `Tcp`: Loopback TCP.
- `Unix`: Unix socket.

Implements: `AsyncRead`, `AsyncWrite`, `Debug`
