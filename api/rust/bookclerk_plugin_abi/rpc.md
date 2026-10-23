**bookclerk_plugin_abi > rpc**

# Module: rpc

## Contents

**Structs**

- [`AdapterSessionHandle`](#adaptersessionhandle) - Public adapter session plus host-private interactive transaction client.
- [`ContentSourceClient`](#contentsourceclient) - Cap'n Proto client for [`ContentSource`](roles.md#bookclerk_plugin_abirolescontentsource).
- [`DatabaseClient`](#databaseclient) - Cap'n Proto client for [`Database`](roles.md#bookclerk_plugin_abirolesdatabase).
- [`DestinationClient`](#destinationclient) - Host-side [`Destination`](roles.md#bookclerk_plugin_abirolesdestination) over a capnp destination stub.
- [`DestinationServer`](#destinationserver) - Cap'n Proto server wrapping a [`Destination`](roles.md#bookclerk_plugin_abirolesdestination) trait object.
- [`EventConsumerClient`](#eventconsumerclient) - Cap'n Proto client for [`EventConsumer`](roles.md#bookclerk_plugin_abiroleseventconsumer) (host → guest delivery).
- [`EventPublisherClient`](#eventpublisherclient) - Guest-side client for the host `EVENTS` binding ([`EventPublisher`](roles.md#bookclerk_plugin_abiroleseventpublisher)).
- [`HostBindings`](#hostbindings) - Host-side [`Bindings`]: plain values plus host-served capabilities the
- [`JobRunnerClient`](#jobrunnerclient) - Cap'n Proto client for [`JobRunner`](roles.md#bookclerk_plugin_abirolesjobrunner) (host → guest job dispatch).
- [`KvNamespaceClient`](#kvnamespaceclient) - Guest-side client for a host KV namespace binding ([`KvNamespace`](roles.md#bookclerk_plugin_abiroleskvnamespace)).
- [`OidcClient`](#oidcclient) - Cap'n Proto client for [`Oidc`](roles.md#bookclerk_plugin_abirolesoidc).
- [`OpenedEntrypoints`](#openedentrypoints) - Typed clients for the entrypoints a guest exported from `open`. `None`
- [`PluginCliClient`](#plugincliclient) - Cap'n Proto client for [`PluginCli`](roles.md#bookclerk_plugin_abirolesplugincli).
- [`PluginClient`](#pluginclient) - Host bootstrap client for a plugin vat.
- [`PluginServer`](#pluginserver) - Cap'n Proto server wrapping [`PluginWorker`](roles.md#bookclerk_plugin_abirolespluginworker).
- [`QueueConsumerClient`](#queueconsumerclient) - Cap'n Proto client for [`QueueConsumer`](roles.md#bookclerk_plugin_abirolesqueueconsumer) (host → guest delivery).
- [`QueueProducerClient`](#queueproducerclient) - Guest-side client for a host queue producer binding ([`QueueProducer`](roles.md#bookclerk_plugin_abirolesqueueproducer)).
- [`RemoteLibraryClient`](#remotelibraryclient) - Cap'n Proto client for [`RemoteLibrary`](roles.md#bookclerk_plugin_abirolesremotelibrary).
- [`SourceClient`](#sourceclient) - Host-side [`Source`](roles.md#bookclerk_plugin_abirolessource) over a capnp source stub.
- [`SourceServer`](#sourceserver) - Cap'n Proto server wrapping a [`Source`](roles.md#bookclerk_plugin_abirolessource).

**Functions**

- [`byte_source_from_async_read`](#byte_source_from_async_read) - Returns a `ByteSource` client that pulls from `reader`.
- [`connect_plugin`](#connect_plugin) - Connects as the client side of a two-party vat.
- [`pull_byte_source_to_writer`](#pull_byte_source_to_writer) - Pulls `source` into `writer` using bounded windows.
- [`serve_plugin`](#serve_plugin) - Serves `plugin` as the bootstrap object on a two-party vat over `reader`/`writer`.
- [`serve_plugin_stdio`](#serve_plugin_stdio) - Serves `plugin` on stdin/stdout. Must run on a current-thread runtime + `LocalSet`.

---

## bookclerk_plugin_abi::rpc::AdapterSessionHandle

*Struct*

Public adapter session plus host-private interactive transaction client.

**Fields:**
- `session: Box<dyn AdapterDatabaseSession>` - Typed `capabilities` / `execute` / `close`.
- `host: crate::host_rpc::HostAdapterDatabaseSessionClient` - Host-only `begin` (capability cast on the same session object).



## bookclerk_plugin_abi::rpc::ContentSourceClient

*Struct*

Cap'n Proto client for [`ContentSource`](roles.md#bookclerk_plugin_abirolescontentsource).

**Methods:**

- `fn new(client: content_source_capnp::Client) -> Self` - Wraps a `ContentSource` capability client (any transport that yields one, including

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ContentSourceClient`
- **ContentSource**
  - `fn login(self: &'life0 Self, params: LoginParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn scan(self: &'life0 Self, params: ScanParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn fetch_title(self: &'life0 Self, params: FetchTitleParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_accounts(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn login_start(self: &'life0 Self, params: LoginParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn login_complete(self: &'life0 Self, params: LoginCompleteParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn search_catalog(self: &'life0 Self, params: SearchCatalogParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn expand_candidates(self: &'life0 Self, params: ExpandCandidatesParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn purchase_hint(self: &'life0 Self, params: PurchaseHintParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_deals(self: &'life0 Self, params: ListDealsParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn catalog_detail(self: &'life0 Self, params: CatalogDetailParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn health(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn diagnose(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_abi::rpc::DatabaseClient

*Struct*

Cap'n Proto client for [`Database`](roles.md#bookclerk_plugin_abirolesdatabase).

**Methods:**

- `fn new(client: database_capnp::Client) -> Self` - Wraps a `Database` capability client (any transport that yields one, including
- `fn open_session_handle(self: &Self) -> Result<AdapterSessionHandle>` - Opens a session and host-private transaction client on one capability.

**Trait Implementations:**

- **Database**
  - `fn open_session(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn drop_unit(self: &'life0 Self, unit_ref: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
- **Clone**
  - `fn clone(self: &Self) -> DatabaseClient`



## bookclerk_plugin_abi::rpc::DestinationClient

*Struct*

Host-side [`Destination`](roles.md#bookclerk_plugin_abirolesdestination) over a capnp destination stub.

**Methods:**

- `fn new(client: dest_iface::Client, window: u32) -> Self` - Wraps a capnp destination client.
- `fn with_max_list_page(self: Self, max_list_page: u32) -> Self` - Applies negotiated list-page cardinality.

**Trait Implementations:**

- **Destination**
  - `fn head(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list(self: &'life0 Self, options: ListOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn get(self: &'life0 Self, key: &'life1 str, range: Option<ByteRange>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn put(self: &'life0 Self, key: &'life1 str, body: Pin<Box<dyn AsyncRead>>, options: WriteOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn copy(self: &'life0 Self, from: &'life1 str, to: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn delete(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn commit(self: &'life0 Self, key: &'life1 str, commit_token: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn abort_stage(self: &'life0 Self, key: &'life1 str, commit_token: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
- **Clone**
  - `fn clone(self: &Self) -> DestinationClient`



## bookclerk_plugin_abi::rpc::DestinationServer

*Struct*

Cap'n Proto server wrapping a [`Destination`](roles.md#bookclerk_plugin_abirolesdestination) trait object.

**Methods:**

- `fn new(inner: Arc<dyn Destination>, window: u32) -> Self` - Serves `inner` with the given stream window.

**Trait Implementations:**

- **Server**
  - `fn head(self: Rc<Self>, params: dest_iface::HeadParams, results: dest_iface::HeadResults) -> capnp::Result<()>`
  - `fn list(self: Rc<Self>, params: dest_iface::ListParams, results: dest_iface::ListResults) -> capnp::Result<()>`
  - `fn get(self: Rc<Self>, params: dest_iface::GetParams, results: dest_iface::GetResults) -> capnp::Result<()>`
  - `fn put(self: Rc<Self>, params: dest_iface::PutParams, results: dest_iface::PutResults) -> capnp::Result<()>`
  - `fn commit(self: Rc<Self>, params: dest_iface::CommitParams, results: dest_iface::CommitResults) -> capnp::Result<()>`
  - `fn abort_stage(self: Rc<Self>, params: dest_iface::AbortStageParams, results: dest_iface::AbortStageResults) -> capnp::Result<()>`
  - `fn copy(self: Rc<Self>, params: dest_iface::CopyParams, results: dest_iface::CopyResults) -> capnp::Result<()>`
  - `fn delete(self: Rc<Self>, params: dest_iface::DeleteParams, results: dest_iface::DeleteResults) -> capnp::Result<()>`



## bookclerk_plugin_abi::rpc::EventConsumerClient

*Struct*

Cap'n Proto client for [`EventConsumer`](roles.md#bookclerk_plugin_abiroleseventconsumer) (host → guest delivery).

**Methods:**

- `fn new(client: event_consumer_capnp::Client) -> Self` - Wraps a `EventConsumer` capability client (any transport that yields one, including

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventConsumerClient`
- **EventConsumer**
  - `fn event(self: &'life0 Self, batch: Vec<DomainEvent>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_abi::rpc::EventPublisherClient

*Struct*

Guest-side client for the host `EVENTS` binding ([`EventPublisher`](roles.md#bookclerk_plugin_abiroleseventpublisher)).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventPublisherClient`
- **EventPublisher**
  - `fn publish(self: &'life0 Self, event: PluginEvent) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_abi::rpc::HostBindings

*Struct*

Host-side [`Bindings`]: plain values plus host-served capabilities the
guest receives on `open`.

**Fields:**
- `values: crate::roles::BindingValues` - `CONFIG` / `SECRETS` / adapter bootstrap.
- `events: Option<std::sync::Arc<dyn EventPublisher>>` - `EVENTS` publisher; `None` unless `[[events.producers]]` is granted.
- `databases: Vec<(String, std::sync::Arc<dyn GuestDatabase>)>` - Named plugin-owned `[[databases]]` sessions.
- `cancel: std::sync::Arc<dyn Cancellation>` - Invocation-wide cancellation (fence / lease loss).
- `storage: Option<std::sync::Arc<dyn Destination>>` - `WORK_FS` object storage; `None` unless `[work_fs]` is granted.
- `queues: Vec<(String, String, std::sync::Arc<dyn QueueProducer>)>` - Named `[[queues.producers]]` producers as `(binding, queue, producer)`.
- `kv: Vec<(String, String, std::sync::Arc<dyn KvNamespace>)>` - Named `[[kv_namespaces]]` namespaces as `(binding, namespace, kv)`.

**Methods:**

- `fn from_values(values: BindingValues) -> Self` - Bindings with only plain values and no cancellation source.

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`



## bookclerk_plugin_abi::rpc::JobRunnerClient

*Struct*

Cap'n Proto client for [`JobRunner`](roles.md#bookclerk_plugin_abirolesjobrunner) (host → guest job dispatch).

**Methods:**

- `fn new(client: job_runner_capnp::Client, window: u32) -> Self` - Wraps a `JobRunner` capability client with the negotiated stream window.
- `fn job(self: &Self, invocation: &JobInvocation, input: Arc<dyn Source>, output: Arc<dyn Destination>, progress: Arc<dyn ProgressSink>, cancel: Arc<dyn Cancellation>) -> Result<JobOutcome>` - Runs one job with host-served input / output / progress / cancel
- `fn job_with_capabilities(self: &Self, invocation: &JobInvocation, input: source_capnp::Client, output: dest_iface::Client, progress: progress_sink::Client, cancel: cancellation::Client) -> Result<JobOutcome>` - Runs one job with caller-built `JobController` capability clients.

**Trait Implementations:**

- **JobRunner**
  - `fn job(self: &'life0 Self, controller: JobController) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
- **Clone**
  - `fn clone(self: &Self) -> JobRunnerClient`



## bookclerk_plugin_abi::rpc::KvNamespaceClient

*Struct*

Guest-side client for a host KV namespace binding ([`KvNamespace`](roles.md#bookclerk_plugin_abiroleskvnamespace)).

**Methods:**

- `fn new(client: kv_namespace_capnp::Client) -> Self` - Wraps a `KvNamespace` capability client.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> KvNamespaceClient`
- **KvNamespace**
  - `fn get(self: &'life0 Self, key: String) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn put(self: &'life0 Self, key: String, value: Vec<u8>, options: crate::KvPutOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn delete(self: &'life0 Self, key: String) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list(self: &'life0 Self, options: crate::KvListOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_abi::rpc::OidcClient

*Struct*

Cap'n Proto client for [`Oidc`](roles.md#bookclerk_plugin_abirolesoidc).

**Methods:**

- `fn new(client: oidc_capnp::Client) -> Self` - Wraps a `Oidc` capability client (any transport that yields one, including

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> OidcClient`
- **Oidc**
  - `fn clients(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn authenticate_user(self: &'life0 Self, params: AuthenticateUserParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_abi::rpc::OpenedEntrypoints

*Struct*

Typed clients for the entrypoints a guest exported from `open`. `None`
means the guest did not export that entrypoint.

**Fields:**
- `event_consumer: Option<EventConsumerClient>` - `[[events.consumers]]` trigger.
- `job_runner: Option<JobRunnerClient>` - `[triggers] jobs` trigger.
- `storefront: Option<ContentSourceClient>` - `storefront` entrypoint.
- `storage: Option<DestinationClient>` - `storage` entrypoint.
- `database_adapter: Option<DatabaseClient>` - `databaseAdapter` entrypoint.
- `remote_library: Option<RemoteLibraryClient>` - `remoteLibrary` entrypoint.
- `cli: Option<PluginCliClient>` - `cli` entrypoint.
- `oidc: Option<OidcClient>` - `oidc` entrypoint.
- `queue_consumer: Option<QueueConsumerClient>` - `[[queues.consumers]]` trigger.

**Trait Implementations:**

- **Default**
  - `fn default() -> OpenedEntrypoints`



## bookclerk_plugin_abi::rpc::PluginCliClient

*Struct*

Cap'n Proto client for [`PluginCli`](roles.md#bookclerk_plugin_abirolesplugincli).

**Methods:**

- `fn new(client: plugin_cli_capnp::Client) -> Self` - Wraps a `PluginCli` capability client (any transport that yields one, including

**Trait Implementations:**

- **PluginCli**
  - `fn describe(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn invoke(self: &'life0 Self, params: CliInvokeParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
- **Clone**
  - `fn clone(self: &Self) -> PluginCliClient`



## bookclerk_plugin_abi::rpc::PluginClient

*Struct*

Host bootstrap client for a plugin vat.

**Fields:**
- `window: u32` - Negotiated stream window.
- `limits: crate::limits::ScalarLimits` - Negotiated scalar / list limits.

**Methods:**

- `fn new(client: plugin_worker::Client, window: u32) -> Self` - Wraps a bootstrap client.
- `fn with_limits(self: Self, limits: ScalarLimits) -> Self` - Applies negotiated limits (stream window + list page).
- `fn describe(self: &Self) -> Result<PluginDescribe>` - Calls `describe`.
- `fn open(self: &Self, invocation: &Invocation, bindings: HostBindings) -> Result<OpenedEntrypoints>` - Opens the exported entrypoints for `invocation` with host-granted
- `fn database_migrations(self: &Self, binding: &str) -> Result<Vec<PluginMigration>>` - Complete ordered plugin-owned migration sequence for one named binding.
- `fn shutdown(self: &Self) -> Result<()>` - Asks the guest to release its resources (`PluginWorker.shutdown`).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginClient`



## bookclerk_plugin_abi::rpc::PluginServer

*Struct*

Cap'n Proto server wrapping [`PluginWorker`](roles.md#bookclerk_plugin_abirolespluginworker).

**Methods:**

- `fn new(inner: Arc<dyn PluginWorker>, window: u32) -> Self` - Serves `inner`.

**Trait Implementations:**

- **Server**
  - `fn describe(self: Rc<Self>, _params: plugin_worker::DescribeParams, results: plugin_worker::DescribeResults) -> capnp::Result<()>`
  - `fn open(self: Rc<Self>, params: plugin_worker::OpenParams, results: plugin_worker::OpenResults) -> capnp::Result<()>`
  - `fn shutdown(self: Rc<Self>, _params: plugin_worker::ShutdownParams, results: plugin_worker::ShutdownResults) -> capnp::Result<()>`
  - `fn database_migrations(self: Rc<Self>, params: plugin_worker::DatabaseMigrationsParams, results: plugin_worker::DatabaseMigrationsResults) -> capnp::Result<()>`



## bookclerk_plugin_abi::rpc::QueueConsumerClient

*Struct*

Cap'n Proto client for [`QueueConsumer`](roles.md#bookclerk_plugin_abirolesqueueconsumer) (host → guest delivery).

**Methods:**

- `fn new(client: queue_consumer_capnp::Client) -> Self` - Wraps a `QueueConsumer` capability client (any transport that yields

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> QueueConsumerClient`
- **QueueConsumer**
  - `fn queue(self: &'life0 Self, batch: crate::QueueBatch) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_abi::rpc::QueueProducerClient

*Struct*

Guest-side client for a host queue producer binding ([`QueueProducer`](roles.md#bookclerk_plugin_abirolesqueueproducer)).

**Methods:**

- `fn new(client: queue_producer_capnp::Client) -> Self` - Wraps a `QueueProducer` capability client.

**Trait Implementations:**

- **QueueProducer**
  - `fn send(self: &'life0 Self, message: crate::QueueSendMessage) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn send_batch(self: &'life0 Self, messages: Vec<crate::QueueSendMessage>, delay_seconds: u32) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
- **Clone**
  - `fn clone(self: &Self) -> QueueProducerClient`



## bookclerk_plugin_abi::rpc::RemoteLibraryClient

*Struct*

Cap'n Proto client for [`RemoteLibrary`](roles.md#bookclerk_plugin_abirolesremotelibrary).

**Methods:**

- `fn new(client: remote_library_capnp::Client) -> Self` - Wraps a `RemoteLibrary` capability client (any transport that yields one, including

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RemoteLibraryClient`
- **RemoteLibrary**
  - `fn health(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn start(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn stop(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn diagnose(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn scan_library(self: &'life0 Self, params: ScanLibraryParams) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn sync_listening(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn poll_events(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_abi::rpc::SourceClient

*Struct*

Host-side [`Source`](roles.md#bookclerk_plugin_abirolessource) over a capnp source stub.

**Methods:**

- `fn new(client: source_capnp::Client, window: u32) -> Self` - Wraps a capnp source client.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SourceClient`
- **Source**
  - `fn open(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_abi::rpc::SourceServer

*Struct*

Cap'n Proto server wrapping a [`Source`](roles.md#bookclerk_plugin_abirolessource).

**Methods:**

- `fn new(inner: Arc<dyn Source>, window: u32) -> Self` - Serves `inner`.

**Trait Implementations:**

- **Server**
  - `fn open(self: Rc<Self>, params: source_capnp::OpenParams, results: source_capnp::OpenResults) -> capnp::Result<()>`



## bookclerk_plugin_abi::rpc::byte_source_from_async_read

*Function*

Returns a `ByteSource` client that pulls from `reader`.

```rust
fn byte_source_from_async_read(reader: std::pin::Pin<Box<dyn AsyncRead>>, window: u32) -> byte_source::Client
```



## bookclerk_plugin_abi::rpc::connect_plugin

*Function*

Connects as the client side of a two-party vat.

Must run inside a `tokio::task::LocalSet`. The returned [`RpcSystem`](https://docs.rs/capnp_rpc/latest/capnp_rpc/struct.RpcSystem.html) must be
spawned with `tokio::task::spawn_local`.

```rust
fn connect_plugin<R, W>(reader: R, writer: W, window: u32) -> (PluginClient, capnp_rpc::RpcSystem<rpc_twoparty_capnp::Side>)
```



## bookclerk_plugin_abi::rpc::pull_byte_source_to_writer

*Function*

Pulls `source` into `writer` using bounded windows.

A failed `ByteSource.pull` is returned as [`PluginError`](error.md#bookclerk_plugin_abierrorpluginerror) — never as a clean
EOF. Destinations must abort rather than commit.

# Errors

Returns a plugin error when the stream or writer fails.

```rust
fn pull_byte_source_to_writer<W>(source: byte_source::Client, writer: & mut W, window: u32) -> crate::Result<u64>
```



## bookclerk_plugin_abi::rpc::serve_plugin

*Function*

Serves `plugin` as the bootstrap object on a two-party vat over `reader`/`writer`.

Must run inside a `tokio::task::LocalSet`.

# Errors

Returns a plugin error when the vat fails.

```rust
fn serve_plugin<R, W>(plugin: std::sync::Arc<dyn PluginWorker>, reader: R, writer: W, window: u32) -> crate::Result<()>
```



## bookclerk_plugin_abi::rpc::serve_plugin_stdio

*Function*

Serves `plugin` on stdin/stdout. Must run on a current-thread runtime + `LocalSet`.

# Errors

Returns a plugin error when the vat fails.

```rust
fn serve_plugin_stdio(plugin: std::sync::Arc<dyn PluginWorker>, window: u32) -> crate::Result<()>
```



