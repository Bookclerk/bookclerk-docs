# bookclerk_plugin_sdk::callback_tunnel

Multiplexed byte tunnel for host-owned OAuth callback listeners.

Thin wrappers around [`crate::mux`]: the host [`TunnelHost::open`](#tunnelhostopen)s one
stream per accepted browser connection; the guest [`TunnelGuest::accept`](#tunnelguestaccept)s
them. Flow control is the mux per-stream credit window.

| Item | Kind | Summary |
| --- | --- | --- |
| [`TunnelHost`](#tunnelhost) | struct | Host end of the callback tunnel: opens logical connections toward the guest. |
| [`TunnelGuest`](#tunnelguest) | struct | Guest end of the callback tunnel: accepts logical connections from the host. |
| [`TunnelStream`](#tunnelstream) | type alias | One multiplexed logical connection implementing `AsyncRead` + `AsyncWrite`. |

## Structs

### TunnelHost

```rust
pub struct TunnelHost {
    /* private fields */
}
```

Host end of the callback tunnel: opens logical connections toward the guest.

#### TunnelHost::new

```rust
pub fn new<R, W>(reader: R, writer: W) -> Self
where
    R: AsyncRead + Send + Unpin + 'static,
    W: AsyncWrite + Send + Unpin + 'static,
```

Spawns reader/writer tasks on the host half of a duplex IPC link.

##### Arguments

* `reader` - Async half that receives guest→host frames.
* `writer` - Async half that sends host→guest frames.

#### TunnelHost::open

```rust
pub async fn open(&self) -> Result<TunnelStream>
```

Opens one logical connection toward the guest and returns its stream.

##### Errors

Returns [`crate::SdkError`](index.md#sdkerror) when the mux writer has exited.

### TunnelGuest

```rust
pub struct TunnelGuest {
    /* private fields */
}
```

Guest end of the callback tunnel: accepts logical connections from the host.

#### TunnelGuest::new

```rust
pub fn new<R, W>(reader: R, writer: W) -> Self
where
    R: AsyncRead + Send + Unpin + 'static,
    W: AsyncWrite + Send + Unpin + 'static,
```

Spawns reader/writer tasks on the guest half of a duplex IPC link.

##### Arguments

* `reader` - Async half that receives host→guest frames.
* `writer` - Async half that sends guest→host frames.

#### TunnelGuest::accept

```rust
pub async fn accept(&mut self) -> Result<TunnelStream>
```

Waits for the next logical connection opened by [`TunnelHost::open`](#tunnelhostopen).

##### Errors

Returns [`crate::SdkError`](index.md#sdkerror) when the mux reader exits.

## Type aliases

### TunnelStream

```rust
pub type TunnelStream = MuxStream;
```

One multiplexed logical connection implementing `AsyncRead` + `AsyncWrite`.
