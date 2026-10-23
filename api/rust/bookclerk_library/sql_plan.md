# bookclerk_library::sql_plan

Host-owned generic SQL atomic plans for database adapters.
Host-owned SQL atomic plans for database plugins.

The library compiles Bookclerk domain operations into a typed
[`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest) plus host-only
[`crate::sql_plan::host_ir::AtomicSelection`](sql_plan/host_ir.md#atomicselection) indexes. Database guests
execute the statements as one transaction and return a typed
[`bookclerk_plugin_abi::ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#executereply).

| Item | Kind | Summary |
| --- | --- | --- |
| [`PlanStmtResult`](#planstmtresult) | struct | Rows produced by one plan statement. |
| [`CompiledAtomic`](#compiledatomic) | struct | Compiled typed request plus host result-selection indexes. |
| [`execute_compiled_on`](#execute_compiled_on) | function | Executes a compiled named atomic as one transaction and interprets the reply. |
| [`execute_compiled_on_capped`](#execute_compiled_on_capped) | function | Like `execute_compiled_on`, failing when a statement returns more than `max_result_rows`. |
| [`execute_typed_on`](#execute_typed_on) | function | Executes a typed request as one transaction on the opened adapter connection against the host library. |
| [`execute_typed_on_binding`](#execute_typed_on_binding) | function | Typed execute against an isolated plugin binding. |
| [`execute_typed_on_session`](#execute_typed_on_session) | function | `execute_typed_on` with session cancel / deadline checks. |
| [`event_inflight_slot`](#event_inflight_slot) | function | Slot key that serializes in-flight event deliveries for one plugin class. |
| [`lock_serialization_slot`](#lock_serialization_slot) | function | Inserts the slot row if missing, then bumps it (transaction-scoped write lock). |
| [`run_typed_conn_vectors`](#run_typed_conn_vectors) | function | Runs the shared contract suite through the host stamp + adapter execute path. |
| [`run_typed_contract_vectors`](#run_typed_contract_vectors) | function | Native typed contract suite (`ExecuteRequest` → `ExecuteReply`). |
| [`run_typed_request_vectors`](#run_typed_request_vectors) | function | Runs the shared contract suite through a typed `ExecuteRequest` callback. |
| [`stamp_typed_vector`](#stamp_typed_vector) | function | Desugars and stamps proofs for one vector, recording DDL in `catalog`. |
| [`validate_plan`](#validate_plan) | function | Rejects a compiled plan that exceeds negotiated guest limits or has out-of-range selectors. |
| [`validate_execute_request`](#validate_execute_request) | function | Rejects a typed `ExecuteRequest` that exceeds negotiated guest limits. |
| [`validate_proven_execute_request`](#validate_proven_execute_request) | function | After host typecheck, reject a resolved batch whose proven lowered size exceeds negotiated `maxLoweredStatementBytes`. |
| [`authorize_typed_request`](#authorize_typed_request) | function | Host-authorizes a guest typed batch: overwrite statement kinds, enforce negotiated caps, and stamp the canonical Cap'n request hash. |
| [`execute_guest_atomic_with`](#execute_guest_atomic_with) | function | Authorizes, receipt-wraps, executes, and unwraps one guest typed batch. |
| [`authorize_guest_typed_request`](#authorize_guest_typed_request) | function | Authorizes a **guest-authored** typed batch. |
| [`wake_page_for_max_binds`](#wake_page_for_max_binds) | function | Wake page size from negotiated `maxBinds` (SET/EXISTS overhead is 4 binds). |
| [`host_statement_kind`](#host_statement_kind) | function | Host-authored statement shape for the wire `kind` field. |
| [`proxy_read_kind`](#proxy_read_kind) | function | Kind for a SeaORM proxy **read**. `SELECT`/`WITH`/`VALUES` become `Select` (LIMIT-wrapped). `RETURNING` stays `Returning`. Everything else (PRAGMA, schema introspection) is `Returning` so it is not LIMIT-wrapped. |
| [`proxy_write_kind`](#proxy_write_kind) | function | Kind for a SeaORM proxy **write**. `RETURNING` is preserved; other DML is `Execute`. |
| [`proxy_insert_returning_identity`](#proxy_insert_returning_identity) | function | Rewrites a SeaORM proxy single-row `INSERT` to return the table's identity key. |
| [`JOB_QUEUE_SLOT`](#job_queue_slot) | constant | Slot key for job admit / claim / scratch-quota serialization. |

## Re-exports

- [`bookclerk_plugin_abi::DbPlanStatementKind`](../bookclerk_plugin_abi/db_execute.md#dbplanstatementkind)
- [`sql_plan::host_ir::AtomicSelection`](sql_plan/host_ir.md#atomicselection)
- [`exec::AtomicSession`](../bookclerk_db_exec/index.md#atomicsession)

## Modules

- [`host_ir`](sql_plan/host_ir.md): Domain result-selection metadata kept beside a typed `bookclerk_plugin_abi::ExecuteRequest`.

## Structs

### PlanStmtResult

```rust
pub struct PlanStmtResult {
    pub rows: Vec<Value>,
    pub rows_affected: u64,
}
```

Rows produced by one plan statement.

**Fields**

- `rows`: Result-set rows (empty for DML).
- `rows_affected`: Engine-reported rows affected.

Implements: `Clone`, `Debug`, `Default`, `From<&StatementResult>`

### CompiledAtomic

```rust
pub struct CompiledAtomic {
    pub request: ExecuteRequest,
    pub selection: AtomicSelection,
    pub expected_hash: String,
}
```

Compiled typed request plus host result-selection indexes.

**Fields**

- `request`: Typed request executed by the guest. Its wire `request_hash` is empty;
  [`authorize_typed_request`](#authorize_typed_request) stamps the canonical Cap'n digest.
- `selection`: Host indexes used to interpret the typed reply.
- `expected_hash`: Domain receipt hash (SHA-256 hex) bound into the receipt SQL and
  compared on receipt replay. Never sent as the wire `request_hash`.

#### CompiledAtomic::into_typed_request

```rust
pub fn into_typed_request(self, operation_id: impl Into<String>) -> ExecuteRequest
```

Typed [`ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest) for adapter `execute` (Cap'n data plane).

Implements: `Clone`, `Debug`

## Functions

### execute_compiled_on

```rust
pub async fn execute_compiled_on(
    db: &DatabaseConnection,
    compiled: CompiledAtomic,
) -> Result<DbAtomicResult>
```

Executes a compiled named atomic as one transaction and interprets the reply.

#### Errors

Returns [`LibraryError::Orm`](index.md#libraryerror) when a statement fails. Application statuses
are returned as [`DbAtomicResult`](index.md#dbatomicresult), not errors.

### execute_compiled_on_capped

```rust
pub async fn execute_compiled_on_capped(
    db: &DatabaseConnection,
    compiled: CompiledAtomic,
    max_result_rows: u32,
) -> Result<DbAtomicResult>
```

Like [`execute_compiled_on`](#execute_compiled_on), failing when a statement returns more than `max_result_rows`.

`max_result_rows` of `0` means unlimited.

#### Errors

Returns [`LibraryError::Orm`](index.md#libraryerror) when a statement fails or exceeds the row cap.

### execute_typed_on

```rust
pub async fn execute_typed_on(
    db: &DatabaseConnection,
    req: &ExecuteRequest,
    max_result_rows: u32,
) -> Result<ExecuteReply>
```

Executes a typed request as one transaction on the opened adapter connection
against the host library.

Merges [`crate::migrations::host_sql_type_env`](migrations.md#host_sql_type_env) so host DML typechecks
against library tables. Isolated plugin bindings must use
[`execute_typed_on_binding`](#execute_typed_on_binding).

#### Errors

Returns [`LibraryError::Orm`](index.md#libraryerror) when a statement fails.

### execute_typed_on_binding

```rust
pub async fn execute_typed_on_binding(
    db: &DatabaseConnection,
    req: &ExecuteRequest,
    _timing_source: &str,
    max_result_rows: u32,
    catalog: &SqlTypeEnv,
) -> Result<ExecuteReply>
```

Typed execute against an isolated plugin binding.

`catalog` is the binding type environment (bootstrap plus already-applied
plugin schema). It must not be the host library catalog.

#### Errors

Returns [`LibraryError::Orm`](index.md#libraryerror) when a statement fails.

### execute_typed_on_session

```rust
pub async fn execute_typed_on_session(
    db: &DatabaseConnection,
    req: &ExecuteRequest,
    max_result_rows: u32,
    session: AtomicSession,
) -> Result<ExecuteReply>
```

[`execute_typed_on`](#execute_typed_on) with session cancel / deadline checks.

Always merges the host library type environment. Binding schema apply
must call [`execute_typed_on_binding`](#execute_typed_on_binding) instead.

#### Errors

Returns [`LibraryError::Orm`](index.md#libraryerror) when a statement fails or the session is interrupted.

### event_inflight_slot

```rust
pub fn event_inflight_slot(plugin_id: &str, resource_class: &str) -> String
```

Slot key that serializes in-flight event deliveries for one plugin class.

### lock_serialization_slot

```rust
pub async fn lock_serialization_slot<C>(
    db: &C,
    in_process: bool,
    slot_key: &str,
) -> Result<()>
where
    C: ConnectionTrait + StreamTrait,
```

Inserts the slot row if missing, then bumps it (transaction-scoped write lock).

Canonical SQL uses `?` placeholders. Host transport never lowers; adapters
realize `INSERT OR IGNORE` at the execute edge. This helper must run on the
caller's transaction, not a nested `BEGIN`. `in_process` of `false` is
canonical leftover SQL (production proxy). `true` physically lowers once
on the opened adapter connection.

#### Errors

Returns [`crate::LibraryError::Orm`](index.md#libraryerror) when either statement fails.

### run_typed_conn_vectors

```rust
pub async fn run_typed_conn_vectors(db: &DatabaseConnection, connect: DbCapabilities)
```

Runs the shared contract suite through the host stamp + adapter execute path.

#### Panics

Panics when a vector fails.

### run_typed_contract_vectors

```rust
pub async fn run_typed_contract_vectors<F, Fut>(
    connect: DbCapabilities,
    row_cap: u32,
    run: F,
)
where
    F: FnMut(ExecuteRequest, u32) -> Fut,
    Fut: Future<Output = Result<ExecuteReply, String>>,
```

Native typed contract suite (`ExecuteRequest` → `ExecuteReply`).

Every reply is validated against its request before the vector's own
assertions run.

### run_typed_request_vectors

```rust
pub async fn run_typed_request_vectors<F, Fut, E>(
    connect: DbCapabilities,
    advertised_cap: u32,
    run: F,
)
where
    F: FnMut(ExecuteRequest) -> Fut,
    Fut: Future<Output = Result<ExecuteReply, E>>,
    E: Display,
```

Runs the shared contract suite through a typed `ExecuteRequest` callback.

`advertised_cap` is the adapter's negotiated `maxResultRows`. The harness does
not wrap SQL or post-reject oversized results.

#### Panics

Panics when a vector fails.

### stamp_typed_vector

```rust
pub fn stamp_typed_vector(
    req: ExecuteRequest,
    catalog: &mut SqlTypeEnv,
) -> Result<AdapterExecuteRequest, String>
```

Desugars and stamps proofs for one vector, recording DDL in `catalog`.

Adapter-boundary tests (guest execute) must stamp against the host catalog
plus tables created by earlier vectors.

#### Errors

Returns when typecheck fails.

### validate_plan

```rust
pub fn validate_plan(compiled: &CompiledAtomic, caps: &DbCapabilities) -> Result<()>
```

Rejects a compiled plan that exceeds negotiated guest limits or has
out-of-range selectors.

#### Errors

Returns [`crate::LibraryError::Other`](index.md#libraryerror) when the plan cannot be sent.

### validate_execute_request

```rust
pub fn validate_execute_request(req: &ExecuteRequest, caps: &DbCapabilities) -> Result<()>
```

Rejects a typed [`ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest) that exceeds negotiated guest limits.

`maxRequestBytes` is measured against the Cap'n-encoded [`ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest).

#### Errors

Returns [`crate::LibraryError::Other`](index.md#libraryerror) when the request cannot be sent.

### validate_proven_execute_request

```rust
pub fn validate_proven_execute_request(
    req: &ExecuteRequest,
    proofs: &[ResolvedStatement],
    caps: &DbCapabilities,
) -> Result<()>
```

After host typecheck, reject a resolved batch whose proven lowered size
exceeds negotiated `maxLoweredStatementBytes`.

Call this with the real [`bookclerk_plugin_abi::ResolvedStatement`](../bookclerk_plugin_abi/index.md#resolvedstatement) proofs
before adapter dispatch. [`validate_execute_request`](#validate_execute_request) only has an empty
catalog, so INTEGER column arithmetic is charged here.

#### Errors

Returns [`crate::LibraryError::Other`](index.md#libraryerror) when proofs are missing/mismatched
or the standardized lowering upper bound exceeds the advertised ceiling.

### authorize_typed_request

```rust
pub fn authorize_typed_request(
    req: &mut ExecuteRequest,
    caps: &DbCapabilities,
) -> Result<()>
```

Host-authorizes a guest typed batch: overwrite statement kinds, enforce
negotiated caps, and stamp the canonical Cap'n request hash.

A non-empty guest `requestHash` must match the host digest (retry token).
An empty `operationId` is replaced with a fresh UUID.

#### Errors

Returns [`crate::LibraryError::Other`](index.md#libraryerror) when the request exceeds caps, the
retry hash does not match, or the canonical digest cannot be encoded.

### execute_guest_atomic_with

```rust
pub async fn execute_guest_atomic_with<F, Fut>(
    req: ExecuteRequest,
    caps: &DbCapabilities,
    policy: &GuestSqlPolicy,
    exec: F,
) -> Result<ExecuteReply, PluginError>
where
    F: FnOnce(AdapterExecuteRequest) -> Fut,
    Fut: Future<Output = Result<ExecuteReply, PluginError>>,
```

Authorizes, receipt-wraps, executes, and unwraps one guest typed batch.

The composable core of
[`LibraryStore::execute_guest_atomic`](index.md#librarystoreexecute_guest_atomic)
for callers that execute on something other than the library connection —
e.g. an isolated plugin database binding session. `exec` receives the
receipt-wrapped envelope and must run it atomically on the target
database (which needs its own `bookclerk_receipts` table — see
[`crate::migrations::binding_bootstrap_ops`](migrations.md#binding_bootstrap_ops)).

#### Errors

Returns `bookclerk_plugin_abi::PluginError::invalid_params` when the SQL
is outside the guest grammar or `policy`, and the executor's error or a
validation error when the batch fails.

### authorize_guest_typed_request

```rust
pub fn authorize_guest_typed_request(
    req: &mut ExecuteRequest,
    caps: &DbCapabilities,
    policy: &GuestSqlPolicy,
) -> Result<()>
```

Authorizes a **guest-authored** typed batch.

Pipeline: plugin SQL → SQL-v1 grammar validation → binding
scope/security authorization → capability/limit validation → canonical
[`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest). Adapter-edge lowering happens
later, at execute, against the isolated physical DB.

Host schema DDL must use [`authorize_typed_request`](#authorize_typed_request) directly.

#### Errors

Returns [`crate::LibraryError::Other`](index.md#libraryerror) when the SQL is outside the guest
grammar or fails host validation.

### wake_page_for_max_binds

```rust
pub fn wake_page_for_max_binds(max_binds: u32) -> u64
```

Wake page size from negotiated `maxBinds` (SET/EXISTS overhead is 4 binds).

### host_statement_kind

```rust
pub fn host_statement_kind(sql: &str) -> DbPlanStatementKind
```

Host-authored statement shape for the wire `kind` field.

Adapters must not reparse SQL; they trust this classification.

### proxy_read_kind

```rust
pub fn proxy_read_kind(sql: &str) -> DbPlanStatementKind
```

Kind for a SeaORM proxy **read**. `SELECT`/`WITH`/`VALUES` become `Select`
(LIMIT-wrapped). `RETURNING` stays `Returning`. Everything else (PRAGMA,
schema introspection) is `Returning` so it is not LIMIT-wrapped.

### proxy_write_kind

```rust
pub fn proxy_write_kind(sql: &str) -> DbPlanStatementKind
```

Kind for a SeaORM proxy **write**. `RETURNING` is preserved; other DML is
`Execute`.

### proxy_insert_returning_identity

```rust
pub fn proxy_insert_returning_identity(sql: &str, catalog: &SqlTypeEnv) -> Option<String>
```

Rewrites a SeaORM proxy single-row `INSERT` to return the table's identity
key.

The typed DB ABI carries no last-insert-id (D1 / Postgres portability), so
a proxy write cannot report SeaORM's `last_insert_id` from `rowsAffected`.
When `sql` is an `INSERT [OR …] INTO t (…) VALUES (…)` of exactly one tuple
and `catalog` gives `t` an `INTEGER PRIMARY KEY AUTOINCREMENT` column, this
returns `sql` with `RETURNING <identity>` appended. The result is proven
single-row ([`bookclerk_plugin_abi::returning_single_row_proven`](../bookclerk_plugin_abi/guest_sql.md#returning_single_row_proven)), so it
may carry `maxRows = 1` on every adapter, including D1.

Returns `None` for every other shape (already `RETURNING`, `INSERT …
SELECT`, multi-tuple `VALUES`, `ON CONFLICT` upserts, tables without an
identity column); those keep the plain execute shape.

## Constants

### JOB_QUEUE_SLOT

```rust
pub const JOB_QUEUE_SLOT: &str = "job-queue";
```

Slot key for job admit / claim / scratch-quota serialization.
