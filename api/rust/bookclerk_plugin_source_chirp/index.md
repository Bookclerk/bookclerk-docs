# bookclerk_plugin_source_chirp

Chirp content source: GraphQL auth, library sync, and DRM-free download.

Uses the reverse-engineered Android Mockingjay GraphQL API
(`https://api.chirpbooks.com/api/graphql`).

| Item | Kind | Summary |
| --- | --- | --- |
| [`ChirpAuthFile`](#chirpauthfile) | struct | Credential envelope for one Chirp account. |
| [`Audiobook`](#audiobook) | struct | Chirp audiobook metadata (+ optional tracks). |
| [`AuthorCatalog`](#authorcatalog) | struct | Author summary expansion result. |
| [`CatalogAudiobook`](#catalogaudiobook) | struct | Catalog-oriented audiobook (search / related / series). |
| [`CatalogAuthor`](#catalogauthor) | struct | Chirp author metadata. |
| [`CatalogSeries`](#catalogseries) | struct | Chirp series metadata. |
| [`ChirpClient`](#chirpclient) | struct | Authenticated Chirp GraphQL helper. |
| [`ChirpProductPricing`](#chirpproductpricing) | struct | Live Chirp storefront pricing (`audiobook.currentProduct`). |
| [`LibraryItem`](#libraryitem) | struct | One `currentUserAudiobooks` row. |
| [`RelatedCatalog`](#relatedcatalog) | struct | Related-audiobook expansion result. |
| [`SeriesCatalog`](#seriescatalog) | struct | Series page expansion result. |
| [`Track`](#track) | struct | One downloadable / playable track. |
| [`TypeaheadCatalog`](#typeaheadcatalog) | struct | Typeahead hits. |
| [`ChirpSource`](#chirpsource) | struct | Chirp storefront adapter implementing `ContentSource`. |
| [`ScanOptions`](#scanoptions) | struct | Options for a Chirp library scan. |
| [`ChirpError`](#chirperror) | enum | Failures from Chirp GraphQL sign-in, library listing, or DRM-free download. |
| [`chirp_slug_candidates`](#chirp_slug_candidates) | function | Slug guesses for Chirp `series(slug:)` / `author(slug:)`. |
| [`fetch_title_materials`](#fetch_title_materials) | function | Fetch one audiobook id into `cache_dir` via AndroidSingleAudiobook track URLs. |
| [`audiobook_to_new_book`](#audiobook_to_new_book) | function | Map a Chirp audiobook to a `NewBook` row. |
| [`collect_account_books`](#collect_account_books) | function | Fetch all library pages for one client and collect books (no DB writes). |
| [`scan_account_into_library`](#scan_account_into_library) | function | Fetch all library pages for one client and upsert books. |
| [`scan_library`](#scan_library) | function | Sync Chirp libraries for configured accounts into `library`. |
| [`Result`](#result) | type alias | Result alias for Chirp auth, sync, and download operations. |
| [`DEFAULT_GRAPHQL_URL`](#default_graphql_url) | constant | Android Mockingjay GraphQL endpoint. |
| [`CHIRP_SOURCE_ID`](#chirp_source_id) | constant | Describe / config id for this store (`chirp` in `[sources.chirp]`). |

## Re-exports

- [`db::delete_auth_from_db`](db.md#delete_auth_from_db)
- [`db::list_auth_from_db`](db.md#list_auth_from_db)
- [`db::load_auth_from_db`](db.md#load_auth_from_db)
- [`db::save_auth_to_db`](db.md#save_auth_to_db)
- [`guest::guest_fetch_title`](guest.md#guest_fetch_title)
- [`guest::guest_fetch_title_rpc`](guest.md#guest_fetch_title_rpc)
- [`guest::guest_login`](guest.md#guest_login)
- [`guest::guest_login_rpc`](guest.md#guest_login_rpc)
- [`guest::guest_scan`](guest.md#guest_scan)
- [`guest::guest_scan_rpc`](guest.md#guest_scan_rpc)
- [`guest::resolve_graphql_url`](guest.md#resolve_graphql_url)

## Modules

- [`db`](db.md): DB-backed credential storage for Chirp accounts.
- [`guest`](guest.md): Guest-process helpers for the external Chirp source plugin.

## Structs

### ChirpAuthFile

```rust
pub struct ChirpAuthFile {
    pub access_token: String,
    pub web_token: Option<String>,
    pub email: String,
    pub user_id: Option<String>,
    pub marketplace: String,
    pub label: Option<String>,
}
```

Credential envelope for one Chirp account.

Persisted as JSON in `encrypted_secrets` (see [`crate::db`](db.md)); never written
to disk under `Accounts/`.

**Fields**

- `access_token`: JWT from GraphQL `signIn` (`user.token`).
- `web_token`: Longer-lived web JWT when present (`user.webToken`).
- `email`: Sign-in email used as fallback account id.
- `user_id`: Chirp user id from `signIn` when present.
- `marketplace`: Marketplace / locale hint stored for Bookclerk (defaults to `us`).
- `label`: Optional operator-facing account label in Accounts UI.

#### ChirpAuthFile::account_id

```rust
pub fn account_id(&self) -> &str
```

Stable account id: prefer Chirp user id, else email.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### Audiobook

```rust
pub struct Audiobook {
    pub id: String,
    pub display_title: Option<String>,
    pub sub_title: Option<String>,
    pub display_authors: Option<String>,
    pub display_narrators: Option<String>,
    pub duration_ms: Option<u64>,
    pub abridged: Option<bool>,
    pub publisher: Option<String>,
    pub released_on: Option<String>,
    pub cover_url: Option<String>,
    pub description: Option<String>,
    pub tracks: Vec<Track>,
}
```

Chirp audiobook metadata (+ optional tracks).

**Fields**

- `id`: Identifier.
- `display_title`: Display title.
- `sub_title`: Sub title.
- `display_authors`: Display authors.
- `display_narrators`: Display narrators.
- `duration_ms`: Duration ms.
- `abridged`: Abridged.
- `publisher`: Publisher.
- `released_on`: Released on.
- `cover_url`: Cover URL.
- `description`: Description.
- `tracks`: Tracks.

Implements: `Clone`, `Debug`, `Deserialize<'de>`

### AuthorCatalog

```rust
pub struct AuthorCatalog {
    pub author: CatalogAuthor,
    pub audiobooks: Vec<CatalogAudiobook>,
}
```

Author summary expansion result.

**Fields**

- `author`: Author.
- `audiobooks`: Audiobooks.

Implements: `Clone`, `Debug`

### CatalogAudiobook

```rust
pub struct CatalogAudiobook {
    pub id: String,
    pub display_title: Option<String>,
    pub display_authors: Option<String>,
    pub display_narrators: Option<String>,
    pub url: Option<String>,
    pub cover_url: Option<String>,
    pub series_audiobook: Option<SeriesAudiobookRef>,
    pub sub_title: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub duration_ms: Option<u64>,
    pub released_on: Option<String>,
    pub language: Option<String>,
    pub abridged: Option<bool>,
    pub promoted_tags: Vec<CatalogTag>,
}
```

Catalog-oriented audiobook (search / related / series).

**Fields**

- `id`: Identifier.
- `display_title`: Display title.
- `display_authors`: Display authors.
- `display_narrators`: Display narrators.
- `url`: URL.
- `cover_url`: Cover URL.
- `series_audiobook`: Series audiobook.
- `sub_title`: Sub title.
- `description`: Description.
- `publisher`: Publisher.
- `duration_ms`: Duration ms.
- `released_on`: Released on.
- `language`: Chirp display language (`English`, `Spanish`, …).
- `abridged`: Abridged.
- `promoted_tags`: Storefront genre chips (`Thrillers`, `Crime Fiction & Mysteries`, …).

#### CatalogAudiobook::title

```rust
pub fn title(&self) -> String
```

Title.

#### CatalogAudiobook::series_name

```rust
pub fn series_name(&self) -> Option<String>
```

Series name.

Implements: `Clone`, `Debug`, `Deserialize<'de>`

### CatalogAuthor

```rust
pub struct CatalogAuthor {
    pub id: String,
    pub name: String,
    pub slug: String,
}
```

Chirp author metadata.

**Fields**

- `id`: Identifier.
- `name`: Display or configuration name.
- `slug`: Slug.

Implements: `Clone`, `Debug`, `Deserialize<'de>`

### CatalogSeries

```rust
pub struct CatalogSeries {
    pub id: String,
    pub name: String,
    pub slug: String,
}
```

Chirp series metadata.

**Fields**

- `id`: Identifier.
- `name`: Display or configuration name.
- `slug`: Slug.

Implements: `Clone`, `Debug`, `Deserialize<'de>`

### ChirpClient

```rust
pub struct ChirpClient {
    /* private fields */
}
```

Authenticated Chirp GraphQL helper.

#### ChirpClient::new

```rust
pub fn new(graphql_url: impl Into<String>) -> Self
```

Constructs a new instance with default or provided parameters.

#### ChirpClient::with_http

```rust
pub fn with_http(self, http: HttpClient) -> Self
```

With HTTP.

#### ChirpClient::with_token

```rust
pub fn with_token(self, token: impl Into<String>) -> Self
```

With token.

#### ChirpClient::access_token

```rust
pub fn access_token(&self) -> Option<&str>
```

Access token.

#### ChirpClient::graphql_url

```rust
pub fn graphql_url(&self) -> &str
```

Graphql URL.

#### ChirpClient::login

```rust
pub async fn login(&mut self, email: &str, password: &str) -> Result<SignInUser>
```

GraphQL `signIn` mutation.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::library_page

```rust
pub async fn library_page(&self, page: u32, page_size: u32) -> Result<Vec<LibraryItem>>
```

Paginated owned library (`currentUserAudiobooks`).

##### Errors

Returns an error when the operation fails.

#### ChirpClient::audiobook

```rust
pub async fn audiobook(&self, id: &str) -> Result<Audiobook>
```

Full audiobook with track `mediaUrl`s.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::audiobook_pricing

```rust
pub async fn audiobook_pricing(&self, id: &str) -> Result<Option<ChirpProductPricing>>
```

Live deal / list pricing for a Chirp audiobook (public GraphQL).

##### Errors

Returns an error when the operation fails.

#### ChirpClient::search_catalog

```rust
pub async fn search_catalog(
    &self,
    query: &str,
    page: u32,
    page_size: u32,
) -> Result<Vec<CatalogAudiobook>>
```

Catalog search (`audiobooks(query:)`) — no auth required.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::related_audiobooks

```rust
pub async fn related_audiobooks(&self, id: &str) -> Result<RelatedCatalog>
```

Related titles for a Chirp audiobook id — no auth required.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::series_catalog

```rust
pub async fn series_catalog(&self, slug: &str) -> Result<Option<SeriesCatalog>>
```

Series catalog by slug (`mistborn-audiobooks`, …) — no auth required.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::author_summary

```rust
pub async fn author_summary(&self, slug: &str) -> Result<Option<AuthorCatalog>>
```

Author summary titles by slug — no auth required.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::typeahead

```rust
pub async fn typeahead(&self, search_term: &str) -> Result<TypeaheadCatalog>
```

Typeahead for author slug resolution and quick title hits — no auth required.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::resolve_author_slug

```rust
pub async fn resolve_author_slug(&self, author_name: &str) -> Result<Option<String>>
```

Resolve a Chirp author slug via typeahead (exact name match only).

Deliberately no substring fallback — short needles like `"ann"` would
otherwise resolve to unrelated authors and pollute candidate expansion.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::resolve_series_catalog

```rust
pub async fn resolve_series_catalog(
    &self,
    series_name: &str,
) -> Result<Option<SeriesCatalog>>
```

Try common Chirp series slug forms derived from a series title.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::top_deals

```rust
pub async fn top_deals(&self, count: u32) -> Result<Vec<CatalogAudiobook>>
```

Current Chirp top deals — no auth required.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::free_deals

```rust
pub async fn free_deals(&self) -> Result<Vec<CatalogAudiobook>>
```

Current Chirp free deals — no auth required.

##### Errors

Returns an error when the operation fails.

#### ChirpClient::download_bytes

```rust
pub async fn download_bytes(&self, url: &str) -> Result<Bytes>
```

Download bytes from an absolute media URL.

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`, `Default`

### ChirpProductPricing

```rust
pub struct ChirpProductPricing {
    pub id: Option<String>,
    pub discount_price: String,
    pub discounted_price_cents: Option<i64>,
    pub listing_price: Option<String>,
    pub is_free_listing: bool,
    pub hot_deal: bool,
    pub purchase_url: Option<String>,
    pub salable_in_current_country: Option<bool>,
}
```

Live Chirp storefront pricing (`audiobook.currentProduct`).

**Fields**

- `id`: Identifier.
- `discount_price`: Discount price.
- `discounted_price_cents`: Discounted price cents.
- `listing_price`: Listing price.
- `is_free_listing`: Is free listing.
- `hot_deal`: Hot deal.
- `purchase_url`: Purchase URL.
- `salable_in_current_country`: Salable in current country.

Implements: `Clone`, `Debug`, `Deserialize<'de>`

### LibraryItem

```rust
pub struct LibraryItem {
    pub id: String,
    pub archived: Option<bool>,
    pub playable: Option<bool>,
    pub audiobook: Option<Audiobook>,
}
```

One `currentUserAudiobooks` row.

**Fields**

- `id`: Identifier.
- `archived`: Archived.
- `playable`: Playable.
- `audiobook`: Audiobook.

Implements: `Clone`, `Debug`, `Deserialize<'de>`

### RelatedCatalog

```rust
pub struct RelatedCatalog {
    pub related: Vec<CatalogAudiobook>,
    pub series: Option<CatalogSeries>,
}
```

Related-audiobook expansion result.

**Fields**

- `related`: Related.
- `series`: Series.

Implements: `Clone`, `Debug`

### SeriesCatalog

```rust
pub struct SeriesCatalog {
    pub series: CatalogSeries,
    pub audiobooks: Vec<CatalogAudiobook>,
}
```

Series page expansion result.

**Fields**

- `series`: Series.
- `audiobooks`: Audiobooks.

Implements: `Clone`, `Debug`

### Track

```rust
pub struct Track {
    pub id: String,
    pub media_url: Option<String>,
    pub chapter_number: Option<i64>,
    pub part_number: Option<i64>,
    pub duration_ms: Option<u64>,
    pub offset_from_book_start_ms: Option<u64>,
    pub display_name: Option<String>,
}
```

One downloadable / playable track.

**Fields**

- `id`: Identifier.
- `media_url`: Media URL.
- `chapter_number`: Chapter number.
- `part_number`: Part number.
- `duration_ms`: Duration ms.
- `offset_from_book_start_ms`: Offset from book start ms.
- `display_name`: Display name.

Implements: `Clone`, `Debug`, `Deserialize<'de>`

### TypeaheadCatalog

```rust
pub struct TypeaheadCatalog {
    pub audiobooks: Vec<CatalogAudiobook>,
    pub authors: Vec<CatalogAuthor>,
}
```

Typeahead hits.

**Fields**

- `audiobooks`: Audiobooks.
- `authors`: Authors.

Implements: `Clone`, `Debug`, `Default`

### ChirpSource

```rust
pub struct ChirpSource {
    /* private fields */
}
```

Chirp storefront adapter implementing [`ContentSource`](../bookclerk_source/index.md#contentsource).

Construct with [`ChirpSource::new`](#chirpsourcenew); use [`ChirpSource::with_graphql_url`](#chirpsourcewith_graphql_url)
for wiremock / staging.

#### ChirpSource::new

```rust
pub fn new() -> Self
```

Returns a Chirp source aimed at the production GraphQL endpoint.

#### ChirpSource::with_graphql_url

```rust
pub fn with_graphql_url(graphql_url: impl Into<String>) -> Self
```

Overrides the GraphQL endpoint (tests / staging).

#### ChirpSource::shared

```rust
pub fn shared() -> Arc<Self>
```

Arc-wrapped instance for [`bookclerk_source::SourceRegistry`](../bookclerk_source/index.md#sourceregistry).

#### ChirpSource::login_account

```rust
pub async fn login_account(
    &self,
    library: &SourceScope,
    opts: LoginOptions,
) -> Result<SourceAccount>
```

Logs in with email/password and seals credentials into `encrypted_secrets`.

##### Errors

Returns an error when the operation fails.

#### ChirpSource::delete_account

```rust
pub async fn delete_account(&self, library: &SourceScope, account_id: &str) -> Result<()>
```

Delete a Chirp account from the DB.

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `ContentSource`, `Debug`, `Default`

### ScanOptions

```rust
pub struct ScanOptions {
    pub accounts: Vec<String>,
    pub page_size: u32,
}
```

Options for a Chirp library scan.

**Fields**

- `accounts`: Accounts.
- `page_size`: Page size.

Implements: `Clone`, `Debug`, `Default`, `From<&ScanOptions>`

## Enums

### ChirpError

```rust
pub enum ChirpError {
    Auth(String),
    NoAccounts(String),
    AccountNotFound(String),
    Api(String),
    Download(String),
    Io(Error),
    Library(LibraryError),
    Other(Error),
}
```

Failures from Chirp GraphQL sign-in, library listing, or DRM-free download.

**Variants**

- `Auth`: GraphQL `signIn` or token use failed.
- `NoAccounts`: No Chirp accounts are configured (safe to skip in multi-source scan).
- `AccountNotFound`: Requested account id is not present in `encrypted_secrets`.
- `Api`: Upstream GraphQL / HTTP API error.
- `Download`: Title download or packaging failed.
- `Io`: Local filesystem I/O while reading or writing cache files.
- `Library`: Library database upsert / secret storage failure.
- `Other`: Unexpected failure wrapped from `anyhow`.

#### ChirpError::auth

```rust
pub fn auth(msg: impl Into<String>) -> Self
```

Builds an [`Self::Auth`](#chirperror) error from a displayable message.

#### ChirpError::no_accounts

```rust
pub fn no_accounts(msg: impl Into<String>) -> Self
```

Builds a [`Self::NoAccounts`](#chirperror) error (multi-source scan may skip this store).

#### ChirpError::api

```rust
pub fn api(msg: impl Into<String>) -> Self
```

Builds an [`Self::Api`](#chirperror) error from a displayable message.

#### ChirpError::download

```rust
pub fn download(msg: impl Into<String>) -> Self
```

Builds a [`Self::Download`](#chirperror) error from a displayable message.

Implements: `Debug`, `Display`, `Error`, `From<ChirpError>`, `From<Error>`, `From<LibraryError>`

## Functions

### chirp_slug_candidates

```rust
pub fn chirp_slug_candidates(name: &str) -> Vec<String>
```

Slug guesses for Chirp `series(slug:)` / `author(slug:)`.

### fetch_title_materials

```rust
pub async fn fetch_title_materials(
    client: &ChirpClient,
    audiobook_id: &str,
    cache_dir: &Path,
) -> Result<PlainFetch>
```

Fetch one audiobook id into `cache_dir` via AndroidSingleAudiobook track URLs.

#### Errors

Returns an error when the operation fails.

### audiobook_to_new_book

```rust
pub fn audiobook_to_new_book(
    book: &Audiobook,
    account_id: &str,
    marketplace: &str,
) -> NewBook
```

Map a Chirp audiobook to a [`NewBook`](../bookclerk_library/index.md#newbook) row.

### collect_account_books

```rust
pub async fn collect_account_books(
    client: &ChirpClient,
    account_id: &str,
    marketplace: &str,
    page_size: u32,
) -> Result<(Vec<NewBook>, u32)>
```

Fetch all library pages for one client and collect books (no DB writes).

#### Errors

Returns an error when the operation fails.

### scan_account_into_library

```rust
pub async fn scan_account_into_library(
    library: &SourceScope,
    client: &ChirpClient,
    account_id: &str,
    marketplace: &str,
    page_size: u32,
) -> Result<(usize, u32)>
```

Fetch all library pages for one client and upsert books.

#### Errors

Returns an error when the operation fails.

### scan_library

```rust
pub async fn scan_library(
    library: &SourceScope,
    options: ScanOptions,
    graphql_url: Option<&str>,
) -> Result<ScanSummary>
```

Sync Chirp libraries for configured accounts into `library`.

Accounts are resolved from `encrypted_secrets` (DB-backed); no
`Accounts/*.chirp.auth` files are read.

#### Errors

Returns an error when the operation fails.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, ChirpError>;
```

Result alias for Chirp auth, sync, and download operations.

## Constants

### DEFAULT_GRAPHQL_URL

```rust
pub const DEFAULT_GRAPHQL_URL: &str = "https://api.chirpbooks.com/api/graphql";
```

Android Mockingjay GraphQL endpoint.

### CHIRP_SOURCE_ID

```rust
pub const CHIRP_SOURCE_ID: &str = "chirp";
```

Describe / config id for this store (`chirp` in `[sources.chirp]`).
