**bookclerk_library > atomic_txn**

# Module: atomic_txn

## Contents

**Traits**

- [`AtomicTxnBackend`](#atomictxnbackend) - Backend that runs [`crate::LibraryStore`](store.md#bookclerk_librarystorelibrarystore) named security operations as a
- [`TypedAtomicExec`](#typedatomicexec) - Runs a host-authorized typed [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest) as one guest `executeAtomic`.

---

## bookclerk_library::atomic_txn::AtomicTxnBackend

*Trait*

Backend that runs [`crate::LibraryStore`](store.md#bookclerk_librarystorelibrarystore) named security operations as a
single guest atomic batch.

Implementations must preserve the same fail-closed semantics as the SeaORM
path: last-owner refusals mutate nothing; a failed claim redeem must not
consume the ticket or write a password/session; consume-once OIDC/WebAuthn
rows must not be observable by a concurrent caller; TOTP enroll/disable must
keep `encrypted_secrets` and `users.totp_enabled` in the same transaction.

**Methods:**

- `delete_user`: Delete a first-party user and personal data (last-owner guarded).
- `set_user_status`: Set user status (`active` / `disabled`; last-owner guarded).
- `set_user_password_hash`: Set or clear the Argon2id password hash and bump `security_version`.
- `set_user_role`: Set user role (last-owner guarded on demotion).
- `redeem_claim_ticket_to_session`: Consume a claim ticket, optionally set a first password, mint a session.
- `take_oidc_rp_state`: Consume a one-time OIDC RP state. `Ok(None)` if missing or expired.
- `take_webauthn_challenge`: Consume a one-time WebAuthn challenge. `Ok(None)` if missing or expired.
- `enqueue_job`: Admit a durable job in one atomic transaction.
- `claim_next_job`: Claim the next ready job; `operation_id` makes a lost response replay-safe.
- `reserve_job_temp_path`: Reserve scratch-quota bytes for `path` on `job_id`.
- `confirm_totp_enrollment`: Promote a sealed TOTP secret to `primary` and set `totp_enabled`.
- `disable_user_totp`: Delete TOTP secrets and clear `totp_enabled`.
- `publish_domain_event`: Persist a domain event in the outbox.
- `set_acquire_status`: Update acquire status and, when acquired, publish `book_acquired` in the same transaction.
- `dispatch_event_deliveries`: Create deliveries for `subscribers`. `mark_dispatched` finishes the parent event.
- `claim_event_delivery`: CAS-claim one pending delivery after the host has filtered eligibility.



## bookclerk_library::atomic_txn::TypedAtomicExec

*Trait*

Runs a host-authorized typed [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest) as one guest `executeAtomic`.

First-party hosts attach this alongside [`AtomicTxnBackend`](atomic_txn.md#bookclerk_libraryatomic_txnatomictxnbackend) so granted job
sessions do not round-trip through the SeaORM proxy (`BEGIN` + nested
`query`/`execute`). In-process tests leave it unset and run the same
request on the local connection.

**Methods:**

- `execute_typed`: Executes `req` as one typed atomic batch.



