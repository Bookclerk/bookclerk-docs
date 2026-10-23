# bookclerk_plugin_source_chirp::guest

Guest-process helpers for the external Chirp source plugin.

These APIs do **not** open the library DB — the host seals credentials and
upserts scan rows via `bookclerk_plugin_host::ExternalSource`. Conversions
between storefront types and the typed ABI live in [`bookclerk_source::abi`](../bookclerk_source/abi.md).

| Item | Kind | Summary |
| --- | --- | --- |
| [`guest_login`](#guest_login) | function | Login against Chirp and return account metadata + credential JSON. |
| [`guest_login_rpc`](#guest_login_rpc) | function | RPC login: build `LoginOptions` from typed params and return the ABI result. |
| [`guest_scan`](#guest_scan) | function | Scan libraries for the credential blobs the host injected. |
| [`guest_scan_rpc`](#guest_scan_rpc) | function | RPC scan: return the typed `ScanSummary` (host upserts books). |
| [`guest_fetch_title`](#guest_fetch_title) | function | Download one title into `cache_dir` using host-injected credentials. |
| [`guest_fetch_title_rpc`](#guest_fetch_title_rpc) | function | RPC fetch: return the typed plain fetch result. |
| [`resolve_graphql_url`](#resolve_graphql_url) | function | Resolve GraphQL endpoint (tests may override via `[sources.chirp]` JSON). |
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

RPC login: build [`LoginOptions`](../bookclerk_source/index.md#loginoptions) from typed params and return the ABI result.

#### Errors

Returns an error when the operation fails.

### guest_scan

```rust
pub async fn guest_scan(
    graphql_url: &str,
    credentials: &BTreeMap<String, Value>,
    account_filter: &[String],
    page_size: u32,
) -> Result<(Vec<NewBook>, usize, u32)>
```

Scan libraries for the credential blobs the host injected.

#### Errors

Returns an error when the operation fails.

### guest_scan_rpc

```rust
pub async fn guest_scan_rpc(graphql_url: &str, params: &ScanParams) -> Result<ScanSummary>
```

RPC scan: return the typed [`ScanSummary`](../bookclerk_plugin_abi/generated.md#scansummary) (host upserts books).

#### Errors

Returns an error when the operation fails.

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

Resolve GraphQL endpoint (tests may override via `[sources.chirp]` JSON).

### source_id

```rust
pub fn source_id() -> &'static str
```

Source id returned in describe(); must match [`crate::source::ID`](index.md#chirp_source_id).
