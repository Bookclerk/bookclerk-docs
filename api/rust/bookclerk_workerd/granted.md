# bookclerk_workerd::granted

Host-granted HTTP reverse channel for workerd isolates: `JobRunner.job`
streams (`Source` / `Destination` / `ProgressSink`), the job's
`Cancellation` long poll (`/cancel`), named database sessions
(`/db/execute`), and the `EVENTS` outbox publisher (`/events/publish`).

HTTP accept/read/write runs on the multi-thread runtime (`tokio::spawn`) so
it is not starved by the Cap'n Proto vat `LocalSet`. Stub calls stay on the
vat thread via a command channel.

| Item | Kind | Summary |
| --- | --- | --- |
| [`GrantedListener`](#grantedlistener) | trait | Accept loop abstraction (unix or tcp). |
| [`GrantedSlot`](#grantedslot) | struct | One grant token's host stubs (vat-thread only): a `JobRunner.job` invocation, a named database binding, or an `EVENTS` publisher. |
| [`spawn_granted`](#spawn_granted) | function | Starts the Send HTTP accept loop and the vat-thread command dispatcher. |
| [`GrantedTable`](#grantedtable) | type alias | Invocation-id → granted stubs. |
| [`MAX_EVENT_PUBLISH_BODY_BYTES`](#max_event_publish_body_bytes) | constant | Largest `/events/publish` request body: a base64 payload at `MAX_EVENT_PAYLOAD_BYTES` plus the envelope fields. |
| [`MAX_QUEUE_SEND_BODY_BYTES`](#max_queue_send_body_bytes) | constant | Largest `/queues/send` request body: a `sendBatch` of base64 bodies. The host producer still caps every message at `MAX_EVENT_PAYLOAD_BYTES` and every batch at `maxListPage`; this only bounds the read. |
| [`CANCEL_POLL_WINDOW`](#cancel_poll_window) | constant | How long one `GET /cancel` long poll waits before answering `204` so the isolate re-arms; short enough that a dropped isolate cannot pin the cancellation stub for long. |

## Traits

### GrantedListener

```rust
pub trait GrantedListener {
    type Stream;
    fn accept(&self) -> impl Future<Output = Result<Self::Stream>> + Send;
}
```

Accept loop abstraction (unix or tcp).

#### GrantedListener::Stream

```rust
type Stream
```

Accepted connection type.

#### GrantedListener::accept

```rust
fn accept(&self) -> impl Future<Output = Result<Self::Stream>> + Send
```

Accept one connection.

Implemented by: `TcpListener`, `UnixListener`

## Structs

### GrantedSlot

```rust
pub struct GrantedSlot {
    pub input: Option<Box<dyn Source>>,
    pub output: Option<Box<dyn Destination>>,
    pub progress: Option<Box<dyn ProgressSink>>,
    pub cancel: Option<Box<dyn Cancellation>>,
    pub expires: Instant,
    pub allow_open: bool,
    pub allow_put: bool,
    pub allow_progress: bool,
    pub database: Option<Rc<dyn GuestDatabase>>,
    pub allow_database: bool,
    pub sql_policy: GuestSqlPolicy,
    pub max_request_bytes: u32,
    pub events: Option<Rc<dyn EventPublisher>>,
    pub queue: Option<Rc<dyn QueueProducer>>,
    pub kv: Option<Rc<dyn KvNamespace>>,
}
```

One grant token's host stubs (vat-thread only): a `JobRunner.job`
invocation, a named database binding, or an `EVENTS` publisher.

**Fields**

- `input`: Host-granted input source.
- `output`: Host-granted output destination.
- `progress`: Progress sink.
- `cancel`: Host cancellation stub behind the `/cancel` long poll.
- `expires`: Absolute expiry; dispatch fails closed after this instant.
- `allow_open`: Whether `open` is permitted on this grant.
- `allow_put`: Whether `put` is permitted on this grant.
- `allow_progress`: Whether `progress` is permitted on this grant.
- `database`: Host-mediated typed SQL session, when this invocation is granted one.
- `allow_database`: Whether `execute` is permitted on this grant.
- `sql_policy`: Host-issued table/column/function allowlist for guest SQL.
- `max_request_bytes`: Negotiated `maxRequestBytes`. Zero is fail-closed (empty body
  only), never unlimited; grants with a database must store `1..=MAX_SCALAR_BYTES`.
- `events`: Host outbox publisher, when this grant carries the `EVENTS` binding.
- `queue`: Host queue producer, when this grant is one `[[queues.producers]]`
  binding.
- `kv`: Host KV namespace, when this grant is one `[[kv_namespaces]]` binding.

#### GrantedSlot::kv_only

```rust
pub fn kv_only(kv: Rc<dyn KvNamespace>, expires: Instant) -> Self
```

A grant that only reaches one KV namespace over `/kv/*` (no streams,
no database, no events).

#### GrantedSlot::queue_only

```rust
pub fn queue_only(producer: Rc<dyn QueueProducer>, expires: Instant) -> Self
```

A grant that only sends into one queue (no streams, no database).

#### GrantedSlot::events_only

```rust
pub fn events_only(events: Rc<dyn EventPublisher>, expires: Instant) -> Self
```

A grant that only publishes events (no streams, no database).

#### GrantedSlot::database_only

```rust
pub fn database_only(database: Rc<dyn GuestDatabase>, expires: Instant) -> Self
```

A grant that only reaches one named plugin database over
`/db/execute` (no streams, no events). The host-side binding session
enforces the binding-owned scope; the broker defers to it.

## Functions

### spawn_granted

```rust
pub fn spawn_granted<L>(listener: L, token: String, table: GrantedTable)
where
    L: GrantedListener + Send + 'static,
    <L as >::Stream: AsyncRead + AsyncWrite + Unpin + Send + 'static,
```

Starts the Send HTTP accept loop and the vat-thread command dispatcher.

Must be called from a `LocalSet` (dispatcher is `spawn_local`).

## Type aliases

### GrantedTable

```rust
pub type GrantedTable = Rc<RefCell<HashMap<String, GrantedSlot>>>;
```

Invocation-id → granted stubs.

## Constants

### MAX_EVENT_PUBLISH_BODY_BYTES

```rust
pub const MAX_EVENT_PUBLISH_BODY_BYTES: u32 = _;
```

Largest `/events/publish` request body: a base64 payload at
[`MAX_EVENT_PAYLOAD_BYTES`](../bookclerk_plugin_abi/index.md#max_event_payload_bytes) plus the envelope fields.

### MAX_QUEUE_SEND_BODY_BYTES

```rust
pub const MAX_QUEUE_SEND_BODY_BYTES: u32 = _;
```

Largest `/queues/send` request body: a `sendBatch` of base64 bodies. The
host producer still caps every message at [`MAX_EVENT_PAYLOAD_BYTES`](../bookclerk_plugin_abi/index.md#max_event_payload_bytes) and
every batch at `maxListPage`; this only bounds the read.

### CANCEL_POLL_WINDOW

```rust
pub const CANCEL_POLL_WINDOW: Duration = _;
```

How long one `GET /cancel` long poll waits before answering `204` so the
isolate re-arms; short enough that a dropped isolate cannot pin the
cancellation stub for long.
