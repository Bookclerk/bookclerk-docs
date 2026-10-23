# bookclerk_plugin_integration_audiobookshelf

Audiobookshelf integration plugin: Workers RPC guest plus the
[`bookclerk_integrations::Integration`](../bookclerk_integrations/index.md#integration) trait used by the guest binary.

Hosts load this package as a staged external guest. Do not link this crate
into `bookclerk` / `bookclerkd` for in-process execution.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AbsApiClient`](#absapiclient) | struct | Thin ABS REST client using Bearer auth. |
| [`AbsLibrary`](#abslibrary) | struct | Audiobookshelf library. |
| [`AbsLibraryItem`](#abslibraryitem) | struct | Audiobookshelf library item. |
| [`AbsMediaProgress`](#absmediaprogress) | struct | Audiobookshelf media progress. |
| [`AbsUser`](#absuser) | struct | Audiobookshelf user. |
| [`AbsUserDetail`](#absuserdetail) | struct | Full ABS user payload including listening progress. |
| [`AbsIntegration`](#absintegration) | struct | If required config (API key / base URL) is missing, the adapter stays registered, reports unhealthy, and refuses operational calls so the misconfiguration is visible. |
| [`AudiobookshelfConfig`](#audiobookshelfconfig) | struct | Audiobookshelf outbound integration. |
| [`brand_for_id`](#brand_for_id) | function | Brand for a provider id when it aliases this plugin. |
| [`matches_id`](#matches_id) | function | Whether `id` refers to this integration (`audiobookshelf` / `abs`). |
| [`collect_listening_snapshots`](#collect_listening_snapshots) | function | Collect listening-progress snapshots from ABS without writing the library DB. |
| [`sync_listening_progress`](#sync_listening_progress) | function | Pull ABS user media progress into the library DB. |
| [`oidc_client_templates`](#oidc_client_templates) | function | Bookclerk-as-IdP client templates declared by this plugin (`oidcClients`). |
| [`BRAND`](#brand) | constant | Canonical ABS brand for connect-portal buttons. |
| [`ID`](#id) | constant | Integration id used in describe() and `[integrations.audiobookshelf]`. |

## Re-exports

- [`guest::AbsGuestState`](guest.md#absgueststate)

## Modules

- [`guest`](guest.md): Guest-process helpers for the external Audiobookshelf integration plugin.

## Structs

### AbsApiClient

```rust
pub struct AbsApiClient {
    /* private fields */
}
```

Thin ABS REST client using Bearer auth.

#### AbsApiClient::new

```rust
pub fn new(base_url: impl Into<String>, api_key: impl Into<String>) -> Result<Self>
```

Build a client. `base_url` should be scheme+host (no trailing slash).

##### Errors

Returns an error when the operation fails.

#### AbsApiClient::base_url

```rust
pub fn base_url(&self) -> &str
```

Base URL.

#### AbsApiClient::authorize

```rust
pub async fn authorize(&self) -> Result<AuthorizeResponse>
```

`POST /api/authorize` — validate API token / key.

##### Errors

Returns an error when the operation fails.

#### AbsApiClient::login

```rust
pub async fn login(&self, username: &str, password: &str) -> Result<LoginResponse>
```

Username/password login (`POST /login`). Password is never logged.

##### Errors

Returns an error when the operation fails.

#### AbsApiClient::authenticate_user

```rust
pub async fn authenticate_user(
    &self,
    username: &str,
    password: &str,
) -> Result<ExternalUser>
```

Map a successful login into an [`ExternalUser`](../bookclerk_integrations/index.md#externaluser).

##### Errors

Returns an error when the operation fails.

#### AbsApiClient::get_user

```rust
pub async fn get_user(&self, user_id: &str) -> Result<AbsUserDetail>
```

`GET /api/users/{id}` — full user including `mediaProgress`.

##### Errors

Returns an error when the operation fails.

#### AbsApiClient::get_library_item

```rust
pub async fn get_library_item(&self, item_id: &str) -> Result<AbsLibraryItem>
```

`GET /api/items/{id}` — library item metadata for matching.

##### Errors

Returns an error when the operation fails.

#### AbsApiClient::list_libraries

```rust
pub async fn list_libraries(&self) -> Result<Vec<AbsLibrary>>
```

`GET /api/libraries`

##### Errors

Returns an error when the operation fails.

#### AbsApiClient::scan_library

```rust
pub async fn scan_library(&self, library_id: &str, force: bool) -> Result<()>
```

`POST /api/libraries/{id}/scan`

##### Errors

Returns an error when the operation fails.

#### AbsApiClient::list_users

```rust
pub async fn list_users(&self) -> Result<Vec<AbsUser>>
```

`GET /api/users` (admin).

##### Errors

Returns an error when the operation fails.

#### AbsApiClient::search_library

```rust
pub async fn search_library(&self, library_id: &str, q: &str) -> Result<Value>
```

`GET /api/libraries/{id}/search?q=`

##### Errors

Returns an error when the operation fails.

Implements: `Clone`

### AbsLibrary

```rust
pub struct AbsLibrary {
    pub id: String,
    pub name: String,
    pub media_type: Option<String>,
}
```

Audiobookshelf library.

**Fields**

- `id`: Identifier.
- `name`: Display or configuration name.
- `media_type`: Media type.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AbsLibraryItem

```rust
pub struct AbsLibraryItem {
    pub id: String,
    pub media: Option<AbsItemMedia>,
}
```

Audiobookshelf library item.

**Fields**

- `id`: Identifier.
- `media`: Media.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AbsMediaProgress

```rust
pub struct AbsMediaProgress {
    pub id: String,
    pub library_item_id: String,
    pub duration: Option<f64>,
    pub progress: Option<f64>,
    pub current_time: Option<f64>,
    pub is_finished: bool,
    pub last_update: Option<i64>,
}
```

Audiobookshelf media progress.

**Fields**

- `id`: Identifier.
- `library_item_id`: Library item Identifier.
- `duration`: Duration.
- `progress`: Progress.
- `current_time`: Current time.
- `is_finished`: Is finished.
- `last_update`: Last update.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AbsUser

```rust
pub struct AbsUser {
    pub id: String,
    pub username: String,
    pub token: Option<String>,
    pub typ: Option<String>,
    pub user_type: Option<String>,
}
```

Audiobookshelf user.

**Fields**

- `id`: Identifier.
- `username`: Username.
- `token`: Token.
- `typ`: Typ.
- `user_type`: User type.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AbsUserDetail

```rust
pub struct AbsUserDetail {
    pub id: String,
    pub username: String,
    pub media_progress: Vec<AbsMediaProgress>,
}
```

Full ABS user payload including listening progress.

**Fields**

- `id`: Identifier.
- `username`: Username.
- `media_progress`: Media progress.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AbsIntegration

```rust
pub struct AbsIntegration {
    /* private fields */
}
```

If required config (API key / base URL) is missing, the adapter stays
registered, reports unhealthy, and refuses operational calls so the
misconfiguration is visible.

#### AbsIntegration::new

```rust
pub fn new(config: AudiobookshelfConfig) -> Result<Self>
```

Build an ABS integration. Prefer [`Self::from_config`](#absintegrationfrom_config) — this only fails
on client construction bugs, not missing API keys.

##### Errors

Returns an error when the operation fails.

#### AbsIntegration::from_config

```rust
pub fn from_config(config: AudiobookshelfConfig) -> Self
```

Construct from config; missing credentials become a soft config error.

#### AbsIntegration::client

```rust
pub fn client(&self) -> Option<&AbsApiClient>
```

HTTP or RPC client used for outbound requests.

#### AbsIntegration::config

```rust
pub fn config(&self) -> &AudiobookshelfConfig
```

Config.

#### AbsIntegration::config_error

```rust
pub fn config_error(&self) -> Option<&str>
```

Config error.

#### AbsIntegration::scan_now

```rust
pub async fn scan_now(&self, force: bool) -> Result<()>
```

Trigger a library scan (optional force).

##### Errors

Returns an error when the operation fails.

Implements: `Integration`

### AudiobookshelfConfig

```rust
pub struct AudiobookshelfConfig {
    pub base_url: String,
    pub api_key: Option<String>,
    pub library_id: Option<String>,
    pub watch_users: bool,
    pub notify_scan_on_acquire: bool,
    pub allow_credential_login: bool,
}
```

Audiobookshelf outbound integration.

The guest's `CONFIG` / `SECRETS`, as declared by `[[settings]]` in
`plugin.toml` and resolved by the host from the plugin instance document.

**Fields**

- `base_url`: Audiobookshelf server origin without trailing slash (e.g. `https://abs.example`).
- `api_key`: ABS API token (a `secret` setting; delivered in `SECRETS`).
- `library_id`: Target ABS library UUID when the server has multiple libraries.
- `watch_users`: When true, poll ABS users for listening-progress sync.
- `notify_scan_on_acquire`: When true, ask ABS to rescan after a successful acquire.
- `allow_credential_login`: When true, allow signing in to the Bookclerk portal with ABS credentials.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### brand_for_id

```rust
pub fn brand_for_id(id: &str) -> Option<Brand>
```

Brand for a provider id when it aliases this plugin.

### matches_id

```rust
pub fn matches_id(id: &str) -> bool
```

Whether `id` refers to this integration (`audiobookshelf` / `abs`).

### collect_listening_snapshots

```rust
pub async fn collect_listening_snapshots(
    client: &AbsApiClient,
) -> Result<Vec<ListeningProgressSnapshot>>
```

Collect listening-progress snapshots from ABS without writing the library DB.

Used by both the in-process sync path and the external plugin guest RPC.

#### Errors

Returns an error when the operation fails.

### sync_listening_progress

```rust
pub async fn sync_listening_progress(
    library: &LibraryStore,
    client: &AbsApiClient,
) -> Result<usize>
```

Pull ABS user media progress into the library DB.

Best-effort matches rows to `book_uuid` / `work_id` via ASIN, ISBN, or title.
Prefer calling this through `bookclerk_integrations::Integration::sync_listening_progress`
on the registered ABS adapter rather than from host binaries.

#### Errors

Returns an error when the operation fails.

### oidc_client_templates

```rust
pub fn oidc_client_templates() -> Vec<OidcClientTemplate>
```

Bookclerk-as-IdP client templates declared by this plugin (`oidcClients`).

## Constants

### BRAND

```rust
pub const BRAND: Brand = _;
```

Canonical ABS brand for connect-portal buttons.

### ID

```rust
pub const ID: &str = "audiobookshelf";
```

Integration id used in describe() and `[integrations.audiobookshelf]`.
