# bookclerk_plugin_source_graphicaudio::guest

Guest-process helpers for the external GraphicAudio source plugin.

These APIs do **not** open the library DB — the host seals credentials and
upserts scan rows via `bookclerk_plugin_host::ExternalSource`. Conversions
between storefront types and the typed ABI live in [`bookclerk_source::abi`](../bookclerk_source/abi.md).

| Item | Kind | Summary |
| --- | --- | --- |
| [`guest_login`](#guest_login) | function | Login against GraphicAudio and return account metadata + credential JSON. |
| [`guest_login_rpc`](#guest_login_rpc) | function | RPC login: build `LoginOptions` from typed params and return the ABI result. |
| [`guest_scan`](#guest_scan) | function | Scan libraries for the credential blobs the host injected. |
| [`guest_scan_rpc`](#guest_scan_rpc) | function | RPC scan: return the typed `ScanSummary` (host upserts books). |
| [`guest_fetch_title`](#guest_fetch_title) | function | Download one title into `cache_dir` using host-injected credentials. |
| [`guest_fetch_title_rpc`](#guest_fetch_title_rpc) | function | RPC fetch: return the typed plain fetch result. |
| [`resolve_access_base_url`](#resolve_access_base_url) | function | Resolve Access App API base URL from `[sources.graphicaudio]` JSON. |
| [`resolve_store_base_url`](#resolve_store_base_url) | function | Resolve Magento storefront base URL from `[sources.graphicaudio]` JSON. |
| [`resolve_access`](#resolve_access) | function | Resolve download access path from granted `CONFIG`. |
| [`resolve_bitrate`](#resolve_bitrate) | function | Resolve device bitrate preference from `[sources.graphicaudio]` JSON. |
| [`resolve_container`](#resolve_container) | function | Resolve ZIP container preference from `[sources.graphicaudio]` JSON. |
| [`source_id`](#source_id) | function | Source id returned in describe(); must match `crate::source::ID`. |

## Functions

### guest_login

```rust
pub async fn guest_login(
    access_base_url: &str,
    store_base_url: &str,
    access: GraphicAudioAccess,
    opts: LoginOptions,
) -> Result<(String, String, Option<String>, bool, Value)>
```

Login against GraphicAudio and return account metadata + credential JSON.

- `access=web|zip`: Magento customer login only (no Access App device slot).
- `access=device`: Access App `activation/login` (registers a device).

Does not write to `encrypted_secrets` — the host seals `credentials`.

#### Errors

Returns an error when the operation fails.

### guest_login_rpc

```rust
pub async fn guest_login_rpc(
    access_base_url: &str,
    store_base_url: &str,
    access: GraphicAudioAccess,
    params: LoginParams,
) -> Result<LoginResult>
```

RPC login: build [`LoginOptions`](../bookclerk_source/index.md#loginoptions) from typed params and return the ABI result.

#### Errors

Returns an error when the operation fails.

### guest_scan

```rust
pub async fn guest_scan(
    access_base_url: &str,
    store_base_url: &str,
    access: GraphicAudioAccess,
    magento_password: Option<&str>,
    credentials: &BTreeMap<String, Value>,
    account_filter: &[String],
    include_samples: bool,
) -> Result<(Vec<NewBook>, usize, u32)>
```

Scan libraries for the credential blobs the host injected.

`magento_password` is required for Magento web/zip listing unless a device
token is present on the credentials (legacy Access App fallback).

#### Errors

Returns an error when the operation fails.

### guest_scan_rpc

```rust
pub async fn guest_scan_rpc(
    access_base_url: &str,
    store_base_url: &str,
    access: GraphicAudioAccess,
    magento_password: Option<&str>,
    params: &ScanParams,
) -> Result<ScanSummary>
```

RPC scan: return the typed [`ScanSummary`](../bookclerk_plugin_abi/generated.md#scansummary) (host upserts books).

#### Errors

Returns an error when the operation fails.

### guest_fetch_title

```rust
pub async fn guest_fetch_title(
    access_base_url: &str,
    store_base_url: &str,
    credentials: &Value,
    title_id: &str,
    cache_dir: &Path,
    access: GraphicAudioAccess,
    bitrate: GraphicAudioBitrate,
    container: GraphicAudioContainer,
    magento_password: Option<&str>,
) -> Result<PlainFetch>
```

Download one title into `cache_dir` using host-injected credentials.

#### Errors

Returns an error when the operation fails.

### guest_fetch_title_rpc

```rust
pub async fn guest_fetch_title_rpc(
    access_base_url: &str,
    store_base_url: &str,
    params: &FetchTitleParams,
    access: GraphicAudioAccess,
    bitrate: GraphicAudioBitrate,
    container: GraphicAudioContainer,
    magento_password: Option<&str>,
) -> Result<PlainFetch>
```

RPC fetch: return the typed plain fetch result.

#### Errors

Returns an error when the operation fails.

### resolve_access_base_url

```rust
pub fn resolve_access_base_url(source_config: &Value) -> String
```

Resolve Access App API base URL from `[sources.graphicaudio]` JSON.

### resolve_store_base_url

```rust
pub fn resolve_store_base_url(source_config: &Value) -> String
```

Resolve Magento storefront base URL from `[sources.graphicaudio]` JSON.

### resolve_access

```rust
pub fn resolve_access(source_config: &Value) -> GraphicAudioAccess
```

Resolve download access path from granted `CONFIG`.

A missing `access` key uses the default. `BOOKCLERK_GA_ACCESS` is not read
here: after import, the instance document is the authority, and a document
that omits `access` must not fall back to the process environment.

### resolve_bitrate

```rust
pub fn resolve_bitrate(source_config: &Value) -> GraphicAudioBitrate
```

Resolve device bitrate preference from `[sources.graphicaudio]` JSON.

### resolve_container

```rust
pub fn resolve_container(source_config: &Value) -> GraphicAudioContainer
```

Resolve ZIP container preference from `[sources.graphicaudio]` JSON.

### source_id

```rust
pub fn source_id() -> &'static str
```

Source id returned in describe(); must match [`crate::source::ID`](index.md#graphicaudio_source_id).
