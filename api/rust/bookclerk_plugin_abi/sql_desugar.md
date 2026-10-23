**bookclerk_plugin_abi > sql_desugar**

# Module: sql_desugar

## Contents

**Functions**

- [`desugar_canonical_sql`](#desugar_canonical_sql) - Applies host semantic desugars to canonical SQL.
- [`desugar_execute_request`](#desugar_execute_request) - Desugars every statement in `req` in place.

---

## bookclerk_plugin_abi::sql_desugar::desugar_canonical_sql

*Function*

Applies host semantic desugars to canonical SQL.

Explicit `NULLS FIRST`/`LAST` and a divisor that is structurally
`NULLIF(<expression>, 0)` are left unchanged. A `NULLIF` call whose second
argument is not the integer `0` (for example `NULLIF(0, 1)`) still gains
an outer zero guard. Callers must run this **once** before proof
generation; adapters must not repeat it.

```rust
fn desugar_canonical_sql(sql: &str) -> String
```



## bookclerk_plugin_abi::sql_desugar::desugar_execute_request

*Function*

Desugars every statement in `req` in place.

```rust
fn desugar_execute_request(req: & mut crate::ExecuteRequest)
```



