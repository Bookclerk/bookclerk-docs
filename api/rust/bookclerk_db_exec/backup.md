**bookclerk_db_exec > backup**

# Module: backup

## Contents

**Functions**

- [`apply_begin_isolation`](#apply_begin_isolation) - Realizes [`IsolationReq`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeisolationreq) on an already-open transaction.
- [`assert_restore_constraints`](#assert_restore_constraints) - SQLite: `PRAGMA foreign_key_check` must be empty before commit.
- [`begin_consistent_snapshot`](#begin_consistent_snapshot) - Begins a consistent capture transaction.
- [`drop_user_relations`](#drop_user_relations) - Drops named relations. PostgreSQL uses `CASCADE` plus identity functions.
- [`export_identity`](#export_identity) - Identity high-water from `sqlite_sequence` and [`SQL_IDENTITY_TABLE`](../bookclerk_plugin_abi/sql_types.md#bookclerk_plugin_abisql_typessql_identity_table).
- [`import_identity`](#import_identity) - Writes identity high-water into adapter catalogs.
- [`list_user_relations`](#list_user_relations) - User-visible base tables in the current schema.
- [`pending_begin_isolation`](#pending_begin_isolation) - Isolation the next SeaORM `begin()` should ask the adapter to realize.
- [`prepare_unit_restore`](#prepare_unit_restore) - SQLite: `PRAGMA defer_foreign_keys = ON` on the restore transaction.

---

## bookclerk_db_exec::backup::apply_begin_isolation

*Function*

Realizes [`IsolationReq`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeisolationreq) on an already-open transaction.

# Errors

Returns when the engine rejects the isolation statement.

```rust
fn apply_begin_isolation<C>(conn: &C, isolation: bookclerk_plugin_abi::IsolationReq) -> Result<(), sea_orm::DbErr>
```



## bookclerk_db_exec::backup::assert_restore_constraints

*Function*

SQLite: `PRAGMA foreign_key_check` must be empty before commit.

# Errors

Returns when violations remain or the pragma fails.

```rust
fn assert_restore_constraints<C>(conn: &C) -> Result<(), sea_orm::DbErr>
```



## bookclerk_db_exec::backup::begin_consistent_snapshot

*Function*

Begins a consistent capture transaction.

Sets `IsolationReq::ConsistentSnapshot` for RPC proxies, and applies
`SET TRANSACTION ISOLATION LEVEL REPEATABLE READ` on a native Postgres
connection (the host RPC proxy always reports the canonical sqlite backend).

# Errors

Returns when `BEGIN` or snapshot isolation fails.

```rust
fn begin_consistent_snapshot(db: &sea_orm::DatabaseConnection) -> Result<sea_orm::DatabaseTransaction, sea_orm::DbErr>
```



## bookclerk_db_exec::backup::drop_user_relations

*Function*

Drops named relations. PostgreSQL uses `CASCADE` plus identity functions.

# Errors

Returns when a drop fails (other than IF EXISTS no-ops).

```rust
fn drop_user_relations<C>(conn: &C, names: &[String]) -> Result<(), sea_orm::DbErr>
```



## bookclerk_db_exec::backup::export_identity

*Function*

Identity high-water from `sqlite_sequence` and [`SQL_IDENTITY_TABLE`](../bookclerk_plugin_abi/sql_types.md#bookclerk_plugin_abisql_typessql_identity_table).

# Errors

Returns when a catalog query fails for a reason other than a missing catalog.

```rust
fn export_identity<C>(conn: &C) -> Result<Vec<bookclerk_plugin_abi::DbIdentityHighWater>, sea_orm::DbErr>
```



## bookclerk_db_exec::backup::import_identity

*Function*

Writes identity high-water into adapter catalogs.

# Errors

Returns when a catalog write fails.

```rust
fn import_identity<C>(conn: &C, rows: &[bookclerk_plugin_abi::DbIdentityHighWater]) -> Result<(), sea_orm::DbErr>
```



## bookclerk_db_exec::backup::list_user_relations

*Function*

User-visible base tables in the current schema.

# Errors

Returns when the catalog probe fails.

```rust
fn list_user_relations<C>(conn: &C) -> Result<Vec<String>, sea_orm::DbErr>
```



## bookclerk_db_exec::backup::pending_begin_isolation

*Function*

Isolation the next SeaORM `begin()` should ask the adapter to realize.

```rust
fn pending_begin_isolation() -> bookclerk_plugin_abi::IsolationReq
```



## bookclerk_db_exec::backup::prepare_unit_restore

*Function*

SQLite: `PRAGMA defer_foreign_keys = ON` on the restore transaction.

# Errors

Returns when the pragma fails.

```rust
fn prepare_unit_restore<C>(conn: &C) -> Result<(), sea_orm::DbErr>
```



