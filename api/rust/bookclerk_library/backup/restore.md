**bookclerk_library > backup > restore**

# Module: backup::restore

## Contents

**Functions**

- [`apply_admitted_sql`](#apply_admitted_sql) - Applies admitted canonical DDL plus binding companions (tests / bootstrap).
- [`restore_backup_unit`](#restore_backup_unit) - Destructively replaces one logical database with `unit`. Never merges.

---

## bookclerk_library::backup::restore::apply_admitted_sql

*Function*

Applies admitted canonical DDL plus binding companions (tests / bootstrap).

# Errors

Returns when a statement fails.

```rust
fn apply_admitted_sql(db: &sea_orm::DatabaseConnection, statements: &[&str], kind: super::CanonicalRestoreKind) -> crate::error::Result<()>
```



## bookclerk_library::backup::restore::restore_backup_unit

*Function*

Destructively replaces one logical database with `unit`. Never merges.

Verifies every referenced object before the first `DROP`. Plugin restore
rebuilds catalog companions from admitted SQL. It does not run plugin-owned
migrations. When `preserve_plugin_registry` is true, library restore does
not drop or replay `plugin_databases` (environment-local placement).
Row inserts are parameterized canonical Bookclerk SQL (`?` + typed
`DbValue` binds); adapters lower placeholders at the execute edge.
SQLite restore defers foreign keys on the restore transaction and runs
`PRAGMA foreign_key_check` before commit. It never toggles
`PRAGMA foreign_keys` on the pool connection.

# Errors

Returns when validation fails, the adapter cannot provide complete unit
replacement, a bind/payload/request budget is exceeded, or DDL/DML cannot
be applied.

```rust
fn restore_backup_unit(db: &sea_orm::DatabaseConnection, repo: &super::repository::BackupRepository, unit: &super::BackupUnit, kind: super::CanonicalRestoreKind, opts: &super::CanonicalRestoreOpts, preserve_plugin_registry: bool) -> crate::error::Result<()>
```



