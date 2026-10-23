# bookclerk_plugin_abi::sql_types

Bookclerk SQL v1 types, CAST matrix, and fail-closed expression checking.

Schema metadata for plugin-owned bindings is durable in
[`SQL_CATALOG_TABLE`](#sql_catalog_table). This module only interprets canonical SQL; adapters
persist catalog rows at execute.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SqlTypeEnv`](#sqltypeenv) | struct | `(table → ordered columns)` environment used to *build* a proof. |
| [`CreateTableSchema`](#createtableschema) | struct | Parsed `CREATE TABLE` column list (canonical SQL v1). |
| [`ColumnReference`](#columnreference) | struct | Column-level `REFERENCES table [(cols…)]` captured in the structured schema IR. |
| [`CreateIndexSchema`](#createindexschema) | struct | Parsed `CREATE [UNIQUE] INDEX` (canonical SQL v1). |
| [`SqlType`](#sqltype) | enum | Canonical SQL v1 column / expression type. |
| [`TableConstraint`](#tableconstraint) | enum | Table-level constraint captured in the structured schema IR. |
| [`sql_catalog_page_rows`](#sql_catalog_page_rows) | function | Page size for durable catalog / schema SELECTs. |
| [`reserved_catalog_relation_missing`](#reserved_catalog_relation_missing) | function | True when `message` is a missing-relation error for reserved `table`. |
| [`catalog_page_statement`](#catalog_page_statement) | function | Requires a catalog page execute reply to contain exactly one statement. |
| [`sql_v1_ident_in_bounds`](#sql_v1_ident_in_bounds) | function | True when `ident` (already case-folded) is a portable SQL v1 identifier. |
| [`postgres_identity_object_digest`](#postgres_identity_object_digest) | function | SHA-256 digest prefix of a folded table name for adapter-private objects. |
| [`postgres_identity_function_name`](#postgres_identity_function_name) | function | Postgres identity trigger function name for `table` (`bc_if_<digest>`). |
| [`postgres_identity_trigger_name`](#postgres_identity_trigger_name) | function | Postgres identity trigger name for `table` (`bc_it_<digest>`). |
| [`statement_sql_hash`](#statement_sql_hash) | function | SHA-256 hex of the exact canonical SQL string a `ResolvedStatement` proves. |
| [`parse_create_table_schema`](#parse_create_table_schema) | function | Parses a canonical `CREATE TABLE` into columns / identity, if the head matches. |
| [`parse_drop_table_name`](#parse_drop_table_name) | function | Parses `DROP TABLE [IF EXISTS] name`. |
| [`parse_create_index_sql`](#parse_create_index_sql) | function | Parses `CREATE [UNIQUE] INDEX [IF NOT EXISTS] name ON table (cols…) [WHERE …]`. |
| [`parse_drop_index_name`](#parse_drop_index_name) | function | Parses `DROP INDEX [IF EXISTS] name`. |
| [`sql_catalog_create_table_sql`](#sql_catalog_create_table_sql) | function | `CREATE TABLE IF NOT EXISTS` for the reserved SQL v1 catalog. |
| [`sql_schema_create_table_sql`](#sql_schema_create_table_sql) | function | Table-level fingerprint companion table. |
| [`sql_ddl_create_table_sql`](#sql_ddl_create_table_sql) | function | Durable canonical DDL companion table. |
| [`catalog_companions`](#catalog_companions) | function | Catalog DML companions for one canonical DDL statement (all backends). |
| [`catalog_companions_for_action`](#catalog_companions_for_action) | function | Catalog companions gated by a resolved schema action. |
| [`apply_schema_action_to_env`](#apply_schema_action_to_env) | function | Applies a resolved schema action to `env` without reparsing CREATE SQL. |
| [`apply_schema_sql_to_env`](#apply_schema_sql_to_env) | function | Applies one statement's `CREATE`/`DROP TABLE` effects to `env`. |
| [`typecheck_execute_request_proofs`](#typecheck_execute_request_proofs) | function | Host-private proofs for one execute request (not a guest SDK surface). |
| [`sql_type_env_from_canonical_statements`](#sql_type_env_from_canonical_statements) | function | Reconstructs a type environment from already-separated canonical `CREATE TABLE` statements. |
| [`sql_type_env_from_canonical_ddl`](#sql_type_env_from_canonical_ddl) | function | Reconstructs a type environment from one canonical `CREATE TABLE` statement. |
| [`sql_host_bookkeeping_type_env`](#sql_host_bookkeeping_type_env) | function | Host bookkeeping tables present on every binding/library database. |
| [`cast_is_legal`](#cast_is_legal) | function | Proven v1 CAST matrix: same type, INTEGER↔REAL, or NULL to any admitted type. |
| [`sql_v1_helper_arity`](#sql_v1_helper_arity) | function | SQL-v1 helper arity `(min, max)` inclusive. Unknown names return `None`. |
| [`sql_v1_helper_arity_ok`](#sql_v1_helper_arity_ok) | function | True when `n` matches the SQL-v1 signature for `name`. |
| [`require_sql_v1_helper_arity`](#require_sql_v1_helper_arity) | function | Rejects a helper call whose argument count is outside the v1 matrix. |
| [`unify_types`](#unify_types) | function | Unifies two types (`NULL` is the identity; INTEGER+REAL → REAL). |
| [`typecheck_execute_request`](#typecheck_execute_request) | function | Type-checks every statement in `req` against `env` plus in-request `CREATE`. |
| [`typecheck_create_index_sql`](#typecheck_create_index_sql) | function | Typechecks one canonical `CREATE INDEX` (including a partial `WHERE`). |
| [`SQL_CATALOG_TABLE`](#sql_catalog_table) | constant | Reserved catalog of `(table, column, sql_type)` for isolated bindings. |
| [`SQL_IDENTITY_TABLE`](#sql_identity_table) | constant | Reserved transactional identity high-water table (Postgres adapter-private). |
| [`INSERT_SELECT_WRAP_ALIAS`](#insert_select_wrap_alias) | constant | Internal alias used when wrapping `INSERT OR IGNORE … SELECT`. |
| [`SQL_V1_MAX_IDENT_BYTES`](#sql_v1_max_ident_bytes) | constant | Portable bound for every canonical SQL v1 identifier after case fold. |
| [`SQL_SCHEMA_TABLE`](#sql_schema_table) | constant | Table-level durable schema fingerprint (identity + constraints). |
| [`SQL_DDL_TABLE`](#sql_ddl_table) | constant | Admitted canonical `CREATE TABLE` / `CREATE INDEX` SQL for backup restore. |
| [`POSTGRES_IDENT_FN_PREFIX`](#postgres_ident_fn_prefix) | constant | Postgres adapter-private identity function prefix (`bc_if_<digest>`). |
| [`POSTGRES_IDENT_TRIGGER_PREFIX`](#postgres_ident_trigger_prefix) | constant | Postgres adapter-private identity trigger prefix (`bc_it_<digest>`). |
| [`SQL_V1_JSON_OBJECT_MAX_ARGS`](#sql_v1_json_object_max_args) | constant | Portable SQL-v1 `json_object` maximum argument count (16 key/value pairs). |

## Structs

### SqlTypeEnv

```rust
pub struct SqlTypeEnv {
    /* private fields */
}
```

`(table → ordered columns)` environment used to *build* a proof.

#### SqlTypeEnv::new

```rust
pub fn new() -> Self
```

Empty environment.

#### SqlTypeEnv::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no tables are registered.

#### SqlTypeEnv::has_table

```rust
pub fn has_table(&self, table: &str) -> bool
```

True when `table` is in the environment.

#### SqlTypeEnv::column_type

```rust
pub fn column_type(&self, table: &str, column: &str) -> Option<SqlType>
```

Declared type of `table.column`, if present.

#### SqlTypeEnv::table_columns

```rust
pub fn table_columns(&self, table: &str) -> Option<&[(String, SqlType)]>
```

Columns of `table` in declaration order.

#### SqlTypeEnv::fingerprint

```rust
pub fn fingerprint(&self, table: &str) -> Option<&str>
```

Stored structured fingerprint for `table`.

#### SqlTypeEnv::identity_column

```rust
pub fn identity_column(&self, table: &str) -> Option<&str>
```

Identity column for `table`.

#### SqlTypeEnv::insert_table

```rust
pub fn insert_table(
    &mut self,
    table: impl Into<String>,
    columns: impl IntoIterator<Item = (String, SqlType)>,
)
```

Inserts or replaces columns for `table`.

#### SqlTypeEnv::insert_table_schema

```rust
pub fn insert_table_schema(
    &mut self,
    table: impl Into<String>,
    columns: impl IntoIterator<Item = (String, SqlType)>,
    identity: Option<String>,
    fingerprint: String,
)
```

Inserts table columns plus fingerprint / identity metadata.

#### SqlTypeEnv::drop_table

```rust
pub fn drop_table(&mut self, table: &str)
```

Drops `table` from the environment.

#### SqlTypeEnv::rename_table

```rust
pub fn rename_table(&mut self, from: &str, to: &str)
```

Renames a table (used for rebuild `ALTER TABLE … RENAME TO`).

#### SqlTypeEnv::merge

```rust
pub fn merge(&mut self, other: &Self)
```

Merges `other` (later entries win per table).

#### SqlTypeEnv::insert_column

```rust
pub fn insert_column(&mut self, table: &str, column: &str, ty: SqlType)
```

Records one catalog row, appending when the column is new.

#### SqlTypeEnv::iter

```rust
pub fn iter(&self) -> impl Iterator<Item = (&str, &str, SqlType)>
```

Iterates `(table, column, type)` in table order then declaration order.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### CreateTableSchema

```rust
pub struct CreateTableSchema {
    pub table: String,
    pub columns: Vec<(String, SqlType)>,
    pub identity_column: Option<String>,
    pub column_not_null: Vec<bool>,
    pub column_unique: Vec<bool>,
    pub column_primary_key: Vec<bool>,
    pub column_defaults: Vec<String>,
    pub column_checks: Vec<String>,
    pub column_references: Vec<Option<ColumnReference>>,
    pub table_constraints: Vec<TableConstraint>,
}
```

Parsed `CREATE TABLE` column list (canonical SQL v1).

**Fields**

- `table`: Unquoted folded table name.
- `columns`: Column names and types in declaration order.
- `identity_column`: `INTEGER PRIMARY KEY AUTOINCREMENT` column, if any.
- `column_not_null`: Per-column NOT NULL flags (same order as `columns`).
- `column_unique`: Per-column UNIQUE flags.
- `column_primary_key`: Per-column PRIMARY KEY flags (inline).
- `column_defaults`: Per-column DEFAULT SQL (empty if none).
- `column_checks`: Per-column CHECK SQL (empty if none).
- `column_references`: Per-column `REFERENCES` targets (same order as `columns`).
- `table_constraints`: Table-level constraints in declaration order.

#### CreateTableSchema::fingerprint

```rust
pub fn fingerprint(&self) -> String
```

Structured fingerprint (SHA-256 hex) of this schema IR.

#### CreateTableSchema::referenced_tables

```rust
pub fn referenced_tables(&self) -> Vec<String>
```

Foreign-key parent table names from column-level and table-level IR.

Does not scan `CREATE TABLE` text, so `REFERENCES` inside DEFAULT/CHECK
string literals is not a dependency.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ColumnReference

```rust
pub struct ColumnReference {
    pub ref_table: String,
    pub ref_columns: Vec<String>,
}
```

Column-level `REFERENCES table [(cols…)]` captured in the structured schema IR.

**Fields**

- `ref_table`: Referenced table.
- `ref_columns`: Referenced columns (empty if omitted).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CreateIndexSchema

```rust
pub struct CreateIndexSchema {
    pub name: String,
    pub table: String,
    pub unique: bool,
    pub columns: Vec<String>,
    pub where_sql: Option<String>,
    pub canonical_sql: String,
}
```

Parsed `CREATE [UNIQUE] INDEX` (canonical SQL v1).

**Fields**

- `name`: Unquoted folded index name.
- `table`: Table the index is on.
- `unique`: True when the statement is `CREATE UNIQUE INDEX`.
- `columns`: Indexed columns in declaration order (`ident [ASC|DESC]`, no expressions).
- `where_sql`: Optional partial-index `WHERE` predicate (no `WHERE` keyword).
- `canonical_sql`: Canonical statement text (no trailing semicolon).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### SqlType

```rust
pub enum SqlType {
    Integer,
    Real,
    Text,
    Blob,
    Boolean,
    Null,
}
```

Canonical SQL v1 column / expression type.

**Variants**

- `Integer`: `INTEGER` / `DbValue::Int64`.
- `Real`: `REAL` / `DbValue::Float64`.
- `Text`: `TEXT` / `DbValue::Text`.
- `Blob`: `BLOB` / `DbValue::Bytes`.
- `Boolean`: `BOOLEAN` / `DbValue::Boolean`.
- `Null`: Untyped SQL `NULL` (unifies with any type).

#### SqlType::from_column_ident

```rust
pub fn from_column_ident(ty: &str) -> Option<Self>
```

Parses a canonical column type ident (already lowercased).

#### SqlType::as_str

```rust
pub const fn as_str(self) -> &'static str
```

Catalog / docs spelling.

#### SqlType::is_text

```rust
pub const fn is_text(self) -> bool
```

True when this type is TEXT.

#### SqlType::is_numeric

```rust
pub const fn is_numeric(self) -> bool
```

True when this type is INTEGER or REAL.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `From<DbType>`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### TableConstraint

```rust
pub enum TableConstraint {
    PrimaryKey(Vec<String>),
    Unique(Vec<String>),
    Check(String),
    ForeignKey { columns: Vec<String>, ref_table: String, ref_columns: Vec<String> },
}
```

Table-level constraint captured in the structured schema IR.

**Variants**

- `PrimaryKey`: `PRIMARY KEY (cols…)`.
- `Unique`: `UNIQUE (cols…)`.
- `Check`: `CHECK (expr)`.
- `ForeignKey`: `FOREIGN KEY (cols) REFERENCES table (refcols)`.
  - `columns`: Local columns.
  - `ref_table`: Referenced table.
  - `ref_columns`: Referenced columns (empty if omitted).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### sql_catalog_page_rows

```rust
pub fn sql_catalog_page_rows(max_result_rows: u32) -> u32
```

Page size for durable catalog / schema SELECTs.

Uses the negotiated adapter `maxResultRows` when it is nonzero, then caps
at [`FIRST_PARTY_MAX_RESULT_ROWS`](db_execute.md#first_party_max_result_rows).

### reserved_catalog_relation_missing

```rust
pub fn reserved_catalog_relation_missing(message: &str, table: &str) -> bool
```

True when `message` is a missing-relation error for reserved `table`.

Matches SQLite `no such table: bookclerk_sql_catalog` (optional `main.`)
and PostgreSQL `relation "bookclerk_sql_catalog" does not exist`. Does not
match missing-column errors (`column "ordinal" does not exist`).

### catalog_page_statement

```rust
pub fn catalog_page_statement(reply: &ExecuteReply) -> Result<&StatementResult>
```

Requires a catalog page execute reply to contain exactly one statement.

Empty **rows** still mean end-of-catalog. An empty `statements` vec is not
EOF — adapters must fail closed rather than treat a dropped statement as
a finished scan.

#### Errors

Returns when `reply.statements` is not length 1.

### sql_v1_ident_in_bounds

```rust
pub fn sql_v1_ident_in_bounds(ident: &str) -> bool
```

True when `ident` (already case-folded) is a portable SQL v1 identifier.

### postgres_identity_object_digest

```rust
pub fn postgres_identity_object_digest(table: &str) -> String
```

SHA-256 digest prefix of a folded table name for adapter-private objects.

### postgres_identity_function_name

```rust
pub fn postgres_identity_function_name(table: &str) -> String
```

Postgres identity trigger function name for `table` (`bc_if_<digest>`).

### postgres_identity_trigger_name

```rust
pub fn postgres_identity_trigger_name(table: &str) -> String
```

Postgres identity trigger name for `table` (`bc_it_<digest>`).

### statement_sql_hash

```rust
pub fn statement_sql_hash(sql: &str) -> String
```

SHA-256 hex of the exact canonical SQL string a [`ResolvedStatement`](index.md#resolvedstatement) proves.

### parse_create_table_schema

```rust
pub fn parse_create_table_schema(sql: &str) -> Option<CreateTableSchema>
```

Parses a canonical `CREATE TABLE` into columns / identity, if the head matches.

### parse_drop_table_name

```rust
pub fn parse_drop_table_name(sql: &str) -> Option<String>
```

Parses `DROP TABLE [IF EXISTS] name`.

### parse_create_index_sql

```rust
pub fn parse_create_index_sql(sql: &str) -> Option<CreateIndexSchema>
```

Parses `CREATE [UNIQUE] INDEX [IF NOT EXISTS] name ON table (cols…) [WHERE …]`.

The column list and any supported tail must be complete SQL v1. Trailing
tokens (`USING`, `INCLUDE`, `COLLATE`, or garbage after `ON table`) fail.

### parse_drop_index_name

```rust
pub fn parse_drop_index_name(sql: &str) -> Option<String>
```

Parses `DROP INDEX [IF EXISTS] name`.

### sql_catalog_create_table_sql

```rust
pub fn sql_catalog_create_table_sql() -> String
```

`CREATE TABLE IF NOT EXISTS` for the reserved SQL v1 catalog.

### sql_schema_create_table_sql

```rust
pub fn sql_schema_create_table_sql() -> String
```

Table-level fingerprint companion table.

### sql_ddl_create_table_sql

```rust
pub fn sql_ddl_create_table_sql() -> String
```

Durable canonical DDL companion table.

### catalog_companions

```rust
pub fn catalog_companions(sql: &str) -> Vec<String>
```

Catalog DML companions for one canonical DDL statement (all backends).

Callers must skip this when [`SchemaAction::Create`](index.md#schemaaction) is a fingerprint no-op.

### catalog_companions_for_action

```rust
pub fn catalog_companions_for_action(
    sql: &str,
    action: Option<&SchemaAction>,
) -> Vec<String>
```

Catalog companions gated by a resolved schema action.

When `action` carries a [`SchemaAction::Create`](index.md#schemaaction) schema, companions are
built from that proof (no CREATE reparse). Callers without a proof still
parse canonical SQL. [`SchemaAction::None`](index.md#schemaaction) still persists `CREATE INDEX`
/ `DROP INDEX` so backups can rebuild admitted indexes. A fingerprint
matching [`SchemaAction::Create`](index.md#schemaaction) no-op emits no companions: the first
admitted CREATE already persisted complete catalog DDL, and a binding
missing that catalog fails closed.

### apply_schema_action_to_env

```rust
pub fn apply_schema_action_to_env(env: &mut SqlTypeEnv, action: &SchemaAction)
```

Applies a resolved schema action to `env` without reparsing CREATE SQL.

### apply_schema_sql_to_env

```rust
pub fn apply_schema_sql_to_env(env: &mut SqlTypeEnv, sql: &str)
```

Applies one statement's `CREATE`/`DROP TABLE` effects to `env`.

### typecheck_execute_request_proofs

```rust
pub fn typecheck_execute_request_proofs(
    req: &ExecuteRequest,
    env: &SqlTypeEnv,
) -> Result<Vec<ResolvedStatement>>
```

Host-private proofs for one execute request (not a guest SDK surface).

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when an expression is not in the
v1 type contract.

### sql_type_env_from_canonical_statements

```rust
pub fn sql_type_env_from_canonical_statements<I, S>(statements: I) -> SqlTypeEnv
where
    I: IntoIterator<Item = S>,
    S: AsRef<str>,
```

Reconstructs a type environment from already-separated canonical
`CREATE TABLE` statements.

Statement boundaries must come from [`crate::sql_v1_pack_statements`](index.md#sql_v1_pack_statements) or an
explicit list — this function does not split on `;`.

### sql_type_env_from_canonical_ddl

```rust
pub fn sql_type_env_from_canonical_ddl(sql: &str) -> SqlTypeEnv
```

Reconstructs a type environment from one canonical `CREATE TABLE` statement.

Multi-statement packs must use [`sql_type_env_from_canonical_statements`](#sql_type_env_from_canonical_statements).

### sql_host_bookkeeping_type_env

```rust
pub fn sql_host_bookkeeping_type_env() -> SqlTypeEnv
```

Host bookkeeping tables present on every binding/library database.

Receipt wrap SQL is typed against this plus the plugin catalog. Adapters
merge it when rebuilding proofs after Cap'n drops host-private proofs.

### cast_is_legal

```rust
pub fn cast_is_legal(from: SqlType, to: SqlType) -> bool
```

Proven v1 CAST matrix: same type, INTEGER↔REAL, or NULL to any admitted type.

### sql_v1_helper_arity

```rust
pub fn sql_v1_helper_arity(name: &str) -> Option<(usize, usize)>
```

SQL-v1 helper arity `(min, max)` inclusive. Unknown names return `None`.

`json_object` is even arity `2..=`[`SQL_V1_JSON_OBJECT_MAX_ARGS`](#sql_v1_json_object_max_args) (16 pairs).

### sql_v1_helper_arity_ok

```rust
pub fn sql_v1_helper_arity_ok(name: &str, n: usize) -> bool
```

True when `n` matches the SQL-v1 signature for `name`.

### require_sql_v1_helper_arity

```rust
pub fn require_sql_v1_helper_arity(index: usize, name: &str, n: usize) -> Result<()>
```

Rejects a helper call whose argument count is outside the v1 matrix.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when `n` is not a legal arity.

### unify_types

```rust
pub fn unify_types(index: usize, a: SqlType, b: SqlType) -> Result<SqlType>
```

Unifies two types (`NULL` is the identity; INTEGER+REAL → REAL).

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when the types cannot unify.

### typecheck_execute_request

```rust
pub fn typecheck_execute_request(req: &ExecuteRequest, env: &SqlTypeEnv) -> Result<()>
```

Type-checks every statement in `req` against `env` plus in-request `CREATE`.

Host-authoritative SQL is still typed (authorization may be skipped
elsewhere). Unknown columns fail closed, including when the catalog is empty.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when an expression is not in the
v1 type contract.

### typecheck_create_index_sql

```rust
pub fn typecheck_create_index_sql(sql: &str, env: &SqlTypeEnv) -> Result<()>
```

Typechecks one canonical `CREATE INDEX` (including a partial `WHERE`).

#### Errors

Returns when the statement is not admitted SQL v1, the table is missing,
an indexed column is not in [`SqlTypeEnv`](#sqltypeenv), or a `WHERE` predicate is not
a complete boolean SQL-v1 expression.

## Constants

### SQL_CATALOG_TABLE

```rust
pub const SQL_CATALOG_TABLE: &str = "bookclerk_sql_catalog";
```

Reserved catalog of `(table, column, sql_type)` for isolated bindings.

### SQL_IDENTITY_TABLE

```rust
pub const SQL_IDENTITY_TABLE: &str = "bookclerk_identity";
```

Reserved transactional identity high-water table (Postgres adapter-private).

### INSERT_SELECT_WRAP_ALIAS

```rust
pub const INSERT_SELECT_WRAP_ALIAS: &str = "bookclerk_src";
```

Internal alias used when wrapping `INSERT OR IGNORE … SELECT`.

### SQL_V1_MAX_IDENT_BYTES

```rust
pub const SQL_V1_MAX_IDENT_BYTES: usize = 63;
```

Portable bound for every canonical SQL v1 identifier after case fold.

PostgreSQL `NAMEDATALEN` is 64 including the NUL terminator, so 63 bytes
is the portable maximum for tables, columns, indexes, aliases, and CTEs.

### SQL_SCHEMA_TABLE

```rust
pub const SQL_SCHEMA_TABLE: &str = "bookclerk_sql_schema";
```

Table-level durable schema fingerprint (identity + constraints).

### SQL_DDL_TABLE

```rust
pub const SQL_DDL_TABLE: &str = "bookclerk_sql_ddl";
```

Admitted canonical `CREATE TABLE` / `CREATE INDEX` SQL for backup restore.

Fingerprints in [`SQL_SCHEMA_TABLE`](#sql_schema_table) are not invertible. This table stores
the host-admitted canonical DDL so a plugin binding can be reconstructed
without reverse-engineering adapter catalogs.

### POSTGRES_IDENT_FN_PREFIX

```rust
pub const POSTGRES_IDENT_FN_PREFIX: &str = "bc_if_";
```

Postgres adapter-private identity function prefix (`bc_if_<digest>`).

### POSTGRES_IDENT_TRIGGER_PREFIX

```rust
pub const POSTGRES_IDENT_TRIGGER_PREFIX: &str = "bc_it_";
```

Postgres adapter-private identity trigger prefix (`bc_it_<digest>`).

### SQL_V1_JSON_OBJECT_MAX_ARGS

```rust
pub const SQL_V1_JSON_OBJECT_MAX_ARGS: usize = 32;
```

Portable SQL-v1 `json_object` maximum argument count (16 key/value pairs).

Matches Cloudflare D1's physical function-argument limit
([`crate::D1_MAX_FUNCTION_ARGS`](db_execute.md#d1_max_function_args)). Larger objects are not admitted; adapters
must not emulate them with `json_patch` (JSON Merge Patch deletes nulls).
