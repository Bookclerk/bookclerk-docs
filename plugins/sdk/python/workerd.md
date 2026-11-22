# bookclerk_plugin_sdk.workerd

Workerd author runtime — `BookclerkEntrypoint` and the named entrypoints.

Python mirror of `@bookclerk/plugin-sdk/workerd` (`embed/bookclerk_plugin.js`)
for `api_version = 3` guests running as Python Workers:

- `from bookclerk_plugin_sdk.workerd import BookclerkEntrypoint, js`

The default export is a module-level `Default` class extending
[`BookclerkEntrypoint`](#bookclerkentrypoint) with optional `event(batch)` / `job(job)`
triggers. Named entrypoints from `plugin.toml` `entrypoints` are module-level
classes named after the wire name (`Storefront`, `Storage`, `RemoteLibrary`,
`DatabaseAdapter`, `Cli`, `Oidc`, `Metadata`, `Covers`, `MetadataFiles`, `Notifier`, `Ai`) extending the matching `*Entrypoint`
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
| [`HostWebSocket`](#hostwebsocket) | class | One host-held WebSocket, shaped like a hibernatable Workers `WebSocket`. |
| [`JobController`](#jobcontroller) | class | Everything one job invocation may touch. |
| [`KvNamespace`](#kvnamespace) | class | `[[kv_namespaces]]` binding (`env.<BINDING>`): Workers `KVNamespace`-shaped storage. |
| [`CoverEntrypoint`](#coverentrypoint) | class | `covers` entrypoint: downloads cover images for the host. |
| [`MetadataFilesEntrypoint`](#metadatafilesentrypoint) | class | `metadataFiles` entrypoint: reads and writes metadata files for the host. |
| [`IngressEntrypoint`](#ingressentrypoint) | class | `ingress` entrypoint: gives the library a public `https` address. |
| [`MetadataEntrypoint`](#metadataentrypoint) | class | `metadata` entrypoint: bibliographic lookups against a public catalog. |
| [`NamedEntrypoint`](#namedentrypoint) | class | Base for named entrypoints. |
| [`AiBinding`](#aibinding) | class | `AI` binding, present when `plugin.toml` lists tasks under `[ai].calls`. |
| [`AiEntrypoint`](#aientrypoint) | class | `ai` entrypoint: serves models to the host and to plugins with the grant. |
| [`NotifierEntrypoint`](#notifierentrypoint) | class | `notifier` entrypoint: delivers messages the host already rendered. |
| [`NotifyBinding`](#notifybinding) | class | `NOTIFY` binding, present when `plugin.toml` declares `[[notifications.types]]`. |
| [`OidcEntrypoint`](#oidcentrypoint) | class | `oidc` entrypoint: relying-party client templates and credential verification. |
| [`PluginError`](#pluginerror) | class | SDK-thrown failure. Unknown wire codes stay on `wire_code`. |
| [`QueueBatch`](#queuebatch) | class | One `queue(batch)` delivery (`QueueBatch` struct), Workers `MessageBatch`-shaped. |
| [`QueueBinding`](#queuebinding) | class | `[[queues.producers]]` binding (`env.<BINDING>`): Workers `Queue`-shaped producer. |
| [`QueueMessage`](#queuemessage) | class | One delivered queue message (`QueueMessage` struct), Workers `Message`-shaped. |
| [`RemoteLibraryEntrypoint`](#remotelibraryentrypoint) | class | `remoteLibrary` entrypoint: lifecycle, rescan, listening sync, user polling. |
| [`ScheduledController`](#scheduledcontroller) | class | One `[triggers] crons` fire, as `scheduled(controller)` receives it. |
| [`SocketsBinding`](#socketsbinding) | class | `[sockets]` binding (`env.SOCKETS`): dial host-held WebSockets. |
| [`StorageEntrypoint`](#storageentrypoint) | class | `storage` entrypoint (`Destination` interface): an object store the host writes to. |
| [`StorefrontEntrypoint`](#storefrontentrypoint) | class | `storefront` entrypoint (`ContentSource` interface). |
| [`WebhookMessage`](#webhookmessage) | class | One stored webhook request, leased to this pull. |
| [`WebhookPullResult`](#webhookpullresult) | class | One pull's leased requests. |
| [`WebhooksBinding`](#webhooksbinding) | class | `[[webhooks]]` binding (`env.WEBHOOKS`). |
| [`pack_embedding_vectors`](#pack_embedding_vectors) | function | Pack embedding vectors into the wire form of `AiTextEmbeddingsOutput["vectors"]`. |
| [`unpack_embedding_vectors`](#unpack_embedding_vectors) | function | Split the wire form of `AiTextEmbeddingsOutput["vectors"]` into vectors. |
| [`cli_args`](#cli_args) | function | Turn `CliInvokeParams.args` (`[{name, value}]`) into a `{name: value}` dict. |
| [`decode_extensible_config`](#decode_extensible_config) | function | Decode an `ExtensibleConfig` into the plain value on `env.CONFIG`. |
| [`decode_socket_attachment`](#decode_socket_attachment) | function | Decode stored socket attachment bytes. |
| [`encode_socket_attachment`](#encode_socket_attachment) | function | Encode a socket attachment as the host stores it: UTF-8 JSON. |
| [`event_batch_results`](#event_batch_results) | function | Translate recorded outcomes into the wire `EventResult` list. |
| [`granted_databases`](#granted_databases) | function | Build named `DatabaseBinding` objects over `POST /db/execute`. |
| [`job_outcome_for`](#job_outcome_for) | function | Translate a `job()` return / raise into the wire `JobOutcome`. |
| [`js`](#js) | function | Convert a Python value to a JS value for Workers RPC results. |
| [`json_payload`](#json_payload) | function | Wrap a JSON-serializable value as an `application/json` `ExtensibleConfig`. |
| [`py`](#py) | function | Convert a JS value delivered over Workers RPC into plain Python. |
| [`queue_batch_results`](#queue_batch_results) | function | Translate recorded outcomes into the wire `QueueResult` list. |
| [`socket_message_data`](#socket_message_data) | function | The message a handler receives for one wire frame. |
| [`transport_database`](#transport_database) | function | Wrap an adapter `GrantedDatabaseTransport` stub as a `DatabaseBinding`. |
| [`transport_kv`](#transport_kv) | function | Wrap the adapter's granted KV stub as a `KvNamespace`. |
| [`transport_queue`](#transport_queue) | function | Wrap the adapter's granted queue stub as a `QueueBinding`. |
| [`transport_sockets`](#transport_sockets) | function | Wrap the adapter's granted sockets stub as a `SocketsBinding`. |
| [`transport_webhooks`](#transport_webhooks) | function | Wrap the adapter's granted webhooks stub as a `WebhooksBinding`. |
| [`FEATURE_SCALAR_LIMITS`](#feature_scalar_limits) | constant | Guest honors scalar / stream-window / list-page caps (`rpc.scalarLimits`). |
| [`FEATURE_STORAGE_CHANGES`](#feature_storage_changes) | constant | Guest implements `Destination.changes` (`storage.changes`). |
| [`FEATURE_STORAGE_CONDITIONAL_WRITE`](#feature_storage_conditional_write) | constant | Guest enforces `WriteOptions.ifMatch` / `ifNoneMatch` (`storage.conditionalWrite`). |
| [`FEATURE_STORAGE_COPY`](#feature_storage_copy) | constant | Guest implements server-side `Destination.copy` (`storage.copy`). |
| [`FEATURE_STORAGE_DELETE_MANY`](#feature_storage_delete_many) | constant | Guest implements `Destination.deleteMany` (`storage.deleteMany`). |
| [`FEATURE_STORAGE_HEAD_MANY`](#feature_storage_head_many) | constant | Guest implements `Destination.headMany` (`storage.headMany`). |
| [`FEATURE_STORAGE_MOVE`](#feature_storage_move) | constant | Guest implements `Destination.move` (`storage.move`). |
| [`FEATURE_STORAGE_MOVE_MANY`](#feature_storage_move_many) | constant | Guest implements `Destination.moveMany` (`storage.moveMany`). |
| [`FEATURE_STREAMS`](#feature_streams) | constant | Media moves through transferred `ByteRange` / `ByteSource` streams (`rpc.streams`). |
| [`MAX_BATCH_KEYS`](#max_batch_keys) | constant | Maximum keys or key pairs in one `Destination.moveMany`, `deleteMany` or `headMany` call, and maximum outcomes in one reply. |
| [`MAX_CHANGES_CURSOR_BYTES`](#max_changes_cursor_bytes) | constant | Maximum bytes of a `Destination.changes` cursor. |
| [`MAX_CHANGES_PAGE`](#max_changes_page) | constant | Maximum changes in one `Destination.changes` page. |
| [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes) | constant | Maximum job / event checkpoint payload size (bytes). |
| [`MAX_LIST_PAGE`](#max_list_page) | constant | Maximum objects in one `Destination.list` page. |
| [`MAX_PLUGIN_MIGRATION_OPS`](#max_plugin_migration_ops) | constant | Maximum already-separated operations in one plugin-owned migration. |
| [`MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](#max_plugin_migration_registration_bytes) | constant | Maximum aggregate UTF-8 bytes of plugin migration ids plus SQL in one `databaseMigrations` registration. |
| [`MAX_PLUGIN_MIGRATION_TOTAL_OPS`](#max_plugin_migration_total_ops) | constant | Maximum already-separated operations across one `databaseMigrations` registration. |
| [`MAX_SCALAR_BYTES`](#max_scalar_bytes) | constant | Maximum decoded size of an ordinary RPC scalar value (not a stream window). |
| [`MAX_SOCKET_ATTACHMENT_BYTES`](#max_socket_attachment_bytes) | constant | Largest per-socket attachment the host keeps for a sleeping guest. |
| [`MAX_SOCKET_FRAME_BYTES`](#max_socket_frame_bytes) | constant | Largest WebSocket message, text or binary, that `Sockets.send` accepts or `SocketHandler.message` delivers. |
| [`MAX_STREAM_WINDOW_BYTES`](#max_stream_window_bytes) | constant | Maximum bytes returned by one `ByteSource.pull` (flow-control window). |
| [`MAX_WEBHOOK_LEASE_ID_BYTES`](#max_webhook_lease_id_bytes) | constant | Longest `WebhookDelivery.leaseId`, in bytes. |
| [`MAX_WEBHOOK_NAME_BYTES`](#max_webhook_name_bytes) | constant | Longest `[[webhooks]]` name, in bytes. |
| [`MAX_WEBHOOK_PULL_BATCH`](#max_webhook_pull_batch) | constant | Most deliveries one `Webhooks.pull` returns. |
| [`MAX_WEBHOOK_VISIBILITY_MS`](#max_webhook_visibility_ms) | constant | Longest `Webhooks.pull` visibility timeout, in milliseconds (12 hours). |
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

#### AdapterDatabaseSession.executeStreamed

```python
async def executeStreamed(self, _request: abi.AdapterExecuteRequest) -> bytes
```

The same atomic batch as `execute`, answered as `ExecuteStreamFrame` bytes.

The frames are counted against the advertised `maxStreamedResultBytes`
/ `maxStreamedResultRows`; adapters that advertise `0` for both
keep this default, which answers `unsupported`. The request carries
the proven statements plus `isolation` and `receipt`, like the
adapter-side `execute` envelope. An isolate adapter's frames travel
inside one invoke reply, so they must fit `MAX_SCALAR_BYTES`;
`db_value.execute_adapter_batch_streamed` runs the batch, stores the
replay receipt and checks that bound and the advertised streamed
budget before it commits.

**Returns**

Every frame of the committed reply, concatenated.

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

Optional trigger methods: `event(batch)` (for `[[events.consumers]]`),
`queue(batch)` (for `[[queues.consumers]]`), `scheduled(controller)`
(for `[triggers] crons`) and `job(job)` (for `[triggers] jobs`);
optional `describe()`
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

#### BookclerkEntrypoint.bookclerkScheduled

```python
async def bookclerkScheduled(self, context: Any=None, wire_controller: Any=None) -> None
```

Adapter dispatch: run one cron fire through `scheduled(controller)`.

The handler receives a [`ScheduledController`](#scheduledcontroller). Returning
completes the fire; raising has the host retry the same fire.

**Raises**

- `PluginError`: `unsupported` when the author defines no `scheduled`.

#### BookclerkEntrypoint.bookclerkSocketMessage

```python
async def bookclerkSocketMessage(self, context: Any=None, wire: Any=None) -> Any
```

Adapter dispatch: one host-held socket message to `web_socket_message(ws, message)`.

Text arrives as `str`, binary as `bytes`. The handler may return
`{"poll_events": True}` (or `pollEvents`) when the message announced
something `poll_events` hands over, such as a new user; the host then
polls now instead of at its next scheduled poll.

**Returns**

The wire `SocketMessageOk`: `{"pollEvents": bool}`.

**Raises**

- `PluginError`: `unsupported` when the author defines no handler.

#### BookclerkEntrypoint.bookclerkSocketClosed

```python
async def bookclerkSocketClosed(self, context: Any=None, wire: Any=None) -> None
```

Adapter dispatch: a host-held socket closed, to `web_socket_close(ws, code, reason, was_clean)`.

Without a handler the close is acknowledged and ignored.

#### BookclerkEntrypoint.bookclerkWebhookDelivered

```python
async def bookclerkWebhookDelivered(self, context: Any=None, wire: Any=None) -> None
```

Adapter dispatch: the host stored requests, to `webhook_delivered(name, waiting)`.

Without a handler the call is acknowledged and ignored; the plugin
pulls on its own schedule.

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

#### GuestDatabase.execute_streamed

```python
async def execute_streamed(self, _request: ExecuteRequest) -> bytes | AsyncIterable[bytes]
```

The same batch answered as `ExecuteStreamFrame` bytes.

The reply is counted against the session's `maxStreamedResultBytes`
/ `maxStreamedResultRows` instead of the inline caps, so a result
past them still arrives whole.

**Returns**

Every frame, concatenated, or the frames as a byte stream.

**Raises**

- `PluginError`: `unsupported` on a host that predates the method.

#### GuestDatabase.close

```python
async def close(self) -> None
```

Close the grant.

#### GuestDatabase.limits

```python
async def limits(self) -> DbCapabilities
```

Negotiated limits of this session (`DbCapabilities`).

**Returns**

The adapter's capability advertisement for this session.

**Raises**

- `PluginError`: `unsupported` on a host that predates the method.

### HostWebSocket

```python
class HostWebSocket
```

One host-held WebSocket, shaped like a hibernatable Workers `WebSocket`.

The broker owns the connection; this handle only names it, so it stays
valid across invocations and the guest may sleep between messages.

**Attributes**

- `id`: Socket id the host assigned (`connect` / `get_web_sockets`).
- `url`: URL the socket dialed, without credentials.
- `protocol`: Subprotocol the server chose, or an empty string.

#### HostWebSocket.__init__

```python
def __init__(self, stub: Any, fields: Any) -> None
```

#### HostWebSocket.send

```python
async def send(self, message: str | bytes) -> None
```

Send one message.

**Args**

- `message`: `str` (text frame) or bytes (binary frame), at most `MAX_SOCKET_FRAME_BYTES` bytes.

#### HostWebSocket.close

```python
async def close(self, code: int=0, reason: str='') -> None
```

Close the socket.

**Args**

- `code`: Close code; `0` sends 1000.
- `reason`: Close reason, at most 123 UTF-8 bytes.

#### HostWebSocket.serialize_attachment

```python
async def serialize_attachment(self, value: Any) -> None
```

Store a JSON value with the socket, handed back with every message.

**Args**

- `value`: JSON-serializable value of at most `MAX_SOCKET_ATTACHMENT_BYTES` encoded bytes; `None` clears it.

#### HostWebSocket.deserialize_attachment

```python
def deserialize_attachment(self) -> Any
```

The stored attachment value, or `None` when none is set.

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

### CoverEntrypoint

```python
class CoverEntrypoint(NamedEntrypoint)
```

`covers` entrypoint: downloads cover images for the host.

The host passes a `sink` destination; put the body for `urls[i]` under
the key `str(i)` and return one `CoverImage` per URL, in request
order. The host sniffs, decodes and re-encodes every body.

#### CoverEntrypoint.health

```python
async def health(self) -> abi.HealthOk
```

Liveness / readiness probe (`HealthOk`).

#### CoverEntrypoint.fetchCovers

```python
async def fetchCovers(
    self,
    _params: abi.CoverFetchParams,
    _sink: Any,
) -> list[abi.CoverImage]
```

Fetch each URL into `_sink` under its index (list of `CoverImage`).

### MetadataFilesEntrypoint

```python
class MetadataFilesEntrypoint(NamedEntrypoint)
```

`metadataFiles` entrypoint: reads and writes metadata files for the host.

The host reads every file, chooses every path and writes every result, so
the entrypoint only converts bytes. Each dialect is a
`[[metadata_files.dialects]]` row in `plugin.toml`; `params["dialect"]`
names the row.

#### MetadataFilesEntrypoint.health

```python
async def health(self) -> abi.HealthOk
```

Liveness / readiness probe (`HealthOk`).

#### MetadataFilesEntrypoint.parse

```python
async def parse(self, _params: abi.MetadataFileParseParams) -> abi.MetadataFileRecord
```

What one file says (`MetadataFileRecord`).

List every field the file states in `present`, including ones it
states as empty. Raise `invalid_params` for a file the dialect cannot
read.

#### MetadataFilesEntrypoint.render

```python
async def render(self, _params: abi.MetadataFileRenderParams) -> bytes | str
```

The new file: `existing` with `fields` set from `record`.

Everything else in the file is kept; a `str` is written as UTF-8.

### IngressEntrypoint

```python
class IngressEntrypoint(NamedEntrypoint)
```

`ingress` entrypoint: gives the library a public `https` address.

The host starts at most one ingress instance at a time. The Python SDK
serves `[ingress] kind = "direct"` plugins (dynamic DNS, port mapping),
which keep the address current and never receive requests; the `ORIGIN`
binding a relay plugin hands requests to is provided by the TypeScript
SDK only.

#### IngressEntrypoint.start

```python
async def start(self) -> abi.IngressStarted
```

Bring the address up and return it once it serves (`IngressStarted`).

The origin is `https://` and a host name, with no port, path or
trailing slash.

#### IngressEntrypoint.health

```python
async def health(self) -> abi.IngressHealth
```

Whether the address is serving (`IngressHealth`).

#### IngressEntrypoint.stop

```python
async def stop(self) -> None
```

Take the address down; the host stops relying on it after this returns.

### MetadataEntrypoint

```python
class MetadataEntrypoint(NamedEntrypoint)
```

`metadata` entrypoint: bibliographic lookups against a public catalog.

Records describe editions; the host decides identity.

#### MetadataEntrypoint.health

```python
async def health(self) -> abi.HealthOk
```

Liveness / readiness probe (`HealthOk`).

#### MetadataEntrypoint.lookup

```python
async def lookup(self, _params: abi.MetadataLookupParams) -> list[abi.MetadataRecord]
```

Records for the editions the identifiers name (list of `MetadataRecord`).

#### MetadataEntrypoint.search

```python
async def search(self, _params: abi.MetadataSearchParams) -> list[abi.MetadataRecord]
```

Records matching a title and author (list of `MetadataRecord`).

#### MetadataEntrypoint.chapters

```python
async def chapters(self, _params: abi.MetadataChaptersParams) -> abi.MetadataChapters
```

Chapter layout of one edition (`MetadataChapters`).

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

### AiBinding

```python
class AiBinding
```

`AI` binding, present when `plugin.toml` lists tasks under `[ai].calls`.

The host routes each run to the deployed plugin that serves the model.
The adapter isolate owns the ai grant; this wrapper converts Python
values into the JS values the adapter stub expects.

#### AiBinding.__init__

```python
def __init__(self, stub: Any) -> None
```

#### AiBinding.run

```python
async def run(
    self,
    model: str,
    *,
    text: str | list[str],
    role: abi.AiTextRole='document',
) -> dict[str, Any]
```

Run one text-embedding model on one input (`env.AI.run`).

**Args**

- `model`: `@<plugin id>/<model id>`, or a task alias such as `@library/text-embeddings` for the model the library chose.
- `text`: One text or a batch (at most `MAX_AI_EMBED_TEXTS`), none empty.
- `role`: `document` for stored content, `query` for a search query matched against documents.

**Returns**

``{"shape": [texts, dimensions], "data": [[...], ...], "model":
..., "revision": ..., "fallbacks": [...]}``. Vectors are
comparable only when their model and revision match.

**Raises**

- `PluginError`: `invalid_params` for a malformed run, `not_found` when nothing deployed serves the model, `forbidden` for a task that is not granted, `rate_limited` past the plugin's budget, `deadline_exceeded` or `unavailable` when the model plugin cannot answer.

### AiEntrypoint

```python
class AiEntrypoint(NamedEntrypoint)
```

`ai` entrypoint: serves models to the host and to plugins with the grant.

`[ai].serves` in `plugin.toml` declares the tasks. The host checks
every listing and every reply, so a wrong vector count or length fails
the run.

#### AiEntrypoint.health

```python
async def health(self) -> abi.HealthOk
```

Liveness / readiness probe (`HealthOk`).

#### AiEntrypoint.models

```python
async def models(self) -> abi.AiModelList
```

The models served now (`AiModelList`).

Must be cheap: do not load a model here. Change a model's
`revision` whenever its weights, tokenizer or loaded language packs
change, because outputs are comparable only within one revision.

#### AiEntrypoint.run

```python
async def run(self, _params: abi.AiRunParams) -> abi.AiRunOk
```

Run one model on one input (`AiRunOk`).

Raise `not_found` for a model this plugin does not serve and
`invalid_params` for an input of another task. For text embeddings
return one vector per text, in input order
([`pack_embedding_vectors`](#pack_embedding_vectors)), with the model's listed
`dimensions` and `revision`; leave `model` unset.

### NotifierEntrypoint

```python
class NotifierEntrypoint(NamedEntrypoint)
```

`notifier` entrypoint: delivers messages the host already rendered.

The host decides what to send and where; the entrypoint only sends to
one outside service (a chat webhook, a mail relay, a paging service).
`[notifier]` in `plugin.toml` declares the destination kinds, formats
and personal address fields.

#### NotifierEntrypoint.health

```python
async def health(self) -> abi.HealthOk
```

Liveness / readiness probe (`HealthOk`).

#### NotifierEntrypoint.send

```python
async def send(self, _params: abi.NotifySendParams) -> abi.NotifySent | None
```

Deliver one message to one destination (`NotifySent`).

Raise `rate_limited` (with `retryAfterMs` in the details when the
service says when) or `unavailable` for a failure worth retrying; any
other error fails this destination. Never log the message when
`sensitive` is set, nor `addressJson`. `None` means the service
gave no message id.

#### NotifierEntrypoint.validateDestination

```python
async def validateDestination(
    self,
    params: abi.NotifyValidateParams,
) -> abi.NotifyValidated
```

Check and normalise a personal address (`NotifyValidated`).

Raise `invalid_params` for an address the service would refuse. The
default accepts the address as given.

### NotifyBinding

```python
class NotifyBinding
```

`NOTIFY` binding, present when `plugin.toml` declares `[[notifications.types]]`.

The host routes each raise by the administrators' rules and each
patron's preferences; the plugin never learns where it went. The adapter
isolate owns the notify grant; this wrapper converts Python values into
the JS values the adapter stub expects.

#### NotifyBinding.__init__

```python
def __init__(self, stub: Any) -> None
```

#### NotifyBinding.raise_

```python
async def raise_(
    self,
    notification_type: str,
    *,
    fields: list[abi.NotificationField] | dict[str, str] | None=None,
    patron_id: str='',
    link_path: str='',
    group_key: str='',
    alert_action: abi.NotificationAlertAction='none',
    alert_key: str='',
) -> abi.NotifyRaised
```

Raise one declared notification type (`env.NOTIFY.raise`).

**Args**

- `notification_type`: Type id from `[[notifications.types]]`, without a plugin prefix.
- `fields`: Labelled values as `{"name", "value"}` rows or a plain `name -> value` dict (at most `MAX_NOTIFICATION_FIELDS`).
- `patron_id`: Patron the notification is for; required for a `patron` type and must be inside the invocation's account scope.
- `link_path`: Path inside the library to link to (`/books/42`).
- `group_key`: Notifications with one group key may be threaded together.
- `alert_action`: `none`, `trigger` or `resolve`.
- `alert_key`: Alert to open or close; required unless `alert_action` is `none`.

**Returns**

`{"notificationId": ...}` the host assigned.

**Raises**

- `PluginError`: `invalid_params` for an undeclared type or a malformed raise, `forbidden` for a patron outside the account scope, `rate_limited` past the per-instance raise budget.

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

**Attributes**

- `code`: Known `PluginErrorCode` wire string, or `unknown`.
- `wire_code`: Raw wire code, including codes this SDK does not know.
- `details`: Structured detail object from the wire (`details` JSON), or `None` when the error carries none.

#### PluginError.__init__

```python
def __init__(self, code: str, message: str, details: dict[str, Any] | str | None=None)
```

#### PluginError.from_wire

```python
@classmethod
def from_wire(cls, code: str, message: str, details: str='') -> PluginError
```

Build a `PluginError` from a wire `code` string.

**Args**

- `code`: Snake_case wire code (unknown codes are kept on `wire_code`).
- `message`: Operator-facing error text.
- `details`: `details` JSON object text from the wire (empty when none).

**Returns**

A `PluginError` whose `code` is a known variant or `unknown`.

#### PluginError.limit_exceeded

```python
@classmethod
def limit_exceeded(cls, limit: LimitExceeded) -> PluginError
```

Build a `limit_exceeded` error carrying `limit` in message and details.

**Args**

- `limit`: Exceeded cap.

**Returns**

A `PluginError` with code `limit_exceeded`.

#### PluginError.limit

```python
@property
def limit(self) -> LimitExceeded | None
```

The exceeded cap when this is a `limit_exceeded` error.

**Returns**

The cap from `details` first, then from the canonical message;
`None` for every other code.

#### PluginError.details_text

```python
@property
def details_text(self) -> str
```

`details` as the JSON text the wire carries.

**Returns**

JSON object text, or an empty string when the error has none.

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

### ScheduledController

```python
@dataclass(frozen=True)
class ScheduledController
```

One `[triggers] crons` fire, as `scheduled(controller)` receives it.

Fires are delivered at least once; a fire that raises is retried with the
same `scheduled_time` and `fire_id`, so dedupe on `fire_id`.

**Attributes**

- `cron`: The schedule that fired, normalized to single spaces.
- `scheduled_time`: Fire time in UTC Unix milliseconds (a whole minute).
- `fire_id`: Durable id of this fire, stable across retries.
- `attempt`: Failure attempt, starting at 1.
- `deadline_unix_ms`: Deadline hint in Unix milliseconds; the host fence is authoritative.

#### ScheduledController.from_wire

```python
@classmethod
def from_wire(cls, wire: Any) -> ScheduledController
```

Builds the controller from a wire `ScheduledController`.

**Args**

- `wire`: Decoded wire struct (a JS proxy or a dict); fields may be missing.

**Returns**

The controller with every field present.

### SocketsBinding

```python
class SocketsBinding
```

`[sockets]` binding (`env.SOCKETS`): dial host-held WebSockets.

Every URL must pass the plugin's network grant. Messages arrive on the
default entrypoint's `web_socket_message(ws, message)`, closes on
`web_socket_close(ws, code, reason, was_clean)`.

#### SocketsBinding.__init__

```python
def __init__(self, stub: Any) -> None
```

#### SocketsBinding.connect

```python
async def connect(
    self,
    url: str,
    *,
    protocols: list[str] | None=None,
    headers: dict[str, str] | list[tuple[str, str]] | None=None,
    auto_responses: list[tuple[str | bytes, str | bytes]] | None=None,
    attachment: Any=None,
) -> HostWebSocket
```

Open one socket the host holds.

**Args**

- `url`: `wss://` or `ws://` URL on the network grant.
- `protocols`: Requested subprotocols.
- `headers`: Extra request headers (values may be credentials).
- `auto_responses`: `(request, response)` heartbeat pairs the host answers without waking the guest.
- `attachment`: Initial JSON attachment.

**Returns**

The socket handle.

#### SocketsBinding.get_web_sockets

```python
async def get_web_sockets(self) -> list[HostWebSocket]
```

The plugin's open sockets, oldest first.

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

#### StorageEntrypoint.move

```python
async def move(self, _from_key: str, _to_key: str, _options: Any=None)
```

Move one object (declare `storage.move`).

`options` may carry `noOverwrite` and `ifSourceMatch`. Returns
`{"atomic": bool, "alreadyMoved": bool}`; `alreadyMoved` reports a
retried move whose source is gone and whose target exists.

#### StorageEntrypoint.moveMany

```python
async def moveMany(self, _request: Any)
```

Move explicit pairs or one page of a prefix (declare `storage.moveMany`).

`request["target"]` is `{"kind": "pairs", "value": [...]}` (at
most `MAX_BATCH_KEYS`) or `{"kind": "prefix", "value": {...}}`,
paged through `cursor`. Returns ``{"results", "nextCursor",
"atomic", "objectsMoved"}``, one result per pair in order.

#### StorageEntrypoint.deleteMany

```python
async def deleteMany(self, _keys: list[str])
```

Delete up to `MAX_BATCH_KEYS` objects (declare `storage.deleteMany`).

Returns one `{"key", "error"?}` outcome per key, in order; a
missing object counts as deleted.

#### StorageEntrypoint.headMany

```python
async def headMany(self, _keys: list[str])
```

Metadata for up to `MAX_BATCH_KEYS` objects (declare `storage.headMany`).

Returns one entry per key, in order: object metadata or `None`.

#### StorageEntrypoint.changes

```python
async def changes(self, _request: Any)
```

One page of the change feed after a cursor (declare `storage.changes`).

Returns `{"changes", "cursor", "more", "complete"}`; `complete`
is false when changes may have been missed and the host must relist.

#### StorageEntrypoint.translateKeys

```python
async def translateKeys(self, _keys: list[str])
```

Translate up to `MAX_BATCH_KEYS` host keys (declare `storage.translateKeys`).

A host key is `/`-joined segments, none empty, `.`, `..` or
holding a backslash or a control character. Returns one relative key
per host key, in order: the key this destination stores it under.
The same host key must always translate to the same key; without the
feature the host stores each host key as written.

### StorefrontEntrypoint

```python
class StorefrontEntrypoint(NamedEntrypoint)
```

`storefront` entrypoint (`ContentSource` interface).

Every method takes and returns the generated [`bookclerk_plugin_sdk.abi`](abi.md)
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

Return one page of one account's library (`ScanParams` → `ScanPage`).

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

`params["lookup"]` says how the host names the title (`"product"`,
`"isbn"`, `"asin"`, `"title"` or `"unspecified"`). For a
`"title"` lookup, return `{"hint": ..., "records": [...]}` with
every priced listing the search matched (each ``{"record":
CatalogHit, "hint": PurchaseHint}``) so the host can keep the
recording it asked for; a bare `PurchaseHint` or `None` also works.

#### StorefrontEntrypoint.purchaseHints

```python
async def purchaseHints(self, params: abi.PurchaseHintsParams)
```

Purchase links / price hints for several titles at once.

Asks [`purchaseHint`](#storefrontentrypointpurchasehint) for each query,
`PURCHASE_HINTS_CONCURRENCY` at a time, those that already name
a product id first (one lookup each) and title searches after, and
starts nothing once `PURCHASE_HINTS_BUDGET_SECONDS` has passed,
so the host's batch gets what the store managed instead of nothing.
A store whose catalog prices many products per request overrides
this to make one request. Raise a `rate_limited`
[`PluginError`](#pluginerror) (with `details.retryAfterMs`) when the store
asks to slow down.

**Args**

- `params`: The titles to price, in order.

**Returns**

One `PurchaseHint` or `None` per query.

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

#### StorefrontEntrypoint.qualityOffers

```python
async def qualityOffers(self, _params: abi.QualityOffersParams) -> abi.QualityOffersResult
```

Audio quality one account would download for each title.

One answer per title id in order (`found` false when the store
cannot say); it must not license or download anything. The host asks
only a store that declares `catalog.qualityOffers`; the default
raises unsupported (`QualityOffersParams` → `QualityOffersResult`).

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

### WebhookMessage

```python
@dataclass(frozen=True)
class WebhookMessage
```

One stored webhook request, leased to this pull.

**Attributes**

- `lease_id`: Pass to [`WebhooksBinding.ack`](#webhooksbindingack) to finish or retry it.
- `timestamp`: When the host received it.
- `attempts`: How many pulls have leased it, this one included.
- `headers`: The kept headers, lowercase names.
- `body`: The request body as sent.

### WebhookPullResult

```python
@dataclass(frozen=True)
class WebhookPullResult
```

One pull's leased requests.

**Attributes**

- `messages`: Leased requests, oldest first.
- `missed`: True when requests expired or were dropped since the last pull, so the plugin should resynchronize another way.

### WebhooksBinding

```python
class WebhooksBinding
```

`[[webhooks]]` binding (`env.WEBHOOKS`).

Requests outside services send to the webhook's URL wait in the library
until the plugin pulls them; each pull leases a batch, and `ack`
finishes or retries each lease.

#### WebhooksBinding.__init__

```python
def __init__(self, stub: Any) -> None
```

#### WebhooksBinding.url

```python
async def url(self, name: str) -> str | None
```

The URL an outside service should call for webhook `name`.

**Returns**

The URL, or `None` when the library cannot be reached from the
Internet or webhooks are turned off.

#### WebhooksBinding.pull

```python
async def pull(
    self,
    name: str,
    *,
    batch_size: int=DEFAULT_WEBHOOK_PULL_BATCH,
    visibility_timeout_ms: int=0,
) -> WebhookPullResult
```

Lease up to `batch_size` waiting requests, oldest first.

**Args**

- `name`: The `[[webhooks]]` row.
- `batch_size`: 1 to `MAX_WEBHOOK_PULL_BATCH`.
- `visibility_timeout_ms`: How long leased requests stay hidden from the next pull; 0 means one minute.

**Returns**

The leased requests and whether any were missed.

**Raises**

- `PluginError`: `invalid_params` for an out-of-range option.

#### WebhooksBinding.ack

```python
async def ack(
    self,
    name: str,
    *,
    acks: list[str] | None=None,
    retries: list[str] | None=None,
) -> None
```

Finish or retry leased requests.

**Args**

- `name`: The `[[webhooks]]` row.
- `acks`: Leases whose requests are done and deleted.
- `retries`: Leases whose requests the next pull offers again.

**Raises**

- `PluginError`: `invalid_params` for a malformed lease, too many leases, or a lease in both lists.

## Functions

### pack_embedding_vectors

```python
def pack_embedding_vectors(vectors: Sequence[Sequence[float]]) -> bytes
```

Pack embedding vectors into the wire form of `AiTextEmbeddingsOutput["vectors"]`.

**Args**

- `vectors`: One vector per input text, all of one length.

**Returns**

Little-endian 32-bit floats, concatenated.

**Raises**

- `PluginError`: `internal` when the vectors differ in length.

### unpack_embedding_vectors

```python
def unpack_embedding_vectors(data: bytes, dimensions: int) -> list[list[float]]
```

Split the wire form of `AiTextEmbeddingsOutput["vectors"]` into vectors.

**Args**

- `data`: Little-endian 32-bit floats, concatenated.
- `dimensions`: Numbers per vector.

**Returns**

One vector per `dimensions` numbers.

**Raises**

- `PluginError`: `internal` when the bytes are not whole vectors.

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

### decode_socket_attachment

```python
def decode_socket_attachment(data: bytes) -> Any
```

Decode stored socket attachment bytes.

**Args**

- `data`: Bytes the host kept.

**Returns**

The JSON value, `None` when empty, or the text itself when the bytes
are not JSON (an attachment another SDK stored).

### encode_socket_attachment

```python
def encode_socket_attachment(value: Any) -> bytes
```

Encode a socket attachment as the host stores it: UTF-8 JSON.

**Args**

- `value`: JSON-serializable value; `None` stores no bytes.

**Returns**

At most `MAX_SOCKET_ATTACHMENT_BYTES` bytes.

**Raises**

- `PluginError`: `invalid_params` when JSON cannot encode the value, `payload_too_large` when it does not fit.

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

Build named [`DatabaseBinding`](db_value.md#databasebinding) objects over `POST /db/execute`.

Each entry of `database_tokens` (binding name → per-invocation grant
token) becomes an isolated D1-shaped binding whose terminal methods
(`first`, `run`, `all`, `batch`) are async and route through the
granted transport — the same objects the adapter installs on `env`.

**Args**

- `granted`: Granted-channel fetcher (`GRANTED` service binding).
- `database_tokens`: Binding name → grant token.
- `signal`: Optional `AbortSignal` forwarded to every request.

**Returns**

Binding name → [`DatabaseBinding`](db_value.md#databasebinding).

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
`bytes` at any depth); Python values pass through unchanged.

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

### socket_message_data

```python
def socket_message_data(frame: Any) -> str | bytes
```

The message a handler receives for one wire frame.

**Args**

- `frame`: Decoded wire `SocketFrame` (`{"kind", "value"}`).

**Returns**

`str` for a text frame, `bytes` for a binary one.

### transport_database

```python
def transport_database(transport: Any) -> DatabaseBinding
```

Wrap an adapter `GrantedDatabaseTransport` stub as a [`DatabaseBinding`](db_value.md#databasebinding).

**Args**

- `transport`: Stub exposing `executeBytes(bytes) -> bytes`, `executeStreamedBytes(bytes) -> bytes` and `limitsBytes() -> bytes`.

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

### transport_sockets

```python
def transport_sockets(stub: Any) -> SocketsBinding
```

Wrap the adapter's granted sockets stub as a [`SocketsBinding`](#socketsbinding).

**Args**

- `stub`: `SocketsTransport`-shaped JS stub from the granted context.

**Returns**

The author-facing binding.

### transport_webhooks

```python
def transport_webhooks(stub: Any) -> WebhooksBinding
```

Wrap the adapter's granted webhooks stub as a [`WebhooksBinding`](#webhooksbinding).

**Args**

- `stub`: `WebhooksTransport`-shaped JS stub from the granted context.

**Returns**

The author-facing binding.

## Constants

### FEATURE_SCALAR_LIMITS

```python
FEATURE_SCALAR_LIMITS: str = 'rpc.scalarLimits'
```

Guest honors scalar / stream-window / list-page caps (`rpc.scalarLimits`).

### FEATURE_STORAGE_CHANGES

```python
FEATURE_STORAGE_CHANGES: str = 'storage.changes'
```

Guest implements `Destination.changes` (`storage.changes`).

### FEATURE_STORAGE_CONDITIONAL_WRITE

```python
FEATURE_STORAGE_CONDITIONAL_WRITE: str = 'storage.conditionalWrite'
```

Guest enforces `WriteOptions.ifMatch` / `ifNoneMatch` (`storage.conditionalWrite`).

### FEATURE_STORAGE_COPY

```python
FEATURE_STORAGE_COPY: str = 'storage.copy'
```

Guest implements server-side `Destination.copy` (`storage.copy`).

### FEATURE_STORAGE_DELETE_MANY

```python
FEATURE_STORAGE_DELETE_MANY: str = 'storage.deleteMany'
```

Guest implements `Destination.deleteMany` (`storage.deleteMany`).

### FEATURE_STORAGE_HEAD_MANY

```python
FEATURE_STORAGE_HEAD_MANY: str = 'storage.headMany'
```

Guest implements `Destination.headMany` (`storage.headMany`).

### FEATURE_STORAGE_MOVE

```python
FEATURE_STORAGE_MOVE: str = 'storage.move'
```

Guest implements `Destination.move` (`storage.move`).

### FEATURE_STORAGE_MOVE_MANY

```python
FEATURE_STORAGE_MOVE_MANY: str = 'storage.moveMany'
```

Guest implements `Destination.moveMany` (`storage.moveMany`).

### FEATURE_STREAMS

```python
FEATURE_STREAMS: str = 'rpc.streams'
```

Media moves through transferred `ByteRange` / `ByteSource` streams (`rpc.streams`).

### MAX_BATCH_KEYS

```python
MAX_BATCH_KEYS: int = 256
```

Maximum keys or key pairs in one `Destination.moveMany`, `deleteMany` or
`headMany` call, and maximum outcomes in one reply.

### MAX_CHANGES_CURSOR_BYTES

```python
MAX_CHANGES_CURSOR_BYTES: int = 4096
```

Maximum bytes of a `Destination.changes` cursor.

### MAX_CHANGES_PAGE

```python
MAX_CHANGES_PAGE: int = 256
```

Maximum changes in one `Destination.changes` page.

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

### MAX_SOCKET_ATTACHMENT_BYTES

```python
MAX_SOCKET_ATTACHMENT_BYTES: int = 2048
```

Largest per-socket attachment the host keeps for a sleeping guest.

### MAX_SOCKET_FRAME_BYTES

```python
MAX_SOCKET_FRAME_BYTES: int = 262144
```

Largest WebSocket message, text or binary, that `Sockets.send` accepts or
`SocketHandler.message` delivers.

### MAX_STREAM_WINDOW_BYTES

```python
MAX_STREAM_WINDOW_BYTES: int = 1048576
```

Maximum bytes returned by one `ByteSource.pull` (flow-control window).

### MAX_WEBHOOK_LEASE_ID_BYTES

```python
MAX_WEBHOOK_LEASE_ID_BYTES: int = 128
```

Longest `WebhookDelivery.leaseId`, in bytes.

### MAX_WEBHOOK_NAME_BYTES

```python
MAX_WEBHOOK_NAME_BYTES: int = 32
```

Longest `[[webhooks]]` name, in bytes.

### MAX_WEBHOOK_PULL_BATCH

```python
MAX_WEBHOOK_PULL_BATCH: int = 100
```

Most deliveries one `Webhooks.pull` returns.

### MAX_WEBHOOK_VISIBILITY_MS

```python
MAX_WEBHOOK_VISIBILITY_MS: int = 43200000
```

Longest `Webhooks.pull` visibility timeout, in milliseconds (12 hours).

### PRODUCT_API_VERSION

```python
PRODUCT_API_VERSION: int = 3
```

Product ABI version (`plugin.toml` `api_version` / `describe().apiVersion`).
