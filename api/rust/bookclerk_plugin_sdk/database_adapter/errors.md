# bookclerk_plugin_sdk::database_adapter::errors

Engine error classification for SQLite and PostgreSQL guests.

Classifies by SQLSTATE / rusqlite codes first. English substrings such as
`"unique"` are not the primary signal. COMMIT-time and connection-loss
failures are `unavailable` so the host retries the same `operationId`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`plugin_error_from_engine`](#plugin_error_from_engine) | function | Maps an engine / library failure onto a structured `PluginError`. |
| [`plugin_error_from_db_err`](#plugin_error_from_db_err) | function | Maps a SeaORM `DbErr` using typed driver codes when present. |

## Functions

### plugin_error_from_engine

```rust
pub fn plugin_error_from_engine(err: impl Display) -> PluginError
```

Maps an engine / library failure onto a structured [`PluginError`](../../bookclerk_plugin_abi/error.md#pluginerror).

#### Arguments

* `err` - Display of `DbErr`, `LibraryError`, or a guest string.

### plugin_error_from_db_err

```rust
pub fn plugin_error_from_db_err(err: &DbErr) -> PluginError
```

Maps a SeaORM [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) using typed driver codes when present.

PostgreSQL SQLSTATE comes from `sqlx::DatabaseError::code()`. SQLite
guests format `SQLITE_*` into `DbErr::Custom` before this mapper runs.
