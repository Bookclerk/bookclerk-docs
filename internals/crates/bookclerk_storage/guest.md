# bookclerk_storage::guest

Serving a backend's intent operations from a storage guest.
Serves a [`StorageBackend`](README.md#storagebackend)'s intent operations through a storage guest's
`Destination` capability.

The `local` and `s3` destination plugins each wrap one backend. The
helpers here turn the plugin ABI's requests into backend calls and the
backend's results into ABI replies, so both guests answer the intent
operations the same way and declare exactly the ones their backend does
natively (see `docs/plugins/abi.md#intent-operations`).

| Item | Kind | Summary |
| --- | --- | --- |
| [`plugin_error`](#plugin_error) | function | Maps a storage error onto the plugin error a guest answers with. |
| [`storage_features`](#storage_features) | function | The `storage.*` features to accept in `describe` for a backend with `capabilities`: one per operation the backend does natively. |
| [`object_metadata`](#object_metadata) | function | The ABI metadata for a backend probe. |
| [`write_condition`](#write_condition) | function | The write precondition in `options`, if any. |
| [`list`](#list) | function | One listing page, honouring the hints the backend supports. |
| [`move_object`](#move_object) | function | Moves one object. |
| [`move_many`](#move_many) | function | Moves explicit pairs, or one page of a prefix. |
| [`delete_many`](#delete_many) | function | Deletes up to `MAX_BATCH_KEYS` keys, one outcome per key in order. |
| [`head_many`](#head_many) | function | Probes up to `MAX_BATCH_KEYS` keys, one entry per key in order. |
| [`translate_keys`](#translate_keys) | function | Translates up to `MAX_BATCH_KEYS` host keys into the keys the backend stores them under, one per key in order. |
| [`changes`](#changes) | function | Serves `changes` from the backend's change feed. |

## Functions

### plugin_error

```rust
pub fn plugin_error(err: StorageError) -> PluginError
```

Maps a storage error onto the plugin error a guest answers with.

A failed precondition is `conflict`, a malformed key `invalidParams`, and
a cursor the backend no longer accepts `invalidCursor`, so the host can
tell them apart from backend failures.

### storage_features

```rust
pub fn storage_features(capabilities: StorageCapabilities) -> Vec<String>
```

The `storage.*` features to accept in `describe` for a backend with
`capabilities`: one per operation the backend does natively.

### object_metadata

```rust
pub fn object_metadata(probe: ObjectProbe) -> ObjectMetadata
```

The ABI metadata for a backend probe.

### write_condition

```rust
pub fn write_condition(
    options: &WriteOptions,
) -> Result<Option<WriteCondition>, PluginError>
```

The write precondition in `options`, if any.

#### Errors

Returns `PluginError::invalid_params` when both `ifMatch` and
`ifNoneMatch` are set, or when a precondition is combined with
`stageOnly` (a staged object is published by `commit`, which carries no
precondition).

### list

```rust
pub async fn list<B: StorageBackend + ?Sized>(
    backend: &B,
    options: ListOptions,
) -> Result<ListPage, PluginError>
```

One listing page, honouring the hints the backend supports.

#### Errors

Returns the backend's listing error as a plugin error.

### move_object

```rust
pub async fn move_object<B: StorageBackend + ?Sized>(
    backend: &B,
    from: &str,
    to: &str,
    options: MoveOptions,
) -> Result<MoveResult, PluginError>
```

Moves one object.

#### Errors

Returns the backend's error as a plugin error (`conflict` for a failed
precondition).

### move_many

```rust
pub async fn move_many<B: StorageBackend + ?Sized>(
    backend: &B,
    request: MoveManyRequest,
) -> Result<MoveManyPage, PluginError>
```

Moves explicit pairs, or one page of a prefix.

Pairs ignore `ifSourceMatch`, which names one object's etag, the same as
the host's fallback does.

#### Errors

Returns `PluginError::invalid_params` for more than
[`MAX_BATCH_KEYS`](../bookclerk_plugin_abi/README.md#max_batch_keys) pairs, and prefix errors (overlapping prefixes, a
stale cursor, listing failures) as plugin errors. Per-object failures
are in the page.

### delete_many

```rust
pub async fn delete_many<B: StorageBackend + ?Sized>(
    backend: &B,
    keys: Vec<String>,
) -> Result<Vec<KeyOutcome>, PluginError>
```

Deletes up to [`MAX_BATCH_KEYS`](../bookclerk_plugin_abi/README.md#max_batch_keys) keys, one outcome per key in order.

#### Errors

Returns `PluginError::invalid_params` for too many keys, and a backend
error that failed the whole batch.

### head_many

```rust
pub async fn head_many<B: StorageBackend + ?Sized>(
    backend: &B,
    keys: Vec<String>,
) -> Result<Vec<Option<ObjectMetadata>>, PluginError>
```

Probes up to [`MAX_BATCH_KEYS`](../bookclerk_plugin_abi/README.md#max_batch_keys) keys, one entry per key in order.

#### Errors

Returns `PluginError::invalid_params` for too many keys, and the first
probe failure other than absence.

### translate_keys

```rust
pub async fn translate_keys<B: StorageBackend + ?Sized>(
    backend: &B,
    keys: Vec<String>,
) -> Result<Vec<String>, PluginError>
```

Translates up to [`MAX_BATCH_KEYS`](../bookclerk_plugin_abi/README.md#max_batch_keys) host keys into the keys the backend
stores them under, one per key in order.

#### Errors

Returns `PluginError::invalid_params` for too many keys or a key that
breaks the host key syntax ([`crate::key_syntax::HostKey`](key_syntax.md#hostkey)), and the
backend's failure.

### changes

```rust
pub async fn changes<B: StorageBackend + ?Sized>(
    backend: &B,
    request: ChangesRequest,
) -> Result<ChangesPage, PluginError>
```

Serves `changes` from the backend's change feed.

A backend without a feed (for the `local` plugin, an instance with
`watch` off) answers with an empty page that says changes may have been
missed, so the host lists instead.

#### Errors

Returns `invalidCursor` for a cursor the feed does not accept, and the
feed's other failures.
