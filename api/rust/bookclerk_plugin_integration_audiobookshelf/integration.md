**bookclerk_plugin_integration_audiobookshelf > integration**

# Module: integration

## Contents

**Structs**

- [`AbsIntegration`](#absintegration) - If required config (API key / base URL) is missing, the adapter stays
- [`AudiobookshelfConfig`](#audiobookshelfconfig) - Audiobookshelf outbound integration.

---

## bookclerk_plugin_integration_audiobookshelf::integration::AbsIntegration

*Struct*

If required config (API key / base URL) is missing, the adapter stays
registered, reports unhealthy, and refuses operational calls so the
misconfiguration is visible.

**Methods:**

- `fn new(config: AudiobookshelfConfig) -> Result<Self>` - Build an ABS integration. Prefer `Self::from_config` — this only fails
- `fn from_config(config: AudiobookshelfConfig) -> Self` - Construct from config; missing credentials become a soft config error.
- `fn client(self: &Self) -> Option<&AbsApiClient>` - HTTP or RPC client used for outbound requests.
- `fn config(self: &Self) -> &AudiobookshelfConfig` - Config.
- `fn config_error(self: &Self) -> Option<&str>` - Config error.
- `fn scan_now(self: &Self, force: bool) -> Result<()>` - Trigger a library scan (optional force).

**Trait Implementations:**

- **Integration**
  - `fn id(self: &Self) -> &str`
  - `fn display_name(self: &Self) -> &str`
  - `fn start(self: &'life0 Self, ctx: IntegrationContext) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn stop(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn deliver_domain_event(self: &'life0 Self, event: DomainEvent) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn event_subscriptions(self: &Self) -> Vec<bookclerk_integrations::EventSubscription>`
  - `fn health(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn authenticate_user(self: &'life0 Self, username: &'life1 str, password: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn supports_credential_login(self: &Self) -> bool`
  - `fn supports_library_scan(self: &Self) -> bool`
  - `fn scan_library(self: &'life0 Self, force: bool) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn supports_listening_sync(self: &Self) -> bool`
  - `fn sync_listening_progress(self: &'life0 Self, library: &'life1 bookclerk_library::LibraryStore) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn diagnose(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn portal_brand(self: &Self) -> Option<Brand>`
  - `fn provided_oidc_clients(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_integration_audiobookshelf::integration::AudiobookshelfConfig

*Struct*

Audiobookshelf outbound integration.

The guest's `CONFIG` / `SECRETS`, as declared by `[[settings]]` in
`plugin.toml` and resolved by the host from the plugin instance document.

**Fields:**
- `base_url: String` - Audiobookshelf server origin without trailing slash (e.g. `https://abs.example`).
- `api_key: Option<String>` - ABS API token (a `secret` setting; delivered in `SECRETS`).
- `library_id: Option<String>` - Target ABS library UUID when the server has multiple libraries.
- `watch_users: bool` - When true, poll ABS users for listening-progress sync.
- `notify_scan_on_acquire: bool` - When true, ask ABS to rescan after a successful acquire.
- `allow_credential_login: bool` - When true, allow signing in to the Bookclerk portal with ABS credentials.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &AudiobookshelfConfig) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AudiobookshelfConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Self`



