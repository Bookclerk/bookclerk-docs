**bookclerk_plugin_database_postgres > postgres**

# Module: postgres

## Contents

**Functions**

- [`drop_binding`](#drop_binding) - Drops an isolated per-binding PostgreSQL database.
- [`open`](#open) - Open Postgres with a host-mediated connection URL (ping only; host applies schema).
- [`open_binding`](#open_binding) - Open a dedicated per-binding **database** (not a schema on the library DB).
- [`open_binding_existing`](#open_binding_existing) - Connects to an existing isolated binding database without creating it.
- [`postgres_tcp_target`](#postgres_tcp_target) - TCP host/port a Postgres URL would dial (`None` for Unix-socket URLs).
- [`postgres_url_with_database`](#postgres_url_with_database) - Rewrites the database name in a Postgres URL, preserving query options.

**Constants**

- [`MIN_POSTGRES_MAJOR`](#min_postgres_major) - Minimum PostgreSQL major BookclerkSQL adapters accept.
- [`MIN_POSTGRES_VERSION_NUM`](#min_postgres_version_num) - `server_version_num` floor (`160000` for PostgreSQL 16).

---

## bookclerk_plugin_database_postgres::postgres::MIN_POSTGRES_MAJOR

*Constant*: `u32`

Minimum PostgreSQL major BookclerkSQL adapters accept.



## bookclerk_plugin_database_postgres::postgres::MIN_POSTGRES_VERSION_NUM

*Constant*: `u32`

`server_version_num` floor (`160000` for PostgreSQL 16).



## bookclerk_plugin_database_postgres::postgres::drop_binding

*Function*

Drops an isolated per-binding PostgreSQL database.

Connects to the admin URL (not `database`) so `DROP DATABASE` is legal.
Other backends attached to the target are terminated first. Missing
databases are success (`IF EXISTS`) so the registry row can be removed.

# Errors

Returns when the name is unsafe or `DROP DATABASE` fails.

```rust
fn drop_binding(url: &str, database: &str) -> std::result::Result<(), sea_orm::DbErr>
```



## bookclerk_plugin_database_postgres::postgres::open

*Function*

Open Postgres with a host-mediated connection URL (ping only; host applies schema).

# Errors

Returns an error when the operation fails, the server is older than
[`MIN_POSTGRES_MAJOR`](postgres.md#bookclerk_plugin_database_postgrespostgresmin_postgres_major), or the session encoding is not UTF8.

```rust
fn open(url: &str) -> std::result::Result<sea_orm::DatabaseConnection, sea_orm::DbErr>
```



## bookclerk_plugin_database_postgres::postgres::open_binding

*Function*

Open a dedicated per-binding **database** (not a schema on the library DB).

Creates the database when missing, then connects to it. Plugin SQL cannot
see host library tables in another database, even via `CREATE TABLE AS`
or `REFERENCES public.books`.

# Errors

Returns an error when the name is unsafe, `CREATE DATABASE` fails (the
role needs `CREATEDB`), the binding connection cannot ping, or the
binding session is older than [`MIN_POSTGRES_MAJOR`](postgres.md#bookclerk_plugin_database_postgrespostgresmin_postgres_major) or not UTF8.

```rust
fn open_binding(url: &str, database: &str) -> std::result::Result<sea_orm::DatabaseConnection, sea_orm::DbErr>
```



## bookclerk_plugin_database_postgres::postgres::open_binding_existing

*Function*

Connects to an existing isolated binding database without creating it.

# Errors

Returns when the name is unsafe, the database is missing, ping fails, or
the binding session is older than [`MIN_POSTGRES_MAJOR`](postgres.md#bookclerk_plugin_database_postgrespostgresmin_postgres_major) or not UTF8.

```rust
fn open_binding_existing(url: &str, database: &str) -> std::result::Result<sea_orm::DatabaseConnection, sea_orm::DbErr>
```



## bookclerk_plugin_database_postgres::postgres::postgres_tcp_target

*Function*

TCP host/port a Postgres URL would dial (`None` for Unix-socket URLs).

Query `host` / `hostaddr` / `port` override the URL authority. A `host`
that starts with `/` is a libpq directory for `.s.PGSQL.{port}`.

```rust
fn postgres_tcp_target(url: &str) -> Option<(String, u16)>
```



## bookclerk_plugin_database_postgres::postgres::postgres_url_with_database

*Function*

Rewrites the database name in a Postgres URL, preserving query options.

```rust
fn postgres_url_with_database(url: &str, database: &str) -> String
```



