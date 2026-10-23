**bookclerk_plugin_source_libro > guest**

# Module: guest

## Contents

**Functions**

- [`guest_fetch_title`](#guest_fetch_title) - Download one title into `cache_dir` using host-injected credentials.
- [`guest_fetch_title_rpc`](#guest_fetch_title_rpc) - RPC fetch: return the typed plain fetch result.
- [`guest_login`](#guest_login) - Login against Libro.fm and return account metadata + credential JSON.
- [`guest_login_rpc`](#guest_login_rpc) - RPC login: build [`LoginOptions`](../bookclerk_source/types.md#bookclerk_sourcetypesloginoptions) from typed params and return the ABI result.
- [`guest_scan`](#guest_scan) - Scan libraries for the credential blobs the host injected.
- [`guest_scan_rpc`](#guest_scan_rpc) - RPC scan: return the typed [`ScanSummary`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedscansummary) (host upserts books).
- [`resolve_base_url`](#resolve_base_url) - Resolve API base URL (tests may override).
- [`resolve_container`](#resolve_container) - Resolve download container from `[sources.libro]` JSON.
- [`source_id`](#source_id) - Source id returned in describe(); must match [`crate::source::ID`](source.md#bookclerk_plugin_source_librosourceid).

---

## bookclerk_plugin_source_libro::guest::guest_fetch_title

*Function*

Download one title into `cache_dir` using host-injected credentials.

# Errors

Returns an error when the operation fails.

```rust
fn guest_fetch_title(base_url: &str, credentials: &serde_json::Value, title_id: &str, cache_dir: &std::path::Path, container: crate::container::LibroContainer) -> crate::error::Result<bookclerk_source::PlainFetch>
```



## bookclerk_plugin_source_libro::guest::guest_fetch_title_rpc

*Function*

RPC fetch: return the typed plain fetch result.

# Errors

Returns an error when the operation fails.

```rust
fn guest_fetch_title_rpc(base_url: &str, params: &bookclerk_plugin_sdk::FetchTitleParams, container: crate::container::LibroContainer) -> crate::error::Result<bookclerk_plugin_sdk::PlainFetch>
```



## bookclerk_plugin_source_libro::guest::guest_login

*Function*

Login against Libro.fm and return account metadata + credential JSON.

Does not write to `encrypted_secrets` — the host seals `credentials`.

# Errors

Returns an error when the operation fails.

```rust
fn guest_login(base_url: &str, opts: bookclerk_source::LoginOptions) -> crate::error::Result<(String, String, Option<String>, bool, serde_json::Value)>
```



## bookclerk_plugin_source_libro::guest::guest_login_rpc

*Function*

RPC login: build [`LoginOptions`](../bookclerk_source/types.md#bookclerk_sourcetypesloginoptions) from typed params and return the ABI result.

# Errors

Returns an error when the operation fails.

```rust
fn guest_login_rpc(base_url: &str, params: bookclerk_plugin_sdk::LoginParams) -> crate::error::Result<bookclerk_plugin_sdk::LoginResult>
```



## bookclerk_plugin_source_libro::guest::guest_scan

*Function*

Scan libraries for the credential blobs the host injected.

# Errors

Returns an error when the operation fails.

```rust
fn guest_scan(base_url: &str, credentials: &std::collections::BTreeMap<String, serde_json::Value>, account_filter: &[String]) -> crate::error::Result<(Vec<bookclerk_library::NewBook>, usize, u32)>
```



## bookclerk_plugin_source_libro::guest::guest_scan_rpc

*Function*

RPC scan: return the typed [`ScanSummary`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedscansummary) (host upserts books).

# Errors

Returns an error when the operation fails.

```rust
fn guest_scan_rpc(base_url: &str, params: &bookclerk_plugin_sdk::ScanParams) -> crate::error::Result<bookclerk_plugin_sdk::ScanSummary>
```



## bookclerk_plugin_source_libro::guest::resolve_base_url

*Function*

Resolve API base URL (tests may override).

```rust
fn resolve_base_url(source_config: &serde_json::Value) -> String
```



## bookclerk_plugin_source_libro::guest::resolve_container

*Function*

Resolve download container from `[sources.libro]` JSON.

```rust
fn resolve_container(source_config: &serde_json::Value) -> crate::container::LibroContainer
```



## bookclerk_plugin_source_libro::guest::source_id

*Function*

Source id returned in describe(); must match [`crate::source::ID`](source.md#bookclerk_plugin_source_librosourceid).

```rust
fn source_id() -> &'static str
```



