**bookclerk_library > sql_plan**

# Module: sql_plan

## Contents

**Modules**

- [`host_ir`](#host_ir) - Domain result-selection metadata kept beside a typed [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest).

**Structs**

- [`CompiledAtomic`](#compiledatomic) - Compiled typed request plus host result-selection indexes.

**Functions**

- [`authorize_guest_typed_request`](#authorize_guest_typed_request) - Authorizes a **guest-authored** typed batch.
- [`authorize_typed_request`](#authorize_typed_request) - Host-authorizes a guest typed batch: overwrite statement kinds, enforce
- [`execute_guest_atomic_with`](#execute_guest_atomic_with) - Authorizes, receipt-wraps, executes, and unwraps one guest typed batch.
- [`host_statement_kind`](#host_statement_kind) - Host-authored statement shape for the wire `kind` field.
- [`proxy_insert_returning_identity`](#proxy_insert_returning_identity) - Rewrites a SeaORM proxy single-row `INSERT` to return the table's identity
- [`proxy_read_kind`](#proxy_read_kind) - Kind for a SeaORM proxy **read**. `SELECT`/`WITH`/`VALUES` become `Select`
- [`proxy_write_kind`](#proxy_write_kind) - Kind for a SeaORM proxy **write**. `RETURNING` is preserved; other DML is
- [`validate_execute_request`](#validate_execute_request) - Rejects a typed [`ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest) that exceeds negotiated guest limits.
- [`validate_plan`](#validate_plan) - Rejects a compiled plan that exceeds negotiated guest limits or has
- [`validate_proven_execute_request`](#validate_proven_execute_request) - After host typecheck, reject a resolved batch whose proven lowered size
- [`wake_page_for_max_binds`](#wake_page_for_max_binds) - Wake page size from negotiated `maxBinds` (SET/EXISTS overhead is 4 binds).

---

## bookclerk_library::sql_plan::CompiledAtomic

*Struct*

Compiled typed request plus host result-selection indexes.

**Fields:**
- `request: bookclerk_plugin_abi::ExecuteRequest` - Typed request executed by the guest. Its wire `request_hash` is empty;
- `selection: AtomicSelection` - Host indexes used to interpret the typed reply.
- `expected_hash: String` - Domain receipt hash (SHA-256 hex) bound into the receipt SQL and

**Methods:**

- `fn into_typed_request<impl Into<String>>(self: Self, operation_id: impl Trait) -> ExecuteRequest` - Typed [`ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest) for adapter `execute` (Cap'n data plane).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CompiledAtomic`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::sql_plan::authorize_guest_typed_request

*Function*

Authorizes a **guest-authored** typed batch.

Pipeline: plugin SQL → SQL-v1 grammar validation → binding
scope/security authorization → capability/limit validation → canonical
[`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest). Adapter-edge lowering happens
later, at execute, against the isolated physical DB.

Host schema DDL must use [`authorize_typed_request`](sql_plan.md#bookclerk_librarysql_planauthorize_typed_request) directly.

# Errors

Returns `crate::LibraryError::Other` when the SQL is outside the guest
grammar or fails host validation.

```rust
fn authorize_guest_typed_request(req: & mut bookclerk_plugin_abi::ExecuteRequest, caps: &bookclerk_plugin_abi::DbCapabilities, policy: &bookclerk_plugin_abi::GuestSqlPolicy) -> crate::error::Result<()>
```



## bookclerk_library::sql_plan::authorize_typed_request

*Function*

Host-authorizes a guest typed batch: overwrite statement kinds, enforce
negotiated caps, and stamp the canonical Cap'n request hash.

A non-empty guest `requestHash` must match the host digest (retry token).
An empty `operationId` is replaced with a fresh UUID.

# Errors

Returns `crate::LibraryError::Other` when the request exceeds caps, the
retry hash does not match, or the canonical digest cannot be encoded.

```rust
fn authorize_typed_request(req: & mut bookclerk_plugin_abi::ExecuteRequest, caps: &bookclerk_plugin_abi::DbCapabilities) -> crate::error::Result<()>
```



## bookclerk_library::sql_plan::execute_guest_atomic_with

*Function*

Authorizes, receipt-wraps, executes, and unwraps one guest typed batch.

The composable core of
`LibraryStore::execute_guest_atomic`
for callers that execute on something other than the library connection —
e.g. an isolated plugin database binding session. `exec` receives the
receipt-wrapped envelope and must run it atomically on the target
database (which needs its own `bookclerk_receipts` table — see
[`crate::migrations::binding_bootstrap_ops`](migrations.md#bookclerk_librarymigrationsbinding_bootstrap_ops)).

# Errors

Returns `bookclerk_plugin_abi::PluginError::invalid_params` when the SQL
is outside the guest grammar or `policy`, and the executor's error or a
validation error when the batch fails.

```rust
fn execute_guest_atomic_with<F, Fut>(req: bookclerk_plugin_abi::ExecuteRequest, caps: &bookclerk_plugin_abi::DbCapabilities, policy: &bookclerk_plugin_abi::GuestSqlPolicy, exec: F) -> std::result::Result<bookclerk_plugin_abi::ExecuteReply, bookclerk_plugin_abi::PluginError>
```



## Module: host_ir

Domain result-selection metadata kept beside a typed [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest).

The planner emits [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest) directly. This
module holds only the host-side indexes used to interpret
[`bookclerk_plugin_abi::ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecutereply) into a named [`crate::DbAtomicResult`](atomic_ops.md#bookclerk_libraryatomic_opsdbatomicresult).



## bookclerk_library::sql_plan::host_statement_kind

*Function*

Host-authored statement shape for the wire `kind` field.

Adapters must not reparse SQL; they trust this classification.

```rust
fn host_statement_kind(sql: &str) -> bookclerk_plugin_abi::DbPlanStatementKind
```



## bookclerk_library::sql_plan::proxy_insert_returning_identity

*Function*

Rewrites a SeaORM proxy single-row `INSERT` to return the table's identity
key.

The typed DB ABI carries no last-insert-id (D1 / Postgres portability), so
a proxy write cannot report SeaORM's `last_insert_id` from `rowsAffected`.
When `sql` is an `INSERT [OR …] INTO t (…) VALUES (…)` of exactly one tuple
and `catalog` gives `t` an `INTEGER PRIMARY KEY AUTOINCREMENT` column, this
returns `sql` with `RETURNING <identity>` appended. The result is proven
single-row ([`bookclerk_plugin_abi::returning_single_row_proven`](../bookclerk_plugin_abi/guest_sql.md#bookclerk_plugin_abiguest_sqlreturning_single_row_proven)), so it
may carry `maxRows = 1` on every adapter, including D1.

Returns `None` for every other shape (already `RETURNING`, `INSERT …
SELECT`, multi-tuple `VALUES`, `ON CONFLICT` upserts, tables without an
identity column); those keep the plain execute shape.

```rust
fn proxy_insert_returning_identity(sql: &str, catalog: &bookclerk_plugin_abi::SqlTypeEnv) -> Option<String>
```



## bookclerk_library::sql_plan::proxy_read_kind

*Function*

Kind for a SeaORM proxy **read**. `SELECT`/`WITH`/`VALUES` become `Select`
(LIMIT-wrapped). `RETURNING` stays `Returning`. Everything else (PRAGMA,
schema introspection) is `Returning` so it is not LIMIT-wrapped.

```rust
fn proxy_read_kind(sql: &str) -> bookclerk_plugin_abi::DbPlanStatementKind
```



## bookclerk_library::sql_plan::proxy_write_kind

*Function*

Kind for a SeaORM proxy **write**. `RETURNING` is preserved; other DML is
`Execute`.

```rust
fn proxy_write_kind(sql: &str) -> bookclerk_plugin_abi::DbPlanStatementKind
```



## bookclerk_library::sql_plan::validate_execute_request

*Function*

Rejects a typed [`ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest) that exceeds negotiated guest limits.

`maxRequestBytes` is measured against the Cap'n-encoded [`ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest).

# Errors

Returns `crate::LibraryError::Other` when the request cannot be sent.

```rust
fn validate_execute_request(req: &bookclerk_plugin_abi::ExecuteRequest, caps: &bookclerk_plugin_abi::DbCapabilities) -> crate::error::Result<()>
```



## bookclerk_library::sql_plan::validate_plan

*Function*

Rejects a compiled plan that exceeds negotiated guest limits or has
out-of-range selectors.

# Errors

Returns `crate::LibraryError::Other` when the plan cannot be sent.

```rust
fn validate_plan(compiled: &CompiledAtomic, caps: &bookclerk_plugin_abi::DbCapabilities) -> crate::error::Result<()>
```



## bookclerk_library::sql_plan::validate_proven_execute_request

*Function*

After host typecheck, reject a resolved batch whose proven lowered size
exceeds negotiated `maxLoweredStatementBytes`.

Call this with the real [`bookclerk_plugin_abi::ResolvedStatement`](../bookclerk_plugin_abi/sql_proof.md#bookclerk_plugin_abisql_proofresolvedstatement) proofs
before adapter dispatch. [`validate_execute_request`](sql_plan.md#bookclerk_librarysql_planvalidate_execute_request) only has an empty
catalog, so INTEGER column arithmetic is charged here.

# Errors

Returns `crate::LibraryError::Other` when proofs are missing/mismatched
or the standardized lowering upper bound exceeds the advertised ceiling.

```rust
fn validate_proven_execute_request(req: &bookclerk_plugin_abi::ExecuteRequest, proofs: &[bookclerk_plugin_abi::ResolvedStatement], caps: &bookclerk_plugin_abi::DbCapabilities) -> crate::error::Result<()>
```



## bookclerk_library::sql_plan::wake_page_for_max_binds

*Function*

Wake page size from negotiated `maxBinds` (SET/EXISTS overhead is 4 binds).

```rust
fn wake_page_for_max_binds(max_binds: u32) -> u64
```



