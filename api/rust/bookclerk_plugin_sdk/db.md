**bookclerk_plugin_sdk > db**

# Module: db

## Contents

**Functions**

- [`db_value_from_sea`](#db_value_from_sea) - Fallible SeaORM → [`bookclerk_plugin_abi::DbValue`](../bookclerk_plugin_abi/db_value.md#bookclerk_plugin_abidb_valuedbvalue) bridge.
- [`proxy_rows_from_typed`](#proxy_rows_from_typed) - Builds SeaORM proxy rows from a typed statement result.

---

## bookclerk_plugin_sdk::db::db_value_from_sea

*Function*

Fallible SeaORM → [`bookclerk_plugin_abi::DbValue`](../bookclerk_plugin_abi/db_value.md#bookclerk_plugin_abidb_valuedbvalue) bridge.

# Errors

Returns a static reason when the SeaORM value is outside the universal
domain, or TEXT contains U+0000.

```rust
fn db_value_from_sea(v: &sea_orm::Value) -> Result<bookclerk_plugin_abi::DbValue, String>
```



## bookclerk_plugin_sdk::db::proxy_rows_from_typed

*Function*

Builds SeaORM proxy rows from a typed statement result.

# Errors

Returns when the result is not positional.

```rust
fn proxy_rows_from_typed(stmt: &bookclerk_plugin_abi::StatementResult) -> Result<Vec<sea_orm::ProxyRow>, String>
```



