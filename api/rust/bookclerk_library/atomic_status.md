# bookclerk_library::atomic_status

Application status strings produced by host plan interpretation.

| Item | Kind | Summary |
| --- | --- | --- |
| [`OK`](#ok) | constant | Operation committed with the expected payload (or no payload). |
| [`EMPTY`](#empty) | constant | Consume-once lookup found nothing (or the row was expired). |
| [`NOT_FOUND`](#not_found) | constant | Target row does not exist. |
| [`LAST_OWNER`](#last_owner) | constant | Refused: would remove the last active owner. |
| [`CLAIM_INVALID`](#claim_invalid) | constant | Claim ticket missing, expired, or already redeemed. |
| [`PASSWORD_REQUIRED`](#password_required) | constant | Local claim login needs a password; the ticket was not consumed. |
| [`IDEMPOTENCY_CONFLICT`](#idempotency_conflict) | constant | Same `operationId` reused with a different request body. |
| [`DUPLICATE`](#duplicate) | constant | Job admit found an equivalent pending/running row. |
| [`QUEUE_FULL`](#queue_full) | constant | Job admit refused because the pending+running cap was reached. |

## Constants

### OK

```rust
pub const OK: &str = "ok";
```

Operation committed with the expected payload (or no payload).

### EMPTY

```rust
pub const EMPTY: &str = "empty";
```

Consume-once lookup found nothing (or the row was expired).

### NOT_FOUND

```rust
pub const NOT_FOUND: &str = "notFound";
```

Target row does not exist.

### LAST_OWNER

```rust
pub const LAST_OWNER: &str = "lastOwner";
```

Refused: would remove the last active owner.

### CLAIM_INVALID

```rust
pub const CLAIM_INVALID: &str = "claimInvalid";
```

Claim ticket missing, expired, or already redeemed.

### PASSWORD_REQUIRED

```rust
pub const PASSWORD_REQUIRED: &str = "passwordRequired";
```

Local claim login needs a password; the ticket was not consumed.

### IDEMPOTENCY_CONFLICT

```rust
pub const IDEMPOTENCY_CONFLICT: &str = "idempotencyConflict";
```

Same `operationId` reused with a different request body.

### DUPLICATE

```rust
pub const DUPLICATE: &str = "duplicate";
```

Job admit found an equivalent pending/running row.

### QUEUE_FULL

```rust
pub const QUEUE_FULL: &str = "queueFull";
```

Job admit refused because the pending+running cap was reached.
