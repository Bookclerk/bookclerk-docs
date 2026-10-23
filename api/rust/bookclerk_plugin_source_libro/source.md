**bookclerk_plugin_source_libro > source**

# Module: source

## Contents

**Structs**

- [`LibroSource`](#librosource) - Libro.fm storefront adapter implementing [`ContentSource`](../bookclerk_source/traits.md#bookclerk_sourcetraitscontentsource).

**Constants**

- [`ID`](#id) - Describe / config id for this store (`libro` in `[sources.libro]`).

---

## bookclerk_plugin_source_libro::source::ID

*Constant*: `&str`

Describe / config id for this store (`libro` in `[sources.libro]`).



## bookclerk_plugin_source_libro::source::LibroSource

*Struct*

Libro.fm storefront adapter implementing [`ContentSource`](../bookclerk_source/traits.md#bookclerk_sourcetraitscontentsource).

Construct with `LibroSource::new`; use `LibroSource::with_base_url`
for wiremock / staging.

**Fields:**
- `container: crate::container::LibroContainer` - Preferred download container (`[sources.libro] container`).

**Methods:**

- `fn new() -> Self` - Production Libro.fm API origin.
- `fn with_base_url<impl Into<String>>(base_url: impl Trait) -> Self` - Override API base (wiremock / staging).
- `fn with_container(self: Self, container: LibroContainer) -> Self` - Sets the preferred download container (`m4b` / track zip).
- `fn shared() -> Arc<Self>` - Arc-wrapped instance for [`bookclerk_source::SourceRegistry`](../bookclerk_source/registry.md#bookclerk_sourceregistrysourceregistry).
- `fn login_account(self: &Self, library: &SourceScope, opts: LoginOptions) -> Result<SourceAccount>` - Logs in with email/password and seals credentials into `encrypted_secrets`.
- `fn delete_account(self: &Self, library: &SourceScope, account_id: &str) -> Result<()>` - Delete a Libro.fm account from the DB.

**Trait Implementations:**

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
  - `fn catalog_detail(self: &'life0 Self, product_id: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn expand_candidates(self: &'life0 Self, seed: &'life1 ExpandSeed, limit: usize) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn purchase_hint(self: &'life0 Self, opts: &'life1 PurchaseHintOpts) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> LibroSource`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



