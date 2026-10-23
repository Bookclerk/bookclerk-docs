**bookclerk_plugin_source_graphicaudio > guest**

# Module: guest

## Contents

**Functions**

- [`guest_fetch_title`](#guest_fetch_title) - Download one title into `cache_dir` using host-injected credentials.
- [`guest_fetch_title_rpc`](#guest_fetch_title_rpc) - RPC fetch: return the typed plain fetch result.
- [`guest_login`](#guest_login) - Login against GraphicAudio and return account metadata + credential JSON.
- [`guest_login_rpc`](#guest_login_rpc) - RPC login: build [`LoginOptions`](../bookclerk_source/types.md#bookclerk_sourcetypesloginoptions) from typed params and return the ABI result.
- [`guest_scan`](#guest_scan) - Scan libraries for the credential blobs the host injected.
- [`guest_scan_rpc`](#guest_scan_rpc) - RPC scan: return the typed [`ScanSummary`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedscansummary) (host upserts books).
- [`resolve_access`](#resolve_access) - Resolve download access path from granted `CONFIG`.
- [`resolve_access_base_url`](#resolve_access_base_url) - Resolve Access App API base URL from `[sources.graphicaudio]` JSON.
- [`resolve_bitrate`](#resolve_bitrate) - Resolve device bitrate preference from `[sources.graphicaudio]` JSON.
- [`resolve_container`](#resolve_container) - Resolve ZIP container preference from `[sources.graphicaudio]` JSON.
- [`resolve_store_base_url`](#resolve_store_base_url) - Resolve Magento storefront base URL from `[sources.graphicaudio]` JSON.
- [`source_id`](#source_id) - Source id returned in describe(); must match [`crate::source::ID`](source.md#bookclerk_plugin_source_graphicaudiosourceid).

---

## bookclerk_plugin_source_graphicaudio::guest::guest_fetch_title

*Function*

Download one title into `cache_dir` using host-injected credentials.

# Errors

Returns an error when the operation fails.

```rust
fn guest_fetch_title(access_base_url: &str, store_base_url: &str, credentials: &serde_json::Value, title_id: &str, cache_dir: &std::path::Path, access: crate::options::GraphicAudioAccess, bitrate: crate::options::GraphicAudioBitrate, container: crate::options::GraphicAudioContainer, magento_password: Option<&str>) -> crate::error::Result<bookclerk_source::PlainFetch>
```



## bookclerk_plugin_source_graphicaudio::guest::guest_fetch_title_rpc

*Function*

RPC fetch: return the typed plain fetch result.

# Errors

Returns an error when the operation fails.

```rust
fn guest_fetch_title_rpc(access_base_url: &str, store_base_url: &str, params: &bookclerk_plugin_sdk::FetchTitleParams, access: crate::options::GraphicAudioAccess, bitrate: crate::options::GraphicAudioBitrate, container: crate::options::GraphicAudioContainer, magento_password: Option<&str>) -> crate::error::Result<bookclerk_plugin_sdk::PlainFetch>
```



## bookclerk_plugin_source_graphicaudio::guest::guest_login

*Function*

Login against GraphicAudio and return account metadata + credential JSON.

- `access=web|zip`: Magento customer login only (no Access App device slot).
- `access=device`: Access App `activation/login` (registers a device).

Does not write to `encrypted_secrets` — the host seals `credentials`.

# Errors

Returns an error when the operation fails.

```rust
fn guest_login(access_base_url: &str, store_base_url: &str, access: crate::options::GraphicAudioAccess, opts: bookclerk_source::LoginOptions) -> crate::error::Result<(String, String, Option<String>, bool, serde_json::Value)>
```



## bookclerk_plugin_source_graphicaudio::guest::guest_login_rpc

*Function*

RPC login: build [`LoginOptions`](../bookclerk_source/types.md#bookclerk_sourcetypesloginoptions) from typed params and return the ABI result.

# Errors

Returns an error when the operation fails.

```rust
fn guest_login_rpc(access_base_url: &str, store_base_url: &str, access: crate::options::GraphicAudioAccess, params: bookclerk_plugin_sdk::LoginParams) -> crate::error::Result<bookclerk_plugin_sdk::LoginResult>
```



## bookclerk_plugin_source_graphicaudio::guest::guest_scan

*Function*

Scan libraries for the credential blobs the host injected.

`magento_password` is required for Magento web/zip listing unless a device
token is present on the credentials (legacy Access App fallback).

# Errors

Returns an error when the operation fails.

```rust
fn guest_scan(access_base_url: &str, store_base_url: &str, access: crate::options::GraphicAudioAccess, magento_password: Option<&str>, credentials: &std::collections::BTreeMap<String, serde_json::Value>, account_filter: &[String], include_samples: bool) -> crate::error::Result<(Vec<bookclerk_library::NewBook>, usize, u32)>
```



## bookclerk_plugin_source_graphicaudio::guest::guest_scan_rpc

*Function*

RPC scan: return the typed [`ScanSummary`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedscansummary) (host upserts books).

# Errors

Returns an error when the operation fails.

```rust
fn guest_scan_rpc(access_base_url: &str, store_base_url: &str, access: crate::options::GraphicAudioAccess, magento_password: Option<&str>, params: &bookclerk_plugin_sdk::ScanParams) -> crate::error::Result<bookclerk_plugin_sdk::ScanSummary>
```



## bookclerk_plugin_source_graphicaudio::guest::resolve_access

*Function*

Resolve download access path from granted `CONFIG`.

A missing `access` key uses the default. `BOOKCLERK_GA_ACCESS` is not read
here: after import, the instance document is the authority, and a document
that omits `access` must not fall back to the process environment.

```rust
fn resolve_access(source_config: &serde_json::Value) -> crate::options::GraphicAudioAccess
```



## bookclerk_plugin_source_graphicaudio::guest::resolve_access_base_url

*Function*

Resolve Access App API base URL from `[sources.graphicaudio]` JSON.

```rust
fn resolve_access_base_url(source_config: &serde_json::Value) -> String
```



## bookclerk_plugin_source_graphicaudio::guest::resolve_bitrate

*Function*

Resolve device bitrate preference from `[sources.graphicaudio]` JSON.

```rust
fn resolve_bitrate(source_config: &serde_json::Value) -> crate::options::GraphicAudioBitrate
```



## bookclerk_plugin_source_graphicaudio::guest::resolve_container

*Function*

Resolve ZIP container preference from `[sources.graphicaudio]` JSON.

```rust
fn resolve_container(source_config: &serde_json::Value) -> crate::options::GraphicAudioContainer
```



## bookclerk_plugin_source_graphicaudio::guest::resolve_store_base_url

*Function*

Resolve Magento storefront base URL from `[sources.graphicaudio]` JSON.

```rust
fn resolve_store_base_url(source_config: &serde_json::Value) -> String
```



## bookclerk_plugin_source_graphicaudio::guest::source_id

*Function*

Source id returned in describe(); must match [`crate::source::ID`](source.md#bookclerk_plugin_source_graphicaudiosourceid).

```rust
fn source_id() -> &'static str
```



