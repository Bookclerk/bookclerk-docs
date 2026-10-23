# bookclerk_plugin_database_d1::atomic

Generic D1 HTTP `batch()` executor for host-authored SQL plans.

The guest does not parse Bookclerk operation names. Production and tests
use [`D1Proxy::run_typed_atomic`](d1.md#d1proxyrun_typed_atomic) (`ExecuteRequest`).

| Item | Kind | Summary |
| --- | --- | --- |
| [`is_ambiguous_d1`](#is_ambiguous_d1) | function | True when `err` is a D1 ambiguous-commit marker. |
| [`plugin_error_from_d1`](#plugin_error_from_d1) | function | Maps a D1 `DbErr` onto the guest ABI: retryable/ambiguous → `unavailable`, client 4xx → `invalid_params`, engine codes via the shared mapper. |

## Functions

### is_ambiguous_d1

```rust
pub fn is_ambiguous_d1(err: &DbErr) -> bool
```

True when `err` is a D1 ambiguous-commit marker.

### plugin_error_from_d1

```rust
pub fn plugin_error_from_d1(err: DbErr) -> PluginError
```

Maps a D1 [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) onto the guest ABI: retryable/ambiguous → `unavailable`,
client 4xx → `invalid_params`, engine codes via the shared mapper.
