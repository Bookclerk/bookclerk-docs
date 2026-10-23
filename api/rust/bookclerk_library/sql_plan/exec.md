**bookclerk_library > sql_plan > exec**

# Module: sql_plan::exec

## Contents

**Functions**

- [`execute_compiled_on`](#execute_compiled_on) - Executes a compiled named atomic as one transaction and interprets the reply.
- [`execute_compiled_on_capped`](#execute_compiled_on_capped) - Like [`execute_compiled_on`](exec.md#bookclerk_librarysql_planexecexecute_compiled_on), failing when a statement returns more than `max_result_rows`.
- [`execute_typed_on`](#execute_typed_on) - Executes a typed request as one transaction on the opened adapter connection
- [`execute_typed_on_binding`](#execute_typed_on_binding) - Typed execute against an isolated plugin binding.
- [`execute_typed_on_session`](#execute_typed_on_session) - [`execute_typed_on`](exec.md#bookclerk_librarysql_planexecexecute_typed_on) with session cancel / deadline checks.

---

## bookclerk_library::sql_plan::exec::execute_compiled_on

*Function*

Executes a compiled named atomic as one transaction and interprets the reply.

# Errors

Returns `LibraryError::Orm` when a statement fails. Application statuses
are returned as [`DbAtomicResult`](../atomic_ops.md#bookclerk_libraryatomic_opsdbatomicresult), not errors.

```rust
fn execute_compiled_on(db: &sea_orm::DatabaseConnection, compiled: crate::sql_plan::CompiledAtomic) -> crate::error::Result<crate::atomic_ops::DbAtomicResult>
```



## bookclerk_library::sql_plan::exec::execute_compiled_on_capped

*Function*

Like [`execute_compiled_on`](exec.md#bookclerk_librarysql_planexecexecute_compiled_on), failing when a statement returns more than `max_result_rows`.

`max_result_rows` of `0` means unlimited.

# Errors

Returns `LibraryError::Orm` when a statement fails or exceeds the row cap.

```rust
fn execute_compiled_on_capped(db: &sea_orm::DatabaseConnection, compiled: crate::sql_plan::CompiledAtomic, max_result_rows: u32) -> crate::error::Result<crate::atomic_ops::DbAtomicResult>
```



## bookclerk_library::sql_plan::exec::execute_typed_on

*Function*

Executes a typed request as one transaction on the opened adapter connection
against the host library.

Merges [`crate::migrations::host_sql_type_env`](../migrations.md#bookclerk_librarymigrationshost_sql_type_env) so host DML typechecks
against library tables. Isolated plugin bindings must use
[`execute_typed_on_binding`](exec.md#bookclerk_librarysql_planexecexecute_typed_on_binding).

# Errors

Returns `LibraryError::Orm` when a statement fails.

```rust
fn execute_typed_on(db: &sea_orm::DatabaseConnection, req: &bookclerk_plugin_abi::ExecuteRequest, max_result_rows: u32) -> crate::error::Result<bookclerk_plugin_abi::ExecuteReply>
```



## bookclerk_library::sql_plan::exec::execute_typed_on_binding

*Function*

Typed execute against an isolated plugin binding.

`catalog` is the binding type environment (bootstrap plus already-applied
plugin schema). It must not be the host library catalog.

# Errors

Returns `LibraryError::Orm` when a statement fails.

```rust
fn execute_typed_on_binding(db: &sea_orm::DatabaseConnection, req: &bookclerk_plugin_abi::ExecuteRequest, _timing_source: &str, max_result_rows: u32, catalog: &bookclerk_plugin_abi::SqlTypeEnv) -> crate::error::Result<bookclerk_plugin_abi::ExecuteReply>
```



## bookclerk_library::sql_plan::exec::execute_typed_on_session

*Function*

[`execute_typed_on`](exec.md#bookclerk_librarysql_planexecexecute_typed_on) with session cancel / deadline checks.

Always merges the host library type environment. Binding schema apply
must call [`execute_typed_on_binding`](exec.md#bookclerk_librarysql_planexecexecute_typed_on_binding) instead.

# Errors

Returns `LibraryError::Orm` when a statement fails or the session is interrupted.

```rust
fn execute_typed_on_session(db: &sea_orm::DatabaseConnection, req: &bookclerk_plugin_abi::ExecuteRequest, max_result_rows: u32, session: AtomicSession) -> crate::error::Result<bookclerk_plugin_abi::ExecuteReply>
```



