**bookclerk_source > abi**

# Module: abi

## Contents

**Functions**

- [`abridgement_from_flag`](#abridgement_from_flag) - Tri-state abridgement flag ↔ the host's `Option<bool>`.
- [`abridgement_to_flag`](#abridgement_to_flag) - Host `Option<bool>` view of the wire tri-state abridgement flag.
- [`account_credentials`](#account_credentials) - Host-issued per-account credential list for `scan`.
- [`account_credentials_json`](#account_credentials_json) - Guest view of the per-account credential list as JSON documents.
- [`catalog_field_from_abi`](#catalog_field_from_abi) - Host search field for a wire facet (`any` → `None`).
- [`catalog_field_to_abi`](#catalog_field_to_abi) - Wire facet for an optional host search field (`None` → `any`).
- [`credentials_from_bytes`](#credentials_from_bytes) - Decodes an opaque credential blob into the JSON document guests store.
- [`credentials_to_bytes`](#credentials_to_bytes) - Encodes a credential document as the opaque bytes the ABI carries.
- [`expand_candidates_params`](#expand_candidates_params) - Builds `expandCandidates` params from a seed plus the result cap.
- [`expand_seed_from_params`](#expand_seed_from_params) - Splits `expandCandidates` params into the host seed and result cap.
- [`login_params`](#login_params) - Builds guest login params. `plugin_data_dir` is the jail-granted data
- [`scan_book_from_new`](#scan_book_from_new) - Guest scan row for a library [`NewBook`](../bookclerk_library/store.md#bookclerk_librarystorenewbook) the storefront produced.
- [`scan_book_to_new`](#scan_book_to_new) - Maps a guest scan row onto [`NewBook`](../bookclerk_library/store.md#bookclerk_librarystorenewbook), forcing `source` to the plugin id
- [`scan_summary`](#scan_summary) - Builds a guest scan summary from the books a scan produced.
- [`scan_summary_from_abi`](#scan_summary_from_abi) - Host counters from a guest scan summary (`books_upserted` is the host's

**Constants**

- [`DEFAULT_EXTERNAL_SORT_KEY`](#default_external_sort_key) - Host-side registry sort key for guests that leave `sortKey` at zero.
- [`DEFAULT_LIST_DEALS_LIMIT`](#default_list_deals_limit) - `limit` sent for `crate::ContentSource::list_deals` when the caller omits one.

---

## bookclerk_source::abi::DEFAULT_EXTERNAL_SORT_KEY

*Constant*: `u32`

Host-side registry sort key for guests that leave `sortKey` at zero.



## bookclerk_source::abi::DEFAULT_LIST_DEALS_LIMIT

*Constant*: `u32`

`limit` sent for `crate::ContentSource::list_deals` when the caller omits one.



## bookclerk_source::abi::abridgement_from_flag

*Function*

Tri-state abridgement flag ↔ the host's `Option<bool>`.

```rust
fn abridgement_from_flag(is_abridged: Option<bool>) -> abi::Abridgement
```



## bookclerk_source::abi::abridgement_to_flag

*Function*

Host `Option<bool>` view of the wire tri-state abridgement flag.

```rust
fn abridgement_to_flag(abridgement: abi::Abridgement) -> Option<bool>
```



## bookclerk_source::abi::account_credentials

*Function*

Host-issued per-account credential list for `scan`.

```rust
fn account_credentials<impl IntoIterator<Item = (String, Vec<u8>)>>(credentials: impl Trait) -> Vec<abi::AccountCredential>
```



## bookclerk_source::abi::account_credentials_json

*Function*

Guest view of the per-account credential list as JSON documents.

# Errors

Returns `crate::SourceError::Auth` when any blob is not JSON.

```rust
fn account_credentials_json(credentials: &[abi::AccountCredential]) -> crate::Result<std::collections::BTreeMap<String, serde_json::Value>>
```



## bookclerk_source::abi::catalog_field_from_abi

*Function*

Host search field for a wire facet (`any` → `None`).

```rust
fn catalog_field_from_abi(field: abi::CatalogField) -> Option<crate::types::CatalogSearchField>
```



## bookclerk_source::abi::catalog_field_to_abi

*Function*

Wire facet for an optional host search field (`None` → `any`).

```rust
fn catalog_field_to_abi(field: Option<crate::types::CatalogSearchField>) -> abi::CatalogField
```



## bookclerk_source::abi::credentials_from_bytes

*Function*

Decodes an opaque credential blob into the JSON document guests store.

# Errors

Returns `crate::SourceError::Auth` when the bytes are not JSON.

```rust
fn credentials_from_bytes(bytes: &[u8]) -> crate::Result<serde_json::Value>
```



## bookclerk_source::abi::credentials_to_bytes

*Function*

Encodes a credential document as the opaque bytes the ABI carries.

# Errors

Returns when `credentials` cannot be serialized.

```rust
fn credentials_to_bytes(credentials: &serde_json::Value) -> crate::Result<Vec<u8>>
```



## bookclerk_source::abi::expand_candidates_params

*Function*

Builds `expandCandidates` params from a seed plus the result cap.

```rust
fn expand_candidates_params(seed: &crate::types::ExpandSeed, limit: usize) -> abi::ExpandCandidatesParams
```



## bookclerk_source::abi::expand_seed_from_params

*Function*

Splits `expandCandidates` params into the host seed and result cap.

```rust
fn expand_seed_from_params(params: abi::ExpandCandidatesParams) -> (crate::types::ExpandSeed, usize)
```



## bookclerk_source::abi::login_params

*Function*

Builds guest login params. `plugin_data_dir` is the jail-granted data
directory; the host fills `callback_ipc` / `callback_public_base` after it
starts the OAuth callback proxy.

```rust
fn login_params(plugin_data_dir: String, opts: crate::types::LoginOptions) -> abi::LoginParams
```



## bookclerk_source::abi::scan_book_from_new

*Function*

Guest scan row for a library [`NewBook`](../bookclerk_library/store.md#bookclerk_librarystorenewbook) the storefront produced.

```rust
fn scan_book_from_new(book: bookclerk_library::NewBook) -> abi::ScanBook
```



## bookclerk_source::abi::scan_book_to_new

*Function*

Maps a guest scan row onto [`NewBook`](../bookclerk_library/store.md#bookclerk_librarystorenewbook), forcing `source` to the plugin id
so a guest can never claim another storefront's rows. The owning
`plugin_key` is stamped by the [`bookclerk_library::SourceScope`](../bookclerk_library/scope.md#bookclerk_libraryscopesourcescope) on write.

```rust
fn scan_book_to_new(plugin_id: &str, book: abi::ScanBook) -> bookclerk_library::NewBook
```



## bookclerk_source::abi::scan_summary

*Function*

Builds a guest scan summary from the books a scan produced.

```rust
fn scan_summary(books: Vec<bookclerk_library::NewBook>, accounts: usize, pages: u32) -> abi::ScanSummary
```



## bookclerk_source::abi::scan_summary_from_abi

*Function*

Host counters from a guest scan summary (`books_upserted` is the host's
own upsert count when it upserted rows itself).

```rust
fn scan_summary_from_abi(summary: &abi::ScanSummary, upserted: usize) -> crate::ScanSummary
```



