**bookclerk_library > db_atomic**

# Module: db_atomic

## Contents

**Functions**

- [`db_atomic_operation_id`](#db_atomic_operation_id) - Caller-owned idempotency key for an atomic attempt.
- [`db_atomic_request_hash`](#db_atomic_request_hash) - SHA-256 hex digest of the idempotency-relevant fields of `op`.
- [`execute_db_atomic`](#execute_db_atomic) - Runs a compiled named atomic as one native SQL transaction.
- [`execute_named_atomic`](#execute_named_atomic) - Compiles `params` and runs the plan as one native SQL transaction.

---

## bookclerk_library::db_atomic::db_atomic_operation_id

*Function*

Caller-owned idempotency key for an atomic attempt.

Consume-once keys are derived from the operation so an HTTP/RPC retry
resumes the same receipt. Claim redeem includes the session hash; callers
must derive that session from the ticket plus browser nonce (see
[`crate::derive_claim_session_token`](token_hash.md#bookclerk_librarytoken_hashderive_claim_session_token)) so a new HTTP request after a lost
reply reuses this id. Other ops mint a UUID for this attempt.

```rust
fn db_atomic_operation_id(op: &crate::atomic_ops::DbAtomicParams) -> String
```



## bookclerk_library::db_atomic::db_atomic_request_hash

*Function*

SHA-256 hex digest of the idempotency-relevant fields of `op`.

Claim redeem omits `expires_at`, client metadata, and the randomized Argon2
`new_password_hash`. Idempotency uses `password_fingerprint` (HMAC of the
plaintext password) so a retry with a fresh Argon2 salt still matches.

# Errors

Returns an error when the operation fails.

```rust
fn db_atomic_request_hash(op: &crate::atomic_ops::DbAtomicParams) -> crate::error::Result<String>
```



## bookclerk_library::db_atomic::execute_db_atomic

*Function*

Runs a compiled named atomic as one native SQL transaction.

# Errors

Returns [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror) for engine failures. Application outcomes
(`lastOwner`, `empty`, …) are returned as [`DbAtomicResult`](atomic_ops.md#bookclerk_libraryatomic_opsdbatomicresult), not errors.

```rust
fn execute_db_atomic(db: &sea_orm::DatabaseConnection, compiled: crate::sql_plan::CompiledAtomic) -> crate::error::Result<crate::atomic_ops::DbAtomicResult>
```



## bookclerk_library::db_atomic::execute_named_atomic

*Function*

Compiles `params` and runs the plan as one native SQL transaction.

# Errors

Returns [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror) for engine failures or invalid named operations.

```rust
fn execute_named_atomic(db: &sea_orm::DatabaseConnection, operation_id: &str, params: &crate::atomic_ops::DbAtomicParams) -> crate::error::Result<crate::atomic_ops::DbAtomicResult>
```



