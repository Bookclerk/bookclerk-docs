# bookclerk_db_exec::guest_receipt

Guest-typed durable receipt finalize (same-transaction payload capture).

Host wrap lives in `bookclerk-library`; adapters run
[`guest_receipt_finalize_stmts`](#guest_receipt_finalize_stmts) in the same atomic transaction before COMMIT.

| Item | Kind | Summary |
| --- | --- | --- |
| [`is_guest_receipt_result_lost`](#is_guest_receipt_result_lost) | function | True when `err` is `GUEST_RECEIPT_RESULT_LOST`. |
| [`is_guest_receipt_stub_insert`](#is_guest_receipt_stub_insert) | function | True when `sql` is the guest-receipt claim stub (`INSERT … WHERE NOT EXISTS`). |
| [`strip_guest_receipt_write_gate`](#strip_guest_receipt_write_gate) | function | Removes the **host-appended** `GUEST_RECEIPT_WRITE_GATE` predicate. |
| [`guest_receipt_applied_stmt`](#guest_receipt_applied_stmt) | function | Host UPDATE that marks a claimed stub as applied after ungated guest SQL. |
| [`prior_receipt_exists`](#prior_receipt_exists) | function | True when the prior-receipt SELECT returned a row. |
| [`prior_receipt_is_claimed`](#prior_receipt_is_claimed) | function | True when `stmt` is an in-flight `GUEST_RECEIPT_STATUS_CLAIMED` receipt. |
| [`receipt_hash_from`](#receipt_hash_from) | function | Reads `request_hash` from a prior-receipt select result, if any row exists. |
| [`prior_receipt_should_resume_guest`](#prior_receipt_should_resume_guest) | function | True when a claimed prior receipt belongs to `guest_hash` (resume ungated DDL). |
| [`should_skip_remaining_guest_work`](#should_skip_remaining_guest_work) | function | True when wrap prefix results show a completed prior receipt (skip guest SQL). |
| [`pad_skipped_guest_results`](#pad_skipped_guest_results) | function | Pads skipped guest + stub results so the wrap shape stays intact. |
| [`guest_receipt_finalize_stmts`](#guest_receipt_finalize_stmts) | function | Builds finalize statements to persist a guest reply before COMMIT. |
| [`GUEST_TYPED_REPLAY_PAYLOAD_MAX_BYTES`](#guest_typed_replay_payload_max_bytes) | constant | Maximum UTF-8 bytes stored in `bookclerk_receipts.payload` for guest replay. |
| [`GUEST_RECEIPT_RESULT_LOST`](#guest_receipt_result_lost) | constant | Error text when a receipt committed but the caller-visible payload was lost. |
| [`GUEST_RECEIPT_STATUS_CLAIMED`](#guest_receipt_status_claimed) | constant | In-flight guest-receipt claim (D1 commits the stub INSERT before DDL). |
| [`GUEST_RECEIPT_STATUS_OK`](#guest_receipt_status_ok) | constant | Finalized guest-receipt row; payload is the durable replay body. |
| [`GUEST_RECEIPT_STATUS_APPLIED`](#guest_receipt_status_applied) | constant | Claimed-owner guest SQL finished on D1 (prune + ungated guest in one HTTP). |
| [`GUEST_RECEIPT_WRITE_GATE`](#guest_receipt_write_gate) | constant | Write-predicate spliced onto guest DML by the host receipt wrap. |
| [`GUEST_RECEIPT_WRAP_PREFIX`](#guest_receipt_wrap_prefix) | constant | Prune + prior-select prefix ahead of the guest statements. |
| [`GUEST_RECEIPT_STUB_SUFFIX`](#guest_receipt_stub_suffix) | constant | Same-batch receipt stub INSERT after gated guest statements. |

## Functions

### is_guest_receipt_result_lost

```rust
pub fn is_guest_receipt_result_lost(err: &DbErr) -> bool
```

True when `err` is [`GUEST_RECEIPT_RESULT_LOST`](#guest_receipt_result_lost).

### is_guest_receipt_stub_insert

```rust
pub fn is_guest_receipt_stub_insert(sql: &str) -> bool
```

True when `sql` is the guest-receipt claim stub (`INSERT … WHERE NOT EXISTS`).

D1 commits each HTTP batch immediately, so the adapter sends this statement
first and only runs ungated guest DDL after `rows_affected = 1`.

### strip_guest_receipt_write_gate

```rust
pub fn strip_guest_receipt_write_gate(sql: &str) -> String
```

Removes the **host-appended** [`GUEST_RECEIPT_WRITE_GATE`](#guest_receipt_write_gate) predicate.

The wrap splices the gate into a code span (before `RETURNING`). A plugin
string or comment may contain the same text; only the last code-span match
is removed. Returns `sql` unchanged when that gate is absent. The caller
must drop the trailing `operation_id` bind the wrap appended for the gate.

### guest_receipt_applied_stmt

```rust
pub fn guest_receipt_applied_stmt(operation_id: &str) -> TypedDbStatement
```

Host UPDATE that marks a claimed stub as applied after ungated guest SQL.

### prior_receipt_exists

```rust
pub fn prior_receipt_exists(stmt: &StatementResult) -> bool
```

True when the prior-receipt SELECT returned a row.

### prior_receipt_is_claimed

```rust
pub fn prior_receipt_is_claimed(stmt: &StatementResult) -> bool
```

True when `stmt` is an in-flight [`GUEST_RECEIPT_STATUS_CLAIMED`](#guest_receipt_status_claimed) receipt.

### receipt_hash_from

```rust
pub fn receipt_hash_from(stmt: &StatementResult) -> Option<String>
```

Reads `request_hash` from a prior-receipt select result, if any row exists.

### prior_receipt_should_resume_guest

```rust
pub fn prior_receipt_should_resume_guest(stmt: &StatementResult, guest_hash: &str) -> bool
```

True when a claimed prior receipt belongs to `guest_hash` (resume ungated DDL).

### should_skip_remaining_guest_work

```rust
pub fn should_skip_remaining_guest_work(
    results: &[StatementResult],
    total: usize,
    guest_hash: &str,
) -> bool
```

True when wrap prefix results show a completed prior receipt (skip guest SQL).

An in-flight [`GUEST_RECEIPT_STATUS_CLAIMED`](#guest_receipt_status_claimed) row with the same hash must
**not** skip remaining work: D1 commits the stub INSERT before ungated
DDL, so a crash between those HTTP batches has to re-run idempotent DDL
and then finalize. A claimed row with a **different** hash still skips
(unwrap reports conflict).

### pad_skipped_guest_results

```rust
pub fn pad_skipped_guest_results(results: &mut Vec<StatementResult>, total: usize)
```

Pads skipped guest + stub results so the wrap shape stays intact.

### guest_receipt_finalize_stmts

```rust
pub fn guest_receipt_finalize_stmts(
    partial: &ExecuteReply,
    guest_len: usize,
    guest_hash: &str,
) -> Result<Vec<TypedDbStatement>, DbErr>
```

Builds finalize statements to persist a guest reply before COMMIT.

Returns an empty list when the prior receipt row already carries payload.

#### Errors

Returns [`DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) when the envelope is malformed or the payload exceeds
[`GUEST_TYPED_REPLAY_PAYLOAD_MAX_BYTES`](#guest_typed_replay_payload_max_bytes).

## Constants

### GUEST_TYPED_REPLAY_PAYLOAD_MAX_BYTES

```rust
pub const GUEST_TYPED_REPLAY_PAYLOAD_MAX_BYTES: usize = 65536;
```

Maximum UTF-8 bytes stored in `bookclerk_receipts.payload` for guest replay.

### GUEST_RECEIPT_RESULT_LOST

```rust
pub const GUEST_RECEIPT_RESULT_LOST: &str = "guest receipt committed without payload; original response lost";
```

Error text when a receipt committed but the caller-visible payload was lost.

### GUEST_RECEIPT_STATUS_CLAIMED

```rust
pub const GUEST_RECEIPT_STATUS_CLAIMED: &str = "claimed";
```

In-flight guest-receipt claim (D1 commits the stub INSERT before DDL).

### GUEST_RECEIPT_STATUS_OK

```rust
pub const GUEST_RECEIPT_STATUS_OK: &str = "ok";
```

Finalized guest-receipt row; payload is the durable replay body.

### GUEST_RECEIPT_STATUS_APPLIED

```rust
pub const GUEST_RECEIPT_STATUS_APPLIED: &str = "applied";
```

Claimed-owner guest SQL finished on D1 (prune + ungated guest in one HTTP).

Resume remaining guest work only for [`GUEST_RECEIPT_STATUS_CLAIMED`](#guest_receipt_status_claimed). An
`applied` row means DML already ran; skip guest SQL and finalize (or
surface result-lost when payload is still empty).

### GUEST_RECEIPT_WRITE_GATE

```rust
pub const GUEST_RECEIPT_WRITE_GATE: &str = "NOT EXISTS (SELECT 1 FROM bookclerk_receipts WHERE operation_id = ?)";
```

Write-predicate spliced onto guest DML by the host receipt wrap.

D1 claimed-owner batches strip this after the stub INSERT commits, because
the claim row would otherwise make the predicate false on first execution.

### GUEST_RECEIPT_WRAP_PREFIX

```rust
pub const GUEST_RECEIPT_WRAP_PREFIX: usize = 2;
```

Prune + prior-select prefix ahead of the guest statements.

### GUEST_RECEIPT_STUB_SUFFIX

```rust
pub const GUEST_RECEIPT_STUB_SUFFIX: usize = 1;
```

Same-batch receipt stub INSERT after gated guest statements.
