# bookclerk_db_guest

First-party database guest internals (SeaORM session workers).

Third-party database plugin authors should depend on
[`bookclerk_plugin_sdk::database_adapter`](../bookclerk_plugin_sdk/database_adapter.md) only. Platform SQLite / Postgres /
D1 guests use this crate for host-mediated session and transaction workers.

## Modules

### [`host_session`](host_session.md)

*2 functions, 2 structs*

### [`session`](session.md)

*39 functions*

### [`sql`](sql.md)

*1 function, 4 structs*

