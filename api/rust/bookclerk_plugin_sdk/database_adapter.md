# bookclerk_plugin_sdk::database_adapter

Database adapter helpers for third-party database plugin authors.

## Audience

Rust authors building a **database** guest (`kind = "database"`). Use this
module with the `db` feature enabled on `bookclerk-plugin-sdk`:

```toml
[dependencies]
bookclerk-plugin-sdk = { version = "0.1", features = ["db"] }
```

## What to use

| Need | Entry point |
| --- | --- |
| Engine error → structured [`crate::PluginError`](../bookclerk_plugin_abi/error.md#pluginerror) | [`plugin_error_from_engine`](database_adapter/errors.md#plugin_error_from_engine), [`plugin_error_from_db_err`](database_adapter/errors.md#plugin_error_from_db_err) |
| Typed SQL `NULL` for proxy row decode | [`typed_null`](database_adapter/migrate.md#typed_null) |

First-party platform database guests use the internal `bookclerk-db-guest`
crate for SeaORM session workers; those helpers are not part of the stable
third-party author API.

## Re-exports

- [`database_adapter::errors::plugin_error_from_db_err`](database_adapter/errors.md#plugin_error_from_db_err)
- [`database_adapter::errors::plugin_error_from_engine`](database_adapter/errors.md#plugin_error_from_engine)
- [`database_adapter::migrate::typed_null`](database_adapter/migrate.md#typed_null)

## Modules

- [`errors`](database_adapter/errors.md): Engine error classification for SQLite and PostgreSQL guests.
- [`migrate`](database_adapter/migrate.md): Generic SQL helpers for database guests.
