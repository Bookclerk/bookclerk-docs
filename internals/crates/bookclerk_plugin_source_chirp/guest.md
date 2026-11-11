# bookclerk_plugin_source_chirp::guest

Guest-process helpers for the external Chirp source plugin.

These APIs do **not** open the library DB — the host seals credentials and
upserts scan rows via `bookclerk_plugin_host::ExternalSource`. Conversions
between storefront types and the typed ABI live in [`bookclerk_storefront::abi`](../bookclerk_storefront/abi.md).

| Item | Kind | Summary |
| --- | --- | --- |
| [`guest_login`](#guest_login) | function | Login against Chirp and return account metadata + credential JSON. |
| [`guest_login_rpc`](#guest_login_rpc) | function | RPC login: build `LoginOptions` from typed params and return the ABI result. |
| [`guest_scan_page`](#guest_scan_page) | function | RPC scan: one page of one account's full pass (the host upserts books). |
| [`guest_fetch_title`](#guest_fetch_title) | function | Download one title into `cache_dir` using host-injected credentials. |
| [`guest_fetch_title_rpc`](#guest_fetch_title_rpc) | function | RPC fetch: return the typed plain fetch result. |
| [`resolve_graphql_url`](#resolve_graphql_url) | function | Resolve GraphQL endpoint (tests may override via the plugin settings document). |
| [`source_id`](#source_id) | function | Source id returned in describe(); must match `crate::source::ID`. |

## Functions

### guest_login

```rust
pub async fn guest_login(
    graphql_url: &str,
    opts: LoginOptions,
) -> Result<(String, String, Option<String>, bool, Value)>
```

Login against Chirp and return account metadata + credential JSON.

Does not write to `encrypted_secrets` — the host seals `credentials`.

#### Errors

Returns an error when the operation fails.

### guest_login_rpc

```rust
pub async fn guest_login_rpc(
    graphql_url: &str,
    params: LoginParams,
) -> Result<LoginResult>
```

RPC login: build [`LoginOptions`](../bookclerk_storefront/README.md#loginoptions) from typed params and return the ABI result.

#### Errors

Returns an error when the operation fails.

### guest_scan_page

```rust
pub async fn guest_scan_page(
    graphql_url: &str,
    params: &ScanParams,
    page: u32,
) -> Result<ScanPage>
```

RPC scan: one page of one account's full pass (the host upserts books).

Chirp has no delta API, so every pass lists the whole library and ends
`complete`.

#### Arguments

* `graphql_url` - Chirp GraphQL endpoint.
* `params` - The pass's account, credential, page size and cursor.
* `page` - The store page the cursor names
  ([`bookclerk_storefront::abi::page_from_cursor`](../bookclerk_storefront/abi.md#page_from_cursor)).

#### Errors

Returns an error when the credential is missing or invalid, or the page
request fails.

### guest_fetch_title

```rust
pub async fn guest_fetch_title(
    graphql_url: &str,
    credentials: &Value,
    title_id: &str,
    cache_dir: &Path,
) -> Result<PlainFetch>
```

Download one title into `cache_dir` using host-injected credentials.

#### Errors

Returns an error when the operation fails.

### guest_fetch_title_rpc

```rust
pub async fn guest_fetch_title_rpc(
    graphql_url: &str,
    params: &FetchTitleParams,
) -> Result<PlainFetch>
```

RPC fetch: return the typed plain fetch result.

#### Errors

Returns an error when the operation fails.

### resolve_graphql_url

```rust
pub fn resolve_graphql_url(source_config: &Value) -> String
```

Resolve GraphQL endpoint (tests may override via the plugin settings document).

### source_id

```rust
pub fn source_id() -> &'static str
```

Source id returned in describe(); must match [`crate::source::ID`](README.md#chirp_source_id).
