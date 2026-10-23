**bookclerk_plugin_source_audible > guest**

# Module: guest

## Contents

**Functions**

- [`credentials_json_from_auth`](#credentials_json_from_auth) - Build credential JSON from an authenticator (optional Widevine CDM bytes).
- [`guest_fetch_title`](#guest_fetch_title) - Download + decrypt one title; return plain paths (never encrypted on the wire).
- [`guest_login_complete`](#guest_login_complete) - Await a pending OAuth session and return account + opaque credential bytes.
- [`guest_login_start`](#guest_login_start) - Start LoginServer OAuth; return `(session_id, browser_url)`.
- [`guest_scan`](#guest_scan) - Scan libraries for host-injected credential blobs.

---

## bookclerk_plugin_source_audible::guest::credentials_json_from_auth

*Function*

Build credential JSON from an authenticator (optional Widevine CDM bytes).

# Errors

Returns an error when the operation fails.

```rust
fn credentials_json_from_auth(auth: &audible_rs::auth::Authenticator, widevine: Option<&[u8]>) -> crate::error::Result<serde_json::Value>
```



## bookclerk_plugin_source_audible::guest::guest_fetch_title

*Function*

Download + decrypt one title; return plain paths (never encrypted on the wire).

`fetch` carries the host fetch knobs from the ABI `FetchTitleParams.fetch`
field; packaging knobs use guest defaults. Plugin bitrate from
`source_config` overlays `quality` (same as in-process).

# Errors

Returns an error when the operation fails.

```rust
fn guest_fetch_title(credentials: &serde_json::Value, title_id: &str, cache_dir: &std::path::Path, source_config: &serde_json::Value, fetch: &bookclerk_plugin_sdk::FetchOptions) -> crate::error::Result<bookclerk_plugin_sdk::PlainFetch>
```



## bookclerk_plugin_source_audible::guest::guest_login_complete

*Function*

Await a pending OAuth session and return account + opaque credential bytes.

# Errors

Returns an error when the operation fails.

```rust
fn guest_login_complete(session_id: &str) -> crate::error::Result<bookclerk_plugin_sdk::LoginResult>
```



## bookclerk_plugin_source_audible::guest::guest_login_start

*Function*

Start LoginServer OAuth; return `(session_id, browser_url)`.

Completes later via [`guest_login_complete`](guest.md#bookclerk_plugin_source_audibleguestguest_login_complete).

When the host sets `callback_ipc` + `callback_public_base`, the guest does
**not** bind TCP — it connects to the host IPC tunnel and serves LoginServer
on forwarded streams (required under Windows AppContainer).

# Errors

Returns an error when the operation fails.

```rust
fn guest_login_start(params: &bookclerk_plugin_sdk::LoginParams) -> crate::error::Result<(String, String)>
```



## bookclerk_plugin_source_audible::guest::guest_scan

*Function*

Scan libraries for host-injected credential blobs.

# Errors

Returns an error when the operation fails.

```rust
fn guest_scan(credentials: &std::collections::BTreeMap<String, serde_json::Value>, account_filter: &[String], page_size: u32, import_episodes: bool, import_plus_titles: bool) -> crate::error::Result<bookclerk_plugin_sdk::ScanSummary>
```



