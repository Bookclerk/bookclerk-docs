# bookclerk_source::abi

Conversions between host storefront types and the typed Cap'n Proto plugin
ABI ([`bookclerk_plugin_abi`](../bookclerk_plugin_abi/index.md)).

## Audience

The plugin host (`ExternalSource`) and every native storefront guest. Both
sides speak the same generated structs, so the mapping lives here once
instead of being copied into each plugin crate.

Wire optionality follows the ABI convention: empty text / zero numbers mean
"absent" and project to `Option::None` on this side.

| Item | Kind | Summary |
| --- | --- | --- |
| [`abridgement_from_flag`](#abridgement_from_flag) | function | Tri-state abridgement flag ↔ the host's `Option<bool>`. |
| [`abridgement_to_flag`](#abridgement_to_flag) | function | Host `Option<bool>` view of the wire tri-state abridgement flag. |
| [`catalog_field_to_abi`](#catalog_field_to_abi) | function | Wire facet for an optional host search field (`None` → `any`). |
| [`catalog_field_from_abi`](#catalog_field_from_abi) | function | Host search field for a wire facet (`any` → `None`). |
| [`expand_candidates_params`](#expand_candidates_params) | function | Builds `expandCandidates` params from a seed plus the result cap. |
| [`expand_seed_from_params`](#expand_seed_from_params) | function | Splits `expandCandidates` params into the host seed and result cap. |
| [`login_params`](#login_params) | function | Builds guest login params. `plugin_data_dir` is the jail-granted data directory; the host fills `callback_ipc` / `callback_public_base` after it starts the OAuth callback proxy. |
| [`scan_book_from_new`](#scan_book_from_new) | function | Guest scan row for a library `NewBook` the storefront produced. |
| [`scan_book_to_new`](#scan_book_to_new) | function | Maps a guest scan row onto `NewBook`, forcing `source` to the plugin id so a guest can never claim another storefront's rows. The owning `plugin_key` is stamped by the `bookclerk_library::SourceScope` on write. |
| [`scan_summary`](#scan_summary) | function | Builds a guest scan summary from the books a scan produced. |
| [`scan_summary_from_abi`](#scan_summary_from_abi) | function | Host counters from a guest scan summary (`books_upserted` is the host's own upsert count when it upserted rows itself). |
| [`credentials_to_bytes`](#credentials_to_bytes) | function | Encodes a credential document as the opaque bytes the ABI carries. |
| [`credentials_from_bytes`](#credentials_from_bytes) | function | Decodes an opaque credential blob into the JSON document guests store. |
| [`account_credentials`](#account_credentials) | function | Host-issued per-account credential list for `scan`. |
| [`account_credentials_json`](#account_credentials_json) | function | Guest view of the per-account credential list as JSON documents. |
| [`DEFAULT_LIST_DEALS_LIMIT`](#default_list_deals_limit) | constant | `limit` sent for `crate::ContentSource::list_deals` when the caller omits one. |
| [`DEFAULT_EXTERNAL_SORT_KEY`](#default_external_sort_key) | constant | Host-side registry sort key for guests that leave `sortKey` at zero. |

## Functions

### abridgement_from_flag

```rust
pub fn abridgement_from_flag(is_abridged: Option<bool>) -> Abridgement
```

Tri-state abridgement flag ↔ the host's `Option<bool>`.

### abridgement_to_flag

```rust
pub fn abridgement_to_flag(abridgement: Abridgement) -> Option<bool>
```

Host `Option<bool>` view of the wire tri-state abridgement flag.

### catalog_field_to_abi

```rust
pub fn catalog_field_to_abi(field: Option<CatalogSearchField>) -> CatalogField
```

Wire facet for an optional host search field (`None` → `any`).

### catalog_field_from_abi

```rust
pub fn catalog_field_from_abi(field: CatalogField) -> Option<CatalogSearchField>
```

Host search field for a wire facet (`any` → `None`).

### expand_candidates_params

```rust
pub fn expand_candidates_params(seed: &ExpandSeed, limit: usize) -> ExpandCandidatesParams
```

Builds `expandCandidates` params from a seed plus the result cap.

### expand_seed_from_params

```rust
pub fn expand_seed_from_params(params: ExpandCandidatesParams) -> (ExpandSeed, usize)
```

Splits `expandCandidates` params into the host seed and result cap.

### login_params

```rust
pub fn login_params(plugin_data_dir: String, opts: LoginOptions) -> LoginParams
```

Builds guest login params. `plugin_data_dir` is the jail-granted data
directory; the host fills `callback_ipc` / `callback_public_base` after it
starts the OAuth callback proxy.

### scan_book_from_new

```rust
pub fn scan_book_from_new(book: NewBook) -> ScanBook
```

Guest scan row for a library [`NewBook`](../bookclerk_library/index.md#newbook) the storefront produced.

### scan_book_to_new

```rust
pub fn scan_book_to_new(plugin_id: &str, book: ScanBook) -> NewBook
```

Maps a guest scan row onto [`NewBook`](../bookclerk_library/index.md#newbook), forcing `source` to the plugin id
so a guest can never claim another storefront's rows. The owning
`plugin_key` is stamped by the [`bookclerk_library::SourceScope`](../bookclerk_library/scope.md#sourcescope) on write.

### scan_summary

```rust
pub fn scan_summary(books: Vec<NewBook>, accounts: usize, pages: u32) -> ScanSummary
```

Builds a guest scan summary from the books a scan produced.

### scan_summary_from_abi

```rust
pub fn scan_summary_from_abi(summary: &ScanSummary, upserted: usize) -> ScanSummary
```

Host counters from a guest scan summary (`books_upserted` is the host's
own upsert count when it upserted rows itself).

### credentials_to_bytes

```rust
pub fn credentials_to_bytes(credentials: &Value) -> Result<Vec<u8>>
```

Encodes a credential document as the opaque bytes the ABI carries.

#### Errors

Returns when `credentials` cannot be serialized.

### credentials_from_bytes

```rust
pub fn credentials_from_bytes(bytes: &[u8]) -> Result<Value>
```

Decodes an opaque credential blob into the JSON document guests store.

#### Errors

Returns [`crate::SourceError::Auth`](index.md#sourceerror) when the bytes are not JSON.

### account_credentials

```rust
pub fn account_credentials(
    credentials: impl IntoIterator<Item = (String, Vec<u8>)>,
) -> Vec<AccountCredential>
```

Host-issued per-account credential list for `scan`.

### account_credentials_json

```rust
pub fn account_credentials_json(
    credentials: &[AccountCredential],
) -> Result<BTreeMap<String, Value>>
```

Guest view of the per-account credential list as JSON documents.

#### Errors

Returns [`crate::SourceError::Auth`](index.md#sourceerror) when any blob is not JSON.

## Constants

### DEFAULT_LIST_DEALS_LIMIT

```rust
pub const DEFAULT_LIST_DEALS_LIMIT: u32 = 20;
```

`limit` sent for [`crate::ContentSource::list_deals`](index.md#contentsourcelist_deals) when the caller omits one.

### DEFAULT_EXTERNAL_SORT_KEY

```rust
pub const DEFAULT_EXTERNAL_SORT_KEY: u32 = 200;
```

Host-side registry sort key for guests that leave `sortKey` at zero.
