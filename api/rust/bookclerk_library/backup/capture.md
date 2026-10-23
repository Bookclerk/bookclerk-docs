**bookclerk_library > backup > capture**

# Module: backup::capture

## Contents

**Functions**

- [`plugin_canonical_schema_from_ddl_catalog`](#plugin_canonical_schema_from_ddl_catalog) - Rebuilds plugin schema from durable `bookclerk_sql_ddl` on an open connection.

---

## bookclerk_library::backup::capture::plugin_canonical_schema_from_ddl_catalog

*Function*

Rebuilds plugin schema from durable `bookclerk_sql_ddl` on an open connection.

# Errors

Returns when the DDL catalog is missing, incomplete, or malformed.

```rust
fn plugin_canonical_schema_from_ddl_catalog(db: &sea_orm::DatabaseConnection) -> crate::error::Result<super::CanonicalDatabaseSchema>
```



