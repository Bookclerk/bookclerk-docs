**bookclerk_plugin_source_audible > artifacts**

# Module: artifacts

## Contents

**Functions**

- [`download_companion_pdf`](#download_companion_pdf) - Download companion PDF via the authenticated companion-file endpoint.
- [`download_cover_jpeg`](#download_cover_jpeg) - Download a JPEG cover at `size` (e.g. `500`, `1215`, or `native`).
- [`fetch_chapter_info`](#fetch_chapter_info) - Fetch curated chapter metadata (`chapter_info` object).
- [`fetch_clips_bookmarks`](#fetch_clips_bookmarks) - Fetch clips/bookmarks/notes sidecar (classic `DownloadClipsBookmarks`).
- [`fetch_product_metadata`](#fetch_product_metadata) - Fetch full catalog metadata for a title (classic `metadata.json` sidecar).

---

## bookclerk_plugin_source_audible::artifacts::download_companion_pdf

*Function*

Download companion PDF via the authenticated companion-file endpoint.

# Errors

Returns an error when the operation fails.

```rust
fn download_companion_pdf(client: &audible_rs::api::client::Client, marketplace: &str, asin: &str, dest: &std::path::Path) -> crate::error::Result<Option<std::path::PathBuf>>
```



## bookclerk_plugin_source_audible::artifacts::download_cover_jpeg

*Function*

Download a JPEG cover at `size` (e.g. `500`, `1215`, or `native`).

# Errors

Returns an error when the operation fails.

```rust
fn download_cover_jpeg(client: &audible_rs::api::client::Client, marketplace: &str, asin: &str, size: &str, dest: &std::path::Path) -> crate::error::Result<Option<std::path::PathBuf>>
```



## bookclerk_plugin_source_audible::artifacts::fetch_chapter_info

*Function*

Fetch curated chapter metadata (`chapter_info` object).

# Errors

Returns an error when the operation fails.

```rust
fn fetch_chapter_info(client: &audible_rs::api::client::Client, marketplace: &str, asin: &str, quality: bookclerk_config::AudioQuality, chapter_layout: &str) -> crate::error::Result<serde_json::Value>
```



## bookclerk_plugin_source_audible::artifacts::fetch_clips_bookmarks

*Function*

Fetch clips/bookmarks/notes sidecar (classic `DownloadClipsBookmarks`).

# Errors

Returns an error when the operation fails.

```rust
fn fetch_clips_bookmarks(client: &audible_rs::api::client::Client, asin: &str, acr: Option<&str>, version: Option<&str>, content_format: Option<&str>) -> crate::error::Result<Option<serde_json::Value>>
```



## bookclerk_plugin_source_audible::artifacts::fetch_product_metadata

*Function*

Fetch full catalog metadata for a title (classic `metadata.json` sidecar).

# Errors

Returns an error when the operation fails.

```rust
fn fetch_product_metadata(client: &audible_rs::api::client::Client, marketplace: &str, asin: &str, quality: bookclerk_config::AudioQuality) -> crate::error::Result<serde_json::Value>
```



