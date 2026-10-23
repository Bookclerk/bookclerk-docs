**bookclerk_db_exec > lower**

# Module: lower

## Contents

**Functions**

- [`lower_canonical_ddl_to_postgres`](#lower_canonical_ddl_to_postgres) - Lowers one canonical Bookclerk **DDL** statement onto PostgreSQL.
- [`lower_canonical_ddl_to_postgres_with`](#lower_canonical_ddl_to_postgres_with) - [`lower_canonical_ddl_to_postgres`](lower.md#bookclerk_db_execlowerlower_canonical_ddl_to_postgres) using `env` for tie-break column types.
- [`lower_canonical_sql`](#lower_canonical_sql) - Lowers canonical Bookclerk **DML/query** SQL for `backend`.
- [`lower_canonical_sql_typed`](#lower_canonical_sql_typed) - [`lower_canonical_sql`](lower.md#bookclerk_db_execlowerlower_canonical_sql) plus proof-directed overflow and Postgres `COLLATE "C"`.
- [`lower_canonical_to_postgres`](#lower_canonical_to_postgres) - Lowers canonical SQLite-shaped SQL onto PostgreSQL.

---

## bookclerk_db_exec::lower::lower_canonical_ddl_to_postgres

*Function*

Lowers one canonical Bookclerk **DDL** statement onto PostgreSQL.

Type/identity rewrites plus [`lower_canonical_to_postgres`](lower.md#bookclerk_db_execlowerlower_canonical_to_postgres) (placeholders,
`INSERT OR IGNORE`). Host schema packs use this through
[`crate::schema_sql_for_backend`](schema_postgres.md#bookclerk_db_execschema_postgresschema_sql_for_backend); binding DDL uses the type rewrite at
the adapter edge and then the DML helper pass.
# Errors

Returns when a Postgres `CREATE INDEX` has a `COLLATE NOCASE` tie-break or
a `lower`/`upper` column argument whose type is not in `env` (and this
call passes no env). The statement is not rewritten into a different index.

```rust
fn lower_canonical_ddl_to_postgres(sql: &str) -> Result<String, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_exec::lower::lower_canonical_ddl_to_postgres_with

*Function*

[`lower_canonical_ddl_to_postgres`](lower.md#bookclerk_db_execlowerlower_canonical_ddl_to_postgres) using `env` for tie-break column types.

# Errors

Returns when a Postgres `CREATE INDEX` key that needs `COLLATE "C"` cannot
be typed from `env`.

```rust
fn lower_canonical_ddl_to_postgres_with(sql: &str, env: Option<&bookclerk_plugin_abi::SqlTypeEnv>) -> Result<String, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_exec::lower::lower_canonical_sql

*Function*

Lowers canonical Bookclerk **DML/query** SQL for `backend`.

Every backend rewrites `INSERT OR IGNORE` to unique/PK `ON CONFLICT DO
NOTHING` (SQLite `OR IGNORE` would otherwise swallow `NOT NULL`). Postgres
adapters then rewrite helpers (`IFNULL`, `json_object` → TEXT,
`json_extract`, 2+-arg `min`/`max`, `json_valid`, `round`/`sum`/`avg`)
and `?` placeholders. Default `ORDER BY` NULLS and `/` `%` by-zero
`NULLIF` are host semantic desugars, not adapter rewrites. Binding and host
**DDL** type/identity rewrites
(`AUTOINCREMENT`, `BLOB`, `INTEGER`) stay on the adapter execution edge
([`crate::schema_sql_for_backend`](schema_postgres.md#bookclerk_db_execschema_postgresschema_sql_for_backend), [`crate::lower_binding_ddl_execute_request`](schema_postgres.md#bookclerk_db_execschema_postgreslower_binding_ddl_execute_request))
so this function does not classify statements.

```rust
fn lower_canonical_sql(backend: sea_orm::DatabaseBackend, sql: &str) -> String
```



## bookclerk_db_exec::lower::lower_canonical_sql_typed

*Function*

[`lower_canonical_sql`](lower.md#bookclerk_db_execlowerlower_canonical_sql) plus proof-directed overflow and Postgres `COLLATE "C"`.

`proof` must be bound to `sql` (the canonical string before mechanical
lowering). When absent, TEXT collation and INTEGER overflow wraps are not
applied. TEXT collation runs **before** INTEGER overflow wrapping so a
TEXT span nested in `length(name) + 1` is remapped into the overflow
operands. Overflow sites are applied innermost-first.

# Errors

Returns when `proof` is present and not bound to `sql`, or when a
hash-correct sidecar cannot be applied (`ResolvedStatement::validate_for`).

```rust
fn lower_canonical_sql_typed(backend: sea_orm::DatabaseBackend, sql: &str, proof: Option<&bookclerk_plugin_abi::ResolvedStatement>) -> Result<String, bookclerk_plugin_abi::PluginError>
```



## bookclerk_db_exec::lower::lower_canonical_to_postgres

*Function*

Lowers canonical SQLite-shaped SQL onto PostgreSQL.

```rust
fn lower_canonical_to_postgres(sql: &str) -> String
```



