# bookclerk_plugin_database_d1

Optional Cloudflare D1 library database plugin.

Alternate `[database]` backend for remote D1. Hosts call [`open`](d1.md#open) after
operator config supplies account/database credentials; see `docs/database.md`.
Atomic library writes use [`D1Proxy::run_typed_atomic`](d1.md#d1proxyrun_typed_atomic) rather than
interactive `BEGIN`.

## Re-exports

- [`d1::delete_database`](d1.md#delete_database)
- [`d1::ensure_database`](d1.md#ensure_database)
- [`d1::lookup_database`](d1.md#lookup_database)
- [`d1::open`](d1.md#open)
- [`d1::set_shared_proxy`](d1.md#set_shared_proxy)
- [`d1::shared_proxy`](d1.md#shared_proxy)
- [`d1::D1Proxy`](d1.md#d1proxy)

## Modules

- [`atomic`](atomic.md): Generic D1 HTTP `batch()` executor for host-authored SQL plans.
- [`d1`](d1.md): Cloudflare D1 HTTP API proxy for SeaORM.
