**bookclerk_library > store**

# Module: store

## Contents

**Structs**

- [`CatalogEnrichmentFields`](#catalogenrichmentfields) - Durable catalog enrichment fields (blurbs, subjects, provenance).
- [`LibraryStore`](#librarystore) - Handle to the Bookclerk library database.
- [`NewBook`](#newbook) - Input for inserting / updating a book from sync.
- [`NewListeningProgress`](#newlisteningprogress) - Input for upserting listening progress.
- [`NewTitleRequest`](#newtitlerequest) - Input for creating a title request / wishlist row.
- [`NewTitleRequestSource`](#newtitlerequestsource) - Input for upserting a per-storefront wishlist snapshot.
- [`NewWork`](#newwork) - Input for upserting a canonical work.
- [`SavedFilterRecord`](#savedfilterrecord) - Saved Lucene-style quick filter.
- [`UserBookFields`](#userbookfields) - Partial update for user-defined book fields.
- [`WishlistIdentity`](#wishlistidentity) - Bibliographic slice used for wishlist dedupe.

**Functions**

- [`fallback_work_key`](#fallback_work_key) - Local fallback when callers do not supply a discover `work_map_key`.
- [`prefer_enrichment_source`](#prefer_enrichment_source) - Prefer an Audible ownership row with the richest metadata for enrichment.
- [`wishlist_identities_match`](#wishlist_identities_match) - Whether two wishlist rows refer to the same work despite differing keys.

---

## bookclerk_library::store::CatalogEnrichmentFields

*Struct*

Durable catalog enrichment fields (blurbs, subjects, provenance).

**Fields:**
- `description: Option<String>` - Blurb / synopsis text (may contain HTML).
- `language: Option<String>` - BCP-47 or storefront language code when known.
- `cover_url: Option<String>` - HTTPS URL for cover art when known.
- `subjects: Option<String>` - Subject / topic tags from enrichment (serialized).
- `categories: Option<String>` - Storefront category / genre path list (serialized).
- `enrich_source: Option<String>` - Plugin or catalog that last enriched bibliographic fields.
- `enrich_confidence: Option<f64>` - 0–1 confidence score for the last enrichment pass.
- `enrich_updated_at: Option<chrono::DateTime<chrono::Utc>>` - RFC 3339 time of the last enrichment write.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> CatalogEnrichmentFields`
- **Clone**
  - `fn clone(self: &Self) -> CatalogEnrichmentFields`



## bookclerk_library::store::LibraryStore

*Struct*

Handle to the Bookclerk library database.

Async API over a SeaORM [`DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html). `DatabaseConnection` is
cheaply cloneable (shared connection), so `LibraryStore` is `Clone`.

**Methods:**

- `fn kv_get(self: &Self, plugin_key: &str, namespace: &str, key: &str) -> Result<Option<KvEntry>>` - Reads one key; `None` when absent or expired.
- `fn kv_put(self: &Self, plugin_key: &str, namespace: &str, key: &str, spec: KvWriteSpec) -> Result<KvPutOutcome>` - Writes one key, replacing any previous value, metadata, and expiry.
- `fn kv_delete(self: &Self, plugin_key: &str, namespace: &str, key: &str) -> Result<bool>` - Deletes one key. Returns whether a live value existed.
- `fn kv_list(self: &Self, plugin_key: &str, namespace: &str, prefix: &str, cursor: &str, limit: u32) -> Result<KvListPage>` - Lists live keys in lexical order, `limit` per page (0 means
- `fn kv_count(self: &Self, plugin_key: &str, namespace: &str) -> Result<u64>` - Live keys in one namespace.
- `fn kv_prune_expired_in(self: &Self, plugin_key: &str, namespace: &str) -> Result<u64>` - Removes expired rows of one namespace. Returns the rows removed.
- `fn kv_prune_expired(self: &Self) -> Result<u64>` - Removes every expired row across all plugins and namespaces.
- `fn kv_delete_plugin(self: &Self, plugin_key: &str) -> Result<u64>` - Removes every row of one plugin (retire / uninstall).
- `fn scope<impl Into<String>, impl Into<String>>(self: &Self, plugin_key: impl Trait, source_id: impl Trait) -> SourceScope` - Plugin-scoped view — first-party and third-party sources use this.
- `fn storage_scan_put_object(self: &Self, scan_id: &str, key: &str, size: u64, media_rank: u8, is_audio: bool) -> Result<()>` - Inserts an object row. A repeat of the same key is a no-op.
- `fn storage_scan_put_identity(self: &Self, scan_id: &str, identity: &str, key: &str, size: u64, media_rank: u8) -> Result<()>` - Inserts an identity → key row. Repeats are no-ops.
- `fn storage_scan_identity_page(self: &Self, scan_id: &str, identity: &str, after: Option<(i64, &str)>, limit: u64) -> Result<Vec<(String, i64)>>` - One page of identity candidates, ordered by `(media_rank, object_key)`.
- `fn storage_scan_object_page(self: &Self, scan_id: &str, prefix: &str, after_key: Option<&str>, limit: u64) -> Result<Vec<String>>` - One page of object keys at or above `prefix`, in `object_key` order.
- `fn storage_scan_claim(self: &Self, scan_id: &str, key: &str) -> Result<()>` - Marks an object claimed so unmatched-audio counts stay in the database.
- `fn storage_scan_unclaimed_audio(self: &Self, scan_id: &str) -> Result<u64>` - Audio objects in the scan that no library row claimed.
- `fn storage_scan_delete(self: &Self, scan_id: &str) -> Result<()>` - Deletes inventory rows and the generation ownership row for `scan_id`.
- `fn storage_scan_register(self: &Self, scan_id: &str, instance_id: &str, job_id: Option<&str>) -> Result<()>` - Binds `scan_id` to `instance_id` and an optional job.
- `fn storage_scan_mark_complete(self: &Self, scan_id: &str) -> Result<()>` - Marks a generation's inventory complete so a later checkpoint can adopt it.
- `fn storage_scan_adoption(self: &Self, scan_id: &str) -> Result<Option<(String, bool)>>` - Returns `(instance_id, completed)` when the generation row still exists.
- `fn storage_scan_touch(self: &Self, scan_id: &str) -> Result<()>` - Refreshes the generation heartbeat after a page is stored.
- `fn storage_scan_generation_count(self: &Self) -> Result<u64>` - How many generation rows exist. Tests use this to bound retained scans.
- `fn storage_scan_reclaim_abandoned(self: &Self, stale_before: &str) -> Result<u64>` - Deletes generations that are not live resumable work.
- `fn storage_scan_reclaim_if_terminal(self: &Self, job_id: &str) -> Result<u64>` - Deletes this job's inventories when the job is terminal or already gone.
- `fn storage_scan_reclaim_terminal_page(self: &Self, after_scan_id: Option<&str>, limit: u64) -> Result<(u64, Option<String>)>` - Deletes up to `limit` generations whose job is terminal or missing.
- `fn list_books_page(self: &Self, account_id: Option<&str>, after_id: Option<i64>, limit: u64) -> Result<Vec<BookRecord>>` - Pages books by surrogate id so a scan does not load the catalog at once.
- `fn list_books_filtered_page(self: &Self, account_id: Option<&str>, status: Option<&str>, limit: u64, offset: u64) -> Result<BookPage>` - Pages books by account, acquire status, and title.
- `fn list_books_by_uuid_page(self: &Self, uuids: &[String], account_id: Option<&str>, status: Option<&str>, limit: u64, offset: u64) -> Result<BookPage>` - Pages books whose uuid is in `uuids`, then applies account and status.
- `fn publish_domain_event(self: &Self, spec: PublishDomainEventSpec) -> Result<PublishDomainEventOutcome>` - Persist a domain event. Duplicate `(account_id, source, event_type, dedup_key)`
- `fn dispatch_event_deliveries(self: &Self, event_id: &str, subscribers: &[EventSubscriber], operation_id: &str) -> Result<u32>` - Snapshot matching subscribers and create one delivery row per plugin.
- `fn next_undispatched_event(self: &Self) -> Result<Option<DomainEventRecord>>` - Next undispatched outbox row (oldest first).
- `fn process_pending_wakes(self: &Self, limit: u64, _owner: &str, lease_secs: u64) -> Result<PendingWakeProgress>` - Claim up to `limit` `wake_pending` events and process one delivery page each.
- `fn claim_next_event_delivery(self: &Self, owner: &str, lease_secs: u64, operation_id: &str, plugin_ids: &[String], max_in_flight: u32, node_id: &str) -> Result<Option<EventDeliveryRecord>>` - Claim the next ready delivery with a fenced `pending` → `running` mutation.
- `fn claim_next_event_delivery_due_by(self: &Self, owner: &str, lease_secs: u64, operation_id: &str, plugin_ids: &[String], max_in_flight: u32, node_id: &str, due_by: chrono::DateTime<Utc>) -> Result<Option<EventDeliveryRecord>>` - `Self::claim_next_event_delivery` restricted to deliveries due at
- `fn release_unexecuted_event_delivery(self: &Self, fence: &EventDeliveryFence, restore_resume: bool) -> Result<bool>` - Return a claimed delivery to `pending` without consuming an attempt.
- `fn heartbeat_event_delivery(self: &Self, fence: &EventDeliveryFence, lease_secs: u64) -> Result<bool>` - Refresh the lease when `fence` still owns the running generation.
- `fn ack_event_delivery(self: &Self, fence: &EventDeliveryFence) -> Result<bool>` - Mark a fenced delivery `acked`.
- `fn reject_event_delivery(self: &Self, fence: &EventDeliveryFence, reason: &str) -> Result<bool>` - Mark a fenced delivery permanently `rejected`.
- `fn dead_letter_event_delivery(self: &Self, fence: &EventDeliveryFence, reason: &str) -> Result<bool>` - Mark a fenced delivery `dead_letter`.
- `fn retry_event_delivery(self: &Self, fence: &EventDeliveryFence, run_after: chrono::DateTime<Utc>, reason: &str) -> Result<bool>` - Return a fenced delivery to `pending` with backoff (retryable handler).
- `fn suspend_event_delivery(self: &Self, fence: &EventDeliveryFence, checkpoint_json: &str, checkpoint_schema_version: i64, wake_at: chrono::DateTime<Utc>, wake_event_type: &str, wake_filter_json: &str, wake_grants_json: &str) -> Result<bool>` - Suspend a fenced delivery (checkpoint + wake) without burning an attempt.
- `fn fail_event_delivery(self: &Self, fence: &EventDeliveryFence, reason: &str) -> Result<bool>` - Fail a running delivery: retry with backoff or dead-letter at max attempts.
- `fn reclaim_expired_event_deliveries(self: &Self) -> Result<u32>` - Reclaim expired running deliveries back to `pending`.
- `fn get_domain_event(self: &Self, id: &str) -> Result<Option<DomainEventRecord>>` - Load one event envelope.
- `fn get_event_delivery(self: &Self, id: &str) -> Result<Option<EventDeliveryRecord>>` - Load one delivery.
- `fn list_domain_events(self: &Self, limit: u64) -> Result<Vec<DomainEventRecord>>` - Recent outbox events, newest first.
- `fn list_domain_events_for_queue(self: &Self, queue_name: &str, limit: u64) -> Result<Vec<DomainEventRecord>>` - Recent outbox rows on one queue, newest first.
- `fn list_queue_names(self: &Self) -> Result<Vec<String>>` - Distinct queue names that hold at least one outbox row, sorted.
- `fn count_queue_deliveries(self: &Self, queue_name: &str, state: &str) -> Result<i64>` - Delivery rows on one queue in `state`.
- `fn list_event_deliveries_for_queue(self: &Self, queue_name: &str, state: Option<&str>, limit: u64) -> Result<Vec<EventDeliveryRecord>>` - Deliveries filtered by optional state and queue, newest first.
- `fn list_event_deliveries(self: &Self, state: Option<&str>, limit: u64) -> Result<Vec<EventDeliveryRecord>>` - Deliveries filtered by optional state, newest first.
- `fn retry_dead_letter_delivery(self: &Self, id: &str) -> Result<bool>` - Re-queue a dead-lettered delivery for operator retry.
- `fn acknowledge_dead_letter_delivery(self: &Self, id: &str) -> Result<bool>` - Operator acknowledge: mark a dead-letter as rejected (no further retry).
- `fn count_event_deliveries(self: &Self, state: &str) -> Result<i64>` - Count deliveries in `state`.
- `fn event_delivery_metrics(self: &Self) -> Result<EventDeliveryMetrics>` - Operator-visible delivery-queue counters (pending vs suspended split).
- `fn record_event_handler_latency(self: &Self, duration_ms: i64) -> Result<()>` - Record one `onEvent` duration sample.
- `fn list_domain_events_by_dispatch_state(self: &Self, dispatch_state: &str, limit: u64) -> Result<Vec<DomainEventRecord>>` - Outbox envelopes in `dispatch_state`, oldest first.
- `fn list_dispatched_events_page(self: &Self, created_after: &str, after: Option<(&str, &str)>, limit: u64) -> Result<Vec<DomainEventRecord>>` - Dispatched events newer than `created_after`, after optional `(created_at, id)` cursor.
- `fn upsert_event_subscriber(self: &Self, node_id: &str, plugin_id: &str, subscriptions: &[EventCatalogSubscription], enabled: bool) -> Result<()>` - Heartbeat one node's plugin registration (does not delete other nodes).
- `fn upsert_event_subscriber_at(self: &Self, node_id: &str, plugin_id: &str, subscriptions: &[EventCatalogSubscription], enabled: bool, heartbeat_at: chrono::DateTime<Utc>) -> Result<()>` - Heartbeat one node's plugin registration at an explicit timestamp.
- `fn list_live_event_subscribers(self: &Self) -> Result<Vec<EventSubscriberCatalogRecord>>` - Live catalog collapsed by `plugin_id` using the default heartbeat TTL.
- `fn list_live_event_subscribers_with_ttl(self: &Self, ttl_secs: i64) -> Result<Vec<EventSubscriberCatalogRecord>>` - Live catalog collapsed by `plugin_id` for a custom heartbeat TTL.
- `fn list_live_event_subscriber_nodes(self: &Self, ttl_secs: i64) -> Result<Vec<EventSubscriberNodeRecord>>` - Live per-node rows whose heartbeat is within `ttl_secs`.
- `fn dispatch_catalog_matches(self: &Self, event: &DomainEventRecord, operation_id: &str) -> Result<u32>` - Create deliveries from the live catalog for one event.
- `fn reconcile_catalog_deliveries(self: &Self, created_after: chrono::DateTime<Utc>) -> Result<u32>` - Late-join catalog deliveries for missing `(event_id, plugin_id)` pairs.
- `fn request_event_delivery_cancel(self: &Self, id: &str) -> Result<Option<EventDeliveryRecord>>` - Cancel a pending delivery or flag a running one for cooperative stop.
- `fn event_delivery_cancel_requested(self: &Self, id: &str) -> Result<bool>` - True when a running delivery worker should abort after the current step.
- `fn resume_event_delivery(self: &Self, id: &str) -> Result<bool>` - Wake a suspended delivery: `run_after = now` and `resume_pending = 1`.
- `fn prune_event_deliveries(self: &Self, retention_days: u64, dead_letter_retention_days: u64) -> Result<u64>` - Delete terminal deliveries past their retention windows, then empty events.
- `fn prune_event_step(self: &Self, step: EventPruneStep, retention_days: u64, dead_letter_retention_days: u64) -> Result<u64>` - Runs one delete of `Self::prune_event_deliveries`.
- `fn record_plugin_database(self: &Self, plugin_id: &str, binding: &str, adapter_plugin_key: &str, backend_kind: &str, unit_ref: &str) -> Result<PluginDatabaseRecord>` - Records one provisioned binding unit, keeping any existing row.
- `fn get_plugin_database(self: &Self, plugin_id: &str, binding: &str) -> Result<Option<PluginDatabaseRecord>>` - The registry row for `(plugin_id, binding)`, if provisioned.
- `fn list_plugin_databases(self: &Self, plugin_id: Option<&str>) -> Result<Vec<PluginDatabaseRecord>>` - All registry rows, optionally filtered to one owner PluginKey.
- `fn remove_plugin_databases(self: &Self, plugin_id: &str, binding: Option<&str>) -> Result<u64>` - Deletes registry rows for a plugin (one binding, or all of them).
- `fn rebind_plugin_database(self: &Self, plugin_id: &str, binding: &str, adapter_plugin_key: &str, backend_kind: &str, unit_ref: &str) -> Result<PluginDatabaseRecord>` - Inserts or updates placement for one `(plugin_id, binding)` on restore.
- `fn record_title_file(self: &Self, destination: &str, key: &str, book_uuid: &str) -> Result<()>` - Records that the title `book_uuid` writes `key` on `destination`
- `fn claim_title_file(self: &Self, destination: &str, key: &str, book_uuid: &str, writer: &str, expected: Option<&TitleFile>, branch_titles_hold: bool) -> Result<bool>` - Claims `key` on `destination` for the title `book_uuid`, with
- `fn release_title_file(self: &Self, destination: &str, key: &str, writer: &str, prior: Option<&TitleFile>) -> Result<()>` - Gives back a claim `writer` made on `key` on `destination`: the
- `fn count_title_files_of(self: &Self, book_uuid: &str, writer: &str) -> Result<u64>` - How many keys `writer` (a host upload id) holds for the title
- `fn title_file(self: &Self, destination: &str, key: &str) -> Result<Option<TitleFile>>` - Who holds `key` on `destination`; `None` when it is unrecorded.
- `fn enqueue_job(self: &Self, spec: EnqueueJobSpec) -> Result<EnqueueOutcome>` - Admit a job in one transaction, coalescing onto an active dedup key.
- `fn claim_next_job(self: &Self, resource_class: JobResourceClass, owner: &str, lease_secs: u64, operation_id: &str) -> Result<Option<JobRecord>>` - Claim the next ready job with a conditional `pending` → `running` mutation.
- `fn list_job_claim_candidates(self: &Self, resource_class: JobResourceClass, after: Option<&JobClaimCursor>, limit: u64) -> Result<JobClaimPage>` - One page of ready pending jobs of `resource_class`, best first, for a
- `fn claim_job_by_id(self: &Self, job_id: &str, expected_generation: i64, owner: &str, lease_secs: u64) -> Result<Option<JobRecord>>` - Claims the pending job `job_id` for `owner` with a single CAS, but
- `fn list_pending_jobs_page(self: &Self, after_id: Option<&str>, limit: u64) -> Result<(Vec<JobRecord>, Option<String>)>` - One page of pending jobs in id order, for maintenance passes.
- `fn rewrite_pending_job_payload(self: &Self, job_id: &str, lease_generation: i64, payload: &JobPayload) -> Result<bool>` - Replaces the payload of a pending job read at `lease_generation`.
- `fn fail_pending_job(self: &Self, job_id: &str, lease_generation: i64, error_kind: &str, error_message: &str) -> Result<bool>` - Fails a pending job read at `lease_generation`, terminally.
- `fn release_job_claim(self: &Self, fence: &JobFence, resumed: bool) -> Result<bool>` - Returns a job claimed under `fence` to `pending` without spending an
- `fn heartbeat_job(self: &Self, fence: &JobFence, lease_secs: u64, progress: Option<&str>) -> Result<bool>` - Refresh the lease when `fence` still owns the running generation.
- `fn set_job_progress(self: &Self, fence: &JobFence, progress: &str) -> Result<bool>` - Persist progress when `fence` still owns the running generation.
- `fn complete_job(self: &Self, fence: &JobFence, progress: Option<&str>) -> Result<bool>` - Mark a running job succeeded when `fence` still owns the generation.
- `fn suspend_job(self: &Self, fence: &JobFence, checkpoint: &bookclerk_plugin_abi::JobCheckpoint, wake_at: chrono::DateTime<Utc>) -> Result<bool>` - Park a running job as `pending` with a checkpoint, to be claimed again after `wake_at`.
- `fn checkpoint_running_job(self: &Self, fence: &JobFence, checkpoint: &bookclerk_plugin_abi::JobCheckpoint) -> Result<bool>` - Replaces `payload.checkpoint` while `fence` still owns the running job.
- `fn fail_job(self: &Self, fence: &JobFence, error_kind: &str, error_message: &str) -> Result<bool>` - Fail a running job when `fence` still owns the generation.
- `fn request_job_cancel(self: &Self, id: &str) -> Result<Option<JobRecord>>` - Cancel a pending job immediately, or flag a running job for cooperative stop.
- `fn job_cancel_requested(self: &Self, id: &str) -> Result<bool>` - True when a running worker should abort after the current step.
- `fn reclaim_expired_leases(self: &Self) -> Result<u32>` - Reclaim expired running leases with a conditional `running` update.
- `fn get_job(self: &Self, id: &str) -> Result<Option<JobRecord>>` - Load one job by id, rejecting unreadable rows as `invalid_job`.
- `fn list_jobs(self: &Self, limit: u64) -> Result<Vec<JobRecord>>` - List recent jobs, newest first, capped at `limit` (minimum 1).
- `fn prune_terminal_jobs(self: &Self, retention_days: u64) -> Result<u64>` - Delete terminal jobs older than `retention_days`.
- `fn reserve_job_temp_path(self: &Self, job_id: &str, path: &str, reserved_bytes: u64, quota_bytes: u64) -> Result<()>` - Reserve `reserved_bytes` against the job temp quota and register `path`.
- `fn register_job_temp_path(self: &Self, job_id: &str, path: &str) -> Result<()>` - Register a scratch path with a zero-byte reservation (legacy helper).
- `fn list_job_temp_paths(self: &Self, job_id: &str) -> Result<Vec<JobTempPath>>` - List scratch paths registered for `job_id`.
- `fn list_all_job_temp_paths(self: &Self) -> Result<Vec<JobTempPath>>` - List every registered scratch path (used by the startup sweeper).
- `fn unregister_job_temp_path(self: &Self, job_id: &str, path: &str) -> Result<()>` - Drop one scratch-path row after that path is gone from disk.
- `fn reserved_temp_bytes(self: &Self) -> Result<u64>` - Sum of reserved scratch bytes across all jobs.
- `fn has_running_acquire_job(self: &Self) -> Result<bool>` - True when any acquire job is currently running.
- `fn list_orphaned_acquire_books(self: &Self) -> Result<Vec<BookRecord>>` - Books left in `queued` / `downloading` with no running acquire job.
- `fn reconcile_orphaned_acquire_rows(self: &Self) -> Result<u32>` - Mark orphaned `queued` / `downloading` books as `error` after a crash.
- `fn count_active_jobs(self: &Self) -> Result<i64>` - Count pending + running jobs (admission cap).
- `fn from_connection(db: DatabaseConnection) -> Self` - Wrap an already-opened (and migrated) SeaORM connection.
- `fn with_atomic_txn(self: Self, backend: Arc<dyn AtomicTxnBackend>) -> Self` - Attach a guest atomic backend for named security operations.
- `fn with_typed_exec(self: Self, backend: Arc<dyn crate::TypedAtomicExec>) -> Self` - Attach a guest `executeAtomic` backend for granted job SQL.
- `fn with_max_binds(self: Self, max_binds: u32) -> Self` - Records the guest's negotiated bind cap for wake `IN (…)` chunking.
- `fn with_plan_limits(self: Self, max_statements: u32, max_result_rows: u32, max_payload_bytes: u32) -> Self` - Records negotiated statement / result / payload caps from connect.
- `fn with_db_capabilities(self: Self, caps: bookclerk_plugin_abi::DbCapabilities) -> Self` - Records the full negotiated [`bookclerk_plugin_abi::DbCapabilities`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executedbcapabilities).
- `fn db_capabilities(self: &Self) -> &bookclerk_plugin_abi::DbCapabilities` - Negotiated capability advertisement used for plan validation.
- `fn execute_guest_atomic(self: &Self, req: bookclerk_plugin_abi::ExecuteRequest, policy: &bookclerk_plugin_abi::GuestSqlPolicy) -> std::result::Result<bookclerk_plugin_abi::ExecuteReply, bookclerk_plugin_abi::PluginError>` - Runs guest-authored SQL after grammar, table-scope, and cap checks.
- `fn dispatch_chunk_size(self: &Self) -> usize` - Subscribers packed into one dispatch plan (receipt overhead is ~12 statements).
- `fn wake_page(self: &Self) -> u64` - Wake delivery page size derived from negotiated `maxBinds`.
- `fn connection(self: &Self) -> &DatabaseConnection` - Borrow the underlying SeaORM connection (e.g. for [`crate::secrets`](secrets.md)).
- `fn db(self: &Self) -> &DatabaseConnection` - Alias for `Self::connection` — convenience name used by credential modules.
- `fn delete_elevated_operator_sessions_for_user(self: &Self, user_id: i64) -> Result<u64>` - Revoke Operator sessions minted by this user's elevate-to-operator flow.
- `fn upsert_account(self: &Self, plugin_key: &str, account_id: &str, marketplace: &str, label: Option<&str>, scan_enabled: bool, source: &str) -> Result<AccountRecord>` - Upsert an account (updates `scan_enabled` on conflict).
- `fn ensure_account(self: &Self, plugin_key: &str, account_id: &str, marketplace: &str, label: Option<&str>, source: &str) -> Result<AccountRecord>` - Ensure an account row exists without overwriting `scan_enabled`.
- `fn remap_account_id(self: &Self, plugin_key: &str, from: &str, to: &str) -> Result<()>` - Remap `from` → `to` account ids within one plugin (books + account row).
- `fn reconcile_account_id(self: &Self, plugin_key: &str, canonical_id: &str, aliases: &[&str], marketplace: &str, label: Option<&str>, scan_enabled: bool, source: &str) -> Result<AccountRecord>` - Remap any existing alias account ids onto `canonical_id`, then upsert.
- `fn get_account(self: &Self, plugin_key: &str, account_id: &str) -> Result<Option<AccountRecord>>` - Loads one account by `(plugin_key, account_id)`, if present.
- `fn find_accounts_by_id(self: &Self, account_id: &str) -> Result<Vec<AccountRecord>>` - Every account row with this store id, across plugins.
- `fn get_account_unique(self: &Self, account_id: &str) -> Result<Option<AccountRecord>>` - Resolves one account by store id, failing closed when two plugins own
- `fn list_accounts(self: &Self) -> Result<Vec<AccountRecord>>` - Lists all store accounts in the library database.
- `fn count_identities(self: &Self) -> Result<i64>` - Count library identity rows (SQL `COUNT`, not a full fetch).
- `fn count_accounts(self: &Self) -> Result<i64>` - Count account rows (SQL `COUNT`, not a full fetch).
- `fn find_accounts(self: &Self, identifier: &str) -> Result<Vec<AccountRecord>>` - Every account whose id or nickname (`label`) matches, case-insensitive.
- `fn find_account(self: &Self, identifier: &str, plugin_key: Option<&str>) -> Result<Option<AccountRecord>>` - Resolve one account by id or nickname (`label`), case-insensitive.
- `fn set_scan_enabled(self: &Self, plugin_key: &str, account_id: &str, scan_enabled: bool) -> Result<()>` - Toggle whether an account is included in automatic library scans.
- `fn mark_connection_active(self: &Self, plugin_key: &str, account_id: &str) -> Result<()>` - Mark bookstore credentials active again (after reconnect).
- `fn delete_account_secrets(self: &Self, plugin_key: &str, account_id: &str) -> Result<()>` - Delete all `encrypted_secrets` rows for an account (source auth + Widevine CDM).
- `fn revoke_credentials(self: &Self, plugin_key: &str, account_id: &str) -> Result<()>` - Mark bookstore credentials revoked without deleting the account or books.
- `fn upsert_portal_identity(self: &Self, provider: &str, external_user_id: &str, label: Option<&str>) -> Result<crate::models::PortalIdentity>` - Create or fetch a portal identity for `(provider, external_user_id)`.
- `fn create_user(self: &Self, role: UserRole, display_name: Option<&str>, password_hash: Option<&str>) -> Result<UserRecord>` - Create a first-party user.
- `fn create_user_with_login(self: &Self, role: UserRole, display_name: Option<&str>, login_name: Option<&str>, password_hash: Option<&str>) -> Result<UserRecord>` - Create a first-party user with optional local login_name and email.
- `fn create_user_with_profile(self: &Self, role: UserRole, display_name: Option<&str>, login_name: Option<&str>, email: Option<&str>, password_hash: Option<&str>) -> Result<UserRecord>` - Create a first-party user with login, email, and optional password hash.
- `fn get_user(self: &Self, id: i64) -> Result<Option<UserRecord>>` - Look up a first-party user by id.
- `fn get_user_by_login_name(self: &Self, login_name: &str) -> Result<Option<UserRecord>>` - Look up by local login_name (case-insensitive).
- `fn get_user_by_email(self: &Self, email: &str) -> Result<Option<UserRecord>>` - Look up by email (case-insensitive).
- `fn get_user_password_hash(self: &Self, id: i64) -> Result<Option<String>>` - Raw password hash for verification (never expose via API).
- `fn list_users(self: &Self) -> Result<Vec<UserRecord>>` - List all first-party users (admin tooling).
- `fn list_user_presence_extras(self: &Self, listening_within: chrono::Duration) -> Result<std::collections::HashMap<i64, crate::models::UserPresenceExtras>>` - Presence, listening, and storefront links for administrator user lists.
- `fn delete_user(self: &Self, id: i64) -> Result<()>` - Delete a first-party user and personal data.
- `fn set_user_status(self: &Self, id: i64, status: UserStatus) -> Result<UserRecord>` - Set user status (`active` / `disabled`).
- `fn set_user_display_name(self: &Self, id: i64, display_name: Option<&str>) -> Result<UserRecord>` - Set display name (does not bump security_version).
- `fn set_user_password_hash(self: &Self, id: i64, password_hash: Option<&str>) -> Result<UserRecord>` - Set or clear Argon2id password hash; bumps `security_version`.
- `fn set_user_login_name(self: &Self, id: i64, login_name: Option<&str>) -> Result<UserRecord>` - Set login_name (unique); does not bump security_version.
- `fn set_user_email(self: &Self, id: i64, email: Option<&str>) -> Result<UserRecord>` - Set contact email (unique when set); does not bump security_version.
- `fn set_user_avatar_source(self: &Self, id: i64, source: Option<&str>) -> Result<UserRecord>` - Set the user's picture source (`monogram` / `gravatar` / `upload` / `sso:{id}` / auto).
- `fn set_portal_identity_picture(self: &Self, provider: &str, external_user_id: &str, picture_url: Option<&str>) -> Result<()>` - Store or clear the HTTPS picture URL on a portal identity.
- `fn list_user_sso_pictures(self: &Self, user_id: i64) -> Result<Vec<crate::models::UserSsoPicture>>` - IdP pictures linked to `user_id`, with last portal-session activity.
- `fn delete_portal_sessions_for_user(self: &Self, user_id: i64) -> Result<u64>` - Delete all portal sessions for identities linked to `user_id`.
- `fn insert_user_invite(self: &Self, token_hash: &str, role: UserRole, login_name: Option<&str>, display_name: Option<&str>, expires_at: chrono::DateTime<Utc>, created_by: &str) -> Result<crate::models::UserInviteRecord>` - Insert a user invite (store hash only).
- `fn redeem_user_invite(self: &Self, token_hash: &str) -> Result<crate::models::UserInviteRecord>` - Atomically redeem a user invite.
- `fn set_user_role(self: &Self, id: i64, role: UserRole) -> Result<UserRecord>` - Set user role (`owner` / `administrator` / `member`).
- `fn count_administrators(self: &Self) -> Result<u64>` - Count administrators (includes disabled).
- `fn count_active_administrators(self: &Self) -> Result<u64>` - Count administrators with status `active`.
- `fn count_owners(self: &Self) -> Result<u64>` - Count owners (includes disabled) — bootstrap / inventory.
- `fn count_active_owners(self: &Self) -> Result<u64>` - Count owners with status `active` (last-owner demote/disable/delete guard).
- `fn ensure_users_bridged(self: &Self) -> Result<usize>` - Backfill `portal_identities.user_id` and remap prefs `portal:{id}` → `user:{id}`.
- `fn ensure_local_portal_identity(self: &Self, user_id: i64, label: Option<&str>) -> Result<crate::models::PortalIdentity>` - Ensure a `local` portal identity exists for a first-party user.
- `fn get_portal_identity(self: &Self, provider: &str, external_user_id: &str) -> Result<Option<crate::models::PortalIdentity>>` - Loads a portal identity by primary key, if present.
- `fn get_portal_identity_by_id(self: &Self, id: i64) -> Result<Option<crate::models::PortalIdentity>>` - Look up portal identity by row id.
- `fn insert_claim_ticket(self: &Self, token_hash: &str, identity_id: Option<i64>, expires_at: chrono::DateTime<Utc>, created_by: &str) -> Result<crate::models::ClaimTicketRecord>` - Insert a claim ticket (store only the hash).
- `fn get_claim_ticket_by_hash(self: &Self, token_hash: &str) -> Result<Option<crate::models::ClaimTicketRecord>>` - Loads a claim ticket row by its token hash, if present.
- `fn list_open_claim_tickets(self: &Self) -> Result<Vec<crate::models::ClaimTicketRecord>>` - List unredeemed, unexpired claim tickets (newest first).
- `fn redeem_claim_ticket(self: &Self, token_hash: &str) -> Result<crate::models::ClaimTicketRecord>` - Atomically mark a claim ticket redeemed.
- `fn redeem_claim_ticket_to_session(self: &Self, token_hash: &str, session_hash: &str, expires_at: chrono::DateTime<Utc>, client: Option<&crate::SessionClientInfo>, new_password_hash: Option<&str>, password_fingerprint: Option<&str>) -> Result<crate::models::PortalIdentity>` - Consume a claim ticket, optionally set an unset local password, and mint
- `fn insert_portal_session(self: &Self, token_hash: &str, identity_id: i64, expires_at: chrono::DateTime<Utc>) -> Result<()>` - Create a portal session (hash only).
- `fn insert_portal_session_with_client(self: &Self, token_hash: &str, identity_id: i64, expires_at: chrono::DateTime<Utc>, client: Option<&crate::SessionClientInfo>) -> Result<()>` - Create a portal session with optional client metadata.
- `fn delete_portal_session(self: &Self, token_hash: &str) -> Result<bool>` - Delete a portal session by token hash (logout / revoke).
- `fn list_portal_sessions_for_identity(self: &Self, identity_id: i64) -> Result<Vec<(i64, String, String)>>` - List portal sessions for an identity (id, created_at, expires_at).
- `fn list_portal_session_records_for_identity(self: &Self, identity_id: i64) -> Result<Vec<crate::models::PortalSessionRecord>>` - List portal sessions with client metadata for session management UIs.
- `fn delete_portal_session_by_id_for_identity(self: &Self, id: i64, identity_id: i64) -> Result<bool>` - Delete a portal session by id only if it belongs to `identity_id`.
- `fn count_account_links_for_account(self: &Self, plugin_key: &str, account_id: &str) -> Result<u64>` - Count account_links referencing `account_id`.
- `fn get_portal_session_identity(self: &Self, token_hash: &str) -> Result<Option<crate::models::PortalIdentity>>` - Resolve a valid portal session to its identity.
- `fn insert_operator_session(self: &Self, token_hash: &str, expires_at: chrono::DateTime<Utc>) -> Result<()>` - Create a durable operator session (hash only).
- `fn insert_operator_session_with_client(self: &Self, token_hash: &str, expires_at: chrono::DateTime<Utc>, client: Option<&crate::SessionClientInfo>) -> Result<()>` - Create a durable operator session with optional client metadata.
- `fn insert_elevated_operator_session(self: &Self, token_hash: &str, expires_at: chrono::DateTime<Utc>, elevated_from_user_id: i64) -> Result<()>` - Create an elevated operator session (Administrator reauth → short TTL).
- `fn insert_elevated_operator_session_with_client(self: &Self, token_hash: &str, expires_at: chrono::DateTime<Utc>, elevated_from_user_id: i64, client: Option<&crate::SessionClientInfo>) -> Result<()>` - Create an elevated operator session with optional client metadata.
- `fn get_operator_session(self: &Self, token_hash: &str) -> Result<Option<crate::models::OperatorSessionRecord>>` - Look up a valid operator session (touches `last_used_at`).
- `fn operator_session_valid(self: &Self, token_hash: &str) -> Result<bool>` - Return whether a hashed operator session is still valid (and touch last_used).
- `fn set_operator_session_impersonating(self: &Self, token_hash: &str, user_id: Option<i64>) -> Result<crate::models::OperatorSessionRecord>` - Set or clear impersonation on an operator session.
- `fn delete_operator_session(self: &Self, token_hash: &str) -> Result<bool>` - Delete an operator session by token hash (logout / revoke).
- `fn delete_operator_session_by_id(self: &Self, id: i64) -> Result<bool>` - Delete an operator session by row id.
- `fn prune_expired_operator_sessions(self: &Self) -> Result<u64>` - Delete expired operator sessions (bounded cleanup).
- `fn list_operator_sessions(self: &Self) -> Result<Vec<crate::models::OperatorSessionRecord>>` - List active operator sessions (newest first) for remote sign-out UI.
- `fn first_portal_identity_for_user(self: &Self, user_id: i64) -> Result<Option<crate::models::PortalIdentity>>` - First portal identity linked to a first-party user (for impersonation scoping).
- `fn list_portal_identities_for_user(self: &Self, user_id: i64) -> Result<Vec<crate::models::PortalIdentity>>` - All portal identities linked to a first-party user.
- `fn link_portal_identity_to_user(self: &Self, provider: &str, external_user_id: &str, user_id: i64, label: Option<&str>) -> Result<crate::models::PortalIdentity>` - Link `(provider, external_user_id)` to an existing user (SSO / invite_only).
- `fn insert_oidc_rp_state(self: &Self, state_hash: &str, provider_id: &str, pkce_verifier: &str, nonce: &str, purpose: &str, user_id: Option<i64>, expires_at: chrono::DateTime<Utc>) -> Result<()>` - Insert a short-lived OIDC RP authorize state row.
- `fn take_oidc_rp_state(self: &Self, state_hash: &str) -> Result<Option<(String, String, String, String, Option<i64>)>>` - Consume (delete) an OIDC RP state by hash if it has not expired.
- `fn insert_webauthn_credential(self: &Self, user_id: i64, credential_id: &str, passkey_json: &str, name: Option<&str>) -> Result<()>` - Store a WebAuthn credential for a user.
- `fn list_webauthn_credentials(self: &Self, user_id: i64) -> Result<Vec<StoredPasskey>>` - Passkeys registered to a user.
- `fn count_webauthn_credentials(self: &Self, user_id: i64) -> Result<u64>` - Count passkeys for a user.
- `fn get_webauthn_credential_by_cred_id(self: &Self, credential_id: &str) -> Result<Option<(i64, i64, String)>>` - Look up a passkey by credential id.
- `fn update_webauthn_credential(self: &Self, id: i64, passkey_json: &str) -> Result<()>` - Replace passkey JSON after a successful assertion (counter update).
- `fn delete_webauthn_credential(self: &Self, user_id: i64, id: i64) -> Result<bool>` - Delete a passkey owned by `user_id`.
- `fn set_user_totp_enabled(self: &Self, user_id: i64, enabled: bool) -> Result<()>` - Set whether this user has a confirmed TOTP authenticator.
- `fn confirm_totp_enrollment(self: &Self, user_id: i64, secret_b32: &str) -> Result<()>` - Promote a pending TOTP secret to primary and set `totp_enabled` in one transaction.
- `fn disable_user_totp(self: &Self, user_id: i64) -> Result<()>` - Delete TOTP secrets and clear `totp_enabled` in one transaction.
- `fn insert_webauthn_challenge(self: &Self, challenge_id: &str, user_id: Option<i64>, kind: &str, state_json: &str, expires_at: chrono::DateTime<Utc>) -> Result<()>` - Insert a WebAuthn ceremony state.
- `fn take_webauthn_challenge(self: &Self, challenge_id: &str, kind: &str) -> Result<Option<(Option<i64>, String)>>` - Consume a WebAuthn ceremony by id (transactional delete-and-return).
- `fn prune_expired_oidc_rp_states(self: &Self) -> Result<u64>` - Drop expired OIDC RP states (bounded cleanup).
- `fn prune_expired_webauthn_challenges(self: &Self) -> Result<u64>` - Drop expired WebAuthn challenges (bounded cleanup).
- `fn count_oidc_rp_states(self: &Self) -> Result<u64>` - Count pending OIDC RP states (for public-begin caps).
- `fn count_webauthn_challenges(self: &Self) -> Result<u64>` - Count pending WebAuthn challenges (for public-begin caps).
- `fn portal_session_created_at(self: &Self, token_hash: &str) -> Result<Option<chrono::DateTime<Utc>>>` - Created-at of a still-valid portal session (recent-auth checks).
- `fn set_portal_session_created_at(self: &Self, token_hash: &str, created_at: chrono::DateTime<Utc>) -> Result<bool>` - Overwrite `created_at` for a portal session (tests / recent-auth fixtures).
- `fn insert_security_audit_event(self: &Self, actor: &str, action: &str, detail_json: Option<&str>) -> Result<crate::models::SecurityAuditEvent>` - Appends one security audit event row.
- `fn insert_oidc_client(self: &Self, client_id: &str, client_secret_hash: Option<&str>, redirect_uris: &[String], name: Option<&str>, issue_refresh_token: bool, allowed_scopes: &[String], enabled: bool, plugin_id: Option<&str>) -> Result<OidcClientRecord>` - Insert a new OIDC client (public clients may omit secret hash).
- `fn upsert_plugin_oidc_client(self: &Self, client_id: &str, plugin_id: &str, name: &str, redirect_uris: &[String], issue_refresh_token: bool, allowed_scopes: &[String]) -> Result<OidcClientRecord>` - Insert a plugin-owned OIDC client when missing; refresh redirect URIs when present.
- `fn update_oidc_client(self: &Self, client_id: &str, redirect_uris: &[String], name: Option<&str>, issue_refresh_token: bool, allowed_scopes: &[String], enabled: bool, secret_hash: Option<Option<String>>) -> Result<Option<OidcClientRecord>>` - Update an existing OIDC client without changing the secret unless `secret_hash` is `Some`.
- `fn set_oidc_client_secret(self: &Self, client_id: &str, client_secret_hash: Option<&str>) -> Result<Option<OidcClientRecord>>` - Replace the stored client-secret hash (or clear it).
- `fn delete_oidc_client(self: &Self, client_id: &str) -> Result<bool>` - Delete a registered OIDC client by `client_id`.
- `fn list_oidc_clients(self: &Self) -> Result<Vec<OidcClientRecord>>` - List registered OIDC clients, oldest first.
- `fn get_oidc_client(self: &Self, client_id: &str) -> Result<Option<OidcClientRecord>>` - Loads a registered OIDC client by `client_id`, if present.
- `fn insert_oidc_auth_code(self: &Self, code_hash: &str, client_id: &str, user_id: i64, redirect_uri: &str, code_challenge: &str, code_challenge_method: &str, scope: &str, expires_at: chrono::DateTime<Utc>) -> Result<()>` - Persists a hashed OIDC authorization code with PKCE metadata.
- `fn peek_oidc_auth_code(self: &Self, code_hash: &str) -> Result<Option<(String, i64, String, String, String, String)>>` - Reads an unconsumed, unexpired auth code without consuming it; returns
- `fn consume_oidc_auth_code(self: &Self, code_hash: &str) -> Result<Option<(String, i64, String, String, String, String)>>` - Atomically consume an auth code; returns (client_id, user_id, redirect_uri, challenge, method, scope).
- `fn insert_oidc_refresh_token(self: &Self, token_hash: &str, client_id: &str, user_id: i64, scope: &str, expires_at: chrono::DateTime<Utc>) -> Result<()>` - Persists a hashed OIDC refresh token.
- `fn get_oidc_refresh_token(self: &Self, token_hash: &str) -> Result<Option<(String, i64, String)>>` - Loads a refresh token row by hash, if present.
- `fn revoke_oidc_refresh_token(self: &Self, token_hash: &str) -> Result<bool>` - Marks a refresh token revoked; returns whether a row was updated.
- `fn list_security_audit_events(self: &Self, limit: u64) -> Result<Vec<crate::models::SecurityAuditEvent>>` - List recent security audit events (newest first).
- `fn link_account(self: &Self, identity_id: i64, plugin_key: &str, account_id: &str, source: &str) -> Result<crate::models::AccountLinkRecord>` - Link a bookstore account to a portal identity.
- `fn list_account_links(self: &Self, identity_id: i64) -> Result<Vec<crate::models::AccountLinkRecord>>` - Lists store accounts linked to a portal identity.
- `fn unlink_account(self: &Self, identity_id: i64, plugin_key: &str, account_id: &str) -> Result<()>` - Remove an account link row (does not delete the account).
- `fn upsert_book(self: &Self, book: &NewBook) -> Result<BookRecord>` - Upsert a book from a library sync.
- `fn get_book_by_uuid(self: &Self, uuid: &str) -> Result<Option<BookRecord>>` - Look up a book by its public `uuid`.
- `fn get_book(self: &Self, title_id: &str, account_id: &str) -> Result<Option<BookRecord>>` - Look up a book by uuid, product_id, asin, or isbn for the given account.
- `fn get_book_in(self: &Self, plugin_key: Option<&str>, title_id: &str, account_id: &str) -> Result<Option<BookRecord>>` - `Self::get_book` narrowed to one plugin key when `plugin_key` is set.
- `fn find_books_by_isbn(self: &Self, isbn: &str) -> Result<Vec<BookRecord>>` - All ownership rows sharing an ISBN (cross-account / cross-store enrichment).
- `fn list_books(self: &Self, account_id: Option<&str>) -> Result<Vec<BookRecord>>` - Lists books, optionally filtered to one `account_id`.
- `fn count_books(self: &Self, account_id: Option<&str>) -> Result<i64>` - Count book rows (SQL `COUNT`, not a full fetch).
- `fn update_user_fields(self: &Self, title_id: &str, account_id: &str, fields: &UserBookFields) -> Result<()>` - Update user-defined fields (tags, ratings, finished) without touching scan metadata.
- `fn set_pdf_status(self: &Self, title_id: &str, account_id: &str, status: AcquireStatus, pdf_storage_key: Option<&str>) -> Result<()>` - Updates companion PDF acquire status and optional storage key.
- `fn is_ignored(self: &Self, title_id: &str, account_id: &str) -> Result<bool>` - Returns whether (`title_id`, `account_id`) is on the ignore list.
- `fn set_ignored(self: &Self, title_id: &str, account_id: &str, ignored: bool, reason: Option<&str>) -> Result<()>` - Adds or removes a title from the per-account ignore list.
- `fn set_acquire_status(self: &Self, title_id: &str, account_id: &str, status: AcquireStatus, storage_key: Option<&str>, error_message: Option<&str>) -> Result<()>` - Updates acquire status, storage key, and optional error message.
- `fn bulk_set_acquire_status(self: &Self, account: Option<&str>, asins: &[String], status: AcquireStatus) -> Result<u32>` - Bulk-update acquire status (classic `set-status --force`).
- `fn list_saved_filters(self: &Self) -> Result<Vec<SavedFilterRecord>>` - List saved quick-filter expressions.
- `fn upsert_saved_filter(self: &Self, name: &str, query: &str) -> Result<SavedFilterRecord>` - Inserts or replaces a named saved library filter query.
- `fn get_saved_filter(self: &Self, name: &str) -> Result<Option<SavedFilterRecord>>` - Loads a saved filter by name, if present.
- `fn delete_saved_filter(self: &Self, name: &str) -> Result<()>` - Deletes a saved filter by name.
- `fn count_by_status(self: &Self, status: AcquireStatus) -> Result<i64>` - Counts books whose acquire status equals `status`.
- `fn update_catalog_enrichment(self: &Self, book_uuid: &str, fields: &CatalogEnrichmentFields) -> Result<()>` - Persist enrichment fields without touching scan / ownership columns.
- `fn upsert_work(self: &Self, work: &NewWork) -> Result<WorkRecord>` - Upsert a canonical work row (COALESCE-preserve on conflict).
- `fn get_work(self: &Self, id: &str) -> Result<Option<WorkRecord>>` - Loads a canonical work by id, if present.
- `fn find_work_by_asin(self: &Self, asin: &str) -> Result<Option<WorkRecord>>` - Find work by Amazon ASIN identifier.
- `fn find_work_by_isbn(self: &Self, isbn: &str) -> Result<Option<WorkRecord>>` - Finds a canonical work whose preferred ISBN matches `isbn`.
- `fn list_works(self: &Self) -> Result<Vec<WorkRecord>>` - Lists every canonical work row in the library database.
- `fn link_book_to_work(self: &Self, work_id: &str, book_uuid: &str) -> Result<()>` - Associates a book UUID with a canonical work id.
- `fn work_id_for_book(self: &Self, book_uuid: &str) -> Result<Option<String>>` - Returns the work id linked to `book_uuid`, if any.
- `fn book_uuids_for_work(self: &Self, work_id: &str) -> Result<Vec<String>>` - Lists book UUIDs linked to `work_id`.
- `fn upsert_listening_progress(self: &Self, row: &NewListeningProgress) -> Result<ListeningProgressRecord>` - Inserts or updates listening progress for a title.
- `fn get_listening_progress(self: &Self, provider: &str, external_user_id: &str, external_item_id: &str) -> Result<Option<ListeningProgressRecord>>` - Loads listening progress for one identity and book.
- `fn list_listening_progress(self: &Self, external_user_id: Option<&str>) -> Result<Vec<ListeningProgressRecord>>` - Lists listening progress rows for an identity.
- `fn create_title_request(self: &Self, req: &NewTitleRequest) -> Result<TitleRequestRecord>` - Creates a wishlist / title-request row.
- `fn upsert_title_request_sources(self: &Self, title_request_id: i64, sources: &[NewTitleRequestSource]) -> Result<TitleRequestRecord>` - Upsert per-storefront snapshots for a wishlist row, then return the merged record.
- `fn list_title_request_sources(self: &Self, title_request_id: i64) -> Result<Vec<TitleRequestSourceRecord>>` - Lists per-storefront snapshots for a title request.
- `fn find_open_wishlist(self: &Self, identity_id: Option<i64>, work_key: &str) -> Result<Option<TitleRequestRecord>>` - Open wishlist row for this identity + work key, if any.
- `fn find_open_wishlist_matching(self: &Self, identity_id: Option<i64>, work_key: &str, title: &str, authors: Option<&str>, asin: Option<&str>, isbn: Option<&str>) -> Result<Option<TitleRequestRecord>>` - Open wishlist row that matches bibliographic identity even when `work_key` differs.
- `fn list_wishlist(self: &Self, identity_id: Option<i64>) -> Result<Vec<TitleRequestRecord>>` - Personal open wishlist for a portal identity, or operator (`identity_id` null).
- `fn list_global_request_queue(self: &Self) -> Result<Vec<GlobalQueueEntry>>` - Global request queue: open wishes grouped by `work_key`.
- `fn get_title_request_by_uuid(self: &Self, uuid: &str) -> Result<Option<TitleRequestRecord>>` - Loads a wishlist title-request row by UUID, if present.
- `fn list_title_requests(self: &Self, status: Option<RequestStatus>) -> Result<Vec<TitleRequestRecord>>` - Lists wishlist / title-request rows.
- `fn update_title_request_status(self: &Self, uuid: &str, status: RequestStatus, resolved_book_uuid: Option<&str>) -> Result<()>` - Updates the lifecycle status of a title request.
- `fn upsert_embedding(self: &Self, target_kind: &str, target_id: &str, model: &str, dims: i64, vector: &[u8], text_hash: &str) -> Result<()>` - Inserts or replaces an embedding vector for a target.
- `fn get_embedding_vector(self: &Self, target_kind: &str, target_id: &str, model: &str) -> Result<Option<(String, Vec<u8>)>>` - Loads the raw embedding bytes for a target, if present.
- `fn list_embeddings(self: &Self, target_kind: &str, model: &str) -> Result<Vec<(String, Vec<u8>)>>` - Lists embedding metadata rows (optionally filtered).
- `fn embedding_text_hash(self: &Self, target_kind: &str, target_id: &str, model: &str) -> Result<Option<String>>` - Returns the stored text hash for an embedding target, if any.
- `fn get_user_preferences(self: &Self, subject_key: &str) -> Result<Option<UserPreferences>>` - Load per-user GUI / Discover preferences by subject key.
- `fn get_user_preferences_or_default(self: &Self, subject_key: &str, identity_id: Option<i64>) -> Result<UserPreferences>` - Preferences for `subject_key`, or in-memory defaults when no row exists.
- `fn upsert_user_preferences(self: &Self, subject_key: &str, identity_id: Option<i64>, default_view: &str, disabled_shelves: &[String], discover_sort: &str, discover_sort_dir: &str, discover_language: Option<&str>, discover_excluded_sources: &[String], theme: &str) -> Result<UserPreferences>` - Insert or replace preferences for a subject (operator or portal identity).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> LibraryStore`



## bookclerk_library::store::NewBook

*Struct*

Input for inserting / updating a book from sync.

**Fields:**
- `uuid: Option<String>` - Stable UUID for this row (API / foreign-key identity).
- `product_id: String` - Storefront product id (ASIN, ISBN, UUID, …).
- `plugin_key: String` - Provenance-qualified PluginKey that owns the account. Forced by the
- `source: String` - Display alias of the owning plugin (`audible`, `libro`, …).
- `account_id: String` - Store or operator account id this row belongs to.
- `asin: Option<String>` - Amazon ASIN when known; otherwise null.
- `isbn: Option<String>` - ISBN-10/13 when known; otherwise null.
- `marketplace: String` - Store marketplace / locale code (for example `us`, `uk`).
- `title: String` - Display title of the work or edition.
- `authors: Option<String>` - Comma-separated or JSON author list from the storefront.
- `narrators: Option<String>` - Comma-separated or JSON narrator list when present.
- `series: Option<String>` - Series name when the title belongs to a series.
- `series_index: Option<String>` - Position within the series (storefront string form).
- `series_asin: Option<String>` - Amazon series ASIN when the storefront exposes one.
- `purchased_at: Option<chrono::DateTime<chrono::Utc>>` - RFC 3339 purchase time from the storefront, when known.
- `publisher: Option<String>` - Publisher name from metadata enrichment or the storefront.
- `length_minutes: Option<i64>` - Runtime in whole minutes when the storefront reports it.
- `is_abridged: bool` - Whether the edition is abridged (0/1 or bool).
- `content_kind: String` - Title kind: `book`, `episode`, `podcast`, ….
- `categories: Option<String>` - Storefront category / genre path list (serialized).
- `subtitle: Option<String>` - Optional subtitle from bibliographic metadata.
- `published_at: Option<chrono::DateTime<chrono::Utc>>` - Publication date string from the storefront or enrichment.

**Methods:**

- `fn needs_html_entity_decode(self: &Self) -> bool` - True when any human-readable field may contain an HTML entity.
- `fn decode_html_entities(self: & mut Self)` - Decode HTML entities in human-readable metadata fields (titles, etc.).
- `fn minimal<impl Into<String>, impl Into<String>, impl Into<String>, impl Into<String>>(product_id: impl Trait, account_id: impl Trait, marketplace: impl Trait, title: impl Trait) -> Self` - Minimal Audible book row with scan metadata defaults.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NewBook`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::store::NewListeningProgress

*Struct*

Input for upserting listening progress.

**Fields:**
- `identity_id: Option<i64>` - Foreign key to `portal_identities.id`.
- `provider: String` - External identity or integration provider id (for example ABS).
- `external_user_id: String` - User id at the external provider.
- `book_uuid: Option<String>` - Foreign key to `books.uuid`.
- `work_id: Option<String>` - Canonical work id this edition or request resolves to.
- `external_item_id: String` - Provider-native listening-progress item id.
- `title: Option<String>` - Display title of the work or edition.
- `authors: Option<String>` - Comma-separated or JSON author list from the storefront.
- `asin: Option<String>` - Amazon ASIN when known; otherwise null.
- `isbn: Option<String>` - ISBN-10/13 when known; otherwise null.
- `progress: Option<f64>` - Fractional progress 0.0–1.0 when the provider reports it.
- `current_time_seconds: Option<f64>` - Current playback position within the title, in seconds.
- `duration_seconds: Option<f64>` - Total duration in seconds when known.
- `is_finished: bool` - Whether the listener marked the title finished (0/1 or bool).
- `last_listened_at: Option<chrono::DateTime<chrono::Utc>>` - RFC 3339 time of the last playback update.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NewListeningProgress`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::store::NewTitleRequest

*Struct*

Input for creating a title request / wishlist row.

**Fields:**
- `uuid: Option<String>` - Stable UUID for this row (API / foreign-key identity).
- `identity_id: Option<i64>` - Foreign key to `portal_identities.id`.
- `title: String` - Display title of the work or edition.
- `authors: Option<String>` - Comma-separated or JSON author list from the storefront.
- `asin: Option<String>` - Amazon ASIN when known; otherwise null.
- `isbn: Option<String>` - ISBN-10/13 when known; otherwise null.
- `notes: Option<String>` - Free-form operator or requester notes.
- `status: crate::models::RequestStatus` - Lifecycle status for the row (user, request, …).
- `work_key: String` - Stable bibliographic key; empty triggers [`fallback_work_key`](store.md#bookclerk_librarystorefallback_work_key).
- `work_id: Option<String>` - Canonical work id this edition or request resolves to.
- `resolved_book_uuid: Option<String>` - Library `books.uuid` once the wishlist item is fulfilled.
- `cover_url: Option<String>` - HTTPS URL for cover art when known.

**Methods:**

- `fn needs_html_entity_decode(self: &Self) -> bool` - Returns true when the string still contains HTML character entities.
- `fn decode_html_entities(self: & mut Self)` - Decode HTML entities in human-readable fields.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NewTitleRequest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::store::NewTitleRequestSource

*Struct*

Input for upserting a per-storefront wishlist snapshot.

**Fields:**
- `source: String` - Content-source plugin id (`audible`, `libro`, …).
- `product_id: String` - Storefront product id (ASIN, ISBN, UUID, …).
- `title: Option<String>` - Display title of the work or edition.
- `subtitle: Option<String>` - Optional subtitle from bibliographic metadata.
- `authors: Option<String>` - Comma-separated or JSON author list from the storefront.
- `narrators: Option<String>` - Comma-separated or JSON narrator list when present.
- `series: Option<String>` - Series name when the title belongs to a series.
- `series_index: Option<String>` - Position within the series (storefront string form).
- `asin: Option<String>` - Amazon ASIN when known; otherwise null.
- `isbn: Option<String>` - ISBN-10/13 when known; otherwise null.
- `description: Option<String>` - Blurb / synopsis text (may contain HTML).
- `publisher: Option<String>` - Publisher name from metadata enrichment or the storefront.
- `length_minutes: Option<i64>` - Runtime in whole minutes when the storefront reports it.
- `published_at: Option<String>` - Publication date string from the storefront or enrichment.
- `categories: Option<String>` - Storefront category / genre path list (serialized).
- `language: Option<String>` - BCP-47 or storefront language code when known.
- `cover_url: Option<String>` - HTTPS URL for cover art when known.
- `url: Option<String>` - Storefront product or purchase URL when known.
- `price_cents: Option<i64>` - Observed price in minor currency units, when known.
- `currency: Option<String>` - ISO 4217 currency code for price fields.
- `price_label: Option<String>` - Storefront-formatted price string for display.
- `list_price_cents: Option<i64>` - List/MSRP price in minor units, when known.
- `list_price_label: Option<String>` - Storefront-formatted list price for display.
- `member_price_cents: Option<i64>` - Member/subscriber price in minor units, when known.
- `member_price_label: Option<String>` - Storefront-formatted member price for display.

**Methods:**

- `fn needs_html_entity_decode(self: &Self) -> bool` - Returns true when the string still contains HTML character entities.
- `fn decode_html_entities(self: & mut Self)` - Decode HTML entities in human-readable metadata fields.

**Trait Implementations:**

- **Default**
  - `fn default() -> NewTitleRequestSource`
- **Clone**
  - `fn clone(self: &Self) -> NewTitleRequestSource`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::store::NewWork

*Struct*

Input for upserting a canonical work.

**Fields:**
- `id: Option<String>` - Surrogate primary key assigned by the database.
- `canonical_asin: Option<String>` - Preferred ASIN representing this canonical work.
- `canonical_isbn: Option<String>` - Preferred ISBN representing this canonical work.
- `title: String` - Display title of the work or edition.
- `authors: Option<String>` - Comma-separated or JSON author list from the storefront.
- `narrators: Option<String>` - Comma-separated or JSON narrator list when present.
- `description: Option<String>` - Blurb / synopsis text (may contain HTML).
- `subjects: Option<String>` - Subject / topic tags from enrichment (serialized).
- `categories: Option<String>` - Storefront category / genre path list (serialized).
- `language: Option<String>` - BCP-47 or storefront language code when known.
- `series: Option<String>` - Series name when the title belongs to a series.
- `series_index: Option<String>` - Position within the series (storefront string form).
- `cover_url: Option<String>` - HTTPS URL for cover art when known.
- `openlibrary_id: Option<String>` - Open Library work/edition id when enrichment found one.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> NewWork`
- **Clone**
  - `fn clone(self: &Self) -> NewWork`



## bookclerk_library::store::SavedFilterRecord

*Struct*

Saved Lucene-style quick filter.

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `name: String` - Human-readable or logical name for this row.
- `query: String` - Saved library filter expression (UI search DSL).
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.
- `updated_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was last modified.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SavedFilterRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::store::UserBookFields

*Struct*

Partial update for user-defined book fields.

**Fields:**
- `tags: Option<String>` - Operator or storefront tags (serialized string).
- `rating_overall: Option<f32>` - Overall user rating from the storefront, if any.
- `rating_performance: Option<f32>` - Narration/performance rating from the storefront, if any.
- `rating_story: Option<f32>` - Story rating from the storefront, if any.
- `is_finished: Option<bool>` - Whether the listener marked the title finished (0/1 or bool).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> UserBookFields`
- **Clone**
  - `fn clone(self: &Self) -> UserBookFields`



## bookclerk_library::store::WishlistIdentity

*Struct*

Bibliographic slice used for wishlist dedupe.

**Generic Parameters:**
- 'a

**Fields:**
- `work_key: &'a str` - Stable merge key used to group editions into a work.
- `title: &'a str` - Display title of the work or edition.
- `authors: Option<&'a str>` - Comma-separated or JSON author list from the storefront.
- `asin: Option<&'a str>` - Amazon ASIN when known; otherwise null.
- `isbn: Option<&'a str>` - ISBN-10/13 when known; otherwise null.

**Methods:**

- `fn matches(self: Self, other: WishlistIdentity) -> bool` - Whether two wishlist rows refer to the same work despite differing keys.

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> WishlistIdentity<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::store::fallback_work_key

*Function*

Local fallback when callers do not supply a discover `work_map_key`.

Converts ISBN-10 → ISBN-13 when possible so 10/13 variants share a key.
Soft keys here are a simple lowercase fallback — Discover re-merges with
richer identity matching when ranking the global queue.

```rust
fn fallback_work_key(title: &str, authors: Option<&str>, asin: Option<&str>, isbn: Option<&str>) -> String
```



## bookclerk_library::store::prefer_enrichment_source

*Function*

Prefer an Audible ownership row with the richest metadata for enrichment.

```rust
fn prefer_enrichment_source(books: &[crate::models::BookRecord]) -> Option<&crate::models::BookRecord>
```



## bookclerk_library::store::wishlist_identities_match

*Function*

Whether two wishlist rows refer to the same work despite differing keys.

```rust
fn wishlist_identities_match(a: WishlistIdentity, b: WishlistIdentity) -> bool
```



