# bookclerk_plugin_sdk.object_store

Object-store destination helpers and the queue change feed.

The same code as the TypeScript SDK's `object-store.ts` and
`queue-feed.ts` and the Rust storage helpers, so a `storage` plugin over a
bucket-style backend behaves the same in every language. A plugin implements
[`ObjectStore`](#objectstore) (head, list, get, put, copy and delete against its
service) and hands it to [`ObjectStoreDestination`](#objectstoredestination), which serves the
whole `storage` entrypoint: staged writes under a commit token with an
idempotent commit, conditional writes, moves and batch calls built from the
backend's calls, object-store key translation, and `changes` from a
[`QueueFeed`](#queuefeed) when the plugin has one. Inside a workerd isolate, import
it as `from bookclerk_plugin_sdk.object_store import ObjectStoreDestination`.

Bodies are opaque here: whatever the backend's `get` returns is what its
`put` takes (a JS `ReadableStream` inside the isolate).

| Name | Kind | Summary |
| --- | --- | --- |
| [`ChangeFeed`](#changefeed) | class | A change feed a destination serves `changes` from. |
| [`LeasedMessage`](#leasedmessage) | class | One leased message. |
| [`NotificationQueue`](#notificationqueue) | class | A queue of notification messages with leases. |
| [`ObjectStore`](#objectstore) | class | The calls a bucket-style backend makes. |
| [`ObjectStoreDestination`](#objectstoredestination) | class | Serves the `storage` entrypoint over an `ObjectStore`. |
| [`ParsedChange`](#parsedchange) | class | One change read from a message, with what orders it. |
| [`QueueFeed`](#queuefeed) | class | A change feed over a `NotificationQueue`, state in KV. |
| [`Received`](#received) | class | What one receive returned. |
| [`WriteCondition`](#writecondition) | class | A write's precondition. |
| [`check_byte_range`](#check_byte_range) | function | Check a read range `{"offset", "length"?}`. |
| [`clamp_page_limit`](#clamp_page_limit) | function | Clamp a page size to `1..=MAX_LIST_PAGE` (0 or `None` asks for the most). |
| [`order_changes`](#order_changes) | function | Order changes by time (unknown first, stable), then each key's by sequencer. |
| [`parse_host_key`](#parse_host_key) | function | Check a host key. |
| [`parse_queue_cursor`](#parse_queue_cursor) | function | Split a `q1.<feed>.<seq>` cursor. |
| [`require_disjoint_prefixes`](#require_disjoint_prefixes) | function | Refuse a prefix move whose source is empty or whose prefixes overlap. |
| [`stage_key`](#stage_key) | function | Return the key a staged write waits under: `.bookclerk-stage/<token>/<key>`. |
| [`translate_for_object_store`](#translate_for_object_store) | function | Translate a checked host key into a key an object store holds well. |
| [`webhook_queue`](#webhook_queue) | function | Return the plugin's `WEBHOOKS` binding as a queue for one `[[webhooks]]` row. |
| [`write_condition`](#write_condition) | function | Return the precondition write `options` ask for. |
| [`MAX_HOST_KEY_BYTES`](#max_host_key_bytes) | constant | Longest host key, in UTF-8 bytes. |
| [`MAX_OBJECT_KEY_BYTES`](#max_object_key_bytes) | constant | Longest object-store key, in UTF-8 bytes (prefix included). |
| [`NO_FEED_CURSOR`](#no_feed_cursor) | constant | Cursor a destination with no feed (or a feed that is off) hands out. |
| [`QUEUE_LEASE_MS`](#queue_lease_ms) | constant | How long a received message stays leased: 12 hours, within the binding's cap. |
| [`STAGE_FOLDER`](#stage_folder) | constant | Folder under which staged writes wait for their commit. |

## Classes

### ChangeFeed

```python
class ChangeFeed(Protocol)
```

A change feed a destination serves `changes` from.

#### ChangeFeed.changes

```python
async def changes(self, cursor: str | None, limit: int) -> dict[str, Any] | None
```

Return one page after `cursor`, or `None` when the feed is off.

### LeasedMessage

```python
@dataclass(frozen=True)
class LeasedMessage
```

One leased message.

**Attributes**

- `body`: The message body.
- `receipt`: The lease to acknowledge or release it by.

### NotificationQueue

```python
class NotificationQueue(Protocol)
```

A queue of notification messages with leases.

#### NotificationQueue.receive

```python
async def receive(self, max_messages: int) -> Received
```

Lease up to `max_messages` waiting messages.

#### NotificationQueue.delete

```python
async def delete(self, receipts: list[str]) -> None
```

Acknowledge handled messages.

#### NotificationQueue.release

```python
async def release(self, receipts: list[str]) -> None
```

Offer messages again on the next receive.

### ObjectStore

```python
class ObjectStore(Protocol)
```

The calls a bucket-style backend makes.

Keys are the host's object keys; the backend adds its own prefix. Objects
are dicts shaped like `ObjectMetadata` (`key`, `size`, optional
`contentType`, `etag`, `sha256`, `modifiedAtMs`, `versionId`,
`signature`) plus an optional `commitToken`. Every failure is a
`PluginError`.

**Attributes**

- `server_copy`: Whether `copy` runs on the server.
- `conditional_writes`: `atomic`, `checked`, or `None` when the backend has no [`put_conditional`](#objectstoreput_conditional).
- `prefix_bytes`: UTF-8 bytes of the backend's own key prefix.

#### ObjectStore.head

```python
async def head(self, key: str) -> dict[str, Any] | None
```

Return the object at `key`, or `None`.

#### ObjectStore.list

```python
async def list(self, prefix: str, cursor: str | None, limit: int) -> dict[str, Any]
```

Return one page `{"objects": [...], "nextCursor"?: str}`.

#### ObjectStore.get

```python
async def get(self, key: str, range: dict[str, int] | None) -> tuple[dict[str, Any], Any]
```

Return `(object, body)`; `not_found` when missing.

#### ObjectStore.put

```python
async def put(self, key: str, body: Any, write: dict[str, Any]) -> dict[str, Any]
```

Store `body`; return `{"bytesWritten", "etag"?, "sha256"?}`.

#### ObjectStore.copy

```python
async def copy(self, source: str, target: str) -> int
```

Copy `source` to `target`; return the bytes copied.

The copy keeps the source's content type, checksum and commit token:
a commit publishes a stage with this call, and a repeated commit after
the stage is gone recognizes the published object by its token.

#### ObjectStore.delete

```python
async def delete(self, key: str) -> None
```

Remove `key`; a missing object is not an error.

#### ObjectStore.put_conditional

```python
async def put_conditional(
    self,
    key: str,
    body: Any,
    write: dict[str, Any],
    condition: WriteCondition,
) -> dict[str, Any]
```

Store `body` only while `condition` holds; `conflict` otherwise.

### ObjectStoreDestination

```python
class ObjectStoreDestination
```

Serves the `storage` entrypoint over an [`ObjectStore`](#objectstore).

**Attributes**

- `store`: The backend.
- `feed`: The change feed, or `None` (the host lists).

#### ObjectStoreDestination.__init__

```python
def __init__(self, store: Any, feed: Any=None) -> None
```

Wrap a backend.

**Args**

- `store`: The backend (an [`ObjectStore`](#objectstore)).
- `feed`: The change feed (a [`ChangeFeed`](#changefeed)), or `None`.

#### ObjectStoreDestination.features

```python
@staticmethod
def features(store: Any, *, changes: bool=False) -> list[str]
```

Return the `rpcFeatures` the destination supports natively.

**Args**

- `store`: The backend (only `conditional_writes` is read).
- `changes`: Whether the plugin serves a change feed.

**Returns**

Feature names.

#### ObjectStoreDestination.head

```python
async def head(self, key: str) -> dict[str, Any] | None
```

Return metadata without a body, or `None` when missing.

**Args**

- `key`: Object key.

**Returns**

The metadata.

#### ObjectStoreDestination.list

```python
async def list(self, options: dict[str, Any]) -> dict[str, Any]
```

Return one page of keys (listing hints not applied).

**Args**

- `options`: `ListOptions` as a dict.

**Returns**

The page.

#### ObjectStoreDestination.get

```python
async def get(self, key: str, options: dict[str, Any] | None=None) -> dict[str, Any]
```

Stream an object.

**Args**

- `key`: Object key.
- `options`: Optional `{"range": ...}`.

**Returns**

`{"meta", "body"}`.

#### ObjectStoreDestination.put

```python
async def put(
    self,
    key: str,
    body: Any,
    options: dict[str, Any] | None=None,
) -> dict[str, Any]
```

Write an object: plain, conditional, or staged under a commit token.

**Args**

- `key`: Object key.
- `body`: The bytes.
- `options`: `WriteOptions` as a dict.

**Returns**

What was written; the etag is the commit token when one is set.

**Raises**

- `PluginError`: `internal` for a SHA-256 that is not 32 bytes.

#### ObjectStoreDestination.copy

```python
async def copy(self, source: str, target: str) -> dict[str, int]
```

Copy within the destination.

**Args**

- `source`: Source key.
- `target`: Target key.

**Returns**

`{"bytesCopied"}`.

#### ObjectStoreDestination.delete

```python
async def delete(self, key: str) -> None
```

Remove an object.

**Args**

- `key`: Object key.

#### ObjectStoreDestination.commit

```python
async def commit(self, key: str, commit_token: str) -> dict[str, Any]
```

Publish a staged write; again after the stage is gone, check the token.

**Args**

- `key`: Object key.
- `commit_token`: The stage's token.

**Returns**

The published object.

**Raises**

- `PluginError`: `not_found` when neither the stage nor a commit of this token exists.

#### ObjectStoreDestination.abort_stage

```python
async def abort_stage(self, key: str, commit_token: str) -> None
```

Discard a staged write; a missing stage is not an error.

**Args**

- `key`: Object key.
- `commit_token`: The stage's token.

#### ObjectStoreDestination.move

```python
async def move(
    self,
    source_key: str,
    target: str,
    options: dict[str, Any] | None=None,
) -> dict[str, bool]
```

Move one object: copy, check it landed whole, then remove the unchanged source.

**Args**

- `source_key`: Source key.
- `target`: Target key.
- `options`: `MoveOptions` as a dict.

**Returns**

`{"atomic", "alreadyMoved"}`.

**Raises**

- `PluginError`: `not_found`, `conflict`, or `internal` when the copy does not match.

#### ObjectStoreDestination.move_many

```python
async def move_many(self, request: dict[str, Any]) -> dict[str, Any]
```

Move pairs, or one page of objects under a prefix.

**Args**

- `request`: `MoveManyRequest` as a dict.

**Returns**

Per-key outcomes and the continuation.

**Raises**

- `PluginError`: `invalid_params` for too many pairs or overlapping prefixes.

#### ObjectStoreDestination.delete_many

```python
async def delete_many(self, keys: list[str]) -> list[dict[str, Any]]
```

Remove up to `MAX_BATCH_KEYS` objects.

**Args**

- `keys`: Object keys.

**Returns**

One outcome per key.

#### ObjectStoreDestination.head_many

```python
async def head_many(self, keys: list[str]) -> list[dict[str, Any] | None]
```

Probe up to `MAX_BATCH_KEYS` objects.

**Args**

- `keys`: Object keys.

**Returns**

One entry per key; `None` when missing.

#### ObjectStoreDestination.translate_keys

```python
async def translate_keys(self, keys: list[str]) -> list[str]
```

Translate host keys into object-store keys.

**Args**

- `keys`: Host keys.

**Returns**

One translated key per host key.

#### ObjectStoreDestination.changes

```python
async def changes(self, request: dict[str, Any]) -> dict[str, Any]
```

Return one page of the change feed, or the "no feed" page.

**Args**

- `request`: `ChangesRequest` as a dict.

**Returns**

The page.

### ParsedChange

```python
@dataclass
class ParsedChange
```

One change read from a message, with what orders it.

**Attributes**

- `change`: The `ObjectChange` dict.
- `time`: Comparable event time text, or `None`.
- `sequencer`: Per-key hex sequencer, or `None`.

### QueueFeed

```python
class QueueFeed
```

A change feed over a [`NotificationQueue`](#notificationqueue), state in KV.

A message is acknowledged once the host sends back a cursor at or past
the page that carried it; a page the host never finished is offered
again. Cursors read `q1.<feed>.<seq>`, as in the other SDKs.

#### QueueFeed.__init__

```python
def __init__(
    self,
    *,
    queue: NotificationQueue,
    parse: Callable[[bytes], list[ParsedChange]],
    kv: Any,
    state_key: str,
) -> None
```

Create the feed; nothing is read until the first page.

**Args**

- `queue`: Where the messages come from.
- `parse`: Reads one message into changes; raise to drop it.
- `kv`: The KV binding the state lives in.
- `state_key`: KV key of the state (name it after the queue, bucket and prefix).

#### QueueFeed.changes

```python
async def changes(self, cursor: str | None, limit: int) -> dict[str, Any]
```

Return one page of changes after `cursor`.

**Args**

- `cursor`: The previous page's cursor, or `None` to start over.
- `limit`: Requested changes (0 for the most).

**Returns**

The page.

**Raises**

- `PluginError`: `invalid_cursor` for a cursor that is not this feed's or is ahead of it.

### Received

```python
@dataclass(frozen=True)
class Received
```

What one receive returned.

**Attributes**

- `messages`: Leased messages, oldest first.
- `missed`: True when the queue dropped messages since the last receive.

### WriteCondition

```python
@dataclass(frozen=True)
class WriteCondition
```

A write's precondition.

**Attributes**

- `kind`: `ifMatch` (the stored etag must equal `etag`) or `ifNoneMatch` (no object may exist).
- `etag`: The expected entity tag, for `ifMatch`.

## Functions

### check_byte_range

```python
def check_byte_range(
    byte_range: dict[str, int] | None,
    size: int | None=None,
) -> dict[str, int] | None
```

Check a read range `{"offset", "length"?}`.

A length of 0 is ambiguous (leave it out to read to the end); with
`size` the range must also lie within the object.

**Args**

- `byte_range`: The requested range, or `None`.
- `size`: The object's size, when known.

**Returns**

The range.

**Raises**

- `PluginError`: `invalid_params` for a range that cannot be read.

### clamp_page_limit

```python
def clamp_page_limit(limit: int | None) -> int
```

Clamp a page size to `1..=MAX_LIST_PAGE` (0 or `None` asks for the most).

**Args**

- `limit`: The requested size.

**Returns**

The size to use.

### order_changes

```python
def order_changes(events: list[ParsedChange]) -> None
```

Order changes by time (unknown first, stable), then each key's by sequencer.

**Args**

- `events`: Changes, reordered in place.

### parse_host_key

```python
def parse_host_key(key: str) -> str
```

Check a host key.

1 to [`MAX_HOST_KEY_BYTES`](#max_host_key_bytes) bytes of `/`-separated segments, none
empty, `.` or `..`, and none holding a backslash or a control
character.

**Args**

- `key`: The host key.

**Returns**

The key.

**Raises**

- `PluginError`: `invalid_params` when it is not one.

### parse_queue_cursor

```python
def parse_queue_cursor(cursor: str) -> tuple[str, int]
```

Split a `q1.<feed>.<seq>` cursor.

**Args**

- `cursor`: A cursor this kind of feed wrote.

**Returns**

The feed id and sequence.

**Raises**

- `PluginError`: `invalid_cursor` for any other text.

### require_disjoint_prefixes

```python
def require_disjoint_prefixes(source: str, target: str) -> None
```

Refuse a prefix move whose source is empty or whose prefixes overlap.

**Args**

- `source`: Source prefix.
- `target`: Target prefix.

**Raises**

- `PluginError`: `invalid_params` when they cannot be moved apart.

### stage_key

```python
def stage_key(token: str | None, key: str) -> str
```

Return the key a staged write waits under: `.bookclerk-stage/<token>/<key>`.

**Args**

- `token`: The commit token.
- `key`: The object key.

**Returns**

The stage key.

**Raises**

- `PluginError`: `invalid_params` for an empty token, one holding a path separator, `..` or NUL, or an empty key.

### translate_for_object_store

```python
def translate_for_object_store(key: str, prefix_bytes: int) -> str
```

Translate a checked host key into a key an object store holds well.

Awkward characters are replaced, and the whole key is shortened to fit
[`MAX_OBJECT_KEY_BYTES`](#max_object_key_bytes) after the backend's own prefix.

**Args**

- `key`: A key [`parse_host_key`](#parse_host_key) accepted.
- `prefix_bytes`: UTF-8 bytes of the backend's prefix.

**Returns**

The object-store key.

### webhook_queue

```python
def webhook_queue(webhooks: Any, name: str) -> NotificationQueue
```

Return the plugin's `WEBHOOKS` binding as a queue for one `[[webhooks]]` row.

**Args**

- `webhooks`: The binding.
- `name`: The row's name.

**Returns**

The queue.

### write_condition

```python
def write_condition(options: dict[str, Any]) -> WriteCondition | None
```

Return the precondition write `options` ask for.

**Args**

- `options`: `WriteOptions` as a dict.

**Returns**

The condition, or `None` for a plain write.

**Raises**

- `PluginError`: `invalid_params` for both `ifMatch` and `ifNoneMatch`, or a conditional write that is also staged.

## Constants

### MAX_HOST_KEY_BYTES

```python
MAX_HOST_KEY_BYTES = 1024
```

Longest host key, in UTF-8 bytes.

### MAX_OBJECT_KEY_BYTES

```python
MAX_OBJECT_KEY_BYTES = 1024
```

Longest object-store key, in UTF-8 bytes (prefix included).

### NO_FEED_CURSOR

```python
NO_FEED_CURSOR = 'none'
```

Cursor a destination with no feed (or a feed that is off) hands out.

### QUEUE_LEASE_MS

```python
QUEUE_LEASE_MS = min(12 * 60 * 60 * 1000, MAX_WEBHOOK_VISIBILITY_MS)
```

How long a received message stays leased: 12 hours, within the binding's cap.

### STAGE_FOLDER

```python
STAGE_FOLDER = '.bookclerk-stage'
```

Folder under which staged writes wait for their commit.
