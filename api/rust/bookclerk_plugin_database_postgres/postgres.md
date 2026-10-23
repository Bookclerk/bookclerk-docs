# bookclerk_plugin_database_postgres::postgres

PostgreSQL engine for the database plugin (SeaORM sqlx-postgres).

| Item | Kind | Summary |
| --- | --- | --- |
| [`open`](#open) | function | Open Postgres with a host-mediated connection URL (ping only; host applies schema). |
| [`postgres_tcp_target`](#postgres_tcp_target) | function | TCP host/port a Postgres URL would dial (`None` for Unix-socket URLs). |
| [`postgres_url_with_database`](#postgres_url_with_database) | function | Rewrites the database name in a Postgres URL, preserving query options. |
| [`open_binding`](#open_binding) | function | Open a dedicated per-binding **database** (not a schema on the library DB). |
| [`open_binding_existing`](#open_binding_existing) | function | Connects to an existing isolated binding database without creating it. |
| [`drop_binding`](#drop_binding) | function | Drops an isolated per-binding PostgreSQL database. |
| [`MIN_POSTGRES_MAJOR`](#min_postgres_major) | constant | Minimum PostgreSQL major BookclerkSQL adapters accept. |
| [`MIN_POSTGRES_VERSION_NUM`](#min_postgres_version_num) | constant | `server_version_num` floor (`160000` for PostgreSQL 16). |

## Functions

### open

```rust
pub async fn open(url: &str) -> Result<DatabaseConnection, DbErr>
```

Open Postgres with a host-mediated connection URL (ping only; host applies schema).

#### Errors

Returns an error when the operation fails, the server is older than
[`MIN_POSTGRES_MAJOR`](#min_postgres_major), or the session encoding is not UTF8.

### postgres_tcp_target

```rust
pub fn postgres_tcp_target(url: &str) -> Option<(String, u16)>
```

TCP host/port a Postgres URL would dial (`None` for Unix-socket URLs).

Query `host` / `hostaddr` / `port` override the URL authority. A `host`
that starts with `/` is a libpq directory for `.s.PGSQL.{port}`.

### postgres_url_with_database

```rust
pub fn postgres_url_with_database(url: &str, database: &str) -> String
```

Rewrites the database name in a Postgres URL, preserving query options.

### open_binding

```rust
pub async fn open_binding(url: &str, database: &str) -> Result<DatabaseConnection, DbErr>
```

Open a dedicated per-binding **database** (not a schema on the library DB).

Creates the database when missing, then connects to it. Plugin SQL cannot
see host library tables in another database, even via `CREATE TABLE AS`
or `REFERENCES public.books`.

#### Errors

Returns an error when the name is unsafe, `CREATE DATABASE` fails (the
role needs `CREATEDB`), the binding connection cannot ping, or the
binding session is older than [`MIN_POSTGRES_MAJOR`](#min_postgres_major) or not UTF8.

### open_binding_existing

```rust
pub async fn open_binding_existing(
    url: &str,
    database: &str,
) -> Result<DatabaseConnection, DbErr>
```

Connects to an existing isolated binding database without creating it.

#### Errors

Returns when the name is unsafe, the database is missing, ping fails, or
the binding session is older than [`MIN_POSTGRES_MAJOR`](#min_postgres_major) or not UTF8.

### drop_binding

```rust
pub async fn drop_binding(url: &str, database: &str) -> Result<(), DbErr>
```

Drops an isolated per-binding PostgreSQL database.

Connects to the admin URL (not `database`) so `DROP DATABASE` is legal.
Other backends attached to the target are terminated first. Missing
databases are success (`IF EXISTS`) so the registry row can be removed.

#### Errors

Returns when the name is unsafe or `DROP DATABASE` fails.

## Constants

### MIN_POSTGRES_MAJOR

```rust
pub const MIN_POSTGRES_MAJOR: u32 = 16;
```

Minimum PostgreSQL major BookclerkSQL adapters accept.

### MIN_POSTGRES_VERSION_NUM

```rust
pub const MIN_POSTGRES_VERSION_NUM: u32 = _;
```

`server_version_num` floor (`160000` for PostgreSQL 16).
