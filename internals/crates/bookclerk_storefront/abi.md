# bookclerk_storefront::abi

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
| [`FetchConversionError`](#fetchconversionerror) | enum | Why a fetch the guest answered cannot become a host `PlainFetch`. |
| [`abridgement_from_flag`](#abridgement_from_flag) | function | Tri-state abridgement flag ↔ the host's `Option<bool>`. |
| [`abridgement_to_flag`](#abridgement_to_flag) | function | Host `Option<bool>` view of the wire tri-state abridgement flag. |
| [`catalog_field_to_abi`](#catalog_field_to_abi) | function | Wire facet for an optional host search field (`None` → `any`). |
| [`catalog_field_from_abi`](#catalog_field_from_abi) | function | Host search field for a wire facet (`any` → `None`). |
| [`expand_candidates_params`](#expand_candidates_params) | function | Builds `expandCandidates` params from a seed plus the result cap. |
| [`expand_seed_from_params`](#expand_seed_from_params) | function | Splits `expandCandidates` params into the host seed and result cap. |
| [`login_params`](#login_params) | function | Builds guest login params. `plugin_data_dir` is the jail-granted data directory; the host fills `callback_ipc` / `callback_public_base` after it starts the OAuth callback proxy. |
| [`parse_rfc3339`](#parse_rfc3339) | function | RFC 3339 wire timestamp as UTC; `None` when it does not parse. |
| [`rfc3339`](#rfc3339) | function | RFC 3339 wire form of a host timestamp, for `ScanBook` date fields. |
| [`store_page_from_cursor`](#store_page_from_cursor) | function | Store page a numbered store cursor names: page 1 for the empty store cursor that starts a pass, otherwise the page `store_page_cursor` wrote. |
| [`store_page_cursor`](#store_page_cursor) | function | Store cursor naming the store page after this one, read back by `store_page_from_cursor`; empty when `next` is `None` (the last page). |
| [`scan_credential`](#scan_credential) | function | The pass's credential document, checked against the account it scans. |
| [`credentials_to_bytes`](#credentials_to_bytes) | function | Encodes a credential document as the opaque bytes the ABI carries. |
| [`credentials_from_bytes`](#credentials_from_bytes) | function | Decodes an opaque credential blob into the JSON document guests store. |
| [`DEFAULT_LIST_DEALS_LIMIT`](#default_list_deals_limit) | constant | `limit` sent for the host's `ContentSource::list_deals` when the caller omits one. |
| [`DEFAULT_EXTERNAL_SORT_KEY`](#default_external_sort_key) | constant | Host-side registry sort key for guests that leave `sortKey` at zero. |

## Enums

### FetchConversionError

```rust
pub enum FetchConversionError {
    CarrierNotAcquired(MediaKind),
}
```

Why a fetch the guest answered cannot become a host [`PlainFetch`](README.md#plainfetch).

**Variants**

- `CarrierNotAcquired`: The host acquires audio editions only today; the fetch carried
  another carrier.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

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

### parse_rfc3339

```rust
pub fn parse_rfc3339(raw: &str) -> Option<DateTime<Utc>>
```

RFC 3339 wire timestamp as UTC; `None` when it does not parse.

### rfc3339

```rust
pub fn rfc3339(at: DateTime<Utc>) -> String
```

RFC 3339 wire form of a host timestamp, for `ScanBook` date fields.

Sub-second precision is kept when the timestamp has it, so a round
trip through the wire form returns the same instant.

### store_page_from_cursor

```rust
pub fn store_page_from_cursor(store_cursor: &str) -> Result<u32, PluginError>
```

Store page a numbered store cursor names: page 1 for the empty store
cursor that starts a pass, otherwise the page [`store_page_cursor`](#store_page_cursor) wrote.

Storefronts with numbered library pages hand this to the SDK's
`scan_page` helper as their own store cursor; the helper wraps it in the
scan cursor the host sees.

#### Errors

Returns an `invalid_cursor` [`abi::PluginError`](../bookclerk_plugin_abi/error.md#pluginerror) when the store cursor is
not one [`store_page_cursor`](#store_page_cursor) wrote (a page below 2 included), so the
host restarts the pass.

### store_page_cursor

```rust
pub fn store_page_cursor(next: Option<u32>) -> String
```

Store cursor naming the store page after this one, read back by
[`store_page_from_cursor`](#store_page_from_cursor); empty when `next` is `None` (the last page).

### scan_credential

```rust
pub fn scan_credential(params: &ScanParams) -> Result<Value>
```

The pass's credential document, checked against the account it scans.

#### Errors

Returns [`crate::SourceError::Auth`](README.md#sourceerror) when the host sent no credential,
a credential for another account, or a blob that is not JSON.

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

## Constants

### DEFAULT_LIST_DEALS_LIMIT

```rust
pub const DEFAULT_LIST_DEALS_LIMIT: u32 = 20;
```

`limit` sent for the host's `ContentSource::list_deals` when the caller omits one.

### DEFAULT_EXTERNAL_SORT_KEY

```rust
pub const DEFAULT_EXTERNAL_SORT_KEY: u32 = 200;
```

Host-side registry sort key for guests that leave `sortKey` at zero.
