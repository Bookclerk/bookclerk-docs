**bookclerk_plugin_sdk > database_adapter > migrate**

# Module: database_adapter::migrate

## Contents

**Functions**

- [`typed_null`](#typed_null) - Typed SQL `NULL` for proxy columns so SeaORM `Option<T>` decoding works.

---

## bookclerk_plugin_sdk::database_adapter::migrate::typed_null

*Function*

Typed SQL `NULL` for proxy columns so SeaORM `Option<T>` decoding works.

# Arguments

* `decl_type` - Declared column type when the engine provides one.
* `column` - Column name used when `decl_type` is missing.

```rust
fn typed_null(decl_type: Option<&str>, column: &str) -> sea_orm::Value
```



