**bookclerk_db_exec > schema_postgres**

# Module: schema_postgres

## Contents

**Functions**

- [`binding_companions`](#binding_companions) - Catalog + Postgres identity companions for one canonical binding statement.
- [`collapse_companion_groups`](#collapse_companion_groups) - Collapses interleaved companion results back to one result per original statement.
- [`collapse_host_schema_results`](#collapse_host_schema_results) - Collapses adapter-expanded host-schema results using per-statement groups.
- [`expand_binding_execute_request`](#expand_binding_execute_request) - Expands a binding request with adapter-private catalog/identity companions.
- [`expand_host_schema_batch`](#expand_host_schema_batch) - Expands `[stmt, …, version_marker]` at the adapter execution edge.
- [`expand_host_schema_execute_request`](#expand_host_schema_execute_request) - Expands a typed host schema batch at the adapter execution edge.
- [`expand_host_schema_execute_request_grouped`](#expand_host_schema_execute_request_grouped) - [`expand_host_schema_execute_request`](schema_postgres.md#bookclerk_db_execschema_postgresexpand_host_schema_execute_request) plus collapse groups.
- [`is_host_schema_version_marker`](#is_host_schema_version_marker) - True when `sql` is a host schema version marker (`bookclerk_schema_migrations` or `PRAGMA user_version`).
- [`lower_binding_ddl_execute_request`](#lower_binding_ddl_execute_request) - Applies [`lower_binding_sql_for_backend`](schema_postgres.md#bookclerk_db_execschema_postgreslower_binding_sql_for_backend) to every statement in `req`.
- [`lower_binding_sql_for_backend`](#lower_binding_sql_for_backend) - Mechanical type/identity lowering for one **binding** statement.
- [`postgres_identity_companions`](#postgres_identity_companions) - Postgres-only companion DDL: transactional identity counter + BEFORE INSERT trigger.
- [`realize_binding_ddl`](#realize_binding_ddl) - Applies one canonical binding DDL statement (plugin restore).
- [`realize_host_ddl`](#realize_host_ddl) - Applies one canonical host DDL statement (library restore / schema).
- [`schema_sql_for_backend`](#schema_sql_for_backend) - SQL text for one host-schema statement on the live connection backend.

---

## bookclerk_db_exec::schema_postgres::binding_companions

*Function*

Catalog + Postgres identity companions for one canonical binding statement.

```rust
fn binding_companions(backend: sea_orm::DatabaseBackend, canonical: &str) -> Vec<String>
```



## bookclerk_db_exec::schema_postgres::collapse_companion_groups

*Function*

Collapses interleaved companion results back to one result per original statement.

```rust
fn collapse_companion_groups(groups: &[usize], results: Vec<bookclerk_plugin_abi::StatementResult>) -> Vec<bookclerk_plugin_abi::StatementResult>
```



## bookclerk_db_exec::schema_postgres::collapse_host_schema_results

*Function*

Collapses adapter-expanded host-schema results using per-statement groups.

```rust
fn collapse_host_schema_results(groups: &[usize], results: Vec<bookclerk_plugin_abi::StatementResult>) -> Vec<bookclerk_plugin_abi::StatementResult>
```



## bookclerk_db_exec::schema_postgres::expand_binding_execute_request

*Function*

Expands a binding request with adapter-private catalog/identity companions.

Returns the expanded request and per-original-statement group sizes so
callers can collapse results back to the wire statement count.

```rust
fn expand_binding_execute_request(backend: sea_orm::DatabaseBackend, req: &bookclerk_plugin_abi::ExecuteRequest) -> (bookclerk_plugin_abi::ExecuteRequest, Vec<usize>)
```



## bookclerk_db_exec::schema_postgres::expand_host_schema_batch

*Function*

Expands `[stmt, …, version_marker]` at the adapter execution edge.

Each incoming string is already one canonical statement. Adapters lower
DDL per statement and may insert identity companions; they do not split
SQL. Non-DDL statements stay canonical so their host proofs still bind
when the executor lowers them.
# Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

```rust
fn expand_host_schema_batch(backend: sea_orm::DatabaseBackend, batch: &[String]) -> Result<Option<Vec<String>>, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_exec::schema_postgres::expand_host_schema_execute_request

*Function*

Expands a typed host schema batch at the adapter execution edge.

Returns the expanded request. When the batch is not a host schema apply
unit, the request is unchanged.

# Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

```rust
fn expand_host_schema_execute_request(backend: sea_orm::DatabaseBackend, req: &bookclerk_plugin_abi::ExecuteRequest) -> Result<bookclerk_plugin_abi::ExecuteRequest, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_exec::schema_postgres::expand_host_schema_execute_request_grouped

*Function*

[`expand_host_schema_execute_request`](schema_postgres.md#bookclerk_db_execschema_postgresexpand_host_schema_execute_request) plus collapse groups.

# Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

```rust
fn expand_host_schema_execute_request_grouped(backend: sea_orm::DatabaseBackend, req: &bookclerk_plugin_abi::ExecuteRequest) -> Result<(bookclerk_plugin_abi::ExecuteRequest, Vec<usize>), bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_exec::schema_postgres::is_host_schema_version_marker

*Function*

True when `sql` is a host schema version marker (`bookclerk_schema_migrations` or `PRAGMA user_version`).

```rust
fn is_host_schema_version_marker(sql: &str) -> bool
```



## bookclerk_db_exec::schema_postgres::lower_binding_ddl_execute_request

*Function*

Applies [`lower_binding_sql_for_backend`](schema_postgres.md#bookclerk_db_execschema_postgreslower_binding_sql_for_backend) to every statement in `req`.

Identity when no statement changes (SQLite/D1, or Postgres DML-only).

# Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

```rust
fn lower_binding_ddl_execute_request(backend: sea_orm::DatabaseBackend, req: &bookclerk_plugin_abi::ExecuteRequest) -> Result<bookclerk_plugin_abi::ExecuteRequest, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_exec::schema_postgres::lower_binding_sql_for_backend

*Function*

Mechanical type/identity lowering for one **binding** statement.

Hosts emit canonical SQLite-shaped `CREATE`/`DROP`. Postgres adapters
rewrite `AUTOINCREMENT`/`BLOB`/`INTEGER`/`REAL` here and turn
`ident COLLATE NOCASE` into `(lower(ident COLLATE "C"))`, with a text
tie-break as `(ident COLLATE "C")`, so page order matches SQLite.
Numeric tie-break keys stay bare. SQLite leaves the statement unchanged,
including `NOCASE` page indexes. DML stays for [`crate::lower_canonical_sql`](lower.md#bookclerk_db_execlowerlower_canonical_sql).

# Panics

Panics when `backend` is not SQLite or PostgreSQL. `DatabaseBackend` is
non-exhaustive; BookclerkSQL adapters do not implement other SeaORM
families.
# Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

```rust
fn lower_binding_sql_for_backend(backend: sea_orm::DatabaseBackend, sql: &str) -> Result<std::borrow::Cow<str>, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_exec::schema_postgres::postgres_identity_companions

*Function*

Postgres-only companion DDL: transactional identity counter + BEFORE INSERT trigger.

SQLite `AUTOINCREMENT` records the largest committed value. A PostgreSQL
`setval` sequence does not roll back with the `ExecuteRequest`. The adapter
keeps a `BIGINT` high-water row in [`bookclerk_plugin_abi::SQL_IDENTITY_TABLE`](../bookclerk_plugin_abi/sql_types.md#bookclerk_plugin_abisql_typessql_identity_table)
and assigns omit-ids from that row under `SELECT … FOR UPDATE`. Companions
are generated from **canonical** SQL (parsed identity column, not assumed
`id`) and executed internally (not extra Cap'n statements). Empty when `sql`
is not `CREATE TABLE` with `INTEGER PRIMARY KEY AUTOINCREMENT`.

```rust
fn postgres_identity_companions(sql: &str) -> Vec<String>
```



## bookclerk_db_exec::schema_postgres::realize_binding_ddl

*Function*

Applies one canonical binding DDL statement (plugin restore).

Includes adapter-private catalog DML and Postgres identity companions.

# Errors

Returns when the engine rejects the lowered DDL or a companion.

```rust
fn realize_binding_ddl<C>(conn: &C, canonical: &str) -> Result<(), sea_orm::DbErr>
```



## bookclerk_db_exec::schema_postgres::realize_host_ddl

*Function*

Applies one canonical host DDL statement (library restore / schema).

Postgres identity companions are installed; binding catalog DML is not.

# Errors

Returns when the engine rejects the lowered DDL or a companion.

```rust
fn realize_host_ddl<C>(conn: &C, canonical: &str) -> Result<(), sea_orm::DbErr>
```



## bookclerk_db_exec::schema_postgres::schema_sql_for_backend

*Function*

SQL text for one host-schema statement on the live connection backend.

SQLite-family backends return the canonical statement unchanged; Postgres
applies the mechanical DDL lowering. Unknown SeaORM backends fail closed.

# Panics

Panics when `backend` is not SQLite or PostgreSQL. `DatabaseBackend` is
non-exhaustive; BookclerkSQL adapters do not implement other SeaORM
families.
# Errors

Returns when Postgres cannot type a `CREATE INDEX` key that needs
`COLLATE "C"`.

```rust
fn schema_sql_for_backend(backend: sea_orm::DatabaseBackend, canonical: &str) -> Result<std::borrow::Cow<str>, bookclerk_plugin_abi::PluginError>
```



