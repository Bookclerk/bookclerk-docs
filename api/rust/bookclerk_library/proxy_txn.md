# bookclerk_library::proxy_txn

Fail-closed transaction injects re-exported from [`bookclerk_db_exec`](../bookclerk_db_exec/index.md).

## Re-exports

- [`bookclerk_db_exec::arm_exec_budget`](../bookclerk_db_exec/proxy_txn.md#arm_exec_budget)
- [`bookclerk_db_exec::clear_exec_budget`](../bookclerk_db_exec/proxy_txn.md#clear_exec_budget)
- [`bookclerk_db_exec::consume_atomic_interrupt`](../bookclerk_db_exec/proxy_txn.md#consume_atomic_interrupt)
- [`bookclerk_db_exec::consume_begin_injection`](../bookclerk_db_exec/proxy_txn.md#consume_begin_injection)
- [`bookclerk_db_exec::consume_commit_injection`](../bookclerk_db_exec/proxy_txn.md#consume_commit_injection)
- [`bookclerk_db_exec::current_exec_budget`](../bookclerk_db_exec/proxy_txn.md#current_exec_budget)
- [`bookclerk_db_exec::exec_deadline_expired`](../bookclerk_db_exec/proxy_txn.md#exec_deadline_expired)
- [`bookclerk_db_exec::exec_deadline_remaining_ms`](../bookclerk_db_exec/proxy_txn.md#exec_deadline_remaining_ms)
- [`bookclerk_db_exec::inject_atomic_interrupt`](../bookclerk_db_exec/proxy_txn.md#inject_atomic_interrupt)
- [`bookclerk_db_exec::inject_atomic_interrupt_after`](../bookclerk_db_exec/proxy_txn.md#inject_atomic_interrupt_after)
- [`bookclerk_db_exec::inject_begin_failures`](../bookclerk_db_exec/proxy_txn.md#inject_begin_failures)
- [`bookclerk_db_exec::inject_commit_failures`](../bookclerk_db_exec/proxy_txn.md#inject_commit_failures)
- [`bookclerk_db_exec::inject_savepoint_release_failures`](../bookclerk_db_exec/proxy_txn.md#inject_savepoint_release_failures)
- [`bookclerk_db_exec::inject_savepoint_rollback_failures`](../bookclerk_db_exec/proxy_txn.md#inject_savepoint_rollback_failures)
- [`bookclerk_db_exec::is_txn_broken`](../bookclerk_db_exec/proxy_txn.md#is_txn_broken)
- [`bookclerk_db_exec::note_begin_failed`](../bookclerk_db_exec/proxy_txn.md#note_begin_failed)
- [`bookclerk_db_exec::note_commit_failed`](../bookclerk_db_exec/proxy_txn.md#note_commit_failed)
- [`bookclerk_db_exec::note_query_row`](../bookclerk_db_exec/proxy_txn.md#note_query_row)
- [`bookclerk_db_exec::query_row_cap`](../bookclerk_db_exec/proxy_txn.md#query_row_cap)
- [`bookclerk_db_exec::query_rows_seen`](../bookclerk_db_exec/proxy_txn.md#query_rows_seen)
- [`bookclerk_db_exec::take_txn_fault`](../bookclerk_db_exec/proxy_txn.md#take_txn_fault)
- [`bookclerk_db_exec::txn_broken_err`](../bookclerk_db_exec/proxy_txn.md#txn_broken_err)
- [`bookclerk_db_exec::with_exec_budget`](../bookclerk_db_exec/proxy_txn.md#with_exec_budget)
- [`bookclerk_db_exec::AtomicInterruptKind`](../bookclerk_db_exec/proxy_txn.md#atomicinterruptkind)
- [`bookclerk_db_exec::AtomicInterruptPhase`](../bookclerk_db_exec/proxy_txn.md#atomicinterruptphase)
- [`bookclerk_db_exec::ExecBudget`](../bookclerk_db_exec/proxy_txn.md#execbudget)
