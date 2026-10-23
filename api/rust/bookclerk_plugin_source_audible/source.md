**bookclerk_plugin_source_audible > source**

# Module: source

## Contents

**Structs**

- [`AudibleSource`](#audiblesource) - Audible storefront adapter implementing [`ContentSource`](../bookclerk_source/traits.md#bookclerk_sourcetraitscontentsource).

**Constants**

- [`ID`](#id) - Describe / config id for this store (`audible` in `[sources.audible]`).

---

## bookclerk_plugin_source_audible::source::AudibleSource

*Struct*

Audible storefront adapter implementing [`ContentSource`](../bookclerk_source/traits.md#bookclerk_sourcetraitscontentsource).

Construct with `AudibleSource::new`; the host delivers the instance's
`bitrate` setting through `CONFIG` at open, and tests use
`AudibleSource::with_bitrate` directly.

**Fields:**
- `bitrate: bookclerk_config::AudioQuality` - License bitrate tier (the instance's `bitrate` setting).

**Methods:**

- `fn new() -> Self` - Returns an Audible source with default bitrate (`High` / config default).
- `fn with_bitrate(self: Self, bitrate: AudioQuality) -> Self` - Overrides the license bitrate tier used for download requests.

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AudibleSource`
- **Default**
  - `fn default() -> AudibleSource`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **ContentSource**
  - `fn id(self: &Self) -> &str`
  - `fn display_name(self: &Self) -> &str`
  - `fn portal_auth_mode(self: &Self) -> PortalAuthMode`
  - `fn portal_brand(self: &Self) -> SourceBrand`
  - `fn sort_key(self: &Self) -> u32`
  - `fn login(self: &'life0 Self, scope: &'life1 SourceScope, opts: LoginOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn login_with_oauth_progress(self: &'life0 Self, scope: &'life1 SourceScope, opts: LoginOptions, on_progress: &'life2 dyn Fn) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn import_credentials(self: &'life0 Self, scope: &'life1 SourceScope, path: &'life2 Path, opts: ImportCredentialsOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn revoke_credentials(self: &'life0 Self, scope: &'life1 SourceScope, account_id: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_accounts(self: &'life0 Self, scope: &'life1 SourceScope) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn scan(self: &'life0 Self, scope: &'life1 SourceScope, opts: ScanOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn fetch_title(self: &'life0 Self, scope: &'life1 SourceScope, account_id: &'life2 str, title_id: &'life3 str, opts: &'life4 FetchOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn config_options(self: &Self) -> &'static [bookclerk_source::SourceConfigOption]`
  - `fn search_catalog(self: &'life0 Self, opts: &'life1 CatalogSearchOpts) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn expand_candidates(self: &'life0 Self, seed: &'life1 ExpandSeed, limit: usize) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn purchase_hint(self: &'life0 Self, opts: &'life1 PurchaseHintOpts) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn inspect_title(self: &'life0 Self, scope: &'life1 SourceScope, account_id: &'life2 str, title_id: &'life3 str, opts: &'life4 FetchOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_source_audible::source::ID

*Constant*: `&str`

Describe / config id for this store (`audible` in `[sources.audible]`).



