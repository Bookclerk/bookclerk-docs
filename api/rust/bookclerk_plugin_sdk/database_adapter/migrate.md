# bookclerk_plugin_sdk::database_adapter::migrate

Generic SQL helpers for database guests.

Guests may run host-provided typed statements. They must not select
Bookclerk schema versions or import `bookclerk_library::migrations`.
Schema packs arrive as already-separate statements — do not split on `;`.

## Functions

### typed_null

```rust
pub fn typed_null(decl_type: Option<&str>, column: &str) -> Value
```

Typed SQL `NULL` for proxy columns so SeaORM `Option<T>` decoding works.

#### Arguments

* `decl_type` - Declared column type when the engine provides one.
* `column` - Column name used when `decl_type` is missing.
