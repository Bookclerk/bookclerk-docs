# bookclerk_plugin_source_audible

Audible storefront guest: OAuth login, library scan, catalog lookups and
download + decrypt, over [audible-rs](https://github.com/mkb79/audible-rs).

The binary serves these to the host as a jailed plugin guest. The host
seals the credentials and passes them in on each call; nothing here opens
the library database.

| Item | Kind | Summary |
| --- | --- | --- |
| [`LicenseSummary`](#licensesummary) | struct | Public summary of a license grant (no secrets). |
| [`AudibleScanCursor`](#audiblescancursor) | struct | Where an Audible scan pass stands between pages: the library listing's continuation token and the delta state token the pass's first page returned (Audible sends it only there, so the pass carries it to the last page, which hands it to the host). |
| [`DownloadOptions`](#downloadoptions) | struct | What one `fetchTitle` call asks of the download path. |
| [`LibraryPageRequest`](#librarypagerequest) | struct | Library listing filters and paging for one `/1.0/library` request. |
| [`LibraryPageRows`](#librarypagerows) | struct | One `/1.0/library` page mapped to scan rows. |
| [`AudibleError`](#audibleerror) | enum | Failures from Audible login, library sync, licensing, or download/decrypt. |
| [`Bitrate`](#bitrate) | enum | Audible license bitrate tier, the instance's `bitrate` setting. |
| [`fetch_chapter_info`](#fetch_chapter_info) | function | Fetch curated chapter metadata (`chapter_info` object). |
| [`request_content_license`](#request_content_license) | function | Request an Adrm/Mpeg download license for `asin`. |
| [`summarize_license`](#summarize_license) | function | Summarize a license without exposing voucher / URL query secrets. |
| [`deregister_device`](#deregister_device) | function | Deregister the device behind `client`, leaving the account's other devices registered. |
| [`guest_fetch_title`](#guest_fetch_title) | function | Download + decrypt one title; return plain paths (never encrypted on the wire). |
| [`guest_login_complete`](#guest_login_complete) | function | Await a pending OAuth session and return account + opaque credential bytes. |
| [`guest_login_start`](#guest_login_start) | function | Start LoginServer OAuth; return `(session_id, browser_url)`. |
| [`guest_revoke_account`](#guest_revoke_account) | function | Deregister the Audible device that login registered for these credentials. |
| [`guest_scan_page`](#guest_scan_page) | function | RPC scan: one page of one account's pass (the host upserts books). |
| [`collect_account_books`](#collect_account_books) | function | Fetch every library page for one account without touching the library DB (a full pass). |
| [`fetch_library_page`](#fetch_library_page) | function | Fetch one library page for one account without touching the library DB. |
| [`Result`](#result) | type alias | Result alias for Audible auth, sync, license, and download operations. |
| [`ID`](#id) | constant | Plugin id of this storefront (`audible`), stamped on accounts it returns. |

## Modules

- [`catalog`](catalog.md): Public Audible catalog helpers for Discover (`search_catalog` / expand / purchase).

## Structs

### LicenseSummary

```rust
pub struct LicenseSummary {
    pub asin: String,
    pub status_code: String,
    pub drm_type: Option<String>,
    pub content_format: Option<String>,
    pub content_size: Option<u64>,
    pub granted: bool,
    pub denial_message: Option<String>,
    pub has_voucher: bool,
    pub offline_url_present: bool,
}
```

Public summary of a license grant (no secrets).

**Fields**

- `asin`: Amazon ASIN identifier.
- `status_code`: Status code.
- `drm_type`: DRM type.
- `content_format`: Content format.
- `content_size`: Content size.
- `granted`: Granted.
- `denial_message`: Denial message.
- `has_voucher`: Has voucher.
- `offline_url_present`: Offline URL present.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AudibleScanCursor

```rust
pub struct AudibleScanCursor {
    pub continuation: String,
    pub state_token: Option<String>,
}
```

Where an Audible scan pass stands between pages: the library listing's
continuation token and the delta state token the pass's first page
returned (Audible sends it only there, so the pass carries it to the last
page, which hands it to the host).

It is the plugin's own store cursor: the SDK's [`scan_page`](../../../plugins/sdk/rust/README.md#scan_page) helper wraps
it in the scan cursor the host sees.

**Fields**

- `continuation`: Continuation token of the next library page.
- `state_token`: State token the pass has seen so far, if any.

#### AudibleScanCursor::parse

```rust
pub fn parse(cursor: &str) -> Result<Option<Self>, PluginError>
```

Reads a store cursor this plugin wrote; the empty cursor starts a pass.

##### Errors

Returns an `invalid_cursor` [`PluginError`](../bookclerk_plugin_abi/error.md#pluginerror) for text this plugin did
not write.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DownloadOptions

```rust
pub struct DownloadOptions {
    pub quality: Bitrate,
    pub widevine: bool,
    pub xhe_aac: bool,
    pub widevine_cdm: Option<PathBuf>,
    pub widevine_cdm_provider: Option<String>,
    pub cover_size: String,
    pub chapter_layout: String,
    pub strip_audible_brand_audio: bool,
    pub download_speed_limit_kbps: u32,
}
```

What one `fetchTitle` call asks of the download path.

Packaging (format, naming, tagging) is the host's job after the guest
returns plain audio, so only the knobs that change what is downloaded
live here.

**Fields**

- `quality`: License bitrate tier.
- `widevine`: Prefer the Widevine/CENC download when Audible offers it.
- `xhe_aac`: Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm`: Bring-your-own Widevine `.wvd` the host granted to the guest.
- `widevine_cdm_provider`: Remote L3 CDM provider URL (`None` = classic Libation AudibleCdm;
  empty or `off` = disabled).
- `cover_size`: Cover image size to request (`500`, `1215` or `native`).
- `chapter_layout`: Chapter API layout to request (`tree` or `flat`).
- `strip_audible_brand_audio`: Trim Audible's brand intro and outro from the remux window.
- `download_speed_limit_kbps`: Download speed cap in KB/s (`0` = unlimited).

#### DownloadOptions::from_host

```rust
pub fn from_host(fetch: &FetchOptions, source_config: &Value) -> Self
```

Builds the options for one fetch from the host's wire knobs.

Empty `cover_size` and `chapter_layout` keep the defaults (`500` and
`tree`).

##### Arguments

* `fetch` - `FetchTitleParams.fetch` as the host sent it.
* `source_config` - The instance's settings document (for `bitrate`).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### LibraryPageRequest

```rust
pub struct LibraryPageRequest {
    pub continuation: Option<String>,
    pub state_token: Option<String>,
    pub page_size: u32,
    pub import_episodes: bool,
    pub import_plus_titles: bool,
}
```

Library listing filters and paging for one `/1.0/library` request.

**Fields**

- `continuation`: Continuation token of the page to fetch; `None` for the first page.
- `state_token`: Delta state token from the account's last pass; `None` for a full pass.
- `page_size`: Items per page (`num_results`).
- `import_episodes`: When true, keep podcast episodes.
- `import_plus_titles`: When true, keep Plus catalog titles that are not owned outright.

Implements: `Clone`, `Debug`, `Default`

### LibraryPageRows

```rust
pub struct LibraryPageRows {
    pub books: Vec<ScanBook>,
    pub removed: Vec<String>,
    pub next_continuation: Option<String>,
    pub state_token: Option<String>,
}
```

One `/1.0/library` page mapped to scan rows.

**Fields**

- `books`: Titles to upsert.
- `removed`: ASINs the delta reports as gone from the library (empty on a full pass).
- `next_continuation`: Continuation token of the next page; `None` on the last page.
- `state_token`: State token the response carried (Audible sends it on the first page).

Implements: `Clone`, `Debug`, `Default`

## Enums

### AudibleError

```rust
pub enum AudibleError {
    Auth(String),
    NoAccounts(String),
    Sync(String),
    License(String),
    NoAaxcAsset { asin: String, message: String },
    Widevine(String),
    Download(String),
    Io(Error),
    Other(Error),
}
```

Failures from Audible login, library sync, licensing, or download/decrypt.

**Variants**

- `Auth`: Login, token refresh, or authenticator load failed.
- `NoAccounts`: The host passed no credentials, or none matching the requested accounts.
- `Sync`: Library listing or metadata sync against Audible APIs failed.
- `License`: Content license request was rejected or returned an unexpected payload.
- `NoAaxcAsset`: Adrm licenserequest returned 000307 — title is Widevine-only (no aaxc).
  - `asin`: Title ASIN that has no AAXC asset.
  - `message`: Upstream error detail from Audible.
- `Widevine`: Widevine CDM load, provisioning, or CENC decrypt failed.
- `Download`: HTTP download of audio/PDF/cover bytes failed.
- `Io`: Local filesystem I/O while reading or writing cache files.
- `Other`: Unexpected failure wrapped from `anyhow`.

#### AudibleError::is_no_aaxc_asset

```rust
pub fn is_no_aaxc_asset(&self) -> bool
```

Returns true when this error is [`Self::NoAaxcAsset`](#audibleerror) (retry via Widevine).

Implements: `Debug`, `Display`, `Error`, `From<ApiError>`, `From<AuthError>`, `From<Error>`, `From<LoginError>`

### Bitrate

```rust
pub enum Bitrate {
    High,
    Normal,
}
```

Audible license bitrate tier, the instance's `bitrate` setting.

**Variants**

- `High`: Highest bitrate Audible offers for the title (the default).
- `Normal`: Audible's standard, lower bitrate.

#### Bitrate::from_source_config

```rust
pub fn from_source_config(source_config: &Value) -> Self
```

Reads the instance's `bitrate` setting (`high` / `normal`, any case).

##### Arguments

* `source_config` - The instance's settings document; may be `null`.

##### Returns

The configured tier, or [`Bitrate::High`](#bitrate) when the setting is missing
or holds an unknown value.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `From<Bitrate>`, `PartialEq`

## Functions

### fetch_chapter_info

```rust
pub async fn fetch_chapter_info(
    client: &Client,
    marketplace: &str,
    asin: &str,
    quality: Bitrate,
    chapter_layout: &str,
) -> Result<Value>
```

Fetch curated chapter metadata (`chapter_info` object).

#### Errors

Returns an error when the operation fails.

### request_content_license

```rust
pub async fn request_content_license(
    client: &Client,
    marketplace: &str,
    asin: &str,
    quality: Bitrate,
) -> Result<DownloadLicense>
```

Request an Adrm/Mpeg download license for `asin`.

#### Errors

Returns an error when the operation fails.

### summarize_license

```rust
pub fn summarize_license(license: &DownloadLicense) -> LicenseSummary
```

Summarize a license without exposing voucher / URL query secrets.

### deregister_device

```rust
pub async fn deregister_device(client: &Client) -> Result<Option<String>>
```

Deregister the device behind `client`, leaving the account's other
devices registered.

Exposed for wiremock CI tests (inject a [`Client`](https://docs.rs/audible_rs/latest/audible_rs/api/client/struct.Client.html) with
`auth_base_override`).

#### Returns

The device name Amazon assigned at registration, when known.

#### Errors

Returns an error when the token refresh fails or Amazon rejects the
deregistration.

### guest_fetch_title

```rust
pub async fn guest_fetch_title(
    credentials: &Value,
    title_id: &str,
    cache_dir: &Path,
    source_config: &Value,
    fetch: &FetchOptions,
) -> Result<PlainFetch>
```

Download + decrypt one title; return plain paths (never encrypted on the wire).

`fetch` carries the host fetch knobs from the ABI `FetchTitleParams.fetch`
field, and the instance's `bitrate` setting in `source_config` picks the
license tier. Packaging stays with the host.

#### Errors

Returns an error when the operation fails.

### guest_login_complete

```rust
pub async fn guest_login_complete(session_id: &str) -> Result<LoginResult>
```

Await a pending OAuth session and return account + opaque credential bytes.

#### Errors

Returns an error when the operation fails.

### guest_login_start

```rust
pub async fn guest_login_start(params: &LoginParams) -> Result<(String, String)>
```

Start LoginServer OAuth; return `(session_id, browser_url)`.

Completes later via [`guest_login_complete`](#guest_login_complete).

When the host sets `callback_ipc` + `callback_public_base`, the guest does
**not** bind TCP — it connects to the host IPC tunnel and serves LoginServer
on forwarded streams (required under Windows AppContainer).

#### Errors

Returns an error when the operation fails.

### guest_revoke_account

```rust
pub async fn guest_revoke_account(credentials: &Value) -> Result<Option<String>>
```

Deregister the Audible device that login registered for these credentials.

Only this device is removed (`deregister_all = false`); the account's other
devices stay. Amazon invalidates the tokens on success, so the host deletes
the credentials afterwards either way.

#### Arguments

* `credentials` - Credential JSON exactly as login returned it.

#### Returns

The device name Amazon assigned at registration, when the credentials
record one.

#### Errors

Returns an error when the credentials do not parse, the token refresh
fails, or Amazon rejects the deregistration.

### guest_scan_page

```rust
pub async fn guest_scan_page(params: &ScanParams) -> Result<ScanPage, PluginError>
```

RPC scan: one page of one account's pass (the host upserts books).

With a non-empty `syncToken` the pass is a delta: Audible lists items
added or revoked since that state token, revoked ASINs come back in
`removed`, and the pass ends with `complete = false`. Without one the pass
lists the whole active library and ends `complete`. Either way the last
page returns the state token Audible issued for the pass, so the next
pass can be a delta. The SDK's [`scan_page`](../../../plugins/sdk/rust/README.md#scan_page) helper pages Audible's
library pages into scan replies.

#### Arguments

* `params` - The pass's account, credential, options, cursor and sync
  token.

#### Errors

Returns `invalid_cursor` for a cursor this plugin did not write, and
`internal` when the credential is missing or invalid, the page request
fails, or the next cursor would exceed `maxScanCursorBytes`.

### collect_account_books

```rust
pub async fn collect_account_books(
    client: &Client,
    account_id: &str,
    marketplace: &str,
    page_size: u32,
    import_episodes: bool,
    import_plus_titles: bool,
) -> Result<(Vec<ScanBook>, u32)>
```

Fetch every library page for one account without touching the library DB
(a full pass).

#### Errors

Returns an error when the operation fails.

### fetch_library_page

```rust
pub async fn fetch_library_page(
    client: &Client,
    account_id: &str,
    marketplace: &str,
    request: &LibraryPageRequest,
) -> Result<LibraryPageRows>
```

Fetch one library page for one account without touching the library DB.

A request with a `state_token` is a delta: it asks for active and revoked
items changed since that token, and revoked or soft-deleted items come
back in [`LibraryPageRows::removed`](#librarypagerows). Without one it lists the active
library and skips soft-deleted items.

#### Errors

Returns an error when the request fails.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, AudibleError>;
```

Result alias for Audible auth, sync, license, and download operations.

## Constants

### ID

```rust
pub const ID: &str = "audible";
```

Plugin id of this storefront (`audible`), stamped on accounts it returns.
