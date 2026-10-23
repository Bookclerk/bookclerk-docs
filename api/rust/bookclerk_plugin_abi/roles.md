**bookclerk_plugin_abi > roles**

# Module: roles

## Contents

**Structs**

- [`BindingValues`](#bindingvalues) - Plain-data portion of [`Bindings`]: every granted value that is not a
- [`Bindings`](#bindings) - Host-granted bindings for one `PluginWorker::open` (`env` in the
- [`ByteRange`](#byterange) - Inclusive byte range for a streamed read.
- [`Entrypoints`](#entrypoints) - Exported entrypoints returned by [`PluginWorker::open`]: one capability
- [`JobController`](#jobcontroller) - Everything one `JobRunner::job` invocation may touch.
- [`NamedKvBinding`](#namedkvbinding) - One granted `[[kv_namespaces]]` binding on `Bindings::kv`.
- [`NamedQueueBinding`](#namedqueuebinding) - One granted `[[queues.producers]]` binding on `Bindings::queues`.
- [`NeverCancel`](#nevercancel) - Cancellation that never fires (tests / hosts without a fence).
- [`ReadResult`](#readresult) - Streamed read result. `body` ownership is transferred to the caller.

**Traits**

- [`AdapterDatabaseSession`](#adapterdatabasesession) - Host ↔ database adapter session (`capabilities` + typed `execute`).
- [`Cancellation`](#cancellation) - Transport cancellation. SDKs project this into a locally created
- [`ContentSource`](#contentsource) - Storefront content source (not byte [`Source`](roles.md#bookclerk_plugin_abirolessource)).
- [`Database`](#database) - Database factory. Sessions cannot survive suspension.
- [`Destination`](#destination) - Destination capability (storage).
- [`EventConsumer`](#eventconsumer) - `[[events.consumers]]` handler (`Entrypoints::event_consumer`).
- [`EventPublisher`](#eventpublisher) - `EVENTS` binding: host-served outbox publisher (`Bindings::events`).
- [`GuestDatabase`](#guestdatabase) - Host-granted SQL transport for job plugin authors (no `capabilities`).
- [`JobRunner`](#jobrunner) - `[triggers] jobs` handler (`Entrypoints::job_runner`).
- [`KvNamespace`](#kvnamespace) - Durable key/value namespace: Workers `KVNamespace`-shaped binding the host
- [`Oidc`](#oidc) - `oidc` entrypoint: relying-party client templates and credential
- [`PluginCli`](#plugincli) - `cli` entrypoint: guest commands under `bookclerk plugins <id> <command>`.
- [`PluginWorker`](#pluginworker) - Root `PluginWorker` capability (`describe` / `open` / `shutdown`).
- [`ProgressSink`](#progresssink) - Progress reports for a job invocation (never carries media).
- [`QueueConsumer`](#queueconsumer) - `[[queues.consumers]]` handler (`Entrypoints::queue_consumer`).
- [`QueueProducer`](#queueproducer) - Queue producer binding: host-served send into the queue a
- [`RemoteLibrary`](#remotelibrary) - `remoteLibrary` entrypoint: long-running remote-library lifecycle.
- [`Source`](#source) - Source capability that can open a named object as a stream.

---

## bookclerk_plugin_abi::roles::AdapterDatabaseSession

*Trait*

Host ↔ database adapter session (`capabilities` + typed `execute`).

**Methods:**

- `capabilities`: Typed SQL-contract advertisement.
- `bootstrap`: Bootstrap-only SeaORM proxy metadata for the open session.
- `execute`: Typed atomic batch (`execute`). Canonical SQL plus required 1:1 proofs.
- `close`: Close the session.
- `export_identity`: Identity high-water from adapter catalogs.
- `import_identity`: Restore identity high-water into adapter catalogs.
- `list_user_relations`: User-visible relation names (excludes engine catalogs).
- `prepare_unit_restore`: Prepare an open restore transaction (deferred FK checks).
- `drop_user_relations`: Drop named user relations (adapter-owned CASCADE / identity companions).
- `assert_restore_constraints`: Fail closed when the restore transaction still has FK violations.



## bookclerk_plugin_abi::roles::BindingValues

*Struct*

Plain-data portion of [`Bindings`]: every granted value that is not a
capability. Hosts build this off the vat thread and attach capabilities
when they call `open`.

**Fields:**
- `config: crate::rpc_types::ExtensibleConfig` - `CONFIG`: granted plugin settings as `application/json`.
- `secrets: crate::rpc_types::ExtensibleConfig` - `SECRETS`: granted secret values as `application/json`; empty payload
- `adapter: crate::generated::DatabaseAdapterConfig` - Database-adapter bootstrap for the `databaseAdapter` entrypoint.

**Methods:**

- `fn config(config: ExtensibleConfig) -> Self` - Bindings carrying only `CONFIG`.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &BindingValues) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> BindingValues`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> BindingValues`



## bookclerk_plugin_abi::roles::Bindings

*Struct*

Host-granted bindings for one `PluginWorker::open` (`env` in the
Workers idiom). Capability fields are `None` when the manifest does not
declare (or the operator did not grant) the binding.

**Fields:**
- `config: crate::rpc_types::ExtensibleConfig` - `CONFIG`: granted plugin settings as `application/json`.
- `secrets: crate::rpc_types::ExtensibleConfig` - `SECRETS`: granted secret values; empty payload when none.
- `adapter: crate::generated::DatabaseAdapterConfig` - Database-adapter bootstrap (see `BindingValues::adapter`).
- `events: Option<Box<dyn EventPublisher>>` - `EVENTS`: outbox publisher; `None` unless `[[events.producers]]` is
- `databases: Vec<(String, Box<dyn GuestDatabase>)>` - Named plugin-owned `[[databases]]` bindings: isolated databases
- `cancel: Box<dyn Cancellation>` - Host cancellation for the whole invocation (fence / lease loss).
- `storage: Option<Box<dyn Destination>>` - `WORK_FS`: host-granted object storage for durable plugin files;
- `queues: Vec<NamedQueueBinding>` - Named `[[queues.producers]]` bindings; empty when none is granted.
- `kv: Vec<NamedKvBinding>` - Named `[[kv_namespaces]]` bindings; empty when none is granted.

**Methods:**

- `fn from_values(values: BindingValues) -> Self` - Bindings with only plain values (no capabilities); tests and
- `fn take_named_queue(self: & mut Self, name: &str) -> Option<NamedQueueBinding>` - Takes the named queue producer binding `name`, if granted.
- `fn take_named_kv(self: & mut Self, name: &str) -> Option<NamedKvBinding>` - Takes the named KV namespace binding `name`, if granted.
- `fn values(self: &Self) -> BindingValues` - Plain-data view (`config` / `secrets` / `adapter`).
- `fn take_named_database(self: & mut Self, name: &str) -> Option<Box<dyn GuestDatabase>>` - Takes the named plugin database binding `name`, if granted.



## bookclerk_plugin_abi::roles::ByteRange

*Struct*

Inclusive byte range for a streamed read.

**Fields:**
- `offset: u64` - Starting offset.
- `length: Option<u64>` - Number of bytes; `None` means to end of object.

**Traits:** Copy, Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ByteRange) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ByteRange`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::roles::Cancellation

*Trait*

Transport cancellation. SDKs project this into a locally created
`AbortSignal` (AbortSignal is not a serializable Workers RPC value).

**Methods:**

- `poll`: Returns true when the host has cancelled this invocation.



## bookclerk_plugin_abi::roles::ContentSource

*Trait*

Storefront content source (not byte [`Source`](roles.md#bookclerk_plugin_abirolessource)).

Every method takes and returns the typed Cap'n Proto payload structs from
[`crate::generated`](generated.md); absent methods return
`PluginError::unsupported`.

**Methods:**

- `login`: Password or one-shot OAuth login. The host seals
- `scan`: Library scan; the host upserts `ScanSummary::books`.
- `fetch_title`: Fetch one title into `params.cache_dir` and return plain media paths.
- `list_accounts`: Accounts the guest knows about.
- `login_start`: Begin an interactive OAuth login.
- `login_complete`: Finish an interactive OAuth login started by `Self::login_start`.
- `search_catalog`: Free-text storefront catalog search.
- `expand_candidates`: Related-title expansion from a seed title.
- `purchase_hint`: Purchase link / price hint; `Ok(None)` when the title is unknown.
- `list_deals`: Current storefront deals.
- `catalog_detail`: Full catalog record for one product; `Ok(None)` when unknown.
- `health`: Storefront health.
- `diagnose`: Operator-facing diagnostic lines.



## bookclerk_plugin_abi::roles::Database

*Trait*

Database factory. Sessions cannot survive suspension.

**Methods:**

- `open_session`: Opens an invocation-scoped adapter session.
- `drop_unit`: Physically deletes one provisioned binding unit.
- `host_session`: Host-private interactive-transaction view of the adapter connection.



## bookclerk_plugin_abi::roles::Destination

*Trait*

Destination capability (storage).

Cap'n Proto stubs are `!Send`; call these traits from a `LocalSet`.

**Methods:**

- `head`: Metadata without a body; `Ok(None)` when the key is missing.
- `list`: One page of keys under `options.prefix`.
- `get`: Streamed read. The body is a transferred stream, not a scalar.
- `put`: Streamed write. `body` ownership is transferred to the destination.
- `copy`: Server-side copy when the backend supports it.
- `delete`: Delete a key (no-op if missing).
- `commit`: Finalize a destination-side staged object using `commit_token`.
- `abort_stage`: Abort a destination-side staged object.



## bookclerk_plugin_abi::roles::Entrypoints

*Struct*

Exported entrypoints returned by [`PluginWorker::open`]: one capability
per `plugin.toml` `entrypoints` entry / trigger, `None` when not exported.

The host refuses an entrypoint the manifest or operator grant did not
allow, so a guest cannot widen its surface here.

**Fields:**
- `event_consumer: Option<Box<dyn EventConsumer>>` - `[[events.consumers]]` trigger.
- `job_runner: Option<Box<dyn JobRunner>>` - `[triggers] jobs` trigger.
- `storefront: Option<Box<dyn ContentSource>>` - `storefront` entrypoint.
- `storage: Option<Box<dyn Destination>>` - `storage` entrypoint.
- `database_adapter: Option<Box<dyn Database>>` - `databaseAdapter` entrypoint.
- `remote_library: Option<Box<dyn RemoteLibrary>>` - `remoteLibrary` entrypoint.
- `cli: Option<Box<dyn PluginCli>>` - `cli` entrypoint.
- `oidc: Option<Box<dyn Oidc>>` - `oidc` entrypoint.
- `queue_consumer: Option<Box<dyn QueueConsumer>>` - `[[queues.consumers]]` trigger.

**Trait Implementations:**

- **Default**
  - `fn default() -> Entrypoints`



## bookclerk_plugin_abi::roles::EventConsumer

*Trait*

`[[events.consumers]]` handler (`Entrypoints::event_consumer`).

Delivery is at-least-once; consume idempotently on
`DomainEvent::deduplication_key`.

**Methods:**

- `event`: Handles one ordered batch and returns exactly one [`EventResult`](rpc_types.md#bookclerk_plugin_abirpc_typeseventresult) per



## bookclerk_plugin_abi::roles::EventPublisher

*Trait*

`EVENTS` binding: host-served outbox publisher (`Bindings::events`).

**Methods:**

- `publish`: Appends `event` to the host outbox. The host stamps `source` (this



## bookclerk_plugin_abi::roles::GuestDatabase

*Trait*

Host-granted SQL transport for job plugin authors (no `capabilities`).

**Methods:**

- `execute`: Host-mediated typed batch (`execute`).
- `close`: Close the grant.



## bookclerk_plugin_abi::roles::JobController

*Struct*

Everything one `JobRunner::job` invocation may touch.

Named database bindings are not per job: they arrive on
`Bindings::databases` at `PluginWorker::open`.

**Fields:**
- `invocation: crate::rpc_types::JobInvocation` - Durable command envelope.
- `input: Box<dyn Source>` - Input source capability.
- `output: Box<dyn Destination>` - Output destination capability.
- `progress: Box<dyn ProgressSink>` - Progress sink (durable job row).
- `cancel: Box<dyn Cancellation>` - Cancellation capability (host fence / lease).



## bookclerk_plugin_abi::roles::JobRunner

*Trait*

`[triggers] jobs` handler (`Entrypoints::job_runner`).

**Methods:**

- `job`: Runs `controller.invocation` using the granted capabilities until



## bookclerk_plugin_abi::roles::KvNamespace

*Trait*

Durable key/value namespace: Workers `KVNamespace`-shaped binding the host
serves from the library database, scoped to the plugin and the namespace a
`[[kv_namespaces]]` row names. Values live in the cluster, so an instance
moved to another host reads its own keys.

**Methods:**

- `get`: Reads one key; `found = false` when absent or expired.
- `put`: Writes one key, replacing any previous value and metadata.
- `delete`: Deletes one key; a missing key is not an error.
- `list`: Lists keys in lexical order, one page at a time.



## bookclerk_plugin_abi::roles::NamedKvBinding

*Struct*

One granted `[[kv_namespaces]]` binding on `Bindings::kv`.

**Fields:**
- `name: String` - Binding name on `env` (`[[kv_namespaces]] binding`).
- `namespace: String` - Namespace the binding is scoped to.
- `kv: Box<dyn KvNamespace>` - Host-served namespace.



## bookclerk_plugin_abi::roles::NamedQueueBinding

*Struct*

One granted `[[queues.producers]]` binding on `Bindings::queues`.

**Fields:**
- `name: String` - Binding name on `env` (`[[queues.producers]] binding`).
- `queue: String` - Queue the binding sends to.
- `producer: Box<dyn QueueProducer>` - Host-served producer.



## bookclerk_plugin_abi::roles::NeverCancel

*Struct*

Cancellation that never fires (tests / hosts without a fence).

**Unit Struct**

**Trait Implementations:**

- **Cancellation**
  - `fn poll(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_abi::roles::Oidc

*Trait*

`oidc` entrypoint: relying-party client templates and credential
verification on behalf of the host authorization server.

**Methods:**

- `clients`: Plugin-provided OIDC authorization-server client templates. Empty when
- `authenticate_user`: Verify remote credentials on behalf of the host.



## bookclerk_plugin_abi::roles::PluginCli

*Trait*

`cli` entrypoint: guest commands under `bookclerk plugins <id> <command>`.

**Methods:**

- `describe`: Declared CLI surface. Empty when the guest exposes no commands.
- `invoke`: Runs one plugin CLI command.



## bookclerk_plugin_abi::roles::PluginWorker

*Trait*

Root `PluginWorker` capability (`describe` / `open` / `shutdown`).

`describe()` advertises typed capabilities; the verified install receipt plus the
operator grant is the host allowlist. `open()` returns the exported
[`Entrypoints`](roles.md#bookclerk_plugin_abirolesentrypoints) for one invocation.

**Methods:**

- `describe`: Advertises identity, features, and scalar limits.
- `open`: Opens the exported entrypoints for `invocation` with the granted
- `database_migrations`: Complete ordered plugin-owned migration sequence for one named binding.
- `shutdown`: Releases guest resources.



## bookclerk_plugin_abi::roles::ProgressSink

*Trait*

Progress reports for a job invocation (never carries media).

**Methods:**

- `report`: Reports `percent` in `0..=100` and an operator-facing `message`.



## bookclerk_plugin_abi::roles::QueueConsumer

*Trait*

`[[queues.consumers]]` handler (`Entrypoints::queue_consumer`).

Delivery is at-least-once; consume idempotently on
`QueueMessage::id`.

**Methods:**

- `queue`: Handles one ordered batch from one queue and returns exactly one



## bookclerk_plugin_abi::roles::QueueProducer

*Trait*

Queue producer binding: host-served send into the queue a
`[[queues.producers]]` row names (`Bindings::queues`).

**Methods:**

- `send`: Sends one message; the host stamps `source` and assigns the id.
- `send_batch`: Sends up to `MAX_LIST_PAGE` messages in order. `delay_seconds` applies



## bookclerk_plugin_abi::roles::ReadResult

*Struct*

Streamed read result. `body` ownership is transferred to the caller.

**Fields:**
- `meta: crate::rpc_types::ObjectMetadata` - Object metadata (size, type, checksums).
- `body: std::pin::Pin<Box<dyn AsyncRead>>` - Byte stream; drop/cancel aborts the read.



## bookclerk_plugin_abi::roles::RemoteLibrary

*Trait*

`remoteLibrary` entrypoint: long-running remote-library lifecycle.

Event delivery is [`EventConsumer`](roles.md#bookclerk_plugin_abiroleseventconsumer); credential verification is [`Oidc`](roles.md#bookclerk_plugin_abirolesoidc).

**Methods:**

- `health`: Liveness.
- `start`: Start long-running work after the host has granted bindings.
- `stop`: Stop long-running work.
- `diagnose`: Operator-facing diagnostic lines.
- `scan_library`: Re-sync the remote library.
- `sync_listening`: Push / pull listening progress; the host upserts the rows.
- `poll_events`: Drain external users observed since the last poll.



## bookclerk_plugin_abi::roles::Source

*Trait*

Source capability that can open a named object as a stream.

**Methods:**

- `open`: Opens `key` for streamed reading.



