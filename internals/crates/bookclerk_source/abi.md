# bookclerk_source::abi

Conversions between host storefront types and the typed Cap'n Proto plugin
ABI ([`bookclerk_plugin_abi`](../bookclerk_plugin_abi/README.md)).

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
| [`scanned_title`](#scanned_title) | function | The title a scan record describes: its flat row and its listing. |
| [`scan_book_to_new`](#scan_book_to_new) | function | Maps a guest scan row onto the flat `NewBook` row the library stores until it keeps works, expressions and editions, forcing `source` to the plugin id so a guest can never claim another storefront's rows. The owning `plugin_key` is stamped by the `bookclerk_library::SourceScope` on write. |
| [`rfc3339`](#rfc3339) | function | RFC 3339 wire form of a host timestamp, for `ScanBook` date fields. |
| [`scan_summary`](#scan_summary) | function | Builds a guest scan summary from the books a scan produced. |
| [`scan_summary_from_abi`](#scan_summary_from_abi) | function | Host counters from a guest scan summary. |
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

### scanned_title

```rust
pub fn scanned_title(plugin_id: &str, book: ScanBook) -> Option<ScannedTitle>
```

The title a scan record describes: its flat row and its listing.

The listing is `None` when the record fails listing validation (the
flat row is still stored and its own listing derived from it); see
[`scan_book_to_new`](#scan_book_to_new) for when no title is returned at all.

#### Arguments

* `plugin_id` - The plugin that produced the record; it owns the listing.
* `book` - The scan record as the guest returned it.

### scan_book_to_new

```rust
pub fn scan_book_to_new(plugin_id: &str, book: ScanBook) -> Option<NewBook>
```

Maps a guest scan row onto the flat [`NewBook`](../bookclerk_library/README.md#newbook) row the library stores
until it keeps works, expressions and editions, forcing `source` to the
plugin id so a guest can never claim another storefront's rows. The
owning `plugin_key` is stamped by the [`bookclerk_library::SourceScope`](../bookclerk_library/scope.md#sourcescope)
on write.

Credits join by role, the first ISBN and ASIN fill the identifier
columns (normalised), the first series entry fills the series columns,
and the work kind becomes `content_kind`. An episode with no store
series id names its show through `parent_product_id`, which fills
`series_asin` so the episode is filed under the show.

The ASIN columns take an ASIN scoped to the book's marketplace, or an
unscoped one, before an ASIN from another marketplace.

#### Returns

`None` for a record with a blank product id, which would overwrite
another blank row, and for an ebook or print edition: the flat table has
no way to keep those out of acquire, so they are not stored until the
library keeps editions.

### rfc3339

```rust
pub fn rfc3339(at: DateTime<Utc>) -> String
```

RFC 3339 wire form of a host timestamp, for `ScanBook` date fields.

Sub-second precision is kept when the timestamp has it, so a round
trip through the wire form returns the same instant.

### scan_summary

```rust
pub fn scan_summary(books: Vec<ScanBook>, accounts: usize, pages: u32) -> ScanSummary
```

Builds a guest scan summary from the books a scan produced.

### scan_summary_from_abi

```rust
pub fn scan_summary_from_abi(summary: &ScanSummary, upserted: usize) -> ScanSummary
```

Host counters from a guest scan summary.

`books_upserted` is always the host's own count of the rows it wrote, so
records the host skipped (ebook and print editions, blank product ids)
never count, even when the guest reported them.

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

Returns [`crate::SourceError::Auth`](README.md#sourceerror) when the bytes are not JSON.

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

Returns [`crate::SourceError::Auth`](README.md#sourceerror) when any blob is not JSON.

## Constants

### DEFAULT_LIST_DEALS_LIMIT

```rust
pub const DEFAULT_LIST_DEALS_LIMIT: u32 = 20;
```

`limit` sent for [`crate::ContentSource::list_deals`](README.md#contentsourcelist_deals) when the caller omits one.

### DEFAULT_EXTERNAL_SORT_KEY

```rust
pub const DEFAULT_EXTERNAL_SORT_KEY: u32 = 200;
```

Host-side registry sort key for guests that leave `sortKey` at zero.
