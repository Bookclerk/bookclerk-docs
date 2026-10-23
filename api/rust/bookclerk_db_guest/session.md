**bookclerk_db_guest > session**

# Module: session

## Contents

**Functions**

- [`bootstrap_for`](#bootstrap_for) - Bootstrap metadata for an explicit connection's engine family.
- [`capabilities_for`](#capabilities_for) - Advertised capabilities for an explicit connection's engine family.
- [`guest_assert_restore_constraints`](#guest_assert_restore_constraints) - Assert restore constraints on the process-wide connection.
- [`guest_assert_restore_constraints_on`](#guest_assert_restore_constraints_on) - Assert restore constraints on `conn`.
- [`guest_assert_restore_constraints_on_txn`](#guest_assert_restore_constraints_on_txn) - Assert restore constraints on an open guest transaction.
- [`guest_begin`](#guest_begin) - Begins a top-level transaction or a nested savepoint.
- [`guest_begin_isolated`](#guest_begin_isolated) - Begins a top-level transaction realizing `isolation` (or a nested savepoint).
- [`guest_begin_on`](#guest_begin_on) - Begins a top-level transaction on an explicit connection (plugin bindings).
- [`guest_begin_on_isolated`](#guest_begin_on_isolated) - [`guest_begin_on`](session.md#bookclerk_db_guestsessionguest_begin_on) with an explicit isolation requirement.
- [`guest_bootstrap`](#guest_bootstrap) - Bootstrap-only SeaORM proxy metadata for the connected engine.
- [`guest_capabilities`](#guest_capabilities) - Typed `AdapterDatabaseSession.capabilities` for the connected engine.
- [`guest_commit`](#guest_commit) - Commits a transaction previously returned by [`guest_begin`](session.md#bookclerk_db_guestsessionguest_begin).
- [`guest_drop_user_relations`](#guest_drop_user_relations) - Drop relations on the process-wide connection.
- [`guest_drop_user_relations_on`](#guest_drop_user_relations_on) - Drop relations on `conn`.
- [`guest_drop_user_relations_on_txn`](#guest_drop_user_relations_on_txn) - Drop relations on an open guest transaction.
- [`guest_execute`](#guest_execute) - Runs a mutating SQL statement through the guest database bridge.
- [`guest_execute_atomic`](#guest_execute_atomic) - Typed `AdapterDatabaseSession.execute` on the shared library connection.
- [`guest_execute_atomic_on`](#guest_execute_atomic_on) - Typed `execute` on an explicit connection (per-binding sessions).
- [`guest_execute_atomic_on_txn`](#guest_execute_atomic_on_txn) - Typed `Transaction.executeAtomic` on an already-open guest transaction.
- [`guest_execute_request`](#guest_execute_request) - First-party adapter execute: proofs required (`require_stamped=true`).
- [`guest_execute_request_on`](#guest_execute_request_on) - First-party adapter execute on an explicit connection.
- [`guest_export_identity`](#guest_export_identity) - Identity high-water on the process-wide connection.
- [`guest_export_identity_on`](#guest_export_identity_on) - Identity high-water on `conn`.
- [`guest_export_identity_on_txn`](#guest_export_identity_on_txn) - Identity high-water on an open guest transaction.
- [`guest_import_identity`](#guest_import_identity) - Restore identity on the process-wide connection.
- [`guest_import_identity_on`](#guest_import_identity_on) - Restore identity on `conn`.
- [`guest_import_identity_on_txn`](#guest_import_identity_on_txn) - Restore identity on an open guest transaction.
- [`guest_list_user_relations`](#guest_list_user_relations) - User-visible relations on the process-wide connection.
- [`guest_list_user_relations_on`](#guest_list_user_relations_on) - User-visible relations on `conn`.
- [`guest_list_user_relations_on_txn`](#guest_list_user_relations_on_txn) - User-visible relations on an open guest transaction.
- [`guest_ping`](#guest_ping) - Issues a guest `db.ping` RPC against the process-local SeaORM connection.
- [`guest_prepare_unit_restore`](#guest_prepare_unit_restore) - Prepare restore on the process-wide connection.
- [`guest_prepare_unit_restore_on`](#guest_prepare_unit_restore_on) - Prepare restore on `conn`.
- [`guest_prepare_unit_restore_on_txn`](#guest_prepare_unit_restore_on_txn) - Prepare restore on an open guest transaction.
- [`guest_query`](#guest_query) - Runs a read-only SQL query through the guest database bridge.
- [`guest_query_page`](#guest_query_page) - Runs a read-only statement and returns one bounded page (`limit + 1` fetch).
- [`guest_rollback`](#guest_rollback) - Rolls back a transaction previously returned by [`guest_begin`](session.md#bookclerk_db_guestsessionguest_begin).
- [`row_to_dto`](#row_to_dto) - Projects one SeaORM query row into a `GuestRow` (integration tests).
- [`set_connection`](#set_connection) - Store the opened engine connection for subsequent ping/query/execute calls.

---

## bookclerk_db_guest::session::bootstrap_for

*Function*

Bootstrap metadata for an explicit connection's engine family.

```rust
fn bootstrap_for(conn: &sea_orm::DatabaseConnection) -> bookclerk_plugin_abi::DbBootstrap
```



## bookclerk_db_guest::session::capabilities_for

*Function*

Advertised capabilities for an explicit connection's engine family.

```rust
fn capabilities_for(conn: &sea_orm::DatabaseConnection) -> bookclerk_plugin_abi::DbCapabilities
```



## bookclerk_db_guest::session::guest_assert_restore_constraints

*Function*

Assert restore constraints on the process-wide connection.

# Errors

Returns when not connected or FK violations remain.

```rust
fn guest_assert_restore_constraints() -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_assert_restore_constraints_on

*Function*

Assert restore constraints on `conn`.

# Errors

Returns when FK violations remain.

```rust
fn guest_assert_restore_constraints_on(conn: &sea_orm::DatabaseConnection) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_assert_restore_constraints_on_txn

*Function*

Assert restore constraints on an open guest transaction.

# Errors

Returns when the txn id is unknown or FK violations remain.

```rust
fn guest_assert_restore_constraints_on_txn(txn_id: String) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_begin

*Function*

Begins a top-level transaction or a nested savepoint.

Top-level begins wait until no other transaction is open so SQLite
never interleaves writers. Nested begins run on the parent worker task.

# Arguments

* `parent_txn_id` - Existing txn id to nest under; `None` for top-level.

# Returns

Opaque txn id the host must attach to subsequent statements.

# Errors

Returns an error string when not connected, the parent is unknown, or the
engine rejects `BEGIN`.

```rust
fn guest_begin(parent_txn_id: Option<String>) -> std::result::Result<String, String>
```



## bookclerk_db_guest::session::guest_begin_isolated

*Function*

Begins a top-level transaction realizing `isolation` (or a nested savepoint).

# Errors

Returns an error string when not connected, the parent is unknown, or the
engine rejects `BEGIN`.

```rust
fn guest_begin_isolated(parent_txn_id: Option<String>, isolation: bookclerk_plugin_abi::IsolationReq) -> std::result::Result<String, String>
```



## bookclerk_db_guest::session::guest_begin_on

*Function*

Begins a top-level transaction on an explicit connection (plugin bindings).

Does not take the process-wide library writer gate; each binding connection
is independent.

# Errors

Returns an error string when begin fails.

```rust
fn guest_begin_on(conn: sea_orm::DatabaseConnection) -> std::result::Result<String, String>
```



## bookclerk_db_guest::session::guest_begin_on_isolated

*Function*

[`guest_begin_on`](session.md#bookclerk_db_guestsessionguest_begin_on) with an explicit isolation requirement.

# Errors

Returns an error string when begin fails.

```rust
fn guest_begin_on_isolated(conn: sea_orm::DatabaseConnection, isolation: bookclerk_plugin_abi::IsolationReq) -> std::result::Result<String, String>
```



## bookclerk_db_guest::session::guest_bootstrap

*Function*

Bootstrap-only SeaORM proxy metadata for the connected engine.

# Errors

Returns when no connection has been opened.

```rust
fn guest_bootstrap() -> std::result::Result<bookclerk_plugin_abi::DbBootstrap, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_guest::session::guest_capabilities

*Function*

Typed `AdapterDatabaseSession.capabilities` for the connected engine.

# Errors

Returns when no connection has been opened.

```rust
fn guest_capabilities() -> std::result::Result<bookclerk_plugin_abi::DbCapabilities, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_guest::session::guest_commit

*Function*

Commits a transaction previously returned by [`guest_begin`](session.md#bookclerk_db_guestsessionguest_begin).

# Arguments

* `txn_id` - Id from [`guest_begin`](session.md#bookclerk_db_guestsessionguest_begin).

# Errors

Returns an error string when the id is unknown or the engine rejects commit.

```rust
fn guest_commit(txn_id: String) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_drop_user_relations

*Function*

Drop relations on the process-wide connection.

# Errors

Returns when not connected or a drop fails.

```rust
fn guest_drop_user_relations(names: &[String]) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_drop_user_relations_on

*Function*

Drop relations on `conn`.

# Errors

Returns when a drop fails.

```rust
fn guest_drop_user_relations_on(conn: &sea_orm::DatabaseConnection, names: &[String]) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_drop_user_relations_on_txn

*Function*

Drop relations on an open guest transaction.

# Errors

Returns when the txn id is unknown or a drop fails.

```rust
fn guest_drop_user_relations_on_txn(txn_id: String, names: Vec<String>) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_execute

*Function*

Runs a mutating SQL statement through the guest database bridge.

# Arguments

* `dto` - RPC statement DTO from the host bridge.

# Returns

Last-insert id and rows-affected in a `GuestExecResult`.

# Errors

Returns an error string when not connected or the engine rejects the statement.

```rust
fn guest_execute(dto: crate::sql::GuestStatement) -> std::result::Result<crate::sql::GuestExecResult, String>
```



## bookclerk_db_guest::session::guest_execute_atomic

*Function*

Typed `AdapterDatabaseSession.execute` on the shared library connection.

Proofs are required. The guest encodes [`ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecutereply) before COMMIT;
post-commit failures are classified as `unavailable`.

# Errors

Returns when no connection is open, proofs are missing, isolation is
unsupported, or the engine rejects the work.

```rust
fn guest_execute_atomic(envelope: bookclerk_plugin_abi::AdapterExecuteRequest) -> std::result::Result<bookclerk_plugin_abi::ExecuteReply, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_guest::session::guest_execute_atomic_on

*Function*

Typed `execute` on an explicit connection (per-binding sessions).

Unlike [`guest_execute_atomic`](session.md#bookclerk_db_guestsessionguest_execute_atomic), no process-wide writer gate applies:
plugin database bindings are independent databases, so their transactions
never interleave with the shared library connection.

# Errors

Returns when the engine rejects the work.

```rust
fn guest_execute_atomic_on(conn: &sea_orm::DatabaseConnection, envelope: bookclerk_plugin_abi::AdapterExecuteRequest) -> std::result::Result<bookclerk_plugin_abi::ExecuteReply, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_guest::session::guest_execute_atomic_on_txn

*Function*

Typed `Transaction.executeAtomic` on an already-open guest transaction.

Does not BEGIN or COMMIT. The host still owns commit/rollback.

# Errors

Returns when the txn id is unknown or the engine rejects the work.

```rust
fn guest_execute_atomic_on_txn(txn_id: String, request: bookclerk_plugin_abi::AdapterExecuteRequest) -> std::result::Result<bookclerk_plugin_abi::ExecuteReply, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_guest::session::guest_execute_request

*Function*

First-party adapter execute: proofs required (`require_stamped=true`).

# Errors

Returns when no connection is open, proofs are missing, or the engine
rejects the work.

```rust
fn guest_execute_request(request: bookclerk_plugin_abi::AdapterExecuteRequest) -> std::result::Result<bookclerk_plugin_abi::ExecuteReply, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_guest::session::guest_execute_request_on

*Function*

First-party adapter execute on an explicit connection.

# Errors

Returns when proofs are missing or the engine rejects the work.

```rust
fn guest_execute_request_on(conn: &sea_orm::DatabaseConnection, request: bookclerk_plugin_abi::AdapterExecuteRequest) -> std::result::Result<bookclerk_plugin_abi::ExecuteReply, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_guest::session::guest_export_identity

*Function*

Identity high-water on the process-wide connection.

# Errors

Returns when not connected or a catalog query fails.

```rust
fn guest_export_identity() -> std::result::Result<Vec<bookclerk_plugin_abi::DbIdentityHighWater>, String>
```



## bookclerk_db_guest::session::guest_export_identity_on

*Function*

Identity high-water on `conn`.

# Errors

Returns when a catalog query fails.

```rust
fn guest_export_identity_on(conn: &sea_orm::DatabaseConnection) -> std::result::Result<Vec<bookclerk_plugin_abi::DbIdentityHighWater>, String>
```



## bookclerk_db_guest::session::guest_export_identity_on_txn

*Function*

Identity high-water on an open guest transaction.

# Errors

Returns when the txn id is unknown or a catalog query fails.

```rust
fn guest_export_identity_on_txn(txn_id: String) -> std::result::Result<Vec<bookclerk_plugin_abi::DbIdentityHighWater>, String>
```



## bookclerk_db_guest::session::guest_import_identity

*Function*

Restore identity on the process-wide connection.

# Errors

Returns when not connected or a catalog write fails.

```rust
fn guest_import_identity(rows: &[bookclerk_plugin_abi::DbIdentityHighWater]) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_import_identity_on

*Function*

Restore identity on `conn`.

# Errors

Returns when a catalog write fails.

```rust
fn guest_import_identity_on(conn: &sea_orm::DatabaseConnection, rows: &[bookclerk_plugin_abi::DbIdentityHighWater]) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_import_identity_on_txn

*Function*

Restore identity on an open guest transaction.

# Errors

Returns when the txn id is unknown or a catalog write fails.

```rust
fn guest_import_identity_on_txn(txn_id: String, rows: Vec<bookclerk_plugin_abi::DbIdentityHighWater>) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_list_user_relations

*Function*

User-visible relations on the process-wide connection.

# Errors

Returns when not connected or the catalog probe fails.

```rust
fn guest_list_user_relations() -> std::result::Result<Vec<String>, String>
```



## bookclerk_db_guest::session::guest_list_user_relations_on

*Function*

User-visible relations on `conn`.

# Errors

Returns when the catalog probe fails.

```rust
fn guest_list_user_relations_on(conn: &sea_orm::DatabaseConnection) -> std::result::Result<Vec<String>, String>
```



## bookclerk_db_guest::session::guest_list_user_relations_on_txn

*Function*

User-visible relations on an open guest transaction.

# Errors

Returns when the txn id is unknown or the catalog probe fails.

```rust
fn guest_list_user_relations_on_txn(txn_id: String) -> std::result::Result<Vec<String>, String>
```



## bookclerk_db_guest::session::guest_ping

*Function*

Issues a guest `db.ping` RPC against the process-local SeaORM connection.

# Returns

`Ok(())` when the engine responds to ping.

# Errors

Returns an error string when [`set_connection`](session.md#bookclerk_db_guestsessionset_connection) was never called or ping fails.

```rust
fn guest_ping() -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_prepare_unit_restore

*Function*

Prepare restore on the process-wide connection.

# Errors

Returns when not connected or the pragma fails.

```rust
fn guest_prepare_unit_restore() -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_prepare_unit_restore_on

*Function*

Prepare restore on `conn`.

# Errors

Returns when the pragma fails.

```rust
fn guest_prepare_unit_restore_on(conn: &sea_orm::DatabaseConnection) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_prepare_unit_restore_on_txn

*Function*

Prepare restore on an open guest transaction.

# Errors

Returns when the txn id is unknown or the pragma fails.

```rust
fn guest_prepare_unit_restore_on_txn(txn_id: String) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::guest_query

*Function*

Runs a read-only SQL query through the guest database bridge.

# Arguments

* `dto` - RPC statement DTO from the host bridge.

# Returns

Query rows projected into `GuestQueryResult`.

# Errors

Returns an error string when not connected or the engine rejects the statement.

```rust
fn guest_query(dto: crate::sql::GuestStatement) -> std::result::Result<crate::sql::GuestQueryResult, String>
```



## bookclerk_db_guest::session::guest_query_page

*Function*

Runs a read-only statement and returns one bounded page (`limit + 1` fetch).

The adapter wraps `SELECT`/`WITH` SQL once with `LIMIT`/`OFFSET` so later
pages do not rematerialize the full result set. Data-modifying CTEs and
`FOR UPDATE`/`FOR SHARE` are rejected with a literal/comment-aware scan.
Postgres pins one transaction for create → probe → stream → drop:
autocommit pages `BEGIN` + `CREATE TEMP TABLE … ON COMMIT DROP AS` (caller
SQL once), then `pg_column_size` and an ordered select from that table.
The adapter-private `ROW_NUMBER` column reuses the temp-table UUID as a
suffix so it cannot collide with a caller alias such as `_bookclerk_page_ord`.
Guest-txn pages reuse the already-pinned `DatabaseTransaction`.
`SET TRANSACTION READ ONLY` cannot wrap this path: PostgreSQL rejects
`CREATE TABLE` in a read-only transaction. SQLite/D1 issue one
`query_all_raw`. Encoding stops at [`MAX_SCALAR_BYTES`](../bookclerk_plugin_abi/limits.md#bookclerk_plugin_abilimitsmax_scalar_bytes). Cursors are parsed
as a bounded `u64` so `offset + page` cannot overflow.

Huge *server-side* expressions can still stress the engine during
materialize; the protocol bound is the decode/scalar cap.

# Errors

Returns an error string when not connected, the cursor is invalid, the
engine rejects the statement, a row exceeds the scalar budget, or the
driver returns more than `limit + 1` rows.

```rust
fn guest_query_page(dto: crate::sql::GuestStatement, cursor: &str, limit: u32) -> std::result::Result<bookclerk_plugin_sdk::QueryPage, String>
```



## bookclerk_db_guest::session::guest_rollback

*Function*

Rolls back a transaction previously returned by [`guest_begin`](session.md#bookclerk_db_guestsessionguest_begin).

# Arguments

* `txn_id` - Id from [`guest_begin`](session.md#bookclerk_db_guestsessionguest_begin).

# Errors

Returns an error string when the id is unknown or the engine rejects rollback.

```rust
fn guest_rollback(txn_id: String) -> std::result::Result<(), String>
```



## bookclerk_db_guest::session::row_to_dto

*Function*

Projects one SeaORM query row into a `GuestRow` (integration tests).

```rust
fn row_to_dto(row: &sea_orm::QueryResult) -> crate::sql::GuestRow
```



## bookclerk_db_guest::session::set_connection

*Function*

Store the opened engine connection for subsequent ping/query/execute calls.

# Arguments

* `conn` - Opened SeaORM database connection for this guest process.

```rust
fn set_connection(conn: sea_orm::DatabaseConnection)
```



