**bookclerk_library > backup > schema**

# Module: backup::schema

## Contents

**Functions**

- [`admit_canonical_schema`](#admit_canonical_schema) - Parses and fully admits a diagnostic/imported canonical SQL script.
- [`admit_canonical_statements`](#admit_canonical_statements) - Admits already-separated canonical `CREATE TABLE` / `CREATE INDEX` statements.
- [`filter_library_pack_ddl`](#filter_library_pack_ddl) - Library migration packs may include seed `INSERT`s. Those are table data
- [`library_canonical_schema`](#library_canonical_schema) - Admitted library schema for this binary's current pack (frozen ups + unreleased).
- [`library_canonical_schema_for_state`](#library_canonical_schema_for_state) - Admitted library schema for `state`.
- [`library_ddl_for_schema_state`](#library_ddl_for_schema_state) - Library DDL that matches `state`, not necessarily this binary's latest pack.
- [`sort_tables_by_foreign_keys`](#sort_tables_by_foreign_keys) - FK-safe table order (parents before children). Cycles fail closed.

---

## bookclerk_library::backup::schema::admit_canonical_schema

*Function*

Parses and fully admits a diagnostic/imported canonical SQL script.

Production capture, checksum, apply, and restore paths use already-separated
statement lists ([`admit_canonical_statements`](schema.md#bookclerk_librarybackupschemaadmit_canonical_statements)). This helper exists for
tests and imported scripts that still arrive as one packable string.

# Errors

Returns when the packer rejects `sql` or any statement is not admitted.

```rust
fn admit_canonical_schema(sql_contract_version: u32, sql: &str) -> crate::error::Result<super::CanonicalDatabaseSchema>
```



## bookclerk_library::backup::schema::admit_canonical_statements

*Function*

Admits already-separated canonical `CREATE TABLE` / `CREATE INDEX` statements.

# Errors

Returns when any statement is not admitted Bookclerk SQL.

```rust
fn admit_canonical_statements(sql_contract_version: u32, statements: &[&str]) -> crate::error::Result<super::CanonicalDatabaseSchema>
```



## bookclerk_library::backup::schema::filter_library_pack_ddl

*Function*

Library migration packs may include seed `INSERT`s. Those are table data
(captured from rows), not schema objects. This keeps admitted
`CREATE TABLE` / `CREATE INDEX` and skips seed DML. Any other statement
fails closed.

# Errors

Returns when a non-empty statement is neither admitted DDL nor seed DML.

```rust
fn filter_library_pack_ddl(sql: &str) -> crate::error::Result<String>
```



## bookclerk_library::backup::schema::library_canonical_schema

*Function*

Admitted library schema for this binary's current pack (frozen ups + unreleased).

# Errors

Returns when current canonical SQL is not fully admitted.

```rust
fn library_canonical_schema() -> crate::error::Result<super::CanonicalDatabaseSchema>
```



## bookclerk_library::backup::schema::library_canonical_schema_for_state

*Function*

Admitted library schema for `state`.

# Errors

Returns when DDL cannot be resolved or is not fully admitted.

```rust
fn library_canonical_schema_for_state(state: &crate::schema_state::SchemaState) -> crate::error::Result<super::CanonicalDatabaseSchema>
```



## bookclerk_library::backup::schema::library_ddl_for_schema_state

*Function*

Library DDL that matches `state`, not necessarily this binary's latest pack.

# Errors

Returns when the database is uninitialized, checksums do not match, or a
frozen version is unknown to this binary.

```rust
fn library_ddl_for_schema_state(state: &crate::schema_state::SchemaState) -> crate::error::Result<String>
```



## bookclerk_library::backup::schema::sort_tables_by_foreign_keys

*Function*

FK-safe table order (parents before children). Cycles fail closed.

Column-level `REFERENCES` and table-level `FOREIGN KEY` both count.
Self-references are ignored. References to tables outside this schema
are ignored. Ready nodes keep original declaration order so a child named
`a_child` that references `z_parent` still waits for the parent.

# Errors

Returns when the FK graph contains a cycle.

```rust
fn sort_tables_by_foreign_keys(tables: Vec<super::CanonicalTableSchema>) -> crate::error::Result<Vec<super::CanonicalTableSchema>>
```



