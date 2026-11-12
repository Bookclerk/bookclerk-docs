# bookclerk_plugin_abi::event_read

Validation of one `EventPublisher.read` request and its opaque cursor.

The request comes from a jailed guest: over Cap'n Proto for native guests
and as JSON on the granted `/events/read` route for workerd isolates. The
host checks it here, sans-IO, before it touches the outbox. The
`event_read` fuzz target drives these functions (`fuzz/oracles/event_read.rs`)
with the same code the host runs.

A cursor names the last outbox row a page scanned: its stored `created_at`
text and its id. On the wire it is `v1.<hex created_at>.<hex id>` in
lowercase hex, which guests treat as opaque. Only the canonical spelling
decodes, so a cursor that decodes always re-encodes to the same text.

| Item | Kind | Summary |
| --- | --- | --- |
| [`EventReadRequest`](#eventreadrequest) | struct | Request of `EventPublisher.read`: one page of past outbox events. |
| [`EventReadPage`](#eventreadpage) | struct | Success payload of `EventPublisher.read`. |
| [`EventCursor`](#eventcursor) | struct | Position of the last outbox row a read scanned. |
| [`EventReadQuery`](#eventreadquery) | struct | A read request the host accepted. |
| [`EventReadStart`](#eventreadstart) | enum | Where a validated read starts. |
| [`parse_event_read_json`](#parse_event_read_json) | function | Parses the JSON body of one `/events/read` request (the workerd route's form of `EventReadRequest`): a JSON object with only the request's camelCase fields, each optional. |
| [`is_cursor_event_id`](#is_cursor_event_id) | function | True when an outbox event id can be named by an `EventCursor`: non-empty, at most `MAX_CURSOR_EVENT_ID_BYTES`, with no whitespace or control characters. The outbox refuses any other caller-chosen id, so every stored row stays reachable by a read. |
| [`validate_event_read_request`](#validate_event_read_request) | function | Checks one guest read request against the bounds `plugin.capnp` declares and the instance's consented subscription types. |
| [`MAX_CURSOR_CREATED_AT_BYTES`](#max_cursor_created_at_bytes) | constant | Longest stored `created_at` text a cursor may carry, in bytes. |
| [`MAX_CURSOR_EVENT_ID_BYTES`](#max_cursor_event_id_bytes) | constant | Longest outbox event id a cursor may carry, in bytes. |

## Structs

### EventReadRequest

```rust
pub struct EventReadRequest {
    pub after: String,
    pub since_unix_ms: u64,
    pub types: Vec<String>,
    pub limit: u32,
}
```

Request of `EventPublisher.read`: one page of past outbox events.

**Fields**

- `after`: Opaque cursor from an earlier [`EventReadPage::cursor`](#eventreadpage); empty starts
  at [`Self::since_unix_ms`](#eventreadrequest).
- `since_unix_ms`: UTC Unix milliseconds to start from when `after` is empty; zero starts
  at the oldest retained event.
- `types`: Event types to return; empty means every consented subscription.
- `limit`: Most events to return; zero means `maxListPage`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventReadPage

```rust
pub struct EventReadPage {
    pub events: Vec<DomainEvent>,
    pub cursor: String,
    pub more: bool,
    pub expired_event_ids: Vec<String>,
    pub truncated: bool,
}
```

Success payload of `EventPublisher.read`.

**Fields**

- `events`: Matching events in outbox order. `delivery_attempt` is zero and
  `deduplication_key` is the event id.
- `cursor`: Opaque position after the last event scanned; empty only when nothing
  was scanned and the request had no cursor.
- `more`: True when more events may follow.
- `expired_event_ids`: Ids of the events on this page whose delivery to this plugin expired
  unleased.
- `truncated`: True when the start position is older than the events retention
  window, so earlier events may be gone.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventCursor

```rust
pub struct EventCursor {
    /* private fields */
}
```

Position of the last outbox row a read scanned.

#### EventCursor::new

```rust
pub fn new(
    created_at: impl Into<String>,
    id: impl Into<String>,
) -> Result<Self, PluginError>
```

Builds a cursor for one outbox row.

##### Arguments

* `created_at` - The row's stored `created_at` text.
* `id` - The row's event id.

##### Errors

Returns `invalid_cursor` when `created_at` is not RFC 3339 or either
part is empty, too long, or contains whitespace or control characters.

#### EventCursor::created_at

```rust
pub fn created_at(&self) -> &str
```

The stored `created_at` text of the row.

#### EventCursor::id

```rust
pub fn id(&self) -> &str
```

The event id of the row.

#### EventCursor::encode

```rust
pub fn encode(&self) -> String
```

Encodes the cursor as its opaque wire text.

#### EventCursor::decode

```rust
pub fn decode(text: &str) -> Result<Self, PluginError>
```

Decodes the opaque wire text a guest sent back.

##### Errors

Returns `invalid_cursor` when `text` is longer than
`maxEventCursorBytes`, is not the canonical `v1.<hex>.<hex>` form, or
names a row [`Self::new`](#eventcursornew) would refuse.

Implements: `Clone`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`

### EventReadQuery

```rust
pub struct EventReadQuery {
    pub start: EventReadStart,
    pub types: BTreeSet<String>,
    pub limit: u32,
}
```

A read request the host accepted.

**Fields**

- `start`: Where the read starts.
- `types`: Consented event types to return; never empty.
- `limit`: Most events to return, `1..=maxListPage`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### EventReadStart

```rust
pub enum EventReadStart {
    Oldest,
    Since(DateTime<Utc>),
    After(EventCursor),
}
```

Where a validated read starts.

**Variants**

- `Oldest`: The oldest retained event.
- `Since`: The first event stored at or after this time.
- `After`: The first event after this cursor.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### parse_event_read_json

```rust
pub fn parse_event_read_json(body: &[u8]) -> Result<EventReadRequest, PluginError>
```

Parses the JSON body of one `/events/read` request (the workerd route's
form of [`EventReadRequest`](#eventreadrequest)): a JSON object with only the request's
camelCase fields, each optional.

#### Arguments

* `body` - Request body bytes from the isolate (untrusted).

#### Errors

Returns `invalid_params` when `body` is not such an object. An array is
refused even though serde could read the struct from one.

### is_cursor_event_id

```rust
pub fn is_cursor_event_id(id: &str) -> bool
```

True when an outbox event id can be named by an [`EventCursor`]:
non-empty, at most [`MAX_CURSOR_EVENT_ID_BYTES`](#max_cursor_event_id_bytes), with no whitespace or
control characters. The outbox refuses any other caller-chosen id, so
every stored row stays reachable by a read.

### validate_event_read_request

```rust
pub fn validate_event_read_request(
    request: &EventReadRequest,
    consented: &BTreeSet<String>,
) -> Result<EventReadQuery, PluginError>
```

Checks one guest read request against the bounds `plugin.capnp` declares
and the instance's consented subscription types.

#### Arguments

* `request` - The guest's request.
* `consented` - Event types the instance's consented `[[events.consumers]]`
  rows subscribe to.

#### Returns

The validated query, with an empty `types` list widened to `consented`
and a zero `limit` raised to `maxListPage`.

#### Errors

* `forbidden` when the instance consumes no events, or a requested type
  is not one it consumes.
* `invalid_cursor` when `after` does not decode.
* `invalid_params` when both `after` and `sinceUnixMs` are set, when
  `sinceUnixMs` is not a representable time, or when `limit` or the type
  list exceeds `maxListPage`.

## Constants

### MAX_CURSOR_CREATED_AT_BYTES

```rust
pub const MAX_CURSOR_CREATED_AT_BYTES: usize = 64;
```

Longest stored `created_at` text a cursor may carry, in bytes.

### MAX_CURSOR_EVENT_ID_BYTES

```rust
pub const MAX_CURSOR_EVENT_ID_BYTES: usize = 128;
```

Longest outbox event id a cursor may carry, in bytes.
