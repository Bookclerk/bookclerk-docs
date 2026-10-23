**bookclerk_plugin_source_graphicaudio > client**

# Module: client

## Contents

**Structs**

- [`DownloadLinks`](#downloadlinks) - Lo/Hi download URLs from `api/links`.
- [`GraphicAudioClient`](#graphicaudioclient) - Authenticated GraphicAudio HTTP helper.
- [`Product`](#product) - One library / sample product from `api/products`.

**Constants**

- [`DEFAULT_BASE_URL`](#default_base_url) - Default API origin (no trailing slash after `/access`).
- [`LOGIN_PATH`](#login_path) - Device activation / password login.
- [`PRODUCTS_PATH`](#products_path) - Library + sample products listing.
- [`REMOVE_PATH`](#remove_path) - Forget a device activation (`client_id` form field).

---

## bookclerk_plugin_source_graphicaudio::client::DEFAULT_BASE_URL

*Constant*: `&str`

Default API origin (no trailing slash after `/access`).



## bookclerk_plugin_source_graphicaudio::client::DownloadLinks

*Struct*

Lo/Hi download URLs from `api/links`.

**Fields:**
- `lo: Option<String>` - Lo.
- `hi: Option<String>` - Hi.

**Methods:**

- `fn preferred_url(self: &Self) -> Option<&str>` - Prefer high-quality URL, else low.
- `fn url_for_quality(self: &Self, prefer_hi: bool) -> Option<&str>` - Select Hi or Lo based on whether high quality is preferred.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DownloadLinks`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_graphicaudio::client::GraphicAudioClient

*Struct*

Authenticated GraphicAudio HTTP helper.

**Methods:**

- `fn new<impl Into<String>>(base_url: impl Trait) -> Self` - Build a client pointed at `base_url` (trailing slash stripped).
- `fn with_http(self: Self, http: HttpClient) -> Self` - Override the HTTP client (tests / custom timeouts).
- `fn with_token<impl Into<String>>(self: Self, token: impl Trait) -> Self` - Attach an activation token for subsequent requests.
- `fn token(self: &Self) -> Option<&str>` - Current token, if any.
- `fn base_url(self: &Self) -> &str` - Base URL in use.
- `fn login(self: & mut Self, username: &str, password: &str, client_id: &str) -> Result<String>` - `POST activation/login` — returns the opaque token string.
- `fn products(self: &Self) -> Result<Vec<Product>>` - `GET api/products` — owned titles plus promotional samples.
- `fn links(self: &Self, product_id: &str) -> Result<DownloadLinks>` - `GET api/links?product=` — Lo/Hi plain media URLs.
- `fn download_bytes(self: &Self, url: &str) -> Result<bytes::Bytes>` - Download bytes from an absolute media URL (no Authorization).
- `fn download_to_path(self: &Self, url: &str, path: &Path) -> Result<()>` - Stream an absolute media URL to `path` without buffering the whole body.
- `fn remove_activation(self: &Self, client_id: &str) -> Result<()>` - `POST activation/remove` — drop a device slot for `client_id`.

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> GraphicAudioClient`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_graphicaudio::client::LOGIN_PATH

*Constant*: `&str`

Device activation / password login.



## bookclerk_plugin_source_graphicaudio::client::PRODUCTS_PATH

*Constant*: `&str`

Library + sample products listing.



## bookclerk_plugin_source_graphicaudio::client::Product

*Struct*

One library / sample product from `api/products`.

**Fields:**
- `id: String` - Identifier.
- `product_type: Option<String>` - Product type.
- `product_name: Option<String>` - Product name.
- `title: Option<String>` - Title.
- `author: Option<String>` - Author.
- `series: Option<String>` - Series.
- `episode: Option<String>` - Episode.
- `genre: Option<String>` - Genre.
- `image: Option<String>` - Image.
- `thumbnail: Option<String>` - Thumbnail.
- `running_time: Option<String>` - Running time.
- `purchased_date: Option<String>` - Purchased date.
- `release_date: Option<String>` - Release date.

**Methods:**

- `fn is_sample(self: &Self) -> bool` - Whether this entry is a promotional sample (not owned).
- `fn display_title(self: &Self) -> String` - Display title preferring `Series` + `ProductName` for owned volumes.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Product`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_graphicaudio::client::REMOVE_PATH

*Constant*: `&str`

Forget a device activation (`client_id` form field).



