# bookclerk_storage::s3_events

Object-store event notifications, parsed into change-feed entries.

An S3 bucket can announce writes and deletes on a notification queue.
The `s3` destination drains that queue to serve `changes`
(see `S3Backend::with_notification_queue`). The queue's messages
come from outside Bookclerk, so this module is the boundary that reads
them: a pure function over the message body, with no I/O, fuzzed by the
`s3_event` target.

Three message shapes are read:

- an S3 event notification (`{"Records": [...]}`), sent by S3 straight to
  the queue (MinIO sends the same shape);
- the same notification wrapped in an SNS envelope
  (`{"Type": "Notification", "Message": "<json>"}`), when the bucket
  notifies a topic the queue subscribes to without raw delivery;
- an EventBridge event (`{"source": "aws.s3", "detail-type": ...}`), when
  an EventBridge rule targets the queue.

The shape is the first of `Records`, `Event`, `Type` and `source` that
the message carries (not null) at its top level. S3's own test message
(`"Event": "s3:TestEvent"`) yields no changes.
Records for another bucket, or for keys outside the destination prefix,
are skipped, so one queue can serve several buckets. Anything else is an
error: the caller counts the message as a change it could not read.

| Item | Kind | Summary |
| --- | --- | --- |
| [`EventScope`](#eventscope) | struct | Which objects a destination owns: the bucket and the key prefix that `S3Backend` prepends to every key. |
| [`ParsedEvent`](#parsedevent) | struct | One change read from a notification, with what orders it. |
| [`EventParseError`](#eventparseerror) | enum | Why a notification message could not be read. |
| [`parse_event_message`](#parse_event_message) | function | Reads one notification message into changes for `scope`, in the order the message lists them. |
| [`MAX_EVENT_MESSAGE_BYTES`](#max_event_message_bytes) | constant | Largest message body read, in bytes. SQS caps a message at 1 MiB. |
| [`MAX_EVENT_RECORDS`](#max_event_records) | constant | Most records read from one message. S3 sends one record per message; the cap bounds what a crafted message can make the feed hold. |
| [`MAX_EVENT_KEY_BYTES`](#max_event_key_bytes) | constant | Longest object key accepted, in bytes after decoding (S3's own limit). |

## Structs

### EventScope

```rust
pub struct EventScope<'a> {
    pub bucket: &'a str,
    pub prefix: &'a str,
}
```

Which objects a destination owns: the bucket and the key prefix that
`S3Backend` prepends to every key.

**Fields**

- `bucket`: Bucket name; records for other buckets are skipped.
- `prefix`: Normalized key prefix (empty, or ending in `/`); it is stripped from
  every key, and keys outside it are skipped.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ParsedEvent

```rust
pub struct ParsedEvent {
    pub change: ObjectChange,
    pub time: Option<String>,
    pub sequencer: Option<String>,
}
```

One change read from a notification, with what orders it.

**Fields**

- `change`: The change, keyed relative to the destination prefix.
- `time`: The event time as the message states it (ISO 8601, UTC), when given.
  Times in one message format compare as text.
- `sequencer`: S3's per-key sequencer (hex), when given: of two events for one key,
  the one with the greater sequencer happened later.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### EventParseError

```rust
pub enum EventParseError {
    TooLarge,
    Malformed(String),
    TooManyRecords,
    BadKey,
}
```

Why a notification message could not be read.

**Variants**

- `TooLarge`: The body is over [`MAX_EVENT_MESSAGE_BYTES`](#max_event_message_bytes).
- `Malformed`: The body is not JSON of any known shape.
- `TooManyRecords`: The message holds more than [`MAX_EVENT_RECORDS`](#max_event_records) records.
- `BadKey`: A record's key is not valid percent-encoded UTF-8, holds a NUL, or is
  too long.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### parse_event_message

```rust
pub fn parse_event_message(
    body: &[u8],
    scope: EventScope<'_>,
) -> Result<Vec<ParsedEvent>, EventParseError>
```

Reads one notification message into changes for `scope`, in the order
the message lists them.

#### Arguments

* `body` - the message body as the queue delivered it.
* `scope` - the destination's bucket and key prefix.

#### Returns

The changes for objects under `scope`; empty for a test message or a
message about other buckets or keys.

#### Errors

Returns an [`EventParseError`](#eventparseerror) when the body is too large, is not one of
the known shapes, holds too many records, or names an unreadable key.

## Constants

### MAX_EVENT_MESSAGE_BYTES

```rust
pub const MAX_EVENT_MESSAGE_BYTES: usize = _;
```

Largest message body read, in bytes. SQS caps a message at 1 MiB.

### MAX_EVENT_RECORDS

```rust
pub const MAX_EVENT_RECORDS: usize = 256;
```

Most records read from one message. S3 sends one record per message;
the cap bounds what a crafted message can make the feed hold.

### MAX_EVENT_KEY_BYTES

```rust
pub const MAX_EVENT_KEY_BYTES: usize = 1024;
```

Longest object key accepted, in bytes after decoding (S3's own limit).
