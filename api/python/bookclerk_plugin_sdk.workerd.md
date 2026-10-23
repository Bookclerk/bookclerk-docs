# bookclerk\_plugin\_sdk.workerd

Workerd author runtime — ``BookclerkEntrypoint`` and the named entrypoints.

Python mirror of ``@bookclerk/plugin-sdk/workerd`` (``embed/bookclerk_plugin.js``)
for ``api_version = 3`` guests running as Python Workers:

- ``from bookclerk_plugin_sdk.workerd import BookclerkEntrypoint, js``

The default export is a module-level ``Default`` class extending
:class:`BookclerkEntrypoint` with optional ``event(batch)`` / ``job(job)``
triggers. Named entrypoints from ``plugin.toml`` ``entrypoints`` are module-level
classes named after the wire name (``Storefront``, ``Storage``, ``RemoteLibrary``,
``DatabaseAdapter``, ``Cli``, ``Oidc``) extending the matching ``*Entrypoint``
base. The launcher's adapter isolate reaches them only through the
``bookclerk*`` dispatch methods; authors neither call nor override those.

Inside a Python Workers isolate, ``bookclerk-workerd`` injects this module under
``bookclerk_plugin_sdk.workerd`` — authors do not vendor a relative filepath.
Outside the isolate (authoring / unit tests), lightweight stubs stand in for
``js`` / ``workers`` imports and :func:`js` returns plain Python values.

#### js

```python
def js(value: Any) -> Any
```

Convert a Python value to a JS value for Workers RPC results.

Inside the isolate this uses ``pyodide.ffi.to_js`` with plain-object
dictionaries (``bytes`` become ``Uint8Array``); when ``to_js`` is not
available it falls back to ``JSON.parse(json.dumps(value))``. Under the
authoring stubs the value is returned unchanged.

**Arguments**:

- `value` - JSON-serializable Python value (``bytes`` allowed for ``Data``).
  

**Returns**:

  The JS value (or the original Python value under the authoring stubs).

#### py

```python
def py(value: Any) -> Any
```

Convert a JS value delivered over Workers RPC into plain Python.

``JsProxy`` objects are converted with ``to_py()`` (``Uint8Array`` →
``bytes``); Python values pass through unchanged.

**Arguments**:

- `value` - A JsProxy or Python value.
  

**Returns**:

  A plain Python value (``dict`` / ``list`` / ``bytes`` / scalars).

## PluginError

```python
class PluginError(RuntimeError)
```

SDK-thrown failure. Unknown wire codes stay on ``wire_code``.

#### PluginError.from\_wire

```python
@classmethod
def from_wire(cls, code: str, message: str) -> PluginError
```

Build a ``PluginError`` from a wire ``code`` string.

**Arguments**:

- `code` - Snake_case wire code (unknown codes are kept on ``wire_code``).
- `message` - Operator-facing error text.
  

**Returns**:

  A ``PluginError`` whose ``code`` is a known variant or ``unknown``.

#### decode\_extensible\_config

```python
def decode_extensible_config(cfg: Any) -> Any
```

Decode an ``ExtensibleConfig`` into the plain value on ``env.CONFIG``.

**Arguments**:

- `cfg` - ``{"schemaVersion", "mediaType", "payload"}`` (JS or Python).
  

**Returns**:

  The parsed JSON object for JSON media types (``{}`` when empty or
  malformed); ``{"mediaType", "text"}`` for other media types.

#### json\_payload

```python
def json_payload(value: Any) -> dict[str, Any]
```

Wrap a JSON-serializable value as an ``application/json`` ``ExtensibleConfig``.

This is the shape of ``CliInvokeResult.payload`` and every other extensible
payload on the wire (schema version 1).

**Arguments**:

- `value` - JSON-serializable Python value.
  

**Returns**:

- ```{"schemaVersion"` - 1, "mediaType": "application/json", "payload": bytes}``.

#### cli\_args

```python
def cli_args(params: Any) -> dict[str, str]
```

Turn ``CliInvokeParams.args`` (``[{name, value}]``) into a ``{name: value}`` dict.

**Arguments**:

- `params` - ``CliInvokeParams`` (JS or Python).
  

**Returns**:

  Argument values keyed by name.

## \_EnvView

```python
class _EnvView()
```

Per-invocation ``env``: granted bindings layered over the isolate's static env.

#### \_EnvView.get

```python
def get(name: str, default: Any = None) -> Any
```

Read a binding, returning ``default`` when it is not granted.

## EventMessage

```python
class EventMessage()
```

One delivered domain event.

Exactly one outcome is recorded per message; the first of ``ack`` /
``retry`` / ``reject`` / ``dead_letter`` / ``suspend`` wins and later calls
are ignored (as with Workers queue messages). Messages left untouched are
acked when ``event()`` returns and retried when it raises.

#### EventMessage.json

```python
def json() -> Any
```

Decode ``body`` as JSON (``{}`` for an empty body).

**Returns**:

  The parsed payload.

#### EventMessage.result

```python
@property
def result() -> dict[str, Any] | None
```

Recorded wire ``EventResult`` (flat ``{"kind": ...}``), or ``None`` while undecided.

#### EventMessage.ack

```python
def ack() -> None
```

Mark handled; the host marks the event delivered.

#### EventMessage.retry

```python
def retry(*,
          retry_at: datetime | int | float | None = None,
          delay_seconds: float = 0,
          reason: str = "") -> None
```

Redeliver later.

**Arguments**:

- `retry_at` - Explicit earliest redelivery instant (``datetime`` or Unix ms).
- `delay_seconds` - Delay from now; ignored when ``retry_at`` is set.
- `reason` - Short operator-facing reason.

#### EventMessage.reject

```python
def reject(reason: str = "") -> None
```

Stop delivering; the host records ``reason``.

**Arguments**:

- `reason` - Short operator-facing reason.

#### EventMessage.dead\_letter

```python
def dead_letter(reason: str = "") -> None
```

Park for operator review.

**Arguments**:

- `reason` - Short operator-facing reason.

#### EventMessage.suspend

```python
def suspend(*,
            checkpoint: Any = None,
            checkpoint_schema_version: int = 1,
            wake_at: datetime | int | float | None = None,
            wake_on_event_type: str = "",
            wake_on_filter: Any = None) -> None
```

Release with a bounded checkpoint; the host redelivers later.

**Arguments**:

- `checkpoint` - JSON-serializable checkpoint (at most ``MAX_CHECKPOINT_BYTES``).
- `checkpoint_schema_version` - Schema version of ``checkpoint``.
- `wake_at` - Earliest resume instant (``datetime`` or Unix ms).
- `wake_on_event_type` - Also wake when an event of this type arrives.
- `wake_on_filter` - Host-owned payload filter for wake events.
  

**Raises**:

- `PluginError` - ``payload_too_large`` when the checkpoint exceeds the cap.

## EventBatch

```python
class EventBatch()
```

One ``event(batch)`` delivery (``EventBatch`` struct), Workers ``MessageBatch``-shaped.

#### EventBatch.type

```python
@property
def type() -> str
```

Event type shared by the batch (``""`` when mixed or empty).

#### EventBatch.ack\_all

```python
def ack_all() -> None
```

Acknowledge every message.

#### EventBatch.retry\_all

```python
def retry_all(**options: Any) -> None
```

Retry every message with the same :meth:`EventMessage.retry` options.

**Arguments**:

- `**options` - Keyword arguments forwarded to :meth:`EventMessage.retry`.

#### event\_batch\_results

```python
def event_batch_results(batch: EventBatch,
                        failed: BaseException | None) -> list[dict[str, Any]]
```

Translate recorded outcomes into the wire ``EventResult`` list.

**Arguments**:

- `batch` - The delivered batch.
- `failed` - Exception raised by the author's ``event()``, if any; undecided
  messages then retry instead of ack.
  

**Returns**:

  One flat ``{"kind": ...}`` result per message, in delivery order.

## QueueMessage

```python
class QueueMessage()
```

One delivered queue message (``QueueMessage`` struct), Workers ``Message``-shaped.

Exactly one outcome is recorded per message; the first of ``ack`` /
``retry`` wins and later calls are ignored. Messages left untouched are
acked when ``queue()`` returns and retried when it raises.

#### QueueMessage.json

```python
def json() -> Any
```

Decode ``body`` as JSON (``None`` for an empty body).

**Returns**:

  The parsed body.

#### QueueMessage.text

```python
def text() -> str
```

Decode ``body`` as UTF-8 text.

**Returns**:

  The body text.

#### QueueMessage.result

```python
@property
def result() -> dict[str, Any] | None
```

Recorded outcome (flat ``{"kind": ...}``), or ``None`` while undecided.

#### QueueMessage.ack

```python
def ack() -> None
```

Mark handled; the host marks the message delivered.

#### QueueMessage.retry

```python
def retry(*, delay_seconds: float = 0, reason: str = "") -> None
```

Redeliver later.

**Arguments**:

- `delay_seconds` - Seconds before redelivery; ``0`` applies the
  consumer's ``retry_delay``.
- `reason` - Short operator-facing reason.

## QueueBatch

```python
class QueueBatch()
```

One ``queue(batch)`` delivery (``QueueBatch`` struct), Workers ``MessageBatch``-shaped.

#### QueueBatch.ack\_all

```python
def ack_all() -> None
```

Acknowledge every message.

#### QueueBatch.retry\_all

```python
def retry_all(**options: Any) -> None
```

Retry every message with the same :meth:`QueueMessage.retry` options.

**Arguments**:

- `**options` - Keyword arguments forwarded to :meth:`QueueMessage.retry`.

#### queue\_batch\_results

```python
def queue_batch_results(batch: QueueBatch,
                        failed: BaseException | None) -> list[dict[str, Any]]
```

Translate recorded outcomes into the wire ``QueueResult`` list.

**Arguments**:

- `batch` - The delivered batch.
- `failed` - Exception raised by the author's ``queue()``, if any; undecided
  messages then retry instead of ack.
  

**Returns**:

  One flat ``{"kind": ...}`` result per message, in delivery order.

## QueueBinding

```python
class QueueBinding()
```

``[[queues.producers]]`` binding (``env.<BINDING>``): Workers ``Queue``-shaped producer.

The adapter isolate owns the queue grant and serves ``send`` over the
granted channel; this wrapper converts Python bodies into the JS values
the adapter stub expects.

#### QueueBinding.send

```python
async def send(body: Any,
               *,
               content_type: str = "json",
               delay_seconds: int = 0) -> dict[str, Any]
```

Send one message.

**Arguments**:

- `body` - JSON value (default), ``str`` for ``text``, or ``bytes`` for ``bytes``.
- `content_type` - ``json``, ``text``, or ``bytes``.
- `delay_seconds` - Seconds before the message may be delivered.
  

**Returns**:

- ```{"messageIds"` - [...]}`` with the outbox id the host assigned.

#### QueueBinding.send\_batch

```python
async def send_batch(messages: list[dict[str, Any]],
                     *,
                     delay_seconds: int = 0) -> dict[str, Any]
```

Send up to ``MAX_LIST_PAGE`` messages in order.

**Arguments**:

- `messages` - ``{"body": ..., "content_type": ..., "delay_seconds": ...}`` entries.
- `delay_seconds` - Default delay for entries that carry none.
  

**Returns**:

- ```{"messageIds"` - [...]}`` with the outbox ids in send order.

#### transport\_queue

```python
def transport_queue(stub: Any) -> QueueBinding
```

Wrap the adapter's granted queue stub as a :class:`QueueBinding`.

**Arguments**:

- `stub` - ``QueueBinding``-shaped JS stub from the granted context.
  

**Returns**:

  The author-facing binding.

## KvNamespace

```python
class KvNamespace()
```

``[[kv_namespaces]]`` binding (``env.<BINDING>``): Workers ``KVNamespace``-shaped storage.

The host serves the namespace from the library database, scoped to this
plugin and the namespace the row names, so values follow the instance
across hosts. The adapter isolate owns the kv grant; this wrapper converts
Python values into the JS values the adapter stub expects.

#### KvNamespace.get

```python
async def get(key: str, type: str = "text") -> Any
```

Read one key.

**Arguments**:

- `key` - Key, at most ``MAX_KV_KEY_BYTES`` UTF-8 bytes.
- `type` - ``text`` (``str``), ``json`` (decoded value), or ``bytes``.
  

**Returns**:

  The value, or ``None`` when absent or expired.

#### KvNamespace.get\_with\_metadata

```python
async def get_with_metadata(key: str, type: str = "text") -> tuple[Any, Any]
```

Read one key with its metadata.

**Arguments**:

- `key` - Key to read.
- `type` - ``text``, ``json``, or ``bytes``.
  

**Returns**:

  ``(value, metadata)``; both ``None`` when absent.

#### KvNamespace.put

```python
async def put(key: str,
              value: str | bytes,
              *,
              expiration: int | None = None,
              expiration_ttl: int | None = None,
              metadata: Any = None) -> None
```

Write one key, replacing any previous value and metadata.

**Arguments**:

- `key` - Key to write.
- `value` - ``str`` (stored as UTF-8) or ``bytes``.
- `expiration` - Absolute expiry in seconds since the Unix epoch.
- `expiration_ttl` - Relative expiry in seconds from the write.
- `metadata` - JSON-serializable metadata.

#### KvNamespace.delete

```python
async def delete(key: str) -> None
```

Delete one key; a missing key is not an error.

#### KvNamespace.list

```python
async def list(*,
               prefix: str = "",
               cursor: str = "",
               limit: int = 0) -> dict[str, Any]
```

List keys in lexical order, one page at a time.

**Arguments**:

- `prefix` - Only keys starting with this prefix.
- `cursor` - Cursor from a previous page.
- `limit` - Most keys to return (host default when 0).
  

**Returns**:

- ```{"keys"` - [{"name", "expiration"?, "metadata"?}], "list_complete": bool, "cursor"?: str}``.

#### transport\_kv

```python
def transport_kv(stub: Any) -> KvNamespace
```

Wrap the adapter's granted KV stub as a :class:`KvNamespace`.

**Arguments**:

- `stub` - ``KvNamespaceBinding``-shaped JS stub from the granted context.
  

**Returns**:

  The author-facing binding.

## JobController

```python
class JobController()
```

Everything one job invocation may touch.

Carries the durable envelope, ``input`` / ``output`` streams, ``progress()``,
``cancelled``, and the terminal-outcome recorders ``suspend()`` /
``retry_later()``. Returning normally without a recorded outcome completes
the job; raising rejects it (``unavailable`` / ``deadline_exceeded`` errors
are retryable, ``cancelled`` is cancelled).

#### JobController.json

```python
def json() -> Any
```

Decode ``payload_json`` (``{}`` when empty).

**Returns**:

  The parsed job payload.

#### JobController.progress

```python
async def progress(percent: float, message: str = "") -> None
```

Report ``percent`` in ``0..=100`` with an operator-facing ``message``.

**Arguments**:

- `percent` - Completion percentage.
- `message` - Operator-facing status text.

#### JobController.result

```python
@property
def result() -> dict[str, Any] | None
```

Recorded terminal outcome (flat ``{"kind": ...}``), or ``None`` while running.

#### JobController.suspend

```python
def suspend(*,
            checkpoint: Any = None,
            checkpoint_schema_version: int = 1,
            wake_at: datetime | int | float | None = None) -> None
```

Release with a bounded checkpoint; the host resumes at ``wake_at``.

**Arguments**:

- `checkpoint` - JSON-serializable checkpoint (at most ``MAX_CHECKPOINT_BYTES``).
- `checkpoint_schema_version` - Schema version of ``checkpoint``.
- `wake_at` - Earliest resume instant (``datetime`` or Unix ms).
  

**Raises**:

- `PluginError` - ``payload_too_large`` when the checkpoint exceeds the cap.

#### JobController.retry\_later

```python
def retry_later(*,
                retry_at: datetime | int | float | None = None,
                reason: str = "") -> None
```

Give up this attempt and let the host retry at ``retry_at`` (or its default).

**Arguments**:

- `retry_at` - Earliest retry instant (``datetime`` or Unix ms).
- `reason` - Short operator-facing reason.

#### JobController.cancel

```python
def cancel() -> None
```

Host-side cancellation observed (adapter-internal).

#### job\_outcome\_for

```python
def job_outcome_for(job: JobController, returned: Any,
                    failed: BaseException | None) -> dict[str, Any]
```

Translate a ``job()`` return / raise into the wire ``JobOutcome``.

**Arguments**:

- `job` - The controller handed to the author.
- `returned` - The author's return value (``{"message", "bytesCopied"}`` or ``None``).
- `failed` - Exception raised by the author, if any.
  

**Returns**:

  A flat ``{"kind": ...}`` outcome record.

## BookclerkEntrypoint

```python
class BookclerkEntrypoint(WorkerEntrypoint)
```

Default-export base for ``api_version = 3`` Python Workers guests.

Optional trigger methods: ``event(batch)`` (for ``[[events.consumers]]``)
and ``job(job)`` (for ``[triggers] jobs``); optional ``describe()``
(presentation fields beyond ``plugin.toml``), ``database_migrations(binding)``,
and ``shutdown()``. Return plain Python values; the SDK converts them.

The ``bookclerk*`` methods are the adapter-facing dispatch surface; authors
neither call nor override them.

#### BookclerkEntrypoint.fetch

```python
async def fetch(_request=None)
```

Reject HTTP fetch — workerd guests are Workers-RPC only.

**Arguments**:

- `_request` - Incoming HTTP request (unused by the default stub).
  

**Returns**:

  A 404 ``Response``.

#### BookclerkEntrypoint.bookclerkDescribe

```python
async def bookclerkDescribe()
```

Adapter dispatch: the author's optional ``describe()`` (or ``None``).

#### BookclerkEntrypoint.bookclerkEvent

```python
async def bookclerkEvent(context=None, wire_batch=None)
```

Adapter dispatch: deliver one ``EventBatch`` to ``event(batch)``.

**Raises**:

- `PluginError` - ``unsupported`` when the author defines no ``event``.

#### BookclerkEntrypoint.bookclerkQueue

```python
async def bookclerkQueue(context=None, wire_batch=None)
```

Adapter dispatch: deliver one ``QueueBatch`` to ``queue(batch)``.

**Raises**:

- `PluginError` - ``unsupported`` when the author defines no ``queue``.

#### BookclerkEntrypoint.bookclerkJob

```python
async def bookclerkJob(context=None, invocation=None, granted=None)
```

Adapter dispatch: run one durable job through ``job(controller)``.

**Raises**:

- `PluginError` - ``unsupported`` when the author defines no ``job``.

#### BookclerkEntrypoint.bookclerkDatabaseMigrations

```python
async def bookclerkDatabaseMigrations(binding="")
```

Adapter dispatch: the author's ``database_migrations(binding)`` (or ``[]``).

#### BookclerkEntrypoint.bookclerkShutdown

```python
async def bookclerkShutdown()
```

Adapter dispatch: the author's optional ``shutdown()``.

## NamedEntrypoint

```python
class NamedEntrypoint(WorkerEntrypoint)
```

Base for named entrypoints.

Subclasses list their RPC surface on ``bookclerk_methods``; the adapter
reaches them only through :meth:`bookclerkInvoke`, which installs the
granted bindings on ``env`` before dispatching. Method arguments arrive as
plain Python values (``py``) and return values are converted with ``js``.

#### NamedEntrypoint.fetch

```python
async def fetch(_request=None)
```

Reject HTTP fetch — workerd guests are Workers-RPC only.

**Arguments**:

- `_request` - Incoming HTTP request (unused by the default stub).
  

**Returns**:

  A 404 ``Response``.

#### NamedEntrypoint.bookclerkInvoke

```python
async def bookclerkInvoke(context=None, method="", *args)
```

Adapter dispatch: call ``method`` with the granted bindings installed.

**Raises**:

- `PluginError` - ``unsupported`` for methods outside ``bookclerk_methods``.

#### STOREFRONT\_METHODS

RPC surface of the ``storefront`` entrypoint.

## StorefrontEntrypoint

```python
class StorefrontEntrypoint(NamedEntrypoint)
```

``storefront`` entrypoint (``ContentSource`` interface).

Every method takes and returns the generated :mod:`bookclerk_plugin_sdk.abi`
TypedDict shapes as plain dicts (``LoginParams`` → ``LoginResult``, …).
Override only the operations the storefront supports; the rest raise
``unsupported``.

#### StorefrontEntrypoint.login

```python
async def login(_params: abi.LoginParams)
```

Password or one-shot OAuth login (``LoginParams`` → ``LoginResult``).

#### StorefrontEntrypoint.scan

```python
async def scan(_params: abi.ScanParams)
```

Enumerate owned titles (``ScanParams`` → ``ScanSummary``).

#### StorefrontEntrypoint.fetchTitle

```python
async def fetchTitle(_params: abi.FetchTitleParams)
```

Fetch one title's media and metadata (``FetchTitleParams`` → ``PlainFetch``).

#### StorefrontEntrypoint.listAccounts

```python
async def listAccounts()
```

List accounts known to this storefront (list of ``SourceAccount``).

#### StorefrontEntrypoint.loginStart

```python
async def loginStart(_params: abi.LoginParams)
```

Begin an interactive OAuth login (``LoginParams`` → ``LoginStartResult``).

#### StorefrontEntrypoint.loginComplete

```python
async def loginComplete(_params: abi.LoginCompleteParams)
```

Finish an interactive OAuth login (``LoginCompleteParams`` → ``LoginResult``).

#### StorefrontEntrypoint.searchCatalog

```python
async def searchCatalog(_params: abi.SearchCatalogParams)
```

Free-text catalog search (``SearchCatalogParams`` → list of ``CatalogHit``).

#### StorefrontEntrypoint.expandCandidates

```python
async def expandCandidates(_params: abi.ExpandCandidatesParams)
```

Related-title expansion (``ExpandCandidatesParams`` → list of ``CatalogHit``).

#### StorefrontEntrypoint.purchaseHint

```python
async def purchaseHint(_params: abi.PurchaseHintParams)
```

Purchase link / price hint (``PurchaseHintParams`` → ``PurchaseHint``).

#### StorefrontEntrypoint.listDeals

```python
async def listDeals(_params: abi.ListDealsParams)
```

Current storefront deals (``ListDealsParams`` → list of ``CatalogHit``).

#### StorefrontEntrypoint.catalogDetail

```python
async def catalogDetail(_params: abi.CatalogDetailParams)
```

Full catalog record (``CatalogDetailParams`` → ``CatalogHit``).

#### StorefrontEntrypoint.health

```python
async def health()
```

Liveness / readiness probe (``HealthOk``).

#### StorefrontEntrypoint.diagnose

```python
async def diagnose()
```

Operator-facing diagnostic lines (list of strings).

#### STORAGE\_METHODS

RPC surface of the ``storage`` entrypoint.

## StorageEntrypoint

```python
class StorageEntrypoint(NamedEntrypoint)
```

``storage`` entrypoint (``Destination`` interface): an object store the host writes to.

#### StorageEntrypoint.head

```python
async def head(_key: str)
```

Object metadata or ``None``.

#### StorageEntrypoint.list

```python
async def list(_options: Any)
```

One page of objects under a prefix.

#### StorageEntrypoint.get

```python
async def get(_key: str, _options: Any = None)
```

Read an object (optionally a byte range).

#### StorageEntrypoint.put

```python
async def put(_key: str, _body: Any, _options: Any = None)
```

Write an object from a byte stream.

#### StorageEntrypoint.copy

```python
async def copy(_from_key: str, _to_key: str)
```

Server-side copy.

#### StorageEntrypoint.delete

```python
async def delete(_key: str)
```

Delete an object.

#### StorageEntrypoint.commit

```python
async def commit(_key: str, _commit_token: str)
```

Publish a staged object.

#### StorageEntrypoint.abortStage

```python
async def abortStage(_key: str, _commit_token: str)
```

Discard a staged object.

#### REMOTE\_LIBRARY\_METHODS

RPC surface of the ``remoteLibrary`` entrypoint.

## RemoteLibraryEntrypoint

```python
class RemoteLibraryEntrypoint(NamedEntrypoint)
```

``remoteLibrary`` entrypoint: lifecycle, rescan, listening sync, user polling.

#### RemoteLibraryEntrypoint.health

```python
async def health()
```

Liveness / readiness probe (``HealthOk``).

#### RemoteLibraryEntrypoint.diagnose

```python
async def diagnose()
```

Operator-facing diagnostic lines (list of strings).

#### RemoteLibraryEntrypoint.start

```python
async def start()
```

Start background work after the host has granted bindings.

#### RemoteLibraryEntrypoint.stop

```python
async def stop()
```

Stop background work.

#### RemoteLibraryEntrypoint.scanLibrary

```python
async def scanLibrary(_params: abi.ScanLibraryParams)
```

Re-sync the remote library.

#### RemoteLibraryEntrypoint.syncListening

```python
async def syncListening()
```

Push / pull listening progress (list of ``ListeningProgress``).

#### RemoteLibraryEntrypoint.pollEvents

```python
async def pollEvents()
```

``ExternalUser`` records observed since the last poll.

## DatabaseAdapterEntrypoint

```python
class DatabaseAdapterEntrypoint(NamedEntrypoint)
```

``databaseAdapter`` entrypoint: opens typed SQL sessions for the host library.

The adapter isolate drives sessions through two dispatch verbs on
:meth:`bookclerkInvoke`: ``openSession`` returns a fresh session id, and
``session(id, method, *params)`` calls ``method`` on the retained
:class:`AdapterDatabaseSession`; ``close`` releases the entry.

#### DatabaseAdapterEntrypoint.openSession

```python
async def openSession()
```

Open an :class:`AdapterDatabaseSession`.

#### DatabaseAdapterEntrypoint.dropUnit

```python
async def dropUnit(_unit_ref: str)
```

Physically drop one provisioned binding unit.

**Arguments**:

- `_unit_ref` - Adapter-native unit identity.
  

**Returns**:

  ``None`` when the unit is dropped.
  

**Raises**:

- `PluginError` - ``unsupported`` unless a subclass implements the drop.

#### DatabaseAdapterEntrypoint.bookclerkInvoke

```python
async def bookclerkInvoke(context=None, method="", *args)
```

Adapter dispatch: ``openSession`` → session id; ``dropUnit``; ``session`` → call on a retained session.

**Arguments**:

- `context` - Granted bindings for this invocation.
- `method` - ``openSession``, ``dropUnit``, or ``session``.
- `*args` - For ``session``: ``(id, session_method, *params)``.
  

**Returns**:

  The session id (``str``) for ``openSession``, else the session
  method result converted with :func:`js`.
  

**Raises**:

- `PluginError` - ``invalid_params`` for an unknown session id,
  ``unsupported`` for a method outside the session surface.

## CliEntrypoint

```python
class CliEntrypoint(NamedEntrypoint)
```

``cli`` entrypoint (``PluginCli``): ``describe()`` → ``CliSchema``, ``invoke(params)`` → ``CliInvokeResult``.

#### CliEntrypoint.describe

```python
async def describe()
```

Guest CLI schema (``{"commands": []}`` when the guest has no CLI).

#### CliEntrypoint.invoke

```python
async def invoke(_params: abi.CliInvokeParams)
```

Invoke a guest CLI command (use :func:`cli_args` / :func:`json_payload`).

## OidcEntrypoint

```python
class OidcEntrypoint(NamedEntrypoint)
```

``oidc`` entrypoint: relying-party client templates and credential verification.

#### OidcEntrypoint.clients

```python
async def clients()
```

Plugin-provided OIDC client templates (list of ``OidcClientTemplate``).

#### OidcEntrypoint.authenticateUser

```python
async def authenticateUser(_params: abi.AuthenticateUserParams)
```

Verify remote credentials on behalf of the host (``ExternalUser``).

## GuestDatabase

```python
class GuestDatabase()
```

Host-granted SQL transport behind a named ``[[databases]]`` binding.

#### GuestDatabase.execute

```python
async def execute(_request: ExecuteRequest) -> ExecuteReply
```

Host-mediated typed batch (``ExecuteRequest`` → ``ExecuteReply``).

#### GuestDatabase.close

```python
async def close() -> None
```

Close the grant.

## AdapterDatabaseSession

```python
class AdapterDatabaseSession()
```

Host ↔ database adapter session (``capabilities`` + typed ``execute``).

The restore / identity methods are optional; the base class raises
``unsupported`` for each.

#### AdapterDatabaseSession.capabilities

```python
async def capabilities() -> Any
```

Typed SQL-contract advertisement.

#### AdapterDatabaseSession.execute

```python
async def execute(_request: ExecuteRequest) -> ExecuteReply
```

Typed atomic batch (``ExecuteRequest`` → ``ExecuteReply``).

#### AdapterDatabaseSession.close

```python
async def close() -> None
```

Close the session.

#### AdapterDatabaseSession.bootstrap

```python
async def bootstrap() -> Any
```

Bootstrap-only diagnostic metadata (``DbBootstrap``).

#### AdapterDatabaseSession.exportIdentity

```python
async def exportIdentity() -> list[Any]
```

Identity high-water marks for a logical export (``IdentityHighWater`` rows).

#### AdapterDatabaseSession.importIdentity

```python
async def importIdentity(_rows: list[Any]) -> None
```

Restore identity high-water marks after a logical import.

#### AdapterDatabaseSession.listUserRelations

```python
async def listUserRelations() -> list[str]
```

Names of user relations the host may drop before a restore.

#### AdapterDatabaseSession.prepareUnitRestore

```python
async def prepareUnitRestore() -> None
```

Prepare the engine for a unit restore.

#### AdapterDatabaseSession.dropUserRelations

```python
async def dropUserRelations(_names: list[str]) -> None
```

Drop the named user relations.

#### AdapterDatabaseSession.assertRestoreConstraints

```python
async def assertRestoreConstraints() -> None
```

Re-check constraints after a unit restore.

#### granted\_databases

```python
def granted_databases(granted: _GrantedFetcher,
                      database_tokens: dict[str, str],
                      *,
                      signal: Any | None = None) -> dict[str, DatabaseBinding]
```

Build named :class:`DatabaseBinding` objects over ``POST /db/execute``.

Each entry of ``database_tokens`` (binding name → per-invocation grant
token) becomes an isolated D1-shaped binding whose terminal methods
(``first``, ``run``, ``all``, ``batch``) are async and route through the
granted transport — the same objects the adapter installs on ``env``.

**Arguments**:

- `granted` - Granted-channel fetcher (``GRANTED`` service binding).
- `database_tokens` - Binding name → grant token.
- `signal` - Optional ``AbortSignal`` forwarded to every request.
  

**Returns**:

  Binding name → :class:`DatabaseBinding`.

#### transport\_database

```python
def transport_database(transport: Any) -> DatabaseBinding
```

Wrap an adapter ``GrantedDatabaseTransport`` stub as a :class:`DatabaseBinding`.

**Arguments**:

- `transport` - Stub exposing ``executeBytes(bytes) -> bytes``.
  

**Returns**:

  D1-shaped binding whose terminal methods route through the transport.

