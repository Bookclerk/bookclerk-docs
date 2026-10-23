**bookclerk_plugin_source_audible > sync**

# Module: sync

## Contents

**Functions**

- [`collect_account_books`](#collect_account_books) - Fetch library pages for one account without touching the library DB.
- [`scan_account_into_library`](#scan_account_into_library) - Fetch library pages for one authenticated client and upsert into `library`.
- [`scan_library`](#scan_library) - Sync Audible library for configured accounts into `library`.

---

## bookclerk_plugin_source_audible::sync::collect_account_books

*Function*

Fetch library pages for one account without touching the library DB.

Used by the external guest plugin (host upserts [`NewBook`](../bookclerk_library/store.md#bookclerk_librarystorenewbook) rows).

# Errors

Returns an error when the operation fails.

```rust
fn collect_account_books(client: &audible_rs::api::client::Client, account_id: &str, marketplace: &str, page_size: u32, import_episodes: bool, import_plus_titles: bool) -> crate::error::Result<(Vec<bookclerk_library::NewBook>, u32)>
```



## bookclerk_plugin_source_audible::sync::scan_account_into_library

*Function*

Fetch library pages for one authenticated client and upsert into `library`.

Exposed for wiremock CI tests (inject a [`Client`](https://docs.rs/audible_rs/latest/audible_rs/api/client/struct.Client.html) with `api_base_override`).

# Errors

Returns an error when the operation fails.

```rust
fn scan_account_into_library(scope: &bookclerk_library::SourceScope, client: &audible_rs::api::client::Client, account_id: &str, marketplace: &str, page_size: u32, import_episodes: bool, import_plus_titles: bool) -> crate::error::Result<(usize, u32)>
```



## bookclerk_plugin_source_audible::sync::scan_library

*Function*

Sync Audible library for configured accounts into `library`.

Accounts are resolved from the `encrypted_secrets` table (DB-backed); no
`Accounts/*.audible.auth` files are read.

# Errors

Returns an error when the operation fails.

```rust
fn scan_library(scope: &bookclerk_library::SourceScope, options: bookclerk_source::ScanOptions) -> crate::error::Result<bookclerk_source::ScanSummary>
```



