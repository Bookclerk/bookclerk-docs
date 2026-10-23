**bookclerk_library > atomic_ops > atomic_status**

# Module: atomic_ops::atomic_status

## Contents

**Constants**

- [`CLAIM_INVALID`](#claim_invalid) - Claim ticket missing, expired, or already redeemed.
- [`DUPLICATE`](#duplicate) - Job admit found an equivalent pending/running row.
- [`EMPTY`](#empty) - Consume-once lookup found nothing (or the row was expired).
- [`IDEMPOTENCY_CONFLICT`](#idempotency_conflict) - Same `operationId` reused with a different request body.
- [`LAST_OWNER`](#last_owner) - Refused: would remove the last active owner.
- [`NOT_FOUND`](#not_found) - Target row does not exist.
- [`OK`](#ok) - Operation committed with the expected payload (or no payload).
- [`PASSWORD_REQUIRED`](#password_required) - Local claim login needs a password; the ticket was not consumed.
- [`QUEUE_FULL`](#queue_full) - Job admit refused because the pending+running cap was reached.

---

## bookclerk_library::atomic_ops::atomic_status::CLAIM_INVALID

*Constant*: `&str`

Claim ticket missing, expired, or already redeemed.



## bookclerk_library::atomic_ops::atomic_status::DUPLICATE

*Constant*: `&str`

Job admit found an equivalent pending/running row.



## bookclerk_library::atomic_ops::atomic_status::EMPTY

*Constant*: `&str`

Consume-once lookup found nothing (or the row was expired).



## bookclerk_library::atomic_ops::atomic_status::IDEMPOTENCY_CONFLICT

*Constant*: `&str`

Same `operationId` reused with a different request body.



## bookclerk_library::atomic_ops::atomic_status::LAST_OWNER

*Constant*: `&str`

Refused: would remove the last active owner.



## bookclerk_library::atomic_ops::atomic_status::NOT_FOUND

*Constant*: `&str`

Target row does not exist.



## bookclerk_library::atomic_ops::atomic_status::OK

*Constant*: `&str`

Operation committed with the expected payload (or no payload).



## bookclerk_library::atomic_ops::atomic_status::PASSWORD_REQUIRED

*Constant*: `&str`

Local claim login needs a password; the ticket was not consumed.



## bookclerk_library::atomic_ops::atomic_status::QUEUE_FULL

*Constant*: `&str`

Job admit refused because the pending+running cap was reached.



