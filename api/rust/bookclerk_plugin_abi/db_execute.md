**bookclerk_plugin_abi > db_execute**

# Module: db_execute

## Contents

**Structs**

- [`DbBootstrap`](#dbbootstrap) - Diagnostic engine identity returned by `AdapterDatabaseSession.bootstrap`.
- [`DbCapabilities`](#dbcapabilities) - Semantic SQL-contract advertisement (`AdapterDatabaseSession.capabilities`).
- [`DbColumn`](#dbcolumn) - One column in a typed result set.
- [`DbIdentityHighWater`](#dbidentityhighwater) - Adapter identity high-water (`sqlite_sequence` / `bookclerk_identity`).
- [`DbRow`](#dbrow) - One positional result row.
- [`DbTiming`](#dbtiming) - Engine timing on [`ExecuteReply`](db_execute.md#bookclerk_plugin_abidb_executeexecutereply).
- [`ExecuteReply`](#executereply) - Typed atomic reply.
- [`ExecuteRequest`](#executerequest) - Typed atomic batch. Every request is a non-empty ordered statement list.
- [`StatementResult`](#statementresult) - Result of one statement in [`ExecuteReply`](db_execute.md#bookclerk_plugin_abidb_executeexecutereply).
- [`TypedDbStatement`](#typeddbstatement) - One statement in a typed [`ExecuteRequest`](db_execute.md#bookclerk_plugin_abidb_executeexecuterequest).

**Enums**

- [`DbPlanStatementKind`](#dbplanstatementkind) - How a guest should run one statement inside an atomic plan.
- [`DbResultSelection`](#dbresultselection) - How the guest should return results for one statement.
- [`IsolationReq`](#isolationreq) - Transaction isolation the adapter must realize.

**Functions**

- [`lowered_statement_preflight_len`](#lowered_statement_preflight_len) - Deterministic lowered-statement preflight without a typed proof.
- [`lowered_statement_preflight_len_proven`](#lowered_statement_preflight_len_proven) - [`lowered_statement_preflight_len`](db_execute.md#bookclerk_plugin_abidb_executelowered_statement_preflight_len) after applying sqlite-family INTEGER
- [`lowered_statement_upper_bound_len`](#lowered_statement_upper_bound_len) - Standardized Bookclerk sqlite-family lowering upper bound for a canonical
- [`sql_payload_bytes`](#sql_payload_bytes) - UTF-8 bytes of SQL text plus JSON binds (ordinary query/execute payload).
- [`sql_payload_exceeds`](#sql_payload_exceeds) - True when ordinary-path SQL+binds exceed the negotiated payload cap.

**Constants**

- [`D1_MAX_BINDS`](#d1_max_binds) - Cloudflare D1 bound-parameter limit.
- [`D1_MAX_FUNCTION_ARGS`](#d1_max_function_args) - Cloudflare D1 maximum arguments to one physical SQL function.
- [`D1_MAX_PAYLOAD_BYTES`](#d1_max_payload_bytes) - Canonical SQL+binds cap advertised for D1.
- [`D1_MAX_SCHEMA_COLUMNS`](#d1_max_schema_columns) - Cloudflare D1 maximum columns per table.
- [`D1_MAX_SQL_STATEMENT_BYTES`](#d1_max_sql_statement_bytes) - Cloudflare D1 physical SQL statement length (bytes) after adapter lowering.
- [`FIRST_PARTY_MAX_RESULT_BYTES`](#first_party_max_result_bytes) - First-party JSON-byte budget for one statement's rows and for one atomic
- [`FIRST_PARTY_MAX_RESULT_ROWS`](#first_party_max_result_rows) - First-party row cap advertised for one query statement.
- [`FIRST_PARTY_MAX_STATEMENTS`](#first_party_max_statements) - D1 / first-party batch statement cap (D1 HTTP batch is 100 queries).
- [`HOST_MIN_BINDS`](#host_min_binds) - Host refuses guests that cannot bind at least this many parameters.
- [`HOST_MIN_CELL_BYTES`](#host_min_cell_bytes) - Host refuses guests that do not bound one result cell (`0` is unspecified).
- [`HOST_MIN_FUNCTION_ARGS`](#host_min_function_args) - Host refuses guests that cannot accept at least this many function args.
- [`HOST_MIN_PATTERN_BYTES`](#host_min_pattern_bytes) - Host refuses guests that cannot prove a LIKE pattern of this many bytes.
- [`HOST_MIN_PAYLOAD_BYTES`](#host_min_payload_bytes) - Host refuses guests that do not bound encoded statement payload bytes.
- [`HOST_MIN_RESULT_BYTES`](#host_min_result_bytes) - Host refuses guests that do not bound JSON bytes of one statement's rows.
- [`HOST_MIN_RESULT_ROWS`](#host_min_result_rows) - Host refuses guests that do not bound result rows (`0` is unspecified).
- [`HOST_MIN_SCHEMA_COLUMNS`](#host_min_schema_columns) - Host refuses guests whose schema-column cap is below the library schema.
- [`HOST_MIN_STATEMENTS`](#host_min_statements) - Host refuses guests that cannot run at least this many statements per batch.
- [`POSTGRES_MAX_BINDS`](#postgres_max_binds) - PostgreSQL bind cap advertised by the optional postgres guest.
- [`POSTGRES_MAX_FUNCTION_ARGS`](#postgres_max_function_args) - PostgreSQL maximum function arguments.
- [`POSTGRES_MAX_SCHEMA_COLUMNS`](#postgres_max_schema_columns) - PostgreSQL maximum columns per table.
- [`SQLITE_MAX_BINDS`](#sqlite_max_binds) - SQLite family bind cap advertised by the platform sqlite guest.
- [`SQLITE_MAX_FUNCTION_ARGS`](#sqlite_max_function_args) - SQLite default `SQLITE_MAX_FUNCTION_ARG`.
- [`SQLITE_MAX_PATTERN_BYTES`](#sqlite_max_pattern_bytes) - SQLite default `SQLITE_MAX_LIKE_PATTERN_LENGTH`.
- [`SQLITE_MAX_SCHEMA_COLUMNS`](#sqlite_max_schema_columns) - SQLite default `SQLITE_MAX_COLUMN`.
- [`SQL_CONTRACT_VERSION`](#sql_contract_version) - Bookclerk SQL contract version advertised by first-party adapters.

---

## bookclerk_plugin_abi::db_execute::D1_MAX_BINDS

*Constant*: `u32`

Cloudflare D1 bound-parameter limit.

<https://developers.cloudflare.com/d1/platform/limits/>



## bookclerk_plugin_abi::db_execute::D1_MAX_FUNCTION_ARGS

*Constant*: `u32`

Cloudflare D1 maximum arguments to one physical SQL function.

Portable SQL-v1 `json_object` uses this same ceiling (16 key/value pairs).



## bookclerk_plugin_abi::db_execute::D1_MAX_PAYLOAD_BYTES

*Constant*: `u32`

Canonical SQL+binds cap advertised for D1.

Largest `n >= `[`HOST_MIN_PAYLOAD_BYTES`](db_execute.md#bookclerk_plugin_abidb_executehost_min_payload_bytes) such that
[`lowered_statement_upper_bound_len`](db_execute.md#bookclerk_plugin_abidb_executelowered_statement_upper_bound_len)`(n) <= `[`D1_MAX_SQL_STATEMENT_BYTES`](db_execute.md#bookclerk_plugin_abidb_executed1_max_sql_statement_bytes).
Host admission uses generic `DbCapabilities::max_payload_bytes` plus
`DbCapabilities::max_lowered_statement_bytes`; it does not name D1.



## bookclerk_plugin_abi::db_execute::D1_MAX_SCHEMA_COLUMNS

*Constant*: `u32`

Cloudflare D1 maximum columns per table.



## bookclerk_plugin_abi::db_execute::D1_MAX_SQL_STATEMENT_BYTES

*Constant*: `u32`

Cloudflare D1 physical SQL statement length (bytes) after adapter lowering.



## bookclerk_plugin_abi::db_execute::DbBootstrap

*Struct*

Diagnostic engine identity returned by `AdapterDatabaseSession.bootstrap`.

Not part of the typed [`DbCapabilities`](db_execute.md#bookclerk_plugin_abidb_executedbcapabilities) plane. The host must not admit,
reject, or generate SQL from this value. Any string is valid (including
engines the host has never heard of).

**Fields:**
- `engine: String` - Diagnostic physical engine name (`sqlite`, `postgres`, `duckdb`, …).

**Methods:**

- `fn diagnostic<impl Into<String>>(engine: impl Trait) -> Self` - Diagnostic bootstrap for an arbitrary physical engine.
- `fn sqlite() -> Self` - Bootstrap metadata for a sqlite-family connection (diagnostics only).
- `fn postgres() -> Self` - Bootstrap metadata for a postgres-family connection (diagnostics only).

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbBootstrap) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DbBootstrap`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> DbBootstrap`



## bookclerk_plugin_abi::db_execute::DbCapabilities

*Struct*

Semantic SQL-contract advertisement (`AdapterDatabaseSession.capabilities`).

Diagnostic engine identity is negotiated separately via [`DbBootstrap`](db_execute.md#bookclerk_plugin_abidb_executedbbootstrap) —
not on this typed capability plane.

**Fields:**
- `sql_contract_version: u32` - Bookclerk SQL contract version.
- `atomic_batch: bool` - Guest can run a bounded statement list as one SQL transaction.
- `returning: bool` - Guest SQL supports `RETURNING`.
- `affected_rows: bool` - Guest reports `rowsAffected`.
- `schema_migrations: bool` - Guest versions schema with a `bookclerk_schema_migrations` table.
- `cancellation: bool` - Guest honors RPC/session cancellation.
- `timing: bool` - Guest can fill `DbTiming::db_execution_us`. Not a host connect minimum.
- `max_binds: u32` - Maximum bound parameters per statement.
- `max_statements: u32` - Maximum statements in one atomic batch.
- `max_result_rows: u32` - Maximum rows a query statement may return.
- `max_payload_bytes: u32` - Maximum UTF-8 bytes of SQL plus binds per statement.
- `max_result_bytes: u32` - Maximum encoded bytes of one statement's result rows.
- `max_cell_bytes: u32` - Maximum UTF-8 / blob bytes of one result cell.
- `max_request_bytes: u32` - Maximum encoded bytes of one [`ExecuteRequest`](db_execute.md#bookclerk_plugin_abidb_executeexecuterequest).
- `max_atomic_result_bytes: u32` - Maximum encoded bytes of one [`ExecuteReply`](db_execute.md#bookclerk_plugin_abidb_executeexecutereply).
- `plugin_databases: bool` - Adapter can open additional isolated sessions for plugin-owned
- `max_function_args: u32` - Maximum arguments in one physical function call after adapter hiding.
- `max_schema_columns: u32` - Maximum columns in one `CREATE TABLE`. `0` is unspecified.
- `max_pattern_bytes: u32` - Maximum UTF-8 bytes of a BookclerkSQL `LIKE` pattern value.
- `max_lowered_statement_bytes: u32` - Maximum UTF-8 bytes of sqlite-family lowered SQL for one statement.
- `consistent_backup_read: bool` - Adapter can expose one stable logical database state while the host
- `atomic_unit_restore: bool` - Adapter can destructively replace one logical database unit so an

**Methods:**

- `fn meets_host_minimums(self: &Self) -> bool` - True when this guest meets the host's compiled minimum SQL contract.
- `fn capability_failure_reason(self: &Self) -> String` - Operator-facing reason when `Self::meets_host_minimums` is false.
- `fn supports_consistent_backup_read(self: &Self) -> bool` - True when the adapter can participate in a consistent canonical backup
- `fn supports_atomic_unit_restore(self: &Self) -> bool` - True when the adapter can replace one logical unit completely
- `fn advertised_sqlite() -> Self` - First-party SQLite capability advertisement (`bookclerk_schema_migrations` rows).
- `fn advertised_d1() -> Self` - First-party Cloudflare D1 capability advertisement (`bookclerk_schema_migrations`).
- `fn advertised_postgres() -> Self` - First-party PostgreSQL capability advertisement (`bookclerk_schema_migrations` rows).
- `fn admit_statement(self: &Self, sql: &str, parameters: &[crate::DbValue]) -> crate::Result<()>` - Rejects BookclerkSQL that exceeds this advertisement before dispatch.
- `fn admit_statement_proven(self: &Self, sql: &str, parameters: &[crate::DbValue], proof: Option<&ResolvedStatement>) -> crate::Result<()>` - `Self::admit_statement` using a host-resolved proof when present.
- `fn admit_proven_execute(self: &Self, req: &ExecuteRequest, proofs: &[ResolvedStatement]) -> crate::Result<()>` - Rejects a resolved batch whose proven lowered size exceeds this advertisement.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbCapabilities) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DbCapabilities`



## bookclerk_plugin_abi::db_execute::DbColumn

*Struct*

One column in a typed result set.

**Fields:**
- `name: String` - Column name.
- `db_type: crate::db_value::DbType` - Declared / inferred type.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DbColumn`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbColumn) -> bool`



## bookclerk_plugin_abi::db_execute::DbIdentityHighWater

*Struct*

Adapter identity high-water (`sqlite_sequence` / `bookclerk_identity`).

The generated column name lives in the canonical backup schema, not here.

**Fields:**
- `table: String` - Folded table name.
- `last: i64` - Highest generated or stored value that must not be reused.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &DbIdentityHighWater) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DbIdentityHighWater`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::db_execute::DbPlanStatementKind

*Enum*

How a guest should run one statement inside an atomic plan.

`Select` versus `Returning` is explicit so adapters never reparse SQL to
decide whether `SELECT * FROM (…)` wrapping is valid. Matches Cap'n
`DbStatementKind` (`execute` | `select` | `returning`).

**Variants:**
- `Execute` - Statement is DML; only `rowsAffected` is required.
- `Select` - Read-only `SELECT` / read-only `WITH` CTE. May be wrapped with `LIMIT`.
- `Returning` - DML that returns rows (`INSERT`/`UPDATE`/`DELETE … RETURNING`), or

**Methods:**

- `fn collects_rows(self: Self) -> bool` - True when the guest must collect `rows` (not only `rowsAffected`).
- `fn wrap_select_limit(self: Self) -> bool` - True when the guest may wrap SQL as `SELECT * FROM (sql) LIMIT cap+1`.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DbPlanStatementKind`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> DbPlanStatementKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbPlanStatementKind) -> bool`



## bookclerk_plugin_abi::db_execute::DbResultSelection

*Enum*

How the guest should return results for one statement.

**Variants:**
- `Discard` - Drop rows and `rowsAffected`.
- `AffectedRows` - Return `rowsAffected` only.
- `Rows` - Return positional rows plus column metadata.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> DbResultSelection`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbResultSelection) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DbResultSelection`



## bookclerk_plugin_abi::db_execute::DbRow

*Struct*

One positional result row.

**Fields:**
- `values: Vec<crate::db_value::DbValue>` - Cells in column order.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbRow) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DbRow`



## bookclerk_plugin_abi::db_execute::DbTiming

*Struct*

Engine timing on [`ExecuteReply`](db_execute.md#bookclerk_plugin_abidb_executeexecutereply).

**Fields:**
- `attempt_elapsed_us: u64` - Monotonic duration of this handler attempt.
- `db_execution_us: u64` - Engine-reported SQL/transaction time when available (`0` = omitted).
- `db_timing_source: String` - How `db_execution_us` was measured.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> DbTiming`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbTiming) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DbTiming`



## bookclerk_plugin_abi::db_execute::ExecuteReply

*Struct*

Typed atomic reply.

**Fields:**
- `operation_id: String` - Echo of the request `operationId`.
- `statements: Vec<StatementResult>` - Per-statement results, in plan order.
- `timing: DbTiming` - Handler/engine timing.

**Methods:**

- `fn validate_positional(self: &Self) -> Result<(), String>` - Rejects positional errors on every statement result.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ExecuteReply) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ExecuteReply`



## bookclerk_plugin_abi::db_execute::ExecuteRequest

*Struct*

Typed atomic batch. Every request is a non-empty ordered statement list.

**Fields:**
- `operation_id: String` - Caller-chosen idempotency key.
- `request_hash: String` - SHA-256 hex of the idempotency-relevant request; empty when omitted.
- `statements: Vec<TypedDbStatement>` - Ordered statements (must be non-empty).
- `deadline_unix_ms: u64` - Guest-visible deadline (unix ms). Zero means omitted.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ExecuteRequest) -> bool`
- **Default**
  - `fn default() -> ExecuteRequest`
- **Clone**
  - `fn clone(self: &Self) -> ExecuteRequest`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::db_execute::FIRST_PARTY_MAX_RESULT_BYTES

*Constant*: `u32`

First-party JSON-byte budget for one statement's rows and for one atomic
request/result scalar. Must stay at or below [`crate::MAX_SCALAR_BYTES`](limits.md#bookclerk_plugin_abilimitsmax_scalar_bytes).



## bookclerk_plugin_abi::db_execute::FIRST_PARTY_MAX_RESULT_ROWS

*Constant*: `u32`

First-party row cap advertised for one query statement.



## bookclerk_plugin_abi::db_execute::FIRST_PARTY_MAX_STATEMENTS

*Constant*: `u32`

D1 / first-party batch statement cap (D1 HTTP batch is 100 queries).



## bookclerk_plugin_abi::db_execute::HOST_MIN_BINDS

*Constant*: `u32`

Host refuses guests that cannot bind at least this many parameters.



## bookclerk_plugin_abi::db_execute::HOST_MIN_CELL_BYTES

*Constant*: `u32`

Host refuses guests that do not bound one result cell (`0` is unspecified).



## bookclerk_plugin_abi::db_execute::HOST_MIN_FUNCTION_ARGS

*Constant*: `u32`

Host refuses guests that cannot accept at least this many function args.



## bookclerk_plugin_abi::db_execute::HOST_MIN_PATTERN_BYTES

*Constant*: `u32`

Host refuses guests that cannot prove a LIKE pattern of this many bytes.



## bookclerk_plugin_abi::db_execute::HOST_MIN_PAYLOAD_BYTES

*Constant*: `u32`

Host refuses guests that do not bound encoded statement payload bytes.



## bookclerk_plugin_abi::db_execute::HOST_MIN_RESULT_BYTES

*Constant*: `u32`

Host refuses guests that do not bound JSON bytes of one statement's rows.



## bookclerk_plugin_abi::db_execute::HOST_MIN_RESULT_ROWS

*Constant*: `u32`

Host refuses guests that do not bound result rows (`0` is unspecified).



## bookclerk_plugin_abi::db_execute::HOST_MIN_SCHEMA_COLUMNS

*Constant*: `u32`

Host refuses guests whose schema-column cap is below the library schema.

The widest host table (`books`) currently has 41 columns.



## bookclerk_plugin_abi::db_execute::HOST_MIN_STATEMENTS

*Constant*: `u32`

Host refuses guests that cannot run at least this many statements per batch.



## bookclerk_plugin_abi::db_execute::IsolationReq

*Enum*

Transaction isolation the adapter must realize.

**Variants:**
- `AtomicBatch` - One atomic batch (`BEGIN` / D1 HTTP batch).
- `NestedSavepoint` - Nested savepoint inside an already-open transaction.
- `ConsistentSnapshot` - Consistent snapshot for backup capture.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> IsolationReq`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &IsolationReq) -> bool`
- **Default**
  - `fn default() -> IsolationReq`



## bookclerk_plugin_abi::db_execute::POSTGRES_MAX_BINDS

*Constant*: `u32`

PostgreSQL bind cap advertised by the optional postgres guest.



## bookclerk_plugin_abi::db_execute::POSTGRES_MAX_FUNCTION_ARGS

*Constant*: `u32`

PostgreSQL maximum function arguments.



## bookclerk_plugin_abi::db_execute::POSTGRES_MAX_SCHEMA_COLUMNS

*Constant*: `u32`

PostgreSQL maximum columns per table.



## bookclerk_plugin_abi::db_execute::SQLITE_MAX_BINDS

*Constant*: `u32`

SQLite family bind cap advertised by the platform sqlite guest.



## bookclerk_plugin_abi::db_execute::SQLITE_MAX_FUNCTION_ARGS

*Constant*: `u32`

SQLite default `SQLITE_MAX_FUNCTION_ARG`.



## bookclerk_plugin_abi::db_execute::SQLITE_MAX_PATTERN_BYTES

*Constant*: `u32`

SQLite default `SQLITE_MAX_LIKE_PATTERN_LENGTH`.



## bookclerk_plugin_abi::db_execute::SQLITE_MAX_SCHEMA_COLUMNS

*Constant*: `u32`

SQLite default `SQLITE_MAX_COLUMN`.



## bookclerk_plugin_abi::db_execute::SQL_CONTRACT_VERSION

*Constant*: `u32`

Bookclerk SQL contract version advertised by first-party adapters.

This integer is a **monotonic backward-compatible language/semantic level**.
Every guarantee in version *N* remains valid in *N+1*; guests advertise the
highest version they implement and hosts require
`sqlContractVersion >= SQL_CONTRACT_VERSION`. That `>=` check is only sound
while each successor is a superset. A non-superset redesign needs a **new
contract identity** and a plugin ABI break (`apiVersion`), not merely
incrementing this scalar.



## bookclerk_plugin_abi::db_execute::StatementResult

*Struct*

Result of one statement in [`ExecuteReply`](db_execute.md#bookclerk_plugin_abidb_executeexecutereply).

**Fields:**
- `rows: Vec<DbRow>` - Positional rows (empty when discarded).
- `columns: Vec<DbColumn>` - Column metadata matching `Self::rows` cell order.
- `rows_affected: u64` - Engine-reported rows affected.

**Methods:**

- `fn from_rows(columns: Vec<DbColumn>, rows: Vec<DbRow>) -> Result<Self, String>` - Builds a row-bearing result and rejects width / name errors.
- `fn from_affected(rows_affected: u64) -> Self` - Builds an affected-rows-only result.
- `fn validate_positional(self: &Self) -> Result<(), String>` - Rejects duplicate column names and row widths that do not match `columns`.

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &StatementResult) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> StatementResult`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> StatementResult`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::db_execute::TypedDbStatement

*Struct*

One statement in a typed [`ExecuteRequest`](db_execute.md#bookclerk_plugin_abidb_executeexecuterequest).

**Fields:**
- `sql: String` - Canonical Bookclerk SQL (`?` placeholders).
- `parameters: Vec<crate::db_value::DbValue>` - Ordered typed binds.
- `kind: DbPlanStatementKind` - Host-authored kind (adapters must not reparse SQL).
- `max_rows: u32` - Proven row upper bound (`0` = unproven).
- `result_selection: DbResultSelection` - Which result fields the caller needs.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> TypedDbStatement`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &TypedDbStatement) -> bool`



## bookclerk_plugin_abi::db_execute::lowered_statement_preflight_len

*Function*

Deterministic lowered-statement preflight without a typed proof.

Mechanical sqlite-family lowering plus the query LIMIT wrap and `unhex(?)`
for each bind. INTEGER overflow wraps are omitted unless the caller uses
proven preflight with `integer_arith_sites`.

# Errors

Returns `crate::PluginError::invalid_params` when the pack lexer rejects
`sql`.

```rust
fn lowered_statement_preflight_len(sql: &str, bind_count: usize) -> crate::Result<usize>
```



## bookclerk_plugin_abi::db_execute::lowered_statement_preflight_len_proven

*Function*

[`lowered_statement_preflight_len`](db_execute.md#bookclerk_plugin_abidb_executelowered_statement_preflight_len) after applying sqlite-family INTEGER
overflow wraps from `proof` (innermost-first, then mechanical `/` `%`
NULLIF / LIKE→GLOB / `INSERT OR IGNORE`).

# Errors

Returns when `proof` does not validate against `sql`, overflow wrapping
fails, or the pack lexer rejects the post-overflow SQL.

```rust
fn lowered_statement_preflight_len_proven(sql: &str, bind_count: usize, proof: Option<&crate::ResolvedStatement>) -> crate::Result<usize>
```



## bookclerk_plugin_abi::db_execute::lowered_statement_upper_bound_len

*Function*

Standardized Bookclerk sqlite-family lowering upper bound for a canonical
statement of `canonical_len` bytes: mechanical LIKE / `NULLIF` /
`INSERT OR IGNORE`, the query LIMIT wrap, and `unhex(?)` for every
advertised sqlite-family bind slot.

This length-only formula does **not** include proof-directed INTEGER
overflow CASE wraps: a worst-case `1+1+…` payload of
[`HOST_MIN_PAYLOAD_BYTES`](db_execute.md#bookclerk_plugin_abidb_executehost_min_payload_bytes) already exceeds [`D1_MAX_SQL_STATEMENT_BYTES`](db_execute.md#bookclerk_plugin_abidb_executed1_max_sql_statement_bytes)
after wrapping. Overflow is charged by proven preflight from
`integer_arith_sites` (or a typecheck of literal-only SQL in
`DbCapabilities::admit_statement`). Hosts compare the proven bound to
`DbCapabilities::max_lowered_statement_bytes`, not engine identity.

```rust
fn lowered_statement_upper_bound_len(canonical_len: usize) -> usize
```



## bookclerk_plugin_abi::db_execute::sql_payload_bytes

*Function*

UTF-8 bytes of SQL text plus JSON binds (ordinary query/execute payload).

```rust
fn sql_payload_bytes(sql: &str, values_json: &str) -> usize
```



## bookclerk_plugin_abi::db_execute::sql_payload_exceeds

*Function*

True when ordinary-path SQL+binds exceed the negotiated payload cap.

The effective cap is `min(max_payload_bytes, MAX_SCALAR_BYTES)`. A cap of
`0` fails closed (any non-empty payload exceeds it).

```rust
fn sql_payload_exceeds(sql: &str, values_json: &str, max_payload_bytes: u32) -> bool
```



