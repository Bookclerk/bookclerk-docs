**bookclerk_db_exec > guest_receipt**

# Module: guest_receipt

## Contents

**Functions**

- [`guest_receipt_applied_stmt`](#guest_receipt_applied_stmt) - Host UPDATE that marks a claimed stub as applied after ungated guest SQL.
- [`guest_receipt_finalize_stmts`](#guest_receipt_finalize_stmts) - Builds finalize statements to persist a guest reply before COMMIT.
- [`is_guest_receipt_result_lost`](#is_guest_receipt_result_lost) - True when `err` is [`GUEST_RECEIPT_RESULT_LOST`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_result_lost).
- [`is_guest_receipt_stub_insert`](#is_guest_receipt_stub_insert) - True when `sql` is the guest-receipt claim stub (`INSERT … WHERE NOT EXISTS`).
- [`pad_skipped_guest_results`](#pad_skipped_guest_results) - Pads skipped guest + stub results so the wrap shape stays intact.
- [`prior_receipt_exists`](#prior_receipt_exists) - True when the prior-receipt SELECT returned a row.
- [`prior_receipt_is_claimed`](#prior_receipt_is_claimed) - True when `stmt` is an in-flight [`GUEST_RECEIPT_STATUS_CLAIMED`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_status_claimed) receipt.
- [`prior_receipt_should_resume_guest`](#prior_receipt_should_resume_guest) - True when a claimed prior receipt belongs to `guest_hash` (resume ungated DDL).
- [`receipt_hash_from`](#receipt_hash_from) - Reads `request_hash` from a prior-receipt select result, if any row exists.
- [`should_skip_remaining_guest_work`](#should_skip_remaining_guest_work) - True when wrap prefix results show a completed prior receipt (skip guest SQL).
- [`strip_guest_receipt_write_gate`](#strip_guest_receipt_write_gate) - Removes the **host-appended** [`GUEST_RECEIPT_WRITE_GATE`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_write_gate) predicate.

**Constants**

- [`GUEST_RECEIPT_RESULT_LOST`](#guest_receipt_result_lost) - Error text when a receipt committed but the caller-visible payload was lost.
- [`GUEST_RECEIPT_STATUS_APPLIED`](#guest_receipt_status_applied) - Claimed-owner guest SQL finished on D1 (prune + ungated guest in one HTTP).
- [`GUEST_RECEIPT_STATUS_CLAIMED`](#guest_receipt_status_claimed) - In-flight guest-receipt claim (D1 commits the stub INSERT before DDL).
- [`GUEST_RECEIPT_STATUS_OK`](#guest_receipt_status_ok) - Finalized guest-receipt row; payload is the durable replay body.
- [`GUEST_RECEIPT_STUB_SUFFIX`](#guest_receipt_stub_suffix) - Same-batch receipt stub INSERT after gated guest statements.
- [`GUEST_RECEIPT_WRAP_PREFIX`](#guest_receipt_wrap_prefix) - Prune + prior-select prefix ahead of the guest statements.
- [`GUEST_RECEIPT_WRITE_GATE`](#guest_receipt_write_gate) - Write-predicate spliced onto guest DML by the host receipt wrap.
- [`GUEST_TYPED_REPLAY_PAYLOAD_MAX_BYTES`](#guest_typed_replay_payload_max_bytes) - Maximum UTF-8 bytes stored in `bookclerk_receipts.payload` for guest replay.

---

## bookclerk_db_exec::guest_receipt::GUEST_RECEIPT_RESULT_LOST

*Constant*: `&str`

Error text when a receipt committed but the caller-visible payload was lost.



## bookclerk_db_exec::guest_receipt::GUEST_RECEIPT_STATUS_APPLIED

*Constant*: `&str`

Claimed-owner guest SQL finished on D1 (prune + ungated guest in one HTTP).

Resume remaining guest work only for [`GUEST_RECEIPT_STATUS_CLAIMED`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_status_claimed). An
`applied` row means DML already ran; skip guest SQL and finalize (or
surface result-lost when payload is still empty).



## bookclerk_db_exec::guest_receipt::GUEST_RECEIPT_STATUS_CLAIMED

*Constant*: `&str`

In-flight guest-receipt claim (D1 commits the stub INSERT before DDL).



## bookclerk_db_exec::guest_receipt::GUEST_RECEIPT_STATUS_OK

*Constant*: `&str`

Finalized guest-receipt row; payload is the durable replay body.



## bookclerk_db_exec::guest_receipt::GUEST_RECEIPT_STUB_SUFFIX

*Constant*: `usize`

Same-batch receipt stub INSERT after gated guest statements.



## bookclerk_db_exec::guest_receipt::GUEST_RECEIPT_WRAP_PREFIX

*Constant*: `usize`

Prune + prior-select prefix ahead of the guest statements.



## bookclerk_db_exec::guest_receipt::GUEST_RECEIPT_WRITE_GATE

*Constant*: `&str`

Write-predicate spliced onto guest DML by the host receipt wrap.

D1 claimed-owner batches strip this after the stub INSERT commits, because
the claim row would otherwise make the predicate false on first execution.



## bookclerk_db_exec::guest_receipt::GUEST_TYPED_REPLAY_PAYLOAD_MAX_BYTES

*Constant*: `usize`

Maximum UTF-8 bytes stored in `bookclerk_receipts.payload` for guest replay.



## bookclerk_db_exec::guest_receipt::guest_receipt_applied_stmt

*Function*

Host UPDATE that marks a claimed stub as applied after ungated guest SQL.

```rust
fn guest_receipt_applied_stmt(operation_id: &str) -> bookclerk_plugin_abi::TypedDbStatement
```



## bookclerk_db_exec::guest_receipt::guest_receipt_finalize_stmts

*Function*

Builds finalize statements to persist a guest reply before COMMIT.

Returns an empty list when the prior receipt row already carries payload.

# Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the envelope is malformed or the payload exceeds
[`GUEST_TYPED_REPLAY_PAYLOAD_MAX_BYTES`](guest_receipt.md#bookclerk_db_execguest_receiptguest_typed_replay_payload_max_bytes).

```rust
fn guest_receipt_finalize_stmts(partial: &bookclerk_plugin_abi::ExecuteReply, guest_len: usize, guest_hash: &str) -> Result<Vec<bookclerk_plugin_abi::TypedDbStatement>, sea_orm::DbErr>
```



## bookclerk_db_exec::guest_receipt::is_guest_receipt_result_lost

*Function*

True when `err` is [`GUEST_RECEIPT_RESULT_LOST`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_result_lost).

```rust
fn is_guest_receipt_result_lost(err: &sea_orm::DbErr) -> bool
```



## bookclerk_db_exec::guest_receipt::is_guest_receipt_stub_insert

*Function*

True when `sql` is the guest-receipt claim stub (`INSERT … WHERE NOT EXISTS`).

D1 commits each HTTP batch immediately, so the adapter sends this statement
first and only runs ungated guest DDL after `rows_affected = 1`.

```rust
fn is_guest_receipt_stub_insert(sql: &str) -> bool
```



## bookclerk_db_exec::guest_receipt::pad_skipped_guest_results

*Function*

Pads skipped guest + stub results so the wrap shape stays intact.

```rust
fn pad_skipped_guest_results(results: & mut Vec<bookclerk_plugin_abi::StatementResult>, total: usize)
```



## bookclerk_db_exec::guest_receipt::prior_receipt_exists

*Function*

True when the prior-receipt SELECT returned a row.

```rust
fn prior_receipt_exists(stmt: &bookclerk_plugin_abi::StatementResult) -> bool
```



## bookclerk_db_exec::guest_receipt::prior_receipt_is_claimed

*Function*

True when `stmt` is an in-flight [`GUEST_RECEIPT_STATUS_CLAIMED`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_status_claimed) receipt.

```rust
fn prior_receipt_is_claimed(stmt: &bookclerk_plugin_abi::StatementResult) -> bool
```



## bookclerk_db_exec::guest_receipt::prior_receipt_should_resume_guest

*Function*

True when a claimed prior receipt belongs to `guest_hash` (resume ungated DDL).

```rust
fn prior_receipt_should_resume_guest(stmt: &bookclerk_plugin_abi::StatementResult, guest_hash: &str) -> bool
```



## bookclerk_db_exec::guest_receipt::receipt_hash_from

*Function*

Reads `request_hash` from a prior-receipt select result, if any row exists.

```rust
fn receipt_hash_from(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::guest_receipt::should_skip_remaining_guest_work

*Function*

True when wrap prefix results show a completed prior receipt (skip guest SQL).

An in-flight [`GUEST_RECEIPT_STATUS_CLAIMED`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_status_claimed) row with the same hash must
**not** skip remaining work: D1 commits the stub INSERT before ungated
DDL, so a crash between those HTTP batches has to re-run idempotent DDL
and then finalize. A claimed row with a **different** hash still skips
(unwrap reports conflict).

```rust
fn should_skip_remaining_guest_work(results: &[bookclerk_plugin_abi::StatementResult], total: usize, guest_hash: &str) -> bool
```



## bookclerk_db_exec::guest_receipt::strip_guest_receipt_write_gate

*Function*

Removes the **host-appended** [`GUEST_RECEIPT_WRITE_GATE`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_write_gate) predicate.

The wrap splices the gate into a code span (before `RETURNING`). A plugin
string or comment may contain the same text; only the last code-span match
is removed. Returns `sql` unchanged when that gate is absent. The caller
must drop the trailing `operation_id` bind the wrap appended for the gate.

```rust
fn strip_guest_receipt_write_gate(sql: &str) -> String
```



