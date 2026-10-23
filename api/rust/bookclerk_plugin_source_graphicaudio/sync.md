**bookclerk_plugin_source_graphicaudio > sync**

# Module: sync

## Contents

**Structs**

- [`ScanContext`](#scancontext) - Runtime context for [`scan_library`](sync.md#bookclerk_plugin_source_graphicaudiosyncscan_library) (Access App vs Magento).
- [`ScanOptions`](#scanoptions) - Options for a GraphicAudio library scan.

**Functions**

- [`collect_account_books`](#collect_account_books) - Collect owned titles for one account (no DB writes).
- [`product_to_new_book`](#product_to_new_book) - Map a GraphicAudio product JSON object to a [`NewBook`](../bookclerk_library/store.md#bookclerk_librarystorenewbook) row.
- [`scan_library`](#scan_library) - Sync GraphicAudio libraries for configured accounts into `library`.

---

## bookclerk_plugin_source_graphicaudio::sync::ScanContext

*Struct*

Runtime context for [`scan_library`](sync.md#bookclerk_plugin_source_graphicaudiosyncscan_library) (Access App vs Magento).

**Generic Parameters:**
- 'a

**Fields:**
- `access_base_url: Option<&'a str>` - Access base URL.
- `store_base_url: Option<&'a str>` - Store base URL.
- `access: crate::options::GraphicAudioAccess` - Access.
- `magento_password: Option<&'a str>` - Magento password.

**Traits:** Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> ScanContext<'a>`



## bookclerk_plugin_source_graphicaudio::sync::ScanOptions

*Struct*

Options for a GraphicAudio library scan.

**Fields:**
- `accounts: Vec<String>` - Limit to specific account emails / labels / ids. Empty = all accounts.
- `include_samples: bool` - When true, also import promotional `Type=sample` entries (default: false).
- `page_size: u32` - Page size.

**Trait Implementations:**

- **From**
  - `fn from(opts: &bookclerk_source::ScanOptions) -> Self`
- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> ScanOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_graphicaudio::sync::collect_account_books

*Function*

Collect owned titles for one account (no DB writes).

Prefer Access App catalog when `access=device` and a device token exists.
For web/zip (default), use Magento Browser Player library so we do not
depend on a device slot for ownership listing. Falls back to Access App
when a legacy device token exists and Magento password is unavailable.

# Errors

Returns an error when the operation fails.

```rust
fn collect_account_books(auth: &crate::auth::GraphicAudioAuthFile, access_base: &str, store_base: &str, access: crate::options::GraphicAudioAccess, magento_password: Option<&str>, include_samples: bool, account_id: &str, marketplace: &str) -> crate::error::Result<(Vec<bookclerk_library::NewBook>, u32)>
```



## bookclerk_plugin_source_graphicaudio::sync::product_to_new_book

*Function*

Map a GraphicAudio product JSON object to a [`NewBook`](../bookclerk_library/store.md#bookclerk_librarystorenewbook) row.

```rust
fn product_to_new_book(product: &crate::client::Product, account_id: &str, marketplace: &str) -> bookclerk_library::NewBook
```



## bookclerk_plugin_source_graphicaudio::sync::scan_library

*Function*

Sync GraphicAudio libraries for configured accounts into `library`.

Accounts are resolved from `encrypted_secrets` (DB-backed); no
`Accounts/*.ga.auth` files are read.

# Errors

Returns an error when the operation fails.

```rust
fn scan_library(library: &bookclerk_library::SourceScope, options: ScanOptions, ctx: ScanContext) -> crate::error::Result<bookclerk_source::ScanSummary>
```



