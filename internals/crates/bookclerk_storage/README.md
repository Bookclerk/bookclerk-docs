# bookclerk_storage

Storage backends for acquired audiobooks, covers, and PDFs.

## Audience

Host acquire / library code that writes through [`StorageBackend`](#storagebackend). Guest
plugins do not depend on this crate.

Product narrative: destination sections in `docs/server/configuration.md`. Style:
`docs/contributing/code-documentation.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`StorageBackend`](#storagebackend) | trait | Pluggable storage for acquired audio and sidecar files. |
| [`FanoutBackend`](#fanoutbackend) | struct | Multiplexes one logical storage key across multiple backends. |
| [`LocalFsBackend`](#localfsbackend) | struct | Stores objects under a root directory; keys map to relative paths. |
| [`S3Backend`](#s3backend) | struct | S3-compatible object storage. |
| [`S3Credentials`](#s3credentials) | struct | Static AWS-style credentials for the S3 destination. |
| [`S3Settings`](#s3settings) | struct | Connection settings for one S3 / MinIO destination. |
| [`ByteRange`](#byterange) | struct | Inclusive byte range for a streamed read. |
| [`ListPage`](#listpage) | struct | One page of `ObjectInfo` from `StorageBackend::list_page`. |
| [`ObjectInfo`](#objectinfo) | struct | Listing entry returned by `StorageBackend::list`. |
| [`ObjectMeta`](#objectmeta) | struct | Metadata attached to a stored object. |
| [`ObjectProbe`](#objectprobe) | struct | Cheap object probe (S3 `HeadObject` / local sidecar meta) — never downloads object bodies. |
| [`PutStreamResult`](#putstreamresult) | struct | Result of `StorageBackend::put_stream`. |
| [`TransferOptions`](#transferoptions) | struct | Knobs for `transfer_object`. |
| [`StorageError`](#storageerror) | enum | Errors from local / S3 / fan-out storage backends. |
| [`TransferOutcome`](#transferoutcome) | enum | How `transfer_object` moved the object. |
| [`ensure_scalar_len`](#ensure_scalar_len) | function | Rejects a scalar put whose buffer already exceeds `limit`. |
| [`normalize_range`](#normalize_range) | function | Normalizes an internal range. |
| [`parse_sha256_hex`](#parse_sha256_hex) | function | Parses a hex SHA-256 (64 hex chars) into raw bytes. |
| [`read_scalar_body`](#read_scalar_body) | function | Reads at most `limit + 1` bytes. |
| [`reject_scalar_hint`](#reject_scalar_hint) | function | Early-reject when a HEAD/stat hint already exceeds `limit`. |
| [`sha256_field_from_raw`](#sha256_field_from_raw) | function | Validates a raw SHA-256 field (ABI `Data`, 32 bytes) into hex. |
| [`node_placement_id`](#node_placement_id) | function | This node's id, or a process-lifetime id when the node identity cannot be read. |
| [`bookclerk_meta_sidecar_key`](#bookclerk_meta_sidecar_key) | function | Sidecar key for probe metadata of this exact object (`{key}.bookclerk-meta.json`). |
| [`is_audio_key`](#is_audio_key) | function | True when `key` ends with a known acquired audio extension. |
| [`transfer_object`](#transfer_object) | function | Copies `source_key` onto `dest_key`. |
| [`Result`](#result) | type alias | Result alias for storage backend operations in this crate. |
| [`MAX_LIST_PAGE`](#max_list_page) | constant | ABI list page ceiling. |
| [`MAX_SCALAR_OBJECT_BYTES`](#max_scalar_object_bytes) | constant | ABI scalar ceiling. Audiobook bodies must not use `crate::StorageBackend::get` / `crate::StorageBackend::put`. |
| [`MAX_SUPPORTED_OBJECT_BYTES`](#max_supported_object_bytes) | constant | Largest object this process will upload or server-copy. |
| [`AUDIO_EXTENSIONS`](#audio_extensions) | constant | Audio extensions considered acquired media for storage matching. |

## Re-exports

- [`intent::ChangeFeedPage`](intent.md#changefeedpage)
- [`intent::ConditionalPutResult`](intent.md#conditionalputresult)
- [`intent::HintedListPage`](intent.md#hintedlistpage)
- [`intent::KeyResult`](intent.md#keyresult)
- [`intent::ListRequest`](intent.md#listrequest)
- [`intent::MoveOptions`](../bookclerk_plugin_abi/README.md#moveoptions)
- [`intent::MovePrefixPage`](intent.md#moveprefixpage)
- [`intent::MoveResult`](../bookclerk_plugin_abi/README.md#moveresult)
- [`intent::ObjectChange`](intent.md#objectchange)
- [`intent::ObjectChangeKind`](../bookclerk_plugin_abi/README.md#objectchangekind)
- [`intent::StorageCapabilities`](intent.md#storagecapabilities)
- [`intent::WriteCondition`](intent.md#writecondition)

## Modules

- [`fs_kind`](fs_kind.md): What kind of file system a local path lives on, so the local backend only leans on hard links, inode numbers and atomic renames where the file system keeps their promises.
- [`guest`](guest.md): Serving a backend's intent operations from a storage guest. Serves a `StorageBackend`'s intent operations through a storage guest's `Destination` capability.
- [`intent`](intent.md): Intent operations (moves, batches, conditional writes, change feeds) and their fallbacks. Intent-based storage operations and the host's fallback for each.

## Traits

### StorageBackend

```rust
pub trait StorageBackend: Send + Sync {
    fn name(&self) -> &'static str;
    fn instance_id(&self) -> String;
    fn scan_placement(&self) -> Option<String>;
    fn clone_box(&self) -> Box<dyn StorageBackend>;
    fn put<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
        data: Bytes,
        meta: ObjectMeta,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn put_file<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
        path: &'life2 Path,
        meta: ObjectMeta,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn get<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<Bytes>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn exists<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<bool>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn list<'life0, 'life1, 'async_trait>(
        &'life0 self,
        prefix: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<ObjectInfo>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn list_audio<'life0, 'life1, 'async_trait>(
        &'life0 self,
        prefix: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<ObjectInfo>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn probe<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<ObjectProbe>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn copy<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        from: &'life1 str,
        to: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn rename<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        from: &'life1 str,
        to: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn capabilities(&self) -> StorageCapabilities;
    fn move_object<'life0, 'life1, 'life2, 'life3, 'async_trait>(
        &'life0 self,
        from: &'life1 str,
        to: &'life2 str,
        options: &'life3 MoveOptions,
    ) -> Pin<Box<dyn Future<Output = Result<MoveResult>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,;
    fn move_many<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        pairs: &'life1 [(String, String)],
        options: &'life2 MoveOptions,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<KeyResult>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn move_prefix<'life0, 'life1, 'life2, 'life3, 'life4, 'async_trait>(
        &'life0 self,
        from_prefix: &'life1 str,
        to_prefix: &'life2 str,
        cursor: Option<&'life3 str>,
        limit: u32,
        options: &'life4 MoveOptions,
    ) -> Pin<Box<dyn Future<Output = Result<MovePrefixPage>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,
        'life4: 'async_trait,;
    fn delete_many<'life0, 'life1, 'async_trait>(
        &'life0 self,
        keys: &'life1 [String],
    ) -> Pin<Box<dyn Future<Output = Result<Vec<KeyResult>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn head_many<'life0, 'life1, 'async_trait>(
        &'life0 self,
        keys: &'life1 [String],
    ) -> Pin<Box<dyn Future<Output = Result<Vec<Option<ObjectProbe>>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn put_stream_conditional<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
        body: Pin<Box<dyn AsyncRead + Send>>,
        meta: ObjectMeta,
        condition: &'life2 WriteCondition,
    ) -> Pin<Box<dyn Future<Output = Result<ConditionalPutResult>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn list_hinted<'life0, 'life1, 'async_trait>(
        &'life0 self,
        request: &'life1 ListRequest,
    ) -> Pin<Box<dyn Future<Output = Result<HintedListPage>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn changes<'life0, 'life1, 'async_trait>(
        &'life0 self,
        cursor: Option<&'life1 str>,
        limit: u32,
    ) -> Pin<Box<dyn Future<Output = Result<Option<ChangeFeedPage>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn delete<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn head<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<Option<ObjectProbe>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn list_page<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        prefix: &'life1 str,
        cursor: Option<&'life2 str>,
        limit: u32,
    ) -> Pin<Box<dyn Future<Output = Result<ListPage>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn get_stream<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
        range: Option<ByteRange>,
    ) -> Pin<Box<dyn Future<Output = Result<(ObjectProbe, Pin<Box<dyn AsyncRead + Send>>)>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn put_stream<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
        body: Pin<Box<dyn AsyncRead + Send>>,
        meta: ObjectMeta,
    ) -> Pin<Box<dyn Future<Output = Result<PutStreamResult>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn supports_server_copy(&self) -> bool;
    fn touch_file<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
        created: Option<SystemTime>,
        modified: Option<SystemTime>,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
}
```

Pluggable storage for acquired audio and sidecar files.

#### StorageBackend::name

```rust
fn name(&self) -> &'static str
```

Backend name for logs (`local`, `s3`).

#### StorageBackend::instance_id

```rust
fn instance_id(&self) -> String
```

Stable identity of this authorized namespace (root, bucket, prefix, session).

[`Self::supports_server_copy`](#storagebackendsupports_server_copy) plus equal ids is the only signal that
[`Self::copy`](#storagebackendcopy) stays inside one backend. Comparing [`Self::name`](#storagebackendname) is not
enough.

#### StorageBackend::scan_placement

```rust
fn scan_placement(&self) -> Option<String>
```

Host placement when this backend's bytes are not portable across nodes.

`None` means a scan may resume on any host with the same
[`Self::instance_id`](#storagebackendinstance_id) (object storage). `Some` is a stable host id.
Wrappers and fan-out must forward a child's placement; they must not
guess locality from an `instance_id` prefix. When placement cannot be
proved, return a host id so the scan restarts on another node instead
of adopting the wrong inventory.

#### StorageBackend::clone_box

```rust
fn clone_box(&self) -> Box<dyn StorageBackend>
```

Clone into a new boxed backend (same client / root).

#### StorageBackend::put

```rust
fn put<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
    data: Bytes,
    meta: ObjectMeta,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Write bytes under `key`.

#### StorageBackend::put_file

```rust
fn put_file<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
    path: &'life2 Path,
    meta: ObjectMeta,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Stream a local file into storage.

The default opens the file and calls [`Self::put_stream`](#storagebackendput_stream). It does not
read the body into a `Bytes` buffer. Backends may override with a
native copy that still avoids a userspace buffer of the whole object.

##### Errors

Returns I/O errors from the file or the backend write.

#### StorageBackend::get

```rust
fn get<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<Bytes>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Download the full object body into memory.

##### Errors

Returns [`crate::StorageError::NotFound`](#storageerror) when missing, and I/O or S3
failures otherwise.

#### StorageBackend::exists

```rust
fn exists<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<bool>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Return whether `key` exists without downloading the body.

##### Errors

Propagates backend probe failures (not merely absence).

#### StorageBackend::list

```rust
fn list<'life0, 'life1, 'async_trait>(
    &'life0 self,
    prefix: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<Vec<ObjectInfo>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Compatibility collector for a namespace that fits in one page.

Returns [`StorageError::PayloadTooLarge`](#storageerror) when another page exists so a
product caller cannot rebuild a full inventory by accident. Scans use
[`Self::list_page`](#storagebackendlist_page).

##### Errors

Returns [`StorageError::PayloadTooLarge`](#storageerror) when the prefix holds more
than one page, plus listing errors from [`Self::list_page`](#storagebackendlist_page).

#### StorageBackend::list_audio

```rust
fn list_audio<'life0, 'life1, 'async_trait>(
    &'life0 self,
    prefix: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<Vec<ObjectInfo>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Audio objects under `prefix`, capped at one page of matches.

See [`AUDIO_EXTENSIONS`](#audio_extensions). Pages are discarded as they are scanned. More
than [`crate::bounded::MAX_LIST_PAGE`](#max_list_page) audio objects is an error.

##### Errors

Returns [`StorageError::PayloadTooLarge`](#storageerror) when the audio match set
exceeds one page, plus listing errors.

#### StorageBackend::probe

```rust
fn probe<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<ObjectProbe>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Probe object metadata without downloading the body.

S3 uses `HeadObject` (user metadata). Local reads an optional
`.bookclerk-meta.json` sidecar written on put.

#### StorageBackend::copy

```rust
fn copy<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    from: &'life1 str,
    to: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Copy `from` → `to` within the same backend (S3 server-side copy / local
file copy). Preserves object metadata when the backend supports it.

#### StorageBackend::rename

```rust
fn rename<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    from: &'life1 str,
    to: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Move `from` → `to`, replacing any object at `to`.

Forwards to [`Self::move_object`](#storagebackendmove_object), so a backend that renames natively
does it in one step.

#### StorageBackend::capabilities

```rust
fn capabilities(&self) -> StorageCapabilities
```

Which intent operations this backend does natively. The default is
none: every operation runs its fallback from the primitives.

#### StorageBackend::move_object

```rust
fn move_object<'life0, 'life1, 'life2, 'life3, 'async_trait>(
    &'life0 self,
    from: &'life1 str,
    to: &'life2 str,
    options: &'life3 MoveOptions,
) -> Pin<Box<dyn Future<Output = Result<MoveResult>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
```

Move one object. The default is [`crate::intent::fallback_move`](intent.md#fallback_move)
(probe, copy, delete).

##### Errors

Returns [`StorageError::NotFound`](#storageerror) when neither key exists,
[`StorageError::Conflict`](#storageerror) when a precondition fails, and backend
errors otherwise.

#### StorageBackend::move_many

```rust
fn move_many<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    pairs: &'life1 [(String, String)],
    options: &'life2 MoveOptions,
) -> Pin<Box<dyn Future<Output = Result<Vec<KeyResult>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Move each `(from, to)` pair, one outcome per pair in order. The
default loops [`Self::move_object`](#storagebackendmove_object).

##### Errors

Returns an error only when the whole call fails; per-pair failures
are in the outcomes.

#### StorageBackend::move_prefix

```rust
fn move_prefix<'life0, 'life1, 'life2, 'life3, 'life4, 'async_trait>(
    &'life0 self,
    from_prefix: &'life1 str,
    to_prefix: &'life2 str,
    cursor: Option<&'life3 str>,
    limit: u32,
    options: &'life4 MoveOptions,
) -> Pin<Box<dyn Future<Output = Result<MovePrefixPage>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
    'life4: 'async_trait,
```

Move one page of the objects under `from_prefix` to `to_prefix`.
The default is [`crate::intent::fallback_move_prefix`](intent.md#fallback_move_prefix).

##### Errors

Returns [`StorageError::InvalidKey`](#storageerror) for overlapping prefixes,
[`StorageError::InvalidCursor`](#storageerror) for a cursor the backend no longer
accepts (start over with `None`), and listing errors.

#### StorageBackend::delete_many

```rust
fn delete_many<'life0, 'life1, 'async_trait>(
    &'life0 self,
    keys: &'life1 [String],
) -> Pin<Box<dyn Future<Output = Result<Vec<KeyResult>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Delete each key, one outcome per key in order. The default loops
[`Self::delete`](#storagebackenddelete).

##### Errors

Returns an error only when the whole call fails.

#### StorageBackend::head_many

```rust
fn head_many<'life0, 'life1, 'async_trait>(
    &'life0 self,
    keys: &'life1 [String],
) -> Pin<Box<dyn Future<Output = Result<Vec<Option<ObjectProbe>>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Probe each key, one entry per key in order. The default loops
[`Self::head`](#storagebackendhead).

##### Errors

Returns the first probe failure other than absence.

#### StorageBackend::put_stream_conditional

```rust
fn put_stream_conditional<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
    body: Pin<Box<dyn AsyncRead + Send>>,
    meta: ObjectMeta,
    condition: &'life2 WriteCondition,
) -> Pin<Box<dyn Future<Output = Result<ConditionalPutResult>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Write `body` under `key` only when `condition` holds. The default
checks first and then writes, and reports that it was not atomic.

##### Errors

Returns [`StorageError::Conflict`](#storageerror) when the condition fails, and
backend errors otherwise.

#### StorageBackend::list_hinted

```rust
fn list_hinted<'life0, 'life1, 'async_trait>(
    &'life0 self,
    request: &'life1 ListRequest,
) -> Pin<Box<dyn Future<Output = Result<HintedListPage>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

One listing page with hints. The default ignores the hints and says
so in the reply.

##### Errors

Returns listing errors.

#### StorageBackend::changes

```rust
fn changes<'life0, 'life1, 'async_trait>(
    &'life0 self,
    cursor: Option<&'life1 str>,
    limit: u32,
) -> Pin<Box<dyn Future<Output = Result<Option<ChangeFeedPage>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Changes since `cursor` from the backend's change feed. `Ok(None)`
means the backend has no feed and the caller relies on listings,
which is the default.

##### Errors

Returns [`StorageError::InvalidCursor`](#storageerror) for an expired cursor (start
over with a full listing) and feed errors.

#### StorageBackend::delete

```rust
fn delete<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Delete an object (no-op if missing).

#### StorageBackend::head

```rust
fn head<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<Option<ObjectProbe>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Probe metadata without downloading the body.

Default maps [`Self::probe`](#storagebackendprobe) absence onto `Ok(None)`.

##### Errors

Propagates backend probe failures other than not-found.

#### StorageBackend::list_page

```rust
fn list_page<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    prefix: &'life1 str,
    cursor: Option<&'life2 str>,
    limit: u32,
) -> Pin<Box<dyn Future<Output = Result<ListPage>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

One page of keys under `prefix`.

`cursor` is opaque and scoped to this backend's ordering. A missing,
stale, or non-advancing cursor returns [`StorageError::InvalidCursor`](#storageerror)
and does not restart at the first page. `limit` is clamped to
[`crate::bounded::MAX_LIST_PAGE`](#max_list_page). Ordering and mutation behavior are
backend-specific; see `docs/internals/storage-bounds.md`.

##### Errors

Returns [`StorageError::InvalidCursor`](#storageerror) or a backend listing error.

#### StorageBackend::get_stream

```rust
fn get_stream<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
    range: Option<ByteRange>,
) -> Pin<Box<dyn Future<Output = Result<(ObjectProbe, Pin<Box<dyn AsyncRead + Send>>)>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Streamed read. Never reassembles the object into host `Bytes`.

[`ObjectProbe::size`](#objectprobe) is the whole object size, not the returned slice.
`range.length == Some(0)` is rejected. `None` means through EOF, matching
the wire encoding where length `0` is "to end".

##### Errors

Returns [`StorageError::NotFound`](#storageerror) when missing, [`StorageError::InvalidKey`](#storageerror)
for a bad range, and I/O or S3 failures otherwise.

#### StorageBackend::put_stream

```rust
fn put_stream<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
    body: Pin<Box<dyn AsyncRead + Send>>,
    meta: ObjectMeta,
) -> Pin<Box<dyn Future<Output = Result<PutStreamResult>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Streamed write. `body` ownership is transferred to the backend.

##### Errors

Returns I/O or S3 failures from the sink.

#### StorageBackend::supports_server_copy

```rust
fn supports_server_copy(&self) -> bool
```

True when [`Self::copy`](#storagebackendcopy) stays inside this namespace without downloading.

The default is false. A backend opts in only for copies that are valid
for its own bucket, root, and prefix.

#### StorageBackend::touch_file

```rust
fn touch_file<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
    created: Option<SystemTime>,
    modified: Option<SystemTime>,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Set filesystem timestamps (local) or best-effort logical timestamp tags (S3).

Local backends update mtime/ctime. S3 backends must **not** CopyObject to
rewrite user-metadata (creates a second full-size version on versioned
buckets). System `Last-Modified` cannot be set on AWS S3; logical times
belong in PutObject `x-amz-meta-*` (S3 backends set them at upload only).

Implemented by: `FanoutBackend`, `LocalFsBackend`, `S3Backend`

## Structs

### FanoutBackend

```rust
pub struct FanoutBackend {
    /* private fields */
}
```

Multiplexes one logical storage key across multiple backends.

Mutations (`put*`, `copy`, `rename`, `delete`, `touch_file`) run on every
child. Reads try children in order and succeed on the first hit.

#### FanoutBackend::new

```rust
pub fn new(backends: Vec<Box<dyn StorageBackend>>) -> Result<Self>
```

Build a fan-out over `backends` (must be non-empty).

##### Errors

Returns an error when the operation fails.

Implements: `StorageBackend`

### LocalFsBackend

```rust
pub struct LocalFsBackend {
    /* private fields */
}
```

Stores objects under a root directory; keys map to relative paths.

An optional key prefix is prepended to every key (same model as S3),
so library `storage_key` values stay relative to the prefix.

#### LocalFsBackend::CAPABILITIES

```rust
pub const CAPABILITIES: StorageCapabilities = _
```

What every local backend does natively: moves are renames, batches
run in one call, and conditional writes are checked here. A backend
built with [`Self::with_watch`](#localfsbackendwith_watch) (feature `watch`) also has a change
feed.

#### LocalFsBackend::new

```rust
pub fn new(root: PathBuf) -> Result<Self>
```

Create a backend rooted at `root` with no key prefix.

##### Errors

Returns an error when the operation fails.

#### LocalFsBackend::with_prefix

```rust
pub fn with_prefix(root: PathBuf, prefix: &str) -> Result<Self>
```

Create a backend rooted at `root` with an optional key prefix
(e.g. `library/`). The prefix directory is created when needed.

##### Errors

Returns an error when the operation fails.

#### LocalFsBackend::with_watch

```rust
pub fn with_watch(self) -> Self
```

Starts a change feed for this backend's root (below its prefix),
served by [`StorageBackend::changes`](#storagebackendchanges) for as long as the backend or
a clone of it lives.

The watcher runs only on a local disk; on a network file system or a
FUSE mount (see [`crate::fs_kind`](fs_kind.md)), or when the operating system
refuses it, every page says the feed may have missed changes, so the
caller lists instead. Call it after [`Self::with_fs_kind`](#localfsbackendwith_fs_kind) for the
override to apply.

#### LocalFsBackend::with_fs_kind

```rust
pub fn with_fs_kind(self, kind: FsKind) -> Self
```

Treats every path as living on a file system of `kind`, instead of
classifying it.

Tests use it to drive the network-mount paths on a local disk.
Production backends leave it unset.

#### LocalFsBackend::with_scan_placement

```rust
pub fn with_scan_placement(self, placement: impl Into<String>) -> Self
```

Overrides the host placement reported for scan adoption.

Production backends leave this unset and use [`crate::node_placement_id`](#node_placement_id).

Implements: `Clone`, `Debug`, `StorageBackend`

### S3Backend

```rust
pub struct S3Backend {
    /* private fields */
}
```

S3-compatible object storage.

#### S3Backend::from_parts_with_journal

```rust
pub async fn from_parts_with_journal(
    cfg: &S3Settings,
    prefix: &str,
    creds: Option<&S3Credentials>,
    journal_dir: &Path,
) -> Result<Self>
```

Build with explicit credentials and a granted multipart journal directory.

The directory must already be a location the process is allowed to
persist. For the destination guest that is its `HOME`, not the host
files directory. A journal that cannot be created or read fails
construction instead of claiming later recovery.

##### Errors

Returns an error when the client cannot be built, the journal directory
cannot be created, or reading existing records fails.

#### S3Backend::with_part_size

```rust
pub fn with_part_size(self, part_size: usize) -> Self
```

Overrides the multipart part size. Values below 1 are ignored.

#### S3Backend::from_parts

```rust
pub async fn from_parts(
    cfg: &S3Settings,
    prefix: &str,
    creds: Option<&S3Credentials>,
) -> Result<Self>
```

Build with explicit settings and credentials.

Without [`Self::from_parts_with_journal`](#s3backendfrom_parts_with_journal), multipart uploads are not
recoverable after process death.

##### Errors

Returns an error when the operation fails.

#### S3Backend::CAPABILITIES

```rust
pub const CAPABILITIES: StorageCapabilities = _
```

What every S3 backend can do natively once the store has shown it
enforces conditions; each operation falls back to the host's
algorithm on a store that does not (see [`Self::conditional_support`](#s3backendconditional_support)).

#### S3Backend::conditional_support

```rust
pub async fn conditional_support(&self) -> ConditionalSupport
```

Which conditional requests this store enforces, probed on first use
and remembered for the backend's lifetime.

Each request kind is probed on its own, because a store can enforce a
condition on one and ignore it on another. Under `.bookclerk-probe/`
the probe writes a scratch object with `If-None-Match: *` twice
(the second must be refused) and overwrites it with a wrong
`If-Match` (refused); completes a one-part multipart upload onto it
with each condition (both refused) and aborts the upload; copies it
onto a second existing object with a wrong
`x-amz-copy-source-if-match` (refused) and with `If-None-Match: *`
(refused); copies it as an upload part with a wrong
`x-amz-copy-source-if-match` (refused) and aborts that upload; and
deletes it with a wrong `If-Match` (refused). It then removes both
objects. Any other answer, including an error, counts as "not
enforced", so a store is never trusted by default.

#### S3Backend::retry_orphans

```rust
pub async fn retry_orphans(&self) -> Result<usize>
```

Aborts multipart uploads recorded for this endpoint, bucket, and prefix
whose owner lock is not held.

Live uploads are left alone. Failed aborts stay on disk and stop being
retried after 8 failed attempts.

##### Errors

Returns I/O errors reading the journal directory.

Implements: `Clone`, `Debug`, `StorageBackend`

### S3Credentials

```rust
pub struct S3Credentials {
    pub access_key_id: String,
    pub secret_access_key: String,
    pub session_token: Option<String>,
}
```

Static AWS-style credentials for the S3 destination.

These come from the instance's sealed secrets; without them the SDK
default provider chain applies inside the guest.

**Fields**

- `access_key_id`: AWS access key id.
- `secret_access_key`: AWS secret access key (redacted in [`Debug`](https://doc.rust-lang.org/stable/core/fmt/macros/derive.Debug.html); never logged).
- `session_token`: Optional STS session token when using temporary credentials.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### S3Settings

```rust
pub struct S3Settings {
    pub bucket: String,
    pub region: String,
    pub endpoint: Option<String>,
    pub force_path_style: bool,
}
```

Connection settings for one S3 / MinIO destination.

The `s3` destination plugin declares these as `[[settings]]`; the host
resolves them from the plugin instance document and hands them to the
guest as its open context.

**Fields**

- `bucket`: Destination bucket name (required).
- `region`: AWS region id (also used by MinIO path-style clients).
- `endpoint`: Optional custom endpoint (MinIO, LocalStack, …).
- `force_path_style`: Force path-style addressing (typical for MinIO).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### ByteRange

```rust
pub struct ByteRange {
    pub offset: u64,
    pub length: Option<u64>,
}
```

Inclusive byte range for a streamed read.

**Fields**

- `offset`: Starting offset.
- `length`: Number of bytes; `None` means to end of object.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ListPage

```rust
pub struct ListPage {
    pub objects: Vec<ObjectInfo>,
    pub next_cursor: Option<String>,
}
```

One page of [`ObjectInfo`](#objectinfo) from [`StorageBackend::list_page`](#storagebackendlist_page).

**Fields**

- `objects`: Objects in this page.
- `next_cursor`: Continuation token; `None` when this is the last page.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### ObjectInfo

```rust
pub struct ObjectInfo {
    pub key: String,
    pub size: u64,
    pub etag: Option<String>,
    pub modified_at_ms: Option<u64>,
    pub version_id: Option<String>,
    pub signature: Option<String>,
}
```

Listing entry returned by [`StorageBackend::list`](#storagebackendlist).

The signature fields are filled only by a listing asked for them
([`crate::ListRequest::with_signatures`](intent.md#listrequest)) on a backend that has them.

**Fields**

- `key`: Relative storage key (prefix + path; no leading slash).
- `size`: Object size in bytes.
- `etag`: Backend entity tag. Opaque; not a content checksum.
- `modified_at_ms`: Last modification time in UTC unix milliseconds, when known.
- `version_id`: Backend object version id (versioned buckets), when there is one.
- `signature`: Opaque change signature: an unchanged value means the object did not
  change. Compare only values from the same backend.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ObjectMeta

```rust
pub struct ObjectMeta {
    pub content_type: Option<String>,
    pub content_length: Option<u64>,
    pub asin: Option<String>,
    pub title: Option<String>,
    pub creation_time: Option<String>,
    pub last_write_time: Option<String>,
    pub sha256_hex: Option<String>,
    pub commit_token: Option<String>,
}
```

Metadata attached to a stored object.

**Fields**

- `content_type`: MIME type stored with the object (e.g. `audio/mp4`).
- `content_length`: Object size in bytes when known at put time.
- `asin`: Free-form ASIN / title tags for S3 object metadata.
- `title`: Display title stored as object user-metadata when supported.
- `creation_time`: Creation timestamp as RFC 3339 (S3 metadata `creation-time`).
- `last_write_time`: Last-write timestamp as RFC 3339 (S3 metadata `last-write-time`).
- `sha256_hex`: Lowercase hex SHA-256 (64 chars) of the object body when known.
  
  This is a content digest. It is never derived from an S3 ETag.
- `commit_token`: Retry-stable publication token stored with the object, when staged.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### ObjectProbe

```rust
pub struct ObjectProbe {
    pub key: String,
    pub size: u64,
    pub content_type: Option<String>,
    pub meta: ObjectMeta,
    pub etag: Option<String>,
    pub modified_at_ms: Option<u64>,
    pub version_id: Option<String>,
    pub signature: Option<String>,
}
```

Cheap object probe (S3 `HeadObject` / local sidecar meta) — never downloads
object bodies.

**Fields**

- `key`: Relative storage key that was probed.
- `size`: Object size in bytes from HeadObject / local metadata.
- `content_type`: MIME type when the backend exposes it.
- `meta`: User-metadata / sidecar fields (ASIN, title, timestamps, …).
- `etag`: Backend entity tag. Opaque; not a content checksum.
- `modified_at_ms`: Last modification time in UTC unix milliseconds, when known.
- `version_id`: Backend object version id (versioned buckets), when there is one.
- `signature`: Opaque change signature, as [`ObjectInfo::signature`](#objectinfo).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### PutStreamResult

```rust
pub struct PutStreamResult {
    pub bytes_written: u64,
    pub etag: Option<String>,
    pub sha256_hex: Option<String>,
}
```

Result of [`StorageBackend::put_stream`](#storagebackendput_stream).

**Fields**

- `bytes_written`: Bytes accepted from the body stream.
- `etag`: Backend etag when available. Opaque; not a content checksum.
- `sha256_hex`: Hex SHA-256 of the bytes accepted, when the backend computed one.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### TransferOptions

```rust
pub struct TransferOptions {
    pub max_attempts: u32,
    pub backoff: Duration,
    pub attempt_timeout: Option<Duration>,
    pub stage_journal_dir: Option<PathBuf>,
}
```

Knobs for [`transfer_object`](#transfer_object).

**Fields**

- `max_attempts`: Total attempts including the first. Zero is treated as one.
- `backoff`: Base backoff between attempts. Attempt `n` waits `backoff * n`.
- `attempt_timeout`: Optional wall-clock budget for a single attempt.
- `stage_journal_dir`: Directory that records attempt-owned stages so a restarted process can
  delete them. `None` still deletes on drop while this process is alive.

Implements: `Clone`, `Debug`, `Default`

## Enums

### StorageError

```rust
pub enum StorageError {
    NotFound(String),
    InvalidKey(String),
    Io(Error),
    S3(String),
    PayloadTooLarge(String),
    Integrity(String),
    Conflict(String),
    InvalidCursor(String),
    Other(Error),
}
```

Errors from local / S3 / fan-out storage backends.

**Variants**

- `NotFound`: Object key was not found on get/probe/copy source.
- `InvalidKey`: Key failed validation (path escape, empty bucket, bad destination config).
- `Io`: Local filesystem I/O failure.
- `S3`: S3 SDK or HTTP failure (operator-facing detail).
- `PayloadTooLarge`: Scalar get/put exceeded the ABI small-object cap.
- `Integrity`: Declared length or SHA-256 did not match the bytes consumed.
- `Conflict`: A precondition failed: the target exists, or the object changed
  since it was read.
- `InvalidCursor`: List cursor is missing, stale, or not from this backend.
- `Other`: Catch-all for wrapped backend failures.

Implements: `Debug`, `Display`, `Error`, `From<Error>`

### TransferOutcome

```rust
pub enum TransferOutcome {
    ServerCopy,
    Streamed(PutStreamResult),
    AlreadyPublished,
}
```

How [`transfer_object`](#transfer_object) moved the object.

**Variants**

- `ServerCopy`: Destination `copy` inside one storage instance.
- `Streamed`: Streamed body. `result.sha256_hex` is the digest of bytes written.
- `AlreadyPublished`: Destination probe already matched the source size and digest.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### ensure_scalar_len

```rust
pub fn ensure_scalar_len(len: usize, limit: u64) -> Result<()>
```

Rejects a scalar put whose buffer already exceeds `limit`.

#### Errors

Returns [`StorageError::PayloadTooLarge`](#storageerror) when `len` is above `limit`.

### normalize_range

```rust
pub fn normalize_range(
    range: Option<ByteRange>,
    object_size: Option<u64>,
) -> Result<Option<ReadSpan>>
```

Normalizes an internal range.

`None` reads the whole object. [`ByteRange::length`](#byterange) `None` reads through
EOF (the Cap'n Proto wire uses integer `0` for that case and decodes it to
`None` before it reaches a backend). `Some(0)` is rejected so local and S3
cannot disagree about an empty window versus "to end".

`object_size`, when known, is the whole object. Offsets past that size, a
window that would extend past it, and `offset + length` overflow are errors.

#### Errors

Returns [`StorageError::InvalidKey`](#storageerror) for an ambiguous, overflowing, or
unsatisfiable range.

### parse_sha256_hex

```rust
pub fn parse_sha256_hex(value: &str) -> Result<[u8; 32]>
```

Parses a hex SHA-256 (64 hex chars) into raw bytes.

#### Errors

Returns [`StorageError::Integrity`](#storageerror) when the shape is wrong.

### read_scalar_body

```rust
pub async fn read_scalar_body(
    reader: Pin<Box<dyn AsyncRead + Send>>,
    limit: u64,
) -> Result<Bytes>
```

Reads at most `limit + 1` bytes.

A body longer than `limit` returns [`StorageError::PayloadTooLarge`](#storageerror) and
stops. The allocation is capped at `limit + 1` regardless of any advertised
content length.

#### Errors

Returns [`StorageError::PayloadTooLarge`](#storageerror) or [`StorageError::Io`](#storageerror).

### reject_scalar_hint

```rust
pub fn reject_scalar_hint(hint: u64, limit: u64) -> Result<()>
```

Early-reject when a HEAD/stat hint already exceeds `limit`.

This does not allocate `hint` bytes. The body must still be counted.

#### Errors

Returns [`StorageError::PayloadTooLarge`](#storageerror) when `hint` is above `limit`.

### sha256_field_from_raw

```rust
pub fn sha256_field_from_raw(bytes: Option<&[u8]>) -> Result<Option<String>>
```

Validates a raw SHA-256 field (ABI `Data`, 32 bytes) into hex.

Empty means "not provided". Any other length is an integrity error.

#### Errors

Returns [`StorageError::Integrity`](#storageerror) when the digest is not 32 bytes.

### node_placement_id

```rust
pub fn node_placement_id() -> String
```

This node's id, or a process-lifetime id when the node identity cannot be
read.

`BOOKCLERK_PLACEMENT_ID` overrides it for tests. Otherwise the value is
the `node_id` in `$BOOKCLERK_FILES_DIR/node-identity.json`, which the
control plane writes when the node first opens the library. When that is
not available the id lasts for this process only, so a scan is not
adopted by another process that cannot prove it is the same node.

### bookclerk_meta_sidecar_key

```rust
pub fn bookclerk_meta_sidecar_key(object_key: &str) -> String
```

Sidecar key for probe metadata of this exact object (`{key}.bookclerk-meta.json`).

The record is not shared with another object that only shares a title stem.
`book.m4b` and `book.jpg` keep separate integrity and commit metadata.

### is_audio_key

```rust
pub fn is_audio_key(key: &str) -> bool
```

True when `key` ends with a known acquired audio extension.

### transfer_object

```rust
pub async fn transfer_object(
    source: &dyn StorageBackend,
    source_key: &str,
    dest: &dyn StorageBackend,
    dest_key: &str,
    meta: ObjectMeta,
    options: &TransferOptions,
) -> Result<TransferOutcome>
```

Copies `source_key` onto `dest_key`.

`meta` supplies content type and any caller-provided digest. A digest on
the source probe is required to match bytes actually written. When the
source probe has no digest, the destination records the digest computed
while streaming. An S3 ETag is an identity hint for "the object changed",
not a content checksum, and is never copied into `sha256_hex`.

#### Errors

Returns [`StorageError::Integrity`](#storageerror) when the source changes between
attempts or a digest/length check fails, [`StorageError::NotFound`](#storageerror) when
the source is missing, and the destination error after retries are exhausted.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, StorageError>;
```

Result alias for storage backend operations in this crate.

## Constants

### MAX_LIST_PAGE

```rust
pub const MAX_LIST_PAGE: u32 = bookclerk_plugin_sdk::MAX_LIST_PAGE;
```

ABI list page ceiling.

### MAX_SCALAR_OBJECT_BYTES

```rust
pub const MAX_SCALAR_OBJECT_BYTES: u64 = _;
```

ABI scalar ceiling. Audiobook bodies must not use [`crate::StorageBackend::get`](#storagebackendget) /
[`crate::StorageBackend::put`](#storagebackendput).

### MAX_SUPPORTED_OBJECT_BYTES

```rust
pub const MAX_SUPPORTED_OBJECT_BYTES: u64 = _;
```

Largest object this process will upload or server-copy.

S3 allows 10,000 parts. Each part is 8 MiB, so the application ceiling is
80 GiB. AWS's own 5 TiB object maximum is wider;
requests above this ceiling fail before any part buffer is filled.

### AUDIO_EXTENSIONS

```rust
pub const AUDIO_EXTENSIONS: &[&str] = _;
```

Audio extensions considered acquired media for storage matching.

Includes default remux/encode outputs (`m4b` / `mp3` / `m4a`) plus plain
passthrough containers Chirp / GraphicAudio may store under noop/`as_is`
output (`flac` / `aac` / `ogg` / `oga`). Keep aligned with
`bookclerk_source::media` sniffing and GraphicAudio ZIP audio filters.
