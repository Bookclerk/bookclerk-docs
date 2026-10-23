**bookclerk_db_exec > canonical**

# Module: canonical

## Contents

**Functions**

- [`canonical_statement`](#canonical_statement) - Builds a SeaORM statement that keeps canonical `?` placeholders.
- [`execute_canonical`](#execute_canonical) - Executes already-canonical host SQL. Never desugars or physically lowers.
- [`query_canonical`](#query_canonical) - Queries already-canonical host SQL. Never desugars or physically lowers.

---

## bookclerk_db_exec::canonical::canonical_statement

*Function*

Builds a SeaORM statement that keeps canonical `?` placeholders.

```rust
fn canonical_statement<impl Into<String>, impl IntoIterator<Item = Value>>(sql: impl Trait, values: impl Trait) -> sea_orm::Statement
```



## bookclerk_db_exec::canonical::execute_canonical

*Function*

Executes already-canonical host SQL. Never desugars or physically lowers.

# Errors

Returns when the connection rejects the statement.

```rust
fn execute_canonical<C, impl IntoIterator<Item = Value>>(db: &C, sql: &str, values: impl Trait) -> Result<sea_orm::ExecResult, sea_orm::DbErr>
```



## bookclerk_db_exec::canonical::query_canonical

*Function*

Queries already-canonical host SQL. Never desugars or physically lowers.

# Errors

Returns when the connection rejects the statement.

```rust
fn query_canonical<C, impl IntoIterator<Item = Value>>(db: &C, sql: &str, values: impl Trait) -> Result<Vec<sea_orm::QueryResult>, sea_orm::DbErr>
```



