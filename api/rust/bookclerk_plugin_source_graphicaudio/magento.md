**bookclerk_plugin_source_graphicaudio > magento**

# Module: magento

## Contents

**Structs**

- [`DownloadableProduct`](#downloadableproduct) - One row from My Downloadable Products.
- [`LibraryItem`](#libraryitem) - One owned title from Browser Player `content_library` HTML.
- [`MagentoClient`](#magentoclient) - Magento customer session + cookie jar (including Browser Player CloudFront cookies).

**Constants**

- [`DEFAULT_STORE_URL`](#default_store_url) - Default Magento storefront origin (no `/access` suffix).

---

## bookclerk_plugin_source_graphicaudio::magento::DEFAULT_STORE_URL

*Constant*: `&str`

Default Magento storefront origin (no `/access` suffix).



## bookclerk_plugin_source_graphicaudio::magento::DownloadableProduct

*Struct*

One row from My Downloadable Products.

**Fields:**
- `title: String` - Title.
- `option_label: String` - Option label.
- `download_url: String` - Download URL.
- `remaining: Option<u32>` - Remaining.
- `status: String` - Status.

**Methods:**

- `fn format_rank(self: &Self) -> u8` - Rank for Magento ZIP SKU selection (lower = better).
- `fn format_rank_for(self: &Self, preference: GraphicAudioContainer) -> u8` - Rank using a configured container preference.
- `fn has_remaining(self: &Self) -> bool` - Has remaining.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> DownloadableProduct`



## bookclerk_plugin_source_graphicaudio::magento::LibraryItem

*Struct*

One owned title from Browser Player `content_library` HTML.

**Fields:**
- `product_id: String` - Product Identifier.
- `title: Option<String>` - Title.
- `listen_path: Option<String>` - Listen path.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LibraryItem`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_graphicaudio::magento::MagentoClient

*Struct*

Magento customer session + cookie jar (including Browser Player CloudFront cookies).

**Methods:**

- `fn new<impl Into<String>>(base_url: impl Trait) -> Result<Self>` - Build a client for `base_url` (trailing slash stripped).
- `fn base_url(self: &Self) -> &str` - Base URL.
- `fn login(self: &Self, email: &str, password: &str) -> Result<()>` - Customer login via Magento `loginPost` (form_key + email/password).
- `fn list_downloadable(self: &Self) -> Result<Vec<DownloadableProduct>>` - Parse My Downloadable Products into link rows.
- `fn resolve_download_url(self: &Self, download_url: &str) -> Result<String>` - Resolve a Magento downloadable link to a CDN URL (or stream body if no redirect).
- `fn download_to_path(self: &Self, url: &str, path: &Path) -> Result<()>` - Stream `url` to `path` (uses Magento/CloudFront cookies when present).
- `fn content_library_html(self: &Self) -> Result<String>` - Fetch Browser Player library HTML (`library/index/content_library`).
- `fn list_library(self: &Self) -> Result<Vec<LibraryItem>>` - List owned titles from Browser Player library (no Access App device token).
- `fn player_listen_url(self: &Self, product_id: &str) -> Result<String>` - Find Browser Player listen URL for `product_id` via `library/index/content_library`.
- `fn player_audio_url(self: &Self, product_id: &str) -> Result<String>` - Open the Browser Player page and return the `<audio>` media URL (sets CF cookies).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> MagentoClient`



