# bookclerk_plugin_source_graphicaudio

GraphicAudio content source: auth, library sync, and DRM-free download.

Access paths (see `docs/contributing/research/source-candidates.md`), selected by
the `access` plugin setting (default `web`):
1. Browser Player (`/library`) — Magento session + CloudFront cookies
2. Magento ZIP (`My Downloadable Products`) — opt-in (`access = "zip"`)
3. Access App Retrofit API (`/access`) — opt-in device activation (`access = "device"`)

| Item | Kind | Summary |
| --- | --- | --- |
| [`GraphicAudioAuthFile`](#graphicaudioauthfile) | struct | Credential envelope for one GraphicAudio account. |
| [`MagentoCatalogProduct`](#magentocatalogproduct) | struct | One Magento catalog product (grid / related / product page). |
| [`DownloadLinks`](#downloadlinks) | struct | Lo/Hi download URLs from `api/links`. |
| [`GraphicAudioClient`](#graphicaudioclient) | struct | Authenticated GraphicAudio HTTP helper. |
| [`Product`](#product) | struct | One library / sample product from `api/products`. |
| [`TitleFetchRequest`](#titlefetchrequest) | struct | Parameters for `fetch_title_with_mode`. |
| [`DownloadableProduct`](#downloadableproduct) | struct | One row from My Downloadable Products. |
| [`LibraryItem`](#libraryitem) | struct | One owned title from Browser Player `content_library` HTML. |
| [`MagentoClient`](#magentoclient) | struct | Magento customer session + cookie jar (including Browser Player CloudFront cookies). |
| [`GraphicAudioSource`](#graphicaudiosource) | struct | GraphicAudio catalog client used by the guest's discovery entrypoints. |
| [`GraphicAudioError`](#graphicaudioerror) | enum | Failures from Magento / Access App auth, library listing, or download. |
| [`GraphicAudioAccess`](#graphicaudioaccess) | enum | How GraphicAudio fetches owned audio (the `access` plugin setting). |
| [`GraphicAudioBitrate`](#graphicaudiobitrate) | enum | GraphicAudio device encode preference (the `bitrate` plugin setting). |
| [`GraphicAudioContainer`](#graphicaudiocontainer) | enum | Preferred GraphicAudio ZIP SKU when `GraphicAudioAccess::Zip`. |
| [`catalog_http_client`](#catalog_http_client) | function | Shared HTTP client for public Magento catalog pages. |
| [`expand_from_product_id`](#expand_from_product_id) | function | Expand a owned Magento product id into related + series siblings. |
| [`expand_from_search`](#expand_from_search) | function | Expand via Magento catalog search (series name or title). |
| [`fetch_product_by_id`](#fetch_product_by_id) | function | Magento product page by numeric product id. |
| [`fetch_series_page`](#fetch_series_page) | function | Fetch titles listed on a series category page (includes series-set SKUs). |
| [`parse_catalog_grid`](#parse_catalog_grid) | function | Parse Magento product grid / related list items. |
| [`parse_related_products`](#parse_related_products) | function | Parse the Recommended / related products block on a product page. |
| [`search_catalog`](#search_catalog) | function | Search Magento catalog; often redirects to a matching series page. |
| [`search_catalog_page`](#search_catalog_page) | function | Like `search_catalog` with an optional Magento page index. |
| [`fetch_title_materials`](#fetch_title_materials) | function | Password env (same as CLI login) for Magento ZIP / Browser Player. Fetch one product id into `cache_dir` via `api/links` Hi/Lo URLs. |
| [`fetch_title_with_mode`](#fetch_title_with_mode) | function | Fetch owned audio for one product using the configured access path only (no ZIP→web→device cascade). |
| [`collect_account_books`](#collect_account_books) | function | Collect owned titles for one account (no DB writes). |
| [`product_to_scan_book`](#product_to_scan_book) | function | Map a GraphicAudio product JSON object to a structured scan row. |
| [`Result`](#result) | type alias | Result alias for GraphicAudio auth, sync, and download operations. |
| [`DEFAULT_BASE_URL`](#default_base_url) | constant | Default API origin (no trailing slash after `/access`). |
| [`LOGIN_PATH`](#login_path) | constant | Device activation / password login. |
| [`PRODUCTS_PATH`](#products_path) | constant | Library + sample products listing. |
| [`REMOVE_PATH`](#remove_path) | constant | Forget a device activation (`client_id` form field). |
| [`GA_ACCESS_ENV`](#ga_access_env) | constant | Preferred env override (`BOOKCLERK_GA_ACCESS`); falls back to `GA_FETCH_ENV`. |
| [`GA_FETCH_ENV`](#ga_fetch_env) | constant | Env override for which GraphicAudio access path to use (legacy name). |
| [`DEFAULT_STORE_URL`](#default_store_url) | constant | Default Magento storefront origin (no `/access` suffix). |
| [`GRAPHICAUDIO_SOURCE_ID`](#graphicaudio_source_id) | constant | Describe / config id for this store (`graphicaudio`). |

## Re-exports

- [`guest::guest_fetch_title`](guest.md#guest_fetch_title)
- [`guest::guest_fetch_title_rpc`](guest.md#guest_fetch_title_rpc)
- [`guest::guest_login`](guest.md#guest_login)
- [`guest::guest_login_rpc`](guest.md#guest_login_rpc)
- [`guest::guest_scan_page`](guest.md#guest_scan_page)
- [`guest::resolve_access`](guest.md#resolve_access)
- [`guest::resolve_access_base_url`](guest.md#resolve_access_base_url)
- [`guest::resolve_bitrate`](guest.md#resolve_bitrate)
- [`guest::resolve_container`](guest.md#resolve_container)
- [`guest::resolve_store_base_url`](guest.md#resolve_store_base_url)

## Modules

- [`guest`](guest.md): Guest-process helpers for the external GraphicAudio source plugin.

## Structs

### GraphicAudioAuthFile

```rust
pub struct GraphicAudioAuthFile {
    pub token: String,
    pub client_id: String,
    pub email: String,
    pub marketplace: String,
    pub label: Option<String>,
}
```

Credential envelope for one GraphicAudio account.

The guest returns it from login; the host seals it into `encrypted_secrets`
and passes it back on scan and fetch. Never written
to disk under `Accounts/`.

**Fields**

- `token`: Opaque device activation token from `POST activation/login`.
  Empty when the account was logged in with Magento-only access (`web`/`zip`).
- `client_id`: Device / client id sent at login (stable for this auth file; used by `device`).
- `email`: Sign-in email (also the Bookclerk account id).
- `marketplace`: Marketplace / locale hint stored for Bookclerk (defaults to `us`).
- `label`: Optional operator-facing account label in Accounts UI.

#### GraphicAudioAuthFile::account_id

```rust
pub fn account_id(&self) -> &str
```

Stable account id for library rows (email).

#### GraphicAudioAuthFile::has_device_token

```rust
pub fn has_device_token(&self) -> bool
```

True when an Access App device token is present.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### MagentoCatalogProduct

```rust
pub struct MagentoCatalogProduct {
    pub product_id: String,
    pub sku: Option<String>,
    pub title: String,
    pub url: Option<String>,
    pub series: Option<String>,
    pub cover_url: Option<String>,
}
```

One Magento catalog product (grid / related / product page).

**Fields**

- `product_id`: Product Identifier.
- `sku`: Sku.
- `title`: Title.
- `url`: URL.
- `series`: Series.
- `cover_url`: Cover URL.

#### MagentoCatalogProduct::is_series_set

```rust
pub fn is_series_set(&self) -> bool
```

Is series set.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DownloadLinks

```rust
pub struct DownloadLinks {
    pub lo: Option<String>,
    pub hi: Option<String>,
}
```

Lo/Hi download URLs from `api/links`.

**Fields**

- `lo`: Lo.
- `hi`: Hi.

#### DownloadLinks::preferred_url

```rust
pub fn preferred_url(&self) -> Option<&str>
```

Prefer high-quality URL, else low.

#### DownloadLinks::url_for_quality

```rust
pub fn url_for_quality(&self, prefer_hi: bool) -> Option<&str>
```

Select Hi or Lo based on whether high quality is preferred.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### GraphicAudioClient

```rust
pub struct GraphicAudioClient {
    /* private fields */
}
```

Authenticated GraphicAudio HTTP helper.

#### GraphicAudioClient::new

```rust
pub fn new(base_url: impl Into<String>) -> Self
```

Build a client pointed at `base_url` (trailing slash stripped).

#### GraphicAudioClient::with_http

```rust
pub fn with_http(self, http: HttpClient) -> Self
```

Override the HTTP client (tests / custom timeouts).

#### GraphicAudioClient::with_token

```rust
pub fn with_token(self, token: impl Into<String>) -> Self
```

Attach an activation token for subsequent requests.

#### GraphicAudioClient::token

```rust
pub fn token(&self) -> Option<&str>
```

Current token, if any.

#### GraphicAudioClient::base_url

```rust
pub fn base_url(&self) -> &str
```

Base URL in use.

#### GraphicAudioClient::login

```rust
pub async fn login(
    &mut self,
    username: &str,
    password: &str,
    client_id: &str,
) -> Result<String>
```

`POST activation/login` — returns the opaque token string.

##### Errors

Returns an error when the operation fails.

#### GraphicAudioClient::products

```rust
pub async fn products(&self) -> Result<Vec<Product>>
```

`GET api/products` — owned titles plus promotional samples.

##### Errors

Returns an error when the operation fails.

#### GraphicAudioClient::links

```rust
pub async fn links(&self, product_id: &str) -> Result<DownloadLinks>
```

`GET api/links?product=` — Lo/Hi plain media URLs.

##### Errors

Returns an error when the operation fails.

#### GraphicAudioClient::download_bytes

```rust
pub async fn download_bytes(&self, url: &str) -> Result<Bytes>
```

Download bytes from an absolute media URL (no Authorization).

Prefer [`Self::download_to_path`](#graphicaudioclientdownload_to_path) for large Hi/Lo media (~100MB–500MB+).

##### Errors

Returns an error when the operation fails.

#### GraphicAudioClient::download_to_path

```rust
pub async fn download_to_path(&self, url: &str, path: &Path) -> Result<()>
```

Stream an absolute media URL to `path` without buffering the whole body.

##### Errors

Returns an error when the operation fails.

#### GraphicAudioClient::remove_activation

```rust
pub async fn remove_activation(&self, client_id: &str) -> Result<()>
```

`POST activation/remove` — drop a device slot for `client_id`.

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`, `Default`

### Product

```rust
pub struct Product {
    pub id: String,
    pub product_type: Option<String>,
    pub product_name: Option<String>,
    pub title: Option<String>,
    pub author: Option<String>,
    pub series: Option<String>,
    pub episode: Option<String>,
    pub genre: Option<String>,
    pub image: Option<String>,
    pub thumbnail: Option<String>,
    pub running_time: Option<String>,
    pub purchased_date: Option<String>,
    pub release_date: Option<String>,
}
```

One library / sample product from `api/products`.

**Fields**

- `id`: Identifier.
- `product_type`: Product type.
- `product_name`: Product name.
- `title`: Title.
- `author`: Author.
- `series`: Series.
- `episode`: Episode.
- `genre`: Genre.
- `image`: Image.
- `thumbnail`: Thumbnail.
- `running_time`: Running time.
- `purchased_date`: Purchased date.
- `release_date`: Release date.

#### Product::is_sample

```rust
pub fn is_sample(&self) -> bool
```

Whether this entry is a promotional sample (not owned).

#### Product::display_title

```rust
pub fn display_title(&self) -> String
```

Display title preferring `Series` + `ProductName` for owned volumes.

Access App owned rows often use a short `ProductName` (e.g. `"Volume 1"`)
with the work title in `Series`. Magento downloadable rows use the full
storefront title — combining these fields keeps ZIP matching reliable.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### TitleFetchRequest

```rust
pub struct TitleFetchRequest<'a> {
    pub store_base_url: &'a str,
    pub email: &'a str,
    pub product_id: &'a str,
    pub product_title: Option<&'a str>,
    pub cache_dir: &'a Path,
    pub prefer_hi: bool,
    pub mode: GraphicAudioAccess,
    pub password: Option<&'a str>,
    pub zip_container: GraphicAudioContainer,
}
```

Parameters for [`fetch_title_with_mode`](#fetch_title_with_mode).

**Fields**

- `store_base_url`: Store base URL.
- `email`: Email.
- `product_id`: Product Identifier.
- `product_title`: Product title.
- `cache_dir`: Absolute path to the download/work cache directory.
- `prefer_hi`: Prefer hi.
- `mode`: Mode.
- `password`: Password.
- `zip_container`: ZIP SKU preference when [`Self::mode`](#titlefetchrequest) is [`GraphicAudioAccess::Zip`](#graphicaudioaccess).

Implements: `Clone`, `Debug`

### DownloadableProduct

```rust
pub struct DownloadableProduct {
    pub title: String,
    pub option_label: String,
    pub download_url: String,
    pub remaining: Option<u32>,
    pub status: String,
}
```

One row from My Downloadable Products.

**Fields**

- `title`: Title.
- `option_label`: Option label.
- `download_url`: Download URL.
- `remaining`: Remaining.
- `status`: Status.

#### DownloadableProduct::format_rank

```rust
pub fn format_rank(&self) -> u8
```

Rank for Magento ZIP SKU selection (lower = better).

#### DownloadableProduct::format_rank_for

```rust
pub fn format_rank_for(&self, preference: GraphicAudioContainer) -> u8
```

Rank using a configured container preference.

#### DownloadableProduct::has_remaining

```rust
pub fn has_remaining(&self) -> bool
```

Has remaining.

Implements: `Clone`, `Debug`

### LibraryItem

```rust
pub struct LibraryItem {
    pub product_id: String,
    pub title: Option<String>,
    pub listen_path: Option<String>,
}
```

One owned title from Browser Player `content_library` HTML.

**Fields**

- `product_id`: Product Identifier.
- `title`: Title.
- `listen_path`: Listen path.

Implements: `Clone`, `Debug`

### MagentoClient

```rust
pub struct MagentoClient {
    /* private fields */
}
```

Magento customer session + cookie jar (including Browser Player CloudFront cookies).

#### MagentoClient::new

```rust
pub fn new(base_url: impl Into<String>) -> Result<Self>
```

Build a client for `base_url` (trailing slash stripped).

##### Errors

Returns an error when the operation fails.

#### MagentoClient::base_url

```rust
pub fn base_url(&self) -> &str
```

Base URL.

#### MagentoClient::login

```rust
pub async fn login(&self, email: &str, password: &str) -> Result<()>
```

Customer login via Magento `loginPost` (form_key + email/password).

##### Errors

Returns an error when the operation fails.

#### MagentoClient::list_downloadable

```rust
pub async fn list_downloadable(&self) -> Result<Vec<DownloadableProduct>>
```

Parse My Downloadable Products into link rows.

##### Errors

Returns an error when the operation fails.

#### MagentoClient::resolve_download_url

```rust
pub async fn resolve_download_url(&self, download_url: &str) -> Result<String>
```

Resolve a Magento downloadable link to a CDN URL (or stream body if no redirect).

Hitting the Magento `/downloadable/download/link/...` URL consumes one of the
limited download attempts — callers must finish the transfer.

##### Errors

Returns an error when the operation fails.

#### MagentoClient::download_to_path

```rust
pub async fn download_to_path(&self, url: &str, path: &Path) -> Result<()>
```

Stream `url` to `path` (uses Magento/CloudFront cookies when present).

##### Errors

Returns an error when the operation fails.

#### MagentoClient::content_library_html

```rust
pub async fn content_library_html(&self) -> Result<String>
```

Fetch Browser Player library HTML (`library/index/content_library`).

##### Errors

Returns an error when the operation fails.

#### MagentoClient::list_library

```rust
pub async fn list_library(&self) -> Result<Vec<LibraryItem>>
```

List owned titles from Browser Player library (no Access App device token).

##### Errors

Returns an error when the operation fails.

#### MagentoClient::player_listen_url

```rust
pub async fn player_listen_url(&self, product_id: &str) -> Result<String>
```

Find Browser Player listen URL for `product_id` via `library/index/content_library`.

##### Errors

Returns an error when the operation fails.

#### MagentoClient::player_audio_url

```rust
pub async fn player_audio_url(&self, product_id: &str) -> Result<String>
```

Open the Browser Player page and return the `<audio>` media URL (sets CF cookies).

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`

### GraphicAudioSource

```rust
pub struct GraphicAudioSource {
    /* private fields */
}
```

GraphicAudio catalog client used by the guest's discovery entrypoints.

The catalog lives on the public Magento storefront, so no account or
access path is involved.

#### GraphicAudioSource::new

```rust
pub fn new() -> Self
```

Production GraphicAudio storefront origin.

#### GraphicAudioSource::with_store_url

```rust
pub fn with_store_url(self, store_url: impl Into<String>) -> Self
```

Override Magento storefront base (wiremock / staging).

#### GraphicAudioSource::search_catalog

```rust
pub async fn search_catalog(&self, opts: &CatalogSearchOpts) -> Result<Vec<CatalogHit>>
```

Searches the public catalog.

##### Errors

Returns an error when a storefront request or its parse fails and the
method cannot fall back. Some lookups fail soft and only yield fewer
results (or none), so an empty result does not prove the store has none.

#### GraphicAudioSource::expand_candidates

```rust
pub async fn expand_candidates(
    &self,
    seed: &ExpandSeed,
    limit: usize,
) -> Result<Vec<CatalogHit>>
```

Finds up to `limit` titles related to `seed` (same author, series, …).

##### Errors

Returns an error when a storefront request or its parse fails and the
method cannot fall back. Some lookups fail soft and only yield fewer
results (or none), so an empty result does not prove the store has none.

#### GraphicAudioSource::purchase_hint

```rust
pub async fn purchase_hint(
    &self,
    opts: &PurchaseHintOpts,
) -> Result<Option<SourcePurchaseHint>>
```

Resolves the storefront price and link for a title; `None` when not sold.

##### Errors

Returns an error when a storefront request or its parse fails and the
method cannot fall back. Failed lookups are soft, so `Ok(None)` can also
mean the store could not be reached, not only that the title is not sold.

Implements: `Clone`, `Debug`, `Default`

## Enums

### GraphicAudioError

```rust
pub enum GraphicAudioError {
    Auth(String),
    NoAccounts(String),
    AccountNotFound(String),
    Api(String),
    Download(String),
    Io(Error),
    Other(Error),
}
```

Failures from Magento / Access App auth, library listing, or download.

**Variants**

- `Auth`: Magento customer login or Access App device activation failed.
- `NoAccounts`: No GraphicAudio accounts are configured (safe to skip in multi-source scan).
- `AccountNotFound`: Requested account id is not present in `encrypted_secrets`.
- `Api`: Upstream Magento, Access App, or CloudFront API error.
- `Download`: Browser-player / ZIP / device download or packaging failed.
- `Io`: Local filesystem I/O while reading or writing cache files.
- `Other`: Unexpected failure wrapped from `anyhow`.

#### GraphicAudioError::auth

```rust
pub fn auth(msg: impl Into<String>) -> Self
```

Builds an [`Self::Auth`](#graphicaudioerror) error from a displayable message.

#### GraphicAudioError::no_accounts

```rust
pub fn no_accounts(msg: impl Into<String>) -> Self
```

Builds a [`Self::NoAccounts`](#graphicaudioerror) error (multi-source scan may skip this store).

#### GraphicAudioError::api

```rust
pub fn api(msg: impl Into<String>) -> Self
```

Builds an [`Self::Api`](#graphicaudioerror) error from a displayable message.

#### GraphicAudioError::download

```rust
pub fn download(msg: impl Into<String>) -> Self
```

Builds a [`Self::Download`](#graphicaudioerror) error from a displayable message.

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<GraphicAudioError>`

### GraphicAudioAccess

```rust
pub enum GraphicAudioAccess {
    Web,
    Zip,
    Device,
}
```

How GraphicAudio fetches owned audio (the `access` plugin setting).

Default is [`Self::Web`](#graphicaudioaccess) (Browser Player). ZIP downloads and Access App
device registration are opt-in — do not assume the user purchased a ZIP
SKU or wants a device slot consumed.

**Variants**

- `Web`: Magento Browser Player (`/library`) — no ZIP attempts, no device slot.
- `Zip`: Magento ZIP download (`My Downloadable Products`) — ≤3 attempts; opt-in.
- `Device`: Access App API (`/access`) — registers/uses a device activation; opt-in.

#### GraphicAudioAccess::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parse a config / env / CLI token (`web`, `zip`, `device`, plus aliases).

#### GraphicAudioAccess::from_env

```rust
pub fn from_env() -> Option<Self>
```

Env override `BOOKCLERK_GA_ACCESS` or legacy `BOOKCLERK_GA_FETCH`.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### GraphicAudioBitrate

```rust
pub enum GraphicAudioBitrate {
    Hi,
    Lo,
}
```

GraphicAudio device encode preference (the `bitrate` plugin setting).

**Variants**

- `Hi`: Higher bitrate / Hi URL when available.
- `Lo`: Lower bitrate / Lo URL.

#### GraphicAudioBitrate::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parse.

#### GraphicAudioBitrate::prefers_hi

```rust
pub fn prefers_hi(self) -> bool
```

Prefers hi.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### GraphicAudioContainer

```rust
pub enum GraphicAudioContainer {
    Auto,
    M4b,
    Mp3,
    Flac,
}
```

Preferred GraphicAudio ZIP SKU when [`GraphicAudioAccess::Zip`](#graphicaudioaccess).

**Variants**

- `Auto`: Prefer M4B, then MP3, then FLAC (default).
- `M4b`: M4b variant.
- `Mp3`: MP3 variant.
- `Flac`: Flac variant.

#### GraphicAudioContainer::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parse.

#### GraphicAudioContainer::format_rank

```rust
pub fn format_rank(self, option_label: &str) -> u8
```

Rank used when sorting Magento downloadable options (lower = better).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### catalog_http_client

```rust
pub fn catalog_http_client() -> Result<Client>
```

Shared HTTP client for public Magento catalog pages.

#### Errors

Returns an error when the operation fails.

### expand_from_product_id

```rust
pub async fn expand_from_product_id(
    http: &Client,
    store_base: Option<&str>,
    product_id: &str,
) -> Result<Vec<MagentoCatalogProduct>>
```

Expand a owned Magento product id into related + series siblings.

#### Errors

Returns an error when the operation fails.

### expand_from_search

```rust
pub async fn expand_from_search(
    http: &Client,
    store_base: Option<&str>,
    query: &str,
) -> Result<Vec<MagentoCatalogProduct>>
```

Expand via Magento catalog search (series name or title).

#### Errors

Returns an error when the operation fails.

### fetch_product_by_id

```rust
pub async fn fetch_product_by_id(
    http: &Client,
    store_base: &str,
    product_id: &str,
) -> Result<(String, Vec<MagentoCatalogProduct>, Option<String>)>
```

Magento product page by numeric product id.

#### Errors

Returns an error when the operation fails.

### fetch_series_page

```rust
pub async fn fetch_series_page(
    http: &Client,
    series_url: &str,
) -> Result<Vec<MagentoCatalogProduct>>
```

Fetch titles listed on a series category page (includes series-set SKUs).

#### Errors

Returns an error when the operation fails.

### parse_catalog_grid

```rust
pub fn parse_catalog_grid(html: &str) -> Vec<MagentoCatalogProduct>
```

Parse Magento product grid / related list items.

### parse_related_products

```rust
pub fn parse_related_products(html: &str) -> Vec<MagentoCatalogProduct>
```

Parse the Recommended / related products block on a product page.

### search_catalog

```rust
pub async fn search_catalog(
    http: &Client,
    store_base: &str,
    query: &str,
) -> Result<Vec<MagentoCatalogProduct>>
```

Search Magento catalog; often redirects to a matching series page.

`page` > 1 tries Magento `?p=` once; when that yields nothing the source is
treated as exhausted by the host cursor (empty page).

#### Errors

Returns an error when the operation fails.

### search_catalog_page

```rust
pub async fn search_catalog_page(
    http: &Client,
    store_base: &str,
    query: &str,
    page: u32,
) -> Result<Vec<MagentoCatalogProduct>>
```

Like [`search_catalog`](#search_catalog) with an optional Magento page index.

#### Errors

Returns an error when the operation fails.

### fetch_title_materials

```rust
pub async fn fetch_title_materials(
    client: &GraphicAudioClient,
    product_id: &str,
    cache_dir: &Path,
) -> Result<PlainFetch>
```

Password env (same as CLI login) for Magento ZIP / Browser Player.
Fetch one product id into `cache_dir` via `api/links` Hi/Lo URLs.

#### Errors

Returns an error when the operation fails.

### fetch_title_with_mode

```rust
pub async fn fetch_title_with_mode(
    access: &GraphicAudioClient,
    req: TitleFetchRequest<'_>,
) -> Result<PlainFetch>
```

Fetch owned audio for one product using the configured access path only
(no ZIP→web→device cascade).

#### Errors

Returns an error when the operation fails.

### collect_account_books

```rust
pub async fn collect_account_books(
    auth: &GraphicAudioAuthFile,
    access_base: &str,
    store_base: &str,
    access: GraphicAudioAccess,
    magento_password: Option<&str>,
    include_samples: bool,
    account_id: &str,
    marketplace: &str,
) -> Result<(Vec<ScanBook>, u32)>
```

Collect owned titles for one account (no DB writes).

Prefer Access App catalog when `access=device` and a device token exists.
For web/zip (default), use Magento Browser Player library so we do not
depend on a device slot for ownership listing. Falls back to Access App
when a legacy device token exists and Magento password is unavailable.

#### Errors

Returns an error when the operation fails.

### product_to_scan_book

```rust
pub fn product_to_scan_book(
    product: &Product,
    account_id: &str,
    marketplace: &str,
) -> ScanBook
```

Map a GraphicAudio product JSON object to a structured scan row.

Every GraphicAudio production is a full-cast dramatisation of the work,
so the expression kind is `ExpressionKind::Dramatisation`.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, GraphicAudioError>;
```

Result alias for GraphicAudio auth, sync, and download operations.

## Constants

### DEFAULT_BASE_URL

```rust
pub const DEFAULT_BASE_URL: &str = "https://www.graphicaudio.net/access";
```

Default API origin (no trailing slash after `/access`).

### LOGIN_PATH

```rust
pub const LOGIN_PATH: &str = "/activation/login";
```

Device activation / password login.

### PRODUCTS_PATH

```rust
pub const PRODUCTS_PATH: &str = "/api/products";
```

Library + sample products listing.

### REMOVE_PATH

```rust
pub const REMOVE_PATH: &str = "/activation/remove";
```

Forget a device activation (`client_id` form field).

### GA_ACCESS_ENV

```rust
pub const GA_ACCESS_ENV: &str = "BOOKCLERK_GA_ACCESS";
```

Preferred env override (`BOOKCLERK_GA_ACCESS`); falls back to [`GA_FETCH_ENV`](#ga_fetch_env).

### GA_FETCH_ENV

```rust
pub const GA_FETCH_ENV: &str = "BOOKCLERK_GA_FETCH";
```

Env override for which GraphicAudio access path to use (legacy name).

### DEFAULT_STORE_URL

```rust
pub const DEFAULT_STORE_URL: &str = "https://www.graphicaudio.net";
```

Default Magento storefront origin (no `/access` suffix).

### GRAPHICAUDIO_SOURCE_ID

```rust
pub const GRAPHICAUDIO_SOURCE_ID: &str = "graphicaudio";
```

Describe / config id for this store (`graphicaudio`).
