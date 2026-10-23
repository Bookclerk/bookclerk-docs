**bookclerk_plugin_source_libro > client**

# Module: client

## Contents

**Structs**

- [`Audiobook`](#audiobook) - Audiobook.
- [`AudiobookInfo`](#audiobookinfo) - Narrator / duration / part counts nested under a library audiobook.
- [`DownloadManifest`](#downloadmanifest) - Download manifest.
- [`DownloadPart`](#downloadpart) - Download part.
- [`Genre`](#genre) - Genre tag from a library-page audiobook (`name` may be missing).
- [`LibraryPage`](#librarypage) - Library page.
- [`LibroClient`](#libroclient) - Authenticated Libro.fm HTTP helper.
- [`ManifestTrack`](#manifesttrack) - Manifest track.
- [`PackagedM4b`](#packagedm4b) - Packaged m4b.
- [`TokenResponse`](#tokenresponse) - Token response.
- [`UserMetadata`](#usermetadata) - Per-user library flags (added date, finished, hidden).

**Enums**

- [`ManifestFormat`](#manifestformat) - Android `MediaFormat` id for `download-manifest?format=…`.

**Constants**

- [`APP_VER`](#app_ver) - Android app version header (`X-LibroFm-AppVer`).
- [`CLIENT_ID`](#client_id) - Optional OAuth `client_id`.
- [`DEFAULT_BASE_URL`](#default_base_url) - Default API origin.
- [`DOWNLOAD_MANIFEST_PATH`](#download_manifest_path) - Download manifest (`parts` + chapter `tracks`).
- [`LIBRARY_PATH`](#library_path) - Paginated library listing (`/api/vN/library` from the Android app prefix).
- [`OAUTH_TOKEN_PATH`](#oauth_token_path) - OAuth password-grant path.
- [`PACKAGED_M4B_PATH`](#packaged_m4b_path) - Packaged single-file M4B when Libro.fm offers it.
- [`USER_AGENT_VALUE`](#user_agent_value) - User-Agent matching the official Android HTTP stack.

---

## bookclerk_plugin_source_libro::client::APP_VER

*Constant*: `&str`

Android app version header (`X-LibroFm-AppVer`).

Keep in sync via `scripts/librofm-apk-probe/` / workflow `librofm-apk-probe.yml`.



## bookclerk_plugin_source_libro::client::Audiobook

*Struct*

Audiobook.

**Fields:**
- `isbn: String` - ISBN may arrive as string or number in JSON.
- `title: String` - Title.
- `authors: Option<Vec<String>>` - Authors.
- `cover_url: Option<String>` - Cover URL.
- `subtitle: Option<String>` - Subtitle.
- `publisher: Option<String>` - Publisher.
- `publication_date: Option<String>` - Publication date.
- `abridged: Option<bool>` - Abridged.
- `series: Option<String>` - Series.
- `series_num: Option<serde_json::Value>` - Series num.
- `genres: Option<Vec<Genre>>` - Genres.
- `audiobook_info: Option<AudiobookInfo>` - Audiobook info.
- `user_metadata: Option<UserMetadata>` - User metadata.
- `id: Option<serde_json::Value>` - Identifier.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Audiobook`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_libro::client::AudiobookInfo

*Struct*

Narrator / duration / part counts nested under a library audiobook.

**Fields:**
- `narrators: Option<Vec<String>>` - Narrator display names when present.
- `duration: Option<u64>` - Duration in seconds.
- `track_count: Option<u32>` - Chapter/track count advertised by the catalog.
- `parts_count: Option<u32>` - Download-part count (ZIP pieces or a single M4B).



## bookclerk_plugin_source_libro::client::CLIENT_ID

*Constant*: `&str`

Optional OAuth `client_id`.

TODO: Historical / reverse-engineered clients sometimes sent a mobile
`client_id` with form-encoded password grants. Current community tools
(libro-client, librofm-downloader, bfordham/librofm) omit it and POST JSON.
Set a non-empty value here if Libro.fm starts requiring it again.



## bookclerk_plugin_source_libro::client::DEFAULT_BASE_URL

*Constant*: `&str`

Default API origin.

TODO: Confirm if regional mirrors exist; community tools all use this host.



## bookclerk_plugin_source_libro::client::DOWNLOAD_MANIFEST_PATH

*Constant*: `&str`

Download manifest (`parts` + chapter `tracks`).

Android `DownloadApi.getDownloadManifest` sends `isbn`, `client_version`, and
optional `format` (`MediaFormat.M4B` → `"m4b"`; `ZIP` → omit / null).



## bookclerk_plugin_source_libro::client::DownloadManifest

*Struct*

Download manifest.

**Fields:**
- `isbn: Option<String>` - ISBN identifier.
- `parts: Vec<DownloadPart>` - Parts.
- `tracks: Vec<ManifestTrack>` - Tracks.
- `expires_at: Option<String>` - Expires at.
- `size_bytes: Option<u64>` - Size bytes.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DownloadManifest`



## bookclerk_plugin_source_libro::client::DownloadPart

*Struct*

Download part.

**Fields:**
- `url: String` - URL.
- `size_bytes: Option<u64>` - Size bytes.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DownloadPart`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_libro::client::Genre

*Struct*

Genre tag from a library-page audiobook (`name` may be missing).

**Fields:**
- `name: Option<String>` - Display name of the genre when the API includes it.



## bookclerk_plugin_source_libro::client::LIBRARY_PATH

*Constant*: `&str`

Paginated library listing (`/api/vN/library` from the Android app prefix).



## bookclerk_plugin_source_libro::client::LibraryPage

*Struct*

Library page.

**Fields:**
- `page: u32` - Page.
- `total_pages: u32` - Total pages.
- `audiobooks: Vec<Audiobook>` - Audiobooks.
- `tags: Vec<String>` - Tags.
- `error: Option<String>` - Error.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> LibraryPage`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_libro::client::LibroClient

*Struct*

Authenticated Libro.fm HTTP helper.

**Methods:**

- `fn new<impl Into<String>>(base_url: impl Trait) -> Self` - Build a client pointed at `base_url` (no trailing slash).
- `fn with_http(self: Self, http: HttpClient) -> Self` - Override the HTTP client (tests / custom timeouts).
- `fn with_token<impl Into<String>>(self: Self, token: impl Trait) -> Self` - Attach a bearer token for subsequent requests.
- `fn access_token(self: &Self) -> Option<&str>` - Current bearer token, if any.
- `fn base_url(self: &Self) -> &str` - Base URL in use.
- `fn login(self: & mut Self, email: &str, password: &str) -> Result<TokenResponse>` - Password-grant login. Returns token metadata and stores the access token.
- `fn library_page(self: &Self, page: u32) -> Result<LibraryPage>` - Fetch one library page (`page` is 1-based).
- `fn download_manifest(self: &Self, isbn: &str, format: ManifestFormat) -> Result<DownloadManifest>` - Download-manifest for parts + chapter tracks.
- `fn packaged_m4b(self: &Self, isbn: &str) -> Result<Option<PackagedM4b>>` - Packaged M4B metadata when available (`None` on 404 / missing URL).
- `fn download_bytes(self: &Self, url: &str) -> Result<bytes::Bytes>` - Download arbitrary URL bytes (CDN part or M4B). Sends auth headers for libro.fm hosts.

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> LibroClient`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_libro::client::ManifestFormat

*Enum*

Android `MediaFormat` id for `download-manifest?format=…`.

- `ManifestFormat::M4b` → `format=m4b` (must be lowercase; `M4B` is ignored)
- `ManifestFormat::Zip` → omit `format` (APK `MediaFormat.ZIP` id is null)

**Variants:**
- `M4b` - Single-file M4B CDN URL in `parts` when the title supports it.
- `Zip` - Multi-part ZIP of MP3s (API default when `format` is omitted).

**Methods:**

- `fn query_value(self: Self) -> Option<&'static str>` - Wire value for the `format` query, if any.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> ManifestFormat`
- **PartialEq**
  - `fn eq(self: &Self, other: &ManifestFormat) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ManifestFormat`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_libro::client::ManifestTrack

*Struct*

Manifest track.

**Fields:**
- `number: Option<u32>` - Number.
- `length_msec: Option<u64>` - APK `ApiTrack` uses `length_msec` (Gson `@SerializedName`).
- `length_sec: Option<u64>` - Legacy / fixture-only key; prefer `length_msec` when both are present.
- `chapter_title: Option<String>` - Chapter title.

**Methods:**

- `fn duration_ms(self: &Self) -> u64` - Duration in milliseconds (APK wire format), falling back to `length_sec`.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ManifestTrack`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_libro::client::OAUTH_TOKEN_PATH

*Constant*: `&str`

OAuth password-grant path.



## bookclerk_plugin_source_libro::client::PACKAGED_M4B_PATH

*Constant*: `&str`

Packaged single-file M4B when Libro.fm offers it.



## bookclerk_plugin_source_libro::client::PackagedM4b

*Struct*

Packaged m4b.

**Fields:**
- `m4b_url: String` - M4b URL.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PackagedM4b`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_libro::client::TokenResponse

*Struct*

Token response.

**Fields:**
- `access_token: String` - Access token.
- `token_type: String` - Token type.
- `created_at: Option<i64>` - Unix seconds when the token was issued (mobile API).
- `expires_in: Option<i64>` - Expires in.
- `error: Option<String>` - Error.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TokenResponse`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_libro::client::USER_AGENT_VALUE

*Constant*: `&str`

User-Agent matching the official Android HTTP stack.



## bookclerk_plugin_source_libro::client::UserMetadata

*Struct*

Per-user library flags (added date, finished, hidden).

**Fields:**
- `added_at: Option<String>` - When the title was added to the user's library (API date string).
- `finished: Option<bool>` - True when the user marked the title finished.
- `hidden: Option<bool>` - True when the user hid the title from the library list.



