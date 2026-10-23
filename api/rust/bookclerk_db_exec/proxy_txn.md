**bookclerk_db_exec > proxy_txn**

# Module: proxy_txn

## Contents

**Structs**

- [`ExecBudget`](#execbudget) - Request-scoped deadline and query row cap for one atomic attempt.
- [`ExecuteRowCapGuard`](#executerowcapguard) - Restores `ExecBudget::raw_query_row_cap` when dropped.

**Enums**

- [`AtomicInterruptKind`](#atomicinterruptkind) - Kind of injected atomic interrupt.
- [`AtomicInterruptPhase`](#atomicinterruptphase) - Phase at which an injected atomic interrupt fires.

**Functions**

- [`arm_exec_budget`](#arm_exec_budget) - Legacy no-op. Budgets are request-scoped via [`with_exec_budget`](proxy_txn.md#bookclerk_db_execproxy_txnwith_exec_budget).
- [`clear_exec_budget`](#clear_exec_budget) - Legacy no-op. Budgets are request-scoped via [`with_exec_budget`](proxy_txn.md#bookclerk_db_execproxy_txnwith_exec_budget).
- [`consume_atomic_interrupt`](#consume_atomic_interrupt) - Consume an injected interrupt when the executor reaches `phase`.
- [`consume_begin_injection`](#consume_begin_injection) - Consume one injected begin failure if any remain for this task.
- [`consume_commit_injection`](#consume_commit_injection) - Consume one injected commit failure if any remain for this task.
- [`consume_savepoint_release_injection`](#consume_savepoint_release_injection) - Consume one injected savepoint `RELEASE` failure if any remain for this task.
- [`consume_savepoint_rollback_injection`](#consume_savepoint_rollback_injection) - Consume one injected savepoint `ROLLBACK TO` failure if any remain for this task.
- [`current_exec_budget`](#current_exec_budget) - Request-scoped budget for the current Tokio task, if an atomic attempt armed one.
- [`exec_deadline_expired`](#exec_deadline_expired) - True when the current task's armed deadline has elapsed.
- [`exec_deadline_remaining_ms`](#exec_deadline_remaining_ms) - Remaining milliseconds until the current task's deadline (`None` if unlimited).
- [`inject_atomic_interrupt`](#inject_atomic_interrupt) - Queue an atomic session interrupt for this task (cancel or deadline at `phase`).
- [`inject_atomic_interrupt_after`](#inject_atomic_interrupt_after) - Like [`inject_atomic_interrupt`](proxy_txn.md#bookclerk_db_execproxy_txninject_atomic_interrupt), ignoring `skip` observations of `phase` first.
- [`inject_begin_failures`](#inject_begin_failures) - Queue `n` injected `BEGIN` failures for this task.
- [`inject_commit_failures`](#inject_commit_failures) - Queue `n` injected `COMMIT` failures for this task.
- [`inject_savepoint_release_failures`](#inject_savepoint_release_failures) - Queue `n` injected nested-savepoint `RELEASE` failures for this task.
- [`inject_savepoint_rollback_failures`](#inject_savepoint_rollback_failures) - Queue `n` injected nested-savepoint `ROLLBACK TO` failures for this task.
- [`is_txn_broken`](#is_txn_broken) - Whether this task has a sticky begin/commit fault.
- [`note_begin_failed`](#note_begin_failed) - Record that `BEGIN` failed so later statements cannot autocommit.
- [`note_commit_failed`](#note_commit_failed) - Record that `COMMIT` failed so the caller cannot treat the txn as durable.
- [`note_query_row`](#note_query_row) - Records one materialized query row against the current task budget.
- [`query_row_cap`](#query_row_cap) - Advertised query row cap for the current task (`None` if unlimited).
- [`query_rows_seen`](#query_rows_seen) - Rows materialized by the last completed capped query (tests).
- [`record_query_rows_seen`](#record_query_rows_seen) - Records `seen` for [`query_rows_seen`](proxy_txn.md#bookclerk_db_execproxy_txnquery_rows_seen) after an atomic attempt completes.
- [`suspend_execute_row_cap`](#suspend_execute_row_cap) - Suspends the execute-request row cap until the guard is dropped.
- [`take_txn_fault`](#take_txn_fault) - Take and clear this task's sticky fault (after SeaORM commit/rollback).
- [`txn_broken_err`](#txn_broken_err) - `DbErr` for refusing statements after a failed begin/commit.
- [`with_exec_budget`](#with_exec_budget) - Runs `fut` with `budget` as the request-scoped execution budget.

---

## bookclerk_db_exec::proxy_txn::AtomicInterruptKind

*Enum*

Kind of injected atomic interrupt.

**Variants:**
- `Cancel` - Session cancel.
- `Deadline` - Deadline elapsed.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &AtomicInterruptKind) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> AtomicInterruptKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_db_exec::proxy_txn::AtomicInterruptPhase

*Enum*

Phase at which an injected atomic interrupt fires.

**Variants:**
- `BeforeBegin` - Before `BEGIN` / HTTP batch.
- `BetweenStatements` - Between statements during the transaction.
- `AroundCommit` - Around `COMMIT` / HTTP return (ambiguous).

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &AtomicInterruptPhase) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> AtomicInterruptPhase`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_db_exec::proxy_txn::ExecBudget

*Struct*

Request-scoped deadline and query row cap for one atomic attempt.

Stored on the Tokio task that owns the attempt and copied onto the engine
connection after it acquires the exclusive lease, so concurrent attempts
cannot overwrite each other.

**Methods:**

- `fn new(deadline_unix_ms: Option<u64>, query_row_cap: u32) -> Arc<Self>` - Builds a budget for one atomic attempt.
- `fn with_cancel(deadline_unix_ms: Option<u64>, query_row_cap: u32, cancel: Option<Arc<AtomicBool>>) -> Arc<Self>` - Builds a budget for one atomic attempt that also carries the session's
- `fn unlimited() -> Arc<Self>` - No deadline and no row cap.
- `fn deadline_expired(self: &Self) -> bool` - True when an armed deadline has elapsed.
- `fn cancelled(self: &Self) -> bool` - True when the owning session's host cancel flag is raised.
- `fn remaining_ms(self: &Self) -> Option<u64>` - Remaining milliseconds until the armed deadline (`None` if unlimited).
- `fn cap(self: &Self) -> Option<usize>` - Advertised query row cap (`None` if unlimited).
- `fn set_query_row_cap(self: &Self, cap: usize)` - Sets the query row cap (`0` = unlimited).
- `fn raw_query_row_cap(self: &Self) -> usize` - Raw cap including `0` for unlimited.
- `fn reset_rows_seen(self: &Self)` - Clears the per-query row counter.
- `fn note_row(self: &Self) -> bool` - Records one materialized query row; `true` when the caller should stop.
- `fn rows_seen(self: &Self) -> usize` - Rows materialized by the current capped query.
- `fn set_positional_columns(self: &Self, columns: Vec<DbColumn>)` - Records rusqlite/engine column metadata for the in-flight statement.
- `fn take_positional_columns(self: &Self) -> Option<Vec<DbColumn>>` - Takes engine column metadata recorded for this request, if any.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_db_exec::proxy_txn::ExecuteRowCapGuard

*Struct*

Restores `ExecBudget::raw_query_row_cap` when dropped.

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)`



## bookclerk_db_exec::proxy_txn::arm_exec_budget

*Function*

Legacy no-op. Budgets are request-scoped via [`with_exec_budget`](proxy_txn.md#bookclerk_db_execproxy_txnwith_exec_budget).

```rust
fn arm_exec_budget(_deadline_unix_ms: Option<u64>, _query_row_cap: u32)
```



## bookclerk_db_exec::proxy_txn::clear_exec_budget

*Function*

Legacy no-op. Budgets are request-scoped via [`with_exec_budget`](proxy_txn.md#bookclerk_db_execproxy_txnwith_exec_budget).

```rust
fn clear_exec_budget()
```



## bookclerk_db_exec::proxy_txn::consume_atomic_interrupt

*Function*

Consume an injected interrupt when the executor reaches `phase`.

```rust
fn consume_atomic_interrupt(phase: AtomicInterruptPhase) -> Option<AtomicInterruptKind>
```



## bookclerk_db_exec::proxy_txn::consume_begin_injection

*Function*

Consume one injected begin failure if any remain for this task.

```rust
fn consume_begin_injection() -> bool
```



## bookclerk_db_exec::proxy_txn::consume_commit_injection

*Function*

Consume one injected commit failure if any remain for this task.

```rust
fn consume_commit_injection() -> bool
```



## bookclerk_db_exec::proxy_txn::consume_savepoint_release_injection

*Function*

Consume one injected savepoint `RELEASE` failure if any remain for this task.

```rust
fn consume_savepoint_release_injection() -> bool
```



## bookclerk_db_exec::proxy_txn::consume_savepoint_rollback_injection

*Function*

Consume one injected savepoint `ROLLBACK TO` failure if any remain for this task.

```rust
fn consume_savepoint_rollback_injection() -> bool
```



## bookclerk_db_exec::proxy_txn::current_exec_budget

*Function*

Request-scoped budget for the current Tokio task, if an atomic attempt armed one.

```rust
fn current_exec_budget() -> Option<std::sync::Arc<ExecBudget>>
```



## bookclerk_db_exec::proxy_txn::exec_deadline_expired

*Function*

True when the current task's armed deadline has elapsed.

```rust
fn exec_deadline_expired() -> bool
```



## bookclerk_db_exec::proxy_txn::exec_deadline_remaining_ms

*Function*

Remaining milliseconds until the current task's deadline (`None` if unlimited).

```rust
fn exec_deadline_remaining_ms() -> Option<u64>
```



## bookclerk_db_exec::proxy_txn::inject_atomic_interrupt

*Function*

Queue an atomic session interrupt for this task (cancel or deadline at `phase`).

```rust
fn inject_atomic_interrupt(phase: AtomicInterruptPhase, kind: AtomicInterruptKind)
```



## bookclerk_db_exec::proxy_txn::inject_atomic_interrupt_after

*Function*

Like [`inject_atomic_interrupt`](proxy_txn.md#bookclerk_db_execproxy_txninject_atomic_interrupt), ignoring `skip` observations of `phase` first.

```rust
fn inject_atomic_interrupt_after(phase: AtomicInterruptPhase, kind: AtomicInterruptKind, skip: u32)
```



## bookclerk_db_exec::proxy_txn::inject_begin_failures

*Function*

Queue `n` injected `BEGIN` failures for this task.

```rust
fn inject_begin_failures(n: u32)
```



## bookclerk_db_exec::proxy_txn::inject_commit_failures

*Function*

Queue `n` injected `COMMIT` failures for this task.

```rust
fn inject_commit_failures(n: u32)
```



## bookclerk_db_exec::proxy_txn::inject_savepoint_release_failures

*Function*

Queue `n` injected nested-savepoint `RELEASE` failures for this task.

Also arms a process-wide counter so a txn worker (different Tokio task)
can observe the fault.

```rust
fn inject_savepoint_release_failures(n: u32)
```



## bookclerk_db_exec::proxy_txn::inject_savepoint_rollback_failures

*Function*

Queue `n` injected nested-savepoint `ROLLBACK TO` failures for this task.

Also arms a process-wide counter so a txn worker (different Tokio task)
can observe the fault.

```rust
fn inject_savepoint_rollback_failures(n: u32)
```



## bookclerk_db_exec::proxy_txn::is_txn_broken

*Function*

Whether this task has a sticky begin/commit fault.

```rust
fn is_txn_broken() -> bool
```



## bookclerk_db_exec::proxy_txn::note_begin_failed

*Function*

Record that `BEGIN` failed so later statements cannot autocommit.

```rust
fn note_begin_failed<impl std::fmt::Display>(err: impl Trait)
```



## bookclerk_db_exec::proxy_txn::note_commit_failed

*Function*

Record that `COMMIT` failed so the caller cannot treat the txn as durable.

```rust
fn note_commit_failed<impl std::fmt::Display>(err: impl Trait)
```



## bookclerk_db_exec::proxy_txn::note_query_row

*Function*

Records one materialized query row against the current task budget.

```rust
fn note_query_row() -> bool
```



## bookclerk_db_exec::proxy_txn::query_row_cap

*Function*

Advertised query row cap for the current task (`None` if unlimited).

```rust
fn query_row_cap() -> Option<usize>
```



## bookclerk_db_exec::proxy_txn::query_rows_seen

*Function*

Rows materialized by the last completed capped query (tests).

```rust
fn query_rows_seen() -> usize
```



## bookclerk_db_exec::proxy_txn::record_query_rows_seen

*Function*

Records `seen` for [`query_rows_seen`](proxy_txn.md#bookclerk_db_execproxy_txnquery_rows_seen) after an atomic attempt completes.

```rust
fn record_query_rows_seen(seen: usize)
```



## bookclerk_db_exec::proxy_txn::suspend_execute_row_cap

*Function*

Suspends the execute-request row cap until the guard is dropped.

Catalog / `sqlite_master` loads are not guest result rows; they must not
fail a tight `maxResultRows` on the surrounding execute.

```rust
fn suspend_execute_row_cap() -> Option<ExecuteRowCapGuard>
```



## bookclerk_db_exec::proxy_txn::take_txn_fault

*Function*

Take and clear this task's sticky fault (after SeaORM commit/rollback).

```rust
fn take_txn_fault() -> Option<String>
```



## bookclerk_db_exec::proxy_txn::txn_broken_err

*Function*

`DbErr` for refusing statements after a failed begin/commit.

```rust
fn txn_broken_err() -> sea_orm::DbErr
```



## bookclerk_db_exec::proxy_txn::with_exec_budget

*Function*

Runs `fut` with `budget` as the request-scoped execution budget.

```rust
fn with_exec_budget<F, Fut, T>(budget: std::sync::Arc<ExecBudget>, fut: F) -> T
```



