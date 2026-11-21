# bookclerk_workerd::socket_broker

The socket broker: holds a plugin instance's WebSockets while its guest
sleeps.

The host starts `bookclerk-workerd` in broker mode
([`SOCKET_BROKER_ENV`](#socket_broker_env)) inside the plugin's jail, under the plugin's
network grant. The broker serves the plugin ABI's `Sockets` interface to
the host on stdio and delivers what arrives through the host's
`SocketHandler` (see `bookclerk_plugin_abi::serve_socket_broker`), in the
manner of hibernatable Workers WebSockets:

- every dial passes the egress policy: the URL's host must be on a
  `fetch` domain grant or a TCP grant for that host and port
  ([`target_granted`](#target_granted)), and the broker connects only to resolved
  addresses the policy allows, with no redirects;
- `wss://` uses rustls on the hosts' one provider with Mozilla's roots;
- heartbeat frames the guest declared at connect are answered here and
  never wake the guest ([`SocketTable::inbound`](socket_table.md#sockettableinbound));
- other messages queue per socket and reach the host one at a time; the
  broker stops reading a socket whose queue is full, and keeps answering
  heartbeats otherwise while a delivery waits for the guest to wake;
- a socket's close reaches the host after its last message, once.

The decisions live in [`crate::socket_table`](socket_table.md), which the fuzz target
drives without I/O.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SocketBroker`](#socketbroker) | struct | The broker's `Sockets` for one plugin instance. |
| [`default_tls`](#default_tls) | function | TLS settings with Mozilla's roots on the hosts' one rustls provider. |
| [`target_granted`](#target_granted) | function | True when the grant covers the socket's target. |
| [`serve_stdio`](#serve_stdio) | function | Serves the broker on stdio until the host hangs up. |
| [`SOCKET_BROKER_ENV`](#socket_broker_env) | constant | Environment variable that starts `bookclerk-workerd` as the plugin's socket broker (value `1`) instead of its front door. |

## Structs

### SocketBroker

```rust
pub struct SocketBroker {
    /* private fields */
}
```

The broker's [`Sockets`](../bookclerk_plugin_abi/README.md#sockets) for one plugin instance.

#### SocketBroker::new

```rust
pub fn new(
    policy: EgressPolicy,
    tls: Arc<ClientConfig>,
    handler: Rc<dyn SocketHandler>,
    id_prefix: &str,
) -> Self
```

A broker with no sockets.

Must be used inside a `tokio::task::LocalSet`: each socket runs on
local tasks because the host's handler is a Cap'n Proto capability.

##### Arguments

* `policy` - The plugin's effective egress policy.
* `tls` - TLS settings for `wss://` ([`default_tls`](#default_tls) in production).
* `handler` - The host's handler for messages and closes.
* `id_prefix` - Prefix for socket ids, unique per broker start so ids
  are not reused across restarts.

#### SocketBroker::with_tunnels

```rust
pub fn with_tunnels(self, tunnels: Arc<Mux>) -> Self
```

Sends every dial through the host's proxy link as a tunnel.

##### Arguments

* `tunnels` - The host proxy link.

##### Returns

Updated `Self` for chaining.

##### Panics

Never: the shared state has no other owner until the broker serves.

Implements: `Sockets`

## Functions

### default_tls

```rust
pub fn default_tls() -> Result<Arc<ClientConfig>>
```

TLS settings with Mozilla's roots on the hosts' one rustls provider.

#### Errors

Returns an internal error when the provider offers no safe protocol
version (it always does).

### target_granted

```rust
pub fn target_granted(policy: &EgressPolicy, target: &SocketTarget) -> bool
```

True when the grant covers the socket's target.

A WebSocket starts as an HTTP request, so a `fetch` domain grant covers
it; a TCP grant for the host and port covers it too, since that grant
already lets the plugin open the same connection and speak the protocol
itself. Either way the addresses are checked afterwards.

### serve_stdio

```rust
pub async fn serve_stdio(policy: EgressPolicy, tunnels: Option<Arc<Mux>>) -> Result<()>
```

Serves the broker on stdio until the host hangs up.

#### Arguments

* `policy` - The plugin's effective egress policy.
* `tunnels` - The host's CONNECT link. When set, every socket is dialed
  through it, so the broker's own jail needs no network; `None` dials
  directly (diagnostic spawns only).

#### Errors

Returns the TLS setup or link failure.

## Constants

### SOCKET_BROKER_ENV

```rust
pub const SOCKET_BROKER_ENV: &str = "BOOKCLERK_SOCKET_BROKER";
```

Environment variable that starts `bookclerk-workerd` as the plugin's
socket broker (value `1`) instead of its front door.
