**bookclerk_plugin_abi > sql_types**

# Module: sql_types

## Contents

**Structs**

- [`ColumnReference`](#columnreference) - Column-level `REFERENCES table [(cols…)]` captured in the structured schema IR.
- [`CreateIndexSchema`](#createindexschema) - Parsed `CREATE [UNIQUE] INDEX` (canonical SQL v1).
- [`CreateTableSchema`](#createtableschema) - Parsed `CREATE TABLE` column list (canonical SQL v1).
- [`SqlTypeEnv`](#sqltypeenv) - `(table → ordered columns)` environment used to *build* a proof.

**Enums**

- [`SqlType`](#sqltype) - Canonical SQL v1 column / expression type.
- [`TableConstraint`](#tableconstraint) - Table-level constraint captured in the structured schema IR.

**Functions**

- [`apply_schema_action_to_env`](#apply_schema_action_to_env) - Applies a resolved schema action to `env` without reparsing CREATE SQL.
- [`apply_schema_sql_to_env`](#apply_schema_sql_to_env) - Applies one statement's `CREATE`/`DROP TABLE` effects to `env`.
- [`cast_is_legal`](#cast_is_legal) - Proven v1 CAST matrix: same type, INTEGER↔REAL, or NULL to any admitted type.
- [`catalog_companions`](#catalog_companions) - Catalog DML companions for one canonical DDL statement (all backends).
- [`catalog_companions_for_action`](#catalog_companions_for_action) - Catalog companions gated by a resolved schema action.
- [`catalog_page_statement`](#catalog_page_statement) - Requires a catalog page execute reply to contain exactly one statement.
- [`parse_create_index_sql`](#parse_create_index_sql) - Parses `CREATE [UNIQUE] INDEX [IF NOT EXISTS] name ON table (cols…) [WHERE …]`.
- [`parse_create_table_schema`](#parse_create_table_schema) - Parses a canonical `CREATE TABLE` into columns / identity, if the head matches.
- [`parse_drop_index_name`](#parse_drop_index_name) - Parses `DROP INDEX [IF EXISTS] name`.
- [`parse_drop_table_name`](#parse_drop_table_name) - Parses `DROP TABLE [IF EXISTS] name`.
- [`postgres_identity_function_name`](#postgres_identity_function_name) - Postgres identity trigger function name for `table` (`bc_if_<digest>`).
- [`postgres_identity_object_digest`](#postgres_identity_object_digest) - SHA-256 digest prefix of a folded table name for adapter-private objects.
- [`postgres_identity_trigger_name`](#postgres_identity_trigger_name) - Postgres identity trigger name for `table` (`bc_it_<digest>`).
- [`require_sql_v1_helper_arity`](#require_sql_v1_helper_arity) - Rejects a helper call whose argument count is outside the v1 matrix.
- [`reserved_catalog_relation_missing`](#reserved_catalog_relation_missing) - True when `message` is a missing-relation error for reserved `table`.
- [`sql_catalog_create_table_sql`](#sql_catalog_create_table_sql) - `CREATE TABLE IF NOT EXISTS` for the reserved SQL v1 catalog.
- [`sql_catalog_page_rows`](#sql_catalog_page_rows) - Page size for durable catalog / schema SELECTs.
- [`sql_ddl_create_table_sql`](#sql_ddl_create_table_sql) - Durable canonical DDL companion table.
- [`sql_host_bookkeeping_type_env`](#sql_host_bookkeeping_type_env) - Host bookkeeping tables present on every binding/library database.
- [`sql_schema_create_table_sql`](#sql_schema_create_table_sql) - Table-level fingerprint companion table.
- [`sql_type_env_from_canonical_ddl`](#sql_type_env_from_canonical_ddl) - Reconstructs a type environment from one canonical `CREATE TABLE` statement.
- [`sql_type_env_from_canonical_statements`](#sql_type_env_from_canonical_statements) - Reconstructs a type environment from already-separated canonical
- [`sql_v1_helper_arity`](#sql_v1_helper_arity) - SQL-v1 helper arity `(min, max)` inclusive. Unknown names return `None`.
- [`sql_v1_helper_arity_ok`](#sql_v1_helper_arity_ok) - True when `n` matches the SQL-v1 signature for `name`.
- [`sql_v1_ident_in_bounds`](#sql_v1_ident_in_bounds) - True when `ident` (already case-folded) is a portable SQL v1 identifier.
- [`statement_sql_hash`](#statement_sql_hash) - SHA-256 hex of the exact canonical SQL string a [`ResolvedStatement`](sql_proof.md#bookclerk_plugin_abisql_proofresolvedstatement) proves.
- [`typecheck_create_index_sql`](#typecheck_create_index_sql) - Typechecks one canonical `CREATE INDEX` (including a partial `WHERE`).
- [`typecheck_execute_request`](#typecheck_execute_request) - Type-checks every statement in `req` against `env` plus in-request `CREATE`.
- [`typecheck_execute_request_proofs`](#typecheck_execute_request_proofs) - Host-private proofs for one execute request (not a guest SDK surface).
- [`unify_types`](#unify_types) - Unifies two types (`NULL` is the identity; INTEGER+REAL → REAL).

**Constants**

- [`INSERT_SELECT_WRAP_ALIAS`](#insert_select_wrap_alias) - Internal alias used when wrapping `INSERT OR IGNORE … SELECT`.
- [`POSTGRES_IDENT_FN_PREFIX`](#postgres_ident_fn_prefix) - Postgres adapter-private identity function prefix (`bc_if_<digest>`).
- [`POSTGRES_IDENT_TRIGGER_PREFIX`](#postgres_ident_trigger_prefix) - Postgres adapter-private identity trigger prefix (`bc_it_<digest>`).
- [`SQL_CATALOG_TABLE`](#sql_catalog_table) - Reserved catalog of `(table, column, sql_type)` for isolated bindings.
- [`SQL_DDL_TABLE`](#sql_ddl_table) - Admitted canonical `CREATE TABLE` / `CREATE INDEX` SQL for backup restore.
- [`SQL_IDENTITY_TABLE`](#sql_identity_table) - Reserved transactional identity high-water table (Postgres adapter-private).
- [`SQL_SCHEMA_TABLE`](#sql_schema_table) - Table-level durable schema fingerprint (identity + constraints).
- [`SQL_V1_JSON_OBJECT_MAX_ARGS`](#sql_v1_json_object_max_args) - Portable SQL-v1 `json_object` maximum argument count (16 key/value pairs).
- [`SQL_V1_MAX_IDENT_BYTES`](#sql_v1_max_ident_bytes) - Portable bound for every canonical SQL v1 identifier after case fold.

---

## bookclerk_plugin_abi::sql_types::ColumnReference

*Struct*

Column-level `REFERENCES table [(cols…)]` captured in the structured schema IR.

**Fields:**
- `ref_table: String` - Referenced table.
- `ref_columns: Vec<String>` - Referenced columns (empty if omitted).

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ColumnReference`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ColumnReference) -> bool`



## bookclerk_plugin_abi::sql_types::CreateIndexSchema

*Struct*

Parsed `CREATE [UNIQUE] INDEX` (canonical SQL v1).

**Fields:**
- `name: String` - Unquoted folded index name.
- `table: String` - Table the index is on.
- `unique: bool` - True when the statement is `CREATE UNIQUE INDEX`.
- `columns: Vec<String>` - Indexed columns in declaration order (`ident [ASC|DESC]`, no expressions).
- `where_sql: Option<String>` - Optional partial-index `WHERE` predicate (no `WHERE` keyword).
- `canonical_sql: String` - Canonical statement text (no trailing semicolon).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CreateIndexSchema`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CreateIndexSchema) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::sql_types::CreateTableSchema

*Struct*

Parsed `CREATE TABLE` column list (canonical SQL v1).

**Fields:**
- `table: String` - Unquoted folded table name.
- `columns: Vec<(String, SqlType)>` - Column names and types in declaration order.
- `identity_column: Option<String>` - `INTEGER PRIMARY KEY AUTOINCREMENT` column, if any.
- `column_not_null: Vec<bool>` - Per-column NOT NULL flags (same order as `columns`).
- `column_unique: Vec<bool>` - Per-column UNIQUE flags.
- `column_primary_key: Vec<bool>` - Per-column PRIMARY KEY flags (inline).
- `column_defaults: Vec<String>` - Per-column DEFAULT SQL (empty if none).
- `column_checks: Vec<String>` - Per-column CHECK SQL (empty if none).
- `column_references: Vec<Option<ColumnReference>>` - Per-column `REFERENCES` targets (same order as `columns`).
- `table_constraints: Vec<TableConstraint>` - Table-level constraints in declaration order.

**Methods:**

- `fn fingerprint(self: &Self) -> String` - Structured fingerprint (SHA-256 hex) of this schema IR.
- `fn referenced_tables(self: &Self) -> Vec<String>` - Foreign-key parent table names from column-level and table-level IR.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CreateTableSchema`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CreateTableSchema) -> bool`



## bookclerk_plugin_abi::sql_types::INSERT_SELECT_WRAP_ALIAS

*Constant*: `&str`

Internal alias used when wrapping `INSERT OR IGNORE … SELECT`.



## bookclerk_plugin_abi::sql_types::POSTGRES_IDENT_FN_PREFIX

*Constant*: `&str`

Postgres adapter-private identity function prefix (`bc_if_<digest>`).



## bookclerk_plugin_abi::sql_types::POSTGRES_IDENT_TRIGGER_PREFIX

*Constant*: `&str`

Postgres adapter-private identity trigger prefix (`bc_it_<digest>`).



## bookclerk_plugin_abi::sql_types::SQL_CATALOG_TABLE

*Constant*: `&str`

Reserved catalog of `(table, column, sql_type)` for isolated bindings.



## bookclerk_plugin_abi::sql_types::SQL_DDL_TABLE

*Constant*: `&str`

Admitted canonical `CREATE TABLE` / `CREATE INDEX` SQL for backup restore.

Fingerprints in [`SQL_SCHEMA_TABLE`](sql_types.md#bookclerk_plugin_abisql_typessql_schema_table) are not invertible. This table stores
the host-admitted canonical DDL so a plugin binding can be reconstructed
without reverse-engineering adapter catalogs.



## bookclerk_plugin_abi::sql_types::SQL_IDENTITY_TABLE

*Constant*: `&str`

Reserved transactional identity high-water table (Postgres adapter-private).



## bookclerk_plugin_abi::sql_types::SQL_SCHEMA_TABLE

*Constant*: `&str`

Table-level durable schema fingerprint (identity + constraints).



## bookclerk_plugin_abi::sql_types::SQL_V1_JSON_OBJECT_MAX_ARGS

*Constant*: `usize`

Portable SQL-v1 `json_object` maximum argument count (16 key/value pairs).

Matches Cloudflare D1's physical function-argument limit
([`crate::D1_MAX_FUNCTION_ARGS`](db_execute.md#bookclerk_plugin_abidb_executed1_max_function_args)). Larger objects are not admitted; adapters
must not emulate them with `json_patch` (JSON Merge Patch deletes nulls).



## bookclerk_plugin_abi::sql_types::SQL_V1_MAX_IDENT_BYTES

*Constant*: `usize`

Portable bound for every canonical SQL v1 identifier after case fold.

PostgreSQL `NAMEDATALEN` is 64 including the NUL terminator, so 63 bytes
is the portable maximum for tables, columns, indexes, aliases, and CTEs.



## bookclerk_plugin_abi::sql_types::SqlType

*Enum*

Canonical SQL v1 column / expression type.

**Variants:**
- `Integer` - `INTEGER` / `DbValue::Int64`.
- `Real` - `REAL` / `DbValue::Float64`.
- `Text` - `TEXT` / `DbValue::Text`.
- `Blob` - `BLOB` / `DbValue::Bytes`.
- `Boolean` - `BOOLEAN` / `DbValue::Boolean`.
- `Null` - Untyped SQL `NULL` (unifies with any type).

**Methods:**

- `fn from_column_ident(ty: &str) -> Option<Self>` - Parses a canonical column type ident (already lowercased).
- `fn as_str(self: Self) -> &'static str` - Catalog / docs spelling.
- `fn is_text(self: Self) -> bool` - True when this type is TEXT.
- `fn is_numeric(self: Self) -> bool` - True when this type is INTEGER or REAL.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SqlType) -> bool`
- **Ord**
  - `fn cmp(self: &Self, other: &SqlType) -> $crate::cmp::Ordering`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &SqlType) -> $crate::option::Option<$crate::cmp::Ordering>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **From**
  - `fn from(ty: DbType) -> Self`
- **Clone**
  - `fn clone(self: &Self) -> SqlType`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::sql_types::SqlTypeEnv

*Struct*

`(table → ordered columns)` environment used to *build* a proof.

**Methods:**

- `fn new() -> Self` - Empty environment.
- `fn is_empty(self: &Self) -> bool` - True when no tables are registered.
- `fn has_table(self: &Self, table: &str) -> bool` - True when `table` is in the environment.
- `fn column_type(self: &Self, table: &str, column: &str) -> Option<SqlType>` - Declared type of `table.column`, if present.
- `fn table_columns(self: &Self, table: &str) -> Option<&[(String, SqlType)]>` - Columns of `table` in declaration order.
- `fn fingerprint(self: &Self, table: &str) -> Option<&str>` - Stored structured fingerprint for `table`.
- `fn identity_column(self: &Self, table: &str) -> Option<&str>` - Identity column for `table`.
- `fn insert_table<impl Into<String>, impl IntoIterator<Item = (String, SqlType)>>(self: & mut Self, table: impl Trait, columns: impl Trait)` - Inserts or replaces columns for `table`.
- `fn insert_table_schema<impl Into<String>, impl IntoIterator<Item = (String, SqlType)>>(self: & mut Self, table: impl Trait, columns: impl Trait, identity: Option<String>, fingerprint: String)` - Inserts table columns plus fingerprint / identity metadata.
- `fn drop_table(self: & mut Self, table: &str)` - Drops `table` from the environment.
- `fn rename_table(self: & mut Self, from: &str, to: &str)` - Renames a table (used for rebuild `ALTER TABLE … RENAME TO`).
- `fn merge(self: & mut Self, other: &Self)` - Merges `other` (later entries win per table).
- `fn insert_column(self: & mut Self, table: &str, column: &str, ty: SqlType)` - Records one catalog row, appending when the column is new.
- `fn iter(self: &Self) -> impl Trait` - Iterates `(table, column, type)` in table order then declaration order.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> SqlTypeEnv`
- **Clone**
  - `fn clone(self: &Self) -> SqlTypeEnv`
- **PartialEq**
  - `fn eq(self: &Self, other: &SqlTypeEnv) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::sql_types::TableConstraint

*Enum*

Table-level constraint captured in the structured schema IR.

**Variants:**
- `PrimaryKey(Vec<String>)` - `PRIMARY KEY (cols…)`.
- `Unique(Vec<String>)` - `UNIQUE (cols…)`.
- `Check(String)` - `CHECK (expr)`.
- `ForeignKey{ columns: Vec<String>, ref_table: String, ref_columns: Vec<String> }` - `FOREIGN KEY (cols) REFERENCES table (refcols)`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &TableConstraint) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TableConstraint`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::sql_types::apply_schema_action_to_env

*Function*

Applies a resolved schema action to `env` without reparsing CREATE SQL.

```rust
fn apply_schema_action_to_env(env: & mut SqlTypeEnv, action: &crate::sql_proof::SchemaAction)
```



## bookclerk_plugin_abi::sql_types::apply_schema_sql_to_env

*Function*

Applies one statement's `CREATE`/`DROP TABLE` effects to `env`.

```rust
fn apply_schema_sql_to_env(env: & mut SqlTypeEnv, sql: &str)
```



## bookclerk_plugin_abi::sql_types::cast_is_legal

*Function*

Proven v1 CAST matrix: same type, INTEGER↔REAL, or NULL to any admitted type.

```rust
fn cast_is_legal(from: SqlType, to: SqlType) -> bool
```



## bookclerk_plugin_abi::sql_types::catalog_companions

*Function*

Catalog DML companions for one canonical DDL statement (all backends).

Callers must skip this when `SchemaAction::Create` is a fingerprint no-op.

```rust
fn catalog_companions(sql: &str) -> Vec<String>
```



## bookclerk_plugin_abi::sql_types::catalog_companions_for_action

*Function*

Catalog companions gated by a resolved schema action.

When `action` carries a `SchemaAction::Create` schema, companions are
built from that proof (no CREATE reparse). Callers without a proof still
parse canonical SQL. `SchemaAction::None` still persists `CREATE INDEX`
/ `DROP INDEX` so backups can rebuild admitted indexes. A fingerprint
matching `SchemaAction::Create` no-op emits no companions: the first
admitted CREATE already persisted complete catalog DDL, and a binding
missing that catalog fails closed.

```rust
fn catalog_companions_for_action(sql: &str, action: Option<&crate::sql_proof::SchemaAction>) -> Vec<String>
```



## bookclerk_plugin_abi::sql_types::catalog_page_statement

*Function*

Requires a catalog page execute reply to contain exactly one statement.

Empty **rows** still mean end-of-catalog. An empty `statements` vec is not
EOF — adapters must fail closed rather than treat a dropped statement as
a finished scan.

# Errors

Returns when `reply.statements` is not length 1.

```rust
fn catalog_page_statement(reply: &crate::ExecuteReply) -> crate::Result<&crate::StatementResult>
```



## bookclerk_plugin_abi::sql_types::parse_create_index_sql

*Function*

Parses `CREATE [UNIQUE] INDEX [IF NOT EXISTS] name ON table (cols…) [WHERE …]`.

The column list and any supported tail must be complete SQL v1. Trailing
tokens (`USING`, `INCLUDE`, `COLLATE`, or garbage after `ON table`) fail.

```rust
fn parse_create_index_sql(sql: &str) -> Option<CreateIndexSchema>
```



## bookclerk_plugin_abi::sql_types::parse_create_table_schema

*Function*

Parses a canonical `CREATE TABLE` into columns / identity, if the head matches.

```rust
fn parse_create_table_schema(sql: &str) -> Option<CreateTableSchema>
```



## bookclerk_plugin_abi::sql_types::parse_drop_index_name

*Function*

Parses `DROP INDEX [IF EXISTS] name`.

```rust
fn parse_drop_index_name(sql: &str) -> Option<String>
```



## bookclerk_plugin_abi::sql_types::parse_drop_table_name

*Function*

Parses `DROP TABLE [IF EXISTS] name`.

```rust
fn parse_drop_table_name(sql: &str) -> Option<String>
```



## bookclerk_plugin_abi::sql_types::postgres_identity_function_name

*Function*

Postgres identity trigger function name for `table` (`bc_if_<digest>`).

```rust
fn postgres_identity_function_name(table: &str) -> String
```



## bookclerk_plugin_abi::sql_types::postgres_identity_object_digest

*Function*

SHA-256 digest prefix of a folded table name for adapter-private objects.

```rust
fn postgres_identity_object_digest(table: &str) -> String
```



## bookclerk_plugin_abi::sql_types::postgres_identity_trigger_name

*Function*

Postgres identity trigger name for `table` (`bc_it_<digest>`).

```rust
fn postgres_identity_trigger_name(table: &str) -> String
```



## bookclerk_plugin_abi::sql_types::require_sql_v1_helper_arity

*Function*

Rejects a helper call whose argument count is outside the v1 matrix.

# Errors

Returns `PluginError::invalid_params` when `n` is not a legal arity.

```rust
fn require_sql_v1_helper_arity(index: usize, name: &str, n: usize) -> crate::Result<()>
```



## bookclerk_plugin_abi::sql_types::reserved_catalog_relation_missing

*Function*

True when `message` is a missing-relation error for reserved `table`.

Matches SQLite `no such table: bookclerk_sql_catalog` (optional `main.`)
and PostgreSQL `relation "bookclerk_sql_catalog" does not exist`. Does not
match missing-column errors (`column "ordinal" does not exist`).

```rust
fn reserved_catalog_relation_missing(message: &str, table: &str) -> bool
```



## bookclerk_plugin_abi::sql_types::sql_catalog_create_table_sql

*Function*

`CREATE TABLE IF NOT EXISTS` for the reserved SQL v1 catalog.

```rust
fn sql_catalog_create_table_sql() -> String
```



## bookclerk_plugin_abi::sql_types::sql_catalog_page_rows

*Function*

Page size for durable catalog / schema SELECTs.

Uses the negotiated adapter `maxResultRows` when it is nonzero, then caps
at [`FIRST_PARTY_MAX_RESULT_ROWS`](db_execute.md#bookclerk_plugin_abidb_executefirst_party_max_result_rows).

```rust
fn sql_catalog_page_rows(max_result_rows: u32) -> u32
```



## bookclerk_plugin_abi::sql_types::sql_ddl_create_table_sql

*Function*

Durable canonical DDL companion table.

```rust
fn sql_ddl_create_table_sql() -> String
```



## bookclerk_plugin_abi::sql_types::sql_host_bookkeeping_type_env

*Function*

Host bookkeeping tables present on every binding/library database.

Receipt wrap SQL is typed against this plus the plugin catalog. Adapters
merge it when rebuilding proofs after Cap'n drops host-private proofs.

```rust
fn sql_host_bookkeeping_type_env() -> SqlTypeEnv
```



## bookclerk_plugin_abi::sql_types::sql_schema_create_table_sql

*Function*

Table-level fingerprint companion table.

```rust
fn sql_schema_create_table_sql() -> String
```



## bookclerk_plugin_abi::sql_types::sql_type_env_from_canonical_ddl

*Function*

Reconstructs a type environment from one canonical `CREATE TABLE` statement.

Multi-statement packs must use [`sql_type_env_from_canonical_statements`](sql_types.md#bookclerk_plugin_abisql_typessql_type_env_from_canonical_statements).

```rust
fn sql_type_env_from_canonical_ddl(sql: &str) -> SqlTypeEnv
```



## bookclerk_plugin_abi::sql_types::sql_type_env_from_canonical_statements

*Function*

Reconstructs a type environment from already-separated canonical
`CREATE TABLE` statements.

Statement boundaries must come from [`crate::sql_v1_pack_statements`](sql_text.md#bookclerk_plugin_abisql_textsql_v1_pack_statements) or an
explicit list — this function does not split on `;`.

```rust
fn sql_type_env_from_canonical_statements<I, S>(statements: I) -> SqlTypeEnv
```



## bookclerk_plugin_abi::sql_types::sql_v1_helper_arity

*Function*

SQL-v1 helper arity `(min, max)` inclusive. Unknown names return `None`.

`json_object` is even arity `2..=`[`SQL_V1_JSON_OBJECT_MAX_ARGS`](sql_types.md#bookclerk_plugin_abisql_typessql_v1_json_object_max_args) (16 pairs).

```rust
fn sql_v1_helper_arity(name: &str) -> Option<(usize, usize)>
```



## bookclerk_plugin_abi::sql_types::sql_v1_helper_arity_ok

*Function*

True when `n` matches the SQL-v1 signature for `name`.

```rust
fn sql_v1_helper_arity_ok(name: &str, n: usize) -> bool
```



## bookclerk_plugin_abi::sql_types::sql_v1_ident_in_bounds

*Function*

True when `ident` (already case-folded) is a portable SQL v1 identifier.

```rust
fn sql_v1_ident_in_bounds(ident: &str) -> bool
```



## bookclerk_plugin_abi::sql_types::statement_sql_hash

*Function*

SHA-256 hex of the exact canonical SQL string a [`ResolvedStatement`](sql_proof.md#bookclerk_plugin_abisql_proofresolvedstatement) proves.

```rust
fn statement_sql_hash(sql: &str) -> String
```



## bookclerk_plugin_abi::sql_types::typecheck_create_index_sql

*Function*

Typechecks one canonical `CREATE INDEX` (including a partial `WHERE`).

# Errors

Returns when the statement is not admitted SQL v1, the table is missing,
an indexed column is not in [`SqlTypeEnv`](sql_types.md#bookclerk_plugin_abisql_typessqltypeenv), or a `WHERE` predicate is not
a complete boolean SQL-v1 expression.

```rust
fn typecheck_create_index_sql(sql: &str, env: &SqlTypeEnv) -> crate::Result<()>
```



## bookclerk_plugin_abi::sql_types::typecheck_execute_request

*Function*

Type-checks every statement in `req` against `env` plus in-request `CREATE`.

Host-authoritative SQL is still typed (authorization may be skipped
elsewhere). Unknown columns fail closed, including when the catalog is empty.

# Errors

Returns `PluginError::invalid_params` when an expression is not in the
v1 type contract.

```rust
fn typecheck_execute_request(req: &crate::ExecuteRequest, env: &SqlTypeEnv) -> crate::Result<()>
```



## bookclerk_plugin_abi::sql_types::typecheck_execute_request_proofs

*Function*

Host-private proofs for one execute request (not a guest SDK surface).

# Errors

Returns `PluginError::invalid_params` when an expression is not in the
v1 type contract.

```rust
fn typecheck_execute_request_proofs(req: &crate::ExecuteRequest, env: &SqlTypeEnv) -> crate::Result<Vec<crate::sql_proof::ResolvedStatement>>
```



## bookclerk_plugin_abi::sql_types::unify_types

*Function*

Unifies two types (`NULL` is the identity; INTEGER+REAL → REAL).

# Errors

Returns `PluginError::invalid_params` when the types cannot unify.

```rust
fn unify_types(index: usize, a: SqlType, b: SqlType) -> crate::Result<SqlType>
```



