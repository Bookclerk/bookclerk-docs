# bookclerk_integrations

Outbound integrations and SPA portal APIs.

Host binaries should depend on [`Integration`](#integration) / [`IntegrationRegistry`](#integrationregistry) only.
First-party adapters (e.g. Audiobookshelf) live in
`bookclerk-plugin-integration-*` crates and run as staged external guests.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Integration`](#integration) | trait | Downstream consumer of acquired books / identity signals (e.g. Audiobookshelf). |
| [`Brand`](#brand) | struct | Visual identity for a store or integration in the portal UI. |
| [`PortalState`](#portalstate) | struct | Shared state for portal handlers. |
| [`IntegrationRegistry`](#integrationregistry) | struct | Fan-out registry for configured integrations. |
| [`MintedClaimTicket`](#mintedclaimticket) | struct | Freshly minted claim ticket (includes plaintext once). |
| [`EventSubscription`](#eventsubscription) | struct | Declared durable `onEvent` subscription (mirrors `plugin.toml`). |
| [`IntegrationContext`](#integrationcontext) | struct | Context passed when starting background watchers. |
| [`ProvidedOidcClient`](#providedoidcclient) | struct | Host-side copy of a guest `oidcClients` template (no ABI crate dependency). |
| [`QueueSubscription`](#queuesubscription) | struct | Declared `[[queues.consumers]]` row (mirrors `plugin.toml`): the queue the integration's `queue(batch)` trigger consumes and its redelivery policy. |
| [`ExternalUser`](#externaluser) | struct | External user resolved via integration credential login or watcher. |
| [`IntegrationHealth`](#integrationhealth-1) | struct | Health snapshot for one integration. |
| [`ListeningProgressSnapshot`](#listeningprogresssnapshot) | struct | One listening / progress row from any integration (ABS, plugins, …). |
| [`SyncListeningProviderResult`](#synclisteningproviderresult) | struct | Per-integration outcome from `crate::IntegrationRegistry::sync_listening_progress_all`. |
| [`SyncListeningSummary`](#synclisteningsummary) | struct | Result of syncing listening progress across one or more integrations. |
| [`IntegrationError`](#integrationerror) | enum | Errors from outbound integrations and the connect portal. |
| [`from_config`](#from_config) | function | Construct an empty registry (no in-process adapters). |
| [`register_builtins`](#register_builtins) | function | Register first-party integrations into an existing registry. |
| [`emit_book_acquired`](#emit_book_acquired) | function | Look up the acquired book and publish `book_acquired` to the durable outbox. |
| [`match_book_uuid`](#match_book_uuid) | function | Best-effort match to an owned library book via ASIN, ISBN, or exact title. |
| [`upsert_listening_snapshots`](#upsert_listening_snapshots) | function | Upsert provider-agnostic listening snapshots into `listening_progress`. |
| [`mint_for_external_user`](#mint_for_external_user) | function | Mint a claim ticket for an external user (daemon watcher / CLI). |
| [`portal_identity_from_headers`](#portal_identity_from_headers) | function | Resolve a portal identity from the request cookie, if present and valid. |
| [`portal_spa_router`](#portal_spa_router) | function | SPA-facing portal API. Nest under `/api/portal`. |
| [`redeem_lose_next_responses`](#redeem_lose_next_responses) | function | Drop the next N successful redeem HTTP responses after the DB commit. |
| [`generate_token`](#generate_token) | function | Generate a URL-safe random token. |
| [`identity_from_session`](#identity_from_session) | function | Resolve identity from a portal session cookie value. |
| [`mint_claim_ticket`](#mint_claim_ticket) | function | Ensure identity exists and mint a claim ticket. |
| [`redeem_ticket_to_session`](#redeem_ticket_to_session) | function | Redeem a claim ticket into a portal session cookie value (plaintext). |
| [`redeem_ticket_to_session_with_client`](#redeem_ticket_to_session_with_client) | function | Redeem a claim ticket and mint a portal session with optional client metadata. |
| [`session_for_identity`](#session_for_identity) | function | Create a portal session for an already-resolved identity (credential login). |
| [`session_for_identity_with_client`](#session_for_identity_with_client) | function | Create a portal session for an identity with optional client metadata. |
| [`ticket_portal_url`](#ticket_portal_url) | function | Build a shareable SPA claim URL when `public_origin` is configured. |
| [`Result`](#result) | type alias | Result alias for this crate. |
| [`MAX_REGISTERED_INTEGRATIONS`](#max_registered_integrations) | constant | Hard ceiling on registered integrations (plugin discovery is user-influenced). |

## Re-exports

- [`bookclerk_plugin_abi::DomainEvent`](../bookclerk_plugin_abi/index.md#domainevent)
- [`bookclerk_plugin_abi::EventResult`](../bookclerk_plugin_abi/index.md#eventresult)
- [`bookclerk_plugin_abi::QueueBatch`](../bookclerk_plugin_abi/generated.md#queuebatch)
- [`bookclerk_plugin_abi::QueueMessage`](../bookclerk_plugin_abi/generated.md#queuemessage)
- [`bookclerk_plugin_abi::QueueResult`](../bookclerk_plugin_abi/generated.md#queueresult)
- [`bookclerk_plugin_abi::QueueRetry`](../bookclerk_plugin_abi/generated.md#queueretry)
- [`tickets::hash_token`](../bookclerk_library/index.md#hash_token)

## Traits

### Integration

```rust
pub trait Integration: Send + Sync {
    fn id(&self) -> &str;
    fn plugin_key(&self) -> &str;
    fn plugin_instance_id(&self) -> Option<&str>;
    fn display_name(&self) -> &str;
    fn start<'life0, 'async_trait>(
        &'life0 self,
        ctx: IntegrationContext,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn stop<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn deliver_domain_event<'life0, 'async_trait>(
        &'life0 self,
        event: DomainEvent,
    ) -> Pin<Box<dyn Future<Output = Result<EventResult>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn deliver_domain_event_cancelable<'life0, 'async_trait>(
        &'life0 self,
        event: DomainEvent,
        cancel: Arc<AtomicBool>,
    ) -> Pin<Box<dyn Future<Output = Result<EventResult>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn event_subscriptions(&self) -> Vec<EventSubscription>;
    fn queue_subscriptions(&self) -> Vec<QueueSubscription>;
    fn deliver_queue_batch_cancelable<'life0, 'async_trait>(
        &'life0 self,
        batch: QueueBatch,
        _cancel: Arc<AtomicBool>,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<QueueResult>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn health<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<IntegrationHealth>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn supports_credential_login(&self) -> bool;
    fn authenticate_user<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        _username: &'life1 str,
        _password: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<ExternalUser>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn supports_library_scan(&self) -> bool;
    fn scan_library<'life0, 'async_trait>(
        &'life0 self,
        _force: bool,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn supports_listening_sync(&self) -> bool;
    fn sync_listening_progress<'life0, 'life1, 'async_trait>(
        &'life0 self,
        _library: &'life1 LibraryStore,
    ) -> Pin<Box<dyn Future<Output = Result<usize>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn diagnose<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn portal_brand(&self) -> Option<Brand>;
    fn provided_oidc_clients<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<ProvidedOidcClient>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Downstream consumer of acquired books / identity signals (e.g. Audiobookshelf).

#### Integration::id

```rust
fn id(&self) -> &str
```

Display / CLI alias (`audiobookshelf`, …). Globally unique within one
installation; durable identity is [`Self::plugin_key`](#integrationplugin_key).

#### Integration::plugin_key

```rust
fn plugin_key(&self) -> &str
```

Provenance-qualified PluginKey when this adapter is an external guest.

Defaults to [`Self::id`](#integrationid) for in-crate test doubles.

#### Integration::plugin_instance_id

```rust
fn plugin_instance_id(&self) -> Option<&str>
```

Plugin instance that owns this deployed integration.

`None` for transitional startup and in-crate adapters. Two deployed
instances of one key each return their own id. Reload keeps a session
only when this id is a present deployment.

#### Integration::display_name

```rust
fn display_name(&self) -> &str
```

Human-facing name for portal / CLI (defaults to [`Self::id`](#integrationid)).

#### Integration::start

```rust
fn start<'life0, 'async_trait>(
    &'life0 self,
    ctx: IntegrationContext,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Start background tasks (user watchers, etc.).

#### Integration::stop

```rust
fn stop<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Stop background tasks started by [`Self::start`](#integrationstart).

Default: no-op. Implementations that spawn poll loops should cancel them
idempotently so config reload can replace the registry safely.

#### Integration::deliver_domain_event

```rust
fn deliver_domain_event<'life0, 'async_trait>(
    &'life0 self,
    event: DomainEvent,
) -> Pin<Box<dyn Future<Output = Result<EventResult>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Deliver one versioned outbox [`DomainEvent`](../bookclerk_plugin_abi/index.md#domainevent) and return a durable
[`EventResult`](../bookclerk_plugin_abi/index.md#eventresult).

Delivery is at-least-once: consume idempotently on
`DomainEvent::deduplication_key`. Return `Reject` for event types or
schema versions this integration does not understand, `Retry` for
transient downstream failures, and `Ack` once the effect is durable.
Returning `Err` is treated like a retryable transport failure.

#### Integration::deliver_domain_event_cancelable

```rust
fn deliver_domain_event_cancelable<'life0, 'async_trait>(
    &'life0 self,
    event: DomainEvent,
    cancel: Arc<AtomicBool>,
) -> Pin<Box<dyn Future<Output = Result<EventResult>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Like [`Self::deliver_domain_event`](#integrationdeliver_domain_event), aborting when `cancel` is set (fence loss).

##### Errors

Returns [`IntegrationError::Message`](#integrationerror) `"fence lost"` when `cancel` is set,
or whatever [`Self::deliver_domain_event`](#integrationdeliver_domain_event) returns.

#### Integration::event_subscriptions

```rust
fn event_subscriptions(&self) -> Vec<EventSubscription>
```

Durable outbox subscriptions. Empty means this integration is not a subscriber.

#### Integration::queue_subscriptions

```rust
fn queue_subscriptions(&self) -> Vec<QueueSubscription>
```

Declared `[[queues.consumers]]` rows. Empty means this integration
consumes no plugin-owned queue.

#### Integration::deliver_queue_batch_cancelable

```rust
fn deliver_queue_batch_cancelable<'life0, 'async_trait>(
    &'life0 self,
    batch: QueueBatch,
    _cancel: Arc<AtomicBool>,
) -> Pin<Box<dyn Future<Output = Result<Vec<QueueResult>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Deliver one ordered [`QueueBatch`](../bookclerk_plugin_abi/generated.md#queuebatch) to the `queue(batch)` trigger and
return exactly one [`QueueResult`](../bookclerk_plugin_abi/generated.md#queueresult) per message, aborting when `cancel`
is set (fence loss).

Delivery is at-least-once: consume idempotently on
`bookclerk_plugin_abi::QueueMessage::id`. Returning `Err` is treated
like a retryable transport failure.

##### Errors

The default returns [`IntegrationError::Message`](#integrationerror) because the
integration declares no queue consumers.

#### Integration::health

```rust
fn health<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<IntegrationHealth>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Probes connectivity and configuration for this integration.

##### Returns

A health snapshot suitable for CLI / SPA status views.

##### Errors

Returns an error when the probe itself fails unexpectedly (transports);
soft failures should usually set [`IntegrationHealth::ok`](#integrationhealth-1) to `false`.

#### Integration::supports_credential_login

```rust
fn supports_credential_login(&self) -> bool
```

Whether this integration may appear as a portal username/password login.

Default: `false`. Integrations that implement credential login override
this (typically gated by a config flag such as `allow_credential_login`).

#### Integration::authenticate_user

```rust
fn authenticate_user<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    _username: &'life1 str,
    _password: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<ExternalUser>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Validate end-user credentials (portal return visits / self-service).

Default: returns an error. Only called when
[`Self::supports_credential_login`](#integrationsupports_credential_login) is `true`.

#### Integration::supports_library_scan

```rust
fn supports_library_scan(&self) -> bool
```

Whether this integration can trigger a remote library scan.

#### Integration::scan_library

```rust
fn scan_library<'life0, 'async_trait>(
    &'life0 self,
    _force: bool,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Trigger a remote library scan (optional force rescan).

Default: unsupported. Hosts (CLI / daemon) call this via the registry
instead of talking to a specific adapter client.

#### Integration::supports_listening_sync

```rust
fn supports_listening_sync(&self) -> bool
```

Whether this integration can sync listening / progress into the library DB.

Default: `false`. Ranking treats listening as optional — when no
integration syncs (or recommend filters exclude it), discovery still
runs on owned-library taste alone.

#### Integration::sync_listening_progress

```rust
fn sync_listening_progress<'life0, 'life1, 'async_trait>(
    &'life0 self,
    _library: &'life1 LibraryStore,
) -> Pin<Box<dyn Future<Output = Result<usize>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Pull listening / progress rows into the generic `listening_progress` table.

Implementations must tag rows with [`Self::id`](#integrationid) as `provider`. Hosts
fan this out via [`crate::IntegrationRegistry::sync_listening_progress_all`](#integrationregistrysync_listening_progress_all)
so one or more integrations can contribute.

Default: unsupported.

#### Integration::diagnose

```rust
fn diagnose<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Human-readable connectivity probe for CLI / ops (`integrations test`).

Default: summarizes [`Self::health`](#integrationhealth).

#### Integration::portal_brand

```rust
fn portal_brand(&self) -> Option<Brand>
```

Optional portal brand for credential-login / connection UI.

Owned by the integration plugin; hosts must not hardcode provider colors.

#### Integration::provided_oidc_clients

```rust
fn provided_oidc_clients<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<ProvidedOidcClient>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Plugin-provided Bookclerk-as-IdP client templates (`oidcClients` RPC).

Empty when the guest is not a relying party or the method is unsupported.

## Structs

### Brand

```rust
pub struct Brand {
    pub id: &'static str,
    pub name: &'static str,
    pub bg: &'static str,
    pub fg: &'static str,
    pub accent: &'static str,
    pub icon_url: &'static str,
}
```

Visual identity for a store or integration in the portal UI.

**Fields**

- `id`: Stable identifier for this item.
- `name`: Operator-facing brand name shown on portal connect buttons.
- `bg`: CSS background color for portal connect buttons (`#RRGGBB`).
- `fg`: CSS foreground / text color for portal connect buttons.
- `accent`: Optional accent (border / focus).
- `icon_url`: Remote favicon / app-icon URL (not stored in-repo).

#### Brand::logo_href

```rust
pub fn logo_href(&self) -> &'static str
```

Favicon URL for `<img src>`.

Implements: `Clone`, `Copy`, `Debug`, `From<SourceBrand>`, `Serialize`

### PortalState

```rust
pub struct PortalState {
    pub config: Arc<RwLock<Config>>,
    pub library: Arc<RwLock<LibraryStore>>,
    pub integrations: Arc<RwLock<IntegrationRegistry>>,
    pub sources: Arc<RwLock<SourceRegistry>>,
}
```

Shared state for portal handlers.

**Fields**

- `config`: Loaded Bookclerk configuration shared with portal handlers.
- `library`: Open library store used for identity / ticket / session rows.
- `integrations`: Configured outbound integration registry.
- `sources`: Content-source registry used for portal store connect / OAuth.

Implements: `Clone`

### IntegrationRegistry

```rust
pub struct IntegrationRegistry {
    /* private fields */
}
```

Fan-out registry for configured integrations.

#### IntegrationRegistry::new

```rust
pub fn new() -> Self
```

Creates an empty registry with no integrations registered.

##### Returns

Newly constructed `new` value.

#### IntegrationRegistry::register

```rust
pub fn register(&mut self, integration: Arc<dyn Integration>)
```

Adds an integration to this registry (later entries keep their order).

Silently refuses registration once [`MAX_REGISTERED_INTEGRATIONS`](#max_registered_integrations) is
reached so health fan-out cannot allocate from an unbounded discovery set.

##### Arguments

* `integration` - Integration instance to register.

#### IntegrationRegistry::take_instance

```rust
pub fn take_instance(&mut self, plugin_instance_id: &str) -> Vec<Arc<dyn Integration>>
```

Removes every registration of `plugin_instance_id`.

Other instances of the same plugin key stay. The caller stops each
returned integration before publishing a replacement.

#### IntegrationRegistry::get

```rust
pub fn get(&self, id: &str) -> Option<Arc<dyn Integration>>
```

Returns the integration with this id, if registered.

##### Arguments

* `id` - Stable id to look up.

##### Returns

`Some(...)` when found / applicable; otherwise `None`.

#### IntegrationRegistry::all

```rust
pub fn all(&self) -> &[Arc<dyn Integration>]
```

Returns every registered integration in registration order.

#### IntegrationRegistry::is_empty

```rust
pub fn is_empty(&self) -> bool
```

Returns true when no integrations are registered.

#### IntegrationRegistry::start_all

```rust
pub async fn start_all(&self, ctx: IntegrationContext) -> Result<()>
```

Start all integrations (background watchers).

##### Arguments

* `ctx` - Lifecycle context (e.g. external-user callback).

##### Returns

The successful result value for this operation.

##### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

#### IntegrationRegistry::stop_all

```rust
pub async fn stop_all(&self)
```

Stop all integrations (background watchers). Errors are logged, not fatal.

#### IntegrationRegistry::stop_except

```rust
pub async fn stop_except(&self, keep: &BTreeSet<String>)
```

Stops integrations that are not a present deployed instance.

`keep` is plugin instance ids. Config reload reattaches those guests
onto the replacement registry and leaves them running. A transitional
integration is stopped even when its plugin key has a deployment.
Errors are logged, not fatal.

#### IntegrationRegistry::health_all

```rust
pub async fn health_all(&self) -> Vec<IntegrationHealth>
```

Probes every registered integration and returns one health row each.

#### IntegrationRegistry::credential_login_providers

```rust
pub fn credential_login_providers(&self) -> Vec<Arc<dyn Integration>>
```

Integrations that currently offer portal username/password login.

#### IntegrationRegistry::listening_sync_providers

```rust
pub fn listening_sync_providers(&self) -> Vec<Arc<dyn Integration>>
```

Integrations that can sync listening / progress.

#### IntegrationRegistry::sync_listening_progress_all

```rust
pub async fn sync_listening_progress_all(
    &self,
    library: &LibraryStore,
) -> SyncListeningSummary
```

Sync listening progress from every capable integration into the library DB.

Individual failures are recorded in the summary and do not abort siblings.

##### Arguments

* `library` - Open library store used for reads/writes.

##### Returns

`crate::types::SyncListeningSummary` result.

Implements: `Clone`, `Default`

### MintedClaimTicket

```rust
pub struct MintedClaimTicket {
    pub record: ClaimTicketRecord,
    pub token: String,
    pub identity: PortalIdentity,
    pub portal_url: Option<String>,
}
```

Freshly minted claim ticket (includes plaintext once).

**Fields**

- `record`: Persisted claim-ticket row (hash only; no plaintext token).
- `token`: Plaintext secret shown once at mint time; only a hash is persisted.
- `identity`: Portal identity bound to this ticket.
- `portal_url`: Shareable SPA claim URL when `public_origin` is configured.

Implements: `Clone`, `Debug`

### EventSubscription

```rust
pub struct EventSubscription {
    pub event_type: String,
    pub schema_versions: Vec<u32>,
    pub supports_suspend: bool,
    pub resource_class: String,
    pub filter: Option<Value>,
}
```

Declared durable `onEvent` subscription (mirrors `plugin.toml`).

**Fields**

- `event_type`: Event type (`book_acquired`).
- `schema_versions`: Schema versions this guest can consume.
- `supports_suspend`: Whether `EventResult::Suspended` is advertised for this type.
- `resource_class`: Concurrency class copied onto deliveries (default `"network"`).
- `filter`: Optional host-owned payload object filter (top-level key equality).

#### EventSubscription::new

```rust
pub fn new(event_type: impl Into<String>, schema_versions: Vec<u32>) -> Self
```

Type + schema subscription with the default `network` resource class.

Implements: `Clone`, `Debug`, `PartialEq`

### IntegrationContext

```rust
pub struct IntegrationContext {
    pub on_external_user: Option<Arc<dyn Fn(ExternalUser) + Send + Sync>>,
}
```

Context passed when starting background watchers.

**Fields**

- `on_external_user`: Optional callback when an external user is observed (claim minting).

Implements: `Clone`, `Default`

### ProvidedOidcClient

```rust
pub struct ProvidedOidcClient {
    pub client_id: String,
    pub display_name: String,
    pub callback_path: String,
    pub public_client: bool,
    pub default_scopes: Vec<String>,
    pub issue_refresh_token: bool,
    pub origin_config_key: String,
}
```

Host-side copy of a guest `oidcClients` template (no ABI crate dependency).

**Fields**

- `client_id`: OAuth `client_id`.
- `display_name`: Operator-facing card title.
- `callback_path`: Path appended to the plugin origin.
- `public_client`: Public PKCE when true.
- `default_scopes`: Scopes for first materialization.
- `issue_refresh_token`: Whether new rows may issue refresh tokens.
- `origin_config_key`: Dotted config key for the player origin.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### QueueSubscription

```rust
pub struct QueueSubscription {
    pub queue: String,
    pub max_retries: u32,
    pub dead_letter_queue: Option<String>,
    pub retry_delay_secs: u32,
}
```

Declared `[[queues.consumers]]` row (mirrors `plugin.toml`): the queue the
integration's `queue(batch)` trigger consumes and its redelivery policy.

**Fields**

- `queue`: Queue name.
- `max_retries`: Redeliveries before a message is dead-lettered.
- `dead_letter_queue`: Queue that receives a copy once `max_retries` is exhausted.
- `retry_delay_secs`: Seconds a retried message waits when the consumer names no delay.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ExternalUser

```rust
pub struct ExternalUser {
    pub provider: String,
    pub external_user_id: String,
    pub display_name: Option<String>,
    pub access_token: Option<String>,
}
```

External user resolved via integration credential login or watcher.

**Fields**

- `provider`: Integration / storefront id that owns this external identity.
- `external_user_id`: Remote system's user id (never a Bookclerk user id).
- `display_name`: Optional human-facing name from the remote system.
- `access_token`: Ephemeral token from the remote system (e.g. ABS JWT). Never persisted.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### IntegrationHealth

```rust
pub struct IntegrationHealth {
    pub id: String,
    pub enabled: bool,
    pub ok: bool,
    pub detail: Option<String>,
}
```

Health snapshot for one integration.

**Fields**

- `id`: Integration id this health row describes (`audiobookshelf`, …).
- `enabled`: When true, this feature or account is active.
- `ok`: When true, the last health probe succeeded.
- `detail`: Optional human-readable health or error detail.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### ListeningProgressSnapshot

```rust
pub struct ListeningProgressSnapshot {
    pub external_user_id: String,
    pub external_item_id: String,
    pub identity_id: Option<i64>,
    pub title: Option<String>,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub progress: Option<f64>,
    pub current_time_seconds: Option<f64>,
    pub duration_seconds: Option<f64>,
    pub is_finished: bool,
    pub last_listened_at: Option<DateTime<Utc>>,
}
```

One listening / progress row from any integration (ABS, plugins, …).

Hosts upsert these into the generic `listening_progress` table; ranking never
sees the originating adapter—only these fields.

**Fields**

- `external_user_id`: Remote system's user id (never a Bookclerk user id).
- `external_item_id`: Remote library item id for progress matching.
- `identity_id`: Bookclerk portal identity row id when already linked.
- `title`: Display title as shown on the storefront or library card.
- `authors`: Comma-separated author names when the storefront provides them.
- `asin`: Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn`: Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `progress`: Fraction complete in `[0.0, 1.0]` when the remote reports it.
- `current_time_seconds`: Playback position in seconds from the start of the title.
- `duration_seconds`: Total title duration in seconds when known.
- `is_finished`: When true, the remote marks this title as finished.
- `last_listened_at`: UTC timestamp of the last playback event when known.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### SyncListeningProviderResult

```rust
pub struct SyncListeningProviderResult {
    pub id: String,
    pub upserted: usize,
    pub error: Option<String>,
}
```

Per-integration outcome from [`crate::IntegrationRegistry::sync_listening_progress_all`](#integrationregistrysync_listening_progress_all).

**Fields**

- `id`: Stable identifier for this item.
- `upserted`: Number of listening_progress rows written or updated.
- `error`: Operator-facing error text when this step failed; `None` on success.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### SyncListeningSummary

```rust
pub struct SyncListeningSummary {
    pub upserted: usize,
    pub by_provider: Vec<SyncListeningProviderResult>,
}
```

Result of syncing listening progress across one or more integrations.

**Fields**

- `upserted`: Number of listening_progress rows written or updated.
- `by_provider`: Per-integration breakdown of upsert counts and errors.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

## Enums

### IntegrationError

```rust
pub enum IntegrationError {
    Api { status: u16, message: String },
    Library(LibraryError),
    Config(ConfigError),
    Other(Error),
    Message(String),
}
```

Errors from outbound integrations and the connect portal.

**Variants**

- `Api`: Remote HTTP API returned a non-success status.
  - `status`: HTTP status code from the remote API.
  - `message`: Response body or summarized error text.
- `Library`: Error propagated from [`bookclerk_library`](../bookclerk_library/index.md).
- `Config`: Error propagated from [`bookclerk_config`](../bookclerk_config/index.md).
- `Other`: Opaque error wrapped from `anyhow`.
- `Message`: Operator-facing error text with no structured code.

#### IntegrationError::message

```rust
pub fn message(msg: impl Into<String>) -> Self
```

Builds an error from a human-readable message string.

##### Arguments

* `msg` - Operator-facing error text.

##### Returns

Updated `Self` for chaining.

#### IntegrationError::api

```rust
pub fn api(status: u16, message: impl Into<String>) -> Self
```

Builds an [`IntegrationError::Api`](#integrationerror) from an HTTP status and response text.

##### Arguments

* `status` - HTTP status code from the remote API.
* `message` - Response body or summarized error text.

##### Returns

Updated `Self` for chaining.

Implements: `Debug`, `Display`, `Error`, `From<ConfigError>`, `From<Error>`, `From<LibraryError>`

## Functions

### from_config

```rust
pub fn from_config(_config: &Config) -> Result<IntegrationRegistry>
```

Construct an empty registry (no in-process adapters).

Hosts that also load plugins should prefer
`bookclerk_plugin_host::load_integrations`.

#### Arguments

* `config` - Loaded Bookclerk configuration.

#### Returns

On success, the inner `IntegrationRegistry` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### register_builtins

```rust
pub fn register_builtins(
    _config: &Config,
    _registry: &mut IntegrationRegistry,
) -> Result<()>
```

Register first-party integrations into an existing registry.

This function is intentionally a no-op; ABS and other adapters load as
external guests through `bookclerk_plugin_host::load_integrations`.

#### Arguments

* `config` - Loaded Bookclerk configuration.
* `registry` - Configured content-source or integration registry.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### emit_book_acquired

```rust
pub async fn emit_book_acquired(
    library: &LibraryStore,
    product_or_uuid: &str,
    storage_key: &str,
) -> Result<PublishDomainEventOutcome>
```

Look up the acquired book and publish `book_acquired` to the durable outbox.

Prefer `bookclerk_library::LibraryStore::set_acquire_status`, which publishes
in the same transaction as the acquire state change. This helper is a
fail-closed catch-up for callers that already committed the book row.

#### Errors

Returns when the library lookup or outbox insert fails, including a missing
book row (so the caller cannot report success without an outbox record).

### match_book_uuid

```rust
pub async fn match_book_uuid(
    library: &LibraryStore,
    asin: Option<&str>,
    isbn: Option<&str>,
    title: Option<&str>,
) -> Result<Option<String>>
```

Best-effort match to an owned library book via ASIN, ISBN, or exact title.

#### Arguments

* `library` - Open library store used for reads/writes.
* `asin` - Optional Audible / Amazon ASIN.
* `isbn` - Optional ISBN (any punctuation; normalized internally).
* `title` - Display title.

#### Returns

On success, the inner `Option<String>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### upsert_listening_snapshots

```rust
pub async fn upsert_listening_snapshots(
    library: &LibraryStore,
    provider: &str,
    items: &[ListeningProgressSnapshot],
) -> Result<usize>
```

Upsert provider-agnostic listening snapshots into `listening_progress`.

`provider` should be the integration id (`audiobookshelf`, plugin id, …).

#### Arguments

* `library` - Open library store used for reads/writes.
* `provider` - Integration id.
* `items` - `items` input for this call.

#### Returns

On success, the inner `usize` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### mint_for_external_user

```rust
pub async fn mint_for_external_user(
    library: &LibraryStore,
    config: &Config,
    user: &ExternalUser,
    created_by: &str,
) -> Result<MintedClaimTicket>
```

Mint a claim ticket for an external user (daemon watcher / CLI).

#### Arguments

* `library` - Open library store used for reads/writes.
* `config` - Loaded Bookclerk configuration.
* `user` - External user identity from an integration login/watcher.
* `created_by` - Actor string recorded on the claim ticket (`daemon`, CLI user, …).

#### Returns

On success, `crate::Result<crate::tickets::MintedClaimTicket>`.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### portal_identity_from_headers

```rust
pub async fn portal_identity_from_headers(
    library: &LibraryStore,
    headers: &HeaderMap,
) -> Option<PortalIdentity>
```

Resolve a portal identity from the request cookie, if present and valid.

#### Arguments

* `library` - Open library store used for reads/writes.
* `headers` - Incoming HTTP headers (cookie lookup).

#### Returns

`Some(...)` when found / applicable; otherwise `None`.

### portal_spa_router

```rust
pub fn portal_spa_router(state: PortalState) -> Router
```

SPA-facing portal API. Nest under `/api/portal`.

#### Arguments

* `state` - Shared portal / handler state.

#### Returns

`Router` result.

### redeem_lose_next_responses

```rust
pub fn redeem_lose_next_responses(n: i32)
```

Drop the next N successful redeem HTTP responses after the DB commit.

Production stays at 0. Tests inject a committed-but-lost first reply so the
retry can recover the receipt through the real handler.

### generate_token

```rust
pub fn generate_token() -> String
```

Generate a URL-safe random token.

### identity_from_session

```rust
pub async fn identity_from_session(
    library: &LibraryStore,
    raw_session: &str,
) -> Result<Option<PortalIdentity>>
```

Resolve identity from a portal session cookie value.

#### Arguments

* `library` - Open library store used for reads/writes.
* `raw_session` - Plaintext portal session cookie value.

#### Returns

On success, the inner `Option<PortalIdentity>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### mint_claim_ticket

```rust
pub async fn mint_claim_ticket(
    library: &LibraryStore,
    integrations: &IntegrationsConfig,
    user: &ExternalUser,
    created_by: &str,
) -> Result<MintedClaimTicket>
```

Ensure identity exists and mint a claim ticket.

#### Arguments

* `library` - Open library store used for reads/writes.
* `integrations` - Integrations config (TTL, public origin, …).
* `user` - External user identity from an integration login/watcher.
* `created_by` - Actor string recorded on the claim ticket (`daemon`, CLI user, …).

#### Returns

On success, the inner `MintedClaimTicket` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### redeem_ticket_to_session

```rust
pub async fn redeem_ticket_to_session(
    library: &LibraryStore,
    integrations: &IntegrationsConfig,
    raw_ticket: &str,
    nonce: &str,
) -> Result<(String, PortalIdentity)>
```

Redeem a claim ticket into a portal session cookie value (plaintext).

#### Arguments

* `library` - Open library store used for reads/writes.
* `integrations` - Integrations config (TTL, public origin, …).
* `raw_ticket` - Plaintext claim ticket from the SPA / share URL.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### redeem_ticket_to_session_with_client

```rust
pub async fn redeem_ticket_to_session_with_client(
    library: &LibraryStore,
    integrations: &IntegrationsConfig,
    raw_ticket: &str,
    nonce: &str,
    client: Option<&SessionClientInfo>,
    password_hash: Option<&str>,
    password_fingerprint: Option<&str>,
) -> Result<(String, PortalIdentity)>
```

Redeem a claim ticket and mint a portal session with optional client metadata.

`password_hash` is applied in the same transaction as ticket consume and
session insert, and only while the ticket-bound local user has no password.
`password_fingerprint` is hashed into the atomic request so a retry with
a freshly salted Argon2id encoding still matches the receipt.

The raw session token is derived from the claim ticket, browser nonce, and
process DEK so a lost atomic reply plus a new HTTP request reuse the
same operation id. Possession of the used magic link without the nonce
cannot recover the session.

#### Errors

Returns an error when the operation fails.

### session_for_identity

```rust
pub async fn session_for_identity(
    library: &LibraryStore,
    integrations: &IntegrationsConfig,
    identity: &PortalIdentity,
) -> Result<String>
```

Create a portal session for an already-resolved identity (credential login).

#### Arguments

* `library` - Open library store used for reads/writes.
* `integrations` - Integrations config (TTL, public origin, …).
* `identity` - Already-resolved portal identity.

#### Returns

On success, the inner `String` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### session_for_identity_with_client

```rust
pub async fn session_for_identity_with_client(
    library: &LibraryStore,
    integrations: &IntegrationsConfig,
    identity: &PortalIdentity,
    client: Option<&SessionClientInfo>,
) -> Result<String>
```

Create a portal session for an identity with optional client metadata.

#### Errors

Returns an error when the operation fails.

### ticket_portal_url

```rust
pub fn ticket_portal_url(integrations: &IntegrationsConfig, token: &str) -> Option<String>
```

Build a shareable SPA claim URL when `public_origin` is configured.

Opens the Bookclerk GUI invite page (`/invite?ticket=`).

#### Arguments

* `integrations` - Integrations config (TTL, public origin, …).
* `token` - Plaintext claim or session token.

#### Returns

`Some(...)` when found / applicable; otherwise `None`.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, IntegrationError>;
```

Result alias for this crate.

## Constants

### MAX_REGISTERED_INTEGRATIONS

```rust
pub const MAX_REGISTERED_INTEGRATIONS: usize = 256;
```

Hard ceiling on registered integrations (plugin discovery is user-influenced).
