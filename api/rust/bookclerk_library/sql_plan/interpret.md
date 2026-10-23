**bookclerk_library > sql_plan > interpret**

# Module: sql_plan::interpret

## Contents

**Structs**

- [`PlanStmtResult`](#planstmtresult) - Rows produced by one plan statement.

**Functions**

- [`interpret_typed_exec`](#interpret_typed_exec) - Maps a typed [`ExecuteReply`](../../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecutereply) onto a host [`DbAtomicResult`](../atomic_ops.md#bookclerk_libraryatomic_opsdbatomicresult).

---

## bookclerk_library::sql_plan::interpret::PlanStmtResult

*Struct*

Rows produced by one plan statement.

**Fields:**
- `rows: Vec<serde_json::Value>` - Result-set rows (empty for DML).
- `rows_affected: u64` - Engine-reported rows affected.

**Trait Implementations:**

- **From**
  - `fn from(stmt: &StatementResult) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> PlanStmtResult`
- **Clone**
  - `fn clone(self: &Self) -> PlanStmtResult`



## bookclerk_library::sql_plan::interpret::interpret_typed_exec

*Function*

Maps a typed [`ExecuteReply`](../../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecutereply) onto a host [`DbAtomicResult`](../atomic_ops.md#bookclerk_libraryatomic_opsdbatomicresult).

```rust
fn interpret_typed_exec(compiled: &super::CompiledAtomic, reply: &bookclerk_plugin_abi::ExecuteReply, expected_hash: &str) -> crate::atomic_ops::DbAtomicResult
```



