**bookclerk_plugin_source_chirp > sync**

# Module: sync

## Contents

**Structs**

- [`ScanOptions`](#scanoptions) - Options for a Chirp library scan.

**Functions**

- [`audiobook_to_new_book`](#audiobook_to_new_book) - Map a Chirp audiobook to a [`NewBook`](../bookclerk_library/store.md#bookclerk_librarystorenewbook) row.
- [`collect_account_books`](#collect_account_books) - Fetch all library pages for one client and collect books (no DB writes).
- [`scan_account_into_library`](#scan_account_into_library) - Fetch all library pages for one client and upsert books.
- [`scan_library`](#scan_library) - Sync Chirp libraries for configured accounts into `library`.

---

## bookclerk_plugin_source_chirp::sync::ScanOptions

*Struct*

Options for a Chirp library scan.

**Fields:**
- `accounts: Vec<String>` - Accounts.
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



## bookclerk_plugin_source_chirp::sync::audiobook_to_new_book

*Function*

Map a Chirp audiobook to a [`NewBook`](../bookclerk_library/store.md#bookclerk_librarystorenewbook) row.

```rust
fn audiobook_to_new_book(book: &crate::client::Audiobook, account_id: &str, marketplace: &str) -> bookclerk_library::NewBook
```



## bookclerk_plugin_source_chirp::sync::collect_account_books

*Function*

Fetch all library pages for one client and collect books (no DB writes).

# Errors

Returns an error when the operation fails.

```rust
fn collect_account_books(client: &crate::client::ChirpClient, account_id: &str, marketplace: &str, page_size: u32) -> crate::error::Result<(Vec<bookclerk_library::NewBook>, u32)>
```



## bookclerk_plugin_source_chirp::sync::scan_account_into_library

*Function*

Fetch all library pages for one client and upsert books.

# Errors

Returns an error when the operation fails.

```rust
fn scan_account_into_library(library: &bookclerk_library::SourceScope, client: &crate::client::ChirpClient, account_id: &str, marketplace: &str, page_size: u32) -> crate::error::Result<(usize, u32)>
```



## bookclerk_plugin_source_chirp::sync::scan_library

*Function*

Sync Chirp libraries for configured accounts into `library`.

Accounts are resolved from `encrypted_secrets` (DB-backed); no
`Accounts/*.chirp.auth` files are read.

# Errors

Returns an error when the operation fails.

```rust
fn scan_library(library: &bookclerk_library::SourceScope, options: ScanOptions, graphql_url: Option<&str>) -> crate::error::Result<bookclerk_source::ScanSummary>
```



