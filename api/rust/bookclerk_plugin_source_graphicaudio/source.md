**bookclerk_plugin_source_graphicaudio > source**

# Module: source

## Contents

**Structs**

- [`GraphicAudioSource`](#graphicaudiosource) - GraphicAudio storefront adapter implementing [`ContentSource`](../bookclerk_source/traits.md#bookclerk_sourcetraitscontentsource).

**Constants**

- [`ID`](#id) - Describe / config id for this store (`graphicaudio` in `[sources.graphicaudio]`).

---

## bookclerk_plugin_source_graphicaudio::source::GraphicAudioSource

*Struct*

GraphicAudio storefront adapter implementing [`ContentSource`](../bookclerk_source/traits.md#bookclerk_sourcetraitscontentsource).

Access path (`web` / `zip` / `device`) comes from config; see crate-level docs.

**Fields:**
- `access: crate::options::GraphicAudioAccess` - Configured access path (login + default fetch). Env may still override fetch.
- `bitrate: crate::options::GraphicAudioBitrate` - Device encode bitrate (`[sources.graphicaudio] bitrate`).
- `container: crate::options::GraphicAudioContainer` - ZIP SKU container preference (`[sources.graphicaudio] container`).

**Methods:**

- `fn new() -> Self` - Production GraphicAudio Access API + storefront origins (access=`web`).
- `fn with_base_url<impl Into<String>>(base_url: impl Trait) -> Self` - Override Access API base (wiremock / staging).
- `fn with_store_url<impl Into<String>>(self: Self, store_url: impl Trait) -> Self` - Override Magento storefront base (wiremock).
- `fn with_access(self: Self, access: GraphicAudioAccess) -> Self` - Set the configured access path (`[sources.graphicaudio] access`).
- `fn with_bitrate(self: Self, bitrate: GraphicAudioBitrate) -> Self` - Sets the Access App encode bitrate preference.
- `fn with_container(self: Self, container: GraphicAudioContainer) -> Self` - Sets the Magento ZIP container preference (`mp3` / `m4b` / auto).
- `fn with_fetch_mode(self: Self, mode: GraphicAudioAccess) -> Self` - Force a fetch path (bypasses config / env).
- `fn with_magento_password<impl Into<String>>(self: Self, password: impl Trait) -> Self` - Magento password for ZIP / Browser Player.
- `fn shared() -> Arc<Self>` - Arc-wrapped instance for [`bookclerk_source::SourceRegistry`](../bookclerk_source/registry.md#bookclerk_sourceregistrysourceregistry).
- `fn login_account(self: &Self, library: &SourceScope, opts: LoginOptions) -> Result<SourceAccount>` - Login and persist credentials to the DB.
- `fn delete_account(self: &Self, library: &SourceScope, account_id: &str) -> Result<()>` - Delete a GraphicAudio account from the DB.

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> GraphicAudioSource`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **ContentSource**
  - `fn id(self: &Self) -> &str`
  - `fn display_name(self: &Self) -> &str`
  - `fn aliases(self: &Self) -> &'static [&'static str]`
  - `fn portal_auth_mode(self: &Self) -> PortalAuthMode`
  - `fn portal_brand(self: &Self) -> SourceBrand`
  - `fn sort_key(self: &Self) -> u32`
  - `fn login(self: &'life0 Self, library: &'life1 SourceScope, opts: LoginOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_accounts(self: &'life0 Self, library: &'life1 SourceScope) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn scan(self: &'life0 Self, library: &'life1 SourceScope, opts: ScanOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn fetch_title(self: &'life0 Self, library: &'life1 SourceScope, account_id: &'life2 str, title_id: &'life3 str, opts: &'life4 FetchOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn config_options(self: &Self) -> &'static [bookclerk_source::SourceConfigOption]`
  - `fn search_catalog(self: &'life0 Self, opts: &'life1 CatalogSearchOpts) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn expand_candidates(self: &'life0 Self, seed: &'life1 ExpandSeed, limit: usize) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn purchase_hint(self: &'life0 Self, opts: &'life1 PurchaseHintOpts) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_source_graphicaudio::source::ID

*Constant*: `&str`

Describe / config id for this store (`graphicaudio` in `[sources.graphicaudio]`).



