**bookclerk_plugin_host > host > source**

# Module: host::source

## Contents

**Structs**

- [`ExternalSource`](#externalsource) - External content source backed by a discovered plugin binary.

---

## bookclerk_plugin_host::host::source::ExternalSource

*Struct*

External content source backed by a discovered plugin binary.

**Methods:**

- `fn spawn_prepared(plugin: &DiscoveredPlugin, config: &Config, services: SessionServices, prepared: crate::instance_bindings::PreparedOpen) -> Result<Self>` - Spawn with bindings already resolved for one plugin instance.
- `fn with_handed_credentials(self: Self, credentials: Vec<bookclerk_plugin_sdk::AccountCredential>) -> Self` - Uses only `credentials` (an owner cluster handed them with a job)
- `fn scan_handed(self: &Self, opts: ScanOptions) -> bookclerk_source::Result<Vec<bookclerk_plugin_sdk::ScanBook>>` - Scans with the handed credentials and answers the guest's rows
- `fn bind_plugin_instance(self: & mut Self, plugin_instance_id: &str)` - Records the plugin instance id used as this source's registry address.
- `fn check_health(self: &Self) -> Result<()>` - Storefront health RPC.
- `fn guest_pid(self: &Self) -> Option<u32>` - Native guest pid, when the session recorded one.

**Trait Implementations:**

- **ContentSource**
  - `fn id(self: &Self) -> &str`
  - `fn plugin_key(self: &Self) -> &str`
  - `fn plugin_instance_id(self: &Self) -> Option<&str>`
  - `fn guest_pid(self: &Self) -> Option<u32>`
  - `fn display_name(self: &Self) -> &str`
  - `fn aliases(self: &Self) -> &'static [&'static str]`
  - `fn portal_auth_mode(self: &Self) -> PortalAuthMode`
  - `fn portal_brand(self: &Self) -> SourceBrand`
  - `fn sort_key(self: &Self) -> u32`
  - `fn login(self: &'life0 Self, scope: &'life1 SourceScope, opts: LoginOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn login_with_oauth_progress(self: &'life0 Self, scope: &'life1 SourceScope, opts: LoginOptions, on_progress: &'life2 dyn Fn) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_accounts(self: &'life0 Self, scope: &'life1 SourceScope) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn scan(self: &'life0 Self, scope: &'life1 SourceScope, opts: ScanOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn fetch_title(self: &'life0 Self, scope: &'life1 SourceScope, account_id: &'life2 str, title_id: &'life3 str, opts: &'life4 FetchOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn search_catalog(self: &'life0 Self, opts: &'life1 CatalogSearchOpts) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn catalog_detail(self: &'life0 Self, product_id: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn expand_candidates(self: &'life0 Self, seed: &'life1 ExpandSeed, limit: usize) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn purchase_hint(self: &'life0 Self, opts: &'life1 PurchaseHintOpts) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_deals(self: &'life0 Self, limit: usize) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



