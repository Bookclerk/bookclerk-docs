# bookclerk_plugin_abi::db_execute

Typed Cap'n database data-plane (`ExecuteRequest` / `ExecuteReply`) and
control-plane (`DbCapabilities`) mirrors of `plugin.capnp`.

Hosts call `AdapterDatabaseSession.capabilities` and
`AdapterDatabaseSession.execute`. Granted job SQL uses `GuestDatabase.execute`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`DbBootstrap`](#dbbootstrap) | struct | Diagnostic engine identity returned by `AdapterDatabaseSession.bootstrap`. |
| [`DbColumn`](#dbcolumn) | struct | One column in a typed result set. |
| [`DbRow`](#dbrow) | struct | One positional result row. |
| [`TypedDbStatement`](#typeddbstatement) | struct | One statement in a typed `ExecuteRequest`. |
| [`ExecuteRequest`](#executerequest) | struct | Typed atomic batch. Every request is a non-empty ordered statement list. |
| [`DbIdentityHighWater`](#dbidentityhighwater) | struct | Adapter identity high-water (`sqlite_sequence` / `bookclerk_identity`). |
| [`StatementResult`](#statementresult) | struct | Result of one statement in `ExecuteReply`. |
| [`DbTiming`](#dbtiming) | struct | Engine timing on `ExecuteReply`. |
| [`ExecuteReply`](#executereply) | struct | Typed atomic reply. |
| [`DbCapabilities`](#dbcapabilities) | struct | Semantic SQL-contract advertisement (`AdapterDatabaseSession.capabilities`). |
| [`DbPlanStatementKind`](#dbplanstatementkind) | enum | How a guest should run one statement inside an atomic plan. |
| [`DbResultSelection`](#dbresultselection) | enum | How the guest should return results for one statement. |
| [`IsolationReq`](#isolationreq) | enum | Transaction isolation the adapter must realize. |
| [`lowered_statement_upper_bound_len`](#lowered_statement_upper_bound_len) | function | Standardized Bookclerk sqlite-family lowering upper bound for a canonical statement of `canonical_len` bytes: mechanical LIKE / `NULLIF` / `INSERT OR IGNORE`, the query LIMIT wrap, and `unhex(?)` for every advertised sqlite-family bind slot. |
| [`lowered_statement_preflight_len`](#lowered_statement_preflight_len) | function | Deterministic lowered-statement preflight without a typed proof. |
| [`lowered_statement_preflight_len_proven`](#lowered_statement_preflight_len_proven) | function | `lowered_statement_preflight_len` after applying sqlite-family INTEGER overflow wraps from `proof` (innermost-first, then mechanical `/` `%` NULLIF / LIKE→GLOB / `INSERT OR IGNORE`). |
| [`sql_payload_bytes`](#sql_payload_bytes) | function | UTF-8 bytes of SQL text plus JSON binds (ordinary query/execute payload). |
| [`sql_payload_exceeds`](#sql_payload_exceeds) | function | True when ordinary-path SQL+binds exceed the negotiated payload cap. |
| [`SQLITE_MAX_BINDS`](#sqlite_max_binds) | constant | SQLite family bind cap advertised by the platform sqlite guest. |
| [`POSTGRES_MAX_BINDS`](#postgres_max_binds) | constant | PostgreSQL bind cap advertised by the optional postgres guest. |
| [`D1_MAX_BINDS`](#d1_max_binds) | constant | Cloudflare D1 bound-parameter limit. |
| [`D1_MAX_SQL_STATEMENT_BYTES`](#d1_max_sql_statement_bytes) | constant | Cloudflare D1 physical SQL statement length (bytes) after adapter lowering. |
| [`D1_MAX_FUNCTION_ARGS`](#d1_max_function_args) | constant | Cloudflare D1 maximum arguments to one physical SQL function. |
| [`SQLITE_MAX_FUNCTION_ARGS`](#sqlite_max_function_args) | constant | SQLite default `SQLITE_MAX_FUNCTION_ARG`. |
| [`POSTGRES_MAX_FUNCTION_ARGS`](#postgres_max_function_args) | constant | PostgreSQL maximum function arguments. |
| [`D1_MAX_SCHEMA_COLUMNS`](#d1_max_schema_columns) | constant | Cloudflare D1 maximum columns per table. |
| [`SQLITE_MAX_SCHEMA_COLUMNS`](#sqlite_max_schema_columns) | constant | SQLite default `SQLITE_MAX_COLUMN`. |
| [`POSTGRES_MAX_SCHEMA_COLUMNS`](#postgres_max_schema_columns) | constant | PostgreSQL maximum columns per table. |
| [`SQLITE_MAX_PATTERN_BYTES`](#sqlite_max_pattern_bytes) | constant | SQLite default `SQLITE_MAX_LIKE_PATTERN_LENGTH`. |
| [`HOST_MIN_FUNCTION_ARGS`](#host_min_function_args) | constant | Host refuses guests that cannot accept at least this many function args. |
| [`HOST_MIN_SCHEMA_COLUMNS`](#host_min_schema_columns) | constant | Host refuses guests whose schema-column cap is below the library schema. |
| [`HOST_MIN_PATTERN_BYTES`](#host_min_pattern_bytes) | constant | Host refuses guests that cannot prove a LIKE pattern of this many bytes. |
| [`FIRST_PARTY_MAX_STATEMENTS`](#first_party_max_statements) | constant | D1 / first-party batch statement cap (D1 HTTP batch is 100 queries). |
| [`FIRST_PARTY_MAX_RESULT_ROWS`](#first_party_max_result_rows) | constant | First-party row cap advertised for one query statement. |
| [`HOST_MIN_BINDS`](#host_min_binds) | constant | Host refuses guests that cannot bind at least this many parameters. |
| [`HOST_MIN_STATEMENTS`](#host_min_statements) | constant | Host refuses guests that cannot run at least this many statements per batch. |
| [`HOST_MIN_RESULT_ROWS`](#host_min_result_rows) | constant | Host refuses guests that do not bound result rows (`0` is unspecified). |
| [`HOST_MIN_PAYLOAD_BYTES`](#host_min_payload_bytes) | constant | Host refuses guests that bound encoded statement payload bytes below this, or not at all. |
| [`D1_MAX_PAYLOAD_BYTES`](#d1_max_payload_bytes) | constant | Canonical SQL+binds cap advertised for D1. |
| [`HOST_MIN_RESULT_BYTES`](#host_min_result_bytes) | constant | Host refuses guests that do not bound JSON bytes of one statement's rows. |
| [`HOST_MIN_REQUEST_BYTES`](#host_min_request_bytes) | constant | Host refuses guests that bound one encoded request below this, or not at all. |
| [`HOST_MIN_CELL_BYTES`](#host_min_cell_bytes) | constant | Host refuses guests that do not bound one result cell (`0` is unspecified). |
| [`FIRST_PARTY_MAX_RESULT_BYTES`](#first_party_max_result_bytes) | constant | First-party JSON-byte budget for one statement's rows and for one atomic request/result scalar. Must stay at or below `crate::MAX_SCALAR_BYTES`. |
| [`FIRST_PARTY_MAX_STREAMED_RESULT_BYTES`](#first_party_max_streamed_result_bytes) | constant | First-party budget for one `executeStreamed` reply (all statements' encoded results together): 64 MiB. The adapter holds up to this much before it commits, so a result above the inline scalar ceiling still arrives whole instead of failing closed. |
| [`FIRST_PARTY_MAX_STREAMED_RESULT_ROWS`](#first_party_max_streamed_result_rows) | constant | First-party row budget for one statement answered through `executeStreamed`. |
| [`HOST_MAX_STREAMED_RESULT_BYTES`](#host_max_streamed_result_bytes) | constant | Largest streamed result byte budget the host accepts from an adapter (the first-party budget): the host holds a streamed reply in memory while it reassembles it, so the ceiling is the host's, not the adapter's. |
| [`HOST_MAX_STREAMED_RESULT_ROWS`](#host_max_streamed_result_rows) | constant | Largest streamed per-statement row budget the host accepts from an adapter. |
| [`D1_MAX_STREAMED_RESULT_BYTES`](#d1_max_streamed_result_bytes) | constant | Cloudflare D1 budget for one `executeStreamed` reply: 8 MiB. D1 answers one HTTP request with the whole result, so the adapter stays well under what one REST reply carries. |
| [`D1_MAX_STREAMED_RESULT_ROWS`](#d1_max_streamed_result_rows) | constant | Cloudflare D1 row budget for one statement answered through `executeStreamed`. |
| [`SQL_CONTRACT_VERSION`](#sql_contract_version) | constant | Bookclerk SQL contract version advertised by first-party adapters. |

## Structs

### DbBootstrap

```rust
pub struct DbBootstrap {
    pub engine: String,
}
```

Diagnostic engine identity returned by `AdapterDatabaseSession.bootstrap`.

Not part of the typed [`DbCapabilities`](#dbcapabilities) plane. The host must not admit,
reject, or generate SQL from this value. Any string is valid (including
engines the host has never heard of).

**Fields**

- `engine`: Diagnostic physical engine name (`sqlite`, `postgres`, `duckdb`, …).

#### DbBootstrap::diagnostic

```rust
pub fn diagnostic(engine: impl Into<String>) -> Self
```

Diagnostic bootstrap for an arbitrary physical engine.

#### DbBootstrap::sqlite

```rust
pub fn sqlite() -> Self
```

Bootstrap metadata for a sqlite-family connection (diagnostics only).

#### DbBootstrap::postgres

```rust
pub fn postgres() -> Self
```

Bootstrap metadata for a postgres-family connection (diagnostics only).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DbColumn

```rust
pub struct DbColumn {
    pub name: String,
    pub db_type: DbType,
}
```

One column in a typed result set.

**Fields**

- `name`: Column name.
- `db_type`: Declared / inferred type.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DbRow

```rust
pub struct DbRow {
    pub values: Vec<DbValue>,
}
```

One positional result row.

**Fields**

- `values`: Cells in column order.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### TypedDbStatement

```rust
pub struct TypedDbStatement {
    pub sql: String,
    pub parameters: Vec<DbValue>,
    pub kind: DbPlanStatementKind,
    pub max_rows: u32,
    pub result_selection: DbResultSelection,
}
```

One statement in a typed [`ExecuteRequest`](#executerequest).

**Fields**

- `sql`: Canonical Bookclerk SQL (`?` placeholders).
- `parameters`: Ordered typed binds.
- `kind`: Host-authored kind (adapters must not reparse SQL).
- `max_rows`: Proven row upper bound (`0` = unproven).
- `result_selection`: Which result fields the caller needs.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### ExecuteRequest

```rust
pub struct ExecuteRequest {
    pub operation_id: String,
    pub request_hash: String,
    pub statements: Vec<TypedDbStatement>,
    pub deadline_unix_ms: u64,
}
```

Typed atomic batch. Every request is a non-empty ordered statement list.

**Fields**

- `operation_id`: Caller-chosen idempotency key.
- `request_hash`: SHA-256 hex of the idempotency-relevant request; empty when omitted.
- `statements`: Ordered statements (must be non-empty).
- `deadline_unix_ms`: Guest-visible deadline (unix ms). Zero means omitted.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### DbIdentityHighWater

```rust
pub struct DbIdentityHighWater {
    pub table: String,
    pub last: i64,
}
```

Adapter identity high-water (`sqlite_sequence` / `bookclerk_identity`).

The generated column name lives in the canonical backup schema, not here.

**Fields**

- `table`: Folded table name.
- `last`: Highest generated or stored value that must not be reused.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### StatementResult

```rust
pub struct StatementResult {
    pub rows: Vec<DbRow>,
    pub columns: Vec<DbColumn>,
    pub rows_affected: u64,
}
```

Result of one statement in [`ExecuteReply`](#executereply).

**Fields**

- `rows`: Positional rows (empty when discarded).
- `columns`: Column metadata matching [`Self::rows`](#statementresult) cell order.
- `rows_affected`: Engine-reported rows affected.

#### StatementResult::from_rows

```rust
pub fn from_rows(columns: Vec<DbColumn>, rows: Vec<DbRow>) -> Result<Self, String>
```

Builds a row-bearing result and rejects width / name errors.

##### Errors

Returns when a row length differs from `columns.len()` or a column name
is duplicated.

#### StatementResult::from_affected

```rust
pub fn from_affected(rows_affected: u64) -> Self
```

Builds an affected-rows-only result.

#### StatementResult::validate_positional

```rust
pub fn validate_positional(&self) -> Result<(), String>
```

Rejects duplicate column names and row widths that do not match `columns`.

##### Errors

Returns when a row has the wrong cell count or two columns share a name.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### DbTiming

```rust
pub struct DbTiming {
    pub attempt_elapsed_us: u64,
    pub db_execution_us: u64,
    pub db_timing_source: String,
}
```

Engine timing on [`ExecuteReply`](#executereply).

**Fields**

- `attempt_elapsed_us`: Monotonic duration of this handler attempt.
- `db_execution_us`: Engine-reported SQL/transaction time when available (`0` = omitted).
- `db_timing_source`: How `db_execution_us` was measured.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ExecuteReply

```rust
pub struct ExecuteReply {
    pub operation_id: String,
    pub statements: Vec<StatementResult>,
    pub timing: DbTiming,
}
```

Typed atomic reply.

**Fields**

- `operation_id`: Echo of the request `operationId`.
- `statements`: Per-statement results, in plan order.
- `timing`: Handler/engine timing.

#### ExecuteReply::validate_positional

```rust
pub fn validate_positional(&self) -> Result<(), String>
```

Rejects positional errors on every statement result.

##### Errors

Returns when any statement has a row-width or duplicate-name error.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### DbCapabilities

```rust
pub struct DbCapabilities {
    pub sql_contract_version: u32,
    pub atomic_batch: bool,
    pub returning: bool,
    pub affected_rows: bool,
    pub schema_migrations: bool,
    pub cancellation: bool,
    pub timing: bool,
    pub max_binds: u32,
    pub max_statements: u32,
    pub max_result_rows: u32,
    pub max_payload_bytes: u32,
    pub max_result_bytes: u32,
    pub max_cell_bytes: u32,
    pub max_request_bytes: u32,
    pub max_atomic_result_bytes: u32,
    pub plugin_databases: bool,
    pub max_function_args: u32,
    pub max_schema_columns: u32,
    pub max_pattern_bytes: u32,
    pub max_lowered_statement_bytes: u32,
    pub consistent_backup_read: bool,
    pub atomic_unit_restore: bool,
    pub max_streamed_result_bytes: u32,
    pub max_streamed_result_rows: u32,
}
```

Semantic SQL-contract advertisement (`AdapterDatabaseSession.capabilities`).

Diagnostic engine identity is negotiated separately via [`DbBootstrap`](#dbbootstrap) —
not on this typed capability plane.

**Fields**

- `sql_contract_version`: Bookclerk SQL contract version.
- `atomic_batch`: Guest can run a bounded statement list as one SQL transaction.
- `returning`: Guest SQL supports `RETURNING`.
- `affected_rows`: Guest reports `rowsAffected`.
- `schema_migrations`: Guest versions schema with a `bookclerk_schema_migrations` table.
- `cancellation`: Guest honors RPC/session cancellation.
- `timing`: Guest can fill [`DbTiming::db_execution_us`](#dbtiming). Not a host connect minimum.
- `max_binds`: Maximum bound parameters per statement.
- `max_statements`: Maximum statements in one atomic batch.
- `max_result_rows`: Maximum rows a query statement may return.
- `max_payload_bytes`: Maximum UTF-8 bytes of SQL plus binds per statement.
- `max_result_bytes`: Maximum encoded bytes of one statement's result rows.
- `max_cell_bytes`: Maximum UTF-8 / blob bytes of one result cell.
- `max_request_bytes`: Maximum encoded bytes of one [`ExecuteRequest`](#executerequest).
- `max_atomic_result_bytes`: Maximum encoded bytes of one [`ExecuteReply`](#executereply).
- `plugin_databases`: Adapter can open additional isolated sessions for plugin-owned
  database bindings (per-binding file / schema / database).
- `max_function_args`: Maximum arguments in one physical function call after adapter hiding.
  `0` is unspecified.
- `max_schema_columns`: Maximum columns in one `CREATE TABLE`. `0` is unspecified.
- `max_pattern_bytes`: Maximum UTF-8 bytes of a BookclerkSQL `LIKE` pattern value.
  `0` is unspecified.
- `max_lowered_statement_bytes`: Maximum UTF-8 bytes of sqlite-family lowered SQL for one statement.
  `0` is unspecified (host does not enforce a lowered-size ceiling).
- `consistent_backup_read`: Adapter can expose one stable logical database state while the host
  reads schema, rows, and identity.
- `atomic_unit_restore`: Adapter can destructively replace one logical database unit so an
  ordinary restore failure does not leave that unit partially replaced.
- `max_streamed_result_bytes`: Maximum encoded bytes of one `ExecuteReply` answered through
  `executeStreamed` (every statement result together). `0` means the
  adapter does not stream; otherwise at least `max_atomic_result_bytes`.
- `max_streamed_result_rows`: Maximum rows one statement may return through `executeStreamed`. `0`
  means the adapter does not stream; otherwise at least `max_result_rows`.

#### DbCapabilities::meets_host_minimums

```rust
pub fn meets_host_minimums(&self) -> bool
```

True when this guest meets the host's compiled minimum SQL contract.

#### DbCapabilities::capability_failure_reason

```rust
pub fn capability_failure_reason(&self) -> String
```

Operator-facing reason when [`Self::meets_host_minimums`](#dbcapabilitiesmeets_host_minimums) is false.

#### DbCapabilities::capability_failure_reason_opt

```rust
pub fn capability_failure_reason_opt(&self) -> Option<String>
```

Failure reason, or `None` when the guest meets host minima.

#### DbCapabilities::supports_consistent_backup_read

```rust
pub fn supports_consistent_backup_read(&self) -> bool
```

True when the adapter can participate in a consistent canonical backup
capture (`consistentBackupRead`).

#### DbCapabilities::supports_atomic_unit_restore

```rust
pub fn supports_atomic_unit_restore(&self) -> bool
```

True when the adapter can replace one logical unit completely
(`atomicUnitRestore`).

#### DbCapabilities::streams_results

```rust
pub fn streams_results(&self) -> bool
```

True when the adapter answers `executeStreamed` (a non-zero streamed
budget).

#### DbCapabilities::streamed

```rust
pub fn streamed(&self) -> Option<Self>
```

The caps a streamed reply is validated against: this advertisement
with the result byte caps raised to `max_streamed_result_bytes` and
the row cap to `max_streamed_result_rows`.

##### Returns

`None` when the adapter does not stream.

#### DbCapabilities::advertised_sqlite

```rust
pub fn advertised_sqlite() -> Self
```

First-party SQLite capability advertisement (`bookclerk_schema_migrations` rows).

#### DbCapabilities::advertised_d1

```rust
pub fn advertised_d1() -> Self
```

First-party Cloudflare D1 capability advertisement (`bookclerk_schema_migrations`).

D1 HTTP has no interactive transaction and no complete per-unit
replacement primitive on sequential REST statements, so it does **not**
advertise consistent backup read or atomic unit restore.

Numeric limits are the Cloudflare D1 table plus lowering headroom —
not a copy of the SQLite advertisement. LIKE patterns use
[`D1_PORTABLE_LIKE_PATTERN_BYTES`](README.md#d1_portable_like_pattern_bytes) so GLOB expansion stays inside the
50-byte physical cap.

#### DbCapabilities::advertised_postgres

```rust
pub fn advertised_postgres() -> Self
```

First-party PostgreSQL capability advertisement (`bookclerk_schema_migrations` rows).

#### DbCapabilities::host_minimum

```rust
pub fn host_minimum() -> Self
```

The smallest advertisement the host connects to: every numeric cap at
its `HOST_MIN_*` value, no streamed reads, no plugin databases, no
consistent backup read or atomic unit restore.

Test kits run host and plugin code under this profile so a write that
works on the shipped adapters also works on a plugin that advertises
only the minimums.

#### DbCapabilities::admit_statement

```rust
pub fn admit_statement(&self, sql: &str, parameters: &[DbValue]) -> Result<()>
```

Rejects BookclerkSQL that exceeds this advertisement before dispatch.

Literal INTEGER arithmetic typechecks against an empty catalog so dense
`1+1+…` contributes overflow wrap cost. Statements that need a real
catalog must use [`Self::admit_statement_proven`](#dbcapabilitiesadmit_statement_proven) / [`Self::admit_proven_execute`](#dbcapabilitiesadmit_proven_execute)
after the host has a [`ResolvedStatement`](README.md#resolvedstatement).

Lowered-size admission compares the standardized Bookclerk lowering
upper bound to [`Self::max_lowered_statement_bytes`](#dbcapabilities) (`0` skips that
check). Canonical `maxPayloadBytes` is unchanged.

##### Errors

Returns a typed `limit_exceeded` error (see [`crate::LimitExceeded`](error.md#limitexceeded))
when a statement exceeds the `maxPatternBytes`, `maxFunctionArgs`,
`maxSchemaColumns` or `maxLoweredStatementBytes` cap, and
[`crate::PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when it leaves the portable TEXT
domain or a LIKE pattern length cannot be proven.

#### DbCapabilities::admit_statement_proven

```rust
pub fn admit_statement_proven(
    &self,
    sql: &str,
    parameters: &[DbValue],
    proof: Option<&ResolvedStatement>,
) -> Result<()>
```

[`Self::admit_statement`](#dbcapabilitiesadmit_statement) using a host-resolved proof when present.

##### Errors

Returns a typed `limit_exceeded` error (see [`crate::LimitExceeded`](error.md#limitexceeded))
when a statement exceeds a negotiated cap,
[`crate::PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when it leaves the portable TEXT
domain or a LIKE pattern length cannot be proven, or
[`crate::PluginError::internal`](error.md#pluginerrorinternal) when overflow wrapping fails.

#### DbCapabilities::admit_proven_execute

```rust
pub fn admit_proven_execute(
    &self,
    req: &ExecuteRequest,
    proofs: &[ResolvedStatement],
) -> Result<()>
```

Rejects a resolved batch whose proven lowered size exceeds this advertisement.

Call after the host has 1:1 [`ResolvedStatement`](README.md#resolvedstatement) proofs and before
adapter dispatch. Does not inspect engine identity.

##### Errors

Returns when `proofs` is the wrong length, a proof does not bind, or a
statement fails [`Self::admit_statement_proven`](#dbcapabilitiesadmit_statement_proven).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### DbPlanStatementKind

```rust
pub enum DbPlanStatementKind {
    Execute,
    Select,
    Returning,
}
```

How a guest should run one statement inside an atomic plan.

`Select` versus `Returning` is explicit so adapters never reparse SQL to
decide whether `SELECT * FROM (…)` wrapping is valid. Matches Cap'n
`DbStatementKind` (`execute` | `select` | `returning`).

**Variants**

- `Execute`: Statement is DML; only `rowsAffected` is required.
- `Select`: Read-only `SELECT` / read-only `WITH` CTE. May be wrapped with `LIMIT`.
- `Returning`: DML that returns rows (`INSERT`/`UPDATE`/`DELETE … RETURNING`), or
  row-producing introspection (`PRAGMA`, schema reads) that must **not**
  be rewritten as a subquery.

#### DbPlanStatementKind::collects_rows

```rust
pub const fn collects_rows(self) -> bool
```

True when the guest must collect `rows` (not only `rowsAffected`).

#### DbPlanStatementKind::wrap_select_limit

```rust
pub const fn wrap_select_limit(self) -> bool
```

True when the guest may wrap SQL as `SELECT * FROM (sql) LIMIT cap+1`.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DbResultSelection

```rust
pub enum DbResultSelection {
    Discard,
    AffectedRows,
    Rows,
}
```

How the guest should return results for one statement.

**Variants**

- `Discard`: Drop rows and `rowsAffected`.
- `AffectedRows`: Return `rowsAffected` only.
- `Rows`: Return positional rows plus column metadata.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### IsolationReq

```rust
pub enum IsolationReq {
    AtomicBatch,
    NestedSavepoint,
    ConsistentSnapshot,
}
```

Transaction isolation the adapter must realize.

**Variants**

- `AtomicBatch`: One atomic batch (`BEGIN` / D1 HTTP batch).
- `NestedSavepoint`: Nested savepoint inside an already-open transaction.
- `ConsistentSnapshot`: Consistent snapshot for backup capture.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### lowered_statement_upper_bound_len

```rust
pub const fn lowered_statement_upper_bound_len(canonical_len: usize) -> usize
```

Standardized Bookclerk sqlite-family lowering upper bound for a canonical
statement of `canonical_len` bytes: mechanical LIKE / `NULLIF` /
`INSERT OR IGNORE`, the query LIMIT wrap, and `unhex(?)` for every
advertised sqlite-family bind slot.

This length-only formula does **not** include proof-directed INTEGER
overflow CASE wraps: a worst-case `1+1+…` payload of the 1,024-byte
search floor already exceeds [`D1_MAX_SQL_STATEMENT_BYTES`](#d1_max_sql_statement_bytes) after
wrapping. Overflow is charged by proven preflight from
`integer_arith_sites` (or a typecheck of literal-only SQL in
[`DbCapabilities::admit_statement`](#dbcapabilitiesadmit_statement)). Hosts compare the proven bound to
[`DbCapabilities::max_lowered_statement_bytes`](#dbcapabilities), not engine identity.

### lowered_statement_preflight_len

```rust
pub fn lowered_statement_preflight_len(sql: &str, bind_count: usize) -> Result<usize>
```

Deterministic lowered-statement preflight without a typed proof.

Mechanical sqlite-family lowering plus the query LIMIT wrap and `unhex(?)`
for each bind. INTEGER overflow wraps are omitted unless the caller uses
proven preflight with `integer_arith_sites`.

#### Errors

Returns [`crate::PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when the pack lexer rejects
`sql`.

### lowered_statement_preflight_len_proven

```rust
pub fn lowered_statement_preflight_len_proven(
    sql: &str,
    bind_count: usize,
    proof: Option<&ResolvedStatement>,
) -> Result<usize>
```

[`lowered_statement_preflight_len`](#lowered_statement_preflight_len) after applying sqlite-family INTEGER
overflow wraps from `proof` (innermost-first, then mechanical `/` `%`
NULLIF / LIKE→GLOB / `INSERT OR IGNORE`).

#### Errors

Returns when `proof` does not validate against `sql`, overflow wrapping
fails, or the pack lexer rejects the post-overflow SQL.

### sql_payload_bytes

```rust
pub fn sql_payload_bytes(sql: &str, values_json: &str) -> usize
```

UTF-8 bytes of SQL text plus JSON binds (ordinary query/execute payload).

### sql_payload_exceeds

```rust
pub fn sql_payload_exceeds(sql: &str, values_json: &str, max_payload_bytes: u32) -> bool
```

True when ordinary-path SQL+binds exceed the negotiated payload cap.

The effective cap is `min(max_payload_bytes, MAX_SCALAR_BYTES)`. A cap of
`0` fails closed (any non-empty payload exceeds it).

## Constants

### SQLITE_MAX_BINDS

```rust
pub const SQLITE_MAX_BINDS: u32 = 32_766;
```

SQLite family bind cap advertised by the platform sqlite guest.

### POSTGRES_MAX_BINDS

```rust
pub const POSTGRES_MAX_BINDS: u32 = 65_535;
```

PostgreSQL bind cap advertised by the optional postgres guest.

### D1_MAX_BINDS

```rust
pub const D1_MAX_BINDS: u32 = 100;
```

Cloudflare D1 bound-parameter limit.

<https://developers.cloudflare.com/d1/platform/limits/>

### D1_MAX_SQL_STATEMENT_BYTES

```rust
pub const D1_MAX_SQL_STATEMENT_BYTES: u32 = 100_000;
```

Cloudflare D1 physical SQL statement length (bytes) after adapter lowering.

### D1_MAX_FUNCTION_ARGS

```rust
pub const D1_MAX_FUNCTION_ARGS: u32 = 32;
```

Cloudflare D1 maximum arguments to one physical SQL function.

Portable SQL-v1 `json_object` uses this same ceiling (16 key/value pairs).

### SQLITE_MAX_FUNCTION_ARGS

```rust
pub const SQLITE_MAX_FUNCTION_ARGS: u32 = 127;
```

SQLite default `SQLITE_MAX_FUNCTION_ARG`.

### POSTGRES_MAX_FUNCTION_ARGS

```rust
pub const POSTGRES_MAX_FUNCTION_ARGS: u32 = 100;
```

PostgreSQL maximum function arguments.

### D1_MAX_SCHEMA_COLUMNS

```rust
pub const D1_MAX_SCHEMA_COLUMNS: u32 = 100;
```

Cloudflare D1 maximum columns per table.

### SQLITE_MAX_SCHEMA_COLUMNS

```rust
pub const SQLITE_MAX_SCHEMA_COLUMNS: u32 = 2_000;
```

SQLite default `SQLITE_MAX_COLUMN`.

### POSTGRES_MAX_SCHEMA_COLUMNS

```rust
pub const POSTGRES_MAX_SCHEMA_COLUMNS: u32 = 1_600;
```

PostgreSQL maximum columns per table.

### SQLITE_MAX_PATTERN_BYTES

```rust
pub const SQLITE_MAX_PATTERN_BYTES: u32 = 50_000;
```

SQLite default `SQLITE_MAX_LIKE_PATTERN_LENGTH`.

### HOST_MIN_FUNCTION_ARGS

```rust
pub const HOST_MIN_FUNCTION_ARGS: u32 = _;
```

Host refuses guests that cannot accept at least this many function args.

The host's own plans build result rows with `json_object` calls up to
the SQL-v1 portable maximum
([`crate::SQL_V1_JSON_OBJECT_MAX_ARGS`](sql_types.md#sql_v1_json_object_max_args), D1's physical ceiling), so a
guest below it could not run a job claim or an outbox publish.

### HOST_MIN_SCHEMA_COLUMNS

```rust
pub const HOST_MIN_SCHEMA_COLUMNS: u32 = 64;
```

Host refuses guests whose schema-column cap is below the library schema.

The widest host table (`books`) currently has 41 columns.

### HOST_MIN_PATTERN_BYTES

```rust
pub const HOST_MIN_PATTERN_BYTES: u32 = 8;
```

Host refuses guests that cannot prove a LIKE pattern of this many bytes.

### FIRST_PARTY_MAX_STATEMENTS

```rust
pub const FIRST_PARTY_MAX_STATEMENTS: u32 = 100;
```

D1 / first-party batch statement cap (D1 HTTP batch is 100 queries).

### FIRST_PARTY_MAX_RESULT_ROWS

```rust
pub const FIRST_PARTY_MAX_RESULT_ROWS: u32 = 1_000;
```

First-party row cap advertised for one query statement.

### HOST_MIN_BINDS

```rust
pub const HOST_MIN_BINDS: u32 = 32;
```

Host refuses guests that cannot bind at least this many parameters.

### HOST_MIN_STATEMENTS

```rust
pub const HOST_MIN_STATEMENTS: u32 = 40;
```

Host refuses guests that cannot run at least this many statements per batch.

### HOST_MIN_RESULT_ROWS

```rust
pub const HOST_MIN_RESULT_ROWS: u32 = 1;
```

Host refuses guests that do not bound result rows (`0` is unspecified).

### HOST_MIN_PAYLOAD_BYTES

```rust
pub const HOST_MIN_PAYLOAD_BYTES: u32 = D1_MAX_PAYLOAD_BYTES;
```

Host refuses guests that bound encoded statement payload bytes below
this, or not at all.

The host's own named plans (job claims, outbox publish and dispatch,
acquire status) are single statements sized for D1's proven cap, so a
guest below it could not run them. The minimum is therefore D1's.

### D1_MAX_PAYLOAD_BYTES

```rust
pub const D1_MAX_PAYLOAD_BYTES: u32 = _;
```

Canonical SQL+binds cap advertised for D1.

Largest `n` at or above the 1,024-byte search floor such that
[`lowered_statement_upper_bound_len`](#lowered_statement_upper_bound_len)`(n) <= `[`D1_MAX_SQL_STATEMENT_BYTES`](#d1_max_sql_statement_bytes).
Host admission uses generic [`DbCapabilities::max_payload_bytes`](#dbcapabilities) plus
[`DbCapabilities::max_lowered_statement_bytes`](#dbcapabilities); it does not name D1.

### HOST_MIN_RESULT_BYTES

```rust
pub const HOST_MIN_RESULT_BYTES: u32 = 4_096;
```

Host refuses guests that do not bound JSON bytes of one statement's rows.

### HOST_MIN_REQUEST_BYTES

```rust
pub const HOST_MIN_REQUEST_BYTES: u32 = 65_536;
```

Host refuses guests that bound one encoded request below this, or not at
all.

The host's named plans are one request each: a job claim with its
receipt encodes to about 5 KiB, and a dispatch page of `maxStatements`
delivery inserts to tens of KiB. 64 KiB carries every one of them at the
host-minimum statement and bind caps.

### HOST_MIN_CELL_BYTES

```rust
pub const HOST_MIN_CELL_BYTES: u32 = 1_024;
```

Host refuses guests that do not bound one result cell (`0` is unspecified).

### FIRST_PARTY_MAX_RESULT_BYTES

```rust
pub const FIRST_PARTY_MAX_RESULT_BYTES: u32 = MAX_SCALAR_BYTES;
```

First-party JSON-byte budget for one statement's rows and for one atomic
request/result scalar. Must stay at or below [`crate::MAX_SCALAR_BYTES`](README.md#max_scalar_bytes).

### FIRST_PARTY_MAX_STREAMED_RESULT_BYTES

```rust
pub const FIRST_PARTY_MAX_STREAMED_RESULT_BYTES: u32 = _;
```

First-party budget for one `executeStreamed` reply (all statements'
encoded results together): 64 MiB. The adapter holds up to this much
before it commits, so a result above the inline scalar ceiling still
arrives whole instead of failing closed.

### FIRST_PARTY_MAX_STREAMED_RESULT_ROWS

```rust
pub const FIRST_PARTY_MAX_STREAMED_RESULT_ROWS: u32 = 250_000;
```

First-party row budget for one statement answered through
`executeStreamed`.

### HOST_MAX_STREAMED_RESULT_BYTES

```rust
pub const HOST_MAX_STREAMED_RESULT_BYTES: u32 = FIRST_PARTY_MAX_STREAMED_RESULT_BYTES;
```

Largest streamed result byte budget the host accepts from an adapter
(the first-party budget): the host holds a streamed reply in memory while
it reassembles it, so the ceiling is the host's, not the adapter's.

### HOST_MAX_STREAMED_RESULT_ROWS

```rust
pub const HOST_MAX_STREAMED_RESULT_ROWS: u32 = FIRST_PARTY_MAX_STREAMED_RESULT_ROWS;
```

Largest streamed per-statement row budget the host accepts from an adapter.

### D1_MAX_STREAMED_RESULT_BYTES

```rust
pub const D1_MAX_STREAMED_RESULT_BYTES: u32 = _;
```

Cloudflare D1 budget for one `executeStreamed` reply: 8 MiB. D1 answers
one HTTP request with the whole result, so the adapter stays well under
what one REST reply carries.

### D1_MAX_STREAMED_RESULT_ROWS

```rust
pub const D1_MAX_STREAMED_RESULT_ROWS: u32 = 50_000;
```

Cloudflare D1 row budget for one statement answered through
`executeStreamed`.

### SQL_CONTRACT_VERSION

```rust
pub const SQL_CONTRACT_VERSION: u32 = 1;
```

Bookclerk SQL contract version advertised by first-party adapters.

This integer is a **monotonic backward-compatible language/semantic level**.
Every guarantee in version *N* remains valid in *N+1*; guests advertise the
highest version they implement and hosts require
`sqlContractVersion >= SQL_CONTRACT_VERSION`. That `>=` check is only sound
while each successor is a superset. A non-superset redesign needs a **new
contract identity** and a plugin ABI break (`apiVersion`), not merely
incrementing this scalar.
