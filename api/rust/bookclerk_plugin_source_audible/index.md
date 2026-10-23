# bookclerk_plugin_source_audible

Bookclerk-facing audible API: auth, accounts, library sync, download options.

Thin wrapper over [audible-rs](https://github.com/mkb79/audible-rs) so Bookclerk
keeps stable naming. All credentials live in the `encrypted_secrets` DB table.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AccountInfo`](#accountinfo) | struct | Summary of a configured account. |
| [`AuthLoginOptions`](#authloginoptions) | struct | Inputs for interactive Audible OAuth (SPA Accounts connect / CLI login helpers). |
| [`AuthSession`](#authsession) | struct | Metadata returned after a successful Audible OAuth session. |
| [`AccountClient`](#accountclient) | struct | Authenticated Audible client bound to one account. |
| [`EncryptedDownload`](#encrypteddownload) | struct | Encrypted (or plain Mpeg) audio downloaded to cache, plus decrypt material. |
| [`LicenseSummary`](#licensesummary) | struct | Public summary of a license grant (no secrets). |
| [`AudibleSource`](#audiblesource) | struct | Audible storefront adapter implementing `ContentSource`. |
| [`WidevineCdm`](#widevinecdm) | struct | Loaded Widevine CDM client. |
| [`WidevineDownload`](#widevinedownload) | struct | Result of a Widevine (or Mpeg-fallback) download into `work_dir`. |
| [`AccountStatus`](#accountstatus) | enum | Token health. |
| [`LoginMode`](#loginmode) | enum | How to complete the browser sign-in. |
| [`LoginProgress`](#loginprogress) | enum | Progress events emitted during login (for CLI / TUI). |
| [`DrmKind`](#drmkind) | enum | DRM / container kind produced by download. |
| [`AudibleError`](#audibleerror) | enum | Failures from Audible login, library sync, licensing, or download/decrypt. |
| [`QrRenderMode`](#qrrendermode) | enum | How to print the login QR in a terminal. |
| [`import_auth_file`](#import_auth_file) | function | Import an audible-rs auth file into the `encrypted_secrets` table. |
| [`import_libation_accounts_json`](#import_libation_accounts_json) | function | Import classic Libation `AccountsSettings.json` account metadata only. |
| [`import_mkb79_auth_json`](#import_mkb79_auth_json) | function | Import mkb79/audible-cli legacy auth JSON (LibationCli: `import-account`). |
| [`download_companion_pdf`](#download_companion_pdf) | function | Download companion PDF via the authenticated companion-file endpoint. |
| [`download_cover_jpeg`](#download_cover_jpeg) | function | Download a JPEG cover at `size` (e.g. `500`, `1215`, or `native`). |
| [`fetch_chapter_info`](#fetch_chapter_info) | function | Fetch curated chapter metadata (`chapter_info` object). |
| [`fetch_clips_bookmarks`](#fetch_clips_bookmarks) | function | Fetch clips/bookmarks/notes sidecar (classic `DownloadClipsBookmarks`). |
| [`fetch_product_metadata`](#fetch_product_metadata) | function | Fetch full catalog metadata for a title (classic `metadata.json` sidecar). |
| [`begin_login`](#begin_login) | function | Runs Audible OAuth via audible-rs and returns `AuthSession` metadata. |
| [`download_licensed_audio`](#download_licensed_audio) | function | Download the licensed Adrm/Mpeg audio file to `dest` (parent dirs created). |
| [`fetch_and_download`](#fetch_and_download) | function | Fetch license + download audio into `cache_dir` for acquire. |
| [`fetch_and_download_with_client`](#fetch_and_download_with_client) | function | Like `fetch_and_download_with_options` but uses a pre-opened `AccountClient`, avoiding a repeated DB lookup + decryption when the caller maintains a per-job cache. |
| [`fetch_and_download_with_options`](#fetch_and_download_with_options) | function | Like `fetch_and_download` but honors Widevine / xHE-AAC options. |
| [`invalidate_account_client_cache`](#invalidate_account_client_cache) | function | Drop cached clients for `account` (after login / revoke / credential rewrite). |
| [`license_full_json`](#license_full_json) | function | Full license API JSON (classic `get-license` without summary mode). |
| [`open_account_client`](#open_account_client) | function | Open an audible-rs `Client` for `account` (customer id, or display label). |
| [`parse_license_json`](#parse_license_json) | function | Parse a license JSON file (API response or classic get-license output). |
| [`request_content_license`](#request_content_license) | function | Request an Adrm/Mpeg download license for `asin`. |
| [`summarize_license`](#summarize_license) | function | Summarize a license without exposing voucher / URL query secrets. |
| [`credentials_json_from_auth`](#credentials_json_from_auth) | function | Build credential JSON from an authenticator (optional Widevine CDM bytes). |
| [`guest_fetch_title`](#guest_fetch_title) | function | Download + decrypt one title; return plain paths (never encrypted on the wire). |
| [`guest_login_complete`](#guest_login_complete) | function | Await a pending OAuth session and return account + opaque credential bytes. |
| [`guest_login_start`](#guest_login_start) | function | Start LoginServer OAuth; return `(session_id, browser_url)`. |
| [`guest_scan`](#guest_scan) | function | Scan libraries for host-injected credential blobs. |
| [`render_login_qr`](#render_login_qr) | function | Render `url` as a terminal QR code string (plus the raw URL). |
| [`resolve_auth_password`](#resolve_auth_password) | function | Resolve the auth encryption passphrase from `BOOKCLERK_AUTH_PASSWORD`, if set. |
| [`collect_account_books`](#collect_account_books) | function | Fetch library pages for one account without touching the library DB. |
| [`scan_account_into_library`](#scan_account_into_library) | function | Fetch library pages for one authenticated client and upsert into `library`. |
| [`scan_library`](#scan_library) | function | Sync Audible library for configured accounts into `library`. |
| [`effective_cdm_provider`](#effective_cdm_provider) | function | Resolve provider URL: `None` config → built-in Bookclerk AudibleCdm; empty/`off` → disabled. |
| [`ensure_widevine_cdm`](#ensure_widevine_cdm) | function | Load an account's Widevine CDM, or auto-provision one via the classic Libation AudibleCdm provider. CDMs are stored in the `encrypted_secrets` DB table only — nothing is written under `Accounts/`. |
| [`load_widevine_cdm`](#load_widevine_cdm) | function | Resolve and load a bring-your-own `.wvd` device blob from local files. |
| [`Result`](#result) | type alias | Result alias for Audible auth, sync, license, and download operations. |
| [`AUTH_PASSWORD_ENV`](#auth_password_env) | constant | Environment variable holding the auth encryption passphrase. |
| [`AUDIBLE_SOURCE_ID`](#audible_source_id) | constant | Describe / config id for this store (`audible` in `[sources.audible]`). |
| [`DEFAULT_WIDEVINE_CDM_PROVIDER`](#default_widevine_cdm_provider) | constant | Classic Libation AudibleCdm endpoint — provisions a unique L3 `.wvd` per account. |

## Re-exports

- [`audible_rs::models::content::DownloadLicense`](https://docs.rs/audible_rs/latest/audible_rs/models/content/struct.DownloadLicense.html)
- [`bookclerk_source::ScanOptions`](../bookclerk_source/index.md#scanoptions)
- [`bookclerk_source::ScanSummary`](../bookclerk_source/index.md#scansummary)
- [`catalog::expand_candidates` as `catalog_expand_candidates`](catalog.md#expand_candidates)
- [`catalog::purchase_hint` as `catalog_purchase_hint`](catalog.md#purchase_hint)
- [`catalog::search_catalog` as `catalog_search`](catalog.md#search_catalog)
- [`db::delete_audible_account_from_db`](db.md#delete_audible_account_from_db)
- [`db::list_audible_accounts_from_db`](db.md#list_audible_accounts_from_db)
- [`db::load_authenticator_from_db`](db.md#load_authenticator_from_db)
- [`db::load_widevine_cdm_from_db`](db.md#load_widevine_cdm_from_db)
- [`db::save_authenticator_to_db`](db.md#save_authenticator_to_db)
- [`db::save_widevine_cdm_to_db`](db.md#save_widevine_cdm_to_db)
- [`options::DownloadOptions`](../bookclerk_source/index.md#downloadoptions)

## Modules

- [`catalog`](catalog.md): Public Audible catalog helpers for Discover (`search_catalog` / expand / purchase).
- [`db`](db.md): DB-backed credential storage for Audible accounts.

## Structs

### AccountInfo

```rust
pub struct AccountInfo {
    pub account_id: String,
    pub marketplace: String,
    pub label: Option<String>,
    pub status: AccountStatus,
    pub auth_file: Option<String>,
}
```

Summary of a configured account.

**Fields**

- `account_id`: Account Identifier.
- `marketplace`: Marketplace.
- `label`: Operator-visible label for this account or item.
- `status`: Status.
- `auth_file`: Reference to where credentials live (`encrypted_secrets:<id>`), for display only.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AuthLoginOptions

```rust
pub struct AuthLoginOptions {
    pub marketplace: String,
    pub label: Option<String>,
    pub callback_bind: SocketAddr,
    pub response_url: Option<String>,
    pub show_qr: bool,
    pub qr_mode: QrRenderMode,
    pub mode: LoginMode,
    pub timeout_secs: u64,
    pub audible_username: bool,
    pub force: bool,
    pub scope: Option<SourceScope>,
}
```

Inputs for interactive Audible OAuth (SPA Accounts connect / CLI login helpers).

Prefer Accounts UI for operator flows; these options also drive guest
`loginStart` / in-process [`begin_login`](#begin_login).

**Fields**

- `marketplace`: Audible marketplace / locale code (`us`, `uk`, `de`, …).
- `label`: Optional operator-facing account label stored with the secret.
- `callback_bind`: Bind address for the local OAuth redirect receiver (`127.0.0.1:0` = ephemeral).
- `response_url`: Pre-captured redirect URL (external mode / paste); skips waiting on the server.
- `show_qr`: When true, emit QR text alongside the authorize URL for headless terminals.
- `qr_mode`: How to render the QR payload ([`QrRenderMode`](#qrrendermode)).
- `mode`: Whether to run a local callback server or require an external redirect paste.
- `timeout_secs`: Seconds to wait for LoginServer capture before timing out.
- `audible_username`: Pre-merger Audible username login (DE/US/UK only).
- `force`: Overwrite an existing sealed authenticator for the same account id.
- `scope`: When `Some`, credentials are persisted via the Audible
  [`bookclerk_library::SourceScope`](../bookclerk_library/scope.md#sourcescope) (same scoping rules as third-party
  plugins). When `None`, only session metadata is returned (guest path).

Implements: `Clone`, `Debug`, `Default`

### AuthSession

```rust
pub struct AuthSession {
    pub account_id: String,
    pub marketplace: String,
    pub label: Option<String>,
    pub customer_id: Option<String>,
}
```

Metadata returned after a successful Audible OAuth session.

Callers that pass [`AuthLoginOptions::scope`](#authloginoptions) already have credentials in
`encrypted_secrets`; guest callers export the authenticator separately.

**Fields**

- `account_id`: Stable Bookclerk account id for library rows and secret lookup.
- `marketplace`: Marketplace used for the login (`us`, `uk`, …).
- `label`: Optional operator-facing label from [`AuthLoginOptions::label`](#authloginoptions).
- `customer_id`: Audible customer id from the authenticator when present.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AccountClient

```rust
pub struct AccountClient {
    pub client: Arc<Client>,
    pub account_id: String,
    pub marketplace: String,
}
```

Authenticated Audible client bound to one account.

Cloneable (cheap — shares the inner `Arc<Client>`). Any token refresh
propagates to all clones because they share the same `Arc<Mutex<Authenticator>>`.

**Fields**

- `client`: HTTP or RPC client used for outbound requests.
- `account_id`: Account Identifier.
- `marketplace`: Marketplace.

Implements: `Clone`, `Debug`

### EncryptedDownload

```rust
pub struct EncryptedDownload {
    pub path: PathBuf,
    pub drm_type: Option<String>,
    pub content_format: Option<String>,
    pub content_size: Option<u64>,
    pub drm_kind: DrmKind,
    pub key: Option<String>,
    pub iv: Option<String>,
    pub kid: Option<String>,
    pub cenc_key: Option<String>,
    pub needs_decrypt: bool,
    pub pdf_url: Option<String>,
}
```

Encrypted (or plain Mpeg) audio downloaded to cache, plus decrypt material.

**Fields**

- `path`: Filesystem path for this value.
- `drm_type`: DRM type.
- `content_format`: Content format.
- `content_size`: Content size.
- `drm_kind`: DRM kind.
- `key`: Adrm aaxc content key (hex).
- `iv`: Adrm aaxc IV (hex).
- `kid`: Widevine CENC key id (hex).
- `cenc_key`: Widevine CENC content key (hex).
- `needs_decrypt`: True when the file still needs decrypt (native Adrm or CENC).
- `pdf_url`: Pdf URL.

Implements: `Clone`, `Debug`

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

### AudibleSource

```rust
pub struct AudibleSource {
    pub bitrate: AudioQuality,
}
```

Audible storefront adapter implementing [`ContentSource`](../bookclerk_source/index.md#contentsource).

Construct with [`AudibleSource::new`](#audiblesourcenew); the host delivers the instance's
`bitrate` setting through `CONFIG` at open, and tests use
[`AudibleSource::with_bitrate`](#audiblesourcewith_bitrate) directly.

**Fields**

- `bitrate`: License bitrate tier (the instance's `bitrate` setting).

#### AudibleSource::new

```rust
pub fn new() -> Self
```

Returns an Audible source with default bitrate (`High` / config default).

#### AudibleSource::with_bitrate

```rust
pub fn with_bitrate(self, bitrate: AudioQuality) -> Self
```

Overrides the license bitrate tier used for download requests.

Implements: `Clone`, `ContentSource`, `Copy`, `Debug`, `Default`

### WidevineCdm

```rust
pub struct WidevineCdm {
    pub security_level: u8,
    /* private fields */
}
```

Loaded Widevine CDM client.

**Fields**

- `security_level`: Security level.

### WidevineDownload

```rust
pub struct WidevineDownload {
    pub path: PathBuf,
    pub drm_type: String,
    pub content_format: Option<String>,
    pub content_size: Option<u64>,
    pub kid: Option<String>,
    pub key: Option<String>,
    pub needs_decrypt: bool,
    pub pdf_url: Option<String>,
}
```

Result of a Widevine (or Mpeg-fallback) download into `work_dir`.

**Fields**

- `path`: Filesystem path for this value.
- `drm_type`: DRM type.
- `content_format`: Content format.
- `content_size`: Content size.
- `kid`: CENC key id (hex); absent for plain Mpeg.
- `key`: CENC content key (hex).
- `needs_decrypt`: Needs decrypt.
- `pdf_url`: Pdf URL.

Implements: `Clone`, `Debug`

## Enums

### AccountStatus

```rust
pub enum AccountStatus {
    Valid,
    ExpiringSoon,
    Expired,
    MissingRefresh,
    Unknown,
}
```

Token health.

**Variants**

- `Valid`: Valid variant.
- `ExpiringSoon`: Expiring soon variant.
- `Expired`: Expired variant.
- `MissingRefresh`: Missing refresh variant.
- `Unknown`: Unknown variant.

#### AccountStatus::as_str

```rust
pub fn as_str(self) -> &'static str
```

As str.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LoginMode

```rust
pub enum LoginMode {
    Server,
    External,
}
```

How to complete the browser sign-in.

**Variants**

- `Server`: Local reverse-proxy + QR (headless / SSH friendly). Default.
- `External`: Print authorize URL; read redirect from `--response-url` or stdin.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### LoginProgress

```rust
pub enum LoginProgress {
    LoginUrl { url: String, qr: Option<String> },
    CallbackListening { addr: SocketAddr },
    WaitingForCallback,
    Completed { account_id: String },
}
```

Progress events emitted during login (for CLI / TUI).

**Variants**

- `LoginUrl`: Browser URL the operator should open (optional pre-rendered QR text).
  - `url`: Absolute HTTPS login URL.
  - `qr`: Optional pre-rendered QR text for terminal UIs.
- `CallbackListening`: Local callback server is listening.
  - `addr`: Socket address of the local redirect receiver.
- `WaitingForCallback`: Waiting for the OAuth redirect / callback.
- `Completed`: Login finished for this account id.
  - `account_id`: Account id stored after a successful login.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### DrmKind

```rust
pub enum DrmKind {
    Adrm,
    Widevine,
    Mpeg,
}
```

DRM / container kind produced by download.

**Variants**

- `Adrm`: Adrm aaxc with voucher key/iv.
- `Widevine`: Widevine CENC with kid/key.
- `Mpeg`: Plain Mpeg (mp3) — no decrypt.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### AudibleError

```rust
pub enum AudibleError {
    Auth(String),
    NoAccounts(String),
    AccountNotFound(String),
    Import(String),
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
- `NoAccounts`: No Audible accounts are configured (safe to skip in multi-source scan).
- `AccountNotFound`: Requested account id is not present in `encrypted_secrets`.
- `Import`: Import of an external auth / Libation / mkb79 file failed.
- `Sync`: Library listing or metadata sync against Audible APIs failed.
- `License`: Content license request was rejected or returned an unexpected payload.
- `NoAaxcAsset`: Adrm licenserequest returned 000307 — title is Widevine-only (no aaxc).
  - `asin`: Title ASIN that has no AAXC asset.
  - `message`: Upstream error detail from Audible.
- `Widevine`: Widevine CDM load, provisioning, or CENC decrypt failed.
- `Download`: HTTP download of audio/PDF/cover bytes failed.
- `Io`: Local filesystem I/O while reading or writing cache files.
- `Other`: Unexpected failure wrapped from `anyhow` / library layers.

#### AudibleError::is_no_aaxc_asset

```rust
pub fn is_no_aaxc_asset(&self) -> bool
```

Returns true when this error is [`Self::NoAaxcAsset`](#audibleerror) (retry via Widevine).

Implements: `Debug`, `Display`, `Error`, `From<ApiError>`, `From<AuthError>`, `From<Error>`, `From<LibraryError>`, `From<LoginError>`

### QrRenderMode

```rust
pub enum QrRenderMode {
    Unicode,
    Ascii,
}
```

How to print the login QR in a terminal.

**Variants**

- `Unicode`: Dense Unicode half-blocks (default; works in most modern terminals).
- `Ascii`: ASCII `#` / space art for limited terminals.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

## Functions

### import_auth_file

```rust
pub async fn import_auth_file(
    scope: &SourceScope,
    source: &Path,
    label: Option<&str>,
    force: bool,
) -> Result<AccountInfo>
```

Import an audible-rs auth file into the `encrypted_secrets` table.

Reads (decrypting via `BOOKCLERK_AUTH_PASSWORD` when needed) a user-supplied
auth file, then persists the authenticator into the DB. No `Accounts/` file
is written.

#### Errors

Returns an error when the operation fails.

### import_libation_accounts_json

```rust
pub fn import_libation_accounts_json(path: &Path) -> Result<Vec<AccountInfo>>
```

Import classic Libation `AccountsSettings.json` account metadata only.

IdentityTokens are not converted (discarded file-based auth path). Use
[`import_auth_file`](#import_auth_file) / [`import_mkb79_auth_json`](#import_mkb79_auth_json) or interactive login for
credentials.

#### Errors

Returns an error when the operation fails.

### import_mkb79_auth_json

```rust
pub async fn import_mkb79_auth_json(
    scope: &SourceScope,
    source: &Path,
    label: Option<&str>,
    force: bool,
) -> Result<AccountInfo>
```

Import mkb79/audible-cli legacy auth JSON (LibationCli: `import-account`).

#### Errors

Returns an error when the operation fails.

### download_companion_pdf

```rust
pub async fn download_companion_pdf(
    client: &Client,
    marketplace: &str,
    asin: &str,
    dest: &Path,
) -> Result<Option<PathBuf>>
```

Download companion PDF via the authenticated companion-file endpoint.

#### Errors

Returns an error when the operation fails.

### download_cover_jpeg

```rust
pub async fn download_cover_jpeg(
    client: &Client,
    marketplace: &str,
    asin: &str,
    size: &str,
    dest: &Path,
) -> Result<Option<PathBuf>>
```

Download a JPEG cover at `size` (e.g. `500`, `1215`, or `native`).

#### Errors

Returns an error when the operation fails.

### fetch_chapter_info

```rust
pub async fn fetch_chapter_info(
    client: &Client,
    marketplace: &str,
    asin: &str,
    quality: AudioQuality,
    chapter_layout: &str,
) -> Result<Value>
```

Fetch curated chapter metadata (`chapter_info` object).

#### Errors

Returns an error when the operation fails.

### fetch_clips_bookmarks

```rust
pub async fn fetch_clips_bookmarks(
    client: &Client,
    asin: &str,
    acr: Option<&str>,
    version: Option<&str>,
    content_format: Option<&str>,
) -> Result<Option<Value>>
```

Fetch clips/bookmarks/notes sidecar (classic `DownloadClipsBookmarks`).

#### Errors

Returns an error when the operation fails.

### fetch_product_metadata

```rust
pub async fn fetch_product_metadata(
    client: &Client,
    marketplace: &str,
    asin: &str,
    quality: AudioQuality,
) -> Result<Value>
```

Fetch full catalog metadata for a title (classic `metadata.json` sidecar).

#### Errors

Returns an error when the operation fails.

### begin_login

```rust
pub async fn begin_login(
    opts: AuthLoginOptions,
    on_progress: impl FnMut(LoginProgress),
) -> Result<AuthSession>
```

Runs Audible OAuth via audible-rs and returns [`AuthSession`](#authsession) metadata.

When [`AuthLoginOptions::scope`](#authloginoptions) is `Some`, credentials are persisted to
`encrypted_secrets`. When `None` (guest / external plugin path), login
completes and returns session metadata only — the caller exports the
authenticator separately.

#### Arguments

* `opts` - Marketplace, callback bind, login mode, and optional DB scope.
* `on_progress` - Callback for URL / QR / waiting / completed events (CLI/TUI).

#### Errors

Returns [`AudibleError::Auth`](#audibleerror) when locale, device registration, or OAuth fails.

### download_licensed_audio

```rust
pub async fn download_licensed_audio(
    client: &Client,
    license: &DownloadLicense,
    dest: &Path,
    speed_limit_kbps: u32,
) -> Result<EncryptedDownload>
```

Download the licensed Adrm/Mpeg audio file to `dest` (parent dirs created).

#### Errors

Returns an error when the operation fails.

### fetch_and_download

```rust
pub async fn fetch_and_download(
    scope: &SourceScope,
    files_dir: &Path,
    account: &str,
    asin: &str,
    quality: AudioQuality,
    cache_dir: &Path,
) -> Result<(AccountClient, EncryptedDownload, LicenseSummary)>
```

Fetch license + download audio into `cache_dir` for acquire.

Strategy:
- If `options.widevine`: Widevine first (CDM required unless Mpeg fallback).
- Else try Adrm; on 000307 automatically fall back to Widevine when a CDM is available.

#### Errors

Returns an error when the operation fails.

### fetch_and_download_with_client

```rust
pub async fn fetch_and_download_with_client(
    account_client: AccountClient,
    files_dir: &Path,
    asin: &str,
    options: &DownloadOptions,
    cache_dir: &Path,
    scope: Option<&SourceScope>,
) -> Result<(AccountClient, EncryptedDownload, LicenseSummary)>
```

Like [`fetch_and_download_with_options`](#fetch_and_download_with_options) but uses a pre-opened `AccountClient`,
avoiding a repeated DB lookup + decryption when the caller maintains a per-job cache.

#### Errors

Returns an error when the operation fails.

### fetch_and_download_with_options

```rust
pub async fn fetch_and_download_with_options(
    scope: &SourceScope,
    files_dir: &Path,
    account: &str,
    asin: &str,
    options: &DownloadOptions,
    cache_dir: &Path,
) -> Result<(AccountClient, EncryptedDownload, LicenseSummary)>
```

Like [`fetch_and_download`](#fetch_and_download) but honors Widevine / xHE-AAC options.

Auth and Widevine CDM bytes come from `encrypted_secrets`. `files_dir` is
only used for temporary cache / optional BYO CDM path resolution.

#### Errors

Returns an error when the operation fails.

### invalidate_account_client_cache

```rust
pub fn invalidate_account_client_cache(account: &str)
```

Drop cached clients for `account` (after login / revoke / credential rewrite).

### license_full_json

```rust
pub fn license_full_json(license: &DownloadLicense) -> String
```

Full license API JSON (classic `get-license` without summary mode).

### open_account_client

```rust
pub async fn open_account_client(
    scope: &SourceScope,
    account: &str,
) -> Result<AccountClient>
```

Open an audible-rs [`Client`](https://docs.rs/audible_rs/latest/audible_rs/api/client/struct.Client.html) for `account` (customer id, or display label).

Credentials are keyed by Audible customer id. A display label is resolved
via the scoped accounts table. Clients are cached process-wide (same idea
as the library unseal cache) so batch acquire does not re-open auth for
every title — no special-casing in the acquire job.

#### Errors

Returns an error when the operation fails.

### parse_license_json

```rust
pub fn parse_license_json(text: &str) -> Result<DownloadLicense>
```

Parse a license JSON file (API response or classic get-license output).

#### Errors

Returns an error when the operation fails.

### request_content_license

```rust
pub async fn request_content_license(
    client: &Client,
    marketplace: &str,
    asin: &str,
    quality: AudioQuality,
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

### credentials_json_from_auth

```rust
pub fn credentials_json_from_auth(
    auth: &Authenticator,
    widevine: Option<&[u8]>,
) -> Result<Value>
```

Build credential JSON from an authenticator (optional Widevine CDM bytes).

#### Errors

Returns an error when the operation fails.

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
field; packaging knobs use guest defaults. Plugin bitrate from
`source_config` overlays `quality` (same as in-process).

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

### render_login_qr

```rust
pub fn render_login_qr(url: &str, mode: QrRenderMode) -> Result<String>
```

Render `url` as a terminal QR code string (plus the raw URL).

#### Errors

Returns an error when the operation fails.

### resolve_auth_password

```rust
pub fn resolve_auth_password() -> Result<Option<SecretString>>
```

Resolve the auth encryption passphrase from `BOOKCLERK_AUTH_PASSWORD`, if set.

Used for reading legacy `json-encrypted` rows and loading external
audible-rs files that were previously encrypted with this passphrase.

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
) -> Result<(Vec<NewBook>, u32)>
```

Fetch library pages for one account without touching the library DB.

Used by the external guest plugin (host upserts [`NewBook`](../bookclerk_library/index.md#newbook) rows).

#### Errors

Returns an error when the operation fails.

### scan_account_into_library

```rust
pub async fn scan_account_into_library(
    scope: &SourceScope,
    client: &Client,
    account_id: &str,
    marketplace: &str,
    page_size: u32,
    import_episodes: bool,
    import_plus_titles: bool,
) -> Result<(usize, u32)>
```

Fetch library pages for one authenticated client and upsert into `library`.

Exposed for wiremock CI tests (inject a [`Client`](https://docs.rs/audible_rs/latest/audible_rs/api/client/struct.Client.html) with `api_base_override`).

#### Errors

Returns an error when the operation fails.

### scan_library

```rust
pub async fn scan_library(
    scope: &SourceScope,
    options: ScanOptions,
) -> Result<ScanSummary>
```

Sync Audible library for configured accounts into `library`.

Accounts are resolved from the `encrypted_secrets` table (DB-backed); no
`Accounts/*.audible.auth` files are read.

#### Errors

Returns an error when the operation fails.

### effective_cdm_provider

```rust
pub fn effective_cdm_provider(configured: Option<&str>) -> Option<&str>
```

Resolve provider URL: `None` config → built-in Bookclerk AudibleCdm; empty/`off` → disabled.

### ensure_widevine_cdm

```rust
pub async fn ensure_widevine_cdm(
    files_dir: &Path,
    configured: Option<&Path>,
    account_stem: Option<&str>,
    provider_url: Option<&str>,
    scope: Option<&SourceScope>,
) -> Result<(WidevineCdm, PathBuf)>
```

Load an account's Widevine CDM, or auto-provision one via the classic
Libation AudibleCdm provider. CDMs are stored in the `encrypted_secrets` DB
table only — nothing is written under `Accounts/`.

Resolution order:
1. `encrypted_secrets` (DB) for `account_stem`
2. BYO `.wvd` file (`output.widevine_cdm` / `{files_dir}/widevine.wvd`),
   imported into the DB when possible
3. Provision a fresh L3 CDM from the provider (auth material from the DB) and
   persist it to the DB

Widevine drmlicense requires an **Android**-registered account
(`bookclerk auth login` always registers as Android). Spatial/Atmos (L1) remains unavailable.

#### Errors

Returns an error when the operation fails.

### load_widevine_cdm

```rust
pub fn load_widevine_cdm(
    files_dir: &Path,
    configured: Option<&Path>,
) -> Result<(WidevineCdm, PathBuf)>
```

Resolve and load a bring-your-own `.wvd` device blob from local files.

This is a one-shot import path only (no `Accounts/`). Search order:
1. Explicit `output.widevine_cdm` path (absolute, or relative to `files_dir`)
2. `{files_dir}/widevine.wvd`

#### Errors

Returns an error when the operation fails.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, AudibleError>;
```

Result alias for Audible auth, sync, license, and download operations.

## Constants

### AUTH_PASSWORD_ENV

```rust
pub const AUTH_PASSWORD_ENV: &str = "BOOKCLERK_AUTH_PASSWORD";
```

Environment variable holding the auth encryption passphrase.

For new installs this is optional — the process DEK (`master.key`) is
sufficient. Set it to wrap `master.key` at rest (strongly recommended for
production) and to read legacy `json-encrypted` DB rows.

### AUDIBLE_SOURCE_ID

```rust
pub const AUDIBLE_SOURCE_ID: &str = "audible";
```

Describe / config id for this store (`audible` in `[sources.audible]`).

### DEFAULT_WIDEVINE_CDM_PROVIDER

```rust
pub const DEFAULT_WIDEVINE_CDM_PROVIDER: &str = "https://ollj0gz40d.execute-api.us-west-2.amazonaws.com/default/AudibleCdm";
```

Classic Libation AudibleCdm endpoint — provisions a unique L3 `.wvd` per account.
