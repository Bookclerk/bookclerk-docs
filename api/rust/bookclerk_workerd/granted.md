**bookclerk_workerd > granted**

# Module: granted

## Contents

**Structs**

- [`GrantedSlot`](#grantedslot) - One grant token's host stubs (vat-thread only): a `JobRunner.job`

**Functions**

- [`spawn_granted`](#spawn_granted) - Starts the Send HTTP accept loop and the vat-thread command dispatcher.

**Traits**

- [`GrantedListener`](#grantedlistener) - Accept loop abstraction (unix or tcp).

**Constants**

- [`CANCEL_POLL_WINDOW`](#cancel_poll_window) - How long one `GET /cancel` long poll waits before answering `204` so the
- [`MAX_EVENT_PUBLISH_BODY_BYTES`](#max_event_publish_body_bytes) - Largest `/events/publish` request body: a base64 payload at
- [`MAX_QUEUE_SEND_BODY_BYTES`](#max_queue_send_body_bytes) - Largest `/queues/send` request body: a `sendBatch` of base64 bodies. The

**Type Aliases**

- [`GrantedTable`](#grantedtable) - Invocation-id → granted stubs.

---

## bookclerk_workerd::granted::CANCEL_POLL_WINDOW

*Constant*: `std::time::Duration`

How long one `GET /cancel` long poll waits before answering `204` so the
isolate re-arms; short enough that a dropped isolate cannot pin the
cancellation stub for long.



## bookclerk_workerd::granted::GrantedListener

*Trait*

Accept loop abstraction (unix or tcp).

**Methods:**

- `Stream`: Accepted connection type.
- `accept`: Accept one connection.



## bookclerk_workerd::granted::GrantedSlot

*Struct*

One grant token's host stubs (vat-thread only): a `JobRunner.job`
invocation, a named database binding, or an `EVENTS` publisher.

**Fields:**
- `input: Option<Box<dyn Source>>` - Host-granted input source.
- `output: Option<Box<dyn Destination>>` - Host-granted output destination.
- `progress: Option<Box<dyn ProgressSink>>` - Progress sink.
- `cancel: Option<Box<dyn Cancellation>>` - Host cancellation stub behind the `/cancel` long poll.
- `expires: std::time::Instant` - Absolute expiry; dispatch fails closed after this instant.
- `allow_open: bool` - Whether `open` is permitted on this grant.
- `allow_put: bool` - Whether `put` is permitted on this grant.
- `allow_progress: bool` - Whether `progress` is permitted on this grant.
- `database: Option<std::rc::Rc<dyn GuestDatabase>>` - Host-mediated typed SQL session, when this invocation is granted one.
- `allow_database: bool` - Whether `execute` is permitted on this grant.
- `sql_policy: bookclerk_plugin_abi::GuestSqlPolicy` - Host-issued table/column/function allowlist for guest SQL.
- `max_request_bytes: u32` - Negotiated `maxRequestBytes`. Zero is fail-closed (empty body
- `events: Option<std::rc::Rc<dyn EventPublisher>>` - Host outbox publisher, when this grant carries the `EVENTS` binding.
- `queue: Option<std::rc::Rc<dyn QueueProducer>>` - Host queue producer, when this grant is one `[[queues.producers]]`
- `kv: Option<std::rc::Rc<dyn KvNamespace>>` - Host KV namespace, when this grant is one `[[kv_namespaces]]` binding.

**Methods:**

- `fn kv_only(kv: Rc<dyn KvNamespace>, expires: std::time::Instant) -> Self` - A grant that only reaches one KV namespace over `/kv/*` (no streams,
- `fn queue_only(producer: Rc<dyn QueueProducer>, expires: std::time::Instant) -> Self` - A grant that only sends into one queue (no streams, no database).
- `fn events_only(events: Rc<dyn EventPublisher>, expires: std::time::Instant) -> Self` - A grant that only publishes events (no streams, no database).
- `fn database_only(database: Rc<dyn GuestDatabase>, expires: std::time::Instant) -> Self` - A grant that only reaches one named plugin database over



## bookclerk_workerd::granted::GrantedTable

*Type Alias*: `std::rc::Rc<std::cell::RefCell<std::collections::HashMap<String, GrantedSlot>>>`

Invocation-id → granted stubs.



## bookclerk_workerd::granted::MAX_EVENT_PUBLISH_BODY_BYTES

*Constant*: `u32`

Largest `/events/publish` request body: a base64 payload at
[`MAX_EVENT_PAYLOAD_BYTES`](../bookclerk_plugin_abi/limits.md#bookclerk_plugin_abilimitsmax_event_payload_bytes) plus the envelope fields.



## bookclerk_workerd::granted::MAX_QUEUE_SEND_BODY_BYTES

*Constant*: `u32`

Largest `/queues/send` request body: a `sendBatch` of base64 bodies. The
host producer still caps every message at [`MAX_EVENT_PAYLOAD_BYTES`](../bookclerk_plugin_abi/limits.md#bookclerk_plugin_abilimitsmax_event_payload_bytes) and
every batch at `maxListPage`; this only bounds the read.



## bookclerk_workerd::granted::spawn_granted

*Function*

Starts the Send HTTP accept loop and the vat-thread command dispatcher.

Must be called from a `LocalSet` (dispatcher is `spawn_local`).

```rust
fn spawn_granted<L>(listener: L, token: String, table: GrantedTable)
```



