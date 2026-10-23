**bookclerk_integrations > traits**

# Module: traits

## Contents

**Structs**

- [`EventSubscription`](#eventsubscription) - Declared durable `onEvent` subscription (mirrors `plugin.toml`).
- [`IntegrationContext`](#integrationcontext) - Context passed when starting background watchers.
- [`ProvidedOidcClient`](#providedoidcclient) - Host-side copy of a guest `oidcClients` template (no ABI crate dependency).
- [`QueueSubscription`](#queuesubscription) - Declared `[[queues.consumers]]` row (mirrors `plugin.toml`): the queue the

**Traits**

- [`Integration`](#integration) - Downstream consumer of acquired books / identity signals (e.g. Audiobookshelf).

---

## bookclerk_integrations::traits::EventSubscription

*Struct*

Declared durable `onEvent` subscription (mirrors `plugin.toml`).

**Fields:**
- `event_type: String` - Event type (`book_acquired`).
- `schema_versions: Vec<u32>` - Schema versions this guest can consume.
- `supports_suspend: bool` - Whether `EventResult::Suspended` is advertised for this type.
- `resource_class: String` - Concurrency class copied onto deliveries (default `"network"`).
- `filter: Option<serde_json::Value>` - Optional host-owned payload object filter (top-level key equality).

**Methods:**

- `fn new<impl Into<String>>(event_type: impl Trait, schema_versions: Vec<u32>) -> Self` - Type + schema subscription with the default `network` resource class.

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &EventSubscription) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> EventSubscription`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_integrations::traits::Integration

*Trait*

Downstream consumer of acquired books / identity signals (e.g. Audiobookshelf).

**Methods:**

- `id`: Display / CLI alias (`audiobookshelf`, …). Globally unique within one
- `plugin_key`: Provenance-qualified PluginKey when this adapter is an external guest.
- `plugin_instance_id`: Plugin instance that owns this deployed integration.
- `display_name`: Human-facing name for portal / CLI (defaults to `Self::id`).
- `start`: Start background tasks (user watchers, etc.).
- `stop`: Stop background tasks started by `Self::start`.
- `deliver_domain_event`: Deliver one versioned outbox [`DomainEvent`](../bookclerk_plugin_abi/rpc_types.md#bookclerk_plugin_abirpc_typesdomainevent) and return a durable
- `deliver_domain_event_cancelable`: Like `Self::deliver_domain_event`, aborting when `cancel` is set (fence loss).
- `event_subscriptions`: Durable outbox subscriptions. Empty means this integration is not a subscriber.
- `queue_subscriptions`: Declared `[[queues.consumers]]` rows. Empty means this integration
- `deliver_queue_batch_cancelable`: Deliver one ordered [`QueueBatch`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedqueuebatch) to the `queue(batch)` trigger and
- `health`: Probes connectivity and configuration for this integration.
- `supports_credential_login`: Whether this integration may appear as a portal username/password login.
- `authenticate_user`: Validate end-user credentials (portal return visits / self-service).
- `supports_library_scan`: Whether this integration can trigger a remote library scan.
- `scan_library`: Trigger a remote library scan (optional force rescan).
- `supports_listening_sync`: Whether this integration can sync listening / progress into the library DB.
- `sync_listening_progress`: Pull listening / progress rows into the generic `listening_progress` table.
- `diagnose`: Human-readable connectivity probe for CLI / ops (`integrations test`).
- `portal_brand`: Optional portal brand for credential-login / connection UI.
- `provided_oidc_clients`: Plugin-provided Bookclerk-as-IdP client templates (`oidcClients` RPC).



## bookclerk_integrations::traits::IntegrationContext

*Struct*

Context passed when starting background watchers.

**Fields:**
- `on_external_user: Option<std::sync::Arc<dyn Fn>>` - Optional callback when an external user is observed (claim minting).

**Trait Implementations:**

- **Default**
  - `fn default() -> IntegrationContext`
- **Clone**
  - `fn clone(self: &Self) -> IntegrationContext`



## bookclerk_integrations::traits::ProvidedOidcClient

*Struct*

Host-side copy of a guest `oidcClients` template (no ABI crate dependency).

**Fields:**
- `client_id: String` - OAuth `client_id`.
- `display_name: String` - Operator-facing card title.
- `callback_path: String` - Path appended to the plugin origin.
- `public_client: bool` - Public PKCE when true.
- `default_scopes: Vec<String>` - Scopes for first materialization.
- `issue_refresh_token: bool` - Whether new rows may issue refresh tokens.
- `origin_config_key: String` - Dotted config key for the player origin.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ProvidedOidcClient`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ProvidedOidcClient) -> bool`



## bookclerk_integrations::traits::QueueSubscription

*Struct*

Declared `[[queues.consumers]]` row (mirrors `plugin.toml`): the queue the
integration's `queue(batch)` trigger consumes and its redelivery policy.

**Fields:**
- `queue: String` - Queue name.
- `max_retries: u32` - Redeliveries before a message is dead-lettered.
- `dead_letter_queue: Option<String>` - Queue that receives a copy once `max_retries` is exhausted.
- `retry_delay_secs: u32` - Seconds a retried message waits when the consumer names no delay.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &QueueSubscription) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> QueueSubscription`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



