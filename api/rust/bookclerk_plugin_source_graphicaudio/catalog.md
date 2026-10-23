**bookclerk_plugin_source_graphicaudio > catalog**

# Module: catalog

## Contents

**Structs**

- [`MagentoCatalogProduct`](#magentocatalogproduct) - One Magento catalog product (grid / related / product page).

**Functions**

- [`catalog_http_client`](#catalog_http_client) - Shared HTTP client for public Magento catalog pages.
- [`expand_from_product_id`](#expand_from_product_id) - Expand a owned Magento product id into related + series siblings.
- [`expand_from_search`](#expand_from_search) - Expand via Magento catalog search (series name or title).
- [`fetch_product_by_id`](#fetch_product_by_id) - Magento product page by numeric product id.
- [`fetch_series_page`](#fetch_series_page) - Fetch titles listed on a series category page (includes series-set SKUs).
- [`parse_catalog_grid`](#parse_catalog_grid) - Parse Magento product grid / related list items.
- [`parse_related_products`](#parse_related_products) - Parse the Recommended / related products block on a product page.
- [`search_catalog`](#search_catalog) - Search Magento catalog; often redirects to a matching series page.
- [`search_catalog_page`](#search_catalog_page) - Like [`search_catalog`](catalog.md#bookclerk_plugin_source_graphicaudiocatalogsearch_catalog) with an optional Magento page index.

---

## bookclerk_plugin_source_graphicaudio::catalog::MagentoCatalogProduct

*Struct*

One Magento catalog product (grid / related / product page).

**Fields:**
- `product_id: String` - Product Identifier.
- `sku: Option<String>` - Sku.
- `title: String` - Title.
- `url: Option<String>` - URL.
- `series: Option<String>` - Series.
- `cover_url: Option<String>` - Cover URL.

**Methods:**

- `fn is_series_set(self: &Self) -> bool` - Is series set.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MagentoCatalogProduct`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &MagentoCatalogProduct) -> bool`



## bookclerk_plugin_source_graphicaudio::catalog::catalog_http_client

*Function*

Shared HTTP client for public Magento catalog pages.

# Errors

Returns an error when the operation fails.

```rust
fn catalog_http_client() -> crate::error::Result<bookclerk_plugin_sdk::http::Client>
```



## bookclerk_plugin_source_graphicaudio::catalog::expand_from_product_id

*Function*

Expand a owned Magento product id into related + series siblings.

# Errors

Returns an error when the operation fails.

```rust
fn expand_from_product_id(http: &bookclerk_plugin_sdk::http::Client, store_base: Option<&str>, product_id: &str) -> crate::error::Result<Vec<MagentoCatalogProduct>>
```



## bookclerk_plugin_source_graphicaudio::catalog::expand_from_search

*Function*

Expand via Magento catalog search (series name or title).

# Errors

Returns an error when the operation fails.

```rust
fn expand_from_search(http: &bookclerk_plugin_sdk::http::Client, store_base: Option<&str>, query: &str) -> crate::error::Result<Vec<MagentoCatalogProduct>>
```



## bookclerk_plugin_source_graphicaudio::catalog::fetch_product_by_id

*Function*

Magento product page by numeric product id.

# Errors

Returns an error when the operation fails.

```rust
fn fetch_product_by_id(http: &bookclerk_plugin_sdk::http::Client, store_base: &str, product_id: &str) -> crate::error::Result<(String, Vec<MagentoCatalogProduct>, Option<String>)>
```



## bookclerk_plugin_source_graphicaudio::catalog::fetch_series_page

*Function*

Fetch titles listed on a series category page (includes series-set SKUs).

# Errors

Returns an error when the operation fails.

```rust
fn fetch_series_page(http: &bookclerk_plugin_sdk::http::Client, series_url: &str) -> crate::error::Result<Vec<MagentoCatalogProduct>>
```



## bookclerk_plugin_source_graphicaudio::catalog::parse_catalog_grid

*Function*

Parse Magento product grid / related list items.

```rust
fn parse_catalog_grid(html: &str) -> Vec<MagentoCatalogProduct>
```



## bookclerk_plugin_source_graphicaudio::catalog::parse_related_products

*Function*

Parse the Recommended / related products block on a product page.

```rust
fn parse_related_products(html: &str) -> Vec<MagentoCatalogProduct>
```



## bookclerk_plugin_source_graphicaudio::catalog::search_catalog

*Function*

Search Magento catalog; often redirects to a matching series page.

`page` > 1 tries Magento `?p=` once; when that yields nothing the source is
treated as exhausted by the host cursor (empty page).

# Errors

Returns an error when the operation fails.

```rust
fn search_catalog(http: &bookclerk_plugin_sdk::http::Client, store_base: &str, query: &str) -> crate::error::Result<Vec<MagentoCatalogProduct>>
```



## bookclerk_plugin_source_graphicaudio::catalog::search_catalog_page

*Function*

Like [`search_catalog`](catalog.md#bookclerk_plugin_source_graphicaudiocatalogsearch_catalog) with an optional Magento page index.

# Errors

Returns an error when the operation fails.

```rust
fn search_catalog_page(http: &bookclerk_plugin_sdk::http::Client, store_base: &str, query: &str, page: u32) -> crate::error::Result<Vec<MagentoCatalogProduct>>
```



