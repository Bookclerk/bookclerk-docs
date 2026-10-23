**bookclerk_plugin_source_audible > download**

# Module: download

## Contents

**Structs**

- [`AccountClient`](#accountclient) - Authenticated Audible client bound to one account.
- [`EncryptedDownload`](#encrypteddownload) - Encrypted (or plain Mpeg) audio downloaded to cache, plus decrypt material.
- [`LicenseSummary`](#licensesummary) - Public summary of a license grant (no secrets).

**Enums**

- [`DrmKind`](#drmkind) - DRM / container kind produced by download.

**Functions**

- [`download_licensed_audio`](#download_licensed_audio) - Download the licensed Adrm/Mpeg audio file to `dest` (parent dirs created).
- [`fetch_and_download`](#fetch_and_download) - Fetch license + download audio into `cache_dir` for acquire.
- [`fetch_and_download_with_client`](#fetch_and_download_with_client) - Like [`fetch_and_download_with_options`](download.md#bookclerk_plugin_source_audibledownloadfetch_and_download_with_options) but uses a pre-opened `AccountClient`,
- [`fetch_and_download_with_options`](#fetch_and_download_with_options) - Like [`fetch_and_download`](download.md#bookclerk_plugin_source_audibledownloadfetch_and_download) but honors Widevine / xHE-AAC options.
- [`invalidate_account_client_cache`](#invalidate_account_client_cache) - Drop cached clients for `account` (after login / revoke / credential rewrite).
- [`license_full_json`](#license_full_json) - Full license API JSON (classic `get-license` without summary mode).
- [`open_account_client`](#open_account_client) - Open an audible-rs [`Client`](https://docs.rs/audible_rs/latest/audible_rs/api/client/struct.Client.html) for `account` (customer id, or display label).
- [`parse_license_json`](#parse_license_json) - Parse a license JSON file (API response or classic get-license output).
- [`request_content_license`](#request_content_license) - Request an Adrm/Mpeg download license for `asin`.
- [`summarize_license`](#summarize_license) - Summarize a license without exposing voucher / URL query secrets.

---

## bookclerk_plugin_source_audible::download::AccountClient

*Struct*

Authenticated Audible client bound to one account.

Cloneable (cheap — shares the inner `Arc<Client>`). Any token refresh
propagates to all clones because they share the same `Arc<Mutex<Authenticator>>`.

**Fields:**
- `client: std::sync::Arc<audible_rs::api::client::Client>` - HTTP or RPC client used for outbound requests.
- `account_id: String` - Account Identifier.
- `marketplace: String` - Marketplace.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> AccountClient`



## bookclerk_plugin_source_audible::download::DrmKind

*Enum*

DRM / container kind produced by download.

**Variants:**
- `Adrm` - Adrm aaxc with voucher key/iv.
- `Widevine` - Widevine CENC with kid/key.
- `Mpeg` - Plain Mpeg (mp3) — no decrypt.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &DrmKind) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DrmKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_audible::download::EncryptedDownload

*Struct*

Encrypted (or plain Mpeg) audio downloaded to cache, plus decrypt material.

**Fields:**
- `path: std::path::PathBuf` - Filesystem path for this value.
- `drm_type: Option<String>` - DRM type.
- `content_format: Option<String>` - Content format.
- `content_size: Option<u64>` - Content size.
- `drm_kind: DrmKind` - DRM kind.
- `key: Option<String>` - Adrm aaxc content key (hex).
- `iv: Option<String>` - Adrm aaxc IV (hex).
- `kid: Option<String>` - Widevine CENC key id (hex).
- `cenc_key: Option<String>` - Widevine CENC content key (hex).
- `needs_decrypt: bool` - True when the file still needs decrypt (native Adrm or CENC).
- `pdf_url: Option<String>` - Pdf URL.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EncryptedDownload`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_audible::download::LicenseSummary

*Struct*

Public summary of a license grant (no secrets).

**Fields:**
- `asin: String` - Amazon ASIN identifier.
- `status_code: String` - Status code.
- `drm_type: Option<String>` - DRM type.
- `content_format: Option<String>` - Content format.
- `content_size: Option<u64>` - Content size.
- `granted: bool` - Granted.
- `denial_message: Option<String>` - Denial message.
- `has_voucher: bool` - Has voucher.
- `offline_url_present: bool` - Offline URL present.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> LicenseSummary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_audible::download::download_licensed_audio

*Function*

Download the licensed Adrm/Mpeg audio file to `dest` (parent dirs created).

# Errors

Returns an error when the operation fails.

```rust
fn download_licensed_audio(client: &audible_rs::api::client::Client, license: &audible_rs::models::content::DownloadLicense, dest: &std::path::Path, speed_limit_kbps: u32) -> crate::error::Result<EncryptedDownload>
```



## bookclerk_plugin_source_audible::download::fetch_and_download

*Function*

Fetch license + download audio into `cache_dir` for acquire.

Strategy:
- If `options.widevine`: Widevine first (CDM required unless Mpeg fallback).
- Else try Adrm; on 000307 automatically fall back to Widevine when a CDM is available.

# Errors

Returns an error when the operation fails.

```rust
fn fetch_and_download(scope: &bookclerk_library::SourceScope, files_dir: &std::path::Path, account: &str, asin: &str, quality: bookclerk_config::AudioQuality, cache_dir: &std::path::Path) -> crate::error::Result<(AccountClient, EncryptedDownload, LicenseSummary)>
```



## bookclerk_plugin_source_audible::download::fetch_and_download_with_client

*Function*

Like [`fetch_and_download_with_options`](download.md#bookclerk_plugin_source_audibledownloadfetch_and_download_with_options) but uses a pre-opened `AccountClient`,
avoiding a repeated DB lookup + decryption when the caller maintains a per-job cache.

# Errors

Returns an error when the operation fails.

```rust
fn fetch_and_download_with_client(account_client: AccountClient, files_dir: &std::path::Path, asin: &str, options: &crate::options::DownloadOptions, cache_dir: &std::path::Path, scope: Option<&bookclerk_library::SourceScope>) -> crate::error::Result<(AccountClient, EncryptedDownload, LicenseSummary)>
```



## bookclerk_plugin_source_audible::download::fetch_and_download_with_options

*Function*

Like [`fetch_and_download`](download.md#bookclerk_plugin_source_audibledownloadfetch_and_download) but honors Widevine / xHE-AAC options.

Auth and Widevine CDM bytes come from `encrypted_secrets`. `files_dir` is
only used for temporary cache / optional BYO CDM path resolution.

# Errors

Returns an error when the operation fails.

```rust
fn fetch_and_download_with_options(scope: &bookclerk_library::SourceScope, files_dir: &std::path::Path, account: &str, asin: &str, options: &crate::options::DownloadOptions, cache_dir: &std::path::Path) -> crate::error::Result<(AccountClient, EncryptedDownload, LicenseSummary)>
```



## bookclerk_plugin_source_audible::download::invalidate_account_client_cache

*Function*

Drop cached clients for `account` (after login / revoke / credential rewrite).

```rust
fn invalidate_account_client_cache(account: &str)
```



## bookclerk_plugin_source_audible::download::license_full_json

*Function*

Full license API JSON (classic `get-license` without summary mode).

```rust
fn license_full_json(license: &audible_rs::models::content::DownloadLicense) -> String
```



## bookclerk_plugin_source_audible::download::open_account_client

*Function*

Open an audible-rs [`Client`](https://docs.rs/audible_rs/latest/audible_rs/api/client/struct.Client.html) for `account` (customer id, or display label).

Credentials are keyed by Audible customer id. A display label is resolved
via the scoped accounts table. Clients are cached process-wide (same idea
as the library unseal cache) so batch acquire does not re-open auth for
every title — no special-casing in the acquire job.

# Errors

Returns an error when the operation fails.

```rust
fn open_account_client(scope: &bookclerk_library::SourceScope, account: &str) -> crate::error::Result<AccountClient>
```



## bookclerk_plugin_source_audible::download::parse_license_json

*Function*

Parse a license JSON file (API response or classic get-license output).

# Errors

Returns an error when the operation fails.

```rust
fn parse_license_json(text: &str) -> crate::error::Result<audible_rs::models::content::DownloadLicense>
```



## bookclerk_plugin_source_audible::download::request_content_license

*Function*

Request an Adrm/Mpeg download license for `asin`.

# Errors

Returns an error when the operation fails.

```rust
fn request_content_license(client: &audible_rs::api::client::Client, marketplace: &str, asin: &str, quality: bookclerk_config::AudioQuality) -> crate::error::Result<audible_rs::models::content::DownloadLicense>
```



## bookclerk_plugin_source_audible::download::summarize_license

*Function*

Summarize a license without exposing voucher / URL query secrets.

```rust
fn summarize_license(license: &audible_rs::models::content::DownloadLicense) -> LicenseSummary
```



