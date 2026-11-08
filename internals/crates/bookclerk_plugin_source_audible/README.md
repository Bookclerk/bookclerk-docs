# bookclerk_plugin_source_audible

Audible storefront guest: OAuth login, library scan, catalog lookups and
download + decrypt, over [audible-rs](https://github.com/mkb79/audible-rs).

The binary serves these to the host as a jailed plugin guest. The host
seals the credentials and passes them in on each call; nothing here opens
the library database.

| Item | Kind | Summary |
| --- | --- | --- |
| [`LicenseSummary`](#licensesummary) | struct | Public summary of a license grant (no secrets). |
| [`DownloadOptions`](#downloadoptions) | struct | What one `fetchTitle` call asks of the download path. |
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
| [`guest_scan`](#guest_scan) | function | Scan libraries for host-injected credential blobs. |
| [`collect_account_books`](#collect_account_books) | function | Fetch library pages for one account without touching the library DB. |
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

### guest_scan

```rust
pub async fn guest_scan(
    credentials: &BTreeMap<String, Value>,
    account_filter: &[String],
    page_size: u32,
    import_episodes: bool,
    import_plus_titles: bool,
) -> Result<ScanSummary>
```

Scan libraries for host-injected credential blobs.

#### Errors

Returns an error when the operation fails.

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

Fetch library pages for one account without touching the library DB.

Used by the external guest plugin (the host resolves the structured rows).

#### Errors

Returns an error when the operation fails.

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
