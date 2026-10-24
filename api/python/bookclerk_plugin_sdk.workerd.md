# bookclerk_plugin_sdk.workerd

Workerd author runtime — `BookclerkEntrypoint` and the named entrypoints.

Python mirror of `@bookclerk/plugin-sdk/workerd` (`embed/bookclerk_plugin.js`)
for `api_version = 3` guests running as Python Workers:

- `from bookclerk_plugin_sdk.workerd import BookclerkEntrypoint, js`

The default export is a module-level `Default` class extending
[`BookclerkEntrypoint`](#bookclerkentrypoint) with optional `event(batch)` / `job(job)`
triggers. Named entrypoints from `plugin.toml` `entrypoints` are module-level
classes named after the wire name (`Storefront`, `Storage`, `RemoteLibrary`,
`DatabaseAdapter`, `Cli`, `Oidc`) extending the matching `*Entrypoint`
base. The launcher's adapter isolate reaches them only through the
`bookclerk*` dispatch methods; authors neither call nor override those.

Inside a Python Workers isolate, `bookclerk-workerd` injects this module under
`bookclerk_plugin_sdk.workerd` — authors do not vendor a relative filepath.
Outside the isolate (authoring / unit tests), lightweight stubs stand in for
`js` / `workers` imports and [`js`](#js) returns plain Python values.

| Name | Kind | Summary |
| --- | --- | --- |
| [`AdapterDatabaseSession`](#adapterdatabasesession) | class | Host ↔ database adapter session (`capabilities` + typed `execute`). |
| [`BookclerkEntrypoint`](#bookclerkentrypoint) | class | Default-export base for `api_version = 3` Python Workers guests. |
| [`CliEntrypoint`](#clientrypoint) | class | `cli` entrypoint (`PluginCli`): `describe()` → `CliSchema`, `invoke(params)` → `CliInvokeResult`. |
| [`DatabaseAdapterEntrypoint`](#databaseadapterentrypoint) | class | `databaseAdapter` entrypoint: opens typed SQL sessions for the host library. |
| [`EventBatch`](#eventbatch) | class | One `event(batch)` delivery (`EventBatch` struct), Workers `MessageBatch`-shaped. |
| [`EventMessage`](#eventmessage) | class | One delivered domain event. |
| [`GuestDatabase`](#guestdatabase) | class | Host-granted SQL transport behind a named `[[databases]]` binding. |
| [`JobController`](#jobcontroller) | class | Everything one job invocation may touch. |
| [`KvNamespace`](#kvnamespace) | class | `[[kv_namespaces]]` binding (`env.<BINDING>`): Workers `KVNamespace`-shaped storage. |
| [`NamedEntrypoint`](#namedentrypoint) | class | Base for named entrypoints. |
| [`OidcEntrypoint`](#oidcentrypoint) | class | `oidc` entrypoint: relying-party client templates and credential verification. |
| [`PluginError`](#pluginerror) | class | SDK-thrown failure. Unknown wire codes stay on `wire_code`. |
| [`QueueBatch`](#queuebatch) | class | One `queue(batch)` delivery (`QueueBatch` struct), Workers `MessageBatch`-shaped. |
| [`QueueBinding`](#queuebinding) | class | `[[queues.producers]]` binding (`env.<BINDING>`): Workers `Queue`-shaped producer. |
| [`QueueMessage`](#queuemessage) | class | One delivered queue message (`QueueMessage` struct), Workers `Message`-shaped. |
| [`RemoteLibraryEntrypoint`](#remotelibraryentrypoint) | class | `remoteLibrary` entrypoint: lifecycle, rescan, listening sync, user polling. |
| [`StorageEntrypoint`](#storageentrypoint) | class | `storage` entrypoint (`Destination` interface): an object store the host writes to. |
| [`StorefrontEntrypoint`](#storefrontentrypoint) | class | `storefront` entrypoint (`ContentSource` interface). |
| [`cli_args`](#cli_args) | function | Turn `CliInvokeParams.args` (`[{name, value}]`) into a `{name: value}` dict. |
| [`decode_extensible_config`](#decode_extensible_config) | function | Decode an `ExtensibleConfig` into the plain value on `env.CONFIG`. |
| [`event_batch_results`](#event_batch_results) | function | Translate recorded outcomes into the wire `EventResult` list. |
| [`granted_databases`](#granted_databases) | function | Build named `DatabaseBinding` objects over `POST /db/execute`. |
| [`job_outcome_for`](#job_outcome_for) | function | Translate a `job()` return / raise into the wire `JobOutcome`. |
| [`js`](#js) | function | Convert a Python value to a JS value for Workers RPC results. |
| [`json_payload`](#json_payload) | function | Wrap a JSON-serializable value as an `application/json` `ExtensibleConfig`. |
| [`py`](#py) | function | Convert a JS value delivered over Workers RPC into plain Python. |
| [`queue_batch_results`](#queue_batch_results) | function | Translate recorded outcomes into the wire `QueueResult` list. |
| [`transport_database`](#transport_database) | function | Wrap an adapter `GrantedDatabaseTransport` stub as a `DatabaseBinding`. |
| [`transport_kv`](#transport_kv) | function | Wrap the adapter's granted KV stub as a `KvNamespace`. |
| [`transport_queue`](#transport_queue) | function | Wrap the adapter's granted queue stub as a `QueueBinding`. |
| [`FEATURE_SCALAR_LIMITS`](#feature_scalar_limits) | constant | Guest honors scalar / stream-window / list-page caps (`rpc.scalarLimits`). |
| [`FEATURE_STORAGE_COPY`](#feature_storage_copy) | constant | Guest implements server-side `Destination.copy` (`storage.copy`). |
| [`FEATURE_STREAMS`](#feature_streams) | constant | Media moves through transferred `ByteRange` / `ByteSource` streams (`rpc.streams`). |
| [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes) | constant | Maximum job / event checkpoint payload size (bytes). |
| [`MAX_LIST_PAGE`](#max_list_page) | constant | Maximum objects in one `Destination.list` page. |
| [`MAX_PLUGIN_MIGRATION_OPS`](#max_plugin_migration_ops) | constant | Maximum already-separated operations in one plugin-owned migration. |
| [`MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](#max_plugin_migration_registration_bytes) | constant | Maximum aggregate UTF-8 bytes of plugin migration ids plus SQL in one `databaseMigrations` registration. |
| [`MAX_PLUGIN_MIGRATION_TOTAL_OPS`](#max_plugin_migration_total_ops) | constant | Maximum already-separated operations across one `databaseMigrations` registration. |
| [`MAX_SCALAR_BYTES`](#max_scalar_bytes) | constant | Maximum decoded size of an ordinary RPC scalar value (not a stream window). |
| [`MAX_STREAM_WINDOW_BYTES`](#max_stream_window_bytes) | constant | Maximum bytes returned by one `ByteSource.pull` (flow-control window). |
| [`PRODUCT_API_VERSION`](#product_api_version) | constant | Product ABI version (`plugin.toml` `api_version` / `describe().apiVersion`). |

## Classes

### AdapterDatabaseSession

```python
class AdapterDatabaseSession
```

Host ↔ database adapter session (`capabilities` + typed `execute`).

The restore / identity methods are optional; the base class raises
`unsupported` for each.

#### AdapterDatabaseSession.capabilities

```python
async def capabilities(self) -> Any
```

Typed SQL-contract advertisement.

#### AdapterDatabaseSession.execute

```python
async def execute(self, _request: ExecuteRequest) -> ExecuteReply
```

Typed atomic batch (`ExecuteRequest` → `ExecuteReply`).

#### AdapterDatabaseSession.close

```python
async def close(self) -> None
```

Close the session.

#### AdapterDatabaseSession.bootstrap

```python
async def bootstrap(self) -> Any
```

Bootstrap-only diagnostic metadata (`DbBootstrap`).

#### AdapterDatabaseSession.exportIdentity

```python
async def exportIdentity(self) -> list[Any]
```

Identity high-water marks for a logical export (`IdentityHighWater` rows).

#### AdapterDatabaseSession.importIdentity

```python
async def importIdentity(self, _rows: list[Any]) -> None
```

Restore identity high-water marks after a logical import.

#### AdapterDatabaseSession.listUserRelations

```python
async def listUserRelations(self) -> list[str]
```

Names of user relations the host may drop before a restore.

#### AdapterDatabaseSession.prepareUnitRestore

```python
async def prepareUnitRestore(self) -> None
```

Prepare the engine for a unit restore.

#### AdapterDatabaseSession.dropUserRelations

```python
async def dropUserRelations(self, _names: list[str]) -> None
```

Drop the named user relations.

#### AdapterDatabaseSession.assertRestoreConstraints

```python
async def assertRestoreConstraints(self) -> None
```

Re-check constraints after a unit restore.

### BookclerkEntrypoint

```python
class BookclerkEntrypoint(WorkerEntrypoint)
```

Default-export base for `api_version = 3` Python Workers guests.

Optional trigger methods: `event(batch)` (for `[[events.consumers]]`)
and `job(job)` (for `[triggers] jobs`); optional `describe()`
(presentation fields beyond `plugin.toml`), `database_migrations(binding)`,
and `shutdown()`. Return plain Python values; the SDK converts them.

The `bookclerk*` methods are the adapter-facing dispatch surface; authors
neither call nor override them.

#### BookclerkEntrypoint.fetch

```python
async def fetch(self, _request=None)
```

Reject HTTP fetch — workerd guests are Workers-RPC only.

**Args**

- `_request`: Incoming HTTP request (unused by the default stub).

**Returns**

A 404 `Response`.

#### BookclerkEntrypoint.bookclerkDescribe

```python
async def bookclerkDescribe(self)
```

Adapter dispatch: the author's optional `describe()` (or `None`).

#### BookclerkEntrypoint.bookclerkEvent

```python
async def bookclerkEvent(self, context=None, wire_batch=None)
```

Adapter dispatch: deliver one `EventBatch` to `event(batch)`.

**Raises**

- `PluginError`: `unsupported` when the author defines no `event`.

#### BookclerkEntrypoint.bookclerkQueue

```python
async def bookclerkQueue(self, context=None, wire_batch=None)
```

Adapter dispatch: deliver one `QueueBatch` to `queue(batch)`.

**Raises**

- `PluginError`: `unsupported` when the author defines no `queue`.

#### BookclerkEntrypoint.bookclerkJob

```python
async def bookclerkJob(self, context=None, invocation=None, granted=None)
```

Adapter dispatch: run one durable job through `job(controller)`.

**Raises**

- `PluginError`: `unsupported` when the author defines no `job`.

#### BookclerkEntrypoint.bookclerkDatabaseMigrations

```python
async def bookclerkDatabaseMigrations(self, binding='')
```

Adapter dispatch: the author's `database_migrations(binding)` (or `[]`).

#### BookclerkEntrypoint.bookclerkShutdown

```python
async def bookclerkShutdown(self)
```

Adapter dispatch: the author's optional `shutdown()`.

### CliEntrypoint

```python
class CliEntrypoint(NamedEntrypoint)
```

`cli` entrypoint (`PluginCli`): `describe()` → `CliSchema`, `invoke(params)` → `CliInvokeResult`.

#### CliEntrypoint.describe

```python
async def describe(self)
```

Guest CLI schema (`{"commands": []}` when the guest has no CLI).

#### CliEntrypoint.invoke

```python
async def invoke(self, _params: abi.CliInvokeParams)
```

Invoke a guest CLI command (use [`cli_args`](#cli_args) / [`json_payload`](#json_payload)).

### DatabaseAdapterEntrypoint

```python
class DatabaseAdapterEntrypoint(NamedEntrypoint)
```

`databaseAdapter` entrypoint: opens typed SQL sessions for the host library.

The adapter isolate drives sessions through two dispatch verbs on
`bookclerkInvoke`: `openSession` returns a fresh session id, and
`session(id, method, *params)` calls `method` on the retained
[`AdapterDatabaseSession`](#adapterdatabasesession); `close` releases the entry.

#### DatabaseAdapterEntrypoint.openSession

```python
async def openSession(self)
```

Open an [`AdapterDatabaseSession`](#adapterdatabasesession).

#### DatabaseAdapterEntrypoint.dropUnit

```python
async def dropUnit(self, _unit_ref: str)
```

Physically drop one provisioned binding unit.

**Args**

- `_unit_ref`: Adapter-native unit identity.

**Returns**

`None` when the unit is dropped.

**Raises**

- `PluginError`: `unsupported` unless a subclass implements the drop.

#### DatabaseAdapterEntrypoint.bookclerkInvoke

```python
async def bookclerkInvoke(self, context=None, method='', *args)
```

Adapter dispatch: `openSession` → session id; `dropUnit`; `session` → call on a retained session.

**Args**

- `context`: Granted bindings for this invocation.
- `method`: `openSession`, `dropUnit`, or `session`.
- `*args`: For `session`: `(id, session_method, *params)`.

**Returns**

The session id (`str`) for `openSession`, else the session
method result converted with [`js`](#js).

**Raises**

- `PluginError`: `invalid_params` for an unknown session id, `unsupported` for a method outside the session surface.

### EventBatch

```python
class EventBatch
```

One `event(batch)` delivery (`EventBatch` struct), Workers `MessageBatch`-shaped.

#### EventBatch.__init__

```python
def __init__(self, events: Any, invocation: dict[str, Any] | None=None) -> None
```

#### EventBatch.type

```python
@property
def type(self) -> str
```

Event type shared by the batch (`""` when mixed or empty).

#### EventBatch.ack_all

```python
def ack_all(self) -> None
```

Acknowledge every message.

#### EventBatch.retry_all

```python
def retry_all(self, **options: Any) -> None
```

Retry every message with the same [`EventMessage.retry`](#eventmessageretry) options.

**Args**

- `**options`: Keyword arguments forwarded to [`EventMessage.retry`](#eventmessageretry).

### EventMessage

```python
class EventMessage
```

One delivered domain event.

Exactly one outcome is recorded per message; the first of `ack` /
`retry` / `reject` / `dead_letter` / `suspend` wins and later calls
are ignored (as with Workers queue messages). Messages left untouched are
acked when `event()` returns and retried when it raises.

#### EventMessage.__init__

```python
def __init__(self, event: Any) -> None
```

#### EventMessage.json

```python
def json(self) -> Any
```

Decode `body` as JSON (`{}` for an empty body).

**Returns**

The parsed payload.

#### EventMessage.result

```python
@property
def result(self) -> dict[str, Any] | None
```

Recorded wire `EventResult` (flat `{"kind": ...}`), or `None` while undecided.

#### EventMessage.ack

```python
def ack(self) -> None
```

Mark handled; the host marks the event delivered.

#### EventMessage.retry

```python
def retry(
    self,
    *,
    retry_at: datetime | int | float | None=None,
    delay_seconds: float=0,
    reason: str='',
) -> None
```

Redeliver later.

**Args**

- `retry_at`: Explicit earliest redelivery instant (`datetime` or Unix ms).
- `delay_seconds`: Delay from now; ignored when `retry_at` is set.
- `reason`: Short operator-facing reason.

#### EventMessage.reject

```python
def reject(self, reason: str='') -> None
```

Stop delivering; the host records `reason`.

**Args**

- `reason`: Short operator-facing reason.

#### EventMessage.dead_letter

```python
def dead_letter(self, reason: str='') -> None
```

Park for operator review.

**Args**

- `reason`: Short operator-facing reason.

#### EventMessage.suspend

```python
def suspend(
    self,
    *,
    checkpoint: Any=None,
    checkpoint_schema_version: int=1,
    wake_at: datetime | int | float | None=None,
    wake_on_event_type: str='',
    wake_on_filter: Any=None,
) -> None
```

Release with a bounded checkpoint; the host redelivers later.

**Args**

- `checkpoint`: JSON-serializable checkpoint (at most `MAX_CHECKPOINT_BYTES`).
- `checkpoint_schema_version`: Schema version of `checkpoint`.
- `wake_at`: Earliest resume instant (`datetime` or Unix ms).
- `wake_on_event_type`: Also wake when an event of this type arrives.
- `wake_on_filter`: Host-owned payload filter for wake events.

**Raises**

- `PluginError`: `payload_too_large` when the checkpoint exceeds the cap.

### GuestDatabase

```python
class GuestDatabase
```

Host-granted SQL transport behind a named `[[databases]]` binding.

#### GuestDatabase.execute

```python
async def execute(self, _request: ExecuteRequest) -> ExecuteReply
```

Host-mediated typed batch (`ExecuteRequest` → `ExecuteReply`).

#### GuestDatabase.close

```python
async def close(self) -> None
```

Close the grant.

### JobController

```python
class JobController
```

Everything one job invocation may touch.

Carries the durable envelope, `input` / `output` streams, `progress()`,
`cancelled`, and the terminal-outcome recorders `suspend()` /
`retry_later()`. Returning normally without a recorded outcome completes
the job; raising rejects it (`unavailable` / `deadline_exceeded` errors
are retryable, `cancelled` is cancelled).

#### JobController.__init__

```python
def __init__(self, invocation: Any, granted: Any=None) -> None
```

#### JobController.json

```python
def json(self) -> Any
```

Decode `payload_json` (`{}` when empty).

**Returns**

The parsed job payload.

#### JobController.progress

```python
async def progress(self, percent: float, message: str='') -> None
```

Report `percent` in `0..=100` with an operator-facing `message`.

**Args**

- `percent`: Completion percentage.
- `message`: Operator-facing status text.

#### JobController.result

```python
@property
def result(self) -> dict[str, Any] | None
```

Recorded terminal outcome (flat `{"kind": ...}`), or `None` while running.

#### JobController.suspend

```python
def suspend(
    self,
    *,
    checkpoint: Any=None,
    checkpoint_schema_version: int=1,
    wake_at: datetime | int | float | None=None,
) -> None
```

Release with a bounded checkpoint; the host resumes at `wake_at`.

**Args**

- `checkpoint`: JSON-serializable checkpoint (at most `MAX_CHECKPOINT_BYTES`).
- `checkpoint_schema_version`: Schema version of `checkpoint`.
- `wake_at`: Earliest resume instant (`datetime` or Unix ms).

**Raises**

- `PluginError`: `payload_too_large` when the checkpoint exceeds the cap.

#### JobController.retry_later

```python
def retry_later(
    self,
    *,
    retry_at: datetime | int | float | None=None,
    reason: str='',
) -> None
```

Give up this attempt and let the host retry at `retry_at` (or its default).

**Args**

- `retry_at`: Earliest retry instant (`datetime` or Unix ms).
- `reason`: Short operator-facing reason.

#### JobController.cancel

```python
def cancel(self) -> None
```

Host-side cancellation observed (adapter-internal).

### KvNamespace

```python
class KvNamespace
```

`[[kv_namespaces]]` binding (`env.<BINDING>`): Workers `KVNamespace`-shaped storage.

The host serves the namespace from the library database, scoped to this
plugin and the namespace the row names, so values follow the instance
across hosts. The adapter isolate owns the kv grant; this wrapper converts
Python values into the JS values the adapter stub expects.

#### KvNamespace.__init__

```python
def __init__(self, stub: Any) -> None
```

#### KvNamespace.get

```python
async def get(self, key: str, type: str='text') -> Any
```

Read one key.

**Args**

- `key`: Key, at most `MAX_KV_KEY_BYTES` UTF-8 bytes.
- `type`: `text` (`str`), `json` (decoded value), or `bytes`.

**Returns**

The value, or `None` when absent or expired.

#### KvNamespace.get_with_metadata

```python
async def get_with_metadata(self, key: str, type: str='text') -> tuple[Any, Any]
```

Read one key with its metadata.

**Args**

- `key`: Key to read.
- `type`: `text`, `json`, or `bytes`.

**Returns**

`(value, metadata)`; both `None` when absent.

#### KvNamespace.put

```python
async def put(
    self,
    key: str,
    value: str | bytes,
    *,
    expiration: int | None=None,
    expiration_ttl: int | None=None,
    metadata: Any=None,
) -> None
```

Write one key, replacing any previous value and metadata.

**Args**

- `key`: Key to write.
- `value`: `str` (stored as UTF-8) or `bytes`.
- `expiration`: Absolute expiry in seconds since the Unix epoch.
- `expiration_ttl`: Relative expiry in seconds from the write.
- `metadata`: JSON-serializable metadata.

#### KvNamespace.delete

```python
async def delete(self, key: str) -> None
```

Delete one key; a missing key is not an error.

#### KvNamespace.list

```python
async def list(self, *, prefix: str='', cursor: str='', limit: int=0) -> dict[str, Any]
```

List keys in lexical order, one page at a time.

**Args**

- `prefix`: Only keys starting with this prefix.
- `cursor`: Cursor from a previous page.
- `limit`: Most keys to return (host default when 0).

**Returns**

`{"keys": [{"name", "expiration"?, "metadata"?}], "list_complete": bool, "cursor"?: str}`.

### NamedEntrypoint

```python
class NamedEntrypoint(WorkerEntrypoint)
```

Base for named entrypoints.

Subclasses list their RPC surface on `bookclerk_methods`; the adapter
reaches them only through `bookclerkInvoke`, which installs the
granted bindings on `env` before dispatching. Method arguments arrive as
plain Python values (`py`) and return values are converted with `js`.

**Fields**

- `bookclerk_methods: tuple[str, ...] = ()`

#### NamedEntrypoint.fetch

```python
async def fetch(self, _request=None)
```

Reject HTTP fetch — workerd guests are Workers-RPC only.

**Args**

- `_request`: Incoming HTTP request (unused by the default stub).

**Returns**

A 404 `Response`.

#### NamedEntrypoint.bookclerkInvoke

```python
async def bookclerkInvoke(self, context=None, method='', *args)
```

Adapter dispatch: call `method` with the granted bindings installed.

**Raises**

- `PluginError`: `unsupported` for methods outside `bookclerk_methods`.

### OidcEntrypoint

```python
class OidcEntrypoint(NamedEntrypoint)
```

`oidc` entrypoint: relying-party client templates and credential verification.

#### OidcEntrypoint.clients

```python
async def clients(self)
```

Plugin-provided OIDC client templates (list of `OidcClientTemplate`).

#### OidcEntrypoint.authenticateUser

```python
async def authenticateUser(self, _params: abi.AuthenticateUserParams)
```

Verify remote credentials on behalf of the host (`ExternalUser`).

### PluginError

```python
class PluginError(RuntimeError)
```

SDK-thrown failure. Unknown wire codes stay on `wire_code`.

#### PluginError.__init__

```python
def __init__(self, code: str, message: str)
```

#### PluginError.from_wire

```python
@classmethod
def from_wire(cls, code: str, message: str) -> PluginError
```

Build a `PluginError` from a wire `code` string.

**Args**

- `code`: Snake_case wire code (unknown codes are kept on `wire_code`).
- `message`: Operator-facing error text.

**Returns**

A `PluginError` whose `code` is a known variant or `unknown`.

### QueueBatch

```python
class QueueBatch
```

One `queue(batch)` delivery (`QueueBatch` struct), Workers `MessageBatch`-shaped.

#### QueueBatch.__init__

```python
def __init__(
    self,
    queue: Any,
    messages: Any,
    invocation: dict[str, Any] | None=None,
) -> None
```

#### QueueBatch.ack_all

```python
def ack_all(self) -> None
```

Acknowledge every message.

#### QueueBatch.retry_all

```python
def retry_all(self, **options: Any) -> None
```

Retry every message with the same [`QueueMessage.retry`](#queuemessageretry) options.

**Args**

- `**options`: Keyword arguments forwarded to [`QueueMessage.retry`](#queuemessageretry).

### QueueBinding

```python
class QueueBinding
```

`[[queues.producers]]` binding (`env.<BINDING>`): Workers `Queue`-shaped producer.

The adapter isolate owns the queue grant and serves `send` over the
granted channel; this wrapper converts Python bodies into the JS values
the adapter stub expects.

#### QueueBinding.__init__

```python
def __init__(self, stub: Any) -> None
```

#### QueueBinding.send

```python
async def send(
    self,
    body: Any,
    *,
    content_type: str='json',
    delay_seconds: int=0,
) -> dict[str, Any]
```

Send one message.

**Args**

- `body`: JSON value (default), `str` for `text`, or `bytes` for `bytes`.
- `content_type`: `json`, `text`, or `bytes`.
- `delay_seconds`: Seconds before the message may be delivered.

**Returns**

`{"messageIds": [...]}` with the outbox id the host assigned.

#### QueueBinding.send_batch

```python
async def send_batch(
    self,
    messages: list[dict[str, Any]],
    *,
    delay_seconds: int=0,
) -> dict[str, Any]
```

Send up to `MAX_LIST_PAGE` messages in order.

**Args**

- `messages`: `{"body": ..., "content_type": ..., "delay_seconds": ...}` entries.
- `delay_seconds`: Default delay for entries that carry none.

**Returns**

`{"messageIds": [...]}` with the outbox ids in send order.

### QueueMessage

```python
class QueueMessage
```

One delivered queue message (`QueueMessage` struct), Workers `Message`-shaped.

Exactly one outcome is recorded per message; the first of `ack` /
`retry` wins and later calls are ignored. Messages left untouched are
acked when `queue()` returns and retried when it raises.

#### QueueMessage.__init__

```python
def __init__(self, message: Any) -> None
```

#### QueueMessage.json

```python
def json(self) -> Any
```

Decode `body` as JSON (`None` for an empty body).

**Returns**

The parsed body.

#### QueueMessage.text

```python
def text(self) -> str
```

Decode `body` as UTF-8 text.

**Returns**

The body text.

#### QueueMessage.result

```python
@property
def result(self) -> dict[str, Any] | None
```

Recorded outcome (flat `{"kind": ...}`), or `None` while undecided.

#### QueueMessage.ack

```python
def ack(self) -> None
```

Mark handled; the host marks the message delivered.

#### QueueMessage.retry

```python
def retry(self, *, delay_seconds: float=0, reason: str='') -> None
```

Redeliver later.

**Args**

- `delay_seconds`: Seconds before redelivery; `0` applies the consumer's `retry_delay`.
- `reason`: Short operator-facing reason.

### RemoteLibraryEntrypoint

```python
class RemoteLibraryEntrypoint(NamedEntrypoint)
```

`remoteLibrary` entrypoint: lifecycle, rescan, listening sync, user polling.

#### RemoteLibraryEntrypoint.health

```python
async def health(self)
```

Liveness / readiness probe (`HealthOk`).

#### RemoteLibraryEntrypoint.diagnose

```python
async def diagnose(self)
```

Operator-facing diagnostic lines (list of strings).

#### RemoteLibraryEntrypoint.start

```python
async def start(self)
```

Start background work after the host has granted bindings.

#### RemoteLibraryEntrypoint.stop

```python
async def stop(self)
```

Stop background work.

#### RemoteLibraryEntrypoint.scanLibrary

```python
async def scanLibrary(self, _params: abi.ScanLibraryParams)
```

Re-sync the remote library.

#### RemoteLibraryEntrypoint.syncListening

```python
async def syncListening(self)
```

Push / pull listening progress (list of `ListeningProgress`).

#### RemoteLibraryEntrypoint.pollEvents

```python
async def pollEvents(self)
```

`ExternalUser` records observed since the last poll.

### StorageEntrypoint

```python
class StorageEntrypoint(NamedEntrypoint)
```

`storage` entrypoint (`Destination` interface): an object store the host writes to.

#### StorageEntrypoint.head

```python
async def head(self, _key: str)
```

Object metadata or `None`.

#### StorageEntrypoint.list

```python
async def list(self, _options: Any)
```

One page of objects under a prefix.

#### StorageEntrypoint.get

```python
async def get(self, _key: str, _options: Any=None)
```

Read an object (optionally a byte range).

#### StorageEntrypoint.put

```python
async def put(self, _key: str, _body: Any, _options: Any=None)
```

Write an object from a byte stream.

#### StorageEntrypoint.copy

```python
async def copy(self, _from_key: str, _to_key: str)
```

Server-side copy.

#### StorageEntrypoint.delete

```python
async def delete(self, _key: str)
```

Delete an object.

#### StorageEntrypoint.commit

```python
async def commit(self, _key: str, _commit_token: str)
```

Publish a staged object.

#### StorageEntrypoint.abortStage

```python
async def abortStage(self, _key: str, _commit_token: str)
```

Discard a staged object.

### StorefrontEntrypoint

```python
class StorefrontEntrypoint(NamedEntrypoint)
```

`storefront` entrypoint (`ContentSource` interface).

Every method takes and returns the generated [`bookclerk_plugin_sdk.abi`](bookclerk_plugin_sdk.abi.md)
TypedDict shapes as plain dicts (`LoginParams` → `LoginResult`, …).
Override only the operations the storefront supports; the rest raise
`unsupported`.

#### StorefrontEntrypoint.login

```python
async def login(self, _params: abi.LoginParams)
```

Password or one-shot OAuth login (`LoginParams` → `LoginResult`).

#### StorefrontEntrypoint.scan

```python
async def scan(self, _params: abi.ScanParams)
```

Enumerate owned titles (`ScanParams` → `ScanSummary`).

#### StorefrontEntrypoint.fetchTitle

```python
async def fetchTitle(self, _params: abi.FetchTitleParams)
```

Fetch one title's media and metadata (`FetchTitleParams` → `PlainFetch`).

#### StorefrontEntrypoint.listAccounts

```python
async def listAccounts(self)
```

List accounts known to this storefront (list of `SourceAccount`).

#### StorefrontEntrypoint.loginStart

```python
async def loginStart(self, _params: abi.LoginParams)
```

Begin an interactive OAuth login (`LoginParams` → `LoginStartResult`).

#### StorefrontEntrypoint.loginComplete

```python
async def loginComplete(self, _params: abi.LoginCompleteParams)
```

Finish an interactive OAuth login (`LoginCompleteParams` → `LoginResult`).

#### StorefrontEntrypoint.searchCatalog

```python
async def searchCatalog(self, _params: abi.SearchCatalogParams)
```

Free-text catalog search (`SearchCatalogParams` → list of `CatalogHit`).

#### StorefrontEntrypoint.expandCandidates

```python
async def expandCandidates(self, _params: abi.ExpandCandidatesParams)
```

Related-title expansion (`ExpandCandidatesParams` → list of `CatalogHit`).

#### StorefrontEntrypoint.purchaseHint

```python
async def purchaseHint(self, _params: abi.PurchaseHintParams)
```

Purchase link / price hint (`PurchaseHintParams` → `PurchaseHint`).

#### StorefrontEntrypoint.listDeals

```python
async def listDeals(self, _params: abi.ListDealsParams)
```

Current storefront deals (`ListDealsParams` → list of `CatalogHit`).

#### StorefrontEntrypoint.catalogDetail

```python
async def catalogDetail(self, _params: abi.CatalogDetailParams)
```

Full catalog record (`CatalogDetailParams` → `CatalogHit`).

#### StorefrontEntrypoint.revokeAccount

```python
async def revokeAccount(self, _params: abi.RevokeAccountParams) -> None
```

Sign the account out at the store before the host deletes its credentials.

Best effort: the host revokes the account whatever this returns. The
default does nothing (`RevokeAccountParams` → `None`).

#### StorefrontEntrypoint.health

```python
async def health(self)
```

Liveness / readiness probe (`HealthOk`).

#### StorefrontEntrypoint.diagnose

```python
async def diagnose(self)
```

Operator-facing diagnostic lines (list of strings).

## Functions

### cli_args

```python
def cli_args(params: Any) -> dict[str, str]
```

Turn `CliInvokeParams.args` (`[{name, value}]`) into a `{name: value}` dict.

**Args**

- `params`: `CliInvokeParams` (JS or Python).

**Returns**

Argument values keyed by name.

### decode_extensible_config

```python
def decode_extensible_config(cfg: Any) -> Any
```

Decode an `ExtensibleConfig` into the plain value on `env.CONFIG`.

**Args**

- `cfg`: `{"schemaVersion", "mediaType", "payload"}` (JS or Python).

**Returns**

The parsed JSON object for JSON media types (`{}` when empty or
malformed); `{"mediaType", "text"}` for other media types.

### event_batch_results

```python
def event_batch_results(
    batch: EventBatch,
    failed: BaseException | None,
) -> list[dict[str, Any]]
```

Translate recorded outcomes into the wire `EventResult` list.

**Args**

- `batch`: The delivered batch.
- `failed`: Exception raised by the author's `event()`, if any; undecided messages then retry instead of ack.

**Returns**

One flat `{"kind": ...}` result per message, in delivery order.

### granted_databases

```python
def granted_databases(
    granted: _GrantedFetcher,
    database_tokens: dict[str, str],
    *,
    signal: Any | None=None,
) -> dict[str, DatabaseBinding]
```

Build named [`DatabaseBinding`](bookclerk_plugin_sdk.db_value.md#databasebinding) objects over `POST /db/execute`.

Each entry of `database_tokens` (binding name → per-invocation grant
token) becomes an isolated D1-shaped binding whose terminal methods
(`first`, `run`, `all`, `batch`) are async and route through the
granted transport — the same objects the adapter installs on `env`.

**Args**

- `granted`: Granted-channel fetcher (`GRANTED` service binding).
- `database_tokens`: Binding name → grant token.
- `signal`: Optional `AbortSignal` forwarded to every request.

**Returns**

Binding name → [`DatabaseBinding`](bookclerk_plugin_sdk.db_value.md#databasebinding).

### job_outcome_for

```python
def job_outcome_for(
    job: JobController,
    returned: Any,
    failed: BaseException | None,
) -> dict[str, Any]
```

Translate a `job()` return / raise into the wire `JobOutcome`.

**Args**

- `job`: The controller handed to the author.
- `returned`: The author's return value (`{"message", "bytesCopied"}` or `None`).
- `failed`: Exception raised by the author, if any.

**Returns**

A flat `{"kind": ...}` outcome record.

### js

```python
def js(value: Any) -> Any
```

Convert a Python value to a JS value for Workers RPC results.

Inside the isolate this uses `pyodide.ffi.to_js` with plain-object
dictionaries (`bytes` become `Uint8Array`); when `to_js` is not
available it falls back to `JSON.parse(json.dumps(value))`. Under the
authoring stubs the value is returned unchanged.

**Args**

- `value`: JSON-serializable Python value (`bytes` allowed for `Data`).

**Returns**

The JS value (or the original Python value under the authoring stubs).

### json_payload

```python
def json_payload(value: Any) -> dict[str, Any]
```

Wrap a JSON-serializable value as an `application/json` `ExtensibleConfig`.

This is the shape of `CliInvokeResult.payload` and every other extensible
payload on the wire (schema version 1).

**Args**

- `value`: JSON-serializable Python value.

**Returns**

`{"schemaVersion": 1, "mediaType": "application/json", "payload": bytes}`.

### py

```python
def py(value: Any) -> Any
```

Convert a JS value delivered over Workers RPC into plain Python.

`JsProxy` objects are converted with `to_py()` (`Uint8Array` →
`bytes`); Python values pass through unchanged.

**Args**

- `value`: A JsProxy or Python value.

**Returns**

A plain Python value (`dict` / `list` / `bytes` / scalars).

### queue_batch_results

```python
def queue_batch_results(
    batch: QueueBatch,
    failed: BaseException | None,
) -> list[dict[str, Any]]
```

Translate recorded outcomes into the wire `QueueResult` list.

**Args**

- `batch`: The delivered batch.
- `failed`: Exception raised by the author's `queue()`, if any; undecided messages then retry instead of ack.

**Returns**

One flat `{"kind": ...}` result per message, in delivery order.

### transport_database

```python
def transport_database(transport: Any) -> DatabaseBinding
```

Wrap an adapter `GrantedDatabaseTransport` stub as a [`DatabaseBinding`](bookclerk_plugin_sdk.db_value.md#databasebinding).

**Args**

- `transport`: Stub exposing `executeBytes(bytes) -> bytes`.

**Returns**

D1-shaped binding whose terminal methods route through the transport.

### transport_kv

```python
def transport_kv(stub: Any) -> KvNamespace
```

Wrap the adapter's granted KV stub as a [`KvNamespace`](#kvnamespace).

**Args**

- `stub`: `KvNamespaceBinding`-shaped JS stub from the granted context.

**Returns**

The author-facing binding.

### transport_queue

```python
def transport_queue(stub: Any) -> QueueBinding
```

Wrap the adapter's granted queue stub as a [`QueueBinding`](#queuebinding).

**Args**

- `stub`: `QueueBinding`-shaped JS stub from the granted context.

**Returns**

The author-facing binding.

## Constants

### FEATURE_SCALAR_LIMITS

```python
FEATURE_SCALAR_LIMITS: str = 'rpc.scalarLimits'
```

Guest honors scalar / stream-window / list-page caps (`rpc.scalarLimits`).

### FEATURE_STORAGE_COPY

```python
FEATURE_STORAGE_COPY: str = 'storage.copy'
```

Guest implements server-side `Destination.copy` (`storage.copy`).

### FEATURE_STREAMS

```python
FEATURE_STREAMS: str = 'rpc.streams'
```

Media moves through transferred `ByteRange` / `ByteSource` streams (`rpc.streams`).

### MAX_CHECKPOINT_BYTES

```python
MAX_CHECKPOINT_BYTES: int = 65536
```

Maximum job / event checkpoint payload size (bytes).

### MAX_LIST_PAGE

```python
MAX_LIST_PAGE: int = 256
```

Maximum objects in one `Destination.list` page.

### MAX_PLUGIN_MIGRATION_OPS

```python
MAX_PLUGIN_MIGRATION_OPS: int = 256
```

Maximum already-separated operations in one plugin-owned migration.

### MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES

```python
MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES: int = 262144
```

Maximum aggregate UTF-8 bytes of plugin migration ids plus SQL in one `databaseMigrations` registration.

### MAX_PLUGIN_MIGRATION_TOTAL_OPS

```python
MAX_PLUGIN_MIGRATION_TOTAL_OPS: int = 2048
```

Maximum already-separated operations across one `databaseMigrations` registration.

### MAX_SCALAR_BYTES

```python
MAX_SCALAR_BYTES: int = 262144
```

Maximum decoded size of an ordinary RPC scalar value (not a stream window).

### MAX_STREAM_WINDOW_BYTES

```python
MAX_STREAM_WINDOW_BYTES: int = 1048576
```

Maximum bytes returned by one `ByteSource.pull` (flow-control window).

### PRODUCT_API_VERSION

```python
PRODUCT_API_VERSION: int = 3
```

Product ABI version (`plugin.toml` `api_version` / `describe().apiVersion`).
