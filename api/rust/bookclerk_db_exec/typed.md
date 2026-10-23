**bookclerk_db_exec > typed**

# Module: typed

## Contents

**Functions**

- [`db_value_from_sea`](#db_value_from_sea) - Reconstruct a typed cell from a SeaORM value.
- [`db_value_to_sea`](#db_value_to_sea) - Convert a typed bind into a SeaORM value without JSON / `b64:` decoding.
- [`execute_canonical_stamped`](#execute_canonical_stamped) - Stamp leftover canonical SQL, then execute it with sqlite-shaped transport.
- [`execute_typed_envelope`](#execute_typed_envelope) - [`execute_typed_on_session`](typed.md#bookclerk_db_exectypedexecute_typed_on_session) using host-private proofs already stamped on `envelope`.
- [`execute_typed_envelope_on_connection`](#execute_typed_envelope_on_connection) - [`execute_typed_envelope`](typed.md#bookclerk_db_exectypedexecute_typed_envelope) selecting the physical engine from `db`.
- [`execute_typed_on_open_connection`](#execute_typed_on_open_connection) - [`execute_typed_on_open_envelope`](typed.md#bookclerk_db_exectypedexecute_typed_on_open_envelope) selecting the physical engine from `conn`.
- [`execute_typed_on_open_envelope`](#execute_typed_on_open_envelope) - Run a stamped envelope on an already-open connection (no BEGIN/COMMIT).
- [`execute_typed_on_session`](#execute_typed_on_session) - Run a typed atomic batch on an existing SeaORM connection.
- [`execute_typed_on_session_then`](#execute_typed_on_session_then) - Like [`execute_typed_on_session`](typed.md#bookclerk_db_exectypedexecute_typed_on_session), running extra statements in the same transaction
- [`execute_typed_on_txn`](#execute_typed_on_txn) - Run a typed batch on an already-open transaction (no BEGIN/COMMIT).
- [`execute_typed_on_txn_envelope`](#execute_typed_on_txn_envelope) - [`execute_typed_on_txn`](typed.md#bookclerk_db_exectypedexecute_typed_on_txn) with a host-only guest-receipt persist hint and proofs.
- [`load_physical_sql_type_env`](#load_physical_sql_type_env) - Live physical tables on this connection (host execute only).
- [`load_sql_type_env`](#load_sql_type_env) - Reconstructs [`SqlTypeEnv`](../bookclerk_plugin_abi/sql_types.md#bookclerk_plugin_abisql_typessqltypeenv) from the durable binding catalog, if present.
- [`load_sql_type_env_capped`](#load_sql_type_env_capped) - [`load_sql_type_env`](typed.md#bookclerk_db_exectypedload_sql_type_env) paging at `min(max_result_rows, FIRST_PARTY_MAX_RESULT_ROWS)`.
- [`proofs_for_host_plan`](#proofs_for_host_plan) - Host plans may include already-lowered schema companions (`PRAGMA`,
- [`query_canonical_stamped`](#query_canonical_stamped) - Stamp leftover canonical SQL, then query it with sqlite-shaped transport.
- [`stamp_adapter_execute`](#stamp_adapter_execute) - Host-stamped [`AdapterExecuteRequest`](../bookclerk_plugin_abi/host_envelope.md#bookclerk_plugin_abihost_envelopeadapterexecuterequest) for first-party adapter execute.
- [`stamp_host_proofs`](#stamp_host_proofs) - Stamps 1:1 proofs for host-authored canonical SQL (planner / SeaORM).

---

## bookclerk_db_exec::typed::db_value_from_sea

*Function*

Reconstruct a typed cell from a SeaORM value.

Unlike the JSON guest path, this preserves engine types: a BLOB column
becomes `DbValue::Bytes`, and `b64:` text is left as `DbValue::Text`.

# Errors

Returns when the SeaORM value is outside the universal domain, or TEXT
contains U+0000.

```rust
fn db_value_from_sea(v: &sea_orm::Value) -> Result<bookclerk_plugin_abi::DbValue, String>
```



## bookclerk_db_exec::typed::db_value_to_sea

*Function*

Convert a typed bind into a SeaORM value without JSON / `b64:` decoding.

```rust
fn db_value_to_sea(value: &bookclerk_plugin_abi::DbValue) -> sea_orm::Value
```



## bookclerk_db_exec::typed::execute_canonical_stamped

*Function*

Stamp leftover canonical SQL, then execute it with sqlite-shaped transport.

Production leftover goes through SeaORM `execute_raw` on the sqlite-shaped
proxy, which stamps with `bookclerk_plugin_abi::CanonicalExecuteRequest::from_desugared`
(no second desugar). Host leftover must therefore desugar `ORDER BY NULLS`
/ `NULLIF` **before** transport. Does not start a nested transaction.

# Errors

Returns when typecheck fails or the connection rejects the statement.

```rust
fn execute_canonical_stamped<C, impl IntoIterator<Item = SeaValue>>(conn: &C, sql: &str, values: impl Trait, type_env: &bookclerk_plugin_abi::SqlTypeEnv) -> Result<u64, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::execute_typed_envelope

*Function*

[`execute_typed_on_session`](typed.md#bookclerk_db_exectypedexecute_typed_on_session) using host-private proofs already stamped on `envelope`.

Envelope proofs are required 1:1 with `envelope.request.statements`. Catalog
changes after admission must not rebuild lowering. Pre-admission public
execute uses [`execute_typed_on_session`](typed.md#bookclerk_db_exectypedexecute_typed_on_session), which may typecheck.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when proofs are missing or mismatched, a statement fails,
the encoded reply exceeds `max_atomic_result_bytes`, or the session is
interrupted.

```rust
fn execute_typed_envelope<impl Into<ExecCaps>>(engine: crate::exec::PhysicalEngine, db: &sea_orm::DatabaseConnection, envelope: &bookclerk_plugin_abi::AdapterExecuteRequest, timing_source: &str, caps: impl Trait, session: crate::exec::AtomicSession) -> Result<bookclerk_plugin_abi::ExecuteReply, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::execute_typed_envelope_on_connection

*Function*

[`execute_typed_envelope`](typed.md#bookclerk_db_exectypedexecute_typed_envelope) selecting the physical engine from `db`.

In-process sqlite/postgres tests call this instead of naming
[`PhysicalEngine`](exec.md#bookclerk_db_execexecphysicalengine) in host/library code. Production RPC paths send the
envelope to adapter `execute` and must not call this on the sqlite-shaped
proxy.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when proofs are missing or mismatched, a statement fails,
the encoded reply exceeds `max_atomic_result_bytes`, or the session is
interrupted.

```rust
fn execute_typed_envelope_on_connection<impl Into<ExecCaps>>(db: &sea_orm::DatabaseConnection, envelope: &bookclerk_plugin_abi::AdapterExecuteRequest, caps: impl Trait, session: crate::exec::AtomicSession) -> Result<bookclerk_plugin_abi::ExecuteReply, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::execute_typed_on_open_connection

*Function*

[`execute_typed_on_open_envelope`](typed.md#bookclerk_db_exectypedexecute_typed_on_open_envelope) selecting the physical engine from `conn`.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when proofs are missing, a statement fails, or the encoded
reply exceeds `max_atomic_result_bytes`.

```rust
fn execute_typed_on_open_connection<C, impl Into<ExecCaps>>(conn: &C, envelope: &bookclerk_plugin_abi::AdapterExecuteRequest, caps: impl Trait, session: crate::exec::AtomicSession, describe: Option<&sea_orm::DatabaseConnection>) -> Result<bookclerk_plugin_abi::ExecuteReply, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::execute_typed_on_open_envelope

*Function*

Run a stamped envelope on an already-open connection (no BEGIN/COMMIT).

In-process postgres tests use this when the caller already holds a SeaORM
connection or transaction and must physically lower canonical SQL. Host
RPC/proxy paths must not call this — they send [`AdapterExecuteRequest`](../bookclerk_plugin_abi/host_envelope.md#bookclerk_plugin_abihost_envelopeadapterexecuterequest).

Binding catalog companions are skipped: leftover DML and restore DDL run
against a catalog the caller already applied (`apply_host_schema` or
[`bookclerk_plugin_abi::catalog_companions`](../bookclerk_plugin_abi/sql_types.md#bookclerk_plugin_abisql_typescatalog_companions)). Re-inserting those rows
conflicts with `bookclerk_sql_catalog_pkey`. Postgres identity companions
still run so restore can recreate `bookclerk_identity` after dropping it.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when proofs are missing, a statement fails, or the encoded
reply exceeds `max_atomic_result_bytes`.

```rust
fn execute_typed_on_open_envelope<C, impl Into<ExecCaps>>(engine: crate::exec::PhysicalEngine, conn: &C, envelope: &bookclerk_plugin_abi::AdapterExecuteRequest, timing_source: &str, caps: impl Trait, session: crate::exec::AtomicSession, describe: Option<&sea_orm::DatabaseConnection>) -> Result<bookclerk_plugin_abi::ExecuteReply, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::execute_typed_on_session

*Function*

Run a typed atomic batch on an existing SeaORM connection.

Encodes the [`ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecutereply) **before** COMMIT. Encoding or result-budget
failures roll the transaction back.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when a statement fails, the encoded reply exceeds
`max_atomic_result_bytes`, TEXT contains U+0000, or the session is
interrupted.

```rust
fn execute_typed_on_session<impl Into<ExecCaps>>(db: &sea_orm::DatabaseConnection, req: &bookclerk_plugin_abi::ExecuteRequest, guest_receipt: bookclerk_plugin_abi::GuestReceiptPersist, timing_source: &str, caps: impl Trait, session: crate::exec::AtomicSession) -> Result<bookclerk_plugin_abi::ExecuteReply, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::execute_typed_on_session_then

*Function*

Like [`execute_typed_on_session`](typed.md#bookclerk_db_exectypedexecute_typed_on_session), running extra statements in the same transaction
before COMMIT (used to persist guest replay payloads on `bookclerk_receipts`).

`guest_hash` is the guest `requestHash` used to decide whether a claimed
prior receipt should resume remaining guest SQL instead of skipping it.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when a statement fails, `then` fails, encoding fails, or COMMIT fails.

```rust
fn execute_typed_on_session_then<F, impl Into<ExecCaps>>(db: &sea_orm::DatabaseConnection, req: &bookclerk_plugin_abi::ExecuteRequest, timing_source: &str, caps: impl Trait, session: crate::exec::AtomicSession, then: F, guest_hash: Option<String>) -> Result<bookclerk_plugin_abi::ExecuteReply, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::execute_typed_on_txn

*Function*

Run a typed batch on an already-open transaction (no BEGIN/COMMIT).

Used by nested SeaORM work: the guest interactive txn is already open, so
a second `executeAtomic` BEGIN would fail. The batch runs inside a
`SAVEPOINT`; statement, encoding, or budget failures roll back to that
savepoint so a later outer `commit()` cannot persist a partial batch.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when a statement fails or the encoded reply exceeds
`max_atomic_result_bytes`.

```rust
fn execute_typed_on_txn<impl Into<ExecCaps>>(txn: &sea_orm::DatabaseTransaction, req: &bookclerk_plugin_abi::ExecuteRequest, timing_source: &str, caps: impl Trait, session: crate::exec::AtomicSession, describe: Option<&sea_orm::DatabaseConnection>) -> Result<bookclerk_plugin_abi::ExecuteReply, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::execute_typed_on_txn_envelope

*Function*

[`execute_typed_on_txn`](typed.md#bookclerk_db_exectypedexecute_typed_on_txn) with a host-only guest-receipt persist hint and proofs.

Finalize SQL runs inside the nested savepoint before returning, so an outer
`commit()` persists the caller-visible payload.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when a statement fails, finalize fails, or the encoded
reply exceeds `max_atomic_result_bytes`.

```rust
fn execute_typed_on_txn_envelope<impl Into<ExecCaps>>(engine: crate::exec::PhysicalEngine, txn: &sea_orm::DatabaseTransaction, envelope: &bookclerk_plugin_abi::AdapterExecuteRequest, timing_source: &str, caps: impl Trait, session: crate::exec::AtomicSession, describe: Option<&sea_orm::DatabaseConnection>) -> Result<bookclerk_plugin_abi::ExecuteReply, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::load_physical_sql_type_env

*Function*

Live physical tables on this connection (host execute only).

Plugin bindings type against [`SQL_CATALOG_TABLE`](../bookclerk_plugin_abi/sql_types.md#bookclerk_plugin_abisql_typessql_catalog_table), not sqlite_master.
Host plans may query tables created outside the canonical host snapshot
(tests, `PRAGMA`/raw DDL); those columns are merged here so typing still
runs.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the physical catalog query fails.

```rust
fn load_physical_sql_type_env<impl ConnectionTrait>(conn: &impl Trait) -> Result<bookclerk_plugin_abi::SqlTypeEnv, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::load_sql_type_env

*Function*

Reconstructs [`SqlTypeEnv`](../bookclerk_plugin_abi/sql_types.md#bookclerk_plugin_abisql_typessqltypeenv) from the durable binding catalog, if present.

A missing catalog table yields an empty environment. RPC/adapter/shape/type
errors fail closed. On PostgreSQL the probe runs under a savepoint so a
missing-table `SELECT` cannot abort the current transaction (`25P02`).

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the catalog query fails for a reason other than a
missing table, or when a catalog cell is malformed.

```rust
fn load_sql_type_env<impl ConnectionTrait>(conn: &impl Trait) -> Result<bookclerk_plugin_abi::SqlTypeEnv, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::load_sql_type_env_capped

*Function*

[`load_sql_type_env`](typed.md#bookclerk_db_exectypedload_sql_type_env) paging at `min(max_result_rows, FIRST_PARTY_MAX_RESULT_ROWS)`.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the catalog query fails for a reason other than a
missing table, or when a catalog cell is malformed.

```rust
fn load_sql_type_env_capped<impl ConnectionTrait>(conn: &impl Trait, max_result_rows: u32) -> Result<bookclerk_plugin_abi::SqlTypeEnv, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::proofs_for_host_plan

*Function*

Host plans may include already-lowered schema companions (`PRAGMA`,
`CREATE FUNCTION`, …) and greenfield DDL. Those get a hash-bound empty
proof. Canonical DML is typed against the merged schema in statement order.

# Errors

Returns `DbErr::Custom` when typecheck fails or a statement yields no proof.

```rust
fn proofs_for_host_plan(req: &bookclerk_plugin_abi::ExecuteRequest, env: &bookclerk_plugin_abi::SqlTypeEnv) -> Result<Vec<bookclerk_plugin_abi::ResolvedStatement>, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::query_canonical_stamped

*Function*

Stamp leftover canonical SQL, then query it with sqlite-shaped transport.

Same desugar-before-transport rule as [`execute_canonical_stamped`](typed.md#bookclerk_db_exectypedexecute_canonical_stamped). Does
not start a nested transaction.

# Errors

Returns when typecheck fails or the connection rejects the statement.

```rust
fn query_canonical_stamped<C, impl IntoIterator<Item = SeaValue>>(conn: &C, sql: &str, values: impl Trait, type_env: &bookclerk_plugin_abi::SqlTypeEnv) -> Result<Vec<sea_orm::QueryResult>, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::stamp_adapter_execute

*Function*

Host-stamped [`AdapterExecuteRequest`](../bookclerk_plugin_abi/host_envelope.md#bookclerk_plugin_abihost_envelopeadapterexecuterequest) for first-party adapter execute.

Desugars canonical SQL exactly once (`ORDER BY NULLS`, `NULLIF`) so proofs
bind to the same text that crosses the adapter boundary. Adapters must not
call this.

# Errors

Returns `DbErr::Custom` when typecheck fails, proofs do not bind, or a
TEXT SQL/bind contains U+0000.

```rust
fn stamp_adapter_execute(request: bookclerk_plugin_abi::ExecuteRequest, catalog: &bookclerk_plugin_abi::SqlTypeEnv) -> Result<bookclerk_plugin_abi::AdapterExecuteRequest, sea_orm::DbErr>
```



## bookclerk_db_exec::typed::stamp_host_proofs

*Function*

Stamps 1:1 proofs for host-authored canonical SQL (planner / SeaORM).

Host companion SQL (`PRAGMA`, identity functions, DDL) gets a hash-bound
empty proof. Canonical DML is typed against bookkeeping + catalog tables
plus `catalog`.

# Errors

Returns `DbErr::Custom` when typecheck fails.

```rust
fn stamp_host_proofs(req: &bookclerk_plugin_abi::ExecuteRequest, catalog: &bookclerk_plugin_abi::SqlTypeEnv) -> Result<Vec<bookclerk_plugin_abi::ResolvedStatement>, sea_orm::DbErr>
```



