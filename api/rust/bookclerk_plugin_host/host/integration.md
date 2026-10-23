**bookclerk_plugin_host > host > integration**

# Module: host::integration

## Contents

**Structs**

- [`ExternalIntegration`](#externalintegration) - External integration backed by a discovered plugin binary.

---

## bookclerk_plugin_host::host::integration::ExternalIntegration

*Struct*

External integration backed by a discovered plugin binary.

**Methods:**

- `fn spawn_prepared(plugin: &DiscoveredPlugin, config: &Config, services: SessionServices, prepared: crate::instance_bindings::PreparedOpen, allow_credential_login: bool) -> Result<Self>` - Spawn with bindings already resolved for one plugin instance.
- `fn bind_plugin_instance(self: & mut Self, plugin_instance_id: &str)` - Records the plugin instance id used to recognize this guest on reload.
- `fn check_health(self: &Self) -> Result<()>` - Integration health RPC.

**Trait Implementations:**

- **Integration**
  - `fn id(self: &Self) -> &str`
  - `fn plugin_key(self: &Self) -> &str`
  - `fn plugin_instance_id(self: &Self) -> Option<&str>`
  - `fn display_name(self: &Self) -> &str`
  - `fn start(self: &'life0 Self, ctx: IntegrationContext) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn stop(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn deliver_domain_event(self: &'life0 Self, event: DomainEvent) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn deliver_domain_event_cancelable(self: &'life0 Self, event: DomainEvent, cancel: Arc<AtomicBool>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn event_subscriptions(self: &Self) -> Vec<EventSubscription>`
  - `fn queue_subscriptions(self: &Self) -> Vec<QueueSubscription>`
  - `fn deliver_queue_batch_cancelable(self: &'life0 Self, batch: bookclerk_integrations::QueueBatch, cancel: Arc<AtomicBool>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn health(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn supports_library_scan(self: &Self) -> bool`
  - `fn scan_library(self: &'life0 Self, force: bool) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn supports_listening_sync(self: &Self) -> bool`
  - `fn sync_listening_progress(self: &'life0 Self, library: &'life1 bookclerk_library::LibraryStore) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn diagnose(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn supports_credential_login(self: &Self) -> bool`
  - `fn authenticate_user(self: &'life0 Self, username: &'life1 str, password: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn portal_brand(self: &Self) -> Option<Brand>`
  - `fn provided_oidc_clients(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



