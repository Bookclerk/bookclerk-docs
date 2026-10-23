# bookclerk_db_guest

First-party database guest internals (SeaORM session workers).

Third-party database plugin authors should depend on
[`bookclerk_plugin_sdk::database_adapter`](../bookclerk_plugin_sdk/database_adapter.md) only. Platform SQLite / Postgres /
D1 guests use this crate for host-mediated session and transaction workers.

| Item | Kind | Summary |
| --- | --- | --- |
| [`BoundGuestHostAdapterSession`](#boundguesthostadaptersession) | struct | Host-private envelope session bound to one dedicated connection (named bindings). |
| [`GuestHostAdapterSession`](#guesthostadaptersession) | struct | In-process SeaORM transaction bridge for host `begin` RPC (shared library connection). |
| [`GuestStatement`](#gueststatement) | struct | SQL text, JSON-encoded binds, and optional txn id for the session worker. |
| [`host_session`](#host_session) | function | Returns the default host adapter session on the shared guest connection. |
| [`host_session_on`](#host_session_on) | function | Host envelope session that persists guest receipts on `conn`. |
| [`bootstrap_for`](#bootstrap_for) | function | Bootstrap metadata for an explicit connection's engine family. |
| [`capabilities_for`](#capabilities_for) | function | Advertised capabilities for an explicit connection's engine family. |
| [`guest_assert_restore_constraints`](#guest_assert_restore_constraints) | function | Assert restore constraints on the process-wide connection. |
| [`guest_assert_restore_constraints_on`](#guest_assert_restore_constraints_on) | function | Assert restore constraints on `conn`. |
| [`guest_assert_restore_constraints_on_txn`](#guest_assert_restore_constraints_on_txn) | function | Assert restore constraints on an open guest transaction. |
| [`guest_begin`](#guest_begin) | function | Begins a top-level transaction or a nested savepoint. |
| [`guest_begin_isolated`](#guest_begin_isolated) | function | Begins a top-level transaction realizing `isolation` (or a nested savepoint). |
| [`guest_begin_on`](#guest_begin_on) | function | Begins a top-level transaction on an explicit connection (plugin bindings). |
| [`guest_begin_on_isolated`](#guest_begin_on_isolated) | function | `guest_begin_on` with an explicit isolation requirement. |
| [`guest_bootstrap`](#guest_bootstrap) | function | Bootstrap-only SeaORM proxy metadata for the connected engine. |
| [`guest_capabilities`](#guest_capabilities) | function | Typed `AdapterDatabaseSession.capabilities` for the connected engine. |
| [`guest_commit`](#guest_commit) | function | Commits a transaction previously returned by `guest_begin`. |
| [`guest_drop_user_relations`](#guest_drop_user_relations) | function | Drop relations on the process-wide connection. |
| [`guest_drop_user_relations_on`](#guest_drop_user_relations_on) | function | Drop relations on `conn`. |
| [`guest_drop_user_relations_on_txn`](#guest_drop_user_relations_on_txn) | function | Drop relations on an open guest transaction. |
| [`guest_execute`](#guest_execute) | function | Runs a mutating SQL statement through the guest database bridge. |
| [`guest_execute_atomic`](#guest_execute_atomic) | function | Typed `AdapterDatabaseSession.execute` on the shared library connection. |
| [`guest_execute_atomic_on`](#guest_execute_atomic_on) | function | Typed `execute` on an explicit connection (per-binding sessions). |
| [`guest_execute_atomic_on_txn`](#guest_execute_atomic_on_txn) | function | Typed `Transaction.executeAtomic` on an already-open guest transaction. |
| [`guest_execute_request`](#guest_execute_request) | function | First-party adapter execute: proofs required (`require_stamped=true`). |
| [`guest_execute_request_on`](#guest_execute_request_on) | function | First-party adapter execute on an explicit connection. |
| [`guest_export_identity`](#guest_export_identity) | function | Identity high-water on the process-wide connection. |
| [`guest_export_identity_on`](#guest_export_identity_on) | function | Identity high-water on `conn`. |
| [`guest_export_identity_on_txn`](#guest_export_identity_on_txn) | function | Identity high-water on an open guest transaction. |
| [`guest_import_identity`](#guest_import_identity) | function | Restore identity on the process-wide connection. |
| [`guest_import_identity_on`](#guest_import_identity_on) | function | Restore identity on `conn`. |
| [`guest_import_identity_on_txn`](#guest_import_identity_on_txn) | function | Restore identity on an open guest transaction. |
| [`guest_list_user_relations`](#guest_list_user_relations) | function | User-visible relations on the process-wide connection. |
| [`guest_list_user_relations_on`](#guest_list_user_relations_on) | function | User-visible relations on `conn`. |
| [`guest_list_user_relations_on_txn`](#guest_list_user_relations_on_txn) | function | User-visible relations on an open guest transaction. |
| [`guest_ping`](#guest_ping) | function | Issues a guest `db.ping` RPC against the process-local SeaORM connection. |
| [`guest_prepare_unit_restore`](#guest_prepare_unit_restore) | function | Prepare restore on the process-wide connection. |
| [`guest_prepare_unit_restore_on`](#guest_prepare_unit_restore_on) | function | Prepare restore on `conn`. |
| [`guest_prepare_unit_restore_on_txn`](#guest_prepare_unit_restore_on_txn) | function | Prepare restore on an open guest transaction. |
| [`guest_query`](#guest_query) | function | Runs a read-only SQL query through the guest database bridge. |
| [`guest_query_page`](#guest_query_page) | function | Runs a read-only statement and returns one bounded page (`limit + 1` fetch). |
| [`guest_rollback`](#guest_rollback) | function | Rolls back a transaction previously returned by `guest_begin`. |
| [`row_to_dto`](#row_to_dto) | function | Projects one SeaORM query row into a `GuestRow` (integration tests). |
| [`set_connection`](#set_connection) | function | Store the opened engine connection for subsequent ping/query/execute calls. |
| [`guest_sql`](#guest_sql) | function | Builds a `GuestStatement` for plain SQL without binds. |

## Structs

### BoundGuestHostAdapterSession

```rust
pub struct BoundGuestHostAdapterSession {
    /* private fields */
}
```

Host-private envelope session bound to one dedicated connection (named bindings).

Implements: `HostAdapterDatabaseSession`

### GuestHostAdapterSession

```rust
pub struct GuestHostAdapterSession;
```

In-process SeaORM transaction bridge for host `begin` RPC (shared library connection).

Implements: `HostAdapterDatabaseSession`

### GuestStatement

```rust
pub struct GuestStatement {
    pub sql: String,
    pub values: Vec<Value>,
    pub txn_id: Option<String>,
}
```

SQL text, JSON-encoded binds, and optional txn id for the session worker.

**Fields**

- `sql`: SQL text with positional placeholders.
- `values`: Ordered bind values (JSON encoding for typed nulls and bytes).
- `txn_id`: Active guest transaction id, if any.

Implements: `Clone`, `Debug`

## Functions

### host_session

```rust
pub fn host_session() -> GuestHostAdapterSession
```

Returns the default host adapter session on the shared guest connection.

### host_session_on

```rust
pub fn host_session_on(conn: DatabaseConnection) -> BoundGuestHostAdapterSession
```

Host envelope session that persists guest receipts on `conn`.

### bootstrap_for

```rust
pub fn bootstrap_for(conn: &DatabaseConnection) -> DbBootstrap
```

Bootstrap metadata for an explicit connection's engine family.

### capabilities_for

```rust
pub fn capabilities_for(conn: &DatabaseConnection) -> DbCapabilities
```

Advertised capabilities for an explicit connection's engine family.

### guest_assert_restore_constraints

```rust
pub async fn guest_assert_restore_constraints() -> Result<(), String>
```

Assert restore constraints on the process-wide connection.

#### Errors

Returns when not connected or FK violations remain.

### guest_assert_restore_constraints_on

```rust
pub async fn guest_assert_restore_constraints_on(
    conn: &DatabaseConnection,
) -> Result<(), String>
```

Assert restore constraints on `conn`.

#### Errors

Returns when FK violations remain.

### guest_assert_restore_constraints_on_txn

```rust
pub async fn guest_assert_restore_constraints_on_txn(txn_id: String) -> Result<(), String>
```

Assert restore constraints on an open guest transaction.

#### Errors

Returns when the txn id is unknown or FK violations remain.

### guest_begin

```rust
pub async fn guest_begin(parent_txn_id: Option<String>) -> Result<String, String>
```

Begins a top-level transaction or a nested savepoint.

Top-level begins wait until no other transaction is open so SQLite
never interleaves writers. Nested begins run on the parent worker task.

#### Arguments

* `parent_txn_id` - Existing txn id to nest under; `None` for top-level.

#### Returns

Opaque txn id the host must attach to subsequent statements.

#### Errors

Returns an error string when not connected, the parent is unknown, or the
engine rejects `BEGIN`.

### guest_begin_isolated

```rust
pub async fn guest_begin_isolated(
    parent_txn_id: Option<String>,
    isolation: IsolationReq,
) -> Result<String, String>
```

Begins a top-level transaction realizing `isolation` (or a nested savepoint).

#### Errors

Returns an error string when not connected, the parent is unknown, or the
engine rejects `BEGIN`.

### guest_begin_on

```rust
pub async fn guest_begin_on(conn: DatabaseConnection) -> Result<String, String>
```

Begins a top-level transaction on an explicit connection (plugin bindings).

Does not take the process-wide library writer gate; each binding connection
is independent.

#### Errors

Returns an error string when begin fails.

### guest_begin_on_isolated

```rust
pub async fn guest_begin_on_isolated(
    conn: DatabaseConnection,
    isolation: IsolationReq,
) -> Result<String, String>
```

[`guest_begin_on`](#guest_begin_on) with an explicit isolation requirement.

#### Errors

Returns an error string when begin fails.

### guest_bootstrap

```rust
pub async fn guest_bootstrap() -> Result<DbBootstrap, PluginError>
```

Bootstrap-only SeaORM proxy metadata for the connected engine.

#### Errors

Returns when no connection has been opened.

### guest_capabilities

```rust
pub async fn guest_capabilities() -> Result<DbCapabilities, PluginError>
```

Typed `AdapterDatabaseSession.capabilities` for the connected engine.

#### Errors

Returns when no connection has been opened.

### guest_commit

```rust
pub async fn guest_commit(txn_id: String) -> Result<(), String>
```

Commits a transaction previously returned by [`guest_begin`](#guest_begin).

#### Arguments

* `txn_id` - Id from [`guest_begin`](#guest_begin).

#### Errors

Returns an error string when the id is unknown or the engine rejects commit.

### guest_drop_user_relations

```rust
pub async fn guest_drop_user_relations(names: &[String]) -> Result<(), String>
```

Drop relations on the process-wide connection.

#### Errors

Returns when not connected or a drop fails.

### guest_drop_user_relations_on

```rust
pub async fn guest_drop_user_relations_on(
    conn: &DatabaseConnection,
    names: &[String],
) -> Result<(), String>
```

Drop relations on `conn`.

#### Errors

Returns when a drop fails.

### guest_drop_user_relations_on_txn

```rust
pub async fn guest_drop_user_relations_on_txn(
    txn_id: String,
    names: Vec<String>,
) -> Result<(), String>
```

Drop relations on an open guest transaction.

#### Errors

Returns when the txn id is unknown or a drop fails.

### guest_execute

```rust
pub async fn guest_execute(dto: GuestStatement) -> Result<GuestExecResult, String>
```

Runs a mutating SQL statement through the guest database bridge.

#### Arguments

* `dto` - RPC statement DTO from the host bridge.

#### Returns

Last-insert id and rows-affected in a `GuestExecResult`.

#### Errors

Returns an error string when not connected or the engine rejects the statement.

### guest_execute_atomic

```rust
pub async fn guest_execute_atomic(
    envelope: AdapterExecuteRequest,
) -> Result<ExecuteReply, PluginError>
```

Typed `AdapterDatabaseSession.execute` on the shared library connection.

Proofs are required. The guest encodes [`ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#executereply) before COMMIT;
post-commit failures are classified as `unavailable`.

#### Errors

Returns when no connection is open, proofs are missing, isolation is
unsupported, or the engine rejects the work.

### guest_execute_atomic_on

```rust
pub async fn guest_execute_atomic_on(
    conn: &DatabaseConnection,
    envelope: AdapterExecuteRequest,
) -> Result<ExecuteReply, PluginError>
```

Typed `execute` on an explicit connection (per-binding sessions).

Unlike [`guest_execute_atomic`](#guest_execute_atomic), no process-wide writer gate applies:
plugin database bindings are independent databases, so their transactions
never interleave with the shared library connection.

#### Errors

Returns when the engine rejects the work.

### guest_execute_atomic_on_txn

```rust
pub async fn guest_execute_atomic_on_txn(
    txn_id: String,
    request: AdapterExecuteRequest,
) -> Result<ExecuteReply, PluginError>
```

Typed `Transaction.executeAtomic` on an already-open guest transaction.

Does not BEGIN or COMMIT. The host still owns commit/rollback.

#### Errors

Returns when the txn id is unknown or the engine rejects the work.

### guest_execute_request

```rust
pub async fn guest_execute_request(
    request: AdapterExecuteRequest,
) -> Result<ExecuteReply, PluginError>
```

First-party adapter execute: proofs required (`require_stamped=true`).

#### Errors

Returns when no connection is open, proofs are missing, or the engine
rejects the work.

### guest_execute_request_on

```rust
pub async fn guest_execute_request_on(
    conn: &DatabaseConnection,
    request: AdapterExecuteRequest,
) -> Result<ExecuteReply, PluginError>
```

First-party adapter execute on an explicit connection.

#### Errors

Returns when proofs are missing or the engine rejects the work.

### guest_export_identity

```rust
pub async fn guest_export_identity() -> Result<Vec<DbIdentityHighWater>, String>
```

Identity high-water on the process-wide connection.

#### Errors

Returns when not connected or a catalog query fails.

### guest_export_identity_on

```rust
pub async fn guest_export_identity_on(
    conn: &DatabaseConnection,
) -> Result<Vec<DbIdentityHighWater>, String>
```

Identity high-water on `conn`.

#### Errors

Returns when a catalog query fails.

### guest_export_identity_on_txn

```rust
pub async fn guest_export_identity_on_txn(
    txn_id: String,
) -> Result<Vec<DbIdentityHighWater>, String>
```

Identity high-water on an open guest transaction.

#### Errors

Returns when the txn id is unknown or a catalog query fails.

### guest_import_identity

```rust
pub async fn guest_import_identity(rows: &[DbIdentityHighWater]) -> Result<(), String>
```

Restore identity on the process-wide connection.

#### Errors

Returns when not connected or a catalog write fails.

### guest_import_identity_on

```rust
pub async fn guest_import_identity_on(
    conn: &DatabaseConnection,
    rows: &[DbIdentityHighWater],
) -> Result<(), String>
```

Restore identity on `conn`.

#### Errors

Returns when a catalog write fails.

### guest_import_identity_on_txn

```rust
pub async fn guest_import_identity_on_txn(
    txn_id: String,
    rows: Vec<DbIdentityHighWater>,
) -> Result<(), String>
```

Restore identity on an open guest transaction.

#### Errors

Returns when the txn id is unknown or a catalog write fails.

### guest_list_user_relations

```rust
pub async fn guest_list_user_relations() -> Result<Vec<String>, String>
```

User-visible relations on the process-wide connection.

#### Errors

Returns when not connected or the catalog probe fails.

### guest_list_user_relations_on

```rust
pub async fn guest_list_user_relations_on(
    conn: &DatabaseConnection,
) -> Result<Vec<String>, String>
```

User-visible relations on `conn`.

#### Errors

Returns when the catalog probe fails.

### guest_list_user_relations_on_txn

```rust
pub async fn guest_list_user_relations_on_txn(
    txn_id: String,
) -> Result<Vec<String>, String>
```

User-visible relations on an open guest transaction.

#### Errors

Returns when the txn id is unknown or the catalog probe fails.

### guest_ping

```rust
pub async fn guest_ping() -> Result<(), String>
```

Issues a guest `db.ping` RPC against the process-local SeaORM connection.

#### Returns

`Ok(())` when the engine responds to ping.

#### Errors

Returns an error string when [`set_connection`](#set_connection) was never called or ping fails.

### guest_prepare_unit_restore

```rust
pub async fn guest_prepare_unit_restore() -> Result<(), String>
```

Prepare restore on the process-wide connection.

#### Errors

Returns when not connected or the pragma fails.

### guest_prepare_unit_restore_on

```rust
pub async fn guest_prepare_unit_restore_on(
    conn: &DatabaseConnection,
) -> Result<(), String>
```

Prepare restore on `conn`.

#### Errors

Returns when the pragma fails.

### guest_prepare_unit_restore_on_txn

```rust
pub async fn guest_prepare_unit_restore_on_txn(txn_id: String) -> Result<(), String>
```

Prepare restore on an open guest transaction.

#### Errors

Returns when the txn id is unknown or the pragma fails.

### guest_query

```rust
pub async fn guest_query(dto: GuestStatement) -> Result<GuestQueryResult, String>
```

Runs a read-only SQL query through the guest database bridge.

#### Arguments

* `dto` - RPC statement DTO from the host bridge.

#### Returns

Query rows projected into `GuestQueryResult`.

#### Errors

Returns an error string when not connected or the engine rejects the statement.

### guest_query_page

```rust
pub async fn guest_query_page(
    dto: GuestStatement,
    cursor: &str,
    limit: u32,
) -> Result<QueryPage, String>
```

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
`query_all_raw`. Encoding stops at [`MAX_SCALAR_BYTES`](../bookclerk_plugin_abi/index.md#max_scalar_bytes). Cursors are parsed
as a bounded `u64` so `offset + page` cannot overflow.

Huge *server-side* expressions can still stress the engine during
materialize; the protocol bound is the decode/scalar cap.

#### Errors

Returns an error string when not connected, the cursor is invalid, the
engine rejects the statement, a row exceeds the scalar budget, or the
driver returns more than `limit + 1` rows.

### guest_rollback

```rust
pub async fn guest_rollback(txn_id: String) -> Result<(), String>
```

Rolls back a transaction previously returned by [`guest_begin`](#guest_begin).

#### Arguments

* `txn_id` - Id from [`guest_begin`](#guest_begin).

#### Errors

Returns an error string when the id is unknown or the engine rejects rollback.

### row_to_dto

```rust
pub fn row_to_dto(row: &QueryResult) -> GuestRow
```

Projects one SeaORM query row into a `GuestRow` (integration tests).

### set_connection

```rust
pub async fn set_connection(conn: DatabaseConnection)
```

Store the opened engine connection for subsequent ping/query/execute calls.

#### Arguments

* `conn` - Opened SeaORM database connection for this guest process.

### guest_sql

```rust
pub fn guest_sql(sql: impl Into<String>) -> GuestStatement
```

Builds a [`GuestStatement`](#gueststatement) for plain SQL without binds.
