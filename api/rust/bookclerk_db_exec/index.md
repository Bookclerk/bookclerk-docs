# bookclerk_db_exec

Neutral SQL executor / adapter SDK for Bookclerk database guests.

Hosts compile domain work into a typed [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest)
of **canonical** Bookclerk SQL (`?` placeholders, host-desugared NULLS /
`NULLIF`). This crate owns physical lowering and native execution. It must
not import Bookclerk entities or host domain planners. Postgres adapters
lower host-schema packs and binding DDL types at execute
([`schema_sql_for_backend`](#schema_sql_for_backend), [`lower_binding_ddl_execute_request`](#lower_binding_ddl_execute_request));
[`lower_canonical_sql`](#lower_canonical_sql) stays DML/query helpers. Host crates must not call
those lowerers.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AtomicSession`](#atomicsession) | struct | Session-level cancel / deadline for one atomic attempt (not hashed). |
| [`ExecCaps`](#execcaps) | struct | Per-statement result bounds. Row/byte `0` means unlimited at execute time. |
| [`DbErrorClass`](#dberrorclass) | enum | Portable Bookclerk SQL error category. |
| [`PhysicalEngine`](#physicalengine) | enum | Real engine the adapter opened. |
| [`SqlExecTarget`](#sqlexectarget) | enum | How leftover host SQL reaches a SeaORM connection. |
| [`set_positional_result_columns`](#set_positional_result_columns) | function | Records engine column metadata for the in-flight query (SQLite rusqlite path). |
| [`take_positional_result_columns`](#take_positional_result_columns) | function | Takes engine column metadata recorded by the SQLite adapter, if any. |
| [`b64_string_to_bytes`](#b64_string_to_bytes) | function | Decode a `b64:`-prefixed string. Returns `None` if not prefixed. |
| [`bytes_to_b64_string`](#bytes_to_b64_string) | function | Encode bytes as a `b64:`-prefixed JSON string. |
| [`apply_begin_isolation`](#apply_begin_isolation) | function | Realizes `IsolationReq` on an already-open transaction. |
| [`assert_restore_constraints`](#assert_restore_constraints) | function | SQLite: `PRAGMA foreign_key_check` must be empty before commit. |
| [`begin_consistent_snapshot`](#begin_consistent_snapshot) | function | Begins a consistent capture transaction. |
| [`drop_user_relations`](#drop_user_relations) | function | Drops named relations. PostgreSQL uses `CASCADE` plus identity functions. |
| [`export_identity`](#export_identity) | function | Identity high-water from `sqlite_sequence` and `SQL_IDENTITY_TABLE`. |
| [`import_identity`](#import_identity) | function | Writes identity high-water into adapter catalogs. |
| [`list_user_relations`](#list_user_relations) | function | User-visible base tables in the current schema. |
| [`pending_begin_isolation`](#pending_begin_isolation) | function | Isolation the next SeaORM `begin()` should ask the adapter to realize. |
| [`prepare_unit_restore`](#prepare_unit_restore) | function | SQLite: `PRAGMA defer_foreign_keys = ON` on the restore transaction. |
| [`canonical_statement`](#canonical_statement) | function | Builds a SeaORM statement that keeps canonical `?` placeholders. |
| [`execute_canonical`](#execute_canonical) | function | Executes already-canonical host SQL. Never desugars or physically lowers. |
| [`query_canonical`](#query_canonical) | function | Queries already-canonical host SQL. Never desugars or physically lowers. |
| [`classify_db_err`](#classify_db_err) | function | Classifies `err` from adapter-stamped codes, not English engine messages. |
| [`classify_db_err_message`](#classify_db_err_message) | function | Classifies an already-formatted adapter/ORM error string. |
| [`is_schema_apply_retryable`](#is_schema_apply_retryable) | function | True when host schema apply may retry after re-reading durable schema state. |
| [`cap_query_sql`](#cap_query_sql) | function | Wraps a host-tagged `bookclerk_plugin_abi::DbPlanStatementKind::Select` so the engine stops after `cap + 1` rows. Callers must not pass DML `RETURNING` SQL. |
| [`encoded_proxy_row_len`](#encoded_proxy_row_len) | function | Encoded JSON length of a SeaORM proxy row (keys, numbers, punctuation). |
| [`execute_physical_sql`](#execute_physical_sql) | function | Executes canonical SQL on a physical engine the adapter opened. |
| [`json_cell_utf8_len`](#json_cell_utf8_len) | function | UTF-8 / JSON length of one cell used for `maxCellBytes`. |
| [`note_encoded_result_bytes`](#note_encoded_result_bytes) | function | Adds `extra` encoded JSON bytes toward `max_result_bytes`. |
| [`query_physical_sql`](#query_physical_sql) | function | Queries canonical SQL on a physical engine the adapter opened. |
| [`query_physical_sql_typed`](#query_physical_sql_typed) | function | `query_physical_sql` with an optional hash-bound proof (TEXT collate / INTEGER overflow on Postgres). |
| [`sea_value_to_json`](#sea_value_to_json) | function | Maps a SeaORM cell onto JSON for plan interpretation. |
| [`lower_canonical_ddl_to_postgres`](#lower_canonical_ddl_to_postgres) | function | Lowers one canonical Bookclerk **DDL** statement onto PostgreSQL. |
| [`lower_canonical_ddl_to_postgres_with`](#lower_canonical_ddl_to_postgres_with) | function | `lower_canonical_ddl_to_postgres` using `env` for tie-break column types. |
| [`lower_canonical_sql`](#lower_canonical_sql) | function | Lowers canonical Bookclerk **DML/query** SQL for `backend`. |
| [`lower_canonical_sql_typed`](#lower_canonical_sql_typed) | function | `lower_canonical_sql` plus proof-directed overflow and Postgres `COLLATE "C"`. |
| [`lower_canonical_to_postgres`](#lower_canonical_to_postgres) | function | Lowers canonical SQLite-shaped SQL onto PostgreSQL. |
| [`binding_companions`](#binding_companions) | function | Catalog + Postgres identity companions for one canonical binding statement. |
| [`collapse_companion_groups`](#collapse_companion_groups) | function | Collapses interleaved companion results back to one result per original statement. |
| [`collapse_host_schema_results`](#collapse_host_schema_results) | function | Collapses adapter-expanded host-schema results using per-statement groups. |
| [`expand_binding_execute_request`](#expand_binding_execute_request) | function | Expands a binding request with adapter-private catalog/identity companions. |
| [`expand_host_schema_batch`](#expand_host_schema_batch) | function | Expands `[stmt, …, version_marker]` at the adapter execution edge. |
| [`expand_host_schema_execute_request`](#expand_host_schema_execute_request) | function | Expands a typed host schema batch at the adapter execution edge. |
| [`expand_host_schema_execute_request_grouped`](#expand_host_schema_execute_request_grouped) | function | `expand_host_schema_execute_request` plus collapse groups. |
| [`is_host_schema_version_marker`](#is_host_schema_version_marker) | function | True when `sql` is a host schema version marker (`bookclerk_schema_migrations` or `PRAGMA user_version`). |
| [`lower_binding_ddl_execute_request`](#lower_binding_ddl_execute_request) | function | Applies `lower_binding_sql_for_backend` to every statement in `req`. |
| [`lower_binding_sql_for_backend`](#lower_binding_sql_for_backend) | function | Mechanical type/identity lowering for one **binding** statement. |
| [`postgres_identity_companions`](#postgres_identity_companions) | function | Postgres-only companion DDL: transactional identity counter + BEFORE INSERT trigger. |
| [`realize_binding_ddl`](#realize_binding_ddl) | function | Applies one canonical binding DDL statement (plugin restore). |
| [`realize_host_ddl`](#realize_host_ddl) | function | Applies one canonical host DDL statement (library restore / schema). |
| [`schema_sql_for_backend`](#schema_sql_for_backend) | function | SQL text for one host-schema statement on the live connection backend. |
| [`db_value_from_sea`](#db_value_from_sea) | function | Reconstruct a typed cell from a SeaORM value. |
| [`db_value_to_sea`](#db_value_to_sea) | function | Convert a typed bind into a SeaORM value without JSON / `b64:` decoding. |
| [`execute_canonical_stamped`](#execute_canonical_stamped) | function | Stamp leftover canonical SQL, then execute it with sqlite-shaped transport. |
| [`execute_typed_envelope`](#execute_typed_envelope) | function | `execute_typed_on_session` using host-private proofs already stamped on `envelope`. |
| [`execute_typed_envelope_on_connection`](#execute_typed_envelope_on_connection) | function | `execute_typed_envelope` selecting the physical engine from `db`. |
| [`execute_typed_on_open_connection`](#execute_typed_on_open_connection) | function | `execute_typed_on_open_envelope` selecting the physical engine from `conn`. |
| [`execute_typed_on_open_envelope`](#execute_typed_on_open_envelope) | function | Run a stamped envelope on an already-open connection (no BEGIN/COMMIT). |
| [`execute_typed_on_session`](#execute_typed_on_session) | function | Run a typed atomic batch on an existing SeaORM connection. |
| [`execute_typed_on_session_then`](#execute_typed_on_session_then) | function | Like `execute_typed_on_session`, running extra statements in the same transaction before COMMIT (used to persist guest replay payloads on `bookclerk_receipts`). |
| [`execute_typed_on_txn`](#execute_typed_on_txn) | function | Run a typed batch on an already-open transaction (no BEGIN/COMMIT). |
| [`execute_typed_on_txn_envelope`](#execute_typed_on_txn_envelope) | function | `execute_typed_on_txn` with a host-only guest-receipt persist hint and proofs. |
| [`load_physical_sql_type_env`](#load_physical_sql_type_env) | function | Live physical tables on this connection (host execute only). |
| [`load_sql_type_env`](#load_sql_type_env) | function | Reconstructs `SqlTypeEnv` from the durable binding catalog, if present. |
| [`load_sql_type_env_capped`](#load_sql_type_env_capped) | function | `load_sql_type_env` paging at `min(max_result_rows, FIRST_PARTY_MAX_RESULT_ROWS)`. |
| [`proofs_for_host_plan`](#proofs_for_host_plan) | function | Host plans may include already-lowered schema companions (`PRAGMA`, `CREATE FUNCTION`, …) and greenfield DDL. Those get a hash-bound empty proof. Canonical DML is typed against the merged schema in statement order. |
| [`query_canonical_stamped`](#query_canonical_stamped) | function | Stamp leftover canonical SQL, then query it with sqlite-shaped transport. |
| [`stamp_adapter_execute`](#stamp_adapter_execute) | function | Host-stamped `AdapterExecuteRequest` for first-party adapter execute. |
| [`stamp_host_proofs`](#stamp_host_proofs) | function | Stamps 1:1 proofs for host-authored canonical SQL (planner / SeaORM). |

## Re-exports

- [`bookclerk_plugin_abi::DbPlanStatementKind`](../bookclerk_plugin_abi/db_execute.md#dbplanstatementkind)
- [`bookclerk_plugin_abi::AdapterExecuteRequest`](../bookclerk_plugin_abi/index.md#adapterexecuterequest)
- [`bookclerk_plugin_abi::GuestReceiptPersist`](../bookclerk_plugin_abi/index.md#guestreceiptpersist)
- [`guest_receipt::guest_receipt_applied_stmt`](guest_receipt.md#guest_receipt_applied_stmt)
- [`guest_receipt::guest_receipt_finalize_stmts`](guest_receipt.md#guest_receipt_finalize_stmts)
- [`guest_receipt::is_guest_receipt_result_lost`](guest_receipt.md#is_guest_receipt_result_lost)
- [`guest_receipt::is_guest_receipt_stub_insert`](guest_receipt.md#is_guest_receipt_stub_insert)
- [`guest_receipt::pad_skipped_guest_results`](guest_receipt.md#pad_skipped_guest_results)
- [`guest_receipt::prior_receipt_exists`](guest_receipt.md#prior_receipt_exists)
- [`guest_receipt::prior_receipt_is_claimed`](guest_receipt.md#prior_receipt_is_claimed)
- [`guest_receipt::prior_receipt_should_resume_guest`](guest_receipt.md#prior_receipt_should_resume_guest)
- [`guest_receipt::receipt_hash_from`](guest_receipt.md#receipt_hash_from)
- [`guest_receipt::should_skip_remaining_guest_work`](guest_receipt.md#should_skip_remaining_guest_work)
- [`guest_receipt::strip_guest_receipt_write_gate`](guest_receipt.md#strip_guest_receipt_write_gate)
- [`guest_receipt::GUEST_RECEIPT_RESULT_LOST`](guest_receipt.md#guest_receipt_result_lost)
- [`guest_receipt::GUEST_RECEIPT_STATUS_APPLIED`](guest_receipt.md#guest_receipt_status_applied)
- [`guest_receipt::GUEST_RECEIPT_STATUS_CLAIMED`](guest_receipt.md#guest_receipt_status_claimed)
- [`guest_receipt::GUEST_RECEIPT_STATUS_OK`](guest_receipt.md#guest_receipt_status_ok)
- [`guest_receipt::GUEST_RECEIPT_STUB_SUFFIX`](guest_receipt.md#guest_receipt_stub_suffix)
- [`guest_receipt::GUEST_RECEIPT_WRAP_PREFIX`](guest_receipt.md#guest_receipt_wrap_prefix)
- [`guest_receipt::GUEST_RECEIPT_WRITE_GATE`](guest_receipt.md#guest_receipt_write_gate)
- [`proxy_txn::arm_exec_budget`](proxy_txn.md#arm_exec_budget)
- [`proxy_txn::clear_exec_budget`](proxy_txn.md#clear_exec_budget)
- [`proxy_txn::consume_atomic_interrupt`](proxy_txn.md#consume_atomic_interrupt)
- [`proxy_txn::consume_begin_injection`](proxy_txn.md#consume_begin_injection)
- [`proxy_txn::consume_commit_injection`](proxy_txn.md#consume_commit_injection)
- [`proxy_txn::current_exec_budget`](proxy_txn.md#current_exec_budget)
- [`proxy_txn::exec_deadline_expired`](proxy_txn.md#exec_deadline_expired)
- [`proxy_txn::exec_deadline_remaining_ms`](proxy_txn.md#exec_deadline_remaining_ms)
- [`proxy_txn::inject_atomic_interrupt`](proxy_txn.md#inject_atomic_interrupt)
- [`proxy_txn::inject_atomic_interrupt_after`](proxy_txn.md#inject_atomic_interrupt_after)
- [`proxy_txn::inject_begin_failures`](proxy_txn.md#inject_begin_failures)
- [`proxy_txn::inject_commit_failures`](proxy_txn.md#inject_commit_failures)
- [`proxy_txn::inject_savepoint_release_failures`](proxy_txn.md#inject_savepoint_release_failures)
- [`proxy_txn::inject_savepoint_rollback_failures`](proxy_txn.md#inject_savepoint_rollback_failures)
- [`proxy_txn::is_txn_broken`](proxy_txn.md#is_txn_broken)
- [`proxy_txn::note_begin_failed`](proxy_txn.md#note_begin_failed)
- [`proxy_txn::note_commit_failed`](proxy_txn.md#note_commit_failed)
- [`proxy_txn::note_query_row`](proxy_txn.md#note_query_row)
- [`proxy_txn::query_row_cap`](proxy_txn.md#query_row_cap)
- [`proxy_txn::query_rows_seen`](proxy_txn.md#query_rows_seen)
- [`proxy_txn::record_query_rows_seen`](proxy_txn.md#record_query_rows_seen)
- [`proxy_txn::take_txn_fault`](proxy_txn.md#take_txn_fault)
- [`proxy_txn::txn_broken_err`](proxy_txn.md#txn_broken_err)
- [`proxy_txn::with_exec_budget`](proxy_txn.md#with_exec_budget)
- [`proxy_txn::AtomicInterruptKind`](proxy_txn.md#atomicinterruptkind)
- [`proxy_txn::AtomicInterruptPhase`](proxy_txn.md#atomicinterruptphase)
- [`proxy_txn::ExecBudget`](proxy_txn.md#execbudget)

## Modules

- [`guest_receipt`](guest_receipt.md): Guest-typed durable receipt finalize (same-transaction payload capture).
- [`proxy_txn`](proxy_txn.md): Fail-closed acknowledgements for SeaORM's infallible proxy transaction hooks.
- [`sql_v1`](sql_v1.md): Shared Bookclerk SQL v1 execution vectors (binding DDL + portable helpers).

## Structs

### AtomicSession

```rust
pub struct AtomicSession {
    pub cancel: Option<Arc<AtomicBool>>,
    pub deadline_unix_ms: Option<u64>,
    pub type_env: SqlTypeEnv,
}
```

Session-level cancel / deadline for one atomic attempt (not hashed).

**Fields**

- `cancel`: Host RPC cancel flag, when the guest shares a process with the host.
- `deadline_unix_ms`: Guest-visible deadline (`deadlineUnixMs` on the wire).
- `type_env`: Extra catalog snapshot merged before typing host-authored SQL
  (canonical host schema). Empty for plugin-binding execute.

#### AtomicSession::from_deadline

```rust
pub fn from_deadline(deadline_unix_ms: Option<u64>) -> Self
```

Builds session control from a wire `deadlineUnixMs`.

#### AtomicSession::with_cancel

```rust
pub fn with_cancel(self, cancel: Option<Arc<AtomicBool>>) -> Self
```

Attaches a host cancel flag (job fence) to this session.

#### AtomicSession::with_type_env

```rust
pub fn with_type_env(self, env: SqlTypeEnv) -> Self
```

Merges `env` (typically the canonical host schema) into this session so
host DML is typed against library tables, not the plugin catalog.

Implements: `Clone`, `Default`

### ExecCaps

```rust
pub struct ExecCaps {
    pub max_result_rows: u32,
    pub max_result_bytes: u32,
    pub max_cell_bytes: u32,
    pub max_atomic_result_bytes: u32,
}
```

Per-statement result bounds. Row/byte `0` means unlimited at execute time.

**Fields**

- `max_result_rows`: Maximum rows a statement may return (`0` = unlimited).
- `max_result_bytes`: Maximum JSON bytes of one statement's rows (`0` = unlimited).
- `max_cell_bytes`: Maximum UTF-8 / blob bytes of one cell (`0` = unlimited).
- `max_atomic_result_bytes`: Maximum encoded bytes of the whole atomic reply (`0` = unlimited).

#### ExecCaps::from_capabilities

```rust
pub fn from_capabilities(caps: &DbCapabilities) -> Self
```

Copies negotiated capability limits into the executor.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `From<u32>`

## Enums

### DbErrorClass

```rust
pub enum DbErrorClass {
    Unavailable,
    Conflict,
    Other,
}
```

Portable Bookclerk SQL error category.

**Variants**

- `Unavailable`: Retry the same operation after re-reading durable state.
- `Conflict`: Uniqueness or duplicate-object; re-read state, do not blindly retry DDL.
- `Other`: Permanent or unclassified failure (including FK / CHECK / NOT NULL).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### PhysicalEngine

```rust
pub enum PhysicalEngine {
    Sqlite,
    Postgres,
    D1,
}
```

Real engine the adapter opened.

Hosts must not construct this from a SeaORM proxy connection. Physical
lowering ([`crate::lower_canonical_sql`](#lower_canonical_sql)) happens only against this identity.
Unknown SeaORM backends fail closed — never default to SQLite.

**Variants**

- `Sqlite`: SQLite physical engine (rusqlite / files).
- `Postgres`: PostgreSQL 16+ physical engine.
- `D1`: Cloudflare D1 (sqlite-family SQL, HTTP batch).

#### PhysicalEngine::sqlite

```rust
pub const fn sqlite() -> Self
```

SQLite physical engine.

#### PhysicalEngine::postgres

```rust
pub const fn postgres() -> Self
```

PostgreSQL physical engine.

#### PhysicalEngine::d1

```rust
pub const fn d1() -> Self
```

Cloudflare D1 physical engine.

#### PhysicalEngine::from_adapter_backend

```rust
pub fn from_adapter_backend(backend: DatabaseBackend) -> Self
```

Adapter-only: the backend of a connection this adapter opened.

##### Panics

Panics on SeaORM backends that are not BookclerkSQL engines (including
MySQL and future unknown variants).

#### PhysicalEngine::backend

```rust
pub const fn backend(self) -> DatabaseBackend
```

SeaORM backend used for physical SQL and typed binds.

#### PhysicalEngine::timing_source

```rust
pub const fn timing_source(self) -> &'static str
```

Timing source label for this physical engine.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### SqlExecTarget

```rust
pub enum SqlExecTarget {
    Canonical,
    InProcess,
}
```

How leftover host SQL reaches a SeaORM connection.

[`Self::Canonical`](#sqlexectarget) is sqlite-shaped transport with no physical lowering
(production plugin-host proxy). [`Self::InProcess`](#sqlexectarget) lowers exactly once
using the engine of the opened adapter connection (native sqlite / postgres
tests). Host/library code must not name [`PhysicalEngine`](#physicalengine).

**Variants**

- `Canonical`: Canonical `?` SQL. Adapters lower after RPC.
- `InProcess`: In-process adapter connection. Infer the engine from the connection
  and lower once.

#### SqlExecTarget::lowers

```rust
pub const fn lowers(self) -> bool
```

True when leftover SQL is physically lowered by the adapter SDK.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

## Functions

### set_positional_result_columns

```rust
pub fn set_positional_result_columns(columns: Vec<DbColumn>)
```

Records engine column metadata for the in-flight query (SQLite rusqlite path).

Prefers the request-scoped [`ExecBudget`](proxy_txn.md#execbudget) so metadata survives `spawn_blocking`.

### take_positional_result_columns

```rust
pub fn take_positional_result_columns() -> Option<Vec<DbColumn>>
```

Takes engine column metadata recorded by the SQLite adapter, if any.

### b64_string_to_bytes

```rust
pub fn b64_string_to_bytes(s: &str) -> Option<Vec<u8>>
```

Decode a `b64:`-prefixed string. Returns `None` if not prefixed.

### bytes_to_b64_string

```rust
pub fn bytes_to_b64_string(bytes: &[u8]) -> String
```

Encode bytes as a `b64:`-prefixed JSON string.

### apply_begin_isolation

```rust
pub async fn apply_begin_isolation<C>(
    conn: &C,
    isolation: IsolationReq,
) -> Result<(), DbErr>
where
    C: ConnectionTrait,
```

Realizes [`IsolationReq`](../bookclerk_plugin_abi/db_execute.md#isolationreq) on an already-open transaction.

#### Errors

Returns when the engine rejects the isolation statement.

### assert_restore_constraints

```rust
pub async fn assert_restore_constraints<C>(conn: &C) -> Result<(), DbErr>
where
    C: ConnectionTrait,
```

SQLite: `PRAGMA foreign_key_check` must be empty before commit.

#### Errors

Returns when violations remain or the pragma fails.

### begin_consistent_snapshot

```rust
pub async fn begin_consistent_snapshot(
    db: &DatabaseConnection,
) -> Result<DatabaseTransaction, DbErr>
```

Begins a consistent capture transaction.

Sets `IsolationReq::ConsistentSnapshot` for RPC proxies, and applies
`SET TRANSACTION ISOLATION LEVEL REPEATABLE READ` on a native Postgres
connection (the host RPC proxy always reports the canonical sqlite backend).

#### Errors

Returns when `BEGIN` or snapshot isolation fails.

### drop_user_relations

```rust
pub async fn drop_user_relations<C>(conn: &C, names: &[String]) -> Result<(), DbErr>
where
    C: ConnectionTrait,
```

Drops named relations. PostgreSQL uses `CASCADE` plus identity functions.

#### Errors

Returns when a drop fails (other than IF EXISTS no-ops).

### export_identity

```rust
pub async fn export_identity<C>(conn: &C) -> Result<Vec<DbIdentityHighWater>, DbErr>
where
    C: ConnectionTrait,
```

Identity high-water from `sqlite_sequence` and [`SQL_IDENTITY_TABLE`](../bookclerk_plugin_abi/sql_types.md#sql_identity_table).

#### Errors

Returns when a catalog query fails for a reason other than a missing catalog.

### import_identity

```rust
pub async fn import_identity<C>(
    conn: &C,
    rows: &[DbIdentityHighWater],
) -> Result<(), DbErr>
where
    C: ConnectionTrait,
```

Writes identity high-water into adapter catalogs.

#### Errors

Returns when a catalog write fails.

### list_user_relations

```rust
pub async fn list_user_relations<C>(conn: &C) -> Result<Vec<String>, DbErr>
where
    C: ConnectionTrait,
```

User-visible base tables in the current schema.

#### Errors

Returns when the catalog probe fails.

### pending_begin_isolation

```rust
pub fn pending_begin_isolation() -> IsolationReq
```

Isolation the next SeaORM `begin()` should ask the adapter to realize.

### prepare_unit_restore

```rust
pub async fn prepare_unit_restore<C>(conn: &C) -> Result<(), DbErr>
where
    C: ConnectionTrait,
```

SQLite: `PRAGMA defer_foreign_keys = ON` on the restore transaction.

#### Errors

Returns when the pragma fails.

### canonical_statement

```rust
pub fn canonical_statement(
    sql: impl Into<String>,
    values: impl IntoIterator<Item = Value>,
) -> Statement
```

Builds a SeaORM statement that keeps canonical `?` placeholders.

### execute_canonical

```rust
pub async fn execute_canonical<C>(
    db: &C,
    sql: &str,
    values: impl IntoIterator<Item = Value>,
) -> Result<ExecResult, DbErr>
where
    C: ConnectionTrait,
```

Executes already-canonical host SQL. Never desugars or physically lowers.

#### Errors

Returns when the connection rejects the statement.

### query_canonical

```rust
pub async fn query_canonical<C>(
    db: &C,
    sql: &str,
    values: impl IntoIterator<Item = Value>,
) -> Result<Vec<QueryResult>, DbErr>
where
    C: ConnectionTrait,
```

Queries already-canonical host SQL. Never desugars or physically lowers.

#### Errors

Returns when the connection rejects the statement.

### classify_db_err

```rust
pub fn classify_db_err(err: &DbErr) -> DbErrorClass
```

Classifies `err` from adapter-stamped codes, not English engine messages.

### classify_db_err_message

```rust
pub fn classify_db_err_message(message: &str) -> DbErrorClass
```

Classifies an already-formatted adapter/ORM error string.

### is_schema_apply_retryable

```rust
pub fn is_schema_apply_retryable(err: &DbErr) -> bool
```

True when host schema apply may retry after re-reading durable schema state.

Uniqueness / duplicate-object races and transient unavailability qualify.
Semantic integrity failures do not.

### cap_query_sql

```rust
pub fn cap_query_sql(sql: &str, max_result_rows: u32) -> String
```

Wraps a host-tagged `bookclerk_plugin_abi::DbPlanStatementKind::Select` so the engine stops
after `cap + 1` rows. Callers must not pass DML `RETURNING` SQL.

### encoded_proxy_row_len

```rust
pub fn encoded_proxy_row_len<'a>(
    values: impl IntoIterator<Item = (&'a String, &'a Value)>,
) -> usize
```

Encoded JSON length of a SeaORM proxy row (keys, numbers, punctuation).

### execute_physical_sql

```rust
pub async fn execute_physical_sql<C>(
    engine: PhysicalEngine,
    db: &C,
    sql: &str,
    values: impl IntoIterator<Item = Value>,
) -> Result<ExecResult, DbErr>
where
    C: ConnectionTrait,
```

Executes canonical SQL on a physical engine the adapter opened.

Does **not** run host semantic desugars. Callers must pass already-canonical
SQL (`?` placeholders). Host/library code must not call this helper.

#### Errors

Returns when the engine rejects the lowered statement.

### json_cell_utf8_len

```rust
pub fn json_cell_utf8_len(v: &Value) -> usize
```

UTF-8 / JSON length of one cell used for `maxCellBytes`.

### note_encoded_result_bytes

```rust
pub fn note_encoded_result_bytes(
    used: &mut usize,
    extra: usize,
    max_result_bytes: u32,
) -> Result<(), DbErr>
```

Adds `extra` encoded JSON bytes toward `max_result_bytes`.

#### Errors

Returns `DbErr::Custom` when the running total would exceed the cap.

### query_physical_sql

```rust
pub async fn query_physical_sql<C>(
    engine: PhysicalEngine,
    db: &C,
    sql: &str,
    values: impl IntoIterator<Item = Value>,
) -> Result<Vec<QueryResult>, DbErr>
where
    C: ConnectionTrait,
```

Queries canonical SQL on a physical engine the adapter opened.

#### Errors

Returns when the engine rejects the lowered statement.

### query_physical_sql_typed

```rust
pub async fn query_physical_sql_typed<C>(
    engine: PhysicalEngine,
    db: &C,
    sql: &str,
    proof: Option<&ResolvedStatement>,
    values: impl IntoIterator<Item = Value>,
) -> Result<Vec<QueryResult>, DbErr>
where
    C: ConnectionTrait,
```

[`query_physical_sql`](#query_physical_sql) with an optional hash-bound proof (TEXT collate /
INTEGER overflow on Postgres).

#### Errors

Returns when the proof is not bound to `sql` or the engine rejects the work.

### sea_value_to_json

```rust
pub fn sea_value_to_json(v: &Value) -> Value
```

Maps a SeaORM cell onto JSON for plan interpretation.

### lower_canonical_ddl_to_postgres

```rust
pub fn lower_canonical_ddl_to_postgres(sql: &str) -> Result<String, PluginError>
```

Lowers one canonical Bookclerk **DDL** statement onto PostgreSQL.

Type/identity rewrites plus [`lower_canonical_to_postgres`](#lower_canonical_to_postgres) (placeholders,
`INSERT OR IGNORE`). Host schema packs use this through
[`crate::schema_sql_for_backend`](#schema_sql_for_backend); binding DDL uses the type rewrite at
the adapter edge and then the DML helper pass.
#### Errors

Returns when a Postgres `CREATE INDEX` has a `COLLATE NOCASE` tie-break or
a `lower`/`upper` column argument whose type is not in `env` (and this
call passes no env). The statement is not rewritten into a different index.

### lower_canonical_ddl_to_postgres_with

```rust
pub fn lower_canonical_ddl_to_postgres_with(
    sql: &str,
    env: Option<&SqlTypeEnv>,
) -> Result<String, PluginError>
```

[`lower_canonical_ddl_to_postgres`](#lower_canonical_ddl_to_postgres) using `env` for tie-break column types.

#### Errors

Returns when a Postgres `CREATE INDEX` key that needs `COLLATE "C"` cannot
be typed from `env`.

### lower_canonical_sql

```rust
pub fn lower_canonical_sql(backend: DatabaseBackend, sql: &str) -> String
```

Lowers canonical Bookclerk **DML/query** SQL for `backend`.

Every backend rewrites `INSERT OR IGNORE` to unique/PK `ON CONFLICT DO
NOTHING` (SQLite `OR IGNORE` would otherwise swallow `NOT NULL`). Postgres
adapters then rewrite helpers (`IFNULL`, `json_object` → TEXT,
`json_extract`, 2+-arg `min`/`max`, `json_valid`, `round`/`sum`/`avg`)
and `?` placeholders. Default `ORDER BY` NULLS and `/` `%` by-zero
`NULLIF` are host semantic desugars, not adapter rewrites. Binding and host
**DDL** type/identity rewrites
(`AUTOINCREMENT`, `BLOB`, `INTEGER`) stay on the adapter execution edge
([`crate::schema_sql_for_backend`](#schema_sql_for_backend), [`crate::lower_binding_ddl_execute_request`](#lower_binding_ddl_execute_request))
so this function does not classify statements.

### lower_canonical_sql_typed

```rust
pub fn lower_canonical_sql_typed(
    backend: DatabaseBackend,
    sql: &str,
    proof: Option<&ResolvedStatement>,
) -> Result<String, PluginError>
```

[`lower_canonical_sql`](#lower_canonical_sql) plus proof-directed overflow and Postgres `COLLATE "C"`.

`proof` must be bound to `sql` (the canonical string before mechanical
lowering). When absent, TEXT collation and INTEGER overflow wraps are not
applied. TEXT collation runs **before** INTEGER overflow wrapping so a
TEXT span nested in `length(name) + 1` is remapped into the overflow
operands. Overflow sites are applied innermost-first.

#### Errors

Returns when `proof` is present and not bound to `sql`, or when a
hash-correct sidecar cannot be applied (`ResolvedStatement::validate_for`).

### lower_canonical_to_postgres

```rust
pub fn lower_canonical_to_postgres(sql: &str) -> String
```

Lowers canonical SQLite-shaped SQL onto PostgreSQL.

### binding_companions

```rust
pub fn binding_companions(backend: DatabaseBackend, canonical: &str) -> Vec<String>
```

Catalog + Postgres identity companions for one canonical binding statement.

### collapse_companion_groups

```rust
pub fn collapse_companion_groups(
    groups: &[usize],
    results: Vec<StatementResult>,
) -> Vec<StatementResult>
```

Collapses interleaved companion results back to one result per original statement.

### collapse_host_schema_results

```rust
pub fn collapse_host_schema_results(
    groups: &[usize],
    results: Vec<StatementResult>,
) -> Vec<StatementResult>
```

Collapses adapter-expanded host-schema results using per-statement groups.

### expand_binding_execute_request

```rust
pub fn expand_binding_execute_request(
    backend: DatabaseBackend,
    req: &ExecuteRequest,
) -> (ExecuteRequest, Vec<usize>)
```

Expands a binding request with adapter-private catalog/identity companions.

Returns the expanded request and per-original-statement group sizes so
callers can collapse results back to the wire statement count.

### expand_host_schema_batch

```rust
pub fn expand_host_schema_batch(
    backend: DatabaseBackend,
    batch: &[String],
) -> Result<Option<Vec<String>>, PluginError>
```

Expands `[stmt, …, version_marker]` at the adapter execution edge.

Each incoming string is already one canonical statement. Adapters lower
DDL per statement and may insert identity companions; they do not split
SQL. Non-DDL statements stay canonical so their host proofs still bind
when the executor lowers them.
#### Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

### expand_host_schema_execute_request

```rust
pub fn expand_host_schema_execute_request(
    backend: DatabaseBackend,
    req: &ExecuteRequest,
) -> Result<ExecuteRequest, PluginError>
```

Expands a typed host schema batch at the adapter execution edge.

Returns the expanded request. When the batch is not a host schema apply
unit, the request is unchanged.

#### Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

### expand_host_schema_execute_request_grouped

```rust
pub fn expand_host_schema_execute_request_grouped(
    backend: DatabaseBackend,
    req: &ExecuteRequest,
) -> Result<(ExecuteRequest, Vec<usize>), PluginError>
```

[`expand_host_schema_execute_request`](#expand_host_schema_execute_request) plus collapse groups.

#### Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

### is_host_schema_version_marker

```rust
pub fn is_host_schema_version_marker(sql: &str) -> bool
```

True when `sql` is a host schema version marker (`bookclerk_schema_migrations` or `PRAGMA user_version`).

### lower_binding_ddl_execute_request

```rust
pub fn lower_binding_ddl_execute_request(
    backend: DatabaseBackend,
    req: &ExecuteRequest,
) -> Result<ExecuteRequest, PluginError>
```

Applies [`lower_binding_sql_for_backend`](#lower_binding_sql_for_backend) to every statement in `req`.

Identity when no statement changes (SQLite/D1, or Postgres DML-only).

#### Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

### lower_binding_sql_for_backend

```rust
pub fn lower_binding_sql_for_backend(
    backend: DatabaseBackend,
    sql: &str,
) -> Result<Cow<'_, str>, PluginError>
```

Mechanical type/identity lowering for one **binding** statement.

Hosts emit canonical SQLite-shaped `CREATE`/`DROP`. Postgres adapters
rewrite `AUTOINCREMENT`/`BLOB`/`INTEGER`/`REAL` here and turn
`ident COLLATE NOCASE` into `(lower(ident COLLATE "C"))`, with a text
tie-break as `(ident COLLATE "C")`, so page order matches SQLite.
Numeric tie-break keys stay bare. SQLite leaves the statement unchanged,
including `NOCASE` page indexes. DML stays for [`crate::lower_canonical_sql`](#lower_canonical_sql).

#### Panics

Panics when `backend` is not SQLite or PostgreSQL. `DatabaseBackend` is
non-exhaustive; BookclerkSQL adapters do not implement other SeaORM
families.
#### Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

### postgres_identity_companions

```rust
pub fn postgres_identity_companions(sql: &str) -> Vec<String>
```

Postgres-only companion DDL: transactional identity counter + BEFORE INSERT trigger.

SQLite `AUTOINCREMENT` records the largest committed value. A PostgreSQL
`setval` sequence does not roll back with the `ExecuteRequest`. The adapter
keeps a `BIGINT` high-water row in [`bookclerk_plugin_abi::SQL_IDENTITY_TABLE`](../bookclerk_plugin_abi/sql_types.md#sql_identity_table)
and assigns omit-ids from that row under `SELECT … FOR UPDATE`. Companions
are generated from **canonical** SQL (parsed identity column, not assumed
`id`) and executed internally (not extra Cap'n statements). Empty when `sql`
is not `CREATE TABLE` with `INTEGER PRIMARY KEY AUTOINCREMENT`.

### realize_binding_ddl

```rust
pub async fn realize_binding_ddl<C>(conn: &C, canonical: &str) -> Result<(), DbErr>
where
    C: ConnectionTrait,
```

Applies one canonical binding DDL statement (plugin restore).

Includes adapter-private catalog DML and Postgres identity companions.

#### Errors

Returns when the engine rejects the lowered DDL or a companion.

### realize_host_ddl

```rust
pub async fn realize_host_ddl<C>(conn: &C, canonical: &str) -> Result<(), DbErr>
where
    C: ConnectionTrait,
```

Applies one canonical host DDL statement (library restore / schema).

Postgres identity companions are installed; binding catalog DML is not.

#### Errors

Returns when the engine rejects the lowered DDL or a companion.

### schema_sql_for_backend

```rust
pub fn schema_sql_for_backend(
    backend: DatabaseBackend,
    canonical: &str,
) -> Result<Cow<'_, str>, PluginError>
```

SQL text for one host-schema statement on the live connection backend.

SQLite-family backends return the canonical statement unchanged; Postgres
applies the mechanical DDL lowering. Unknown SeaORM backends fail closed.

#### Panics

Panics when `backend` is not SQLite or PostgreSQL. `DatabaseBackend` is
non-exhaustive; BookclerkSQL adapters do not implement other SeaORM
families.
#### Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

### db_value_from_sea

```rust
pub fn db_value_from_sea(v: &Value) -> Result<DbValue, String>
```

Reconstruct a typed cell from a SeaORM value.

Unlike the JSON guest path, this preserves engine types: a BLOB column
becomes `DbValue::Bytes`, and `b64:` text is left as `DbValue::Text`.

#### Errors

Returns when the SeaORM value is outside the universal domain, or TEXT
contains U+0000.

### db_value_to_sea

```rust
pub fn db_value_to_sea(value: &DbValue) -> Value
```

Convert a typed bind into a SeaORM value without JSON / `b64:` decoding.

### execute_canonical_stamped

```rust
pub async fn execute_canonical_stamped<C>(
    conn: &C,
    sql: &str,
    values: impl IntoIterator<Item = Value>,
    type_env: &SqlTypeEnv,
) -> Result<u64, DbErr>
where
    C: ConnectionTrait + StreamTrait,
```

Stamp leftover canonical SQL, then execute it with sqlite-shaped transport.

Production leftover goes through SeaORM `execute_raw` on the sqlite-shaped
proxy, which stamps with `bookclerk_plugin_abi::CanonicalExecuteRequest::from_desugared`
(no second desugar). Host leftover must therefore desugar `ORDER BY NULLS`
/ `NULLIF` **before** transport. Does not start a nested transaction.

#### Errors

Returns when typecheck fails or the connection rejects the statement.

### execute_typed_envelope

```rust
pub async fn execute_typed_envelope(
    engine: PhysicalEngine,
    db: &DatabaseConnection,
    envelope: &AdapterExecuteRequest,
    timing_source: &str,
    caps: impl Into<ExecCaps>,
    session: AtomicSession,
) -> Result<ExecuteReply, DbErr>
```

[`execute_typed_on_session`](#execute_typed_on_session) using host-private proofs already stamped on `envelope`.

Envelope proofs are required 1:1 with `envelope.request.statements`. Catalog
changes after admission must not rebuild lowering. Pre-admission public
execute uses [`execute_typed_on_session`](#execute_typed_on_session), which may typecheck.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when proofs are missing or mismatched, a statement fails,
the encoded reply exceeds `max_atomic_result_bytes`, or the session is
interrupted.

### execute_typed_envelope_on_connection

```rust
pub async fn execute_typed_envelope_on_connection(
    db: &DatabaseConnection,
    envelope: &AdapterExecuteRequest,
    caps: impl Into<ExecCaps>,
    session: AtomicSession,
) -> Result<ExecuteReply, DbErr>
```

[`execute_typed_envelope`](#execute_typed_envelope) selecting the physical engine from `db`.

In-process sqlite/postgres tests call this instead of naming
[`PhysicalEngine`](#physicalengine) in host/library code. Production RPC paths send the
envelope to adapter `execute` and must not call this on the sqlite-shaped
proxy.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when proofs are missing or mismatched, a statement fails,
the encoded reply exceeds `max_atomic_result_bytes`, or the session is
interrupted.

### execute_typed_on_open_connection

```rust
pub async fn execute_typed_on_open_connection<C>(
    conn: &C,
    envelope: &AdapterExecuteRequest,
    caps: impl Into<ExecCaps>,
    session: AtomicSession,
    describe: Option<&DatabaseConnection>,
) -> Result<ExecuteReply, DbErr>
where
    C: ConnectionTrait + StreamTrait,
```

[`execute_typed_on_open_envelope`](#execute_typed_on_open_envelope) selecting the physical engine from `conn`.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when proofs are missing, a statement fails, or the encoded
reply exceeds `max_atomic_result_bytes`.

### execute_typed_on_open_envelope

```rust
pub async fn execute_typed_on_open_envelope<C>(
    engine: PhysicalEngine,
    conn: &C,
    envelope: &AdapterExecuteRequest,
    timing_source: &str,
    caps: impl Into<ExecCaps>,
    session: AtomicSession,
    describe: Option<&DatabaseConnection>,
) -> Result<ExecuteReply, DbErr>
where
    C: ConnectionTrait + StreamTrait,
```

Run a stamped envelope on an already-open connection (no BEGIN/COMMIT).

In-process postgres tests use this when the caller already holds a SeaORM
connection or transaction and must physically lower canonical SQL. Host
RPC/proxy paths must not call this — they send [`AdapterExecuteRequest`](../bookclerk_plugin_abi/index.md#adapterexecuterequest).

Binding catalog companions are skipped: leftover DML and restore DDL run
against a catalog the caller already applied (`apply_host_schema` or
[`bookclerk_plugin_abi::catalog_companions`](../bookclerk_plugin_abi/sql_types.md#catalog_companions)). Re-inserting those rows
conflicts with `bookclerk_sql_catalog_pkey`. Postgres identity companions
still run so restore can recreate `bookclerk_identity` after dropping it.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when proofs are missing, a statement fails, or the encoded
reply exceeds `max_atomic_result_bytes`.

### execute_typed_on_session

```rust
pub async fn execute_typed_on_session(
    db: &DatabaseConnection,
    req: &ExecuteRequest,
    guest_receipt: GuestReceiptPersist,
    timing_source: &str,
    caps: impl Into<ExecCaps>,
    session: AtomicSession,
) -> Result<ExecuteReply, DbErr>
```

Run a typed atomic batch on an existing SeaORM connection.

Encodes the [`ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#executereply) **before** COMMIT. Encoding or result-budget
failures roll the transaction back.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when a statement fails, the encoded reply exceeds
`max_atomic_result_bytes`, TEXT contains U+0000, or the session is
interrupted.

### execute_typed_on_session_then

```rust
pub async fn execute_typed_on_session_then<F>(
    db: &DatabaseConnection,
    req: &ExecuteRequest,
    timing_source: &str,
    caps: impl Into<ExecCaps>,
    session: AtomicSession,
    then: F,
    guest_hash: Option<String>,
) -> Result<ExecuteReply, DbErr>
where
    F: FnOnce(ExecuteReply) -> Result<Vec<TypedDbStatement>, DbErr>,
```

Like [`execute_typed_on_session`](#execute_typed_on_session), running extra statements in the same transaction
before COMMIT (used to persist guest replay payloads on `bookclerk_receipts`).

`guest_hash` is the guest `requestHash` used to decide whether a claimed
prior receipt should resume remaining guest SQL instead of skipping it.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when a statement fails, `then` fails, encoding fails, or COMMIT fails.

### execute_typed_on_txn

```rust
pub async fn execute_typed_on_txn(
    txn: &DatabaseTransaction,
    req: &ExecuteRequest,
    timing_source: &str,
    caps: impl Into<ExecCaps>,
    session: AtomicSession,
    describe: Option<&DatabaseConnection>,
) -> Result<ExecuteReply, DbErr>
```

Run a typed batch on an already-open transaction (no BEGIN/COMMIT).

Used by nested SeaORM work: the guest interactive txn is already open, so
a second `executeAtomic` BEGIN would fail. The batch runs inside a
`SAVEPOINT`; statement, encoding, or budget failures roll back to that
savepoint so a later outer `commit()` cannot persist a partial batch.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when a statement fails or the encoded reply exceeds
`max_atomic_result_bytes`.

### execute_typed_on_txn_envelope

```rust
pub async fn execute_typed_on_txn_envelope(
    engine: PhysicalEngine,
    txn: &DatabaseTransaction,
    envelope: &AdapterExecuteRequest,
    timing_source: &str,
    caps: impl Into<ExecCaps>,
    session: AtomicSession,
    describe: Option<&DatabaseConnection>,
) -> Result<ExecuteReply, DbErr>
```

[`execute_typed_on_txn`](#execute_typed_on_txn) with a host-only guest-receipt persist hint and proofs.

Finalize SQL runs inside the nested savepoint before returning, so an outer
`commit()` persists the caller-visible payload.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when a statement fails, finalize fails, or the encoded
reply exceeds `max_atomic_result_bytes`.

### load_physical_sql_type_env

```rust
pub async fn load_physical_sql_type_env(
    conn: &impl ConnectionTrait,
) -> Result<SqlTypeEnv, DbErr>
```

Live physical tables on this connection (host execute only).

Plugin bindings type against [`SQL_CATALOG_TABLE`](../bookclerk_plugin_abi/sql_types.md#sql_catalog_table), not sqlite_master.
Host plans may query tables created outside the canonical host snapshot
(tests, `PRAGMA`/raw DDL); those columns are merged here so typing still
runs.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the physical catalog query fails.

### load_sql_type_env

```rust
pub async fn load_sql_type_env(conn: &impl ConnectionTrait) -> Result<SqlTypeEnv, DbErr>
```

Reconstructs [`SqlTypeEnv`](../bookclerk_plugin_abi/sql_types.md#sqltypeenv) from the durable binding catalog, if present.

A missing catalog table yields an empty environment. RPC/adapter/shape/type
errors fail closed. On PostgreSQL the probe runs under a savepoint so a
missing-table `SELECT` cannot abort the current transaction (`25P02`).

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the catalog query fails for a reason other than a
missing table, or when a catalog cell is malformed.

### load_sql_type_env_capped

```rust
pub async fn load_sql_type_env_capped(
    conn: &impl ConnectionTrait,
    max_result_rows: u32,
) -> Result<SqlTypeEnv, DbErr>
```

[`load_sql_type_env`](#load_sql_type_env) paging at `min(max_result_rows, FIRST_PARTY_MAX_RESULT_ROWS)`.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the catalog query fails for a reason other than a
missing table, or when a catalog cell is malformed.

### proofs_for_host_plan

```rust
pub fn proofs_for_host_plan(
    req: &ExecuteRequest,
    env: &SqlTypeEnv,
) -> Result<Vec<ResolvedStatement>, DbErr>
```

Host plans may include already-lowered schema companions (`PRAGMA`,
`CREATE FUNCTION`, …) and greenfield DDL. Those get a hash-bound empty
proof. Canonical DML is typed against the merged schema in statement order.

#### Errors

Returns `DbErr::Custom` when typecheck fails or a statement yields no proof.

### query_canonical_stamped

```rust
pub async fn query_canonical_stamped<C>(
    conn: &C,
    sql: &str,
    values: impl IntoIterator<Item = Value>,
    type_env: &SqlTypeEnv,
) -> Result<Vec<QueryResult>, DbErr>
where
    C: ConnectionTrait + StreamTrait,
```

Stamp leftover canonical SQL, then query it with sqlite-shaped transport.

Same desugar-before-transport rule as [`execute_canonical_stamped`](#execute_canonical_stamped). Does
not start a nested transaction.

#### Errors

Returns when typecheck fails or the connection rejects the statement.

### stamp_adapter_execute

```rust
pub fn stamp_adapter_execute(
    request: ExecuteRequest,
    catalog: &SqlTypeEnv,
) -> Result<AdapterExecuteRequest, DbErr>
```

Host-stamped [`AdapterExecuteRequest`](../bookclerk_plugin_abi/index.md#adapterexecuterequest) for first-party adapter execute.

Desugars canonical SQL exactly once (`ORDER BY NULLS`, `NULLIF`) so proofs
bind to the same text that crosses the adapter boundary. Adapters must not
call this.

#### Errors

Returns `DbErr::Custom` when typecheck fails, proofs do not bind, or a
TEXT SQL/bind contains U+0000.

### stamp_host_proofs

```rust
pub fn stamp_host_proofs(
    req: &ExecuteRequest,
    catalog: &SqlTypeEnv,
) -> Result<Vec<ResolvedStatement>, DbErr>
```

Stamps 1:1 proofs for host-authored canonical SQL (planner / SeaORM).

Host companion SQL (`PRAGMA`, identity functions, DDL) gets a hash-bound
empty proof. Canonical DML is typed against bookkeeping + catalog tables
plus `catalog`.

#### Errors

Returns `DbErr::Custom` when typecheck fails.
