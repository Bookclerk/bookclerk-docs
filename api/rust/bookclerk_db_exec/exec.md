**bookclerk_db_exec > exec**

# Module: exec

## Contents

**Structs**

- [`AtomicSession`](#atomicsession) - Session-level cancel / deadline for one atomic attempt (not hashed).
- [`ExecCaps`](#execcaps) - Per-statement result bounds. Row/byte `0` means unlimited at execute time.

**Enums**

- [`PhysicalEngine`](#physicalengine) - Real engine the adapter opened.
- [`SqlExecTarget`](#sqlexectarget) - How leftover host SQL reaches a SeaORM connection.

**Functions**

- [`cap_query_sql`](#cap_query_sql) - Wraps a host-tagged `bookclerk_plugin_abi::DbPlanStatementKind::Select` so the engine stops
- [`encoded_proxy_row_len`](#encoded_proxy_row_len) - Encoded JSON length of a SeaORM proxy row (keys, numbers, punctuation).
- [`execute_physical_sql`](#execute_physical_sql) - Executes canonical SQL on a physical engine the adapter opened.
- [`json_cell_utf8_len`](#json_cell_utf8_len) - UTF-8 / JSON length of one cell used for `maxCellBytes`.
- [`note_encoded_result_bytes`](#note_encoded_result_bytes) - Adds `extra` encoded JSON bytes toward `max_result_bytes`.
- [`query_physical_sql`](#query_physical_sql) - Queries canonical SQL on a physical engine the adapter opened.
- [`query_physical_sql_typed`](#query_physical_sql_typed) - [`query_physical_sql`](exec.md#bookclerk_db_execexecquery_physical_sql) with an optional hash-bound proof (TEXT collate /
- [`sea_value_to_json`](#sea_value_to_json) - Maps a SeaORM cell onto JSON for plan interpretation.

---

## bookclerk_db_exec::exec::AtomicSession

*Struct*

Session-level cancel / deadline for one atomic attempt (not hashed).

**Fields:**
- `cancel: Option<std::sync::Arc<std::sync::atomic::AtomicBool>>` - Host RPC cancel flag, when the guest shares a process with the host.
- `deadline_unix_ms: Option<u64>` - Guest-visible deadline (`deadlineUnixMs` on the wire).
- `type_env: bookclerk_plugin_abi::SqlTypeEnv` - Extra catalog snapshot merged before typing host-authored SQL

**Methods:**

- `fn from_deadline(deadline_unix_ms: Option<u64>) -> Self` - Builds session control from a wire `deadlineUnixMs`.
- `fn with_cancel(self: Self, cancel: Option<Arc<AtomicBool>>) -> Self` - Attaches a host cancel flag (job fence) to this session.
- `fn with_type_env(self: Self, env: SqlTypeEnv) -> Self` - Merges `env` (typically the canonical host schema) into this session so

**Trait Implementations:**

- **Default**
  - `fn default() -> AtomicSession`
- **Clone**
  - `fn clone(self: &Self) -> AtomicSession`



## bookclerk_db_exec::exec::ExecCaps

*Struct*

Per-statement result bounds. Row/byte `0` means unlimited at execute time.

**Fields:**
- `max_result_rows: u32` - Maximum rows a statement may return (`0` = unlimited).
- `max_result_bytes: u32` - Maximum JSON bytes of one statement's rows (`0` = unlimited).
- `max_cell_bytes: u32` - Maximum UTF-8 / blob bytes of one cell (`0` = unlimited).
- `max_atomic_result_bytes: u32` - Maximum encoded bytes of the whole atomic reply (`0` = unlimited).

**Methods:**

- `fn from_capabilities(caps: &DbCapabilities) -> Self` - Copies negotiated capability limits into the executor.

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ExecCaps`
- **From**
  - `fn from(max_result_rows: u32) -> Self`
- **Default**
  - `fn default() -> ExecCaps`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_db_exec::exec::PhysicalEngine

*Enum*

Real engine the adapter opened.

Hosts must not construct this from a SeaORM proxy connection. Physical
lowering ([`crate::lower_canonical_sql`](lower.md#bookclerk_db_execlowerlower_canonical_sql)) happens only against this identity.
Unknown SeaORM backends fail closed — never default to SQLite.

**Variants:**
- `Sqlite` - SQLite physical engine (rusqlite / files).
- `Postgres` - PostgreSQL 16+ physical engine.
- `D1` - Cloudflare D1 (sqlite-family SQL, HTTP batch).

**Methods:**

- `fn sqlite() -> Self` - SQLite physical engine.
- `fn postgres() -> Self` - PostgreSQL physical engine.
- `fn d1() -> Self` - Cloudflare D1 physical engine.
- `fn from_adapter_backend(backend: sea_orm::DatabaseBackend) -> Self` - Adapter-only: the backend of a connection this adapter opened.
- `fn backend(self: Self) -> sea_orm::DatabaseBackend` - SeaORM backend used for physical SQL and typed binds.
- `fn timing_source(self: Self) -> &'static str` - Timing source label for this physical engine.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PhysicalEngine`
- **PartialEq**
  - `fn eq(self: &Self, other: &PhysicalEngine) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_db_exec::exec::SqlExecTarget

*Enum*

How leftover host SQL reaches a SeaORM connection.

`Self::Canonical` is sqlite-shaped transport with no physical lowering
(production plugin-host proxy). `Self::InProcess` lowers exactly once
using the engine of the opened adapter connection (native sqlite / postgres
tests). Host/library code must not name [`PhysicalEngine`](exec.md#bookclerk_db_execexecphysicalengine).

**Variants:**
- `Canonical` - Canonical `?` SQL. Adapters lower after RPC.
- `InProcess` - In-process adapter connection. Infer the engine from the connection

**Methods:**

- `fn lowers(self: Self) -> bool` - True when leftover SQL is physically lowered by the adapter SDK.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &SqlExecTarget) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> SqlExecTarget`
- **Default**
  - `fn default() -> SqlExecTarget`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_db_exec::exec::cap_query_sql

*Function*

Wraps a host-tagged `bookclerk_plugin_abi::DbPlanStatementKind::Select` so the engine stops
after `cap + 1` rows. Callers must not pass DML `RETURNING` SQL.

```rust
fn cap_query_sql(sql: &str, max_result_rows: u32) -> String
```



## bookclerk_db_exec::exec::encoded_proxy_row_len

*Function*

Encoded JSON length of a SeaORM proxy row (keys, numbers, punctuation).

```rust
fn encoded_proxy_row_len<'a, impl IntoIterator<Item = (&'a String, &'a Value)>>(values: impl Trait) -> usize
```



## bookclerk_db_exec::exec::execute_physical_sql

*Function*

Executes canonical SQL on a physical engine the adapter opened.

Does **not** run host semantic desugars. Callers must pass already-canonical
SQL (`?` placeholders). Host/library code must not call this helper.

# Errors

Returns when the engine rejects the lowered statement.

```rust
fn execute_physical_sql<C, impl IntoIterator<Item = Value>>(engine: PhysicalEngine, db: &C, sql: &str, values: impl Trait) -> Result<sea_orm::ExecResult, sea_orm::DbErr>
```



## bookclerk_db_exec::exec::json_cell_utf8_len

*Function*

UTF-8 / JSON length of one cell used for `maxCellBytes`.

```rust
fn json_cell_utf8_len(v: &serde_json::Value) -> usize
```



## bookclerk_db_exec::exec::note_encoded_result_bytes

*Function*

Adds `extra` encoded JSON bytes toward `max_result_bytes`.

# Errors

Returns `DbErr::Custom` when the running total would exceed the cap.

```rust
fn note_encoded_result_bytes(used: & mut usize, extra: usize, max_result_bytes: u32) -> Result<(), sea_orm::DbErr>
```



## bookclerk_db_exec::exec::query_physical_sql

*Function*

Queries canonical SQL on a physical engine the adapter opened.

# Errors

Returns when the engine rejects the lowered statement.

```rust
fn query_physical_sql<C, impl IntoIterator<Item = Value>>(engine: PhysicalEngine, db: &C, sql: &str, values: impl Trait) -> Result<Vec<sea_orm::QueryResult>, sea_orm::DbErr>
```



## bookclerk_db_exec::exec::query_physical_sql_typed

*Function*

[`query_physical_sql`](exec.md#bookclerk_db_execexecquery_physical_sql) with an optional hash-bound proof (TEXT collate /
INTEGER overflow on Postgres).

# Errors

Returns when the proof is not bound to `sql` or the engine rejects the work.

```rust
fn query_physical_sql_typed<C, impl IntoIterator<Item = Value>>(engine: PhysicalEngine, db: &C, sql: &str, proof: Option<&bookclerk_plugin_abi::ResolvedStatement>, values: impl Trait) -> Result<Vec<sea_orm::QueryResult>, sea_orm::DbErr>
```



## bookclerk_db_exec::exec::sea_value_to_json

*Function*

Maps a SeaORM cell onto JSON for plan interpretation.

```rust
fn sea_value_to_json(v: &sea_orm::Value) -> serde_json::Value
```



