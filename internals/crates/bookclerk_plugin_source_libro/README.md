# bookclerk_plugin_source_libro

Libro.fm content source: auth, library sync, and DRM-free download.

Uses the unofficial mobile API documented by community clients (see
`client` module). Audible ASIN enrichment (metadata confidence matching)
lives in `bookclerk-enrich` / CLI — this crate must not depend on
`bookclerk-audible` or `bookclerk-enrich`. Public explore catalog helpers
live in [`catalog`](catalog.md) (SDK HTTP, no enrich).

| Item | Kind | Summary |
| --- | --- | --- |
| [`LibroAuthFile`](#libroauthfile) | struct | Credential envelope for one Libro.fm account. |
| [`Audiobook`](#audiobook) | struct | Audiobook. |
| [`DownloadManifest`](#downloadmanifest) | struct | Download manifest. |
| [`DownloadPart`](#downloadpart) | struct | Download part. |
| [`LibraryPage`](#librarypage) | struct | Library page. |
| [`LibroClient`](#libroclient) | struct | Authenticated Libro.fm HTTP helper. |
| [`ManifestTrack`](#manifesttrack) | struct | Manifest track. |
| [`PackagedM4b`](#packagedm4b) | struct | Packaged m4b. |
| [`TokenResponse`](#tokenresponse) | struct | Token response. |
| [`LibroSource`](#librosource) | struct | Libro.fm catalog client used by the guest's discovery entrypoints. |
| [`ManifestFormat`](#manifestformat) | enum | Android `MediaFormat` id for `download-manifest?format=…`. |
| [`LibroContainer`](#librocontainer) | enum | Preferred Libro.fm download packaging. |
| [`LibroError`](#libroerror) | enum | Failures from Libro.fm OAuth, library listing, or DRM-free download. |
| [`chapters_from_tracks`](#chapters_from_tracks) | function | Chapters from tracks. |
| [`fetch_title_materials`](#fetch_title_materials) | function | Fetch one ISBN into `cache_dir`, preferring M4B when available. |
| [`audiobook_to_scan_book`](#audiobook_to_scan_book) | function | Map a Libro.fm audiobook JSON object to a structured scan row. |
| [`collect_account_books`](#collect_account_books) | function | Fetch all library pages for one client and collect books (no DB writes). |
| [`Result`](#result) | type alias | Result alias for Libro.fm auth, sync, and download operations. |
| [`APP_VER`](#app_ver) | constant | Android app version header (`X-LibroFm-AppVer`). |
| [`CLIENT_ID`](#client_id) | constant | Optional OAuth `client_id`. |
| [`DEFAULT_BASE_URL`](#default_base_url) | constant | Default API origin. |
| [`DOWNLOAD_MANIFEST_PATH`](#download_manifest_path) | constant | Download manifest (`parts` + chapter `tracks`). |
| [`LIBRARY_PATH`](#library_path) | constant | Paginated library listing (`/api/vN/library` from the Android app prefix). |
| [`OAUTH_TOKEN_PATH`](#oauth_token_path) | constant | OAuth password-grant path. |
| [`PACKAGED_M4B_PATH`](#packaged_m4b_path) | constant | Packaged single-file M4B when Libro.fm offers it. |
| [`USER_AGENT_VALUE`](#user_agent_value) | constant | User-Agent matching the official Android HTTP stack. |
| [`LIBRO_SOURCE_ID`](#libro_source_id) | constant | Describe / config id for this store (`libro`). |

## Re-exports

- [`catalog::catalog_detail`](catalog.md#catalog_detail)
- [`catalog::expand_candidates` as `catalog_expand_candidates`](catalog.md#expand_candidates)
- [`catalog::explore_search_path`](catalog.md#explore_search_path)
- [`catalog::parse_details` as `parse_catalog_details`](catalog.md#parse_details)
- [`catalog::parse_search_page` as `parse_catalog_search_page`](catalog.md#parse_search_page)
- [`catalog::purchase_hint` as `catalog_purchase_hint`](catalog.md#purchase_hint)
- [`catalog::search_catalog` as `catalog_search`](catalog.md#search_catalog)
- [`catalog::LibroDetails`](catalog.md#librodetails)
- [`catalog::CATALOG_USER_AGENT`](catalog.md#catalog_user_agent)
- [`guest::guest_fetch_title`](guest.md#guest_fetch_title)
- [`guest::guest_fetch_title_rpc`](guest.md#guest_fetch_title_rpc)
- [`guest::guest_login`](guest.md#guest_login)
- [`guest::guest_login_rpc`](guest.md#guest_login_rpc)
- [`guest::guest_scan`](guest.md#guest_scan)
- [`guest::guest_scan_rpc`](guest.md#guest_scan_rpc)
- [`guest::resolve_base_url`](guest.md#resolve_base_url)
- [`guest::resolve_container`](guest.md#resolve_container)

## Modules

- [`catalog`](catalog.md): Libro.fm catalog search, detail, related titles and purchase links.
- [`guest`](guest.md): Guest-process helpers for the external Libro.fm source plugin.

## Structs

### LibroAuthFile

```rust
pub struct LibroAuthFile {
    pub access_token: String,
    pub token_type: String,
    pub expires_at: Option<DateTime<Utc>>,
    pub email: String,
    pub user_id: Option<String>,
    pub marketplace: String,
    pub label: Option<String>,
}
```

Credential envelope for one Libro.fm account.

The guest returns it from login; the host seals it into `encrypted_secrets`
and passes it back on scan and fetch. Never written
to disk under `Accounts/`.

**Fields**

- `access_token`: Bearer token from Libro.fm OAuth (`access_token`).
- `token_type`: OAuth token type (almost always `Bearer`).
- `expires_at`: Absolute expiry when known (RFC3339 when serialized).
- `email`: Sign-in email used as fallback account id.
- `user_id`: Libro.fm user id when the token response includes one.
- `marketplace`: Marketplace / locale hint (not part of the mobile API; stored for Bookclerk).
- `label`: Optional operator-facing account label in Accounts UI.

#### LibroAuthFile::account_id

```rust
pub fn account_id(&self) -> &str
```

Stable account id for library rows: prefer user id, else email.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### Audiobook

```rust
pub struct Audiobook {
    pub isbn: String,
    pub title: String,
    pub authors: Option<Vec<String>>,
    pub cover_url: Option<String>,
    pub subtitle: Option<String>,
    pub publisher: Option<String>,
    pub publication_date: Option<String>,
    pub abridged: Option<bool>,
    pub series: Option<String>,
    pub series_num: Option<Value>,
    pub genres: Option<Vec<Genre>>,
    pub audiobook_info: Option<AudiobookInfo>,
    pub user_metadata: Option<UserMetadata>,
    pub id: Option<Value>,
}
```

Audiobook.

**Fields**

- `isbn`: ISBN may arrive as string or number in JSON.
- `title`: Title.
- `authors`: Authors.
- `cover_url`: Cover URL.
- `subtitle`: Subtitle.
- `publisher`: Publisher.
- `publication_date`: Publication date.
- `abridged`: Abridged.
- `series`: Series.
- `series_num`: Series num.
- `genres`: Genres.
- `audiobook_info`: Audiobook info.
- `user_metadata`: User metadata.
- `id`: Identifier.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### DownloadManifest

```rust
pub struct DownloadManifest {
    pub isbn: Option<String>,
    pub parts: Vec<DownloadPart>,
    pub tracks: Vec<ManifestTrack>,
    pub expires_at: Option<String>,
    pub size_bytes: Option<u64>,
}
```

Download manifest.

**Fields**

- `isbn`: ISBN identifier.
- `parts`: Parts.
- `tracks`: Tracks.
- `expires_at`: Expires at.
- `size_bytes`: Size bytes.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### DownloadPart

```rust
pub struct DownloadPart {
    pub url: String,
    pub size_bytes: Option<u64>,
}
```

Download part.

**Fields**

- `url`: URL.
- `size_bytes`: Size bytes.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### LibraryPage

```rust
pub struct LibraryPage {
    pub page: u32,
    pub total_pages: u32,
    pub audiobooks: Vec<Audiobook>,
    pub tags: Vec<String>,
    pub error: Option<String>,
}
```

Library page.

**Fields**

- `page`: Page.
- `total_pages`: Total pages.
- `audiobooks`: Audiobooks.
- `tags`: Tags.
- `error`: Error.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### LibroClient

```rust
pub struct LibroClient {
    /* private fields */
}
```

Authenticated Libro.fm HTTP helper.

#### LibroClient::new

```rust
pub fn new(base_url: impl Into<String>) -> Self
```

Build a client pointed at `base_url` (no trailing slash).

#### LibroClient::with_http

```rust
pub fn with_http(self, http: HttpClient) -> Self
```

Override the HTTP client (tests / custom timeouts).

#### LibroClient::with_token

```rust
pub fn with_token(self, token: impl Into<String>) -> Self
```

Attach a bearer token for subsequent requests.

#### LibroClient::access_token

```rust
pub fn access_token(&self) -> Option<&str>
```

Current bearer token, if any.

#### LibroClient::base_url

```rust
pub fn base_url(&self) -> &str
```

Base URL in use.

#### LibroClient::login

```rust
pub async fn login(&mut self, email: &str, password: &str) -> Result<TokenResponse>
```

Password-grant login. Returns token metadata and stores the access token.

##### Errors

Returns an error when the operation fails.

#### LibroClient::library_page

```rust
pub async fn library_page(&self, page: u32) -> Result<LibraryPage>
```

Fetch one library page (`page` is 1-based).

##### Errors

Returns an error when the operation fails.

#### LibroClient::download_manifest

```rust
pub async fn download_manifest(
    &self,
    isbn: &str,
    format: ManifestFormat,
) -> Result<DownloadManifest>
```

Download-manifest for parts + chapter tracks.

Pass [`ManifestFormat::M4b`](#manifestformat) to request a single `.m4b` part (same asset as
[`Self::packaged_m4b`](#libroclientpackaged_m4b)) plus tracks in one response. [`ManifestFormat::Zip`](#manifestformat)
(or omitting `format`) returns multi-part `.zip` URLs.

##### Errors

Returns an error when the operation fails.

#### LibroClient::packaged_m4b

```rust
pub async fn packaged_m4b(&self, isbn: &str) -> Result<Option<PackagedM4b>>
```

Packaged M4B metadata when available (`None` on 404 / missing URL).

##### Errors

Returns an error when the operation fails.

#### LibroClient::download_bytes

```rust
pub async fn download_bytes(&self, url: &str) -> Result<Bytes>
```

Download arbitrary URL bytes (CDN part or M4B). Sends auth headers for libro.fm hosts.

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`, `Default`

### ManifestTrack

```rust
pub struct ManifestTrack {
    pub number: Option<u32>,
    pub length_msec: Option<u64>,
    pub length_sec: Option<u64>,
    pub chapter_title: Option<String>,
}
```

Manifest track.

**Fields**

- `number`: Number.
- `length_msec`: APK `ApiTrack` uses `length_msec` (Gson `@SerializedName`).
- `length_sec`: Legacy / fixture-only key; prefer `length_msec` when both are present.
- `chapter_title`: Chapter title.

#### ManifestTrack::duration_ms

```rust
pub fn duration_ms(&self) -> u64
```

Duration in milliseconds (APK wire format), falling back to `length_sec`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### PackagedM4b

```rust
pub struct PackagedM4b {
    pub m4b_url: String,
}
```

Packaged m4b.

**Fields**

- `m4b_url`: M4b URL.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### TokenResponse

```rust
pub struct TokenResponse {
    pub access_token: String,
    pub token_type: String,
    pub created_at: Option<i64>,
    pub expires_in: Option<i64>,
    pub error: Option<String>,
}
```

Token response.

**Fields**

- `access_token`: Access token.
- `token_type`: Token type.
- `created_at`: Unix seconds when the token was issued (mobile API).
- `expires_in`: Expires in.
- `error`: Error.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### LibroSource

```rust
pub struct LibroSource {
    /* private fields */
}
```

Libro.fm catalog client used by the guest's discovery entrypoints.

Construct with [`LibroSource::new`](#librosourcenew); use [`LibroSource::with_base_url`](#librosourcewith_base_url)
for wiremock / staging.

#### LibroSource::new

```rust
pub fn new() -> Self
```

Production Libro.fm API origin.

#### LibroSource::with_base_url

```rust
pub fn with_base_url(base_url: impl Into<String>) -> Self
```

Override API base (wiremock / staging).

#### LibroSource::search_catalog

```rust
pub async fn search_catalog(&self, opts: &CatalogSearchOpts) -> Result<Vec<CatalogHit>>
```

Searches the public catalog.

##### Errors

Returns an error when the storefront request or its parse fails.

#### LibroSource::catalog_detail

```rust
pub async fn catalog_detail(&self, product_id: &str) -> Result<Option<CatalogHit>>
```

Loads one catalog title by product id (ISBN); `None` when it is unknown.

##### Errors

Returns an error when the storefront request or its parse fails.

#### LibroSource::expand_candidates

```rust
pub async fn expand_candidates(
    &self,
    seed: &ExpandSeed,
    limit: usize,
) -> Result<Vec<CatalogHit>>
```

Finds up to `limit` titles related to `seed` (same author, series, …).

##### Errors

Returns an error when the storefront request or its parse fails.

#### LibroSource::purchase_hint

```rust
pub async fn purchase_hint(
    &self,
    opts: &PurchaseHintOpts,
) -> Result<Option<SourcePurchaseHint>>
```

Resolves the storefront price and link for a title; `None` when not sold.

##### Errors

Currently never: a failed lookup is treated as no match, so `Ok(None)`
can also mean the store could not be reached.

Implements: `Clone`, `Debug`, `Default`

## Enums

### ManifestFormat

```rust
pub enum ManifestFormat {
    M4b,
    Zip,
}
```

Android `MediaFormat` id for `download-manifest?format=…`.

- [`ManifestFormat::M4b`](#manifestformat) → `format=m4b` (must be lowercase; `M4B` is ignored)
- [`ManifestFormat::Zip`](#manifestformat) → omit `format` (APK `MediaFormat.ZIP` id is null)

**Variants**

- `M4b`: Single-file M4B CDN URL in `parts` when the title supports it.
- `Zip`: Multi-part ZIP of MP3s (API default when `format` is omitted).

#### ManifestFormat::query_value

```rust
pub const fn query_value(self) -> Option<&'static str>
```

Wire value for the `format` query, if any.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### LibroContainer

```rust
pub enum LibroContainer {
    M4b,
    Zip,
}
```

Preferred Libro.fm download packaging.

**Variants**

- `M4b`: Single M4B when the store offers it (default).
- `Zip`: Multi-part ZIP of MP3s.

#### LibroContainer::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parse.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LibroError

```rust
pub enum LibroError {
    Auth(String),
    NoAccounts(String),
    AccountNotFound(String),
    Api(String),
    Download(String),
    Io(Error),
    Other(Error),
}
```

Failures from Libro.fm OAuth, library listing, or DRM-free download.

**Variants**

- `Auth`: Password/token login or refresh failed.
- `NoAccounts`: No Libro.fm accounts are configured (safe to skip in multi-source scan).
- `AccountNotFound`: Requested account id is not present in `encrypted_secrets`.
- `Api`: Upstream mobile API or JSON payload error.
- `Download`: Title download or packaging into the preferred container failed.
- `Io`: Local filesystem I/O while reading or writing cache files.
- `Other`: Unexpected failure wrapped from `anyhow`.

#### LibroError::auth

```rust
pub fn auth(msg: impl Into<String>) -> Self
```

Builds an [`Self::Auth`](#libroerror) error from a displayable message.

#### LibroError::no_accounts

```rust
pub fn no_accounts(msg: impl Into<String>) -> Self
```

Builds a [`Self::NoAccounts`](#libroerror) error (multi-source scan may skip this store).

#### LibroError::api

```rust
pub fn api(msg: impl Into<String>) -> Self
```

Builds an [`Self::Api`](#libroerror) error from a displayable message.

#### LibroError::download

```rust
pub fn download(msg: impl Into<String>) -> Self
```

Builds a [`Self::Download`](#libroerror) error from a displayable message.

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<LibroError>`

## Functions

### chapters_from_tracks

```rust
pub fn chapters_from_tracks(tracks: &[ManifestTrack]) -> Vec<(String, u64)>
```

Chapters from tracks.

### fetch_title_materials

```rust
pub async fn fetch_title_materials(
    client: &LibroClient,
    isbn: &str,
    cache_dir: &Path,
) -> Result<PlainFetch>
```

Fetch one ISBN into `cache_dir`, preferring M4B when available.

#### Errors

Returns an error when the operation fails.

### audiobook_to_scan_book

```rust
pub fn audiobook_to_scan_book(
    book: &Audiobook,
    account_id: &str,
    marketplace: &str,
) -> ScanBook
```

Map a Libro.fm audiobook JSON object to a structured scan row.

### collect_account_books

```rust
pub async fn collect_account_books(
    client: &LibroClient,
    account_id: &str,
    marketplace: &str,
) -> Result<(Vec<ScanBook>, u32)>
```

Fetch all library pages for one client and collect books (no DB writes).

#### Errors

Returns an error when the operation fails.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, LibroError>;
```

Result alias for Libro.fm auth, sync, and download operations.

## Constants

### APP_VER

```rust
pub const APP_VER: &str = "7.37.4";
```

Android app version header (`X-LibroFm-AppVer`).

Keep in sync via `scripts/librofm-apk-probe/` / workflow `librofm-apk-probe.yml`.

### CLIENT_ID

```rust
pub const CLIENT_ID: &str = "";
```

Optional OAuth `client_id`.

TODO: Historical / reverse-engineered clients sometimes sent a mobile
`client_id` with form-encoded password grants. Current community tools
(libro-client, librofm-downloader, bfordham/librofm) omit it and POST JSON.
Set a non-empty value here if Libro.fm starts requiring it again.

### DEFAULT_BASE_URL

```rust
pub const DEFAULT_BASE_URL: &str = "https://libro.fm";
```

Default API origin.

TODO: Confirm if regional mirrors exist; community tools all use this host.

### DOWNLOAD_MANIFEST_PATH

```rust
pub const DOWNLOAD_MANIFEST_PATH: &str = "/api/v12/download-manifest";
```

Download manifest (`parts` + chapter `tracks`).

Android `DownloadApi.getDownloadManifest` sends `isbn`, `client_version`, and
optional `format` (`MediaFormat.M4B` → `"m4b"`; `ZIP` → omit / null).

### LIBRARY_PATH

```rust
pub const LIBRARY_PATH: &str = "/api/v12/library";
```

Paginated library listing (`/api/vN/library` from the Android app prefix).

### OAUTH_TOKEN_PATH

```rust
pub const OAUTH_TOKEN_PATH: &str = "/oauth/token";
```

OAuth password-grant path.

### PACKAGED_M4B_PATH

```rust
pub const PACKAGED_M4B_PATH: &str = "/api/v12/audiobooks/{isbn}/packaged_m4b";
```

Packaged single-file M4B when Libro.fm offers it.

### USER_AGENT_VALUE

```rust
pub const USER_AGENT_VALUE: &str = "okhttp/4.12.0";
```

User-Agent matching the official Android HTTP stack.

### LIBRO_SOURCE_ID

```rust
pub const LIBRO_SOURCE_ID: &str = "libro";
```

Describe / config id for this store (`libro`).
