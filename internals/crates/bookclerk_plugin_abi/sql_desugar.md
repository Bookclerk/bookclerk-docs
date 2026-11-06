# bookclerk_plugin_abi::sql_desugar

Host semantic desugars for canonical Bookclerk SQL.

These rewrites are **backend-independent** and define Bookclerk behavior:
unspecified `ORDER BY` matches SQLite NULL ordering, and `/` `%` by zero
yield NULL via portable `NULLIF`. Adapters must not repeat them (doing so
would double-wrap `NULLIF` and shift proof spans).

| Item | Kind | Summary |
| --- | --- | --- |
| [`desugar_canonical_sql`](#desugar_canonical_sql) | function | Applies host semantic desugars to canonical SQL. |
| [`desugar_execute_request`](#desugar_execute_request) | function | Desugars every statement in `req` in place. |

## Functions

### desugar_canonical_sql

```rust
pub fn desugar_canonical_sql(sql: &str) -> String
```

Applies host semantic desugars to canonical SQL.

Explicit `NULLS FIRST`/`LAST` and a divisor that is structurally
`NULLIF(<expression>, 0)` are left unchanged. A `NULLIF` call whose second
argument is not the integer `0` (for example `NULLIF(0, 1)`) still gains
an outer zero guard. Callers must run this **once** before proof
generation; adapters must not repeat it.

### desugar_execute_request

```rust
pub fn desugar_execute_request(req: &mut ExecuteRequest)
```

Desugars every statement in `req` in place.
