**bookclerk_plugin_source_chirp > source**

# Module: source

## Contents

**Structs**

- [`ChirpSource`](#chirpsource) - Chirp storefront adapter implementing [`ContentSource`](../bookclerk_source/traits.md#bookclerk_sourcetraitscontentsource).

**Constants**

- [`ID`](#id) - Describe / config id for this store (`chirp` in `[sources.chirp]`).

---

## bookclerk_plugin_source_chirp::source::ChirpSource

*Struct*

Chirp storefront adapter implementing [`ContentSource`](../bookclerk_source/traits.md#bookclerk_sourcetraitscontentsource).

Construct with `ChirpSource::new`; use `ChirpSource::with_graphql_url`
for wiremock / staging.

**Methods:**

- `fn new() -> Self` - Returns a Chirp source aimed at the production GraphQL endpoint.
- `fn with_graphql_url<impl Into<String>>(graphql_url: impl Trait) -> Self` - Overrides the GraphQL endpoint (tests / staging).
- `fn shared() -> Arc<Self>` - Arc-wrapped instance for [`bookclerk_source::SourceRegistry`](../bookclerk_source/registry.md#bookclerk_sourceregistrysourceregistry).
- `fn login_account(self: &Self, library: &SourceScope, opts: LoginOptions) -> Result<SourceAccount>` - Logs in with email/password and seals credentials into `encrypted_secrets`.
- `fn delete_account(self: &Self, library: &SourceScope, account_id: &str) -> Result<()>` - Delete a Chirp account from the DB.

**Trait Implementations:**

- **ContentSource**
  - `fn id(self: &Self) -> &str`
  - `fn display_name(self: &Self) -> &str`
  - `fn portal_auth_mode(self: &Self) -> PortalAuthMode`
  - `fn portal_brand(self: &Self) -> SourceBrand`
  - `fn sort_key(self: &Self) -> u32`
  - `fn login(self: &'life0 Self, library: &'life1 SourceScope, opts: LoginOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_accounts(self: &'life0 Self, library: &'life1 SourceScope) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn scan(self: &'life0 Self, library: &'life1 SourceScope, opts: ScanOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn fetch_title(self: &'life0 Self, library: &'life1 SourceScope, account_id: &'life2 str, title_id: &'life3 str, opts: &'life4 FetchOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn search_catalog(self: &'life0 Self, opts: &'life1 CatalogSearchOpts) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn expand_candidates(self: &'life0 Self, seed: &'life1 ExpandSeed, limit: usize) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn purchase_hint(self: &'life0 Self, opts: &'life1 PurchaseHintOpts) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_deals(self: &'life0 Self, limit: usize) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> ChirpSource`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::source::ID

*Constant*: `&str`

Describe / config id for this store (`chirp` in `[sources.chirp]`).



