# bookclerk_plugin_database_postgres

Optional PostgreSQL library database plugin.

Alternate `[database]` backend for a networked Postgres instance. Hosts call
[`open`](postgres.md#open) with the configured connection URL; see `docs/database.md`.

## Re-exports

- [`postgres::drop_binding`](postgres.md#drop_binding)
- [`postgres::open`](postgres.md#open)
- [`postgres::open_binding`](postgres.md#open_binding)
- [`postgres::open_binding_existing`](postgres.md#open_binding_existing)
- [`postgres::postgres_tcp_target`](postgres.md#postgres_tcp_target)
- [`postgres::MIN_POSTGRES_MAJOR`](postgres.md#min_postgres_major)
- [`postgres::MIN_POSTGRES_VERSION_NUM`](postgres.md#min_postgres_version_num)

## Modules

- [`postgres`](postgres.md): PostgreSQL engine for the database plugin (SeaORM sqlx-postgres).
