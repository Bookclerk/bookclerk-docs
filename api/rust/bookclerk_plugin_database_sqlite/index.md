# bookclerk_plugin_database_sqlite

Platform SQLite library database plugin.

Default `[database]` backend for Bookclerk: opens `library.db` under
`$BOOKCLERK_FILES_DIR` and exposes a SeaORM proxy. The host applies schema
after connect. Prefer [`open_store`](sqlite.md#bookclerk_plugin_database_sqlitesqliteopen_store) from tests; guests speak the DB ABI
through `bookclerk-plugin-sdk`.

## Modules

### [`bookclerk_plugin_database_sqlite`](bookclerk_plugin_database_sqlite.md)

*1 constant, 2 modules*

### [`plugin`](plugin.md)

*1 struct*

### [`sqlite`](sqlite.md)

*1 struct, 6 functions*

