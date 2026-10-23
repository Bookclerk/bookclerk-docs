**bookclerk_plugin_sdk > json**

# Module: json

## Contents

**Functions**

- [`decode`](#decode) - Deserialize a role-method JSON argument.
- [`encode`](#encode) - Serialize a role-method JSON result.
- [`encode_atomic_result`](#encode_atomic_result) - Encodes an atomic result; an oversized scalar after commit is `unavailable`.
- [`page_rows`](#page_rows) - Page a JSON array of rows. `limit == 0` means [`MAX_LIST_PAGE`](../bookclerk_plugin_abi/limits.md#bookclerk_plugin_abilimitsmax_list_page).

---

## bookclerk_plugin_sdk::json::decode

*Function*

Deserialize a role-method JSON argument.

Empty input is treated as `{}`.

# Errors

Returns `PluginError::invalid_params` when the payload is not valid JSON for `T`.

```rust
fn decode<T>(json: &str) -> Result<T, crate::PluginError>
```



## bookclerk_plugin_sdk::json::encode

*Function*

Serialize a role-method JSON result.

# Errors

Returns `PluginError::invalid_params` when serialization fails, or
`PluginError::payload_too_large` when the encoded size exceeds
[`MAX_SCALAR_BYTES`](../bookclerk_plugin_abi/limits.md#bookclerk_plugin_abilimitsmax_scalar_bytes).

```rust
fn encode<T>(value: T) -> Result<String, crate::PluginError>
```



## bookclerk_plugin_sdk::json::encode_atomic_result

*Function*

Encodes an atomic result; an oversized scalar after commit is `unavailable`.

# Errors

Returns `PluginError::payload_too_large` mapped to
`PluginError::unavailable` so the host retries the same `operationId`.

```rust
fn encode_atomic_result<T>(value: T) -> Result<String, crate::PluginError>
```



## bookclerk_plugin_sdk::json::page_rows

*Function*

Page a JSON array of rows. `limit == 0` means [`MAX_LIST_PAGE`](../bookclerk_plugin_abi/limits.md#bookclerk_plugin_abilimitsmax_list_page).

# Errors

Returns `PluginError::invalid_cursor` for a non-numeric cursor, or
`PluginError::payload_too_large` when the page exceeds [`MAX_SCALAR_BYTES`](../bookclerk_plugin_abi/limits.md#bookclerk_plugin_abilimitsmax_scalar_bytes).

```rust
fn page_rows<T>(rows: &[T], cursor: &str, limit: u32) -> Result<bookclerk_plugin_abi::QueryPage, crate::PluginError>
```



