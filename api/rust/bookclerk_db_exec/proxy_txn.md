# bookclerk_db_exec::proxy_txn

Fail-closed acknowledgements for SeaORM's infallible proxy transaction hooks.

`ProxyDatabaseTrait::{begin, commit, rollback}` return `()`. A failed
`BEGIN` that is only logged leaves later statements in autocommit; a failed
`COMMIT` still looks like success to `sea_orm::DatabaseTransaction::commit`.
Proxies record a per-task fault instead and refuse subsequent statements.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ExecBudget`](#execbudget) | struct | Request-scoped deadline and query row cap for one atomic attempt. |
| [`ExecuteRowCapGuard`](#executerowcapguard) | struct | Restores `ExecBudget::raw_query_row_cap` when dropped. |
| [`AtomicInterruptPhase`](#atomicinterruptphase) | enum | Phase at which an injected atomic interrupt fires. |
| [`AtomicInterruptKind`](#atomicinterruptkind) | enum | Kind of injected atomic interrupt. |
| [`note_begin_failed`](#note_begin_failed) | function | Record that `BEGIN` failed so later statements cannot autocommit. |
| [`note_commit_failed`](#note_commit_failed) | function | Record that `COMMIT` failed so the caller cannot treat the txn as durable. |
| [`is_txn_broken`](#is_txn_broken) | function | Whether this task has a sticky begin/commit fault. |
| [`txn_broken_err`](#txn_broken_err) | function | `DbErr` for refusing statements after a failed begin/commit. |
| [`take_txn_fault`](#take_txn_fault) | function | Take and clear this task's sticky fault (after SeaORM commit/rollback). |
| [`inject_begin_failures`](#inject_begin_failures) | function | Queue `n` injected `BEGIN` failures for this task. |
| [`inject_commit_failures`](#inject_commit_failures) | function | Queue `n` injected `COMMIT` failures for this task. |
| [`inject_savepoint_release_failures`](#inject_savepoint_release_failures) | function | Queue `n` injected nested-savepoint `RELEASE` failures for this task. |
| [`inject_savepoint_rollback_failures`](#inject_savepoint_rollback_failures) | function | Queue `n` injected nested-savepoint `ROLLBACK TO` failures for this task. |
| [`consume_begin_injection`](#consume_begin_injection) | function | Consume one injected begin failure if any remain for this task. |
| [`consume_commit_injection`](#consume_commit_injection) | function | Consume one injected commit failure if any remain for this task. |
| [`consume_savepoint_release_injection`](#consume_savepoint_release_injection) | function | Consume one injected savepoint `RELEASE` failure if any remain for this task. |
| [`consume_savepoint_rollback_injection`](#consume_savepoint_rollback_injection) | function | Consume one injected savepoint `ROLLBACK TO` failure if any remain for this task. |
| [`inject_atomic_interrupt`](#inject_atomic_interrupt) | function | Queue an atomic session interrupt for this task (cancel or deadline at `phase`). |
| [`inject_atomic_interrupt_after`](#inject_atomic_interrupt_after) | function | Like `inject_atomic_interrupt`, ignoring `skip` observations of `phase` first. |
| [`consume_atomic_interrupt`](#consume_atomic_interrupt) | function | Consume an injected interrupt when the executor reaches `phase`. |
| [`with_exec_budget`](#with_exec_budget) | function | Runs `fut` with `budget` as the request-scoped execution budget. |
| [`current_exec_budget`](#current_exec_budget) | function | Request-scoped budget for the current Tokio task, if an atomic attempt armed one. |
| [`suspend_execute_row_cap`](#suspend_execute_row_cap) | function | Suspends the execute-request row cap until the guard is dropped. |
| [`record_query_rows_seen`](#record_query_rows_seen) | function | Records `seen` for `query_rows_seen` after an atomic attempt completes. |
| [`arm_exec_budget`](#arm_exec_budget) | function | Legacy no-op. Budgets are request-scoped via `with_exec_budget`. |
| [`clear_exec_budget`](#clear_exec_budget) | function | Legacy no-op. Budgets are request-scoped via `with_exec_budget`. |
| [`exec_deadline_expired`](#exec_deadline_expired) | function | True when the current task's armed deadline has elapsed. |
| [`exec_deadline_remaining_ms`](#exec_deadline_remaining_ms) | function | Remaining milliseconds until the current task's deadline (`None` if unlimited). |
| [`query_row_cap`](#query_row_cap) | function | Advertised query row cap for the current task (`None` if unlimited). |
| [`note_query_row`](#note_query_row) | function | Records one materialized query row against the current task budget. |
| [`query_rows_seen`](#query_rows_seen) | function | Rows materialized by the last completed capped query (tests). |

## Structs

### ExecBudget

```rust
pub struct ExecBudget {
    /* private fields */
}
```

Request-scoped deadline and query row cap for one atomic attempt.

Stored on the Tokio task that owns the attempt and copied onto the engine
connection after it acquires the exclusive lease, so concurrent attempts
cannot overwrite each other.

#### ExecBudget::new

```rust
pub fn new(deadline_unix_ms: Option<u64>, query_row_cap: u32) -> Arc<Self>
```

Builds a budget for one atomic attempt.

#### ExecBudget::with_cancel

```rust
pub fn with_cancel(
    deadline_unix_ms: Option<u64>,
    query_row_cap: u32,
    cancel: Option<Arc<AtomicBool>>,
) -> Arc<Self>
```

Builds a budget for one atomic attempt that also carries the session's
host cancel flag, so engine retry loops can stop when it is raised.

##### Arguments

* `deadline_unix_ms` - Guest-visible deadline (`None` = none).
* `query_row_cap` - Advertised query row cap (`0` = unlimited).
* `cancel` - Host cancel flag of the atomic session, if any.

#### ExecBudget::unlimited

```rust
pub fn unlimited() -> Arc<Self>
```

No deadline and no row cap.

#### ExecBudget::deadline_expired

```rust
pub fn deadline_expired(&self) -> bool
```

True when an armed deadline has elapsed.

#### ExecBudget::cancelled

```rust
pub fn cancelled(&self) -> bool
```

True when the owning session's host cancel flag is raised.

#### ExecBudget::remaining_ms

```rust
pub fn remaining_ms(&self) -> Option<u64>
```

Remaining milliseconds until the armed deadline (`None` if unlimited).

#### ExecBudget::cap

```rust
pub fn cap(&self) -> Option<usize>
```

Advertised query row cap (`None` if unlimited).

#### ExecBudget::set_query_row_cap

```rust
pub fn set_query_row_cap(&self, cap: usize)
```

Sets the query row cap (`0` = unlimited).

#### ExecBudget::raw_query_row_cap

```rust
pub fn raw_query_row_cap(&self) -> usize
```

Raw cap including `0` for unlimited.

#### ExecBudget::reset_rows_seen

```rust
pub fn reset_rows_seen(&self)
```

Clears the per-query row counter.

#### ExecBudget::note_row

```rust
pub fn note_row(&self) -> bool
```

Records one materialized query row; `true` when the caller should stop.

#### ExecBudget::rows_seen

```rust
pub fn rows_seen(&self) -> usize
```

Rows materialized by the current capped query.

#### ExecBudget::set_positional_columns

```rust
pub fn set_positional_columns(&self, columns: Vec<DbColumn>)
```

Records rusqlite/engine column metadata for the in-flight statement.

#### ExecBudget::take_positional_columns

```rust
pub fn take_positional_columns(&self) -> Option<Vec<DbColumn>>
```

Takes engine column metadata recorded for this request, if any.

Implements: `Debug`

### ExecuteRowCapGuard

```rust
pub struct ExecuteRowCapGuard {
    /* private fields */
}
```

Restores [`ExecBudget::raw_query_row_cap`](#execbudgetraw_query_row_cap) when dropped.

Implements: `Drop`

## Enums

### AtomicInterruptPhase

```rust
pub enum AtomicInterruptPhase {
    BeforeBegin,
    BetweenStatements,
    AroundCommit,
}
```

Phase at which an injected atomic interrupt fires.

**Variants**

- `BeforeBegin`: Before `BEGIN` / HTTP batch.
- `BetweenStatements`: Between statements during the transaction.
- `AroundCommit`: Around `COMMIT` / HTTP return (ambiguous).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### AtomicInterruptKind

```rust
pub enum AtomicInterruptKind {
    Cancel,
    Deadline,
}
```

Kind of injected atomic interrupt.

**Variants**

- `Cancel`: Session cancel.
- `Deadline`: Deadline elapsed.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### note_begin_failed

```rust
pub fn note_begin_failed(err: impl Display)
```

Record that `BEGIN` failed so later statements cannot autocommit.

### note_commit_failed

```rust
pub fn note_commit_failed(err: impl Display)
```

Record that `COMMIT` failed so the caller cannot treat the txn as durable.

### is_txn_broken

```rust
pub fn is_txn_broken() -> bool
```

Whether this task has a sticky begin/commit fault.

### txn_broken_err

```rust
pub fn txn_broken_err() -> DbErr
```

`DbErr` for refusing statements after a failed begin/commit.

### take_txn_fault

```rust
pub fn take_txn_fault() -> Option<String>
```

Take and clear this task's sticky fault (after SeaORM commit/rollback).

### inject_begin_failures

```rust
pub fn inject_begin_failures(n: u32)
```

Queue `n` injected `BEGIN` failures for this task.

### inject_commit_failures

```rust
pub fn inject_commit_failures(n: u32)
```

Queue `n` injected `COMMIT` failures for this task.

### inject_savepoint_release_failures

```rust
pub fn inject_savepoint_release_failures(n: u32)
```

Queue `n` injected nested-savepoint `RELEASE` failures for this task.

Also arms a process-wide counter so a txn worker (different Tokio task)
can observe the fault.

### inject_savepoint_rollback_failures

```rust
pub fn inject_savepoint_rollback_failures(n: u32)
```

Queue `n` injected nested-savepoint `ROLLBACK TO` failures for this task.

Also arms a process-wide counter so a txn worker (different Tokio task)
can observe the fault.

### consume_begin_injection

```rust
pub fn consume_begin_injection() -> bool
```

Consume one injected begin failure if any remain for this task.

### consume_commit_injection

```rust
pub fn consume_commit_injection() -> bool
```

Consume one injected commit failure if any remain for this task.

### consume_savepoint_release_injection

```rust
pub fn consume_savepoint_release_injection() -> bool
```

Consume one injected savepoint `RELEASE` failure if any remain for this task.

### consume_savepoint_rollback_injection

```rust
pub fn consume_savepoint_rollback_injection() -> bool
```

Consume one injected savepoint `ROLLBACK TO` failure if any remain for this task.

### inject_atomic_interrupt

```rust
pub fn inject_atomic_interrupt(phase: AtomicInterruptPhase, kind: AtomicInterruptKind)
```

Queue an atomic session interrupt for this task (cancel or deadline at `phase`).

### inject_atomic_interrupt_after

```rust
pub fn inject_atomic_interrupt_after(
    phase: AtomicInterruptPhase,
    kind: AtomicInterruptKind,
    skip: u32,
)
```

Like [`inject_atomic_interrupt`](#inject_atomic_interrupt), ignoring `skip` observations of `phase` first.

### consume_atomic_interrupt

```rust
pub fn consume_atomic_interrupt(
    phase: AtomicInterruptPhase,
) -> Option<AtomicInterruptKind>
```

Consume an injected interrupt when the executor reaches `phase`.

### with_exec_budget

```rust
pub async fn with_exec_budget<F, Fut, T>(budget: Arc<ExecBudget>, fut: F) -> T
where
    F: FnOnce() -> Fut,
    Fut: Future<Output = T>,
```

Runs `fut` with `budget` as the request-scoped execution budget.

### current_exec_budget

```rust
pub fn current_exec_budget() -> Option<Arc<ExecBudget>>
```

Request-scoped budget for the current Tokio task, if an atomic attempt armed one.

### suspend_execute_row_cap

```rust
pub fn suspend_execute_row_cap() -> Option<ExecuteRowCapGuard>
```

Suspends the execute-request row cap until the guard is dropped.

Catalog / `sqlite_master` loads are not guest result rows; they must not
fail a tight `maxResultRows` on the surrounding execute.

### record_query_rows_seen

```rust
pub fn record_query_rows_seen(seen: usize)
```

Records `seen` for [`query_rows_seen`](#query_rows_seen) after an atomic attempt completes.

### arm_exec_budget

```rust
pub fn arm_exec_budget(_deadline_unix_ms: Option<u64>, _query_row_cap: u32)
```

Legacy no-op. Budgets are request-scoped via [`with_exec_budget`](#with_exec_budget).

### clear_exec_budget

```rust
pub fn clear_exec_budget()
```

Legacy no-op. Budgets are request-scoped via [`with_exec_budget`](#with_exec_budget).

### exec_deadline_expired

```rust
pub fn exec_deadline_expired() -> bool
```

True when the current task's armed deadline has elapsed.

### exec_deadline_remaining_ms

```rust
pub fn exec_deadline_remaining_ms() -> Option<u64>
```

Remaining milliseconds until the current task's deadline (`None` if unlimited).

### query_row_cap

```rust
pub fn query_row_cap() -> Option<usize>
```

Advertised query row cap for the current task (`None` if unlimited).

### note_query_row

```rust
pub fn note_query_row() -> bool
```

Records one materialized query row against the current task budget.

### query_rows_seen

```rust
pub fn query_rows_seen() -> usize
```

Rows materialized by the last completed capped query (tests).
