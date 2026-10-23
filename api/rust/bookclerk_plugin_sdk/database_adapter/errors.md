**bookclerk_plugin_sdk > database_adapter > errors**

# Module: database_adapter::errors

## Contents

**Functions**

- [`plugin_error_from_db_err`](#plugin_error_from_db_err) - Maps a SeaORM [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) using typed driver codes when present.
- [`plugin_error_from_engine`](#plugin_error_from_engine) - Maps an engine / library failure onto a structured [`PluginError`](../../bookclerk_plugin_abi/error.md#bookclerk_plugin_abierrorpluginerror).

---

## bookclerk_plugin_sdk::database_adapter::errors::plugin_error_from_db_err

*Function*

Maps a SeaORM [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) using typed driver codes when present.

PostgreSQL SQLSTATE comes from `sqlx::DatabaseError::code()`. SQLite
guests format `SQLITE_*` into `DbErr::Custom` before this mapper runs.

```rust
fn plugin_error_from_db_err(err: &sea_orm::DbErr) -> crate::PluginError
```



## bookclerk_plugin_sdk::database_adapter::errors::plugin_error_from_engine

*Function*

Maps an engine / library failure onto a structured [`PluginError`](../../bookclerk_plugin_abi/error.md#bookclerk_plugin_abierrorpluginerror).

# Arguments

* `err` - Display of `DbErr`, `LibraryError`, or a guest string.

```rust
fn plugin_error_from_engine<impl std::fmt::Display>(err: impl Trait) -> crate::PluginError
```



