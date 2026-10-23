**bookclerk_library > in_process_atomic**

# Module: in_process_atomic

## Contents

**Structs**

- [`InProcessSqliteAtomic`](#inprocesssqliteatomic) - Compiles named ops and runs them on a local SeaORM SQLite connection.

---

## bookclerk_library::in_process_atomic::InProcessSqliteAtomic

*Struct*

Compiles named ops and runs them on a local SeaORM SQLite connection.

**Fields:**
- `db: sea_orm::DatabaseConnection` - Open library connection (already migrated).

**Trait Implementations:**

- **AtomicTxnBackend**
  - `fn delete_user(self: &'life0 Self, _id: i64) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn set_user_status(self: &'life0 Self, _id: i64, _status: UserStatus) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn set_user_password_hash(self: &'life0 Self, _id: i64, _password_hash: Option<&'life1 str>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn set_user_role(self: &'life0 Self, _id: i64, _role: UserRole) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn redeem_claim_ticket_to_session(self: &'life0 Self, _token_hash: &'life1 str, _session_hash: &'life2 str, _expires_at: chrono::DateTime<chrono::Utc>, _client: Option<&'life3 SessionClientInfo>, _new_password_hash: Option<&'life4 str>, _password_fingerprint: Option<&'life5 str>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn take_oidc_rp_state(self: &'life0 Self, _state_hash: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn take_webauthn_challenge(self: &'life0 Self, _challenge_id: &'life1 str, _kind: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn enqueue_job(self: &'life0 Self, _spec: EnqueueJobSpec) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn claim_next_job(self: &'life0 Self, _resource_class: JobResourceClass, _owner: &'life1 str, _lease_secs: u64, _operation_id: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn reserve_job_temp_path(self: &'life0 Self, _job_id: &'life1 str, _path: &'life2 str, _reserved_bytes: u64, _quota_bytes: u64) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn confirm_totp_enrollment(self: &'life0 Self, _user_id: i64, _record: &'life1 crate::secrets::EncryptedSecretRecord) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn disable_user_totp(self: &'life0 Self, _user_id: i64) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn publish_domain_event(self: &'life0 Self, _spec: PublishDomainEventSpec) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn set_acquire_status(self: &'life0 Self, _book_uuid: &'life1 str, _status: crate::AcquireStatus, _storage_key: Option<&'life2 str>, _error_message: Option<&'life3 str>, _event: Option<PublishDomainEventSpec>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn dispatch_event_deliveries(self: &'life0 Self, event_id: &'life1 str, subscribers: &'life2 [EventSubscriber], operation_id: &'life3 str, mark_dispatched: bool) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn claim_event_delivery(self: &'life0 Self, delivery_id: &'life1 str, owner: &'life2 str, lease_secs: u64, operation_id: &'life3 str, plugin_id: &'life4 str, resource_class: &'life5 str, max_in_flight: u32) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



