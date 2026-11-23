# bookclerk_plugin_notifier_email::client

Moves an SMTP [`Session`](smtp.md#session)'s bytes over a connection.

[`converse`](#converse) reads replies, feeds them to the session and writes what it
answers, upgrading the connection when it asks for TLS. The connection
comes from a [`Transport`]: [`HostTransport`](#hosttransport) in the guest (a TCP tunnel
through the host, which checks it against the approved `server`
destination, then rustls), and in-memory pipes in the tests.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Io`](#io) | trait | A byte stream both ways. |
| [`Transport`](#transport) | trait | Opens connections and upgrades them to TLS. |
| [`HostTransport`](#hosttransport) | struct | The guest's transport: a TCP tunnel through the host, then rustls with Mozilla's root certificates. |
| [`converse`](#converse) | function | Runs one session over a fresh connection to `server`. |
| [`BoxIo`](#boxio) | type alias | A connection, plain or TLS. |
| [`STEP_TIMEOUT`](#step_timeout) | constant | How long the server may take to answer one step. |
| [`MAX_BUFFERED_BYTES`](#max_buffered_bytes) | constant | Most unread bytes held from the server; a server that sends more without waiting for us is refused. |

## Traits

### Io

```rust
pub trait Io: AsyncRead + AsyncWrite + Unpin {}
```

A byte stream both ways.

Implemented by: `T`

### Transport

```rust
pub trait Transport {
    fn connect<'life0, 'life1, 'async_trait>(
        &'life0 self,
        server: &'life1 HostPort,
        security: Security,
    ) -> Pin<Box<dyn Future<Output = Result<BoxIo, PluginError>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn start_tls<'life0, 'life1, 'async_trait>(
        &'life0 self,
        hostname: &'life1 str,
        io: BoxIo,
    ) -> Pin<Box<dyn Future<Output = Result<BoxIo, PluginError>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
}
```

Opens connections and upgrades them to TLS.

#### Transport::connect

```rust
fn connect<'life0, 'life1, 'async_trait>(
    &'life0 self,
    server: &'life1 HostPort,
    security: Security,
) -> Pin<Box<dyn Future<Output = Result<BoxIo, PluginError>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Connects to `server`; for [`Security::Tls`](smtp.md#security) the connection is TLS
from the first byte.

##### Errors

When the server can't be reached or TLS fails.

#### Transport::start_tls

```rust
fn start_tls<'life0, 'life1, 'async_trait>(
    &'life0 self,
    hostname: &'life1 str,
    io: BoxIo,
) -> Pin<Box<dyn Future<Output = Result<BoxIo, PluginError>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Upgrades a plain connection to TLS, verifying the certificate for
`hostname`.

##### Errors

When the handshake fails.

Implemented by: `HostTransport`

## Structs

### HostTransport

```rust
pub struct HostTransport;
```

The guest's transport: a TCP tunnel through the host, then rustls with
Mozilla's root certificates.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Transport`

## Functions

### converse

```rust
pub async fn converse(
    transport: &dyn Transport,
    server: &HostPort,
    security: Security,
    session: &mut Session,
    limit: Duration,
) -> Result<(), PluginError>
```

Runs one session over a fresh connection to `server`.

#### Arguments

* `transport` - Where connections come from.
* `server` - The approved server.
* `security` - How the connection is protected.
* `session` - The protocol state.
* `limit` - How long each step may take ([`STEP_TIMEOUT`](#step_timeout) in the guest).

#### Errors

When the server refuses, misbehaves, or can't be reached; the session's
mapping decides which errors the host retries.

## Type aliases

### BoxIo

```rust
pub type BoxIo = Box<dyn Io>;
```

A connection, plain or TLS.

## Constants

### STEP_TIMEOUT

```rust
pub const STEP_TIMEOUT: Duration = _;
```

How long the server may take to answer one step.

### MAX_BUFFERED_BYTES

```rust
pub const MAX_BUFFERED_BYTES: usize = _;
```

Most unread bytes held from the server; a server that sends more without
waiting for us is refused.
