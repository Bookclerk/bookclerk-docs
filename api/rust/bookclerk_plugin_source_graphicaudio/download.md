**bookclerk_plugin_source_graphicaudio > download**

# Module: download

## Contents

**Structs**

- [`TitleFetchRequest`](#titlefetchrequest) - Parameters for [`fetch_title_with_mode`](download.md#bookclerk_plugin_source_graphicaudiodownloadfetch_title_with_mode).

**Functions**

- [`fetch_title_materials`](#fetch_title_materials) - Password env (same as CLI login) for Magento ZIP / Browser Player.
- [`fetch_title_with_mode`](#fetch_title_with_mode) - Fetch owned audio for one product using the configured access path only

**Constants**

- [`GA_ACCESS_ENV`](#ga_access_env) - Preferred env override (`BOOKCLERK_GA_ACCESS`); falls back to [`GA_FETCH_ENV`](download.md#bookclerk_plugin_source_graphicaudiodownloadga_fetch_env).
- [`GA_FETCH_ENV`](#ga_fetch_env) - Env override for which GraphicAudio access path to use (legacy name).

---

## bookclerk_plugin_source_graphicaudio::download::GA_ACCESS_ENV

*Constant*: `&str`

Preferred env override (`BOOKCLERK_GA_ACCESS`); falls back to [`GA_FETCH_ENV`](download.md#bookclerk_plugin_source_graphicaudiodownloadga_fetch_env).



## bookclerk_plugin_source_graphicaudio::download::GA_FETCH_ENV

*Constant*: `&str`

Env override for which GraphicAudio access path to use (legacy name).



## bookclerk_plugin_source_graphicaudio::download::TitleFetchRequest

*Struct*

Parameters for [`fetch_title_with_mode`](download.md#bookclerk_plugin_source_graphicaudiodownloadfetch_title_with_mode).

**Generic Parameters:**
- 'a

**Fields:**
- `store_base_url: &'a str` - Store base URL.
- `email: &'a str` - Email.
- `product_id: &'a str` - Product Identifier.
- `product_title: Option<&'a str>` - Product title.
- `cache_dir: &'a std::path::Path` - Absolute path to the download/work cache directory.
- `prefer_hi: bool` - Prefer hi.
- `mode: crate::options::GraphicAudioAccess` - Mode.
- `password: Option<&'a str>` - Password.
- `zip_container: crate::options::GraphicAudioContainer` - ZIP SKU preference when `Self::mode` is `GraphicAudioAccess::Zip`.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> TitleFetchRequest<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_graphicaudio::download::fetch_title_materials

*Function*

Password env (same as CLI login) for Magento ZIP / Browser Player.
Fetch one product id into `cache_dir` via `api/links` Hi/Lo URLs.

# Errors

Returns an error when the operation fails.

```rust
fn fetch_title_materials(client: &crate::client::GraphicAudioClient, product_id: &str, cache_dir: &std::path::Path) -> crate::error::Result<bookclerk_source::PlainFetch>
```



## bookclerk_plugin_source_graphicaudio::download::fetch_title_with_mode

*Function*

Fetch owned audio for one product using the configured access path only
(no ZIP→web→device cascade).

# Errors

Returns an error when the operation fails.

```rust
fn fetch_title_with_mode(access: &crate::client::GraphicAudioClient, req: TitleFetchRequest) -> crate::error::Result<bookclerk_source::PlainFetch>
```



