**bookclerk_plugin_host > rpc_session**

# Module: rpc_session

## Contents

**Structs**

- [`ExecutorIdentity`](#executoridentity) - Expanded executor identity. Pooling is an optimization; correctness must not
- [`PluginSession`](#pluginsession) - Host-side plugin session (one jailed child + one vat thread).
- [`PluginStorage`](#pluginstorage) - [`StorageBackend`](../bookclerk_storage/traits.md#bookclerk_storagetraitsstoragebackend) over a plugin destination capability (streams, fail-closed scalars).
- [`RpcBackupOps`](#rpcbackupops) - Send-safe backup primitives over a vat-held adapter session (or its open txn).
- [`SessionServices`](#sessionservices) - Host services a session hands its guest as `Bindings` on every

**Functions**

- [`plugin_instance_key`](#plugin_instance_key) - Isolation key: different accounts never share a plugin isolate.

**Constants**

- [`HOST_SHARED_ACCOUNT`](#host_shared_account) - Registry-loaded source/integration guests (not a user account).
- [`OPERATOR_ACCOUNT`](#operator_account) - Isolation key: different accounts never share a plugin isolate.

**Type Aliases**

- [`GuestDatabaseFactory`](#guestdatabasefactory) - Send-safe constructor for a per-binding [`bookclerk_plugin_sdk::GuestDatabase`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolesguestdatabase).

---

## bookclerk_plugin_host::rpc_session::ExecutorIdentity

*Struct*

Expanded executor identity. Pooling is an optimization; correctness must not
depend on a PID surviving.

**Fields:**
- `plugin_id: String` - Provenance-qualified [`bookclerk_plugin_catalog::PluginKey`](../bookclerk_plugin_catalog/identity.md#bookclerk_plugin_catalogidentitypluginkey) (canonical text).
- `artifact_digest: String` - Artifact payload-root digest (or install-path stand-in when unsigned/dev).
- `version: String` - Manifest version.
- `role: String` - Role (`destination`, `source`, `database`, `integration`).
- `account_id: String` - Account / principal.
- `configuration_revision: String` - Configuration revision (installed `plugin.toml` SHA-256).
- `grant_revision: String` - Grant revision (persisted operator consent; revocation changes this).
- `authority_revision: String` - Effective runtime authority (host overlays + clamped budgets).
- `runtime_backend: String` - `workerd` / `native-behind-workerd` / `native-direct`
- `compatibility_date: String` - Workerd compatibility date when applicable.

**Methods:**

- `fn from_plugin(plugin: &DiscoveredPlugin, account_id: &str) -> Self` - Builds an identity from a discovered plugin and account on the default
- `fn from_plugin_on(plugin: &DiscoveredPlugin, account_id: &str, transport: SpawnTransport) -> Self` - Builds an identity for `plugin` spawned over `transport`.
- `fn from_plugin_with_runtime(plugin: &DiscoveredPlugin, account_id: &str, runtime: GuestRuntimeKind) -> Self` - Builds an identity whose `runtime_backend` is the resolved launcher tree.
- `fn with_overlay_config(self: Self, config: &Config) -> Self` - Folds overlay-relevant host config into `Self::configuration_revision`.
- `fn with_grant_revision(self: Self, grant: &crate::PluginGrant) -> Self` - Fills persisted `Self::grant_revision` and effective
- `fn with_persisted_and_effective(self: Self, persisted: &crate::PluginGrant, effective: &crate::PluginGrant) -> Self` - Fills revisions from a persisted operator grant and the effective runtime grant.
- `fn session_key(self: &Self) -> String` - Stable session key. Distinct PIDs with the same key are the same logical

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Clone**
  - `fn clone(self: &Self) -> ExecutorIdentity`
- **PartialEq**
  - `fn eq(self: &Self, other: &ExecutorIdentity) -> bool`



## bookclerk_plugin_host::rpc_session::GuestDatabaseFactory

*Type Alias*: `std::sync::Arc<dyn Fn>`

Send-safe constructor for a per-binding [`bookclerk_plugin_sdk::GuestDatabase`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolesguestdatabase).

`GuestDatabase` trait objects are `?Send`, so named plugin database
bindings cross into the vat task as factories and are constructed on the
vat thread just before the per-job `PluginWorker.open`. The factory
receives the job cancel flag and the host lease deadline so binding
execute can abort and cap `deadlineUnixMs`.



## bookclerk_plugin_host::rpc_session::HOST_SHARED_ACCOUNT

*Constant*: `&str`

Registry-loaded source/integration guests (not a user account).



## bookclerk_plugin_host::rpc_session::OPERATOR_ACCOUNT

*Constant*: `&str`

Isolation key: different accounts never share a plugin isolate.



## bookclerk_plugin_host::rpc_session::PluginSession

*Struct*

Host-side plugin session (one jailed child + one vat thread).

**Methods:**

- `fn spawn(plugin: &DiscoveredPlugin, config: &Config, config_table: Value) -> Result<Self>` - Spawns a plugin guest and connects Cap'n Proto on stdio.
- `fn spawn_for_account(plugin: &DiscoveredPlugin, config: &Config, config_table: Value, account_id: &str) -> Result<Self>` - `Self::spawn` keyed by `(plugin_id, account_id)` so different accounts
- `fn spawn_for_account_with_env(plugin: &DiscoveredPlugin, config: &Config, config_table: Value, account_id: &str, extra_env: &[(&str, std::ffi::OsString)]) -> Result<Self>` - `Self::spawn_for_account` with transport-private extra environment.
- `fn spawn_with(plugin: &DiscoveredPlugin, config: &Config, config_table: Value, account_id: &str, extra_env: &[(&str, std::ffi::OsString)], services: SessionServices) -> Result<Self>` - `Self::spawn_for_account_with_env` plus the host services the guest
- `fn instance_key(self: &Self) -> &str` - Isolation instance key (`plugin_id:account_id`).
- `fn account_id(self: &Self) -> &str` - Account scope carried on every `PluginWorker.open` invocation.
- `fn session_key(self: &Self) -> &str` - Expanded executor session key (artifact, role, grant revision, …).
- `fn guest_pid(self: &Self) -> Option<u32>` - Native guest PID (sibling jail) or the single child when there is none.
- `fn gateway_pid(self: &Self) -> Option<u32>` - Gateway / Cap'n Proto child PID for native-behind-workerd, when known.
- `fn guest_running(self: &Self) -> bool` - True when the guest process, and the gateway when one was spawned, are
- `fn session_dir(self: &Self) -> Option<&std::path::Path>` - Host-owned gateway session directory, when this session has a sibling.
- `fn limits(self: &Self) -> ScalarLimits` - Negotiated scalar limits.
- `fn supports_server_copy(self: &Self) -> bool` - True when the guest accepted `storage.copy`.
- `fn id(self: &Self) -> &str` - Provenance-qualified PluginKey (canonical text).
- `fn alias(self: &Self) -> &str` - Manifest display / CLI alias (`plugin.toml` `id`).
- `fn data_dir(self: &Self) -> &std::path::Path` - Guest data directory.
- `fn open(self: &Self, values: BindingValues) -> Result<()>` - Opens the session's primary entrypoints with the granted binding
- `fn describe(self: &Self) -> Result<PluginDescribe>` - Calls `PluginWorker.describe`.
- `fn stream_copy(self: &Self, job_id: &str, from: &str, to: &str) -> Result<bookclerk_plugin_sdk::JobOutcome>` - Runs the stream-copy job handler on the guest.
- `fn stream_copy_with_cancel(self: &Self, lease: JobInvocationLease, from: &str, to: &str, cancel: Arc<AtomicBool>, progress: Option<(bookclerk_library::LibraryStore, bookclerk_library::JobFence)>) -> Result<bookclerk_plugin_sdk::JobOutcome>` - `Self::stream_copy` raced against a host cancel/fence flag.
- `fn stream_copy_with_databases(self: &Self, lease: JobInvocationLease, from: &str, to: &str, cancel: Arc<AtomicBool>, progress: Option<(bookclerk_library::LibraryStore, bookclerk_library::JobFence)>, databases: Vec<(String, GuestDatabaseFactory)>) -> Result<bookclerk_plugin_sdk::JobOutcome>` - `Self::stream_copy_with_cancel` with named plugin database bindings.
- `fn describe_snapshot(self: &Self) -> &PluginDescribe` - Snapshot from the guest `describe()` call at spawn.
- `fn grant(self: &Self) -> &crate::PluginGrant` - Covering consent grant from spawn.
- `fn require_binding(self: &Self, name: &str) -> Result<()>` - Fail closed when a delivery site needs an ungranted binding.
- `fn scratch_dir(self: &Self) -> &std::path::Path` - Guest TMPDIR / scratch directory.
- `fn guest_ipc_dir(self: &Self) -> Option<&std::path::Path>` - Directory the guest may use for pathname sockets, when this session has one.
- `fn package_sid(self: &Self) -> Option<&str>` - AppContainer package SID when the guest is jailed on Windows.
- `fn spawn_config(self: &Self) -> &Value` - Spawn config JSON captured at spawn.
- `fn has_entrypoint(self: &Self, entrypoint: crate::Entrypoint) -> bool` - True when the guest exported `entrypoint` in `describe()`.
- `fn consumes_event(self: &Self, event_type: &str) -> bool` - True when the guest's default entrypoint consumes `event_type`.
- `fn consumes_events(self: &Self) -> bool` - True when the guest's default entrypoint consumes any event type.
- `fn consumes_queues(self: &Self) -> bool` - True when the guest's default entrypoint consumes at least one
- `fn storefront<T, F, Fut>(self: &Self, call: F) -> Result<T>` - One typed `storefront` method on the opened primary entrypoints.
- `fn remote_library<T, F, Fut>(self: &Self, call: F) -> Result<T>` - One typed `remoteLibrary` method on the opened primary entrypoints.
- `fn remote_library_cancelable<T, F, Fut>(self: &Self, cancel: Arc<AtomicBool>, call: F) -> Result<T>` - Typed `remoteLibrary` method aborted when `cancel` is set.
- `fn deliver_events(self: &Self, batch: Vec<DomainEvent>, cancel: Arc<AtomicBool>) -> Result<Vec<EventResult>>` - Delivers one ordered event batch to the guest `eventConsumer` trigger
- `fn deliver_queue(self: &Self, batch: bookclerk_plugin_abi::QueueBatch, cancel: Arc<AtomicBool>) -> Result<Vec<bookclerk_plugin_abi::QueueResult>>` - Delivers one ordered batch from a `[[queues.consumers]]` queue to the
- `fn cli_describe(self: &Self) -> Result<bookclerk_plugin_sdk::CliSchema>` - Guest CLI schema.
- `fn cli_invoke(self: &Self, params: bookclerk_plugin_sdk::CliInvokeParams) -> Result<bookclerk_plugin_sdk::CliInvokeResult>` - Guest CLI invoke.
- `fn oidc_clients(self: &Self) -> Result<Vec<OidcClientTemplate>>` - Plugin-provided OIDC authorization-server client templates
- `fn oidc_authenticate_user(self: &Self, params: bookclerk_plugin_sdk::AuthenticateUserParams) -> Result<bookclerk_plugin_sdk::ExternalUser>` - Verifies remote credentials through the guest `oidc` entrypoint
- `fn database_migrations(self: &Self, binding: &str) -> Result<Vec<bookclerk_plugin_sdk::PluginMigration>>` - Complete ordered plugin-owned migration sequence for one named binding.
- `fn db_open(self: &Self, values: BindingValues) -> Result<()>` - Opens the library adapter session (held on the vat until drop).
- `fn db_drop_unit(self: &Self, values: BindingValues, unit_ref: &str) -> Result<()>` - Physically drops one provisioned binding unit via `Database.dropUnit`.
- `fn db_open_binding(self: &Self, name: &str, values: BindingValues) -> Result<bookclerk_plugin_sdk::DbCapabilities>` - Opens an isolated adapter session for one named plugin database binding.
- `fn db_execute_binding_request(self: &Self, name: &str, request: bookclerk_plugin_abi::AdapterExecuteRequest, cancel: Arc<AtomicBool>) -> Result<bookclerk_plugin_sdk::ExecuteReply>` - Typed execute on a named plugin database binding session.
- `fn db_execute_binding_envelope_request(self: &Self, name: &str, envelope: bookclerk_plugin_abi::AdapterExecuteRequest, cancel: Arc<AtomicBool>) -> Result<bookclerk_plugin_sdk::ExecuteReply>` - Host-private envelope execute on a named plugin database binding.
- `fn db_capabilities(self: &Self) -> Result<bookclerk_plugin_sdk::DbCapabilities>` - Typed `AdapterDatabaseSession.capabilities`.
- `fn db_bootstrap(self: &Self) -> Result<bookclerk_plugin_sdk::DbBootstrap>` - Typed `AdapterDatabaseSession.bootstrap`.
- `fn db_execute_request(self: &Self, request: bookclerk_plugin_abi::AdapterExecuteRequest, cancel: Arc<AtomicBool>) -> Result<bookclerk_plugin_sdk::ExecuteReply>` - Typed `AdapterDatabaseSession.execute`.
- `fn db_execute_envelope_request(self: &Self, envelope: bookclerk_plugin_abi::AdapterExecuteRequest, cancel: Arc<AtomicBool>) -> Result<bookclerk_plugin_sdk::ExecuteReply>` - Typed `HostAdapterDatabaseSession.execute`.
- `fn db_txn_execute_request(self: &Self, request: bookclerk_plugin_abi::AdapterExecuteRequest, cancel: Arc<AtomicBool>) -> Result<bookclerk_plugin_sdk::ExecuteReply>` - Typed `AdapterTransaction.execute` on the vat-held open transaction.
- `fn db_begin(self: &Self, isolation: bookclerk_plugin_abi::IsolationReq) -> Result<()>` - Begin a vat-held transaction.
- `fn db_commit(self: &Self) -> Result<()>` - Commit the vat-held transaction.
- `fn db_rollback(self: &Self) -> Result<()>` - Roll back the vat-held transaction.
- `fn db_begin_binding(self: &Self, name: &str, isolation: bookclerk_plugin_abi::IsolationReq) -> Result<()>` - Begin a transaction on a named plugin database binding.
- `fn db_commit_binding(self: &Self, name: &str) -> Result<()>` - Commit a named plugin database binding transaction.
- `fn db_rollback_binding(self: &Self, name: &str) -> Result<()>` - Roll back a named plugin database binding transaction.
- `fn db_txn_execute_binding_request(self: &Self, name: &str, request: bookclerk_plugin_abi::AdapterExecuteRequest, cancel: Arc<AtomicBool>) -> Result<bookclerk_plugin_sdk::ExecuteReply>` - Typed execute on the vat-held binding transaction.
- `fn db_export_identity(self: &Self, binding: Option<&str>) -> Result<Vec<bookclerk_plugin_abi::DbIdentityHighWater>>` - Identity high-water from the adapter (open txn if any).
- `fn db_import_identity(self: &Self, binding: Option<&str>, rows: Vec<bookclerk_plugin_abi::DbIdentityHighWater>) -> Result<()>` - Restore identity high-water into the adapter.
- `fn db_list_user_relations(self: &Self, binding: Option<&str>) -> Result<Vec<String>>` - User-visible relation names from the adapter.
- `fn db_prepare_unit_restore(self: &Self, binding: Option<&str>) -> Result<()>` - Prepare the open restore transaction.
- `fn db_drop_user_relations(self: &Self, binding: Option<&str>, names: Vec<String>) -> Result<()>` - Drop named user relations.
- `fn db_assert_restore_constraints(self: &Self, binding: Option<&str>) -> Result<()>` - Fail closed when restore FK checks still fail.

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)`



## bookclerk_plugin_host::rpc_session::PluginStorage

*Struct*

[`StorageBackend`](../bookclerk_storage/traits.md#bookclerk_storagetraitsstoragebackend) over a plugin destination capability (streams, fail-closed scalars).

**Methods:**

- `fn new(session: Arc<PluginSession>) -> Self` - Wraps a connected session after `PluginSession::open` granted the

**Trait Implementations:**

- **StorageBackend**
  - `fn name(self: &Self) -> &'static str`
  - `fn instance_id(self: &Self) -> String`
  - `fn scan_placement(self: &Self) -> Option<String>`
  - `fn clone_box(self: &Self) -> Box<dyn StorageBackend>`
  - `fn put(self: &'life0 Self, key: &'life1 str, data: Bytes, meta: ObjectMeta) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn put_file(self: &'life0 Self, key: &'life1 str, path: &'life2 std::path::Path, meta: ObjectMeta) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn get(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn exists(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn probe(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn copy(self: &'life0 Self, from: &'life1 str, to: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn delete(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_page(self: &'life0 Self, prefix: &'life1 str, cursor: Option<&'life2 str>, limit: u32) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn get_stream(self: &'life0 Self, key: &'life1 str, range: Option<ByteRange>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn put_stream(self: &'life0 Self, key: &'life1 str, body: Pin<Box<dyn AsyncRead>>, meta: ObjectMeta) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn head(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn supports_server_copy(self: &Self) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PluginStorage`



## bookclerk_plugin_host::rpc_session::RpcBackupOps

*Struct*

Send-safe backup primitives over a vat-held adapter session (or its open txn).

**Methods:**

- `fn library(session: Arc<PluginSession>) -> Self` - Library adapter session (no named binding).
- `fn binding<impl Into<String>>(session: Arc<PluginSession>, name: impl Trait) -> Self` - Named plugin-database binding (`plugin_id/binding`).
- `fn shared(self: Self) -> bookclerk_plugin_abi::SharedAdapterBackupOps` - Shared handle stored on backup/restore options.

**Trait Implementations:**

- **AdapterBackupOps**
  - `fn export_identity(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn import_identity(self: &'life0 Self, rows: &'life1 [bookclerk_plugin_abi::DbIdentityHighWater]) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_user_relations(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn prepare_unit_restore(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn drop_user_relations(self: &'life0 Self, names: &'life1 [String]) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn assert_restore_constraints(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_host::rpc_session::SessionServices

*Struct*

Host services a session hands its guest as `Bindings` on every
`PluginWorker.open`.

Nothing here is guest-visible on its own: the session still gates each
binding on the manifest declaration plus the covering consent grant.

**Fields:**
- `event_outbox: Option<bookclerk_library::LibraryStore>` - Library store whose outbox backs the `EVENTS` binding. `None` (the
- `spawn_transport: crate::spawn_plan::SpawnTransport` - How the guest is reached. The default fronts every guest with
- `selected_instance_id: Option<String>` - Plugin instance id selected by CLI scan or acquire.
- `local_output_root: Option<std::path::PathBuf>` - Writable root for a deployed local destination.
- `deployed_endpoint: Option<Option<String>>` - Network authority from the deployed instance document: the S3

**Methods:**

- `fn with_event_outbox(store: bookclerk_library::LibraryStore) -> Self` - Services with the library outbox attached.
- `fn from_outbox(store: Option<&bookclerk_library::LibraryStore>) -> Self` - Services with the library outbox attached when `store` is present.
- `fn direct_native_diagnostic() -> Self` - Default services on the direct native diagnostic transport.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SessionServices`
- **Default**
  - `fn default() -> SessionServices`



## bookclerk_plugin_host::rpc_session::plugin_instance_key

*Function*

Isolation key: different accounts never share a plugin isolate.

```rust
fn plugin_instance_key(plugin_id: &str, account_id: &str) -> String
```



