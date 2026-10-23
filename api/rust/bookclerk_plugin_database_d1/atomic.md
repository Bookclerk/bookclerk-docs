**bookclerk_plugin_database_d1 > atomic**

# Module: atomic

## Contents

**Functions**

- [`is_ambiguous_d1`](#is_ambiguous_d1) - True when `err` is a D1 ambiguous-commit marker.
- [`plugin_error_from_d1`](#plugin_error_from_d1) - Maps a D1 [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) onto the guest ABI: retryable/ambiguous → `unavailable`,

---

## bookclerk_plugin_database_d1::atomic::is_ambiguous_d1

*Function*

True when `err` is a D1 ambiguous-commit marker.

```rust
fn is_ambiguous_d1(err: &sea_orm::DbErr) -> bool
```



## bookclerk_plugin_database_d1::atomic::plugin_error_from_d1

*Function*

Maps a D1 [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) onto the guest ABI: retryable/ambiguous → `unavailable`,
client 4xx → `invalid_params`, engine codes via the shared mapper.

```rust
fn plugin_error_from_d1(err: sea_orm::DbErr) -> bookclerk_plugin_sdk::PluginError
```



