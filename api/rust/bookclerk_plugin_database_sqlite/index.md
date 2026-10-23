# bookclerk_plugin_database_sqlite

Platform SQLite library database plugin.

Default `[database]` backend for Bookclerk: opens `library.db` under
`$BOOKCLERK_FILES_DIR` and exposes a SeaORM proxy. The host applies schema
after connect. Prefer [`open_store`](sqlite.md#open_store) from tests; guests speak the DB ABI
through `bookclerk-plugin-sdk`.

## Re-exports

- [`sqlite::drop_unit_files`](sqlite.md#drop_unit_files)
- [`sqlite::open`](sqlite.md#open)
- [`sqlite::open_memory_unmigrated`](sqlite.md#open_memory_unmigrated)
- [`sqlite::open_memory`](sqlite.md#open_memory)
- [`sqlite::open_store`](sqlite.md#open_store)
- [`sqlite::open_store_memory`](sqlite.md#open_store_memory)

## Modules

- [`plugin`](plugin.md): Database factory for the platform SQLite guest.
- [`sqlite`](sqlite.md): Local SQLite engine for the database plugin (rusqlite SeaORM proxy).

## Constants

### ID

```rust
pub const ID: &str = "sqlite";
```

Plugin id advertised in describe and `plugin.toml`.
