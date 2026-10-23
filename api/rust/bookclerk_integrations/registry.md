**bookclerk_integrations > registry**

# Module: registry

## Contents

**Structs**

- [`IntegrationRegistry`](#integrationregistry) - Fan-out registry for configured integrations.

**Constants**

- [`MAX_REGISTERED_INTEGRATIONS`](#max_registered_integrations) - Hard ceiling on registered integrations (plugin discovery is user-influenced).

---

## bookclerk_integrations::registry::IntegrationRegistry

*Struct*

Fan-out registry for configured integrations.

**Methods:**

- `fn new() -> Self` - Creates an empty registry with no integrations registered.
- `fn register(self: & mut Self, integration: Arc<dyn Integration>)` - Adds an integration to this registry (later entries keep their order).
- `fn take_instance(self: & mut Self, plugin_instance_id: &str) -> Vec<Arc<dyn Integration>>` - Removes every registration of `plugin_instance_id`.
- `fn get(self: &Self, id: &str) -> Option<Arc<dyn Integration>>` - Returns the integration with this id, if registered.
- `fn all(self: &Self) -> &[Arc<dyn Integration>]` - Returns every registered integration in registration order.
- `fn is_empty(self: &Self) -> bool` - Returns true when no integrations are registered.
- `fn start_all(self: &Self, ctx: IntegrationContext) -> Result<()>` - Start all integrations (background watchers).
- `fn stop_all(self: &Self)` - Stop all integrations (background watchers). Errors are logged, not fatal.
- `fn stop_except(self: &Self, keep: &BTreeSet<String>)` - Stops integrations that are not a present deployed instance.
- `fn health_all(self: &Self) -> Vec<IntegrationHealth>` - Probes every registered integration and returns one health row each.
- `fn credential_login_providers(self: &Self) -> Vec<Arc<dyn Integration>>` - Integrations that currently offer portal username/password login.
- `fn listening_sync_providers(self: &Self) -> Vec<Arc<dyn Integration>>` - Integrations that can sync listening / progress.
- `fn sync_listening_progress_all(self: &Self, library: &bookclerk_library::LibraryStore) -> crate::types::SyncListeningSummary` - Sync listening progress from every capable integration into the library DB.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> IntegrationRegistry`
- **Default**
  - `fn default() -> IntegrationRegistry`



## bookclerk_integrations::registry::MAX_REGISTERED_INTEGRATIONS

*Constant*: `usize`

Hard ceiling on registered integrations (plugin discovery is user-influenced).



