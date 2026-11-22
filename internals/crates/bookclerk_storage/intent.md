# bookclerk_storage::intent

Intent operations (moves, batches, conditional writes, change feeds) and
their fallbacks.
Intent-based storage operations and the host's fallback for each.

The host says what it wants (move these objects, delete these keys, probe
these keys, write only if unchanged, what changed since a cursor) and the
backend chooses how. A backend that does an operation natively says so in
[`StorageCapabilities`](#storagecapabilities) and overrides the [`StorageBackend`](README.md#storagebackend) method; every
other backend gets the fallback here, built only from the required
primitives (`head`, `list_page`, `copy`, `get_stream`, `put_stream`,
`delete`). See `docs/plugins/abi.md#intent-operations`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`StorageCapabilities`](#storagecapabilities) | struct | Which intent operations a backend does natively. |
| [`KeyResult`](#keyresult) | struct | Outcome for one key of a batch operation. |
| [`MovePrefixPage`](#moveprefixpage) | struct | One page of a prefix move. |
| [`ConditionalPutResult`](#conditionalputresult) | struct | Result of `StorageBackend::put_stream_conditional`. |
| [`ListRequest`](#listrequest) | struct | A listing request with hints. |
| [`HintedListPage`](#hintedlistpage) | struct | A listing page that says how far the hints were honoured. |
| [`ObjectChange`](#objectchange) | struct | One change in a change feed. |
| [`ChangeFeedPage`](#changefeedpage) | struct | One page of a change feed. |
| [`WriteCondition`](#writecondition) | enum | A precondition on a write. |
| [`fallback_move`](#fallback_move) | function | Moves one object with the required primitives: probe, copy (server-side when the backend supports it, else streamed), check that the copy has the source's size, then delete the source. |
| [`fallback_move_many`](#fallback_move_many) | function | Moves each pair in turn with `StorageBackend::move_object`. One pair's failure does not stop the others. |
| [`require_disjoint_prefixes`](#require_disjoint_prefixes) | function | Rejects a prefix move whose namespaces overlap. |
| [`fallback_move_prefix`](#fallback_move_prefix) | function | Moves one listing page under `from_prefix` to `to_prefix`. |
| [`fallback_delete_many`](#fallback_delete_many) | function | Deletes each key in turn. One key's failure does not stop the others. |
| [`fallback_head_many`](#fallback_head_many) | function | Probes each key in turn. |
| [`fallback_put_conditional`](#fallback_put_conditional) | function | Checks `condition` with a probe, then writes. Not atomic: the result says so. |
| [`fallback_list_hinted`](#fallback_list_hinted) | function | Lists one page and reports that no hint was honoured: no signatures beyond what the plain listing carries, no time filter. |

## Re-exports

- [`bookclerk_plugin_sdk::MoveOptions`](../bookclerk_plugin_abi/README.md#moveoptions)
- [`bookclerk_plugin_sdk::MoveResult`](../bookclerk_plugin_abi/README.md#moveresult)
- [`bookclerk_plugin_sdk::ObjectChangeKind`](../bookclerk_plugin_abi/README.md#objectchangekind)

## Structs

### StorageCapabilities

```rust
pub struct StorageCapabilities {
    pub move_object: bool,
    pub move_many: bool,
    pub delete_many: bool,
    pub head_many: bool,
    pub conditional_write: bool,
    pub checked_write: bool,
    pub changes: bool,
    pub translate_keys: bool,
}
```

Which intent operations a backend does natively.

A `false` field means the host runs its fallback for that operation. The
fallback gives the same result when no one else writes the same keys
meanwhile, but it may be slower (a copy and a delete instead of a rename)
and is not atomic: a concurrent writer can slip in between its steps (a
conditional write checked before writing, a `no_overwrite` target created
after the check).

**Fields**

- `move_object`: Single moves ([`StorageBackend::move_object`](README.md#storagebackendmove_object)).
- `move_many`: Batch and prefix moves ([`StorageBackend::move_many`](README.md#storagebackendmove_many),
  [`StorageBackend::move_prefix`](README.md#storagebackendmove_prefix)).
- `delete_many`: Batch deletes ([`StorageBackend::delete_many`](README.md#storagebackenddelete_many)).
- `head_many`: Batch probes ([`StorageBackend::head_many`](README.md#storagebackendhead_many)).
- `conditional_write`: Conditional writes enforced by the backend
  ([`StorageBackend::put_stream_conditional`](README.md#storagebackendput_stream_conditional)).
- `checked_write`: Conditional writes the backend enforces without an atomic
  primitive: it writes, then checks and withdraws its write when
  another got in first, so a reader may briefly see the losing write
  ([`StorageBackend::put_stream_conditional`](README.md#storagebackendput_stream_conditional) reports `atomic: false`).
  Ignored when [`Self::conditional_write`](#storagecapabilities) is set.
- `changes`: A change feed ([`StorageBackend::changes`](README.md#storagebackendchanges)).
- `translate_keys`: Its own translation of host keys ([`StorageBackend::translate_keys`](README.md#storagebackendtranslate_keys));
  without it a host key is stored as written.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### KeyResult

```rust
pub struct KeyResult {
    pub key: String,
    pub error: Option<StorageError>,
}
```

Outcome for one key of a batch operation.

**Fields**

- `key`: The key (the source key for moves).
- `error`: The failure for this key; `None` when it succeeded.

#### KeyResult::from_result

```rust
pub fn from_result(key: impl Into<String>, result: Result<()>) -> Self
```

Builds an outcome for `key` from a per-key result.

Implements: `Debug`

### MovePrefixPage

```rust
pub struct MovePrefixPage {
    pub results: Vec<KeyResult>,
    pub next_cursor: Option<String>,
    pub atomic: bool,
    pub objects_moved: u64,
}
```

One page of a prefix move.

**Fields**

- `results`: Outcomes, one per object attempted (or one for the whole prefix when
  the backend moved it in one step).
- `next_cursor`: Cursor for the next call; `None` when every object moved.
- `atomic`: True when every move in this page was atomic.
- `objects_moved`: Objects moved in this page.

Implements: `Debug`, `Default`

### ConditionalPutResult

```rust
pub struct ConditionalPutResult {
    pub put: PutStreamResult,
    pub atomic: bool,
}
```

Result of [`StorageBackend::put_stream_conditional`](README.md#storagebackendput_stream_conditional).

**Fields**

- `put`: The write itself.
- `atomic`: True when the backend checked the condition atomically with the
  write. False for the fallback, which checks first and then writes, so
  a concurrent writer can slip in between (the caller's next listing
  sees it as an outside change).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### ListRequest

```rust
pub struct ListRequest {
    pub prefix: String,
    pub cursor: Option<String>,
    pub limit: u32,
    pub with_signatures: bool,
    pub modified_since_ms: Option<u64>,
}
```

A listing request with hints.

**Fields**

- `prefix`: Key prefix filter.
- `cursor`: Opaque continuation token from the previous page.
- `limit`: Requested page size, clamped to [`crate::MAX_LIST_PAGE`](README.md#max_list_page).
- `with_signatures`: Hint: fill the signature fields of each listed object.
- `modified_since_ms`: Hint: only objects modified at or after this UTC unix millisecond.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### HintedListPage

```rust
pub struct HintedListPage {
    pub page: ListPage,
    pub signatures_included: bool,
    pub modified_since_applied: bool,
}
```

A listing page that says how far the hints were honoured.

**Fields**

- `page`: The page.
- `signatures_included`: True when every entry carries the signature fields the backend has.
- `modified_since_applied`: True when objects modified before the requested time were left out;
  false means the page is unfiltered and the caller compares.

Implements: `Clone`, `Debug`, `Default`

### ObjectChange

```rust
pub struct ObjectChange {
    pub key: String,
    pub kind: ObjectChangeKind,
    pub info: Option<ObjectInfo>,
}
```

One change in a change feed.

**Fields**

- `key`: Object key.
- `kind`: What happened to it.
- `info`: Listing fields after the change, when the feed carries them.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ChangeFeedPage

```rust
pub struct ChangeFeedPage {
    pub changes: Vec<ObjectChange>,
    pub cursor: String,
    pub more: bool,
    pub complete: bool,
    pub live_watch: bool,
}
```

One page of a change feed.

**Fields**

- `changes`: Changes since the cursor, oldest first.
- `cursor`: Cursor for the next call.
- `more`: True when more changes are ready now.
- `complete`: False when the feed may have missed changes: back it with a listing.
- `live_watch`: True when the backend's own running process produces the feed (a
  file-system watcher), so stopping it loses changes; durable feeds
  leave it false.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

## Enums

### WriteCondition

```rust
pub enum WriteCondition {
    IfMatch(String),
    IfNoneMatch,
}
```

A precondition on a write.

**Variants**

- `IfMatch`: Write only while the stored object's etag equals this value.
- `IfNoneMatch`: Write only when no object exists at the key.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### fallback_move

```rust
pub async fn fallback_move<B: StorageBackend + ?Sized>(
    backend: &B,
    from: &str,
    to: &str,
    options: &MoveOptions,
) -> Result<MoveResult>
```

Moves one object with the required primitives: probe, copy (server-side
when the backend supports it, else streamed), check that the copy has the
source's size, then delete the source.

A repeated move whose source is gone and whose target exists succeeds
with `already_moved`.

Concurrent writers are handled as far as the primitives allow:

- With `no_overwrite` on a backend that enforces conditional writes, the
  target is written with `IfNoneMatch`, so an object created at the
  target after the probe is never replaced. Elsewhere the check runs
  just before the copy, and the result's `atomic: false` says so.
- The source is probed again after the copy and deleted only when it is
  provably still the object that was copied (same etag, else same size
  and modification time). A source replaced during the copy, or one whose
  probes cannot prove it, is kept and the move fails with `Conflict`. The
  ABI has no conditional delete, so a rewrite between that probe and the
  delete can still be lost.

#### Errors

Returns [`StorageError::NotFound`](README.md#storageerror) when neither key exists,
[`StorageError::Conflict`](README.md#storageerror) when a precondition in `options` fails or the
source changed during the copy, and backend errors otherwise.

### fallback_move_many

```rust
pub async fn fallback_move_many<B: StorageBackend + ?Sized>(
    backend: &B,
    pairs: &[(String, String)],
    options: &MoveOptions,
) -> Vec<KeyResult>
```

Moves each pair in turn with [`StorageBackend::move_object`](README.md#storagebackendmove_object). One pair's
failure does not stop the others.

### require_disjoint_prefixes

```rust
pub fn require_disjoint_prefixes(from_prefix: &str, to_prefix: &str) -> Result<()>
```

Rejects a prefix move whose namespaces overlap.

A target inside the source would move objects it just moved; a source
inside the target lets one moved object overwrite another that has not
moved yet (`a/b/` to `a/` sends `a/b/b/x` onto `a/b/x`). Prefixes are
compared after dropping `.` and empty path segments, with `\` read as
`/`, so `./a/sub/`, `a//sub/` and `a\sub/` count as `a/sub/`.

#### Errors

Returns [`StorageError::InvalidKey`](README.md#storageerror) for an empty source prefix or
overlapping prefixes.

### fallback_move_prefix

```rust
pub async fn fallback_move_prefix<B: StorageBackend + ?Sized>(
    backend: &B,
    from_prefix: &str,
    to_prefix: &str,
    cursor: Option<&str>,
    limit: u32,
    options: &MoveOptions,
) -> Result<MovePrefixPage>
```

Moves one listing page under `from_prefix` to `to_prefix`.

The cursor is the backend's own listing cursor. Moved objects leave the
source prefix, so a backend whose listing cursor does not survive deletes
answers [`StorageError::InvalidCursor`](README.md#storageerror); the caller then starts over with
`None`, which is safe because everything already moved is gone.

#### Errors

Returns [`StorageError::InvalidKey`](README.md#storageerror) for overlapping prefixes and listing
errors; per-object failures are in the page.

### fallback_delete_many

```rust
pub async fn fallback_delete_many<B: StorageBackend + ?Sized>(
    backend: &B,
    keys: &[String],
) -> Vec<KeyResult>
```

Deletes each key in turn. One key's failure does not stop the others.

### fallback_head_many

```rust
pub async fn fallback_head_many<B: StorageBackend + ?Sized>(
    backend: &B,
    keys: &[String],
) -> Result<Vec<Option<ObjectProbe>>>
```

Probes each key in turn.

#### Errors

Returns the first probe failure other than absence.

### fallback_put_conditional

```rust
pub async fn fallback_put_conditional<B: StorageBackend + ?Sized>(
    backend: &B,
    key: &str,
    body: Pin<Box<dyn AsyncRead + Send>>,
    meta: ObjectMeta,
    condition: &WriteCondition,
) -> Result<ConditionalPutResult>
```

Checks `condition` with a probe, then writes. Not atomic: the result says
so.

#### Errors

Returns [`StorageError::Conflict`](README.md#storageerror) when the condition fails, and backend
errors otherwise.

### fallback_list_hinted

```rust
pub async fn fallback_list_hinted<B: StorageBackend + ?Sized>(
    backend: &B,
    request: &ListRequest,
) -> Result<HintedListPage>
```

Lists one page and reports that no hint was honoured: no signatures
beyond what the plain listing carries, no time filter.

#### Errors

Returns listing errors.
