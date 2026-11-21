# bookclerk_storage::r2_events

Cloudflare R2 event notifications and Queues pull replies, parsed into
change-feed entries.

An R2 bucket announces object writes and deletes on a Cloudflare Queue
(an event notification rule), and the `r2` destination drains that queue
through its HTTP pull consumer to serve `changes`. Both the pull reply
and each message in it come from outside Bookclerk, so this module is
the boundary that reads them: pure functions over bytes, with no I/O,
fuzzed by the `r2_event` target.

A pull reply (`{"success": true, "result": {"messages": [...]}}`) lists
messages with a body and the lease id that acknowledges or retries the
delivery. A body published as `json` or `bytes` comes base64-encoded and
one published as `text` comes as it is; this module accepts either, and
reads the decoded text as an R2 event:

```json
{"account": "...", "action": "PutObject", "bucket": "books",
 "object": {"key": "library/a.m4b", "size": 12, "eTag": "abc"},
 "eventTime": "2026-11-18T08:00:00.000Z"}
```

Events for another account or bucket, or for keys outside the
destination prefix, are skipped. The feed still acknowledges the message
that held them, so a queue belongs to one destination: a second reader
would lose whatever the first one skipped.
Anything else unreadable is an error: the caller counts the message as a
change it could not read.

| Item | Kind | Summary |
| --- | --- | --- |
| [`R2EventScope`](#r2eventscope) | struct | Which objects an `r2` destination owns. |
| [`PulledMessage`](#pulledmessage) | struct | One delivery listed in a pull reply. |
| [`PullReplyError`](#pullreplyerror) | enum | Why a pull reply could not be read. |
| [`parse_pull_reply`](#parse_pull_reply) | function | Reads a Queues pull reply. |
| [`parse_r2_event`](#parse_r2_event) | function | Reads one R2 event notification into changes for `scope`. |
| [`MAX_PULL_REPLY_BYTES`](#max_pull_reply_bytes) | constant | Largest pull reply read, in bytes: a full batch of the largest messages Queues accepts (128 KiB each, base64-encoded) with room for the envelope. |
| [`MAX_PULL_MESSAGES`](#max_pull_messages) | constant | Most messages one pull reply may list (the Queues batch maximum). |
| [`MAX_LEASE_ID_BYTES`](#max_lease_id_bytes) | constant | Longest lease id accepted, in bytes. |

## Structs

### R2EventScope

```rust
pub struct R2EventScope<'a> {
    pub account: &'a str,
    pub bucket: &'a str,
    pub prefix: &'a str,
}
```

Which objects an `r2` destination owns.

**Fields**

- `account`: Cloudflare account id; events naming another account are skipped.
- `bucket`: Bucket name; events for other buckets are skipped.
- `prefix`: Normalized key prefix (empty, or ending in `/`); it is stripped from
  every key, and keys outside it are skipped.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### PulledMessage

```rust
pub struct PulledMessage {
    pub body: Vec<u8>,
    pub lease_id: String,
}
```

One delivery listed in a pull reply.

**Fields**

- `body`: The message body, decoded from base64 when it was encoded.
- `lease_id`: The lease that acknowledges or retries this delivery.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### PullReplyError

```rust
pub enum PullReplyError {
    TooLarge,
    Malformed(String),
    Refused(String),
    TooManyMessages,
}
```

Why a pull reply could not be read.

**Variants**

- `TooLarge`: The reply is over [`MAX_PULL_REPLY_BYTES`](#max_pull_reply_bytes).
- `Malformed`: The reply is not a pull reply.
- `Refused`: The API reported failure.
- `TooManyMessages`: The reply lists more than [`MAX_PULL_MESSAGES`](#max_pull_messages) messages.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### parse_pull_reply

```rust
pub fn parse_pull_reply(reply: &[u8]) -> Result<Vec<PulledMessage>, PullReplyError>
```

Reads a Queues pull reply.

The reply is read as a JSON tree first, so a repeated member counts
once, as its last value, and the reply and each message must be JSON
objects (not the arrays serde would otherwise take for a struct).

#### Arguments

* `reply` - the HTTP response body.

#### Returns

The deliveries in the order the reply lists them.

#### Errors

Returns a [`PullReplyError`](#pullreplyerror) when the reply is too large, malformed,
reports failure, or lists too many messages. A message whose lease id
is empty or over [`MAX_LEASE_ID_BYTES`](#max_lease_id_bytes) makes the whole reply malformed,
since it could be neither acknowledged nor retried.

### parse_r2_event

```rust
pub fn parse_r2_event(
    body: &[u8],
    scope: R2EventScope<'_>,
) -> Result<Vec<ParsedEvent>, EventParseError>
```

Reads one R2 event notification into changes for `scope`.

The body is read as a JSON tree first, so a repeated member counts once,
as its last value, and the event and its `object` must be JSON objects.

#### Arguments

* `body` - the decoded message body.
* `scope` - the destination's account, bucket and key prefix.

#### Returns

The change for an object under `scope` (R2 sends one event per
message), or nothing for another account, bucket or key.

#### Errors

Returns an [`EventParseError`](s3_events.md#eventparseerror) when the body is too large, is not an R2
event, or names an unreadable key.

## Constants

### MAX_PULL_REPLY_BYTES

```rust
pub const MAX_PULL_REPLY_BYTES: usize = _;
```

Largest pull reply read, in bytes: a full batch of the largest messages
Queues accepts (128 KiB each, base64-encoded) with room for the
envelope.

### MAX_PULL_MESSAGES

```rust
pub const MAX_PULL_MESSAGES: usize = 100;
```

Most messages one pull reply may list (the Queues batch maximum).

### MAX_LEASE_ID_BYTES

```rust
pub const MAX_LEASE_ID_BYTES: usize = 4096;
```

Longest lease id accepted, in bytes.
