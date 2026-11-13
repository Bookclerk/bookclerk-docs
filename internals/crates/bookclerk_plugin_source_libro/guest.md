# bookclerk_plugin_source_libro::guest

Guest-process helpers for the external Libro.fm source plugin.

These APIs do **not** open the library DB — the host seals credentials and
upserts scan rows via `bookclerk_plugin_host::ExternalSource`. Conversions
between storefront types and the typed ABI live in [`bookclerk_storefront::abi`](../bookclerk_storefront/abi.md).

| Item | Kind | Summary |
| --- | --- | --- |
| [`guest_login`](#guest_login) | function | Login against Libro.fm and return account metadata + credential JSON. |
| [`guest_login_rpc`](#guest_login_rpc) | function | RPC login: build `LoginOptions` from typed params and return the ABI result. |
| [`guest_scan_page`](#guest_scan_page) | function | RPC scan: one page of one account's full pass (the host upserts books). |
| [`guest_fetch_title`](#guest_fetch_title) | function | Download one title into `cache_dir` using host-injected credentials. |
| [`guest_fetch_title_rpc`](#guest_fetch_title_rpc) | function | RPC fetch: return the typed plain fetch result. |
| [`resolve_base_url`](#resolve_base_url) | function | Resolve API base URL (tests may override). |
| [`resolve_container`](#resolve_container) | function | Resolve download container from the plugin settings document. |
| [`source_id`](#source_id) | function | Source id returned in describe(); must match `crate::source::ID`. |

## Functions

### guest_login

```rust
pub async fn guest_login(
    base_url: &str,
    opts: LoginOptions,
) -> Result<(String, String, Option<String>, bool, Value)>
```

Login against Libro.fm and return account metadata + credential JSON.

Does not write to `encrypted_secrets` — the host seals `credentials`.

#### Errors

Returns an error when the operation fails.

### guest_login_rpc

```rust
pub async fn guest_login_rpc(base_url: &str, params: LoginParams) -> Result<LoginResult>
```

RPC login: build [`LoginOptions`](../bookclerk_storefront/README.md#loginoptions) from typed params and return the ABI result.

#### Errors

Returns an error when the operation fails.

### guest_scan_page

```rust
pub async fn guest_scan_page(
    base_url: &str,
    params: &ScanParams,
) -> Result<ScanPage, PluginError>
```

RPC scan: one page of one account's full pass (the host upserts books).

Libro.fm has no delta API, so every pass lists the whole library and
ends `complete`. The SDK's [`scan_page`](../../../plugins/sdk/rust/README.md#scan_page) helper pages the store's
numbered library pages into scan replies.

#### Arguments

* `base_url` - Libro.fm API base URL.
* `params` - The pass's account, credential and cursor.

#### Errors

Returns `invalid_cursor` for a cursor this plugin did not write, and
`internal` when the credential is missing or invalid or the page request
fails.

### guest_fetch_title

```rust
pub async fn guest_fetch_title(
    base_url: &str,
    credentials: &Value,
    title_id: &str,
    cache_dir: &Path,
    container: LibroContainer,
) -> Result<PlainFetch>
```

Download one title into `cache_dir` using host-injected credentials.

#### Errors

Returns an error when the operation fails.

### guest_fetch_title_rpc

```rust
pub async fn guest_fetch_title_rpc(
    base_url: &str,
    params: &FetchTitleParams,
    container: LibroContainer,
) -> Result<PlainFetch>
```

RPC fetch: return the typed plain fetch result.

#### Errors

Returns an error when the operation fails.

### resolve_base_url

```rust
pub fn resolve_base_url(source_config: &Value) -> String
```

Resolve API base URL (tests may override).

### resolve_container

```rust
pub fn resolve_container(source_config: &Value) -> LibroContainer
```

Resolve download container from the plugin settings document.

### source_id

```rust
pub fn source_id() -> &'static str
```

Source id returned in describe(); must match [`crate::source::ID`](README.md#libro_source_id).
