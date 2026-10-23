# bookclerk_plugin_database_d1

Optional Cloudflare D1 library database plugin.

Alternate `[database]` backend for remote D1. Hosts call [`open`](d1.md#bookclerk_plugin_database_d1d1open) after
operator config supplies account/database credentials; see `docs/database.md`.
Atomic library writes use `D1Proxy::run_typed_atomic` rather than
interactive `BEGIN`.

## Modules

### [`bookclerk_plugin_database_d1`](bookclerk_plugin_database_d1.md)

*2 modules*

### [`atomic`](atomic.md)

*2 functions*

### [`d1`](d1.md)

*1 struct, 6 functions*

