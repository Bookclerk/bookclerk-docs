**bookclerk_library > atomic_ops**

# Module: atomic_ops

## Contents

**Modules**

- [`atomic_status`](#atomic_status) - Application status strings produced by host plan interpretation.

**Structs**

- [`DbAtomicResult`](#dbatomicresult) - Host-interpreted result of a named atomic operation.

**Enums**

- [`DbAtomicParams`](#dbatomicparams) - Named atomic library operation compiled by the host into a generic plan.

---

## bookclerk_library::atomic_ops::DbAtomicParams

*Enum*

Named atomic library operation compiled by the host into a generic plan.

**Variants:**
- `DeleteUser{ user_id: i64 }` - Delete a first-party user and personal data (last-owner guarded).
- `SetUserStatus{ user_id: i64, status: String }` - Set `users.status` (`active` / `disabled`; last-owner guarded).
- `SetUserPasswordHash{ user_id: i64, password_hash: Option<String> }` - Set or clear the Argon2id password hash and bump `security_version`.
- `SetUserRole{ user_id: i64, role: String }` - Set `users.role` (last-owner guarded on demotion).
- `RedeemClaimTicket{ token_hash: String, session_hash: String, expires_at: String, user_agent: Option<String>, device_type: Option<String>, client_label: Option<String>, new_password_hash: Option<String>, password_fingerprint: Option<String> }` - Consume a claim ticket, optionally set a first password, mint a session.
- `TakeOidcRpState{ state_hash: String }` - Consume a one-time OIDC RP state (`DELETE` + expiry check).
- `TakeWebauthnChallenge{ challenge_id: String, kind: String }` - Consume a one-time WebAuthn challenge (`DELETE` + expiry check).
- `EnqueueJob{ kind: String, payload_json: String, priority: i64, max_attempts: i64, max_pending: i64, run_after: Option<String> }` - Admit a durable job (dedup + pending cap + insert).
- `ClaimNextJob{ resource_class: String, owner: String, lease_secs: i64 }` - Claim the next ready job in `resource_class` for `owner`.
- `ReserveJobTemp{ job_id: String, path: String, reserved_bytes: i64, quota_bytes: i64 }` - Reserve scratch-quota bytes for one job path.
- `ConfirmTotpEnrollment{ user_id: i64, format: String, ciphertext: String, cipher_algorithm: Option<String>, cipher_nonce: Option<String>, kdf_algorithm: Option<String>, kdf_salt: Option<String>, kdf_m_cost: Option<i64>, kdf_t_cost: Option<i64>, kdf_p_cost: Option<i64>, created_at: String }` - Promote a sealed TOTP secret to `primary` and set `users.totp_enabled`.
- `DisableUserTotp{ user_id: i64 }` - Delete TOTP secrets and clear `users.totp_enabled`.
- `PublishDomainEvent{ id: String, event_type: String, schema_version: i64, account_id: String, source: String, correlation_id: String, causation_id: String, dedup_key: String, payload: String, ordering_key: String, queue_name: String, content_type: String, deliver_after: String }` - Persist a domain event in the outbox (dedup on eventType+dedupKey).
- `SetAcquireStatus{ book_uuid: String, status: String, storage_key: Option<String>, error_message: Option<String>, event_id: String, event_type: String, schema_version: i64, event_account_id: String, source: String, correlation_id: String, causation_id: String, dedup_key: String, payload: String, ordering_key: String }` - Update a book row and optionally publish `book_acquired` in the same batch.
- `DispatchEventDeliveries{ event_id: String, subscribers_json: String, mark_dispatched: bool }` - Create per-subscriber deliveries and mark the event dispatched.
- `ClaimNextEventDelivery{ owner: String, lease_secs: i64, plugin_ids_json: String, max_in_flight: i64 }` - Claim the next ready event delivery for `owner`.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DbAtomicParams`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbAtomicParams) -> bool`



## bookclerk_library::atomic_ops::DbAtomicResult

*Struct*

Host-interpreted result of a named atomic operation.

**Fields:**
- `status: String` - Application outcome (`ok`, `empty`, `notFound`, `lastOwner`, …).
- `payload: Option<serde_json::Value>` - Op-specific record when `status` is `ok`.
- `operation_id: String` - Echo of the request `operationId`.
- `replayed: bool` - True when this result was loaded from a durable receipt.
- `receipt_created_at: Option<String>` - RFC 3339 timestamp when the receipt was first written.
- `timing: Option<bookclerk_plugin_abi::DbTiming>` - Handler/engine timing. Not hashed for idempotency.

**Methods:**

- `fn ok(payload: JsonValue) -> Self` - Successful operation with a JSON payload.
- `fn ok_unit() -> Self` - Successful operation with no payload (`deleteUser`).
- `fn with_status(status: &str) -> Self` - Application failure or empty consume-once result (no payload).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbAtomicResult) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DbAtomicResult`



## Module: atomic_status

Application status strings produced by host plan interpretation.



