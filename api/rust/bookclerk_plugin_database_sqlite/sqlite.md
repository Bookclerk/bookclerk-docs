**bookclerk_plugin_database_sqlite > sqlite**

# Module: sqlite

## Contents

**Structs**

- [`SqliteProxy`](#sqliteproxy) - SeaORM proxy over a shared rusqlite connection.

**Functions**

- [`drop_unit_files`](#drop_unit_files) - Deletes a SQLite binding unit and journal sidecars. Missing files are success.
- [`open`](#open) - Opens a SQLite file and returns a SeaORM proxy (no schema application).
- [`open_memory`](#open_memory) - Opens an in-memory SQLite database, applies host schema, and returns a SeaORM proxy.
- [`open_memory_unmigrated`](#open_memory_unmigrated) - Opens in-memory SQLite without applying schema (guest-shaped connect).
- [`open_store`](#open_store) - Opens SQLite at `path`, applies host schema, and wraps it as a [`LibraryStore`](../bookclerk_library/store.md#bookclerk_librarystorelibrarystore).
- [`open_store_memory`](#open_store_memory) - Opens an in-memory [`LibraryStore`](../bookclerk_library/store.md#bookclerk_librarystorelibrarystore) for tests.

---

## bookclerk_plugin_database_sqlite::sqlite::SqliteProxy

*Struct*

SeaORM proxy over a shared rusqlite connection.

**Methods:**

- `fn new(conn: Connection) -> rusqlite::Result<Self>` - Wraps an already-opened rusqlite connection for SeaORM proxy queries.

**Trait Implementations:**

- **ProxyDatabaseTrait**
  - `fn query(self: &'life0 Self, statement: Statement) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn execute(self: &'life0 Self, statement: Statement) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn ping(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn begin(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn commit(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn rollback(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn start_rollback(self: &Self)`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_plugin_database_sqlite::sqlite::drop_unit_files

*Function*

Deletes a SQLite binding unit and journal sidecars. Missing files are success.

Called from the sqlite adapter's `Database.dropUnit` implementation so the
host does not link this crate to unlink plugin-database files.

# Errors

Returns when a sidecar exists but cannot be removed.

```rust
fn drop_unit_files(unit_ref: &str) -> std::result::Result<(), sea_orm::DbErr>
```



## bookclerk_plugin_database_sqlite::sqlite::open

*Function*

Opens a SQLite file and returns a SeaORM proxy (no schema application).

The host applies DDL after `openSession` + capability negotiation.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the parent directory cannot be created, the file
cannot be opened, the query-deadline handler cannot be installed, or the
SeaORM proxy cannot connect.

```rust
fn open(path: &std::path::Path) -> std::result::Result<sea_orm::DatabaseConnection, sea_orm::DbErr>
```



## bookclerk_plugin_database_sqlite::sqlite::open_memory

*Function*

Opens an in-memory SQLite database, applies host schema, and returns a SeaORM proxy.

Intended for unit tests and dry-run paths (no durable file). Version
selection lives in [`bookclerk_library::apply_host_schema`](../bookclerk_library/host_schema.md#bookclerk_libraryhost_schemaapply_host_schema).

# Errors

Returns [`bookclerk_library::LibraryError`](../bookclerk_library/error.md#bookclerk_libraryerrorlibraryerror) when the SeaORM proxy or host schema fails.

```rust
fn open_memory() -> bookclerk_library::Result<sea_orm::DatabaseConnection>
```



## bookclerk_plugin_database_sqlite::sqlite::open_memory_unmigrated

*Function*

Opens in-memory SQLite without applying schema (guest-shaped connect).

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the in-memory database cannot be opened, the
query-deadline handler cannot be installed, or the SeaORM proxy fails.

```rust
fn open_memory_unmigrated() -> std::result::Result<sea_orm::DatabaseConnection, sea_orm::DbErr>
```



## bookclerk_plugin_database_sqlite::sqlite::open_store

*Function*

Opens SQLite at `path`, applies host schema, and wraps it as a [`LibraryStore`](../bookclerk_library/store.md#bookclerk_librarystorelibrarystore).

Prefer this entry point from CLI / tests that need the high-level library
API rather than a raw [`DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html). Production guests use [`open`](sqlite.md#bookclerk_plugin_database_sqlitesqliteopen).

# Arguments

* `path` - Absolute path to the SQLite database file.

# Errors

Propagates errors from [`open`](sqlite.md#bookclerk_plugin_database_sqlitesqliteopen) or host schema application.

```rust
fn open_store(path: &std::path::Path) -> bookclerk_library::Result<bookclerk_library::LibraryStore>
```



## bookclerk_plugin_database_sqlite::sqlite::open_store_memory

*Function*

Opens an in-memory [`LibraryStore`](../bookclerk_library/store.md#bookclerk_librarystorelibrarystore) for tests.

# Errors

Propagates errors from [`open_memory`](sqlite.md#bookclerk_plugin_database_sqlitesqliteopen_memory).

```rust
fn open_store_memory() -> bookclerk_library::Result<bookclerk_library::LibraryStore>
```



