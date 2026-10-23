**bookclerk_plugin_source_chirp > guest**

# Module: guest

## Contents

**Functions**

- [`guest_fetch_title`](#guest_fetch_title) - Download one title into `cache_dir` using host-injected credentials.
- [`guest_fetch_title_rpc`](#guest_fetch_title_rpc) - RPC fetch: return the typed plain fetch result.
- [`guest_login`](#guest_login) - Login against Chirp and return account metadata + credential JSON.
- [`guest_login_rpc`](#guest_login_rpc) - RPC login: build [`LoginOptions`](../bookclerk_source/types.md#bookclerk_sourcetypesloginoptions) from typed params and return the ABI result.
- [`guest_scan`](#guest_scan) - Scan libraries for the credential blobs the host injected.
- [`guest_scan_rpc`](#guest_scan_rpc) - RPC scan: return the typed [`ScanSummary`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedscansummary) (host upserts books).
- [`resolve_graphql_url`](#resolve_graphql_url) - Resolve GraphQL endpoint (tests may override via `[sources.chirp]` JSON).
- [`source_id`](#source_id) - Source id returned in describe(); must match [`crate::source::ID`](source.md#bookclerk_plugin_source_chirpsourceid).

---

## bookclerk_plugin_source_chirp::guest::guest_fetch_title

*Function*

Download one title into `cache_dir` using host-injected credentials.

# Errors

Returns an error when the operation fails.

```rust
fn guest_fetch_title(graphql_url: &str, credentials: &serde_json::Value, title_id: &str, cache_dir: &std::path::Path) -> crate::error::Result<bookclerk_source::PlainFetch>
```



## bookclerk_plugin_source_chirp::guest::guest_fetch_title_rpc

*Function*

RPC fetch: return the typed plain fetch result.

# Errors

Returns an error when the operation fails.

```rust
fn guest_fetch_title_rpc(graphql_url: &str, params: &bookclerk_plugin_sdk::FetchTitleParams) -> crate::error::Result<bookclerk_plugin_sdk::PlainFetch>
```



## bookclerk_plugin_source_chirp::guest::guest_login

*Function*

Login against Chirp and return account metadata + credential JSON.

Does not write to `encrypted_secrets` — the host seals `credentials`.

# Errors

Returns an error when the operation fails.

```rust
fn guest_login(graphql_url: &str, opts: bookclerk_source::LoginOptions) -> crate::error::Result<(String, String, Option<String>, bool, serde_json::Value)>
```



## bookclerk_plugin_source_chirp::guest::guest_login_rpc

*Function*

RPC login: build [`LoginOptions`](../bookclerk_source/types.md#bookclerk_sourcetypesloginoptions) from typed params and return the ABI result.

# Errors

Returns an error when the operation fails.

```rust
fn guest_login_rpc(graphql_url: &str, params: bookclerk_plugin_sdk::LoginParams) -> crate::error::Result<bookclerk_plugin_sdk::LoginResult>
```



## bookclerk_plugin_source_chirp::guest::guest_scan

*Function*

Scan libraries for the credential blobs the host injected.

# Errors

Returns an error when the operation fails.

```rust
fn guest_scan(graphql_url: &str, credentials: &std::collections::BTreeMap<String, serde_json::Value>, account_filter: &[String], page_size: u32) -> crate::error::Result<(Vec<bookclerk_library::NewBook>, usize, u32)>
```



## bookclerk_plugin_source_chirp::guest::guest_scan_rpc

*Function*

RPC scan: return the typed [`ScanSummary`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedscansummary) (host upserts books).

# Errors

Returns an error when the operation fails.

```rust
fn guest_scan_rpc(graphql_url: &str, params: &bookclerk_plugin_sdk::ScanParams) -> crate::error::Result<bookclerk_plugin_sdk::ScanSummary>
```



## bookclerk_plugin_source_chirp::guest::resolve_graphql_url

*Function*

Resolve GraphQL endpoint (tests may override via `[sources.chirp]` JSON).

```rust
fn resolve_graphql_url(source_config: &serde_json::Value) -> String
```



## bookclerk_plugin_source_chirp::guest::source_id

*Function*

Source id returned in describe(); must match [`crate::source::ID`](source.md#bookclerk_plugin_source_chirpsourceid).

```rust
fn source_id() -> &'static str
```



