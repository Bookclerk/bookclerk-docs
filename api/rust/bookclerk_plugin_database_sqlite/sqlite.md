# bookclerk_plugin_database_sqlite::sqlite

Local SQLite engine for the database plugin (rusqlite SeaORM proxy).

| Item | Kind | Summary |
| --- | --- | --- |
| [`SqliteProxy`](#sqliteproxy) | struct | SeaORM proxy over a shared rusqlite connection. |
| [`open`](#open) | function | Opens a SQLite file and returns a SeaORM proxy (no schema application). |
| [`drop_unit_files`](#drop_unit_files) | function | Deletes a SQLite binding unit and journal sidecars. Missing files are success. |
| [`open_memory`](#open_memory) | function | Opens an in-memory SQLite database, applies host schema, and returns a SeaORM proxy. |
| [`open_memory_unmigrated`](#open_memory_unmigrated) | function | Opens in-memory SQLite without applying schema (guest-shaped connect). |
| [`open_store`](#open_store) | function | Opens SQLite at `path`, applies host schema, and wraps it as a `LibraryStore`. |
| [`open_store_memory`](#open_store_memory) | function | Opens an in-memory `LibraryStore` for tests. |

## Structs

### SqliteProxy

```rust
pub struct SqliteProxy {
    /* private fields */
}
```

SeaORM proxy over a shared rusqlite connection.

#### SqliteProxy::new

```rust
pub fn new(conn: Connection) -> Result<Self>
```

Wraps an already-opened rusqlite connection for SeaORM proxy queries.

Call after the host applies schema (see [`open`](#open) / [`open_memory`](#open_memory)).

##### Errors

Returns when this connection refuses the query-deadline progress
handler. rusqlite 0.38+ returns that error when the handle is not owned
by this [`Connection`](https://docs.rs/rusqlite/latest/rusqlite/struct.Connection.html), which would leave statements without a deadline.

Implements: `Debug`, `ProxyDatabaseTrait`

## Functions

### open

```rust
pub async fn open(path: &Path) -> Result<DatabaseConnection, DbErr>
```

Opens a SQLite file and returns a SeaORM proxy (no schema application).

The host applies DDL after `openSession` + capability negotiation.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the parent directory cannot be created, the file
cannot be opened, the query-deadline handler cannot be installed, or the
SeaORM proxy cannot connect.

### drop_unit_files

```rust
pub fn drop_unit_files(unit_ref: &str) -> Result<(), DbErr>
```

Deletes a SQLite binding unit and journal sidecars. Missing files are success.

Called from the sqlite adapter's `Database.dropUnit` implementation so the
host does not link this crate to unlink plugin-database files.

#### Errors

Returns when a sidecar exists but cannot be removed.

### open_memory

```rust
pub async fn open_memory() -> Result<DatabaseConnection>
```

Opens an in-memory SQLite database, applies host schema, and returns a SeaORM proxy.

Intended for unit tests and dry-run paths (no durable file). Version
selection lives in [`bookclerk_library::apply_host_schema`](../bookclerk_library/index.md#apply_host_schema).

#### Errors

Returns [`bookclerk_library::LibraryError`](../bookclerk_library/index.md#libraryerror) when the SeaORM proxy or host schema fails.

### open_memory_unmigrated

```rust
pub async fn open_memory_unmigrated() -> Result<DatabaseConnection, DbErr>
```

Opens in-memory SQLite without applying schema (guest-shaped connect).

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the in-memory database cannot be opened, the
query-deadline handler cannot be installed, or the SeaORM proxy fails.

### open_store

```rust
pub async fn open_store(path: &Path) -> Result<LibraryStore>
```

Opens SQLite at `path`, applies host schema, and wraps it as a [`LibraryStore`](../bookclerk_library/index.md#librarystore).

Prefer this entry point from CLI / tests that need the high-level library
API rather than a raw [`DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html). Production guests use [`open`](#open).

#### Arguments

* `path` - Absolute path to the SQLite database file.

#### Errors

Propagates errors from [`open`](#open) or host schema application.

### open_store_memory

```rust
pub async fn open_store_memory() -> Result<LibraryStore>
```

Opens an in-memory [`LibraryStore`](../bookclerk_library/index.md#librarystore) for tests.

#### Errors

Propagates errors from [`open_memory`](#open_memory).
