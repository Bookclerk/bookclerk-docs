# bookclerk_library

Canonical Bookclerk library database.

[`LibraryStore`](#librarystore) is SeaORM-backed: every backend is a
[`sea_orm::DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html). Engine connect/migrate/proxy quirks live in
the database plugin (`bookclerk-plugin-database-sqlite` (and optional d1/postgres guests)); hosts open the store via
the external guest RPC. This crate owns the greenfield schema
([`migrations`](migrations.md)), typed [`entities`](entities.md), and the store API. See
`docs/database.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AtomicTxnBackend`](#atomictxnbackend) | trait | Backend that runs `crate::LibraryStore` named security operations as a single guest atomic batch. |
| [`TypedAtomicExec`](#typedatomicexec) | trait | Runs a host-authorized typed `bookclerk_plugin_abi::ExecuteRequest` as one guest `executeAtomic`. |
| [`DbAtomicResult`](#dbatomicresult) | struct | Host-interpreted result of a named atomic operation. |
| [`BackendMigrateOptions`](#backendmigrateoptions) | struct | Options for a backend-to-backend library copy. |
| [`BackendMigrateSummary`](#backendmigratesummary) | struct | Per-table row counts copied (or that would be copied in dry-run mode). |
| [`BackupRepository`](#backuprepository) | struct | On-disk backup repository rooted at `files_dir/backups`. |
| [`BackupListEntry`](#backuplistentry) | struct | Catalog entry used by `bookclerk db backup list`. |
| [`BackupManifest`](#backupmanifest) | struct | On-disk recovery-point metadata (`manifests/<id>.json`). |
| [`BackupOutcome`](#backupoutcome) | struct | Result of publishing a recovery point. |
| [`BackupRequest`](#backuprequest) | struct | Inputs for an automatic or CLI backup. |
| [`BackupUnit`](#backupunit) | struct | One logical database in a recovery point (library or plugin binding). |
| [`CanonicalDatabaseSchema`](#canonicaldatabaseschema) | struct | Admitted canonical schema used to export or restore one database unit. |
| [`CanonicalExportOpts`](#canonicalexportopts) | struct | Options for a consistent canonical export. |
| [`CanonicalRestoreOpts`](#canonicalrestoreopts) | struct | Negotiated limits for parameterized canonical restore. |
| [`CanonicalTableSchema`](#canonicaltableschema) | struct | One admitted `CREATE TABLE` plus its parsed IR. |
| [`IdentityHighWater`](#identityhighwater) | struct | Identity high-water for one generated column. |
| [`PreparedPluginUnit`](#preparedpluginunit) | struct | Open plugin binding captured through the active adapter session. |
| [`RestorePlan`](#restoreplan) | struct | Library restore result plus plugin units for the host to apply. |
| [`SchemaBackupOpts`](#schemabackupopts) | struct | Optional in-place backup taken before applying ups or downs. |
| [`ValidatedBackup`](#validatedbackup) | struct | Recovery point whose objects, schema, and typed rows have been verified. |
| [`SchemaApplyOptions`](#schemaapplyoptions) | struct | Options for `apply_host_schema_with_options`. |
| [`SchemaBatch`](#schemabatch) | struct | Canonical schema apply batch: already-separate host statements plus marker. |
| [`InProcessSqliteAtomic`](#inprocesssqliteatomic) | struct | Compiles named ops and runs them on a local SeaORM SQLite connection. |
| [`PluginMigrationHistory`](#pluginmigrationhistory) | struct | Ordered durable plugin migration history. |
| [`PluginMigrationSequence`](#pluginmigrationsequence) | struct | Proven, ordered plugin migration sequence. |
| [`ProvenPluginMigration`](#provenpluginmigration) | struct | One proven registered migration. |
| [`AccountLinkRecord`](#accountlinkrecord) | struct | Link between a portal identity and a bookstore account. |
| [`AccountRecord`](#accountrecord) | struct | Account row stored in the Bookclerk DB. |
| [`BookRecord`](#bookrecord) | struct | Book / library item (one ownership row per store product per account). |
| [`ClaimTicketRecord`](#claimticketrecord) | struct | Claim ticket metadata (token plaintext is never stored). |
| [`DomainEventRecord`](#domaineventrecord) | struct | Immutable outbox envelope. |
| [`EmbeddingRecord`](#embeddingrecord) | struct | Stored embedding vector metadata (blob fetched separately when needed). |
| [`EnqueueJobSpec`](#enqueuejobspec) | struct | Admission request for `crate::LibraryStore::enqueue_job`. |
| [`EventCatalogSubscription`](#eventcatalogsubscription) | struct | One `[capabilities.events.subscriptions]` row persisted in `event_subscriber_nodes`. |
| [`EventDeliveryFence`](#eventdeliveryfence) | struct | Claim fence for an event delivery (mirrors `JobFence`). |
| [`EventDeliveryMetrics`](#eventdeliverymetrics) | struct | Operator-visible delivery-queue counters for `GET /api/status`. |
| [`EventDeliveryRecord`](#eventdeliveryrecord) | struct | Per-subscriber delivery row. |
| [`EventSubscriber`](#eventsubscriber) | struct | Subscriber snapshot used when creating deliveries for one event. |
| [`EventSubscriberCatalogRecord`](#eventsubscribercatalogrecord) | struct | Cluster-authoritative catalog row after collapsing live nodes by `plugin_id`. |
| [`EventSubscriberNodeRecord`](#eventsubscribernoderecord) | struct | Live per-node catalog row (`event_subscriber_nodes`). |
| [`EventWakeGrant`](#eventwakegrant) | struct | Host-derived wake grant: schema versions plus an intersected payload filter. |
| [`GlobalQueueEntry`](#globalqueueentry) | struct | Aggregated global request-queue entry (one work, many wishers). |
| [`JobClaimCursor`](#jobclaimcursor) | struct | Where the next page of routed-claim candidates starts (keyset paging). |
| [`JobClaimPage`](#jobclaimpage) | struct | One page of routed-claim candidates. |
| [`JobFence`](#jobfence) | struct | Lease identity a worker must present to mutate a running job. |
| [`JobPayload`](#jobpayload) | struct | JSON command envelope stored on a job row. |
| [`JobRecord`](#jobrecord) | struct | Durable job row returned to the daemon API and worker. |
| [`JobTempPath`](#jobtemppath) | struct | Scratch path registered against a job for crash cleanup. |
| [`ListeningProgressRecord`](#listeningprogressrecord) | struct | Listening progress snapshot from an external player (e.g. AudioBookshelf). |
| [`OidcClientRecord`](#oidcclientrecord) | struct | Registered OIDC authorization-server client (Bookclerk as IdP). |
| [`OperatorSessionRecord`](#operatorsessionrecord) | struct | Durable operator session metadata (hashed token is never exposed). |
| [`PendingWakeProgress`](#pendingwakeprogress) | struct | How far one claimed wake tick got. |
| [`PortalIdentity`](#portalidentity) | struct | Portal identity bound to an external provider user (e.g. ABS). |
| [`PortalSessionRecord`](#portalsessionrecord) | struct | Portal session row for session lists (includes hash for current-session match). |
| [`PublishDomainEventSpec`](#publishdomaineventspec) | struct | Producer-side domain event (or queue message) to persist in the outbox. |
| [`QueueWisher`](#queuewisher) | struct | Compact wisher shown on the global queue (avatar + label). |
| [`SecurityAuditEvent`](#securityauditevent) | struct | Security audit event (elevate / impersonate / login / provision). |
| [`StoredPasskey`](#storedpasskey) | struct | Stored WebAuthn credential (passkey JSON plus the label shown in Settings). |
| [`TitleRequestRecord`](#titlerequestrecord) | struct | Personal wishlist row (also contributes to the shared global queue while open). |
| [`TitleRequestSourceRecord`](#titlerequestsourcerecord) | struct | Per-storefront catalog/pricing snapshot attached to a wishlist row. |
| [`UserIntegrationHint`](#userintegrationhint) | struct | Storefront / integration connection summary for admin user lists. |
| [`UserInviteRecord`](#userinviterecord) | struct | Invite ticket for provisioning a User (token plaintext never stored). |
| [`UserListeningHint`](#userlisteninghint) | struct | Recent unfinished listening hint for admin user lists. |
| [`UserPreferences`](#userpreferences) | struct | Per-user GUI / Discover preferences (operator or portal identity). |
| [`UserPresenceExtras`](#userpresenceextras) | struct | Presence + connection extras for administrator user management. |
| [`UserRecord`](#userrecord) | struct | First-party Bookclerk user (security principal for portal paths). |
| [`UserSsoPicture`](#userssopicture) | struct | IdP-supplied avatar available as a profile-picture choice. |
| [`WishlistPurchaseHint`](#wishlistpurchasehint) | struct | Snapshotted purchase link/price for a wishlist title. |
| [`WishlistStoreEdition`](#wishliststoreedition) | struct | Storefront edition key for wishlist / queue payloads. |
| [`WorkRecord`](#workrecord) | struct | Canonical work spanning one or more ownership rows / editions. |
| [`SchemaWalk`](#schemawalk) | struct | Result of walking a host migration plan from one version to another. |
| [`SessionClientInfo`](#sessionclientinfo) | struct | Parsed client presentation for a session row. |
| [`PluginDatabaseRecord`](#plugindatabaserecord) | struct | One provisioned plugin database binding registry row. |
| [`KvEntry`](#kventry) | struct | One live key/value row. |
| [`KvListPage`](#kvlistpage) | struct | One page of `LibraryStore::kv_list`. |
| [`KvWriteSpec`](#kvwritespec) | struct | A write plus the namespace limits it must satisfy. |
| [`BookPage`](#bookpage) | struct | One page of books plus the unpaged match count. |
| [`CatalogEnrichmentFields`](#catalogenrichmentfields) | struct | Durable catalog enrichment fields (blurbs, subjects, provenance). |
| [`LibraryStore`](#librarystore) | struct | Handle to the Bookclerk library database. |
| [`NewBook`](#newbook) | struct | Input for inserting / updating a book from sync. |
| [`NewListeningProgress`](#newlisteningprogress) | struct | Input for upserting listening progress. |
| [`NewTitleRequest`](#newtitlerequest) | struct | Input for creating a title request / wishlist row. |
| [`NewTitleRequestSource`](#newtitlerequestsource) | struct | Input for upserting a per-storefront wishlist snapshot. |
| [`NewWork`](#newwork) | struct | Input for upserting a canonical work. |
| [`SavedFilterRecord`](#savedfilterrecord) | struct | Saved Lucene-style quick filter. |
| [`TitleFile`](#titlefile) | struct | Who holds a recorded file key. |
| [`UserBookFields`](#userbookfields) | struct | Partial update for user-defined book fields. |
| [`WishlistIdentity`](#wishlistidentity) | struct | Bibliographic slice used for wishlist dedupe. |
| [`DbAtomicParams`](#dbatomicparams) | enum | Named atomic library operation compiled by the host into a generic plan. |
| [`CanonicalObject`](#canonicalobject) | enum | One immutable canonical object (hashed before compression). |
| [`BackupReason`](#backupreason) | enum | Why a backup was written. |
| [`BackupResolve`](#backupresolve) | enum | How `resolve_backup_spec` located a recovery point. |
| [`CanonicalRestoreKind`](#canonicalrestorekind) | enum | How restore applies DDL companions. |
| [`DatabaseUnitKind`](#databaseunitkind) | enum | Logical kind of one database unit in a recovery point. |
| [`LibraryError`](#libraryerror) | enum | Failures from library store, schema apply, secrets, or config. |
| [`AcquireStatus`](#acquirestatus) | enum | Acquire / download state for a title. |
| [`EnqueueOutcome`](#enqueueoutcome) | enum | Result of admitting a job into the durable queue. |
| [`JobKind`](#jobkind) | enum | Kind of durable daemon work stored in the `jobs` table. |
| [`JobResourceClass`](#jobresourceclass) | enum | Concurrency class that bounds how many jobs of this type run at once. |
| [`JobState`](#jobstate) | enum | Lifecycle state of a durable job row. |
| [`JobTrigger`](#jobtrigger) | enum | Who admitted the job (API vs periodic scheduler). |
| [`PublishDomainEventOutcome`](#publishdomaineventoutcome) | enum | Result of `crate::LibraryStore::publish_domain_event`. |
| [`RequestStatus`](#requeststatus) | enum | Wishlist row status (`open` while wishlisted; `cancelled` after un-wishlist). |
| [`UserRole`](#userrole) | enum | First-party user role (`owner` \| `administrator` \| `host` \| `member`). |
| [`UserStatus`](#userstatus) | enum | First-party user status. |
| [`SchemaState`](#schemastate) | enum | Durable Bookclerk library schema state. |
| [`EventPruneStep`](#eventprunestep) | enum | One delete of `LibraryStore::prune_event_deliveries`, in run order. |
| [`KvPutOutcome`](#kvputoutcome) | enum | Why a write was refused, so the binding can map it onto a wire code. |
| [`migrate_library_backend`](#migrate_library_backend) | function | Copy all library tables from `source` into `dest`. |
| [`restore_backup_unit`](#restore_backup_unit) | function | Destructively replaces one logical database with `unit`. Never merges. |
| [`library_ddl_for_schema_state`](#library_ddl_for_schema_state) | function | Library DDL that matches `state`, not necessarily this binary's latest pack. |
| [`sort_tables_by_foreign_keys`](#sort_tables_by_foreign_keys) | function | FK-safe table order (parents before children). Cycles fail closed. |
| [`verify_recovery_point`](#verify_recovery_point) | function | Parses the manifest, rejects unsupported formats, and verifies every referenced object **before** the caller starts a destructive restore. |
| [`admit_canonical_schema`](#admit_canonical_schema) | function | Parses and fully admits a diagnostic/imported canonical SQL script. |
| [`admit_canonical_statements`](#admit_canonical_statements) | function | Admits already-separated canonical `CREATE TABLE` / `CREATE INDEX` statements. |
| [`apply_admitted_sql`](#apply_admitted_sql) | function | Applies admitted canonical DDL plus binding companions (tests / bootstrap). |
| [`archive_backup`](#archive_backup) | function | Packages one recovery point (manifest + referenced objects) as `.tar.gz`. |
| [`backup_library`](#backup_library) | function | Writes a crash-safe recovery point under `files_dir/backups/`. |
| [`extract_backup_archive`](#extract_backup_archive) | function | Extracts a `.tar.gz` recovery-point archive into `dest` (a backups root). |
| [`filter_library_pack_ddl`](#filter_library_pack_ddl) | function | Library migration packs may include seed `INSERT`s. Those are table data (captured from rows), not schema objects. This keeps admitted `CREATE TABLE` / `CREATE INDEX` and skips seed DML. Any other statement fails closed. |
| [`library_canonical_schema`](#library_canonical_schema) | function | Admitted library schema for this binary's current pack (frozen ups + unreleased). |
| [`library_canonical_schema_for_state`](#library_canonical_schema_for_state) | function | Admitted library schema for `state`. |
| [`list_backups`](#list_backups) | function | Lists published recovery points oldest-first. |
| [`plugin_canonical_schema_from_ddl_catalog`](#plugin_canonical_schema_from_ddl_catalog) | function | Rebuilds plugin schema from durable `bookclerk_sql_ddl` on an open connection. |
| [`prune_automatic_backups`](#prune_automatic_backups) | function | Deletes oldest automatic pre-migrate recovery points. Never deletes `manual`. |
| [`resolve_backup_spec`](#resolve_backup_spec) | function | Resolves a recovery-point id, timestamp prefix, or archive path. |
| [`restore_backup`](#restore_backup) | function | Restores the library unit after verifying **all** objects and units. |
| [`restore_backup_in_repo`](#restore_backup_in_repo) | function | Restores from an already-open repository (extracted archives). |
| [`apply_binding_bootstrap`](#apply_binding_bootstrap) | function | Applies host-owned binding bootstrap when the binding is uninitialized. |
| [`binding_bootstrap_plan`](#binding_bootstrap_plan) | function | Statements to apply for `state`, or `None` when the binding already matches. |
| [`db_atomic_operation_id`](#db_atomic_operation_id) | function | Caller-owned idempotency key for an atomic attempt. |
| [`db_atomic_request_hash`](#db_atomic_request_hash) | function | SHA-256 hex digest of the idempotency-relevant fields of `op`. |
| [`execute_db_atomic`](#execute_db_atomic) | function | Runs a compiled named atomic as one native SQL transaction. |
| [`execute_named_atomic`](#execute_named_atomic) | function | Compiles `params` and runs the plan as one native SQL transaction. |
| [`apply_host_schema`](#apply_host_schema) | function | Applies pending host-authored DDL from the canonical migration plan. |
| [`apply_host_schema_with_batch`](#apply_host_schema_with_batch) | function | Applies schema using `run_batch` (typed `executeAtomic`) for each version. |
| [`apply_host_schema_with_batch_opts`](#apply_host_schema_with_batch_opts) | function | Applies schema with backups using `run_batch` for each version. |
| [`apply_host_schema_with_options`](#apply_host_schema_with_options) | function | Applies pending host schema with optional in-place backups. |
| [`current_schema_state`](#current_schema_state) | function | Reads explicit `SchemaState` in the Bookclerk host/bootstrap namespace. |
| [`current_schema_state_in`](#current_schema_state_in) | function | Reads explicit `SchemaState` for one ledger `namespace`. |
| [`current_schema_version`](#current_schema_version) | function | Frozen revision when the database is `SchemaState::Frozen`; otherwise `0`. |
| [`ensure_restore_target_is_replaceable`](#ensure_restore_target_is_replaceable) | function | Fails closed when this binary cannot interpret the target database's schema history well enough to drop every unknown table before restore. |
| [`migrate_host_schema_to`](#migrate_host_schema_to) | function | Migrates toward `target`, including reversible downs for CLI rollback. |
| [`migrate_host_schema_to_with_batch`](#migrate_host_schema_to_with_batch) | function | Migrates toward `target` using `run_batch` (typed `executeAtomic`) per version. |
| [`require_schema_migrations`](#require_schema_migrations) | function | Requires the `schemaMigrations` versioning contract from typed `bookclerk_plugin_abi::DbCapabilities`. |
| [`schema_state_compatible`](#schema_state_compatible) | function | True when `state` is exactly the schema this binary applies: a start against it neither migrates nor fails closed. |
| [`apply_plugin_migrations`](#apply_plugin_migrations) | function | Applies the pending registered suffix. Idempotent when history already matches. |
| [`history_from_execute_reply`](#history_from_execute_reply) | function | Parses journal rows from a typed select reply. |
| [`load_plugin_migration_history`](#load_plugin_migration_history) | function | Reads the host-private plugin migration journal. |
| [`next_pending_plugin_migration`](#next_pending_plugin_migration) | function | Next registered migration after a verified durable prefix, if any. |
| [`plugin_apply_statements`](#plugin_apply_statements) | function | One atomic apply unit: serialization slot, registered ops, host journal append. |
| [`plugin_binding_type_env`](#plugin_binding_type_env) | function | Binding catalog after host bootstrap plus schema ops from the first `applied` registered migrations. |
| [`plugin_history_session_matches`](#plugin_history_session_matches) | function | True when a binding session's captured history digest still matches durable history. |
| [`plugin_journal_has_entry`](#plugin_journal_has_entry) | function | True when `history` already contains this `(ordinal, id, checksum)` row. |
| [`plugin_journal_select_request`](#plugin_journal_select_request) | function | Typed SELECT used by RPC hosts to read the plugin migration journal. |
| [`prove_plugin_migration_sequence`](#prove_plugin_migration_sequence) | function | Proves a complete registration with one evolving `SqlTypeEnv`. |
| [`require_history_prefix`](#require_history_prefix) | function | Fail closed unless durable history is an exact prefix of `registered`. |
| [`sql_string_literal`](#sql_string_literal) | function | SQL string literal with `'` doubled. |
| [`catalog_subscribers_for_event`](#catalog_subscribers_for_event) | function | Enabled catalog rows that match `event` type, schema version, and filter. |
| [`collapse_live_subscriber_nodes`](#collapse_live_subscriber_nodes) | function | Collapse live per-node rows into one catalog record per `plugin_id`. |
| [`content_kind_from_classic`](#content_kind_from_classic) | function | Map classic EF `ContentType` to bookclerk `content_kind`. |
| [`content_kind_to_classic`](#content_kind_to_classic) | function | Classic EF `ContentType` enum value. |
| [`event_filter_matches`](#event_filter_matches) | function | True when `filter` is absent/empty or every filter key equals the payload object value. |
| [`event_matches_wake_grants`](#event_matches_wake_grants) | function | True when an event matches persisted wake grants, or legacy type+filter rows. |
| [`intersect_event_filters`](#intersect_event_filters) | function | Merge declared and requested payload filters; `None` on a same-key conflict. |
| [`is_downloadable`](#is_downloadable) | function | True when the title can be downloaded (classic `WithoutParents`). |
| [`is_episode`](#is_episode) | function | True when `content_kind` is a podcast episode. |
| [`is_podcast_parent`](#is_podcast_parent) | function | True when `content_kind` is a podcast parent / show (no audio to acquire). |
| [`is_valid_queue_name`](#is_valid_queue_name) | function | True when `name` is `a-z0-9*` and at most `MAX_QUEUE_NAME_LEN` chars. |
| [`job_backoff_run_after`](#job_backoff_run_after) | function | Next `run_after` after a failed attempt, exponential from a 5s base (capped). |
| [`normalize_theme`](#normalize_theme) | function | Canonical appearance preference; unknown values become `system`. |
| [`portal_prefs_key`](#portal_prefs_key) | function | Subject key for a portal identity (legacy; prefer `user_prefs_key`). |
| [`push_queue_wisher`](#push_queue_wisher) | function | Adds `wisher` when they are not already present (capped at `MAX_QUEUE_WISHERS`). |
| [`queue_name_or_bus`](#queue_name_or_bus) | function | Resolves an optional queue name: empty means `CORE_EVENTS_QUEUE`. |
| [`subscription_matches_event`](#subscription_matches_event) | function | True when one catalog subscription matches `event` type, schema, class, and filter. |
| [`user_prefs_key`](#user_prefs_key) | function | Subject key for a first-party user. |
| [`wake_grants_from_subscriptions`](#wake_grants_from_subscriptions) | function | Build wake grants from declared subscriptions for `wake_type`. |
| [`plan_schema_walk`](#plan_schema_walk) | function | Plans ups or downs from `from` toward `to` using `plan`. |
| [`plan_schema_walk_from_state`](#plan_schema_walk_from_state) | function | Plans a frozen walk from an explicit `SchemaState`. |
| [`classify_session_client`](#classify_session_client) | function | Classify a User-Agent (or missing UA for API/bearer clients). |
| [`compile_claim_event_delivery`](#compile_claim_event_delivery) | function | Compiles a host-side CAS claim for one pending `event_deliveries` row. |
| [`compile_named_request`](#compile_named_request) | function | Compiles a named atomic request into canonical Bookclerk SQL. |
| [`interpret_typed_exec`](#interpret_typed_exec) | function | Maps a typed `ExecuteReply` onto a host `DbAtomicResult`. |
| [`validate_execute_reply`](#validate_execute_reply) | function | Rejects a guest `ExecuteReply` that does not echo the sent request or exceeds negotiated caps. |
| [`deliver_after_for`](#deliver_after_for) | function | RFC 3339 earliest delivery time for `spec`; empty when it may deliver now. |
| [`prepare_publish_domain_event`](#prepare_publish_domain_event) | function | Mint an event id when empty and reject oversized or blank publish fields. |
| [`fallback_work_key`](#fallback_work_key) | function | Local fallback when callers do not supply a discover `work_map_key`. |
| [`inject_dispatch_page_failures`](#inject_dispatch_page_failures) | function | Fail after the next `n` successful dispatch pages (tests only). |
| [`prefer_enrichment_source`](#prefer_enrichment_source) | function | Prefer an Audible ownership row with the richest metadata for enrichment. |
| [`set_dispatch_chunk_for_test`](#set_dispatch_chunk_for_test) | function | Override subscribers-per-plan for dispatch paging tests. |
| [`wishlist_identities_match`](#wishlist_identities_match) | function | Whether two wishlist rows refer to the same work despite differing keys. |
| [`decode_html_entities`](#decode_html_entities) | function | Owned decode (allocates even when unchanged — prefer `decode_html_entities_cow` or `decode_html_entities_in_place` on hot paths). |
| [`decode_html_entities_cow`](#decode_html_entities_cow) | function | Decode HTML entities in storefront metadata (`Memory&#39;s Blade` → `Memory's Blade`). |
| [`decode_html_entities_in_place`](#decode_html_entities_in_place) | function | Decode in place when the value contains `&`; otherwise leave untouched. |
| [`decode_html_entities_opt`](#decode_html_entities_opt) | function | Decode when present; leave `None` unchanged. |
| [`decode_html_entities_opt_in_place`](#decode_html_entities_opt_in_place) | function | Decode an optional string in place. |
| [`str_maybe_html_entity`](#str_maybe_html_entity) | function | True when any human-readable field may contain an HTML entity. |
| [`derive_claim_password_fingerprint`](#derive_claim_password_fingerprint) | function | Stable, non-secret fingerprint of an invite/reset password for idempotency. |
| [`derive_claim_session_token`](#derive_claim_session_token) | function | Derive the portal session token for a claim ticket plus browser nonce. |
| [`hash_token`](#hash_token) | function | SHA-256 hex digest of a raw ticket/session token for durable storage. |
| [`parse_claim_redeem_nonce`](#parse_claim_redeem_nonce) | function | Accept a browser-generated claim-redeem nonce. |
| [`apply_merged_sources`](#apply_merged_sources) | function | Apply merged bib / editions / hints onto a wishlist row from its source snapshots. |
| [`pick_better_description`](#pick_better_description) | function | Prefer HTML blurbs over plain text; otherwise the longer string wins. |
| [`Result`](#result) | type alias | Result alias for `LibraryError`. |
| [`CHUNK_TARGET_UNCOMPRESSED_BYTES`](#chunk_target_uncompressed_bytes) | constant | Target uncompressed JSON size for one table-data chunk. |
| [`BACKUPS_DIR`](#backups_dir) | constant | Subdirectory of `$BOOKCLERK_FILES_DIR` holding the backup repository. |
| [`BACKUP_FORMAT_VERSION`](#backup_format_version) | constant | Manifest format written by this binary. |
| [`BACKUP_RETENTION`](#backup_retention) | constant | Automatic `pre-migrate` recovery points retained after prune. |
| [`LIBRARY_SKIP_TABLES`](#library_skip_tables) | constant | Library registry rows are environment-local (`unit_ref` is not portable). |
| [`MIGRATION_LEASE_TTL`](#migration_lease_ttl) | constant | How long a schema migration lease lasts without renewal. The migrator renews it every third of this while it works. |
| [`MIGRATION_LEASE_WAIT`](#migration_lease_wait) | constant | How long a starter waits for another starter's migration before it fails closed. Longer than `MIGRATION_LEASE_TTL`, so a migrator that died is replaced by a waiter rather than failing every waiter. |
| [`BOOKCLERK_SCHEMA_NAMESPACE`](#bookclerk_schema_namespace) | constant | Ledger namespace for host library schema and Bookclerk-owned binding bootstrap. |
| [`MAX_PLUGIN_MIGRATION_APPLY_ATTEMPTS`](#max_plugin_migration_apply_attempts) | constant | Bounded retries for **one** expected plugin migration apply unit. |
| [`CORE_EVENTS_QUEUE`](#core_events_queue) | constant | Queue that carries the domain-event bus (`[events]` producers and consumers). |
| [`EVENT_DELIVERY_MAX_ATTEMPTS`](#event_delivery_max_attempts) | constant | Default max delivery attempts before dead-letter. |
| [`EVENT_RESOURCE_CLASS_NETWORK`](#event_resource_class_network) | constant | Default delivery resource class (`network`). |
| [`EVENT_SUBSCRIBER_HEARTBEAT_TTL_SECS`](#event_subscriber_heartbeat_ttl_secs) | constant | Live catalog rows quieter than this many seconds are ignored at dispatch. |
| [`JOB_PAYLOAD_VERSION`](#job_payload_version) | constant | Current job command envelope version stored in `JobPayload::v`. |
| [`MAX_QUEUE_NAME_LEN`](#max_queue_name_len) | constant | Longest queue name. |
| [`MAX_QUEUE_WISHERS`](#max_queue_wishers) | constant | Maximum distinct people serialized on a global-queue or Discover card. |
| [`NODE_QUEUE_PREFIX`](#node_queue_prefix) | constant | Prefix of the per-node queues (`node.<node_id>`) node subsystems consume. |
| [`OPERATOR_PREFS_KEY`](#operator_prefs_key) | constant | Subject key for the shared operator account. |
| [`QUEUE_CONTENT_TYPE_BYTES`](#queue_content_type_bytes) | constant | Queue message body is raw bytes stored as base64 text. |
| [`QUEUE_CONTENT_TYPE_JSON`](#queue_content_type_json) | constant | Queue message body is UTF-8 JSON text. |
| [`QUEUE_CONTENT_TYPE_TEXT`](#queue_content_type_text) | constant | Queue message body is plain UTF-8 text. |
| [`SCHEMA_STATE_FROZEN`](#schema_state_frozen) | constant | On-disk `bookclerk_schema_migrations.state` for a frozen plan version. |
| [`SCHEMA_STATE_UNRELEASED`](#schema_state_unreleased) | constant | On-disk `bookclerk_schema_migrations.state` for an unreleased development pack. |
| [`KV_LIST_DEFAULT_LIMIT`](#kv_list_default_limit) | constant | Default page size for `LibraryStore::kv_list` when the caller passes 0. |
| [`ACQUIRE_WRITER`](#acquire_writer) | constant | The `writer` of a key an acquire run on this node wrote. |
| [`BOOK_PAGE_DEFAULT_LIMIT`](#book_page_default_limit) | constant | Default page size for `LibraryStore::list_books_filtered_page`. |
| [`BOOK_PAGE_MAX_LIMIT`](#book_page_max_limit) | constant | Upper clamp for a library book page. Matches the HTTP handler. |
| [`CLAIM_REDEEM_NONCE_HEX_LEN`](#claim_redeem_nonce_hex_len) | constant | Hex length of a browser-generated claim-redeem nonce (32 bytes). |

## Re-exports

- [`bookclerk_plugin_abi::AdapterBackupOps`](../bookclerk_plugin_abi/backup_ops.md#adapterbackupops)
- [`bookclerk_plugin_abi::GuestSqlPolicy`](../bookclerk_plugin_abi/guest_sql.md#guestsqlpolicy)
- [`bookclerk_plugin_abi::SharedAdapterBackupOps`](../bookclerk_plugin_abi/backup_ops.md#sharedadapterbackupops)
- [`email::gravatar_hash`](email.md#gravatar_hash)
- [`master_key::configure_master_key`](master_key.md#configure_master_key)
- [`master_key::configure_master_key_with`](master_key.md#configure_master_key_with)
- [`master_key::discard_minted_master_key`](master_key.md#discard_minted_master_key)
- [`master_key::encode_master_key_file`](master_key.md#encode_master_key_file)
- [`master_key::inspect_master_key`](master_key.md#inspect_master_key)
- [`master_key::install_master_key`](master_key.md#install_master_key)
- [`master_key::master_key_fingerprint`](master_key.md#master_key_fingerprint)
- [`master_key::master_key_path`](master_key.md#master_key_path)
- [`master_key::prepare_master_key_dir`](master_key.md#prepare_master_key_dir)
- [`master_key::require_master_key`](master_key.md#require_master_key)
- [`master_key::resolve_master_key_detailed`](master_key.md#resolve_master_key_detailed)
- [`master_key::uncache_master_key`](master_key.md#uncache_master_key)
- [`master_key::warn_unwrapped_master_key`](master_key.md#warn_unwrapped_master_key)
- [`master_key::wrap_master_key`](master_key.md#wrap_master_key)
- [`master_key::MasterKey`](master_key.md#masterkey)
- [`master_key::MasterKeyFormat`](master_key.md#masterkeyformat)
- [`master_key::MasterKeyResolution`](master_key.md#masterkeyresolution)
- [`master_key::AUTH_PASSWORD_ENV` as `MASTER_KEY_AUTH_PASSWORD_ENV`](master_key.md#auth_password_env)
- [`migrations::binding_bootstrap_type_env`](migrations.md#binding_bootstrap_type_env)
- [`migrations::host_migration_plan`](migrations.md#host_migration_plan)
- [`migrations::min_supported_schema_version`](migrations.md#min_supported_schema_version)
- [`migrations::CLUSTER_LEASES_DDL`](migrations.md#cluster_leases_ddl)
- [`migrations::SCHEMA_MIGRATIONS_DDL`](migrations.md#schema_migrations_ddl)
- [`migrations::SCHEMA_VERSION`](migrations.md#schema_version)
- [`operator_token::read_or_create_operator_token`](operator_token.md#read_or_create_operator_token)
- [`operator_token::resolve_operator_token`](operator_token.md#resolve_operator_token)
- [`operator_token::rotate_operator_token`](operator_token.md#rotate_operator_token)
- [`operator_token::ResolveOperatorToken`](operator_token.md#resolveoperatortoken)
- [`password::hash_password`](password.md#hash_password)
- [`password::verify_password`](password.md#verify_password)
- [`proxy_txn::consume_begin_injection`](../bookclerk_db_exec/proxy_txn.md#consume_begin_injection)
- [`proxy_txn::consume_commit_injection`](../bookclerk_db_exec/proxy_txn.md#consume_commit_injection)
- [`proxy_txn::current_exec_budget`](../bookclerk_db_exec/proxy_txn.md#current_exec_budget)
- [`proxy_txn::inject_atomic_interrupt`](../bookclerk_db_exec/proxy_txn.md#inject_atomic_interrupt)
- [`proxy_txn::inject_atomic_interrupt_after`](../bookclerk_db_exec/proxy_txn.md#inject_atomic_interrupt_after)
- [`proxy_txn::inject_begin_failures`](../bookclerk_db_exec/proxy_txn.md#inject_begin_failures)
- [`proxy_txn::inject_commit_failures`](../bookclerk_db_exec/proxy_txn.md#inject_commit_failures)
- [`proxy_txn::inject_savepoint_release_failures`](../bookclerk_db_exec/proxy_txn.md#inject_savepoint_release_failures)
- [`proxy_txn::inject_savepoint_rollback_failures`](../bookclerk_db_exec/proxy_txn.md#inject_savepoint_rollback_failures)
- [`proxy_txn::is_txn_broken`](../bookclerk_db_exec/proxy_txn.md#is_txn_broken)
- [`proxy_txn::note_begin_failed`](../bookclerk_db_exec/proxy_txn.md#note_begin_failed)
- [`proxy_txn::note_commit_failed`](../bookclerk_db_exec/proxy_txn.md#note_commit_failed)
- [`proxy_txn::query_rows_seen`](../bookclerk_db_exec/proxy_txn.md#query_rows_seen)
- [`proxy_txn::take_txn_fault`](../bookclerk_db_exec/proxy_txn.md#take_txn_fault)
- [`proxy_txn::txn_broken_err`](../bookclerk_db_exec/proxy_txn.md#txn_broken_err)
- [`proxy_txn::AtomicInterruptKind`](../bookclerk_db_exec/proxy_txn.md#atomicinterruptkind)
- [`proxy_txn::AtomicInterruptPhase`](../bookclerk_db_exec/proxy_txn.md#atomicinterruptphase)
- [`proxy_txn::ExecBudget`](../bookclerk_db_exec/proxy_txn.md#execbudget)
- [`scope::SourceScope`](scope.md#sourcescope)
- [`secrets::b64_string_to_bytes`](secrets.md#b64_string_to_bytes)
- [`secrets::build_sealed_record`](secrets.md#build_sealed_record)
- [`secrets::bytes_to_b64_string`](secrets.md#bytes_to_b64_string)
- [`secrets::delete_secret`](secrets.md#delete_secret)
- [`secrets::delete_secrets_for_account`](secrets.md#delete_secrets_for_account)
- [`secrets::get_secret`](secrets.md#get_secret)
- [`secrets::list_secrets`](secrets.md#list_secrets)
- [`secrets::unseal_secret`](secrets.md#unseal_secret)
- [`secrets::upsert_secret`](secrets.md#upsert_secret)
- [`secrets::EncryptedSecretRecord`](secrets.md#encryptedsecretrecord)
- [`secrets::SecretStore`](secrets.md#secretstore)
- [`secrets::FORMAT_SEALED_V1`](secrets.md#format_sealed_v1)
- [`sql_plan::authorize_typed_request`](sql_plan.md#authorize_typed_request)
- [`sql_plan::execute_guest_atomic_with`](sql_plan.md#execute_guest_atomic_with)
- [`sql_plan::proxy_insert_returning_identity`](sql_plan.md#proxy_insert_returning_identity)
- [`sql_plan::proxy_read_kind`](sql_plan.md#proxy_read_kind)
- [`sql_plan::proxy_write_kind`](sql_plan.md#proxy_write_kind)
- [`sql_plan::validate_plan`](sql_plan.md#validate_plan)
- [`sql_plan::wake_page_for_max_binds`](sql_plan.md#wake_page_for_max_binds)
- [`sql_plan::AtomicSession`](../bookclerk_db_exec/index.md#atomicsession)
- [`sql_plan::CompiledAtomic`](sql_plan.md#compiledatomic)

## Modules

- [`control_plane`](control_plane.md): Database-backed control plane: node identity and typed configuration.
- [`email`](email.md): Contact-email normalization and library-backed validation.
- [`entities`](entities.md): SeaORM entities mirroring the greenfield schema in `crate::migrations`.
- [`master_key`](master_key.md): Data-encryption key (DEK) for `encrypted_secrets`.
- [`migrations`](migrations.md): Host library schema: frozen plan steps plus the unreleased development pack.
- [`operator_token`](operator_token.md): Durable operator API token in `encrypted_secrets`.
- [`password`](password.md): Argon2id password hashing for first-party local login.
- [`proxy_txn`](proxy_txn.md): Fail-closed transaction injects re-exported from `bookclerk_db_exec`.
- [`scope`](scope.md): Plugin-scoped library access.
- [`secrets`](secrets.md): Encrypted secrets store — DB-backed credential storage.
- [`sql_plan`](sql_plan.md): Host-owned generic SQL atomic plans for database adapters. Host-owned SQL atomic plans for database plugins.
- [`atomic_status`](atomic_status.md): Application status strings produced by host plan interpretation.
- [`secret_account_type`](secret_account_type.md): Ownership namespace for `encrypted_secrets.account_type`.
- [`secret_kind`](secret_kind.md): Well-known `kind` values for the `encrypted_secrets` table.

## Traits

### AtomicTxnBackend

```rust
pub trait AtomicTxnBackend: Send + Sync {
    fn delete_user<'life0, 'async_trait>(
        &'life0 self,
        id: i64,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn set_user_status<'life0, 'async_trait>(
        &'life0 self,
        id: i64,
        status: UserStatus,
    ) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn set_user_password_hash<'life0, 'life1, 'async_trait>(
        &'life0 self,
        id: i64,
        password_hash: Option<&'life1 str>,
    ) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn set_user_role<'life0, 'async_trait>(
        &'life0 self,
        id: i64,
        role: UserRole,
    ) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn redeem_claim_ticket_to_session<'life0, 'life1, 'life2, 'life3, 'life4, 'life5, 'async_trait>(
        &'life0 self,
        token_hash: &'life1 str,
        session_hash: &'life2 str,
        expires_at: DateTime<Utc>,
        client: Option<&'life3 SessionClientInfo>,
        new_password_hash: Option<&'life4 str>,
        password_fingerprint: Option<&'life5 str>,
    ) -> Pin<Box<dyn Future<Output = Result<PortalIdentity>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,
        'life4: 'async_trait,
        'life5: 'async_trait,;
    fn take_oidc_rp_state<'life0, 'life1, 'async_trait>(
        &'life0 self,
        state_hash: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<Option<(String, String, String, String, Option<i64>)>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn take_webauthn_challenge<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        challenge_id: &'life1 str,
        kind: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<Option<(Option<i64>, String)>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn enqueue_job<'life0, 'async_trait>(
        &'life0 self,
        spec: EnqueueJobSpec,
    ) -> Pin<Box<dyn Future<Output = Result<EnqueueOutcome>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn claim_next_job<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        resource_class: JobResourceClass,
        owner: &'life1 str,
        lease_secs: u64,
        operation_id: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<Option<JobRecord>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn reserve_job_temp_path<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        job_id: &'life1 str,
        path: &'life2 str,
        reserved_bytes: u64,
        quota_bytes: u64,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn confirm_totp_enrollment<'life0, 'life1, 'async_trait>(
        &'life0 self,
        user_id: i64,
        record: &'life1 EncryptedSecretRecord,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn disable_user_totp<'life0, 'async_trait>(
        &'life0 self,
        user_id: i64,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn publish_domain_event<'life0, 'async_trait>(
        &'life0 self,
        spec: PublishDomainEventSpec,
    ) -> Pin<Box<dyn Future<Output = Result<PublishDomainEventOutcome>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn set_acquire_status<'life0, 'life1, 'life2, 'life3, 'async_trait>(
        &'life0 self,
        book_uuid: &'life1 str,
        status: AcquireStatus,
        storage_key: Option<&'life2 str>,
        error_message: Option<&'life3 str>,
        event: Option<PublishDomainEventSpec>,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,;
    fn dispatch_event_deliveries<'life0, 'life1, 'life2, 'life3, 'async_trait>(
        &'life0 self,
        event_id: &'life1 str,
        subscribers: &'life2 [EventSubscriber],
        operation_id: &'life3 str,
        mark_dispatched: bool,
    ) -> Pin<Box<dyn Future<Output = Result<u32>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,;
    fn claim_event_delivery<'life0, 'life1, 'life2, 'life3, 'life4, 'life5, 'async_trait>(
        &'life0 self,
        delivery_id: &'life1 str,
        owner: &'life2 str,
        lease_secs: u64,
        operation_id: &'life3 str,
        plugin_id: &'life4 str,
        resource_class: &'life5 str,
        max_in_flight: u32,
    ) -> Pin<Box<dyn Future<Output = Result<Option<EventDeliveryRecord>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,
        'life4: 'async_trait,
        'life5: 'async_trait,;
}
```

Backend that runs [`crate::LibraryStore`](#librarystore) named security operations as a
single guest atomic batch.

Implementations must preserve the same fail-closed semantics as the SeaORM
path: last-owner refusals mutate nothing; a failed claim redeem must not
consume the ticket or write a password/session; consume-once OIDC/WebAuthn
rows must not be observable by a concurrent caller; TOTP enroll/disable must
keep `encrypted_secrets` and `users.totp_enabled` in the same transaction.

#### AtomicTxnBackend::delete_user

```rust
fn delete_user<'life0, 'async_trait>(
    &'life0 self,
    id: i64,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Delete a first-party user and personal data (last-owner guarded).

#### AtomicTxnBackend::set_user_status

```rust
fn set_user_status<'life0, 'async_trait>(
    &'life0 self,
    id: i64,
    status: UserStatus,
) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Set user status (`active` / `disabled`; last-owner guarded).

#### AtomicTxnBackend::set_user_password_hash

```rust
fn set_user_password_hash<'life0, 'life1, 'async_trait>(
    &'life0 self,
    id: i64,
    password_hash: Option<&'life1 str>,
) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Set or clear the Argon2id password hash and bump `security_version`.

#### AtomicTxnBackend::set_user_role

```rust
fn set_user_role<'life0, 'async_trait>(
    &'life0 self,
    id: i64,
    role: UserRole,
) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Set user role (last-owner guarded on demotion).

#### AtomicTxnBackend::redeem_claim_ticket_to_session

```rust
fn redeem_claim_ticket_to_session<'life0, 'life1, 'life2, 'life3, 'life4, 'life5, 'async_trait>(
    &'life0 self,
    token_hash: &'life1 str,
    session_hash: &'life2 str,
    expires_at: DateTime<Utc>,
    client: Option<&'life3 SessionClientInfo>,
    new_password_hash: Option<&'life4 str>,
    password_fingerprint: Option<&'life5 str>,
) -> Pin<Box<dyn Future<Output = Result<PortalIdentity>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
    'life4: 'async_trait,
    'life5: 'async_trait,
```

Consume a claim ticket, optionally set a first password, mint a session.

#### AtomicTxnBackend::take_oidc_rp_state

```rust
fn take_oidc_rp_state<'life0, 'life1, 'async_trait>(
    &'life0 self,
    state_hash: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<Option<(String, String, String, String, Option<i64>)>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Consume a one-time OIDC RP state. `Ok(None)` if missing or expired.

Tuple is `(provider_id, pkce_verifier, nonce, purpose, user_id)`.

#### AtomicTxnBackend::take_webauthn_challenge

```rust
fn take_webauthn_challenge<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    challenge_id: &'life1 str,
    kind: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<Option<(Option<i64>, String)>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Consume a one-time WebAuthn challenge. `Ok(None)` if missing or expired.

Tuple is `(user_id, state_json)`.

#### AtomicTxnBackend::enqueue_job

```rust
fn enqueue_job<'life0, 'async_trait>(
    &'life0 self,
    spec: EnqueueJobSpec,
) -> Pin<Box<dyn Future<Output = Result<EnqueueOutcome>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Admit a durable job in one atomic transaction.

#### AtomicTxnBackend::claim_next_job

```rust
fn claim_next_job<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    resource_class: JobResourceClass,
    owner: &'life1 str,
    lease_secs: u64,
    operation_id: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<Option<JobRecord>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Claim the next ready job; `operation_id` makes a lost response replay-safe.

#### AtomicTxnBackend::reserve_job_temp_path

```rust
fn reserve_job_temp_path<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    job_id: &'life1 str,
    path: &'life2 str,
    reserved_bytes: u64,
    quota_bytes: u64,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Reserve scratch-quota bytes for `path` on `job_id`.

#### AtomicTxnBackend::confirm_totp_enrollment

```rust
fn confirm_totp_enrollment<'life0, 'life1, 'async_trait>(
    &'life0 self,
    user_id: i64,
    record: &'life1 EncryptedSecretRecord,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Promote a sealed TOTP secret to `primary` and set `totp_enabled`.

#### AtomicTxnBackend::disable_user_totp

```rust
fn disable_user_totp<'life0, 'async_trait>(
    &'life0 self,
    user_id: i64,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Delete TOTP secrets and clear `totp_enabled`.

#### AtomicTxnBackend::publish_domain_event

```rust
fn publish_domain_event<'life0, 'async_trait>(
    &'life0 self,
    spec: PublishDomainEventSpec,
) -> Pin<Box<dyn Future<Output = Result<PublishDomainEventOutcome>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Persist a domain event in the outbox.

#### AtomicTxnBackend::set_acquire_status

```rust
fn set_acquire_status<'life0, 'life1, 'life2, 'life3, 'async_trait>(
    &'life0 self,
    book_uuid: &'life1 str,
    status: AcquireStatus,
    storage_key: Option<&'life2 str>,
    error_message: Option<&'life3 str>,
    event: Option<PublishDomainEventSpec>,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
```

Update acquire status and, when acquired, publish `book_acquired` in the same transaction.

#### AtomicTxnBackend::dispatch_event_deliveries

```rust
fn dispatch_event_deliveries<'life0, 'life1, 'life2, 'life3, 'async_trait>(
    &'life0 self,
    event_id: &'life1 str,
    subscribers: &'life2 [EventSubscriber],
    operation_id: &'life3 str,
    mark_dispatched: bool,
) -> Pin<Box<dyn Future<Output = Result<u32>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
```

Create deliveries for `subscribers`. `mark_dispatched` finishes the parent event.

#### AtomicTxnBackend::claim_event_delivery

```rust
fn claim_event_delivery<'life0, 'life1, 'life2, 'life3, 'life4, 'life5, 'async_trait>(
    &'life0 self,
    delivery_id: &'life1 str,
    owner: &'life2 str,
    lease_secs: u64,
    operation_id: &'life3 str,
    plugin_id: &'life4 str,
    resource_class: &'life5 str,
    max_in_flight: u32,
) -> Pin<Box<dyn Future<Output = Result<Option<EventDeliveryRecord>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
    'life4: 'async_trait,
    'life5: 'async_trait,
```

CAS-claim one pending delivery after the host has filtered eligibility.

Implemented by: `InProcessSqliteAtomic`

### TypedAtomicExec

```rust
pub trait TypedAtomicExec: Send + Sync {
    fn execute_typed<'life0, 'async_trait>(
        &'life0 self,
        envelope: AdapterExecuteRequest,
    ) -> Pin<Box<dyn Future<Output = Result<ExecuteReply, PluginError>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Runs a host-authorized typed [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest) as one guest `executeAtomic`.

First-party hosts attach this alongside [`AtomicTxnBackend`](#atomictxnbackend) so granted job
sessions do not round-trip through the SeaORM proxy (`BEGIN` + nested
`query`/`execute`). In-process tests leave it unset and run the same
request on the local connection.

#### TypedAtomicExec::execute_typed

```rust
fn execute_typed<'life0, 'async_trait>(
    &'life0 self,
    envelope: AdapterExecuteRequest,
) -> Pin<Box<dyn Future<Output = Result<ExecuteReply, PluginError>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Executes `req` as one typed atomic batch.

##### Errors

Returns a plugin ABI error when validation, transport, or the engine
rejects the batch.

## Structs

### DbAtomicResult

```rust
pub struct DbAtomicResult {
    pub status: String,
    pub payload: Option<Value>,
    pub operation_id: String,
    pub replayed: bool,
    pub receipt_created_at: Option<String>,
    pub timing: Option<DbTiming>,
}
```

Host-interpreted result of a named atomic operation.

**Fields**

- `status`: Application outcome (`ok`, `empty`, `notFound`, `lastOwner`, …).
- `payload`: Op-specific record when `status` is `ok`.
- `operation_id`: Echo of the request `operationId`.
- `replayed`: True when this result was loaded from a durable receipt.
- `receipt_created_at`: RFC 3339 timestamp when the receipt was first written.
- `timing`: Handler/engine timing. Not hashed for idempotency.

#### DbAtomicResult::ok

```rust
pub fn ok(payload: JsonValue) -> Self
```

Successful operation with a JSON payload.

#### DbAtomicResult::ok_unit

```rust
pub fn ok_unit() -> Self
```

Successful operation with no payload (`deleteUser`).

#### DbAtomicResult::with_status

```rust
pub fn with_status(status: &str) -> Self
```

Application failure or empty consume-once result (no payload).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### BackendMigrateOptions

```rust
pub struct BackendMigrateOptions {
    pub dry_run: bool,
    pub force: bool,
}
```

Options for a backend-to-backend library copy.

**Fields**

- `dry_run`: When true, report counts only — do not write to the destination.
- `force`: Allow copying into a destination that already has library rows.

Implements: `Clone`, `Debug`

### BackendMigrateSummary

```rust
pub struct BackendMigrateSummary {
    pub tables: BTreeMap<String, usize>,
    pub dry_run: bool,
}
```

Per-table row counts copied (or that would be copied in dry-run mode).

**Fields**

- `tables`: Map of table name → rows copied (or counted in dry-run).
- `dry_run`: When true, counts rows without writing to the destination.

#### BackendMigrateSummary::total_rows

```rust
pub fn total_rows(&self) -> usize
```

Sums row counts across all tables in the summary.

Implements: `Clone`, `Debug`, `Default`, `Serialize`

### BackupRepository

```rust
pub struct BackupRepository {
    /* private fields */
}
```

On-disk backup repository rooted at `files_dir/backups`.

#### BackupRepository::open

```rust
pub fn open(files_dir: &Path) -> Result<Self>
```

Opens (creating) the repository under `files_dir/backups`.

##### Errors

Returns when the backups directory cannot be created.

#### BackupRepository::open_root

```rust
pub fn open_root(root: &Path) -> Result<Self>
```

Opens `root` as a backups repository (`manifests/` + `objects/`).

##### Errors

Returns when the directories cannot be created.

#### BackupRepository::root

```rust
pub fn root(&self) -> &Path
```

Repository root (`…/backups`).

#### BackupRepository::lock_exclusive

```rust
pub fn lock_exclusive(&self) -> Result<RepoLockGuard<'_>>
```

Exclusive lock covering object publication and GC/prune.

Same-thread reentry is allowed so [`Self::put_object`](#backuprepositoryput_object) can nest under a
backup that already holds the lock. Other threads wait. Other processes
block on `backups/.lock`.

##### Errors

Returns when the lock file cannot be created or flocked.

#### BackupRepository::put_object

```rust
pub fn put_object(&self, object: &CanonicalObject) -> Result<String>
```

Writes `object` if missing. Returns the uncompressed SHA-256 hex.

The write uses a unique same-directory temp file, fsyncs it, then
installs with a no-clobber hard link. An existing object is reused only
after its bytes verify; a truncated or corrupt file is replaced.

##### Errors

Returns when encoding, compression, or the filesystem write fails.

#### BackupRepository::get_object

```rust
pub fn get_object(&self, digest: &str) -> Result<CanonicalObject>
```

Loads and verifies one object by uncompressed digest.

##### Errors

Returns when the object is missing, corrupt, or the digest mismatches.

#### BackupRepository::object_exists

```rust
pub fn object_exists(&self, digest: &str) -> bool
```

True when an object file exists (does not imply the bytes are valid).

#### BackupRepository::publish_manifest

```rust
pub fn publish_manifest(&self, manifest: &BackupManifest) -> Result<PathBuf>
```

Publishes `manifest` only after all referenced objects exist **and**
verify.

Writes JSON then a sibling `.sha256` of the exact file bytes. Incomplete
staging uses `manifests/.staging-<id>.json` and is ignored by list/GC.

##### Errors

Returns when a referenced object is missing or corrupt, or the
filesystem write fails.

#### BackupRepository::read_manifest

```rust
pub fn read_manifest(&self, id: &str) -> Result<BackupManifest>
```

Reads and integrity-checks a published manifest.

##### Errors

Returns when the file is missing, JSON is malformed, the sidecar hash
mismatches, or the embedded id does not match the filename.

#### BackupRepository::list_manifests

```rust
pub fn list_manifests(&self) -> Vec<BackupManifest>
```

Lists published recovery-point manifests oldest-first. Staging files
and unreadable manifests are skipped.

#### BackupRepository::list_manifests_strict

```rust
pub fn list_manifests_strict(&self) -> Result<Vec<BackupManifest>>
```

Lists every published recovery-point manifest, failing if any cannot
be verified. Staging files (dot-prefixed) are ignored.

Use this for prune/GC. [`Self::list_manifests`](#backuprepositorylist_manifests) stays best-effort for
non-destructive listing.

##### Errors

Returns when the manifests directory cannot be read, a published
`*.json` fails [`Self::read_manifest`](#backuprepositoryread_manifest), or an unexpected non-sidecar
file is present.

#### BackupRepository::delete_manifest

```rust
pub fn delete_manifest(&self, id: &str) -> Result<bool>
```

Deletes one published manifest (not its objects). Returns whether a
file was removed.

##### Errors

Returns when the id is unsafe or unlink fails for a reason other than
not found.

#### BackupRepository::gc_unreferenced_objects

```rust
pub fn gc_unreferenced_objects(&self) -> Result<usize>
```

Deletes objects not referenced by any retained published manifest.

Reachability is computed from a **strict** enumeration of published
manifests. An unreadable retained manifest fails closed instead of
treating its objects as garbage.

##### Errors

Returns when a published manifest cannot be verified, a directory
cannot be read, or an unreferenced object cannot be unlinked.

Implements: `Clone`, `Debug`

### BackupListEntry

```rust
pub struct BackupListEntry {
    pub manifest: BackupManifest,
}
```

Catalog entry used by `bookclerk db backup list`.

**Fields**

- `manifest`: Parsed manifest.

Implements: `Clone`, `Debug`

### BackupManifest

```rust
pub struct BackupManifest {
    pub format_version: u32,
    pub id: String,
    pub created_at: String,
    pub schema_state: String,
    pub schema_checksum: Option<String>,
    pub frozen_version: Option<i64>,
    pub sql_contract_version: u32,
    pub app_version: String,
    pub reason: BackupReason,
    pub migrate_to: i64,
    pub include_plugin_databases: bool,
    pub units: Vec<BackupUnit>,
}
```

On-disk recovery-point metadata (`manifests/<id>.json`).

**Fields**

- `format_version`: Catalog format (`1` for content-addressed canonical objects).
- `id`: Stable recovery-point id (UUID).
- `created_at`: RFC 3339 UTC timestamp.
- `schema_state`: [`SchemaState::display`](#schemastatedisplay) at capture.
- `schema_checksum`: SHA-256 recorded on the library schema state marker.
- `frozen_version`: Frozen revision when the source library was frozen.
- `sql_contract_version`: Bookclerk SQL contract of the library unit.
- `app_version`: Bookclerk version that wrote the backup.
- `reason`: Operator vs automatic.
- `migrate_to`: Target frozen version a following migrate step was heading toward.
- `include_plugin_databases`: When true, every registered plugin binding was required at capture.
- `units`: Logical database units (library first).

#### BackupManifest::referenced_objects

```rust
pub fn referenced_objects(&self) -> BTreeSet<String>
```

Union of every object digest referenced by this recovery point.

#### BackupManifest::library_unit

```rust
pub fn library_unit(&self) -> Result<&BackupUnit>
```

Library unit from this manifest.

##### Errors

Returns when the library unit is missing or duplicated.

#### BackupManifest::plugin_units

```rust
pub fn plugin_units(&self) -> Vec<&BackupUnit>
```

Plugin binding units.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### BackupOutcome

```rust
pub struct BackupOutcome {
    pub dir: PathBuf,
    pub manifest: BackupManifest,
}
```

Result of publishing a recovery point.

**Fields**

- `dir`: Repository root (`files_dir/backups`).
- `manifest`: Parsed manifest.

Implements: `Clone`, `Debug`

### BackupRequest

```rust
pub struct BackupRequest {
    pub files_dir: PathBuf,
    pub schema_state: SchemaState,
    pub reason: BackupReason,
    pub to_version: i64,
    pub include_plugin_databases: bool,
    pub consistent_backup_read: bool,
    pub backend_at_capture: String,
    pub max_result_rows: u32,
    pub max_result_bytes: u32,
    pub max_atomic_result_bytes: u32,
    pub plugin_units: Vec<PreparedPluginUnit>,
    pub adapter: Option<SharedAdapterBackupOps>,
    pub in_process: bool,
}
```

Inputs for an automatic or CLI backup.

**Fields**

- `files_dir`: `$BOOKCLERK_FILES_DIR`.
- `schema_state`: Durable library schema state at capture. [`SchemaState::Uninitialized`](#schemastate) skips.
- `reason`: Operator vs pre-migrate.
- `to_version`: Version the upcoming migrate step will target (`0` for manual).
- `include_plugin_databases`: Include every registered plugin-owned database binding.
- `consistent_backup_read`: Guest advertised consistent backup read.
- `backend_at_capture`: Adapter id at capture (diagnostic; not used to branch restore).
- `max_result_rows`: Negotiated `maxResultRows` (clamped).
- `max_result_bytes`: Negotiated `maxResultBytes` for one SELECT page.
- `max_atomic_result_bytes`: Negotiated `maxAtomicResultBytes` (capture page budget is the min of both).
- `plugin_units`: Plugin sessions prepared by the host (must match the registry when inclusion is on).
- `adapter`: Library adapter hooks. `None` uses the in-process SDK (tests).
- `in_process`: True when leftover SQL is physically lowered on the opened connection.

Implements: `Clone`, `Debug`

### BackupUnit

```rust
pub struct BackupUnit {
    pub kind: DatabaseUnitKind,
    pub plugin_id: Option<String>,
    pub binding: Option<String>,
    pub backend_at_capture: String,
    pub sql_contract_version: u32,
    pub schema_object: String,
    pub identity_object: String,
    pub tables: Vec<BackupTable>,
    pub plugin_schema_namespace: Option<String>,
    pub plugin_schema_state: Option<String>,
    pub plugin_schema_checksum: Option<String>,
}
```

One logical database in a recovery point (library or plugin binding).

**Fields**

- `kind`: Library vs plugin binding.
- `plugin_id`: Plugin id when [`DatabaseUnitKind::PluginBinding`](#databaseunitkind).
- `binding`: Binding name when [`DatabaseUnitKind::PluginBinding`](#databaseunitkind).
- `backend_at_capture`: Adapter that captured this unit (diagnostic only; restore is cross-adapter).
- `sql_contract_version`: Bookclerk SQL grammar/ABI contract, not a library schema revision.
- `schema_object`: SHA-256 of the schema object.
- `identity_object`: SHA-256 of the identity object.
- `tables`: User-visible tables (not catalog/identity companions).
- `plugin_schema_namespace`: Plugin id when [`DatabaseUnitKind::PluginBinding`](#databaseunitkind) (owner identity).
- `plugin_schema_state`: [`crate::PluginMigrationHistory::display`](#pluginmigrationhistorydisplay) at capture (`history@{n}+{digest}`).
- `plugin_schema_checksum`: Digest of the complete ordered `(migration_id, checksum)` journal.

#### BackupUnit::restore_kind

```rust
pub fn restore_kind(&self) -> CanonicalRestoreKind
```

Restore companion kind for this unit.

#### BackupUnit::referenced_objects

```rust
pub fn referenced_objects(&self) -> BTreeSet<String>
```

Object digests this unit needs to restore.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CanonicalDatabaseSchema

```rust
pub struct CanonicalDatabaseSchema {
    pub sql_contract_version: u32,
    pub tables: Vec<CanonicalTableSchema>,
    pub indexes: Vec<CreateIndexSchema>,
}
```

Admitted canonical schema used to export or restore one database unit.

**Fields**

- `sql_contract_version`: Bookclerk SQL grammar/ABI contract, not a library schema revision.
- `tables`: `CREATE TABLE` statements in FK-safe order after [`crate::sort_tables_by_foreign_keys`](#sort_tables_by_foreign_keys).
- `indexes`: `CREATE INDEX` statements (after tables).

#### CanonicalDatabaseSchema::schema_sql

```rust
pub fn schema_sql(&self) -> Vec<String>
```

Ordered canonical DDL (tables then indexes).

#### CanonicalDatabaseSchema::table_names

```rust
pub fn table_names(&self) -> Vec<String>
```

Folded user table names in CREATE order.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CanonicalExportOpts

```rust
pub struct CanonicalExportOpts {
    pub consistent_backup_read: bool,
    pub skip_tables: BTreeSet<String>,
    pub max_result_rows: u32,
    pub max_result_bytes: u32,
    pub max_atomic_result_bytes: u32,
    pub chunk_target_bytes: usize,
    pub adapter: Option<SharedAdapterBackupOps>,
    pub in_process: bool,
}
```

Options for a consistent canonical export.

**Fields**

- `consistent_backup_read`: Guest advertised `DbCapabilities::supports_consistent_backup_read`.
- `skip_tables`: Folded table names omitted from row capture (library `plugin_databases`).
- `max_result_rows`: Page size for `SELECT … LIMIT` (honors adapter `maxResultRows`).
- `max_result_bytes`: Encoded-byte budget for one SELECT page (`maxResultBytes`).
- `max_atomic_result_bytes`: Encoded-byte budget for one atomic reply (`maxAtomicResultBytes`).
- `chunk_target_bytes`: Uncompressed JSON target for one table-data chunk.
- `adapter`: Adapter snapshot/identity hooks. `None` uses the in-process SDK (tests).
- `in_process`: True when leftover SQL is physically lowered on the opened connection.

#### CanonicalExportOpts::from_caps

```rust
pub fn from_caps(caps: &DbCapabilities) -> Self
```

Page-size and result-byte limits from negotiated adapter capabilities.

#### CanonicalExportOpts::page_byte_budget

```rust
pub fn page_byte_budget(&self) -> usize
```

Effective encoded-byte budget for one capture SELECT page.

Implements: `Clone`, `Debug`, `Default`

### CanonicalRestoreOpts

```rust
pub struct CanonicalRestoreOpts {
    pub atomic_unit_restore: bool,
    pub max_binds: u32,
    pub max_payload_bytes: u32,
    pub max_request_bytes: u32,
    pub adapter: Option<SharedAdapterBackupOps>,
    pub in_process: bool,
}
```

Negotiated limits for parameterized canonical restore.

**Fields**

- `atomic_unit_restore`: Guest advertised `atomicUnitRestore`.
- `max_binds`: Maximum bound parameters per INSERT (`maxBinds`).
- `max_payload_bytes`: Maximum UTF-8 bytes of SQL plus binds per statement (`maxPayloadBytes`).
- `max_request_bytes`: Maximum encoded bytes of one [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest).
- `adapter`: Adapter restore hooks. `None` uses the in-process SDK (tests).
- `in_process`: True when leftover SQL is physically lowered on the opened connection.

#### CanonicalRestoreOpts::from_caps

```rust
pub fn from_caps(caps: &DbCapabilities) -> Result<Self>
```

Restore limits copied from negotiated adapter capabilities.

The versioning contract is checked with [`require_schema_migrations`](#require_schema_migrations),
never inferred from SeaORM [`sea_orm::DbBackend`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/type.DbBackend.html). Host policy requires
`schemaMigrations`.

##### Errors

Returns when `caps` is not a known versioning contract (`schemaMigrations`
missing).

Implements: `Clone`, `Debug`, `Default`

### CanonicalTableSchema

```rust
pub struct CanonicalTableSchema {
    pub create_sql: String,
    pub parsed: CreateTableSchema,
}
```

One admitted `CREATE TABLE` plus its parsed IR.

**Fields**

- `create_sql`: Original canonical SQL (no trailing semicolon).
- `parsed`: Parsed columns, constraints, and identity.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### IdentityHighWater

```rust
pub struct IdentityHighWater {
    pub column: String,
    pub last: i64,
}
```

Identity high-water for one generated column.

**Fields**

- `column`: Identity column name.
- `last`: Highest generated or stored value that must not be reused.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PreparedPluginUnit

```rust
pub struct PreparedPluginUnit {
    pub plugin_id: String,
    pub binding: String,
    pub backend_at_capture: String,
    pub db: DatabaseConnection,
    pub max_result_rows: u32,
    pub max_result_bytes: u32,
    pub max_atomic_result_bytes: u32,
    pub adapter: Option<SharedAdapterBackupOps>,
}
```

Open plugin binding captured through the active adapter session.

**Fields**

- `plugin_id`: Owning plugin id.
- `binding`: Binding name.
- `backend_at_capture`: Adapter that produced this unit (diagnostic).
- `db`: SeaORM connection opened via the active adapter (not a native dump).
- `max_result_rows`: Negotiated `maxResultRows` for this binding session.
- `max_result_bytes`: Negotiated `maxResultBytes` for this binding session.
- `max_atomic_result_bytes`: Negotiated `maxAtomicResultBytes` for this binding session.
- `adapter`: Binding-scoped adapter hooks. `None` uses the in-process SDK (tests).

Implements: `Clone`, `Debug`

### RestorePlan

```rust
pub struct RestorePlan {
    pub manifest: BackupManifest,
    pub plugin_units: Vec<BackupUnit>,
}
```

Library restore result plus plugin units for the host to apply.

Plugin databases are restored separately after this returns. A failure
while restoring a later unit leaves earlier units replaced (no bundle-level
atomicity). Each unit still uses complete replacement semantics.

**Fields**

- `manifest`: Verified manifest.
- `plugin_units`: Plugin units to restore onto the target adapter.

Implements: `Clone`, `Debug`

### SchemaBackupOpts

```rust
pub struct SchemaBackupOpts {
    pub files_dir: PathBuf,
    pub include_plugin_databases: bool,
    pub consistent_backup_read: bool,
    pub backend_at_capture: String,
    pub max_result_rows: u32,
    pub max_result_bytes: u32,
    pub max_atomic_result_bytes: u32,
    pub plugin_units: Vec<PreparedPluginUnit>,
    pub adapter: Option<SharedAdapterBackupOps>,
}
```

Optional in-place backup taken before applying ups or downs.

**Fields**

- `files_dir`: `$BOOKCLERK_FILES_DIR` root for `backups/`.
- `include_plugin_databases`: Include every registered plugin-owned database binding.
- `consistent_backup_read`: Guest advertised consistent backup read.
- `backend_at_capture`: Adapter id at capture (diagnostic).
- `max_result_rows`: Negotiated page size.
- `max_result_bytes`: Negotiated encoded SELECT-page budget.
- `max_atomic_result_bytes`: Negotiated encoded atomic-reply budget.
- `plugin_units`: Plugin sessions prepared by the host when inclusion is requested.
- `adapter`: Library adapter hooks. `None` uses the in-process SDK (tests).

Implements: `Clone`, `Debug`, `Default`

### ValidatedBackup

```rust
pub struct ValidatedBackup {
    pub manifest: BackupManifest,
    pub library: BackupUnit,
    pub plugin_units: Vec<BackupUnit>,
}
```

Recovery point whose objects, schema, and typed rows have been verified.

**Fields**

- `manifest`: Parsed, supported manifest.
- `library`: Library unit metadata (objects already verified in the repository).
- `plugin_units`: Plugin binding units (already verified).

Implements: `Clone`, `Debug`

### SchemaApplyOptions

```rust
pub struct SchemaApplyOptions {
    pub backup: Option<SchemaBackupOpts>,
}
```

Options for [`apply_host_schema_with_options`](#apply_host_schema_with_options).

**Fields**

- `backup`: When set, write a recovery point before applying DDL to a non-empty database.

Implements: `Clone`, `Debug`, `Default`

### SchemaBatch

```rust
pub struct SchemaBatch {
    pub statements: Vec<String>,
}
```

Canonical schema apply batch: already-separate host statements plus marker.

Adapters lower each statement at execution; they must not re-split SQL.

**Fields**

- `statements`: Ordered SQL strings; the last statement is the version/state marker.

#### SchemaBatch::from_statements_and_marker

```rust
pub fn from_statements_and_marker(
    stmts: impl IntoIterator<Item = impl Into<String>>,
    marker: impl Into<String>,
) -> Self
```

Builds a batch from an ordered statement list plus a trailing marker.

#### SchemaBatch::unreleased

```rust
pub fn unreleased(
    stmts: impl IntoIterator<Item = impl Into<String>>,
    checksum: &str,
) -> Self
```

Unreleased development pack plus `bookclerk_schema_migrations` unreleased row.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### InProcessSqliteAtomic

```rust
pub struct InProcessSqliteAtomic {
    pub db: DatabaseConnection,
}
```

Compiles named ops and runs them on a local SeaORM SQLite connection.

**Fields**

- `db`: Open library connection (already migrated).

Implements: `AtomicTxnBackend`

### PluginMigrationHistory

```rust
pub struct PluginMigrationHistory {
    pub entries: Vec<PluginJournalEntry>,
}
```

Ordered durable plugin migration history.

**Fields**

- `entries`: Journal rows in ordinal order.

#### PluginMigrationHistory::digest

```rust
pub fn digest(&self) -> String
```

SHA-256 digest of the complete ordered `(migration_id, checksum)` list.

#### PluginMigrationHistory::len

```rust
pub fn len(&self) -> usize
```

Number of applied journal rows.

#### PluginMigrationHistory::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no plugin migrations have been applied.

#### PluginMigrationHistory::display

```rust
pub fn display(&self) -> String
```

Operator-facing history label (`history@{n}+{digest}`).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### PluginMigrationSequence

```rust
pub struct PluginMigrationSequence {
    pub migrations: Vec<ProvenPluginMigration>,
}
```

Proven, ordered plugin migration sequence.

**Fields**

- `migrations`: Registration-order migrations with checksums.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ProvenPluginMigration

```rust
pub struct ProvenPluginMigration {
    pub id: String,
    pub operations: Vec<PluginMigrationOp>,
    pub checksum: String,
}
```

One proven registered migration.

**Fields**

- `id`: Opaque plugin-chosen identity.
- `operations`: Ordered operations.
- `checksum`: Deterministic checksum of `(id, classified ops)`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AccountLinkRecord

```rust
pub struct AccountLinkRecord {
    pub id: i64,
    pub identity_id: i64,
    pub plugin_key: String,
    pub account_id: String,
    pub source: String,
    pub created_at: DateTime<Utc>,
}
```

Link between a portal identity and a bookstore account.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `identity_id`: Foreign key to `portal_identities.id`.
- `plugin_key`: Provenance-qualified PluginKey that owns the account.
- `account_id`: Store or operator account id this row belongs to.
- `source`: Display alias of the owning plugin (`audible`, `libro`, …).
- `created_at`: RFC 3339 timestamp when the row was inserted.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AccountRecord

```rust
pub struct AccountRecord {
    pub id: i64,
    pub plugin_key: String,
    pub account_id: String,
    pub source: String,
    pub marketplace: String,
    pub label: Option<String>,
    pub scan_enabled: bool,
    pub connection_status: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Account row stored in the Bookclerk DB.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `plugin_key`: Provenance-qualified PluginKey that owns this account. Any instance
  of that plugin, on any node, may act for it.
- `account_id`: Store or operator account id, unique within one plugin key.
- `source`: Display alias of the owning plugin (`audible`, `libro`, …).
- `marketplace`: Store marketplace / locale code (for example `us`, `uk`).
- `label`: Optional operator-facing display label.
- `scan_enabled`: When nonzero/true, scheduled scans include this account.
- `connection_status`: `active` or `revoked` (credentials removed; books retained).
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### BookRecord

```rust
pub struct BookRecord {
    pub id: i64,
    pub uuid: String,
    pub plugin_key: String,
    pub source: String,
    pub account_id: String,
    pub product_id: String,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub marketplace: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub series_asin: Option<String>,
    pub acquire_status: AcquireStatus,
    pub storage_key: Option<String>,
    pub error_message: Option<String>,
    pub purchased_at: Option<DateTime<Utc>>,
    pub tags: Option<String>,
    pub rating_overall: Option<f32>,
    pub rating_performance: Option<f32>,
    pub rating_story: Option<f32>,
    pub is_finished: bool,
    pub pdf_status: AcquireStatus,
    pub pdf_storage_key: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub is_abridged: bool,
    pub content_kind: String,
    pub categories: Option<String>,
    pub subtitle: Option<String>,
    pub published_at: Option<DateTime<Utc>>,
    pub description: Option<String>,
    pub language: Option<String>,
    pub cover_url: Option<String>,
    pub subjects: Option<String>,
    pub enrich_source: Option<String>,
    pub enrich_confidence: Option<f64>,
    pub enrich_updated_at: Option<DateTime<Utc>>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Book / library item (one ownership row per store product per account).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `uuid`: Stable UUID for this row (API / foreign-key identity).
- `plugin_key`: Provenance-qualified PluginKey that owns the account.
- `source`: Display alias of the owning plugin (`audible`, `libro`, …).
- `account_id`: Store or operator account id this row belongs to.
- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).
- `asin`: Audible ASIN when known (`None` for Libro-only rows without enrichment).
- `isbn`: ISBN-13 when known (Libro always; Audible when the API provides it).
- `marketplace`: Store marketplace / locale code (for example `us`, `uk`).
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `series_asin`: Audible series / podcast-parent ASIN (`Series.AudibleSeriesId`).
- `acquire_status`: Download/acquire pipeline state (`not_acquired`, `queued`, …).
- `storage_key`: Object-storage key for the primary audio artifact, if acquired.
- `error_message`: Last acquire/convert failure message for operators.
- `purchased_at`: RFC 3339 purchase time from the storefront, when known.
- `tags`: Operator or storefront tags (serialized string).
- `rating_overall`: Overall user rating from the storefront, if any.
- `rating_performance`: Narration/performance rating from the storefront, if any.
- `rating_story`: Story rating from the storefront, if any.
- `is_finished`: Whether the listener marked the title finished (0/1 or bool).
- `pdf_status`: Companion PDF acquire state (`not_acquired`, `acquired`, …).
- `pdf_storage_key`: Object-storage key for the companion PDF, if present.
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `is_abridged`: Whether the edition is abridged (0/1 or bool).
- `content_kind`: Title kind: `book`, `episode`, `podcast`, ….
- `categories`: Storefront category / genre path list (serialized).
- `subtitle`: Optional subtitle from bibliographic metadata.
- `published_at`: Publication date string from the storefront or enrichment.
- `description`: Blurb / synopsis text (may contain HTML).
- `language`: BCP-47 or storefront language code when known.
- `cover_url`: HTTPS URL for cover art when known.
- `subjects`: Subject / topic tags (often from Open Library; `;`- or `,`-separated).
- `enrich_source`: Plugin or catalog that last enriched bibliographic fields.
- `enrich_confidence`: 0–1 confidence score for the last enrichment pass.
- `enrich_updated_at`: RFC 3339 time of the last enrichment write.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

#### BookRecord::title_id

```rust
pub fn title_id(&self) -> &str
```

Public stable id used for CLI / API / acquire lookups.

#### BookRecord::asin_or_isbn

```rust
pub fn asin_or_isbn(&self) -> &str
```

Naming / display fallback: ASIN, else ISBN, else source product id.

#### BookRecord::download_product_id

```rust
pub fn download_product_id(&self) -> &str
```

Source-native id for download / fetch APIs (`product_id`).

Always the owning store's id (Audible ASIN or Libro ISBN), never an
enrichment ASIN copied onto a non-Audible row.

#### BookRecord::audible_asin

```rust
pub fn audible_asin(&self) -> Option<&str>
```

Audible ASIN for Audible license / catalog APIs when one is known.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### ClaimTicketRecord

```rust
pub struct ClaimTicketRecord {
    pub id: i64,
    pub token_hash: String,
    pub identity_id: Option<i64>,
    pub expires_at: DateTime<Utc>,
    pub redeemed_at: Option<DateTime<Utc>>,
    pub created_by: String,
    pub created_at: DateTime<Utc>,
}
```

Claim ticket metadata (token plaintext is never stored).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `token_hash`: SHA-256 hex digest of the opaque token (plaintext never stored).
- `identity_id`: Foreign key to `portal_identities.id`.
- `expires_at`: RFC 3339 expiry for the ticket, session, or code.
- `redeemed_at`: RFC 3339 time when the ticket/invite was redeemed, if any.
- `created_by`: Actor that created the row (user id or operator label).
- `created_at`: RFC 3339 timestamp when the row was inserted.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### DomainEventRecord

```rust
pub struct DomainEventRecord {
    pub id: String,
    pub event_type: String,
    pub schema_version: i64,
    pub occurred_at: DateTime<Utc>,
    pub account_id: String,
    pub source: String,
    pub correlation_id: String,
    pub causation_id: String,
    pub dedup_key: String,
    pub payload: String,
    pub ordering_key: String,
    pub dispatch_state: String,
    pub created_at: DateTime<Utc>,
    pub wake_pending: bool,
    pub queue_name: String,
    pub content_type: String,
    pub deliver_after: Option<DateTime<Utc>>,
}
```

Immutable outbox envelope.

**Fields**

- `id`: Event id.
- `event_type`: Event type.
- `schema_version`: Payload schema version.
- `occurred_at`: Occurrence time.
- `account_id`: Tenant / account id.
- `source`: Producer plugin id (`audible`, …); empty when unknown.
- `correlation_id`: Trace correlation id.
- `causation_id`: Causing id.
- `dedup_key`: Producer idempotency key.
- `payload`: Bounded JSON payload.
- `ordering_key`: Producer FIFO key copied onto each delivery.
- `dispatch_state`: `pending` or `dispatched`.
- `created_at`: Insert time.
- `wake_pending`: True until a wake pass for this event has completed.
- `queue_name`: Queue this row belongs to (`core.events` for domain events).
- `content_type`: Body encoding of `payload` (`json`, `text`, `bytes`).
- `deliver_after`: Earliest delivery time for a delayed queue message.

#### DomainEventRecord::is_queue_message

```rust
pub fn is_queue_message(&self) -> bool
```

True for a plugin-owned queue message (not a domain event).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### EmbeddingRecord

```rust
pub struct EmbeddingRecord {
    pub id: i64,
    pub target_kind: String,
    pub target_id: String,
    pub model: String,
    pub dims: i64,
    pub text_hash: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Stored embedding vector metadata (blob fetched separately when needed).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `target_kind`: Embedding target kind (`book`, `work`, …).
- `target_id`: Id of the embedded target within `target_kind`.
- `model`: Embedding model identifier used to produce `vector`.
- `dims`: Dimensionality of the stored embedding vector.
- `text_hash`: Hash of the text that was embedded (skip re-embed when unchanged).
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### EnqueueJobSpec

```rust
pub struct EnqueueJobSpec {
    pub kind: JobKind,
    pub payload: JobPayload,
    pub priority: i64,
    pub max_attempts: i64,
    pub max_pending: i64,
    pub run_after: Option<DateTime<Utc>>,
}
```

Admission request for [`crate::LibraryStore::enqueue_job`](#librarystoreenqueue_job).

**Fields**

- `kind`: Job kind to run.
- `payload`: Kind-specific filters and trigger metadata.
- `priority`: Higher values are claimed first (default 0).
- `max_attempts`: Maximum claims before a failure is terminal.
- `max_pending`: Global cap on pending+running rows; enqueue fails when at the cap.
- `run_after`: Optional delay before the job becomes claimable.

Implements: `Clone`, `Debug`

### EventCatalogSubscription

```rust
pub struct EventCatalogSubscription {
    pub event_type: String,
    pub schema_versions: Vec<u32>,
    pub supports_suspend: bool,
    pub resource_class: String,
    pub filter: Option<Value>,
    pub queue: String,
    pub max_retries: Option<u32>,
    pub dead_letter_queue: Option<String>,
    pub retry_delay_secs: u32,
}
```

One `[capabilities.events.subscriptions]` row persisted in `event_subscriber_nodes`.

**Fields**

- `event_type`: Versioned event type (`book_acquired`, …).
- `schema_versions`: Schema versions this guest can consume (default `[1]`).
- `supports_suspend`: Whether `EventResult::Suspended` is supported for this type.
- `resource_class`: Concurrency class (default `"network"`).
- `filter`: Optional host-owned payload object filter (top-level key equality).
- `queue`: Queue this subscription consumes; empty means the domain-event bus.
  A plugin-owned queue matches by name only (no type, schema, or filter).
- `max_retries`: Redeliveries before dead-lettering; `None` uses the host default.
- `dead_letter_queue`: Queue that receives a message once `max_retries` is exhausted.
- `retry_delay_secs`: Seconds a retried message waits before redelivery.

#### EventCatalogSubscription::new

```rust
pub fn new(event_type: impl Into<String>, schema_versions: Vec<u32>) -> Self
```

Build a type+schema subscription with default resource class and no filter.

#### EventCatalogSubscription::queue

```rust
pub fn queue(queue: impl Into<String>) -> Self
```

Build a plugin-owned queue subscription (`[[queues.consumers]]`).

#### EventCatalogSubscription::queue_name

```rust
pub fn queue_name(&self) -> &str
```

Queue this subscription consumes (`core.events` when unset).

#### EventCatalogSubscription::is_queue

```rust
pub fn is_queue(&self) -> bool
```

True for a plugin-owned queue subscription (not the domain-event bus).

#### EventCatalogSubscription::max_attempts

```rust
pub fn max_attempts(&self) -> Option<u32>
```

`max_attempts` for deliveries created from this subscription.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### EventDeliveryFence

```rust
pub struct EventDeliveryFence {
    pub delivery_id: String,
    pub owner: String,
    pub generation: i64,
}
```

Claim fence for an event delivery (mirrors [`JobFence`](#jobfence)).

**Fields**

- `delivery_id`: Delivery id that was claimed.
- `owner`: Worker id stored in `lease_owner`.
- `generation`: `lease_generation` assigned by the successful claim.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### EventDeliveryMetrics

```rust
pub struct EventDeliveryMetrics {
    pub pending: i64,
    pub running: i64,
    pub suspended: i64,
    pub dead_letter: i64,
    pub acked: i64,
    pub oldest_pending_age_secs: Option<i64>,
    pub retries_total: i64,
    pub suspensions_total: i64,
    pub dead_letters_total: i64,
    pub dispatch_latency_ms_avg: Option<i64>,
    pub handler_latency_ms_avg: Option<i64>,
}
```

Operator-visible delivery-queue counters for `GET /api/status`.

**Fields**

- `pending`: `pending` rows with no checkpoint (not suspended).
- `running`: Currently claimed rows.
- `suspended`: `pending` rows that hold a suspend checkpoint.
- `dead_letter`: Dead-lettered rows.
- `acked`: Successfully acked rows.
- `oldest_pending_age_secs`: Age in seconds of the oldest `pending` row, when any exist.
- `retries_total`: Durable retry outcomes (including reclaim-as-retry).
- `suspensions_total`: Durable suspend outcomes.
- `dead_letters_total`: Durable transitions into `dead_letter`.
- `dispatch_latency_ms_avg`: Average first-dispatch latency in milliseconds, when sampled.
- `handler_latency_ms_avg`: Average `onEvent` handler duration in milliseconds, when sampled.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventDeliveryRecord

```rust
pub struct EventDeliveryRecord {
    pub id: String,
    pub event_id: String,
    pub plugin_id: String,
    pub idempotency_key: String,
    pub state: String,
    pub attempt_count: i64,
    pub max_attempts: i64,
    pub lease_owner: Option<String>,
    pub lease_expires_at: Option<DateTime<Utc>>,
    pub lease_generation: i64,
    pub run_after: DateTime<Utc>,
    pub invocation_sequence: i64,
    pub resume_pending: bool,
    pub checkpoint_json: Option<String>,
    pub checkpoint_schema_version: i64,
    pub ordering_key: String,
    pub outcome: Option<String>,
    pub error_message: Option<String>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub cancel_requested: bool,
    pub resource_class: String,
    pub wake_event_type: String,
    pub wake_filter_json: String,
    pub wake_grants_json: String,
    pub queue_name: String,
}
```

Per-subscriber delivery row.

**Fields**

- `id`: Delivery id (`{event_id}:{plugin_id}`).
- `event_id`: Parent event id.
- `plugin_id`: Subscriber plugin id.
- `idempotency_key`: Stable idempotency key.
- `state`: Lifecycle state.
- `attempt_count`: Claim count (not incremented on resume).
- `max_attempts`: Max claims before dead-letter.
- `lease_owner`: Current lease owner.
- `lease_expires_at`: Lease expiry.
- `lease_generation`: Fence generation.
- `run_after`: Earliest claim time.
- `invocation_sequence`: Resume ordinal.
- `resume_pending`: Next claim should resume rather than increment attempt.
- `checkpoint_json`: Suspend checkpoint JSON.
- `checkpoint_schema_version`: Checkpoint schema version.
- `ordering_key`: FIFO key.
- `outcome`: Terminal outcome.
- `error_message`: Operator-facing error.
- `created_at`: Insert time.
- `updated_at`: Last-modified time.
- `cancel_requested`: Cooperative cancel flag for a running delivery.
- `resource_class`: Concurrency class (`network` today).
- `wake_event_type`: Event type that can wake this suspended delivery; empty = timestamp-only.
- `wake_filter_json`: Host-owned payload object filter JSON for event-wake; empty = type only.
- `wake_grants_json`: Host-derived wake grants JSON; empty falls back to type + `wake_filter_json`.
- `queue_name`: Queue of the parent event, copied at dispatch.

#### EventDeliveryRecord::fence

```rust
pub fn fence(&self) -> EventDeliveryFence
```

Fence for heartbeat / complete / fail / suspend.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### EventSubscriber

```rust
pub struct EventSubscriber {
    pub plugin_id: String,
    pub resource_class: String,
    pub max_attempts: Option<u32>,
}
```

Subscriber snapshot used when creating deliveries for one event.

**Fields**

- `plugin_id`: Plugin id that declared a matching subscription.
- `resource_class`: Concurrency class copied onto the delivery row.
- `max_attempts`: Claims before the delivery dead-letters; `None` uses the host default
  ([`EVENT_DELIVERY_MAX_ATTEMPTS`](#event_delivery_max_attempts)). Derived from the consumer's
  `max_retries + 1`.

#### EventSubscriber::plugin

```rust
pub fn plugin(plugin_id: impl Into<String>) -> Self
```

Catalog match for `plugin_id` with the default `network` resource class.

#### EventSubscriber::max_attempts_or_default

```rust
pub fn max_attempts_or_default(&self) -> i64
```

Effective `max_attempts` for a new delivery row.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventSubscriberCatalogRecord

```rust
pub struct EventSubscriberCatalogRecord {
    pub plugin_id: String,
    pub subscriptions: Vec<EventCatalogSubscription>,
    pub enabled: bool,
    pub updated_at: DateTime<Utc>,
}
```

Cluster-authoritative catalog row after collapsing live nodes by `plugin_id`.

**Fields**

- `plugin_id`: Canonical PluginKey of the subscriber (not the display alias).
- `subscriptions`: Union of subscriptions from live enabled nodes.
- `enabled`: Whether any live node has this plugin enabled.
- `updated_at`: Latest heartbeat among contributing live rows.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### EventSubscriberNodeRecord

```rust
pub struct EventSubscriberNodeRecord {
    pub node_id: String,
    pub plugin_id: String,
    pub subscriptions: Vec<EventCatalogSubscription>,
    pub enabled: bool,
    pub heartbeat_at: DateTime<Utc>,
}
```

Live per-node catalog row (`event_subscriber_nodes`).

**Fields**

- `node_id`: Daemon node id.
- `plugin_id`: Canonical PluginKey of the subscriber (not the display alias).
- `subscriptions`: Declared subscriptions on this node.
- `enabled`: Whether this node wants matching events for `plugin_id`.
- `heartbeat_at`: Last heartbeat from this node.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### EventWakeGrant

```rust
pub struct EventWakeGrant {
    pub schema_versions: Vec<u32>,
    pub filter: Option<Value>,
}
```

Host-derived wake grant: schema versions plus an intersected payload filter.

**Fields**

- `schema_versions`: Schema versions this grant may observe.
- `filter`: Intersected payload object filter; `None` / empty object matches any payload.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### GlobalQueueEntry

```rust
pub struct GlobalQueueEntry {
    pub work_key: String,
    pub title: String,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub cover_url: Option<String>,
    pub description: Option<String>,
    pub subtitle: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub genres: Option<String>,
    pub language: Option<String>,
    pub store_editions: Vec<WishlistStoreEdition>,
    pub purchase_hints: Vec<WishlistPurchaseHint>,
    pub wish_count: i64,
    pub wishers: Vec<QueueWisher>,
    pub sample_uuids: Vec<String>,
    pub first_requested_at: DateTime<Utc>,
    pub last_requested_at: DateTime<Utc>,
}
```

Aggregated global request-queue entry (one work, many wishers).

**Fields**

- `work_key`: Stable merge key used to group editions into a work.
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `cover_url`: HTTPS URL for cover art when known.
- `description`: Blurb / synopsis text (may contain HTML).
- `subtitle`: Optional subtitle from bibliographic metadata.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `published_at`: Publication date string from the storefront or enrichment.
- `genres`: Merged genre / category list for display.
- `language`: BCP-47 or storefront language code when known.
- `store_editions`: Distinct storefront editions contributing to this wishlist item.
- `purchase_hints`: Purchase URLs and prices observed per storefront.
- `wish_count`: Number of users who requested this title.
- `wishers`: Distinct people who have this title on an open wishlist.
- `sample_uuids`: Sample title-request UUIDs represented in this aggregate row.
- `first_requested_at`: Earliest request timestamp in this aggregate group.
- `last_requested_at`: Most recent request timestamp in this aggregate group.

#### GlobalQueueEntry::push_wisher

```rust
pub fn push_wisher(&mut self, wisher: QueueWisher)
```

Adds `wisher` when they are not already present (capped at 16).

Duplicate people keep the richer picture and name fields.

##### Arguments

* `wisher` - Person who has this title on an open wishlist.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### JobClaimCursor

```rust
pub struct JobClaimCursor {
    pub priority: i64,
    pub created_at: String,
    pub id: String,
}
```

Where the next page of routed-claim candidates starts (keyset paging).

Candidates are ordered by `priority DESC, created_at ASC, id ASC`; the
cursor is the last row of the previous page, so rows that leave the
pending state between pages never shift the window.

**Fields**

- `priority`: Priority of the last row read.
- `created_at`: `created_at` (RFC 3339 text) of the last row read.
- `id`: Id of the last row read.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### JobClaimPage

```rust
pub struct JobClaimPage {
    pub jobs: Vec<JobRecord>,
    pub next: Option<JobClaimCursor>,
}
```

One page of routed-claim candidates.

**Fields**

- `jobs`: Readable, not cancel-requested pending jobs, best first.
- `next`: Where the next page starts, or `None` when no row remains.

Implements: `Clone`, `Debug`, `Default`

### JobFence

```rust
pub struct JobFence {
    pub job_id: String,
    pub owner: String,
    pub generation: i64,
}
```

Lease identity a worker must present to mutate a running job.

**Fields**

- `job_id`: Job id that was claimed.
- `owner`: Worker id stored in `lease_owner`.
- `generation`: `lease_generation` assigned by the successful claim.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### JobPayload

```rust
pub struct JobPayload {
    pub v: u32,
    pub account: Option<String>,
    pub title: Option<String>,
    pub plugin_instance_id: Option<String>,
    pub requires: Vec<String>,
    pub trigger: JobTrigger,
    pub integration_id: Option<String>,
    pub force: bool,
    pub plugin_id: Option<String>,
    pub source_key: Option<String>,
    pub dest_key: Option<String>,
    pub checkpoint: Option<JobCheckpoint>,
    pub invocation_sequence: Option<u32>,
    pub resume_pending: bool,
    pub pdf_only: bool,
    pub requires_unsettled_since_ms: Option<i64>,
}
```

JSON command envelope stored on a job row.

**Fields**

- `v`: Envelope schema version; must equal [`JOB_PAYLOAD_VERSION`](#job_payload_version).
- `account`: Optional account filter (`None` = all eligible accounts).
- `title`: Optional title filter (uuid / ASIN / ISBN / product id).
- `plugin_instance_id`: Pins the plugin instance that serves the job. `None` routes to the
  plugin's default instance on the claiming host.
- `requires`: Plugin instance ids the claiming node must run, with its guest up.
  
  Recorded at admission (the pinned instance and any node-bound
  destination). Empty means any node may claim the job. The routed claim
  (`control_plane::claim_routed_job`) skips a job whose instances are not
  deployed and observed up on the claiming node, and the unrouted
  [`crate::LibraryStore::claim_next_job`](#librarystoreclaim_next_job) never claims a job that has any.
- `trigger`: Admission source (`api` or `scheduler`).
- `integration_id`: Integration plugin id for [`JobKind::IntegrationScan`](#jobkind).
- `force`: When true, an integration scan asks the remote to rescan, and an
  acquire re-downloads titles already acquired (run only by nodes that
  hold the library database).
- `plugin_id`: Destination plugin id for [`JobKind::PluginCopy`](#jobkind).
- `source_key`: Source object key for [`JobKind::PluginCopy`](#jobkind).
- `dest_key`: Destination object key for [`JobKind::PluginCopy`](#jobkind).
- `checkpoint`: Bounded checkpoint restored after `bookclerk_plugin_abi::JobOutcome::Suspended`.
- `invocation_sequence`: Resume ordinal; distinct from failure [`Self`](#jobpayload) attempt_count on the job row.
- `resume_pending`: One-shot: the next claim is a suspend-resume and must not consume a failure attempt.
  
  Cleared when that claim is taken. A leftover [`Self::checkpoint`](#jobpayload) after a
  retryable failure must not keep later claims in the resume path.
- `pdf_only`: For [`JobKind::Acquire`]: fetch only each title's companion PDF.
  
  Run only by nodes that hold the library database; a Host is never
  handed such a job (`control_plane::node_may_claim_job`).
- `requires_unsettled_since_ms`: Unix milliseconds when the requirements sweep first found this pending
  job's [`Self::requires`](#jobpayload) unsettled: no node runs all of them, or one of
  them is no longer deployed anywhere.
  
  Cleared when they settle again. The sweep fails the job (or, when only
  a dropped instance is involved, rewrites it) once it has been unsettled
  for its grace period, so a placement move that briefly leaves no common
  node does not fail queued work (`control_plane::sweep_job_requirements`).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobRecord

```rust
pub struct JobRecord {
    pub id: String,
    pub kind: JobKind,
    pub state: JobState,
    pub priority: i64,
    pub resource_class: JobResourceClass,
    pub payload: JobPayload,
    pub progress: Option<String>,
    pub attempt_count: i64,
    pub max_attempts: i64,
    pub run_after: DateTime<Utc>,
    pub lease_owner: Option<String>,
    pub lease_expires_at: Option<DateTime<Utc>>,
    pub dedup_key: String,
    pub error_kind: Option<String>,
    pub error_message: Option<String>,
    pub cancel_requested: bool,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub started_at: Option<DateTime<Utc>>,
    pub finished_at: Option<DateTime<Utc>>,
    pub lease_generation: i64,
}
```

Durable job row returned to the daemon API and worker.

**Fields**

- `id`: Stable job id (`scan-{uuid}`, …).
- `kind`: Job kind (`scan`, `acquire`, `listen_sync`).
- `state`: Lifecycle state.
- `priority`: Higher values are claimed first.
- `resource_class`: Concurrency class.
- `payload`: Kind-specific filters and trigger metadata.
- `progress`: Human-readable progress string.
- `attempt_count`: Number of times a worker has claimed this row.
- `max_attempts`: Maximum claims before a failure becomes terminal.
- `run_after`: Earliest time a pending job may be claimed.
- `lease_owner`: Worker id that currently holds the lease.
- `lease_expires_at`: Lease expiry; stale running rows are reclaimed after this.
- `dedup_key`: Idempotency key among pending/running rows.
- `error_kind`: Structured error kind when failed or cancelled.
- `error_message`: Operator-facing error text.
- `cancel_requested`: Cooperative cancel flag.
- `created_at`: When the row was inserted.
- `updated_at`: When the row was last modified.
- `started_at`: When a worker first claimed the row.
- `finished_at`: When the row reached a terminal state.
- `lease_generation`: Per-claim generation used to fence heartbeat and finalization.

#### JobRecord::fence

```rust
pub fn fence(&self) -> Option<JobFence>
```

Lease identity for heartbeat and finalization, when this row is claimed.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### JobTempPath

```rust
pub struct JobTempPath {
    pub id: i64,
    pub job_id: String,
    pub path: String,
    pub created_at: DateTime<Utc>,
    pub reserved_bytes: u64,
}
```

Scratch path registered against a job for crash cleanup.

**Fields**

- `id`: Surrogate primary key.
- `job_id`: Owning job id.
- `path`: Absolute filesystem path.
- `created_at`: When the path was registered.
- `reserved_bytes`: Bytes reserved against the job temp quota for this path.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### ListeningProgressRecord

```rust
pub struct ListeningProgressRecord {
    pub id: i64,
    pub identity_id: Option<i64>,
    pub provider: String,
    pub external_user_id: String,
    pub book_uuid: Option<String>,
    pub work_id: Option<String>,
    pub external_item_id: String,
    pub title: Option<String>,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub progress: Option<f64>,
    pub current_time_seconds: Option<f64>,
    pub duration_seconds: Option<f64>,
    pub is_finished: bool,
    pub last_listened_at: Option<DateTime<Utc>>,
    pub updated_at: DateTime<Utc>,
}
```

Listening progress snapshot from an external player (e.g. AudioBookshelf).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `identity_id`: Foreign key to `portal_identities.id`.
- `provider`: External identity or integration provider id (for example ABS).
- `external_user_id`: User id at the external provider.
- `book_uuid`: Foreign key to `books.uuid`.
- `work_id`: Canonical work id this edition or request resolves to.
- `external_item_id`: Provider-native listening-progress item id.
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `progress`: Fractional progress 0.0–1.0 when the provider reports it.
- `current_time_seconds`: Current playback position within the title, in seconds.
- `duration_seconds`: Total duration in seconds when known.
- `is_finished`: Whether the listener marked the title finished (0/1 or bool).
- `last_listened_at`: RFC 3339 time of the last playback update.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### OidcClientRecord

```rust
pub struct OidcClientRecord {
    pub id: i64,
    pub client_id: String,
    pub client_secret_hash: Option<String>,
    pub redirect_uris: Vec<String>,
    pub name: Option<String>,
    pub issue_refresh_token: bool,
    pub allowed_scopes: Vec<String>,
    pub enabled: bool,
    pub plugin_id: Option<String>,
    pub created_at: String,
}
```

Registered OIDC authorization-server client (Bookclerk as IdP).

**Fields**

- `id`: Surrogate primary key.
- `client_id`: Public client_id used at authorize / token.
- `client_secret_hash`: Hash of the client secret; `None` for public PKCE clients.
- `redirect_uris`: Allowed OAuth redirect URIs.
- `name`: Operator-facing display name.
- `issue_refresh_token`: When true, token responses include a refresh token.
- `allowed_scopes`: Scopes this client may be granted (`openid`, `profile`, `email`, …).
- `enabled`: When true, authorize and token endpoints accept this client.
- `plugin_id`: Plugin id that owns this client; `None` for operator-created clients.
- `created_at`: RFC 3339 timestamp when the row was inserted.

#### OidcClientRecord::has_secret

```rust
pub fn has_secret(&self) -> bool
```

True when a client secret hash is stored (confidential client).

#### OidcClientRecord::is_plugin_provided

```rust
pub fn is_plugin_provided(&self) -> bool
```

True when a plugin owns this client's identity and redirect URIs.

Implements: `Clone`, `Debug`

### OperatorSessionRecord

```rust
pub struct OperatorSessionRecord {
    pub id: i64,
    pub expires_at: DateTime<Utc>,
    pub created_at: DateTime<Utc>,
    pub last_used_at: Option<DateTime<Utc>>,
    pub elevated_from_user_id: Option<i64>,
    pub impersonating_user_id: Option<i64>,
    pub user_agent: Option<String>,
    pub device_type: Option<String>,
    pub client_label: Option<String>,
}
```

Durable operator session metadata (hashed token is never exposed).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `expires_at`: RFC 3339 expiry for the ticket, session, or code.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `last_used_at`: RFC 3339 time of the last authenticated use of this session.
- `elevated_from_user_id`: User id that elevated into this operator session, if any.
- `impersonating_user_id`: User id being impersonated by this operator session, if any.
- `user_agent`: Raw User-Agent captured at session mint (optional).
- `device_type`: Best-effort device class (`desktop` / `mobile` / `tablet` / `api`).
- `client_label`: Best-effort OS / client label (`Windows`, `Android`, `API`, …).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### PendingWakeProgress

```rust
pub struct PendingWakeProgress {
    pub claimed: u32,
    pub still_pending: bool,
}
```

How far one claimed wake tick got.

**Fields**

- `claimed`: Events this tick claimed and processed one delivery page for.
- `still_pending`: True when another tick should run (saved cursor or remaining flags).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### PortalIdentity

```rust
pub struct PortalIdentity {
    pub id: i64,
    pub provider: String,
    pub external_user_id: String,
    pub label: Option<String>,
    pub user_id: Option<i64>,
    pub created_at: DateTime<Utc>,
    pub picture_url: Option<String>,
}
```

Portal identity bound to an external provider user (e.g. ABS).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `provider`: External identity or integration provider id (for example ABS).
- `external_user_id`: User id at the external provider.
- `label`: Optional operator-facing display label.
- `user_id`: First-party user this external identity is linked to (Phase 1).
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `picture_url`: Last HTTPS profile picture URL from the identity provider, when any.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### PortalSessionRecord

```rust
pub struct PortalSessionRecord {
    pub id: i64,
    pub token_hash: String,
    pub created_at: String,
    pub expires_at: String,
    pub last_used_at: Option<String>,
    pub user_agent: Option<String>,
    pub device_type: Option<String>,
    pub client_label: Option<String>,
}
```

Portal session row for session lists (includes hash for current-session match).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `token_hash`: SHA-256 hex of the session token (never serialized to clients).
- `created_at`: RFC 3339 created timestamp.
- `expires_at`: RFC 3339 expiry timestamp.
- `last_used_at`: RFC 3339 last-used timestamp, when known.
- `user_agent`: Raw User-Agent captured at session mint (optional).
- `device_type`: Best-effort device class.
- `client_label`: Best-effort OS / client label.

Implements: `Clone`, `Debug`

### PublishDomainEventSpec

```rust
pub struct PublishDomainEventSpec {
    pub id: String,
    pub event_type: String,
    pub schema_version: i64,
    pub account_id: String,
    pub source: String,
    pub correlation_id: String,
    pub causation_id: String,
    pub dedup_key: String,
    pub payload: String,
    pub ordering_key: String,
    pub queue_name: String,
    pub content_type: String,
    pub delay_seconds: u32,
}
```

Producer-side domain event (or queue message) to persist in the outbox.

A domain event leaves `queue_name` empty (the bus, [`CORE_EVENTS_QUEUE`](#core_events_queue))
and sets `event_type`. A queue message names its queue and leaves
`event_type` empty; `content_type` says how `payload` is encoded and
`delay_seconds` holds its deliveries back.

**Fields**

- `id`: Stable event id (UUID). Generated when empty.
- `event_type`: Event type (`book_acquired`); empty for queue messages.
- `schema_version`: Payload schema version.
- `account_id`: Tenant / account id.
- `source`: Producer plugin id (`audible`, …); empty when unknown.
- `correlation_id`: Trace correlation id.
- `causation_id`: Causing event or job id.
- `dedup_key`: Unique with `event_type` (duplicate publishes coalesce).
- `payload`: Bounded JSON payload (never media bytes).
- `ordering_key`: FIFO key copied onto each delivery (`book uuid` for `book_acquired`).
- `queue_name`: Queue name; empty means the domain-event bus ([`CORE_EVENTS_QUEUE`](#core_events_queue)).
- `content_type`: Body encoding (`json`, `text`, `bytes`); empty means `json`.
- `delay_seconds`: Seconds before the first delivery may be claimed (queue messages).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### QueueWisher

```rust
pub struct QueueWisher {
    pub user_id: Option<i64>,
    pub identity_id: Option<i64>,
    pub display_name: Option<String>,
    pub login_name: Option<String>,
    pub operator: bool,
    pub has_avatar: bool,
    pub avatar_source: Option<String>,
    pub gravatar_hash: Option<String>,
    pub picture_url: Option<String>,
}
```

Compact wisher shown on the global queue (avatar + label).

**Fields**

- `user_id`: First-party `users.id` when the portal identity is linked.
- `identity_id`: Portal identity that created the wish (dedupe for unlinked identities).
- `display_name`: Display name, identity label, or `"Operator"`.
- `login_name`: Local login name when the first-party user has one.
- `operator`: True when the wish came from the operator token (no portal user).
- `has_avatar`: True when an uploaded avatar file exists (filled by the daemon).
- `avatar_source`: Explicit picture choice when the user has set one.
- `gravatar_hash`: SHA-256 hex of the contact email for Gravatar.
- `picture_url`: HTTPS picture from the wishing identity, when any.

#### QueueWisher::same_person

```rust
pub fn same_person(&self, other: &Self) -> bool
```

Returns true when both rows represent the same person for aggregation.

##### Arguments

* `other` - Candidate wisher to compare.

##### Returns

True for two operator wishes, the same `user_id`, or the same unlinked identity.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### SecurityAuditEvent

```rust
pub struct SecurityAuditEvent {
    pub id: i64,
    pub at: DateTime<Utc>,
    pub actor: String,
    pub action: String,
    pub detail_json: Option<String>,
}
```

Security audit event (elevate / impersonate / login / provision).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `at`: RFC 3339 timestamp of the audit event.
- `actor`: Actor label (user id, operator, or system).
- `action`: Audit action verb (for example `login`, `rotate_token`).
- `detail_json`: JSON object with structured event details (no secrets).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### StoredPasskey

```rust
pub struct StoredPasskey {
    pub id: i64,
    pub credential_id: String,
    pub passkey_json: String,
    pub name: Option<String>,
}
```

Stored WebAuthn credential (passkey JSON plus the label shown in Settings).

**Fields**

- `id`: Surrogate primary key.
- `credential_id`: Base64url credential id.
- `passkey_json`: Serialized `webauthn_rs` passkey JSON.
- `name`: Label chosen at registration; `None` for rows created before names existed.

Implements: `Clone`, `Debug`

### TitleRequestRecord

```rust
pub struct TitleRequestRecord {
    pub id: i64,
    pub uuid: String,
    pub identity_id: Option<i64>,
    pub title: String,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub notes: Option<String>,
    pub status: RequestStatus,
    pub work_key: String,
    pub work_id: Option<String>,
    pub resolved_book_uuid: Option<String>,
    pub cover_url: Option<String>,
    pub sources: Vec<TitleRequestSourceRecord>,
    pub description: Option<String>,
    pub subtitle: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub genres: Option<String>,
    pub language: Option<String>,
    pub store_editions: Vec<WishlistStoreEdition>,
    pub purchase_hints: Vec<WishlistPurchaseHint>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Personal wishlist row (also contributes to the shared global queue while open).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `uuid`: Stable UUID for this row (API / foreign-key identity).
- `identity_id`: Foreign key to `portal_identities.id`.
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `notes`: Free-form operator or requester notes.
- `status`: Lifecycle status for the row (user, request, …).
- `work_key`: Stable bibliographic key (`isbn:…` / `asin:…` / `soft:…`) for aggregation.
- `work_id`: Canonical work id this edition or request resolves to.
- `resolved_book_uuid`: Library `books.uuid` once the wishlist item is fulfilled.
- `cover_url`: HTTPS URL for cover art when known.
- `sources`: Per-storefront snapshots (empty for legacy rows).
- `description`: Blurb / synopsis text (may contain HTML).
- `subtitle`: Optional subtitle from bibliographic metadata.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `published_at`: Publication date string from the storefront or enrichment.
- `genres`: Merged genre / category list for display.
- `language`: BCP-47 or storefront language code when known.
- `store_editions`: Distinct storefront editions contributing to this wishlist item.
- `purchase_hints`: Purchase URLs and prices observed per storefront.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### TitleRequestSourceRecord

```rust
pub struct TitleRequestSourceRecord {
    pub id: i64,
    pub title_request_id: i64,
    pub source: String,
    pub product_id: String,
    pub title: Option<String>,
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub cover_url: Option<String>,
    pub url: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub list_price_cents: Option<i64>,
    pub list_price_label: Option<String>,
    pub member_price_cents: Option<i64>,
    pub member_price_label: Option<String>,
    pub observed_at: Option<DateTime<Utc>>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Per-storefront catalog/pricing snapshot attached to a wishlist row.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `title_request_id`: Foreign key to `title_requests.id`.
- `source`: Content-source plugin id (`audible`, `libro`, …).
- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).
- `title`: Display title of the work or edition.
- `subtitle`: Optional subtitle from bibliographic metadata.
- `authors`: Comma-separated or JSON author list from the storefront.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `description`: Blurb / synopsis text (may contain HTML).
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `published_at`: Publication date string from the storefront or enrichment.
- `categories`: Storefront category / genre path list (serialized).
- `language`: BCP-47 or storefront language code when known.
- `cover_url`: HTTPS URL for cover art when known.
- `url`: Storefront product or purchase URL when known.
- `price_cents`: Observed price in minor currency units, when known.
- `currency`: ISO 4217 currency code for price fields.
- `price_label`: Storefront-formatted price string for display.
- `list_price_cents`: List/MSRP price in minor units, when known.
- `list_price_label`: Storefront-formatted list price for display.
- `member_price_cents`: Member/subscriber price in minor units, when known.
- `member_price_label`: Storefront-formatted member price for display.
- `observed_at`: RFC 3339 time when this storefront snapshot was observed.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### UserIntegrationHint

```rust
pub struct UserIntegrationHint {
    pub source: String,
    pub account_id: String,
    pub label: Option<String>,
}
```

Storefront / integration connection summary for admin user lists.

**Fields**

- `source`: Content-source or integration plugin id (`audible`, `libro`, …).
- `account_id`: Store or operator account id this row belongs to.
- `label`: Optional operator-facing account label.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserInviteRecord

```rust
pub struct UserInviteRecord {
    pub id: i64,
    pub token_hash: String,
    pub role: UserRole,
    pub login_name: Option<String>,
    pub display_name: Option<String>,
    pub expires_at: DateTime<Utc>,
    pub redeemed_at: Option<DateTime<Utc>>,
    pub created_by: String,
    pub created_at: DateTime<Utc>,
}
```

Invite ticket for provisioning a User (token plaintext never stored).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `token_hash`: SHA-256 hex digest of the opaque token (plaintext never stored).
- `role`: First-party role (`administrator` or `member`).
- `login_name`: Local username for password login, when set.
- `display_name`: Human-readable name shown in the UI.
- `expires_at`: RFC 3339 expiry for the ticket, session, or code.
- `redeemed_at`: RFC 3339 time when the ticket/invite was redeemed, if any.
- `created_by`: Actor that created the row (user id or operator label).
- `created_at`: RFC 3339 timestamp when the row was inserted.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserListeningHint

```rust
pub struct UserListeningHint {
    pub title: Option<String>,
    pub provider: String,
    pub last_listened_at: DateTime<Utc>,
}
```

Recent unfinished listening hint for admin user lists.

**Fields**

- `title`: Display title when the provider reported one.
- `provider`: Integration / storefront provider id.
- `last_listened_at`: RFC 3339 time of the last playback update.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserPreferences

```rust
pub struct UserPreferences {
    pub id: i64,
    pub subject_key: String,
    pub identity_id: Option<i64>,
    pub default_view: String,
    pub disabled_shelves: Vec<String>,
    pub discover_sort: String,
    pub discover_sort_dir: String,
    pub discover_language: Option<String>,
    pub discover_excluded_sources: Vec<String>,
    pub theme: String,
    pub updated_at: DateTime<Utc>,
}
```

Per-user GUI / Discover preferences (operator or portal identity).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `subject_key`: Preference subject key (`operator`, `user:<id>`, portal key).
- `identity_id`: Foreign key to `portal_identities.id`.
- `default_view`: Preferred library SPA view identifier (for example `grid` or `list`).
- `disabled_shelves`: Shelf kind ids to hide (`author`, `chirp_deals`, …). Empty = all on.
- `discover_sort`: Preferred Discover sort column (for example `title` or `added`).
- `discover_sort_dir`: Discover sort direction (`asc` / `desc`).
- `discover_language`: Preferred content language (`en`, `__all__`, …). `None` = browser default.
- `discover_excluded_sources`: Store ids to hide in Discover. Empty = all sources (including future).
- `theme`: Appearance preference (`system`, `light`, or `dark`).
- `updated_at`: RFC 3339 timestamp when the row was last modified.

#### UserPreferences::defaults_for

```rust
pub fn defaults_for(subject_key: &str, identity_id: Option<i64>) -> Self
```

Defaults when no row exists yet.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserPresenceExtras

```rust
pub struct UserPresenceExtras {
    pub online: bool,
    pub listening: Option<UserListeningHint>,
    pub integrations: Vec<UserIntegrationHint>,
    pub last_active_at: Option<DateTime<Utc>>,
}
```

Presence + connection extras for administrator user management.

**Fields**

- `online`: True when the user has a non-expired portal session.
- `listening`: Most recent unfinished listen within the listening window, if any.
- `integrations`: Linked storefront / integration accounts across portal identities.
- `last_active_at`: Most recent *unexpired* portal session activity (RFC 3339), when known.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserRecord

```rust
pub struct UserRecord {
    pub id: i64,
    pub role: UserRole,
    pub status: UserStatus,
    pub display_name: Option<String>,
    pub login_name: Option<String>,
    pub email: Option<String>,
    pub has_password: bool,
    pub security_version: i64,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub last_seen_at: Option<DateTime<Utc>>,
    pub avatar_source: Option<String>,
    pub totp_enabled: bool,
}
```

First-party Bookclerk user (security principal for portal paths).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `role`: First-party role (`owner`, `administrator`, `host`, or `member`).
- `status`: Lifecycle status for the row (user, request, …).
- `display_name`: Human-readable name shown in the UI.
- `login_name`: Local username for password login, when set.
- `email`: Optional contact email (notifications / magic-link invites later).
- `has_password`: Whether a local password hash is stored for this user.
- `security_version`: Incremented to invalidate existing sessions after security changes.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.
- `last_seen_at`: Last authenticated portal activity (RFC 3339); survives session expiry.
- `avatar_source`: Explicit picture choice (`monogram` / `gravatar` / `upload` / `sso:{id}`); `None` is auto.
- `totp_enabled`: True when a TOTP authenticator has been confirmed for local password login.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserSsoPicture

```rust
pub struct UserSsoPicture {
    pub identity_id: i64,
    pub provider: String,
    pub picture_url: String,
    pub last_used_at: Option<DateTime<Utc>>,
}
```

IdP-supplied avatar available as a profile-picture choice.

**Fields**

- `identity_id`: `portal_identities.id` this picture belongs to.
- `provider`: Identity-broker id (`oidc:google`, `oidc:github`, …).
- `picture_url`: HTTPS URL to display (hotlinked from the IdP / CDN).
- `last_used_at`: Most recent portal session activity for this identity, when any.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### WishlistPurchaseHint

```rust
pub struct WishlistPurchaseHint {
    pub source: String,
    pub product_id: String,
    pub title: Option<String>,
    pub url: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub list_price_cents: Option<i64>,
    pub list_price_label: Option<String>,
    pub member_price_cents: Option<i64>,
    pub member_price_label: Option<String>,
}
```

Snapshotted purchase link/price for a wishlist title.

**Fields**

- `source`: Content-source plugin id (`audible`, `libro`, …).
- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).
- `title`: Display title of the work or edition.
- `url`: Storefront product or purchase URL when known.
- `price_cents`: Observed price in minor currency units, when known.
- `currency`: ISO 4217 currency code for price fields.
- `price_label`: Storefront-formatted price string for display.
- `list_price_cents`: List/MSRP price in minor units, when known.
- `list_price_label`: Storefront-formatted list price for display.
- `member_price_cents`: Member/subscriber price in minor units, when known.
- `member_price_label`: Storefront-formatted member price for display.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### WishlistStoreEdition

```rust
pub struct WishlistStoreEdition {
    pub source: String,
    pub product_id: String,
}
```

Storefront edition key for wishlist / queue payloads.

**Fields**

- `source`: Content-source plugin id (`audible`, `libro`, …).
- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### WorkRecord

```rust
pub struct WorkRecord {
    pub id: String,
    pub canonical_asin: Option<String>,
    pub canonical_isbn: Option<String>,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub description: Option<String>,
    pub subjects: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub cover_url: Option<String>,
    pub openlibrary_id: Option<String>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Canonical work spanning one or more ownership rows / editions.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `canonical_asin`: Preferred ASIN representing this canonical work.
- `canonical_isbn`: Preferred ISBN representing this canonical work.
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `narrators`: Comma-separated or JSON narrator list when present.
- `description`: Blurb / synopsis text (may contain HTML).
- `subjects`: Subject / topic tags from enrichment (serialized).
- `categories`: Storefront category / genre path list (serialized).
- `language`: BCP-47 or storefront language code when known.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `cover_url`: HTTPS URL for cover art when known.
- `openlibrary_id`: Open Library work/edition id when enrichment found one.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### SchemaWalk

```rust
pub struct SchemaWalk {
    pub from: i64,
    pub requested_to: i64,
    pub stopped_at: i64,
    pub ups: Vec<HostMigrationStep>,
    pub downs: Vec<HostMigrationStep>,
    pub blocked: bool,
}
```

Result of walking a host migration plan from one version to another.

**Fields**

- `from`: Database version before the walk.
- `requested_to`: Version the caller asked to reach.
- `stopped_at`: Version the walk can actually reach (may be higher than `requested_to`
  when an irreversible step blocks a downgrade).
- `ups`: Remaining `up` steps in order (empty when rolling back).
- `downs`: `down` steps newest-first (empty when upgrading).
- `blocked`: True when a missing `down` stopped a rollback short of `requested_to`.

#### SchemaWalk::is_noop

```rust
pub fn is_noop(&self) -> bool
```

True when no DDL remains to apply.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### SessionClientInfo

```rust
pub struct SessionClientInfo {
    pub user_agent: Option<String>,
    pub device_type: String,
    pub client_label: String,
}
```

Parsed client presentation for a session row.

**Fields**

- `user_agent`: Raw User-Agent (or synthetic `API` / empty).
- `device_type`: High-level device class: `desktop`, `mobile`, `tablet`, `api`, or `unknown`.
- `client_label`: Human label such as `Windows`, `macOS`, `Android`, `iPhone`, or `API`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PluginDatabaseRecord

```rust
pub struct PluginDatabaseRecord {
    pub plugin_id: String,
    pub binding: String,
    pub adapter_plugin_key: String,
    pub backend_kind: String,
    pub unit_ref: String,
    pub created_at: String,
}
```

One provisioned plugin database binding registry row.

**Fields**

- `plugin_id`: Owning plugin PluginKey (canonical text).
- `binding`: Binding name from `plugin.toml` `capabilities.bindings.databases`.
- `adapter_plugin_key`: Adapter PluginKey that provisioned the physical unit.
- `backend_kind`: Diagnostic adapter family (`sqlite`, `postgres`, `d1`, or custom).
- `unit_ref`: Backend-native unit: file path, schema name, or D1 database name.
- `created_at`: RFC 3339 provisioning time.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### KvEntry

```rust
pub struct KvEntry {
    pub key: String,
    pub value: Vec<u8>,
    pub metadata: String,
    pub expires_at: Option<DateTime<Utc>>,
}
```

One live key/value row.

**Fields**

- `key`: Key within the namespace.
- `value`: Value bytes.
- `metadata`: Opaque JSON metadata text; empty when none.
- `expires_at`: Absolute expiry, when set.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### KvListPage

```rust
pub struct KvListPage {
    pub keys: Vec<KvEntry>,
    pub list_complete: bool,
    pub cursor: String,
}
```

One page of [`LibraryStore::kv_list`](#librarystorekv_list).

**Fields**

- `keys`: Keys in lexical order (values are not loaded).
- `list_complete`: True when no further page exists.
- `cursor`: Opaque cursor for the next page; empty when complete.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### KvWriteSpec

```rust
pub struct KvWriteSpec {
    pub value: Vec<u8>,
    pub metadata: String,
    pub expires_at: Option<DateTime<Utc>>,
    pub max_value_bytes: u32,
    pub max_keys: u32,
}
```

A write plus the namespace limits it must satisfy.

**Fields**

- `value`: Value bytes.
- `metadata`: Opaque JSON metadata text; empty when none.
- `expires_at`: Absolute expiry; `None` never expires.
- `max_value_bytes`: Largest value this namespace accepts.
- `max_keys`: Most live keys this namespace holds.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### BookPage

```rust
pub struct BookPage {
    pub books: Vec<BookRecord>,
    pub total: usize,
}
```

One page of books plus the unpaged match count.

**Fields**

- `books`: Rows after `ORDER BY` and `LIMIT`/`OFFSET`.
- `total`: Rows matching the filter before pagination.

Implements: `Clone`, `Debug`

### CatalogEnrichmentFields

```rust
pub struct CatalogEnrichmentFields {
    pub description: Option<String>,
    pub language: Option<String>,
    pub cover_url: Option<String>,
    pub subjects: Option<String>,
    pub categories: Option<String>,
    pub enrich_source: Option<String>,
    pub enrich_confidence: Option<f64>,
    pub enrich_updated_at: Option<DateTime<Utc>>,
}
```

Durable catalog enrichment fields (blurbs, subjects, provenance).

**Fields**

- `description`: Blurb / synopsis text (may contain HTML).
- `language`: BCP-47 or storefront language code when known.
- `cover_url`: HTTPS URL for cover art when known.
- `subjects`: Subject / topic tags from enrichment (serialized).
- `categories`: Storefront category / genre path list (serialized).
- `enrich_source`: Plugin or catalog that last enriched bibliographic fields.
- `enrich_confidence`: 0–1 confidence score for the last enrichment pass.
- `enrich_updated_at`: RFC 3339 time of the last enrichment write.

Implements: `Clone`, `Debug`, `Default`

### LibraryStore

```rust
pub struct LibraryStore {
    /* private fields */
}
```

Handle to the Bookclerk library database.

Async API over a SeaORM [`DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html). `DatabaseConnection` is
cheaply cloneable (shared connection), so `LibraryStore` is `Clone`.

#### LibraryStore::scope

```rust
pub fn scope(
    &self,
    plugin_key: impl Into<String>,
    source_id: impl Into<String>,
) -> SourceScope
```

Plugin-scoped view — first-party and third-party sources use this.

`plugin_key` isolates rows; `source_id` is the display alias written
to the `source` column.

#### LibraryStore::publish_domain_event

```rust
pub async fn publish_domain_event(
    &self,
    spec: PublishDomainEventSpec,
) -> Result<PublishDomainEventOutcome>
```

Persist a domain event. Duplicate `(account_id, source, event_type, dedup_key)`
coalesces. Wake processing is left to [`Self::process_pending_wakes`](#librarystoreprocess_pending_wakes).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails, or
[`LibraryError::Other`](#libraryerror) when the payload exceeds 64 KiB.

#### LibraryStore::dispatch_event_deliveries

```rust
pub async fn dispatch_event_deliveries(
    &self,
    event_id: &str,
    subscribers: &[EventSubscriber],
    operation_id: &str,
) -> Result<u32>
```

Snapshot matching subscribers and create one delivery row per plugin.

Idempotent under replay: existing `(event_id, plugin_id)` rows are kept.
Marks the outbox row `dispatched` in the same transaction.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::next_undispatched_event

```rust
pub async fn next_undispatched_event(&self) -> Result<Option<DomainEventRecord>>
```

Next undispatched outbox row (oldest first).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::process_pending_wakes

```rust
pub async fn process_pending_wakes(
    &self,
    limit: u64,
    _owner: &str,
    lease_secs: u64,
) -> Result<PendingWakeProgress>
```

Claim up to `limit` `wake_pending` events and process one delivery page each.

Each claimed event is leased with a unique UUID token so two processors
cannot own the same slice (process `node_id` is not the fence — it is ABA
across restarts). Cursor release and finish require that token.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read or wake write fails.

#### LibraryStore::claim_next_event_delivery

```rust
pub async fn claim_next_event_delivery(
    &self,
    owner: &str,
    lease_secs: u64,
    operation_id: &str,
    plugin_ids: &[String],
    max_in_flight: u32,
    node_id: &str,
) -> Result<Option<EventDeliveryRecord>>
```

Claim the next ready delivery with a fenced `pending` → `running` mutation.

When `node_id` is non-empty, this node’s own catalog must match the parent
event’s type, schema version, and filter. The host loads the catalog and
filters candidates in Rust, then CAS-updates one row (portable slot +
`UPDATE … WHERE id = ? AND state = pending`). An empty `node_id` keeps
plugin-id-only filtering (tests).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the claim write fails.

#### LibraryStore::claim_next_event_delivery_due_by

```rust
pub async fn claim_next_event_delivery_due_by(
    &self,
    owner: &str,
    lease_secs: u64,
    operation_id: &str,
    plugin_ids: &[String],
    max_in_flight: u32,
    node_id: &str,
    due_by: DateTime<Utc>,
) -> Result<Option<EventDeliveryRecord>>
```

[`Self::claim_next_event_delivery`](#librarystoreclaim_next_event_delivery) restricted to deliveries due at
or before `due_by` (`run_after <= due_by`).

A host that is not the best placed to run a plugin passes a cutoff in
the past, so it only picks up deliveries the better hosts left
waiting that long.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the claim write fails.

#### LibraryStore::release_unexecuted_event_delivery

```rust
pub async fn release_unexecuted_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    restore_resume: bool,
) -> Result<bool>
```

Return a claimed delivery to `pending` without consuming an attempt.

Used when this worker cannot execute the subscriber (plugin not loaded).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::heartbeat_event_delivery

```rust
pub async fn heartbeat_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    lease_secs: u64,
) -> Result<bool>
```

Refresh the lease when `fence` still owns the running generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::ack_event_delivery

```rust
pub async fn ack_event_delivery(&self, fence: &EventDeliveryFence) -> Result<bool>
```

Mark a fenced delivery `acked`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::reject_event_delivery

```rust
pub async fn reject_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    reason: &str,
) -> Result<bool>
```

Mark a fenced delivery permanently `rejected`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::dead_letter_event_delivery

```rust
pub async fn dead_letter_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    reason: &str,
) -> Result<bool>
```

Mark a fenced delivery `dead_letter`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::retry_event_delivery

```rust
pub async fn retry_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    run_after: DateTime<Utc>,
    reason: &str,
) -> Result<bool>
```

Return a fenced delivery to `pending` with backoff (retryable handler).

When `attempt_count` has already reached `max_attempts`, the delivery is
dead-lettered instead of looping forever on a guest that keeps returning
retry.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::suspend_event_delivery

```rust
pub async fn suspend_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    checkpoint_json: &str,
    checkpoint_schema_version: i64,
    wake_at: DateTime<Utc>,
    wake_event_type: &str,
    wake_filter_json: &str,
    wake_grants_json: &str,
) -> Result<bool>
```

Suspend a fenced delivery (checkpoint + wake) without burning an attempt.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::fail_event_delivery

```rust
pub async fn fail_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    reason: &str,
) -> Result<bool>
```

Fail a running delivery: retry with backoff or dead-letter at max attempts.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::reclaim_expired_event_deliveries

```rust
pub async fn reclaim_expired_event_deliveries(&self) -> Result<u32>
```

Reclaim expired running deliveries back to `pending`.

Rows with `cancel_requested` become `rejected` instead of pending so an
operator cancel survives a worker crash. A stored checkpoint restores
`resume_pending` so a crash during resume does not consume an attempt.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::get_domain_event

```rust
pub async fn get_domain_event(&self, id: &str) -> Result<Option<DomainEventRecord>>
```

Load one event envelope.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::get_event_delivery

```rust
pub async fn get_event_delivery(&self, id: &str) -> Result<Option<EventDeliveryRecord>>
```

Load one delivery.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_domain_events

```rust
pub async fn list_domain_events(&self, limit: u64) -> Result<Vec<DomainEventRecord>>
```

Recent outbox events, newest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_domain_events_for_queue

```rust
pub async fn list_domain_events_for_queue(
    &self,
    queue_name: &str,
    limit: u64,
) -> Result<Vec<DomainEventRecord>>
```

Recent outbox rows on one queue, newest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_queue_names

```rust
pub async fn list_queue_names(&self) -> Result<Vec<String>>
```

Distinct queue names that hold at least one outbox row, sorted.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::count_queue_deliveries

```rust
pub async fn count_queue_deliveries(&self, queue_name: &str, state: &str) -> Result<i64>
```

Delivery rows on one queue in `state`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_event_deliveries_for_queue

```rust
pub async fn list_event_deliveries_for_queue(
    &self,
    queue_name: &str,
    state: Option<&str>,
    limit: u64,
) -> Result<Vec<EventDeliveryRecord>>
```

Deliveries filtered by optional state and queue, newest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_event_deliveries

```rust
pub async fn list_event_deliveries(
    &self,
    state: Option<&str>,
    limit: u64,
) -> Result<Vec<EventDeliveryRecord>>
```

Deliveries filtered by optional state, newest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::retry_dead_letter_delivery

```rust
pub async fn retry_dead_letter_delivery(&self, id: &str) -> Result<bool>
```

Re-queue a dead-lettered delivery for operator retry.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::acknowledge_dead_letter_delivery

```rust
pub async fn acknowledge_dead_letter_delivery(&self, id: &str) -> Result<bool>
```

Operator acknowledge: mark a dead-letter as rejected (no further retry).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::count_event_deliveries

```rust
pub async fn count_event_deliveries(&self, state: &str) -> Result<i64>
```

Count deliveries in `state`.

A peer writer on the same SQLite file can make this read return
`SQLITE_BUSY`. The read is repeated until the lock clears or the
attempt budget is spent. `state` is unchanged.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::event_delivery_metrics

```rust
pub async fn event_delivery_metrics(&self) -> Result<EventDeliveryMetrics>
```

Operator-visible delivery-queue counters (pending vs suspended split).

Each read waits out a peer SQLite file lock the same way
[`Self::count_event_deliveries`](#librarystorecount_event_deliveries) does.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails for a reason other than
file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when a lock is
still held after the retries.

#### LibraryStore::record_event_handler_latency

```rust
pub async fn record_event_handler_latency(&self, duration_ms: i64) -> Result<()>
```

Record one `onEvent` duration sample.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::list_domain_events_by_dispatch_state

```rust
pub async fn list_domain_events_by_dispatch_state(
    &self,
    dispatch_state: &str,
    limit: u64,
) -> Result<Vec<DomainEventRecord>>
```

Outbox envelopes in `dispatch_state`, oldest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_dispatched_events_page

```rust
pub async fn list_dispatched_events_page(
    &self,
    created_after: &str,
    after: Option<(&str, &str)>,
    limit: u64,
) -> Result<Vec<DomainEventRecord>>
```

Dispatched events newer than `created_after`, after optional `(created_at, id)` cursor.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::upsert_event_subscriber

```rust
pub async fn upsert_event_subscriber(
    &self,
    node_id: &str,
    plugin_id: &str,
    subscriptions: &[EventCatalogSubscription],
    enabled: bool,
) -> Result<()>
```

Heartbeat one node's plugin registration (does not delete other nodes).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::upsert_event_subscriber_at

```rust
pub async fn upsert_event_subscriber_at(
    &self,
    node_id: &str,
    plugin_id: &str,
    subscriptions: &[EventCatalogSubscription],
    enabled: bool,
    heartbeat_at: DateTime<Utc>,
) -> Result<()>
```

Heartbeat one node's plugin registration at an explicit timestamp.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::list_live_event_subscribers

```rust
pub async fn list_live_event_subscribers(
    &self,
) -> Result<Vec<EventSubscriberCatalogRecord>>
```

Live catalog collapsed by `plugin_id` using the default heartbeat TTL.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_live_event_subscribers_with_ttl

```rust
pub async fn list_live_event_subscribers_with_ttl(
    &self,
    ttl_secs: i64,
) -> Result<Vec<EventSubscriberCatalogRecord>>
```

Live catalog collapsed by `plugin_id` for a custom heartbeat TTL.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_live_event_subscriber_nodes

```rust
pub async fn list_live_event_subscriber_nodes(
    &self,
    ttl_secs: i64,
) -> Result<Vec<EventSubscriberNodeRecord>>
```

Live per-node rows whose heartbeat is within `ttl_secs`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::dispatch_catalog_matches

```rust
pub async fn dispatch_catalog_matches(
    &self,
    event: &DomainEventRecord,
    operation_id: &str,
) -> Result<u32>
```

Create deliveries from the live catalog for one event.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::reconcile_catalog_deliveries

```rust
pub async fn reconcile_catalog_deliveries(
    &self,
    created_after: DateTime<Utc>,
) -> Result<u32>
```

Late-join catalog deliveries for missing `(event_id, plugin_id)` pairs.

Pages an anti-join of dispatched events in the retention window that
lack a delivery for each live plugin. Unchanged catalogs with no missing
pairs issue bounded empty `SELECT`s and **zero** dispatch writes.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or write fails.

#### LibraryStore::request_event_delivery_cancel

```rust
pub async fn request_event_delivery_cancel(
    &self,
    id: &str,
) -> Result<Option<EventDeliveryRecord>>
```

Cancel a pending delivery or flag a running one for cooperative stop.

Pending (including suspended) rows become `rejected`. Running rows set
`cancel_requested` so the heartbeat loop aborts `onEvent`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::event_delivery_cancel_requested

```rust
pub async fn event_delivery_cancel_requested(&self, id: &str) -> Result<bool>
```

True when a running delivery worker should abort after the current step.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::resume_event_delivery

```rust
pub async fn resume_event_delivery(&self, id: &str) -> Result<bool>
```

Wake a suspended delivery: `run_after = now` and `resume_pending = 1`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::prune_event_deliveries

```rust
pub async fn prune_event_deliveries(
    &self,
    retention_days: u64,
    dead_letter_retention_days: u64,
) -> Result<u64>
```

Delete terminal deliveries past their retention windows, then empty events.

Acked/rejected rows use `retention_days`. Dead letters use
`dead_letter_retention_days`. Dispatched parent events with no remaining
deliveries are removed only after the same `retention_days` cutoff.
Expired per-node catalog heartbeats are swept here too. Runs every
[`EventPruneStep`](#eventprunestep) in order; a caller that must recheck a lease between
deletes runs [`Self::prune_event_step`](#librarystoreprune_event_step) itself.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::prune_event_step

```rust
pub async fn prune_event_step(
    &self,
    step: EventPruneStep,
    retention_days: u64,
    dead_letter_retention_days: u64,
) -> Result<u64>
```

Runs one delete of [`Self::prune_event_deliveries`](#librarystoreprune_event_deliveries).

##### Arguments

* `step` - Which rows to delete.
* `retention_days` - Window for acked/rejected deliveries and empty
  dispatched events.
* `dead_letter_retention_days` - Window for dead letters.

##### Returns

The number of rows removed.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::record_plugin_database

```rust
pub async fn record_plugin_database(
    &self,
    plugin_id: &str,
    binding: &str,
    adapter_plugin_key: &str,
    backend_kind: &str,
    unit_ref: &str,
) -> Result<PluginDatabaseRecord>
```

Records one provisioned binding unit, keeping any existing row.

Returns the authoritative row (an earlier registration wins so a
re-open never silently re-targets a binding at a different unit).

##### Errors

Returns an error when the insert or lookup fails.

#### LibraryStore::get_plugin_database

```rust
pub async fn get_plugin_database(
    &self,
    plugin_id: &str,
    binding: &str,
) -> Result<Option<PluginDatabaseRecord>>
```

The registry row for `(plugin_id, binding)`, if provisioned.

##### Errors

Returns an error when the query fails.

#### LibraryStore::list_plugin_databases

```rust
pub async fn list_plugin_databases(
    &self,
    plugin_id: Option<&str>,
) -> Result<Vec<PluginDatabaseRecord>>
```

All registry rows, optionally filtered to one owner PluginKey.

##### Errors

Returns an error when the query fails.

#### LibraryStore::remove_plugin_databases

```rust
pub async fn remove_plugin_databases(
    &self,
    plugin_id: &str,
    binding: Option<&str>,
) -> Result<u64>
```

Deletes registry rows for a plugin (one binding, or all of them).

Removes only the registry entry; dropping the underlying unit is the
caller's responsibility (operator CLI / uninstall flow).

##### Errors

Returns an error when the delete fails.

#### LibraryStore::rebind_plugin_database

```rust
pub async fn rebind_plugin_database(
    &self,
    plugin_id: &str,
    binding: &str,
    adapter_plugin_key: &str,
    backend_kind: &str,
    unit_ref: &str,
) -> Result<PluginDatabaseRecord>
```

Inserts or updates placement for one `(plugin_id, binding)` on restore.

Logical identity is portable; `adapter_plugin_key` / `backend_kind` /
`unit_ref` are the **target** adapter's physical placement, never the
source's.

##### Errors

Returns when the upsert fails.

#### LibraryStore::kv_get

```rust
pub async fn kv_get(
    &self,
    plugin_key: &str,
    namespace: &str,
    key: &str,
) -> Result<Option<KvEntry>>
```

Reads one key; `None` when absent or expired.

##### Errors

Returns an error when the query fails.

#### LibraryStore::kv_put

```rust
pub async fn kv_put(
    &self,
    plugin_key: &str,
    namespace: &str,
    key: &str,
    spec: KvWriteSpec,
) -> Result<KvPutOutcome>
```

Writes one key, replacing any previous value, metadata, and expiry.

Enforces `spec.max_value_bytes` and `spec.max_keys` (expired rows do
not count and are reclaimed to make room).

##### Errors

Returns an error when the write fails.

#### LibraryStore::kv_delete

```rust
pub async fn kv_delete(
    &self,
    plugin_key: &str,
    namespace: &str,
    key: &str,
) -> Result<bool>
```

Deletes one key. Returns whether a live value existed.

##### Errors

Returns an error when the delete fails.

#### LibraryStore::kv_list

```rust
pub async fn kv_list(
    &self,
    plugin_key: &str,
    namespace: &str,
    prefix: &str,
    cursor: &str,
    limit: u32,
) -> Result<KvListPage>
```

Lists live keys in lexical order, `limit` per page (0 means
[`KV_LIST_DEFAULT_LIMIT`](#kv_list_default_limit)). `cursor` is the opaque value a previous
page returned; empty starts from the beginning.

##### Errors

Returns an error when the query fails or the cursor is malformed.

#### LibraryStore::kv_count

```rust
pub async fn kv_count(&self, plugin_key: &str, namespace: &str) -> Result<u64>
```

Live keys in one namespace.

##### Errors

Returns an error when the query fails.

#### LibraryStore::kv_prune_expired_in

```rust
pub async fn kv_prune_expired_in(&self, plugin_key: &str, namespace: &str) -> Result<u64>
```

Removes expired rows of one namespace. Returns the rows removed.

##### Errors

Returns an error when the delete fails.

#### LibraryStore::kv_prune_expired

```rust
pub async fn kv_prune_expired(&self) -> Result<u64>
```

Removes every expired row across all plugins and namespaces.

##### Errors

Returns an error when the delete fails.

#### LibraryStore::kv_delete_plugin

```rust
pub async fn kv_delete_plugin(&self, plugin_key: &str) -> Result<u64>
```

Removes every row of one plugin (retire / uninstall).

##### Errors

Returns an error when the delete fails.

#### LibraryStore::storage_scan_put_object

```rust
pub async fn storage_scan_put_object(
    &self,
    scan_id: &str,
    key: &str,
    size: u64,
    media_rank: u8,
    is_audio: bool,
) -> Result<()>
```

Inserts an object row. A repeat of the same key is a no-op.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::storage_scan_put_identity

```rust
pub async fn storage_scan_put_identity(
    &self,
    scan_id: &str,
    identity: &str,
    key: &str,
    size: u64,
    media_rank: u8,
) -> Result<()>
```

Inserts an identity → key row. Repeats are no-ops.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::storage_scan_identity_page

```rust
pub async fn storage_scan_identity_page(
    &self,
    scan_id: &str,
    identity: &str,
    after: Option<(i64, &str)>,
    limit: u64,
) -> Result<Vec<(String, i64)>>
```

One page of identity candidates, ordered by `(media_rank, object_key)`.

`after` is the last key already considered. The page is bounded by
`limit` (clamped to 1..=64). Callers try each key and request the next
page; this does not load every match.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::storage_scan_object_page

```rust
pub async fn storage_scan_object_page(
    &self,
    scan_id: &str,
    prefix: &str,
    after_key: Option<&str>,
    limit: u64,
) -> Result<Vec<String>>
```

One page of object keys at or above `prefix`, in `object_key` order.

The upper bound is the byte-wise successor of `prefix`, not a SQL `LIKE`
pattern, so wildcard characters in a storage key stay literal.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::storage_scan_claim

```rust
pub async fn storage_scan_claim(&self, scan_id: &str, key: &str) -> Result<()>
```

Marks an object claimed so unmatched-audio counts stay in the database.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::storage_scan_unclaimed_audio

```rust
pub async fn storage_scan_unclaimed_audio(&self, scan_id: &str) -> Result<u64>
```

Audio objects in the scan that no library row claimed.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the count fails.

#### LibraryStore::storage_scan_delete

```rust
pub async fn storage_scan_delete(&self, scan_id: &str) -> Result<()>
```

Deletes inventory rows and the generation ownership row for `scan_id`.

Both deletes commit together. A failure leaves the generation and its
rows in place so an apply checkpoint cannot adopt an empty inventory.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::storage_scan_register

```rust
pub async fn storage_scan_register(
    &self,
    scan_id: &str,
    instance_id: &str,
    job_id: Option<&str>,
) -> Result<()>
```

Binds `scan_id` to `instance_id` and an optional job.

A second register for the same id and instance refreshes the heartbeat.
A different instance fails closed so a reused id cannot adopt another scan.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails, or
[`LibraryError::Other`](#libraryerror) when `instance_id` does not match the existing row.

#### LibraryStore::storage_scan_mark_complete

```rust
pub async fn storage_scan_mark_complete(&self, scan_id: &str) -> Result<()>
```

Marks a generation's inventory complete so a later checkpoint can adopt it.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::storage_scan_adoption

```rust
pub async fn storage_scan_adoption(&self, scan_id: &str) -> Result<Option<(String, bool)>>
```

Returns `(instance_id, completed)` when the generation row still exists.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::storage_scan_touch

```rust
pub async fn storage_scan_touch(&self, scan_id: &str) -> Result<()>
```

Refreshes the generation heartbeat after a page is stored.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::storage_scan_generation_count

```rust
pub async fn storage_scan_generation_count(&self) -> Result<u64>
```

How many generation rows exist. Tests use this to bound retained scans.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the count fails.

#### LibraryStore::storage_scan_reclaim_abandoned

```rust
pub async fn storage_scan_reclaim_abandoned(&self, stale_before: &str) -> Result<u64>
```

Deletes generations that are not live resumable work.

A generation with an active job is kept even when `updated_at` is older
than `stale_before`. A jobless generation is kept only while its
heartbeat is at least `stale_before`. Terminal, missing, and stale
jobless generations are removed with their inventory rows.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or delete fails.

#### LibraryStore::storage_scan_reclaim_if_terminal

```rust
pub async fn storage_scan_reclaim_if_terminal(&self, job_id: &str) -> Result<u64>
```

Deletes this job's inventories when the job is terminal or already gone.

Pending and running jobs, including a retry after one failed attempt,
are left untouched.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or delete fails.

#### LibraryStore::storage_scan_reclaim_terminal_page

```rust
pub async fn storage_scan_reclaim_terminal_page(
    &self,
    after_scan_id: Option<&str>,
    limit: u64,
) -> Result<(u64, Option<String>)>
```

Deletes up to `limit` generations whose job is terminal or missing.

Pending, running, and jobless generations are kept. `after_scan_id`
walks past a page of still-active rows so one sweep cannot stall on
them. Pass the returned cursor back; `None` means the walk wrapped.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or delete fails. A delete
error leaves that generation for a later retry.

#### LibraryStore::list_books_page

```rust
pub async fn list_books_page(
    &self,
    account_id: Option<&str>,
    after_id: Option<i64>,
    limit: u64,
) -> Result<Vec<BookRecord>>
```

Pages books by surrogate id so a scan does not load the catalog at once.

`limit` is clamped to 1..=64. The read starts at that width and halves
when the guest result would exceed `maxResultBytes` (256 KiB). A full
page of enriched `books` rows is larger than that cap. The smaller
width that fits is reused until `limit` rows are collected or the
catalog ends.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than the result cap, or when a single row still exceeds the cap.

#### LibraryStore::record_title_file

```rust
pub async fn record_title_file(
    &self,
    destination: &str,
    key: &str,
    book_uuid: &str,
) -> Result<()>
```

Records that the title `book_uuid` writes `key` on `destination`
(an acquire run on this node, just before the write).

The record is replaced unconditionally: the file is this title's
once written, whichever title held the key before.

##### Errors

Returns an error when the write fails.

#### LibraryStore::claim_title_file

```rust
pub async fn claim_title_file(
    &self,
    destination: &str,
    key: &str,
    book_uuid: &str,
    writer: &str,
    expected: Option<&TitleFile>,
    branch_titles_hold: bool,
) -> Result<bool>
```

Claims `key` on `destination` for the title `book_uuid`, with
`writer` (a host upload id) as its latest writer.

The claim succeeds when the key is unrecorded, already the title's,
or recorded for a title that holds no key against this claim;
answers false, changing nothing, when another library title holds
it. A book holds its keys against every claim. A title a branch
shares here (`branch_titles`) holds its keys only against another
branch title's claim (`branch_titles_hold`, a catalog upload): a
Host job's upload of a book takes them over, as an acquire on this
node would. Each claim is one conditional write, so of two uploads
of different titles racing for one key only one wins.

`expected` is the record the caller read before ([`Self::title_file`](#librarystoretitle_file);
`None` when unrecorded), which a refused claim's release restores: the
claim also answers false, changing nothing, when the record is no
longer that one (an acquire recorded the key in between), so a
release never removes a record the claim did not replace.

##### Errors

Returns an error when a write fails.

#### LibraryStore::release_title_file

```rust
pub async fn release_title_file(
    &self,
    destination: &str,
    key: &str,
    writer: &str,
    prior: Option<&TitleFile>,
) -> Result<()>
```

Gives back a claim `writer` made on `key` on `destination`: the
record returns to `prior` (who held it before the claim), or is
removed when `prior` is `None` (the claim created it). Changes
nothing once someone else has written the key since (its writer is
no longer `writer`), so a release never takes a key from a later
claim or acquire.

##### Errors

Returns an error when the write fails.

#### LibraryStore::count_title_files_of

```rust
pub async fn count_title_files_of(&self, book_uuid: &str, writer: &str) -> Result<u64>
```

How many keys `writer` (a host upload id) holds for the title
`book_uuid`, on every destination together.

##### Errors

Returns an error when the read fails.

#### LibraryStore::title_file

```rust
pub async fn title_file(&self, destination: &str, key: &str) -> Result<Option<TitleFile>>
```

Who holds `key` on `destination`; `None` when it is unrecorded.

##### Errors

Returns an error when the read fails.

#### LibraryStore::enqueue_job

```rust
pub async fn enqueue_job(&self, spec: EnqueueJobSpec) -> Result<EnqueueOutcome>
```

Admit a job in one transaction, coalescing onto an active dedup key.

##### Arguments

* `spec` - Kind, payload, priority, attempt cap, and pending cap.

##### Returns

[`EnqueueOutcome::Created`](#enqueueoutcome), [`EnqueueOutcome::Duplicate`](#enqueueoutcome), or
[`EnqueueOutcome::QueueFull`](#enqueueoutcome).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the database write fails.

#### LibraryStore::claim_next_job

```rust
pub async fn claim_next_job(
    &self,
    resource_class: JobResourceClass,
    owner: &str,
    lease_secs: u64,
    operation_id: &str,
) -> Result<Option<JobRecord>>
```

Claim the next ready job with a conditional `pending` → `running` mutation.

`operation_id` is the atomic / local-txn idempotency key: retrying a
lost response with the same id must not claim a different row.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the claim write fails.

#### LibraryStore::list_job_claim_candidates

```rust
pub async fn list_job_claim_candidates(
    &self,
    resource_class: JobResourceClass,
    after: Option<&JobClaimCursor>,
    limit: u64,
) -> Result<JobClaimPage>
```

One page of ready pending jobs of `resource_class`, best first, for a
routed claim.

Pages are keyset-ordered by `priority DESC, created_at ASC, id ASC`:
pass `None` for the first page and the previous page's
[`JobClaimPage::next`](#jobclaimpage) for the following one, until it is `None`. A
caller that skips every job of a page (for example jobs another node
must run) can therefore still reach the runnable rows behind them.

Each returned row is readable and not cancel-requested. On the way,
pending rows with an unknown class or an unreadable command become
`invalid_job`, and cancel-requested pending rows become `cancelled`,
exactly as [`Self::claim_next_job`](#librarystoreclaim_next_job) does. Every write is a CAS on
`state = 'pending'`, so a concurrent claimer is never overwritten.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or write fails.

#### LibraryStore::claim_job_by_id

```rust
pub async fn claim_job_by_id(
    &self,
    job_id: &str,
    expected_generation: i64,
    owner: &str,
    lease_secs: u64,
) -> Result<Option<JobRecord>>
```

Claims the pending job `job_id` for `owner` with a single CAS, but
only while its lease generation is still `expected_generation`.

The update matches `state = 'pending'` and `expected_generation`, the
lease generation of the record the caller routed on, so of several
concurrent claimers on any number of connections exactly one moves the
row to `running` (and bumps the generation that fences every later
write). A successful claim therefore always runs at
`expected_generation + 1`, which lets the caller name the claim's
generation (in its permit holders, say) before claiming. A row whose
payload was rewritten after the caller read it
([`Self::rewrite_pending_job_payload`](#librarystorerewrite_pending_job_payload) bumps the generation) is not
claimed, so a job never runs on requirements its claimer did not
check. No transaction is needed: a pending → running move does not
change the admission count.

##### Arguments

* `job_id` - The job to claim.
* `expected_generation` - `lease_generation` of the record the caller
  evaluated; a successful claim runs at `expected_generation + 1`.
* `owner` - Worker lease owner written on the row.
* `lease_secs` - Lease length.

##### Returns

The claimed record, or `None` when the row is missing, not pending,
not yet due, cancel-requested, unreadable, moved past
`expected_generation`, or another claimer won.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or write fails.

#### LibraryStore::list_pending_jobs_page

```rust
pub async fn list_pending_jobs_page(
    &self,
    after_id: Option<&str>,
    limit: u64,
) -> Result<(Vec<JobRecord>, Option<String>)>
```

One page of pending jobs in id order, for maintenance passes.

Pass `None` for the first page and the returned cursor for the next,
until it is `None`. Unlike the claim pages this ignores `run_after`
and the resource class; unreadable rows are skipped (the claim paths
mark them `invalid_job`).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::rewrite_pending_job_payload

```rust
pub async fn rewrite_pending_job_payload(
    &self,
    job_id: &str,
    lease_generation: i64,
    payload: &JobPayload,
) -> Result<bool>
```

Replaces the payload of a pending job read at `lease_generation`.

A compare-and-swap on `state = 'pending'` and the generation, which it
also bumps: a claimer that read the row before this write loses its
own CAS (and re-reads), so a concurrent claim is never overwritten and
never runs on the replaced payload.

##### Returns

`true` when this write won; `false` when the row moved on.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails, or
[`LibraryError::Other`](#libraryerror) when the payload cannot be serialized.

#### LibraryStore::fail_pending_job

```rust
pub async fn fail_pending_job(
    &self,
    job_id: &str,
    lease_generation: i64,
    error_kind: &str,
    error_message: &str,
) -> Result<bool>
```

Fails a pending job read at `lease_generation`, terminally.

A compare-and-swap on `state = 'pending'` and the generation, so a job
a worker claimed meanwhile is left alone. The row leaves the pending
count and frees its dedup key.

##### Returns

`true` when this write won; `false` when the row moved on.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::release_job_claim

```rust
pub async fn release_job_claim(&self, fence: &JobFence, resumed: bool) -> Result<bool>
```

Returns a job claimed under `fence` to `pending` without spending an
attempt, as if the claim had not happened.

Used when the claim is refused at handoff (the node is no longer
trusted with the job). `resumed` says the claim resumed a suspended
job: its attempt count was not raised, and the one-shot resume flag
is set again. Otherwise the count the claim added is taken back. The
lease generation stays bumped, so the refused claimer's fence can
never write to the row again.

##### Returns

`true` when the row was released; `false` when `fence` no longer owns
the running generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or write fails, or
[`LibraryError::Other`](#libraryerror) when the stored payload is not valid JSON.

#### LibraryStore::heartbeat_job

```rust
pub async fn heartbeat_job(
    &self,
    fence: &JobFence,
    lease_secs: u64,
    progress: Option<&str>,
) -> Result<bool>
```

Refresh the lease when `fence` still owns the running generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::set_job_progress

```rust
pub async fn set_job_progress(&self, fence: &JobFence, progress: &str) -> Result<bool>
```

Persist progress when `fence` still owns the running generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::complete_job

```rust
pub async fn complete_job(&self, fence: &JobFence, progress: Option<&str>) -> Result<bool>
```

Mark a running job succeeded when `fence` still owns the generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::suspend_job

```rust
pub async fn suspend_job(
    &self,
    fence: &JobFence,
    checkpoint: &JobCheckpoint,
    wake_at: DateTime<Utc>,
) -> Result<bool>
```

Park a running job as `pending` with a checkpoint, to be claimed again after `wake_at`.

The CAS matches [`Self::complete_job`]: `state=running` and the caller's fence. On success the
lease is cleared so another replica can claim the next attempt. `payload.checkpoint` is
replaced with `checkpoint` (other payload fields are preserved).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read or update fails, or [`LibraryError::Other`](#libraryerror)
when the stored payload is not valid JSON.

#### LibraryStore::checkpoint_running_job

```rust
pub async fn checkpoint_running_job(
    &self,
    fence: &JobFence,
    checkpoint: &JobCheckpoint,
) -> Result<bool>
```

Replaces `payload.checkpoint` while `fence` still owns the running job.

Unlike [`Self::suspend_job`](#librarystoresuspend_job), the row stays `running` and `resume_pending`
is left unchanged. A stale fence updates zero rows. Reclaim keeps the
payload, so the next claim observes the checkpoint.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) on database failure, or [`LibraryError::Other`](#libraryerror)
when the checkpoint exceeds [`bookclerk_plugin_abi::MAX_CHECKPOINT_BYTES`](../bookclerk_plugin_abi/index.md#max_checkpoint_bytes).

#### LibraryStore::fail_job

```rust
pub async fn fail_job(
    &self,
    fence: &JobFence,
    error_kind: &str,
    error_message: &str,
) -> Result<bool>
```

Fail a running job when `fence` still owns the generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::request_job_cancel

```rust
pub async fn request_job_cancel(&self, id: &str) -> Result<Option<JobRecord>>
```

Cancel a pending job immediately, or flag a running job for cooperative stop.

Pending and running are handled atomically against claim: a CAS miss
(the worker claimed, or the job left `running`) retries until the row
is terminal, cancelled, or flagged `cancel_requested`.

##### Returns

The updated record, or `None` when the id is unknown.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::job_cancel_requested

```rust
pub async fn job_cancel_requested(&self, id: &str) -> Result<bool>
```

True when a running worker should abort after the current step.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::reclaim_expired_leases

```rust
pub async fn reclaim_expired_leases(&self) -> Result<u32>
```

Reclaim expired running leases with a conditional `running` update.

##### Returns

Number of rows moved out of `running`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::get_job

```rust
pub async fn get_job(&self, id: &str) -> Result<Option<JobRecord>>
```

Load one job by id, rejecting unreadable rows as `invalid_job`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_jobs

```rust
pub async fn list_jobs(&self, limit: u64) -> Result<Vec<JobRecord>>
```

List recent jobs, newest first, capped at `limit` (minimum 1).

A peer writer on the same SQLite file (TRUNCATE journal) can make this
read return `SQLITE_BUSY`. The read is repeated until the lock clears
or the attempt budget is spent. `limit` is unchanged.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::prune_terminal_jobs

```rust
pub async fn prune_terminal_jobs(&self, retention_days: u64) -> Result<u64>
```

Delete terminal jobs older than `retention_days`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::reserve_job_temp_path

```rust
pub async fn reserve_job_temp_path(
    &self,
    job_id: &str,
    path: &str,
    reserved_bytes: u64,
    quota_bytes: u64,
) -> Result<()>
```

Reserve `reserved_bytes` against the job temp quota and register `path`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails, or
[`LibraryError::Other`](#libraryerror) when the quota would be exceeded.

#### LibraryStore::register_job_temp_path

```rust
pub async fn register_job_temp_path(&self, job_id: &str, path: &str) -> Result<()>
```

Register a scratch path with a zero-byte reservation (legacy helper).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the insert fails.

#### LibraryStore::list_job_temp_paths

```rust
pub async fn list_job_temp_paths(&self, job_id: &str) -> Result<Vec<JobTempPath>>
```

List scratch paths registered for `job_id`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_all_job_temp_paths

```rust
pub async fn list_all_job_temp_paths(&self) -> Result<Vec<JobTempPath>>
```

List every registered scratch path (used by the startup sweeper).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::unregister_job_temp_path

```rust
pub async fn unregister_job_temp_path(&self, job_id: &str, path: &str) -> Result<()>
```

Drop one scratch-path row after that path is gone from disk.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::reserved_temp_bytes

```rust
pub async fn reserved_temp_bytes(&self) -> Result<u64>
```

Sum of reserved scratch bytes across all jobs.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::has_running_acquire_job

```rust
pub async fn has_running_acquire_job(&self) -> Result<bool>
```

True when any acquire job is currently running.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_orphaned_acquire_books

```rust
pub async fn list_orphaned_acquire_books(&self) -> Result<Vec<BookRecord>>
```

Books left in `queued` / `downloading` with no running acquire job.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::reconcile_orphaned_acquire_rows

```rust
pub async fn reconcile_orphaned_acquire_rows(&self) -> Result<u32>
```

Mark orphaned `queued` / `downloading` books as `error` after a crash.

##### Returns

Number of book rows updated.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a status write fails.

#### LibraryStore::count_active_jobs

```rust
pub async fn count_active_jobs(&self) -> Result<i64>
```

Count pending + running jobs (admission cap).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_books_filtered_page

```rust
pub async fn list_books_filtered_page(
    &self,
    account_id: Option<&str>,
    status: Option<&str>,
    limit: u64,
    offset: u64,
) -> Result<BookPage>
```

Pages books by account, acquire status, and title.

`status` is the wire string from [`crate::AcquireStatus::as_str`](#acquirestatusas_str).
`limit` is clamped to 1..=500. `total` uses the same `WHERE` as the page.

The page is read in pieces of at most 64 rows so a requested limit of
256 stays under the guest result-byte cap. A piece is halved when that
result would exceed the cap, and later pieces keep that smaller width.
`limit`, `offset`, and `total` are the caller's page, not one of those
pieces. `total` is counted once.

##### Errors

Returns [`LibraryError::Other`](#libraryerror) when the read fails.

#### LibraryStore::list_books_by_uuid_page

```rust
pub async fn list_books_by_uuid_page(
    &self,
    uuids: &[String],
    account_id: Option<&str>,
    status: Option<&str>,
    limit: u64,
    offset: u64,
) -> Result<BookPage>
```

Pages books whose uuid is in `uuids`, then applies account and status.

`uuids` is capped at [`BOOK_PAGE_MAX_LIMIT`](#book_page_max_limit) (the search-hit cap).
Lists longer than 64 uuids are ordered from narrow `uuid, title` reads,
then only the requested page is loaded. Shorter lists sort and page in
SQL, in pieces that shrink when a result would exceed the guest byte
cap. An empty uuid list is an empty page. Two stored uuids that differ
only by ASCII case are different books and both hydrate.

##### Errors

Returns [`LibraryError::Other`](#libraryerror) when the read fails.

#### LibraryStore::from_connection

```rust
pub fn from_connection(db: DatabaseConnection) -> Self
```

Wrap an already-opened (and migrated) SeaORM connection.

Prefer the database plugin host (`open_library_store`) or, in tests,
`bookclerk_plugin_database_sqlite::open_memory` — this crate does not
open engine-specific connections.

#### LibraryStore::with_atomic_txn

```rust
pub fn with_atomic_txn(self, backend: Arc<dyn AtomicTxnBackend>) -> Self
```

Attach a guest atomic backend for named security operations.

Hosts attach this for every database plugin. Tests that open a local
connection leave it unset and run the same commands through
[`crate::execute_db_atomic`](#execute_db_atomic).

#### LibraryStore::with_typed_exec

```rust
pub fn with_typed_exec(self, backend: Arc<dyn TypedAtomicExec>) -> Self
```

Attach a guest `executeAtomic` backend for granted job SQL.

Hosts attach this for every database plugin so job sessions do not use
the SeaORM proxy. Tests that open a local connection leave it unset.

#### LibraryStore::with_max_binds

```rust
pub fn with_max_binds(self, max_binds: u32) -> Self
```

Records the guest's negotiated bind cap for wake `IN (…)` chunking.

#### LibraryStore::with_plan_limits

```rust
pub fn with_plan_limits(
    self,
    max_statements: u32,
    max_result_rows: u32,
    max_payload_bytes: u32,
) -> Self
```

Records negotiated statement / result / payload caps from connect.

#### LibraryStore::with_db_capabilities

```rust
pub fn with_db_capabilities(self, caps: DbCapabilities) -> Self
```

Records the full negotiated [`bookclerk_plugin_abi::DbCapabilities`](../bookclerk_plugin_abi/db_execute.md#dbcapabilities).

#### LibraryStore::db_capabilities

```rust
pub fn db_capabilities(&self) -> &DbCapabilities
```

Negotiated capability advertisement used for plan validation.

#### LibraryStore::execute_guest_atomic

```rust
pub async fn execute_guest_atomic(
    &self,
    req: ExecuteRequest,
    policy: &GuestSqlPolicy,
) -> Result<ExecuteReply, PluginError>
```

Runs guest-authored SQL after grammar, table-scope, and cap checks.

Hosts inject this into `JobRunner.job` as the granted
`bookclerk_plugin_abi::GuestDatabase::execute` path.
Job sessions pass a host-issued [`GuestSqlPolicy`](../bookclerk_plugin_abi/guest_sql.md#guestsqlpolicy)
(`books` by default).

##### Errors

Returns `bookclerk_plugin_abi::PluginError::invalid_params` when the
SQL is outside the guest grammar or allowlist, and an engine/transport
error when the batch fails.

#### LibraryStore::dispatch_chunk_size

```rust
pub fn dispatch_chunk_size(&self) -> usize
```

Subscribers packed into one dispatch plan (receipt overhead is ~12 statements).

#### LibraryStore::wake_page

```rust
pub fn wake_page(&self) -> u64
```

Wake delivery page size derived from negotiated `maxBinds`.

#### LibraryStore::connection

```rust
pub fn connection(&self) -> &DatabaseConnection
```

Borrow the underlying SeaORM connection (e.g. for [`crate::secrets`](secrets.md)).

#### LibraryStore::db

```rust
pub fn db(&self) -> &DatabaseConnection
```

Alias for [`Self::connection`](#librarystoreconnection) — convenience name used by credential modules.

#### LibraryStore::delete_elevated_operator_sessions_for_user

```rust
pub async fn delete_elevated_operator_sessions_for_user(
    &self,
    user_id: i64,
) -> Result<u64>
```

Revoke Operator sessions minted by this user's elevate-to-operator flow.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_account

```rust
pub async fn upsert_account(
    &self,
    plugin_key: &str,
    account_id: &str,
    marketplace: &str,
    label: Option<&str>,
    scan_enabled: bool,
    source: &str,
) -> Result<AccountRecord>
```

Upsert an account (updates `scan_enabled` on conflict).

Accounts are keyed by `(plugin_key, account_id)`: the same store id may
exist under two plugins. `source` is the owning plugin's display alias.
Content sources should prefer [`crate::SourceScope::upsert_account`](scope.md#sourcescopeupsert_account),
which forces both.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::ensure_account

```rust
pub async fn ensure_account(
    &self,
    plugin_key: &str,
    account_id: &str,
    marketplace: &str,
    label: Option<&str>,
    source: &str,
) -> Result<AccountRecord>
```

Ensure an account row exists without overwriting `scan_enabled`.

Content sources should prefer [`crate::SourceScope::ensure_account`](scope.md#sourcescopeensure_account).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::remap_account_id

```rust
pub async fn remap_account_id(&self, plugin_key: &str, from: &str, to: &str) -> Result<()>
```

Remap `from` → `to` account ids within one plugin (books + account row).

Used when classic Libation stored an email `AccountId` and a later
login/scan discovers Audible `customer_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::reconcile_account_id

```rust
pub async fn reconcile_account_id(
    &self,
    plugin_key: &str,
    canonical_id: &str,
    aliases: &[&str],
    marketplace: &str,
    label: Option<&str>,
    scan_enabled: bool,
    source: &str,
) -> Result<AccountRecord>
```

Remap any existing alias account ids onto `canonical_id`, then upsert.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_account

```rust
pub async fn get_account(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<Option<AccountRecord>>
```

Loads one account by `(plugin_key, account_id)`, if present.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::find_accounts_by_id

```rust
pub async fn find_accounts_by_id(&self, account_id: &str) -> Result<Vec<AccountRecord>>
```

Every account row with this store id, across plugins.

##### Errors

Returns a crate error when the database operation fails.

#### LibraryStore::get_account_unique

```rust
pub async fn get_account_unique(&self, account_id: &str) -> Result<Option<AccountRecord>>
```

Resolves one account by store id, failing closed when two plugins own
that id.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when the id is ambiguous.

#### LibraryStore::list_accounts

```rust
pub async fn list_accounts(&self) -> Result<Vec<AccountRecord>>
```

Lists all store accounts in the library database.


##### Returns

`Result<Vec<AccountRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::count_identities

```rust
pub async fn count_identities(&self) -> Result<i64>
```

Count library identity rows (SQL `COUNT`, not a full fetch).

Same cardinality as [`Self::count_accounts`](#librarystorecount_accounts). Prefer this when the caller
only needs a number for an operator-facing summary.

A peer writer on the same SQLite file (TRUNCATE journal) can make this
read return `SQLITE_BUSY`. The read is repeated until the lock clears
or the attempt budget is spent.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::count_accounts

```rust
pub async fn count_accounts(&self) -> Result<i64>
```

Count account rows (SQL `COUNT`, not a full fetch).

Uses [`Self::count_identities`](#librarystorecount_identities), including its wait for a peer SQLite lock.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::find_accounts

```rust
pub async fn find_accounts(&self, identifier: &str) -> Result<Vec<AccountRecord>>
```

Every account whose id or nickname (`label`) matches, case-insensitive.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::find_account

```rust
pub async fn find_account(
    &self,
    identifier: &str,
    plugin_key: Option<&str>,
) -> Result<Option<AccountRecord>>
```

Resolve one account by id or nickname (`label`), case-insensitive.

`plugin_key` narrows the match to one plugin. Without it, a needle that
matches accounts of two plugins fails closed.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when the needle is ambiguous.

#### LibraryStore::set_scan_enabled

```rust
pub async fn set_scan_enabled(
    &self,
    plugin_key: &str,
    account_id: &str,
    scan_enabled: bool,
) -> Result<()>
```

Toggle whether an account is included in automatic library scans.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::mark_connection_active

```rust
pub async fn mark_connection_active(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<()>
```

Mark bookstore credentials active again (after reconnect).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_account_secrets

```rust
pub async fn delete_account_secrets(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<()>
```

Delete all `encrypted_secrets` rows for an account (source auth + Widevine CDM).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::revoke_credentials

```rust
pub async fn revoke_credentials(&self, plugin_key: &str, account_id: &str) -> Result<()>
```

Mark bookstore credentials revoked without deleting the account or books.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_portal_identity

```rust
pub async fn upsert_portal_identity(
    &self,
    provider: &str,
    external_user_id: &str,
    label: Option<&str>,
) -> Result<PortalIdentity>
```

Create or fetch a portal identity for `(provider, external_user_id)`.

Ensures a first-party [`UserRecord`](#userrecord) (role `member`) is linked via
`portal_identities.user_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::create_user

```rust
pub async fn create_user(
    &self,
    role: UserRole,
    display_name: Option<&str>,
    password_hash: Option<&str>,
) -> Result<UserRecord>
```

Create a first-party user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::create_user_with_login

```rust
pub async fn create_user_with_login(
    &self,
    role: UserRole,
    display_name: Option<&str>,
    login_name: Option<&str>,
    password_hash: Option<&str>,
) -> Result<UserRecord>
```

Create a first-party user with optional local login_name and email.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::create_user_with_profile

```rust
pub async fn create_user_with_profile(
    &self,
    role: UserRole,
    display_name: Option<&str>,
    login_name: Option<&str>,
    email: Option<&str>,
    password_hash: Option<&str>,
) -> Result<UserRecord>
```

Create a first-party user with login, email, and optional password hash.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_user

```rust
pub async fn get_user(&self, id: i64) -> Result<Option<UserRecord>>
```

Look up a first-party user by id.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_user_by_login_name

```rust
pub async fn get_user_by_login_name(&self, login_name: &str) -> Result<Option<UserRecord>>
```

Look up by local login_name (case-insensitive).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_user_by_email

```rust
pub async fn get_user_by_email(&self, email: &str) -> Result<Option<UserRecord>>
```

Look up by email (case-insensitive).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_user_password_hash

```rust
pub async fn get_user_password_hash(&self, id: i64) -> Result<Option<String>>
```

Raw password hash for verification (never expose via API).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_users

```rust
pub async fn list_users(&self) -> Result<Vec<UserRecord>>
```

List all first-party users (admin tooling).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_user_presence_extras

```rust
pub async fn list_user_presence_extras(
    &self,
    listening_within: Duration,
) -> Result<HashMap<i64, UserPresenceExtras>>
```

Presence, listening, and storefront links for administrator user lists.

`online` means any non-expired portal session. `listening` is the newest
unfinished progress row with `last_listened_at` within
`listening_within` (typical UI window: ~30 minutes). Used by
`GET /api/users`. Session activity (`last_active_at`) comes from
unexpired `portal_sessions.last_used_at`. Durable last-seen lives on
`users.last_seen_at` (survives logout) and is refreshed with session
use at most once per minute.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_user

```rust
pub async fn delete_user(&self, id: i64) -> Result<()>
```

Delete a first-party user and personal data.

Removes portal identities, account links, wishlist rows, claim tickets,
sessions, and prefs for the user. Acquired library books are retained.
Refuses to delete the last active owner.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_status

```rust
pub async fn set_user_status(&self, id: i64, status: UserStatus) -> Result<UserRecord>
```

Set user status (`active` / `disabled`).

Refuses to disable the last active owner.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_display_name

```rust
pub async fn set_user_display_name(
    &self,
    id: i64,
    display_name: Option<&str>,
) -> Result<UserRecord>
```

Set display name (does not bump security_version).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_password_hash

```rust
pub async fn set_user_password_hash(
    &self,
    id: i64,
    password_hash: Option<&str>,
) -> Result<UserRecord>
```

Set or clear Argon2id password hash; bumps `security_version`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_login_name

```rust
pub async fn set_user_login_name(
    &self,
    id: i64,
    login_name: Option<&str>,
) -> Result<UserRecord>
```

Set login_name (unique); does not bump security_version.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_email

```rust
pub async fn set_user_email(&self, id: i64, email: Option<&str>) -> Result<UserRecord>
```

Set contact email (unique when set); does not bump security_version.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_avatar_source

```rust
pub async fn set_user_avatar_source(
    &self,
    id: i64,
    source: Option<&str>,
) -> Result<UserRecord>
```

Set the user's picture source (`monogram` / `gravatar` / `upload` / `sso:{id}` / auto).

`None` or empty stores auto-resolve (upload, then last-used SSO, then Gravatar).

##### Errors

Returns an error when the user is missing or the write fails.

#### LibraryStore::set_portal_identity_picture

```rust
pub async fn set_portal_identity_picture(
    &self,
    provider: &str,
    external_user_id: &str,
    picture_url: Option<&str>,
) -> Result<()>
```

Store or clear the HTTPS picture URL on a portal identity.

Missing identities are ignored so callers can run this after a failed
lookup without an extra round-trip.

##### Errors

Returns an error when the write fails.

#### LibraryStore::list_user_sso_pictures

```rust
pub async fn list_user_sso_pictures(&self, user_id: i64) -> Result<Vec<UserSsoPicture>>
```

IdP pictures linked to `user_id`, with last portal-session activity.

Identities without a `picture_url` are omitted. `last_used_at` is the
latest session `last_used_at` or `created_at` for that identity.

##### Errors

Returns an error when the query fails.

#### LibraryStore::delete_portal_sessions_for_user

```rust
pub async fn delete_portal_sessions_for_user(&self, user_id: i64) -> Result<u64>
```

Delete all portal sessions for identities linked to `user_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_user_invite

```rust
pub async fn insert_user_invite(
    &self,
    token_hash: &str,
    role: UserRole,
    login_name: Option<&str>,
    display_name: Option<&str>,
    expires_at: DateTime<Utc>,
    created_by: &str,
) -> Result<UserInviteRecord>
```

Insert a user invite (store hash only).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::redeem_user_invite

```rust
pub async fn redeem_user_invite(&self, token_hash: &str) -> Result<UserInviteRecord>
```

Atomically redeem a user invite.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_role

```rust
pub async fn set_user_role(&self, id: i64, role: UserRole) -> Result<UserRecord>
```

Set user role (`owner` / `administrator` / `member`).

Refuses to demote the last active owner.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_administrators

```rust
pub async fn count_administrators(&self) -> Result<u64>
```

Count administrators (includes disabled).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_active_administrators

```rust
pub async fn count_active_administrators(&self) -> Result<u64>
```

Count administrators with status `active`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_owners

```rust
pub async fn count_owners(&self) -> Result<u64>
```

Count owners (includes disabled) — bootstrap / inventory.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_active_owners

```rust
pub async fn count_active_owners(&self) -> Result<u64>
```

Count owners with status `active` (last-owner demote/disable/delete guard).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::ensure_users_bridged

```rust
pub async fn ensure_users_bridged(&self) -> Result<usize>
```

Backfill `portal_identities.user_id` and remap prefs `portal:{id}` → `user:{id}`.

Safe to call repeatedly after migrations. Does not link CLI-created store
accounts that have no `account_links` (avoids widening access).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::ensure_local_portal_identity

```rust
pub async fn ensure_local_portal_identity(
    &self,
    user_id: i64,
    label: Option<&str>,
) -> Result<PortalIdentity>
```

Ensure a `local` portal identity exists for a first-party user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_portal_identity

```rust
pub async fn get_portal_identity(
    &self,
    provider: &str,
    external_user_id: &str,
) -> Result<Option<PortalIdentity>>
```

Loads a portal identity by primary key, if present.

##### Arguments

* `provider` - External provider id.
* `external_user_id` - User id at the external provider.

##### Returns

`Result<Option<crate::models::PortalIdentity>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_portal_identity_by_id

```rust
pub async fn get_portal_identity_by_id(&self, id: i64) -> Result<Option<PortalIdentity>>
```

Look up portal identity by row id.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_claim_ticket

```rust
pub async fn insert_claim_ticket(
    &self,
    token_hash: &str,
    identity_id: Option<i64>,
    expires_at: DateTime<Utc>,
    created_by: &str,
) -> Result<ClaimTicketRecord>
```

Insert a claim ticket (store only the hash).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_claim_ticket_by_hash

```rust
pub async fn get_claim_ticket_by_hash(
    &self,
    token_hash: &str,
) -> Result<Option<ClaimTicketRecord>>
```

Loads a claim ticket row by its token hash, if present.

##### Arguments

* `token_hash` - SHA-256 hex digest of the opaque token.

##### Returns

`Result<Option<crate::models::ClaimTicketRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::list_open_claim_tickets

```rust
pub async fn list_open_claim_tickets(&self) -> Result<Vec<ClaimTicketRecord>>
```

List unredeemed, unexpired claim tickets (newest first).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::redeem_claim_ticket

```rust
pub async fn redeem_claim_ticket(&self, token_hash: &str) -> Result<ClaimTicketRecord>
```

Atomically mark a claim ticket redeemed.

Uses a single `UPDATE … WHERE redeemed_at IS NULL AND expires_at > now` so
concurrent redeemers cannot both succeed.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::redeem_claim_ticket_to_session

```rust
pub async fn redeem_claim_ticket_to_session(
    &self,
    token_hash: &str,
    session_hash: &str,
    expires_at: DateTime<Utc>,
    client: Option<&SessionClientInfo>,
    new_password_hash: Option<&str>,
    password_fingerprint: Option<&str>,
) -> Result<PortalIdentity>
```

Consume a claim ticket, optionally set an unset local password, and mint
a portal session in one transaction.

`new_password_hash` is assigned only while the ticket-bound local user
still has no password. A failed consume rolls back password and session
writes. Missing password for an unset local user aborts without consuming
the ticket.

##### Arguments

* `token_hash` - SHA-256 hex digest of the claim ticket.
* `session_hash` - SHA-256 hex digest of the new portal session token.
* `expires_at` - RFC 3339 expiry for the minted session.
* `client` - Optional client metadata stored on the session row.
* `new_password_hash` - Argon2id hash to set when the local user has none.
* `password_fingerprint` - Stable HMAC of the plaintext password for
  atomic-execute idempotency; ignored by the mutation SQL.

##### Returns

The portal identity bound to the consumed ticket.

##### Errors

Returns a crate error when the ticket is invalid/consumed, a required
invite password is missing, or the database operation fails.

#### LibraryStore::insert_portal_session

```rust
pub async fn insert_portal_session(
    &self,
    token_hash: &str,
    identity_id: i64,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Create a portal session (hash only).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_portal_session_with_client

```rust
pub async fn insert_portal_session_with_client(
    &self,
    token_hash: &str,
    identity_id: i64,
    expires_at: DateTime<Utc>,
    client: Option<&SessionClientInfo>,
) -> Result<()>
```

Create a portal session with optional client metadata.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_portal_session

```rust
pub async fn delete_portal_session(&self, token_hash: &str) -> Result<bool>
```

Delete a portal session by token hash (logout / revoke).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_portal_sessions_for_identity

```rust
pub async fn list_portal_sessions_for_identity(
    &self,
    identity_id: i64,
) -> Result<Vec<(i64, String, String)>>
```

List portal sessions for an identity (id, created_at, expires_at).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_portal_session_records_for_identity

```rust
pub async fn list_portal_session_records_for_identity(
    &self,
    identity_id: i64,
) -> Result<Vec<PortalSessionRecord>>
```

List portal sessions with client metadata for session management UIs.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_portal_session_by_id_for_identity

```rust
pub async fn delete_portal_session_by_id_for_identity(
    &self,
    id: i64,
    identity_id: i64,
) -> Result<bool>
```

Delete a portal session by id only if it belongs to `identity_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_account_links_for_account

```rust
pub async fn count_account_links_for_account(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<u64>
```

Count account_links referencing `account_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_portal_session_identity

```rust
pub async fn get_portal_session_identity(
    &self,
    token_hash: &str,
) -> Result<Option<PortalIdentity>>
```

Resolve a valid portal session to its identity.

Refreshes `portal_sessions.last_used_at` and `users.last_seen_at` at most
once per minute so administrator presence can tell active from idle
without a write on every request. `last_seen_at` is kept after logout.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_operator_session

```rust
pub async fn insert_operator_session(
    &self,
    token_hash: &str,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Create a durable operator session (hash only).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_operator_session_with_client

```rust
pub async fn insert_operator_session_with_client(
    &self,
    token_hash: &str,
    expires_at: DateTime<Utc>,
    client: Option<&SessionClientInfo>,
) -> Result<()>
```

Create a durable operator session with optional client metadata.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_elevated_operator_session

```rust
pub async fn insert_elevated_operator_session(
    &self,
    token_hash: &str,
    expires_at: DateTime<Utc>,
    elevated_from_user_id: i64,
) -> Result<()>
```

Create an elevated operator session (Administrator reauth → short TTL).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_elevated_operator_session_with_client

```rust
pub async fn insert_elevated_operator_session_with_client(
    &self,
    token_hash: &str,
    expires_at: DateTime<Utc>,
    elevated_from_user_id: i64,
    client: Option<&SessionClientInfo>,
) -> Result<()>
```

Create an elevated operator session with optional client metadata.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_operator_session

```rust
pub async fn get_operator_session(
    &self,
    token_hash: &str,
) -> Result<Option<OperatorSessionRecord>>
```

Look up a valid operator session (touches `last_used_at`).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::operator_session_valid

```rust
pub async fn operator_session_valid(&self, token_hash: &str) -> Result<bool>
```

Return whether a hashed operator session is still valid (and touch last_used).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_operator_session_impersonating

```rust
pub async fn set_operator_session_impersonating(
    &self,
    token_hash: &str,
    user_id: Option<i64>,
) -> Result<OperatorSessionRecord>
```

Set or clear impersonation on an operator session.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_operator_session

```rust
pub async fn delete_operator_session(&self, token_hash: &str) -> Result<bool>
```

Delete an operator session by token hash (logout / revoke).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_operator_session_by_id

```rust
pub async fn delete_operator_session_by_id(&self, id: i64) -> Result<bool>
```

Delete an operator session by row id.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::prune_expired_operator_sessions

```rust
pub async fn prune_expired_operator_sessions(&self) -> Result<u64>
```

Delete expired operator sessions (bounded cleanup).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_operator_sessions

```rust
pub async fn list_operator_sessions(&self) -> Result<Vec<OperatorSessionRecord>>
```

List active operator sessions (newest first) for remote sign-out UI.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::first_portal_identity_for_user

```rust
pub async fn first_portal_identity_for_user(
    &self,
    user_id: i64,
) -> Result<Option<PortalIdentity>>
```

First portal identity linked to a first-party user (for impersonation scoping).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_portal_identities_for_user

```rust
pub async fn list_portal_identities_for_user(
    &self,
    user_id: i64,
) -> Result<Vec<PortalIdentity>>
```

All portal identities linked to a first-party user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::link_portal_identity_to_user

```rust
pub async fn link_portal_identity_to_user(
    &self,
    provider: &str,
    external_user_id: &str,
    user_id: i64,
    label: Option<&str>,
) -> Result<PortalIdentity>
```

Link `(provider, external_user_id)` to an existing user (SSO / invite_only).

Creates the identity row when missing. Refuses to steal an identity already
bound to a different user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_oidc_rp_state

```rust
pub async fn insert_oidc_rp_state(
    &self,
    state_hash: &str,
    provider_id: &str,
    pkce_verifier: &str,
    nonce: &str,
    purpose: &str,
    user_id: Option<i64>,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Insert a short-lived OIDC RP authorize state row.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::take_oidc_rp_state

```rust
pub async fn take_oidc_rp_state(
    &self,
    state_hash: &str,
) -> Result<Option<(String, String, String, String, Option<i64>)>>
```

Consume (delete) an OIDC RP state by hash if it has not expired.

Find + delete run in one transaction so concurrent callbacks cannot both
observe the same unused state.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_webauthn_credential

```rust
pub async fn insert_webauthn_credential(
    &self,
    user_id: i64,
    credential_id: &str,
    passkey_json: &str,
    name: Option<&str>,
) -> Result<()>
```

Store a WebAuthn credential for a user.

##### Arguments

* `user_id` - First-party user who owns the passkey.
* `credential_id` - Base64url credential id.
* `passkey_json` - Serialized `webauthn_rs` passkey.
* `name` - Label shown in Settings; empty becomes `None`.

##### Errors

Returns an error when the insert fails (including unique credential id).

#### LibraryStore::list_webauthn_credentials

```rust
pub async fn list_webauthn_credentials(&self, user_id: i64) -> Result<Vec<StoredPasskey>>
```

Passkeys registered to a user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_webauthn_credentials

```rust
pub async fn count_webauthn_credentials(&self, user_id: i64) -> Result<u64>
```

Count passkeys for a user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_webauthn_credential_by_cred_id

```rust
pub async fn get_webauthn_credential_by_cred_id(
    &self,
    credential_id: &str,
) -> Result<Option<(i64, i64, String)>>
```

Look up a passkey by credential id.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::update_webauthn_credential

```rust
pub async fn update_webauthn_credential(&self, id: i64, passkey_json: &str) -> Result<()>
```

Replace passkey JSON after a successful assertion (counter update).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_webauthn_credential

```rust
pub async fn delete_webauthn_credential(&self, user_id: i64, id: i64) -> Result<bool>
```

Delete a passkey owned by `user_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_totp_enabled

```rust
pub async fn set_user_totp_enabled(&self, user_id: i64, enabled: bool) -> Result<()>
```

Set whether this user has a confirmed TOTP authenticator.

##### Arguments

* `user_id` - First-party user id.
* `enabled` - `true` after a successful enroll confirm; `false` on disable.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the user does not exist.

#### LibraryStore::confirm_totp_enrollment

```rust
pub async fn confirm_totp_enrollment(&self, user_id: i64, secret_b32: &str) -> Result<()>
```

Promote a pending TOTP secret to primary and set `totp_enabled` in one transaction.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the user does not exist.

#### LibraryStore::disable_user_totp

```rust
pub async fn disable_user_totp(&self, user_id: i64) -> Result<()>
```

Delete TOTP secrets and clear `totp_enabled` in one transaction.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the user does not exist.

#### LibraryStore::insert_webauthn_challenge

```rust
pub async fn insert_webauthn_challenge(
    &self,
    challenge_id: &str,
    user_id: Option<i64>,
    kind: &str,
    state_json: &str,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Insert a WebAuthn ceremony state.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::take_webauthn_challenge

```rust
pub async fn take_webauthn_challenge(
    &self,
    challenge_id: &str,
    kind: &str,
) -> Result<Option<(Option<i64>, String)>>
```

Consume a WebAuthn ceremony by id (transactional delete-and-return).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::prune_expired_oidc_rp_states

```rust
pub async fn prune_expired_oidc_rp_states(&self) -> Result<u64>
```

Drop expired OIDC RP states (bounded cleanup).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::prune_expired_webauthn_challenges

```rust
pub async fn prune_expired_webauthn_challenges(&self) -> Result<u64>
```

Drop expired WebAuthn challenges (bounded cleanup).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_oidc_rp_states

```rust
pub async fn count_oidc_rp_states(&self) -> Result<u64>
```

Count pending OIDC RP states (for public-begin caps).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_webauthn_challenges

```rust
pub async fn count_webauthn_challenges(&self) -> Result<u64>
```

Count pending WebAuthn challenges (for public-begin caps).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::portal_session_created_at

```rust
pub async fn portal_session_created_at(
    &self,
    token_hash: &str,
) -> Result<Option<DateTime<Utc>>>
```

Created-at of a still-valid portal session (recent-auth checks).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_portal_session_created_at

```rust
pub async fn set_portal_session_created_at(
    &self,
    token_hash: &str,
    created_at: DateTime<Utc>,
) -> Result<bool>
```

Overwrite `created_at` for a portal session (tests / recent-auth fixtures).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_security_audit_event

```rust
pub async fn insert_security_audit_event(
    &self,
    actor: &str,
    action: &str,
    detail_json: Option<&str>,
) -> Result<SecurityAuditEvent>
```

Appends one security audit event row.

##### Arguments

* `actor` - Actor label.
* `action` - Audit action verb.
* `detail_json` - `detail_json` argument for this call.

##### Returns

`Result<crate::models::SecurityAuditEvent>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::insert_oidc_client

```rust
pub async fn insert_oidc_client(
    &self,
    client_id: &str,
    client_secret_hash: Option<&str>,
    redirect_uris: &[String],
    name: Option<&str>,
    issue_refresh_token: bool,
    allowed_scopes: &[String],
    enabled: bool,
    plugin_id: Option<&str>,
) -> Result<OidcClientRecord>
```

Insert a new OIDC client (public clients may omit secret hash).

Custom (operator) clients pass `plugin_id = None` and typically
`enabled = true`. Plugin-owned templates pass the plugin id and start
disabled.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_plugin_oidc_client

```rust
pub async fn upsert_plugin_oidc_client(
    &self,
    client_id: &str,
    plugin_id: &str,
    name: &str,
    redirect_uris: &[String],
    issue_refresh_token: bool,
    allowed_scopes: &[String],
) -> Result<OidcClientRecord>
```

Insert a plugin-owned OIDC client when missing; refresh redirect URIs when present.

New rows start **disabled**. Existing rows owned by the **same** plugin keep
`enabled`, name, scopes, and secret. Collisions with custom (operator) clients
or a different plugin's `client_id` are hard errors — this never mutates
an unowned row. Alias-era rows (`plugin_id = "audiobookshelf"`) are rewritten
to the provenance-qualified PluginKey (`…#audiobookshelf`) on the next
heartbeat from that guest; a different PluginKey with the same alias
cannot take the client over.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when `client_id` is already owned by a
custom client or another plugin.

#### LibraryStore::update_oidc_client

```rust
pub async fn update_oidc_client(
    &self,
    client_id: &str,
    redirect_uris: &[String],
    name: Option<&str>,
    issue_refresh_token: bool,
    allowed_scopes: &[String],
    enabled: bool,
    secret_hash: Option<Option<String>>,
) -> Result<Option<OidcClientRecord>>
```

Update an existing OIDC client without changing the secret unless `secret_hash` is `Some`.

Pass `Some(None)` as `secret_hash` to clear the secret (public PKCE). Pass `None` to keep.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_oidc_client_secret

```rust
pub async fn set_oidc_client_secret(
    &self,
    client_id: &str,
    client_secret_hash: Option<&str>,
) -> Result<Option<OidcClientRecord>>
```

Replace the stored client-secret hash (or clear it).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_oidc_client

```rust
pub async fn delete_oidc_client(&self, client_id: &str) -> Result<bool>
```

Delete a registered OIDC client by `client_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_oidc_clients

```rust
pub async fn list_oidc_clients(&self) -> Result<Vec<OidcClientRecord>>
```

List registered OIDC clients, oldest first.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_oidc_client

```rust
pub async fn get_oidc_client(&self, client_id: &str) -> Result<Option<OidcClientRecord>>
```

Loads a registered OIDC client by `client_id`, if present.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::insert_oidc_auth_code

```rust
pub async fn insert_oidc_auth_code(
    &self,
    code_hash: &str,
    client_id: &str,
    user_id: i64,
    redirect_uri: &str,
    code_challenge: &str,
    code_challenge_method: &str,
    scope: &str,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Persists a hashed OIDC authorization code with PKCE metadata.

##### Arguments

* `code_hash` - SHA-256 hex digest of the authorization code.
* `client_id` - OIDC client_id.
* `user_id` - First-party user id.
* `redirect_uri` - OAuth redirect URI.
* `code_challenge` - PKCE code_challenge.
* `code_challenge_method` - PKCE method (`S256` or `plain`).
* `scope` - OAuth scope string.
* `expires_at` - Expiry timestamp.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::peek_oidc_auth_code

```rust
pub async fn peek_oidc_auth_code(
    &self,
    code_hash: &str,
) -> Result<Option<(String, i64, String, String, String, String)>>
```

Reads an unconsumed, unexpired auth code without consuming it; returns
(client_id, user_id, redirect_uri, challenge, method, scope). For a
caller that records what the code grants before it consumes the code,
so a failed write leaves the code usable for a retry.

##### Errors

Returns an error when the read fails.

#### LibraryStore::consume_oidc_auth_code

```rust
pub async fn consume_oidc_auth_code(
    &self,
    code_hash: &str,
) -> Result<Option<(String, i64, String, String, String, String)>>
```

Atomically consume an auth code; returns (client_id, user_id, redirect_uri, challenge, method, scope).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_oidc_refresh_token

```rust
pub async fn insert_oidc_refresh_token(
    &self,
    token_hash: &str,
    client_id: &str,
    user_id: i64,
    scope: &str,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Persists a hashed OIDC refresh token.

##### Arguments

* `token_hash` - SHA-256 hex digest of the opaque token.
* `client_id` - OIDC client_id.
* `user_id` - First-party user id.
* `scope` - OAuth scope string.
* `expires_at` - Expiry timestamp.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_oidc_refresh_token

```rust
pub async fn get_oidc_refresh_token(
    &self,
    token_hash: &str,
) -> Result<Option<(String, i64, String)>>
```

Loads a refresh token row by hash, if present.

##### Arguments

* `token_hash` - SHA-256 hex digest of the opaque token.

##### Returns

`Result<Option<(String, i64, String)>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::revoke_oidc_refresh_token

```rust
pub async fn revoke_oidc_refresh_token(&self, token_hash: &str) -> Result<bool>
```

Marks a refresh token revoked; returns whether a row was updated.

##### Arguments

* `token_hash` - SHA-256 hex digest of the opaque token.

##### Returns

`Result<bool>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::list_security_audit_events

```rust
pub async fn list_security_audit_events(
    &self,
    limit: u64,
) -> Result<Vec<SecurityAuditEvent>>
```

List recent security audit events (newest first).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::link_account

```rust
pub async fn link_account(
    &self,
    identity_id: i64,
    plugin_key: &str,
    account_id: &str,
    source: &str,
) -> Result<AccountLinkRecord>
```

Link a bookstore account to a portal identity.

Account links are exclusive: one `(plugin_key, account_id)` may belong
to only one identity (`idx_account_links_account_exclusive`).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_account_links

```rust
pub async fn list_account_links(&self, identity_id: i64) -> Result<Vec<AccountLinkRecord>>
```

Lists store accounts linked to a portal identity.

##### Arguments

* `identity_id` - Portal identity id.

##### Returns

`Result<Vec<crate::models::AccountLinkRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::unlink_account

```rust
pub async fn unlink_account(
    &self,
    identity_id: i64,
    plugin_key: &str,
    account_id: &str,
) -> Result<()>
```

Remove an account link row (does not delete the account).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_book

```rust
pub async fn upsert_book(&self, book: &NewBook) -> Result<BookRecord>
```

Upsert a book from a library sync.

Insert on first sight; on `(source, account_id, product_id)` conflict,
merge fields with Audible-precedence rules (a Libro rescan without an
ASIN must not wipe Audible enrichment). Expressed as load-then-merge over
the `books` entity so the precedence logic is explicit and portable.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_book_by_uuid

```rust
pub async fn get_book_by_uuid(&self, uuid: &str) -> Result<Option<BookRecord>>
```

Look up a book by its public `uuid`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_book

```rust
pub async fn get_book(
    &self,
    title_id: &str,
    account_id: &str,
) -> Result<Option<BookRecord>>
```

Look up a book by uuid, product_id, asin, or isbn for the given account.

Prefer exact uuid / product_id matches, then asin, then isbn. When two
plugins own the same account id and both match at the same priority,
the lookup fails closed; pass the plugin key through
[`Self::get_book_in`](#librarystoreget_book_in) instead.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when the match is ambiguous.

#### LibraryStore::get_book_in

```rust
pub async fn get_book_in(
    &self,
    plugin_key: Option<&str>,
    title_id: &str,
    account_id: &str,
) -> Result<Option<BookRecord>>
```

[`Self::get_book`](#librarystoreget_book) narrowed to one plugin key when `plugin_key` is set.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when the match is ambiguous.

#### LibraryStore::find_books_by_isbn

```rust
pub async fn find_books_by_isbn(&self, isbn: &str) -> Result<Vec<BookRecord>>
```

All ownership rows sharing an ISBN (cross-account / cross-store enrichment).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_books

```rust
pub async fn list_books(&self, account_id: Option<&str>) -> Result<Vec<BookRecord>>
```

Lists books, optionally filtered to one `account_id`.

##### Arguments

* `account_id` - Store account id.

##### Returns

`Result<Vec<BookRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::count_books

```rust
pub async fn count_books(&self, account_id: Option<&str>) -> Result<i64>
```

Count book rows (SQL `COUNT`, not a full fetch).

A peer writer on the same SQLite file (TRUNCATE journal) can make this
read return `SQLITE_BUSY`. The read is repeated until the lock clears
or the attempt budget is spent. `account_id` is unchanged.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::update_user_fields

```rust
pub async fn update_user_fields(
    &self,
    title_id: &str,
    account_id: &str,
    fields: &UserBookFields,
) -> Result<()>
```

Update user-defined fields (tags, ratings, finished) without touching scan metadata.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_pdf_status

```rust
pub async fn set_pdf_status(
    &self,
    title_id: &str,
    account_id: &str,
    status: AcquireStatus,
    pdf_storage_key: Option<&str>,
) -> Result<()>
```

Updates companion PDF acquire status and optional storage key.

##### Arguments

* `title_id` - Product id / ASIN / ISBN used as the title key.
* `account_id` - Store account id.
* `status` - Acquire or request status value.
* `pdf_storage_key` - Object-storage key for the companion PDF.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::is_ignored

```rust
pub async fn is_ignored(&self, title_id: &str, account_id: &str) -> Result<bool>
```

Returns whether (`title_id`, `account_id`) is on the ignore list.

##### Arguments

* `title_id` - Product id / ASIN / ISBN used as the title key.
* `account_id` - Store account id.

##### Returns

`Result<bool>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::set_ignored

```rust
pub async fn set_ignored(
    &self,
    title_id: &str,
    account_id: &str,
    ignored: bool,
    reason: Option<&str>,
) -> Result<()>
```

Adds or removes a title from the per-account ignore list.

##### Arguments

* `title_id` - Product id / ASIN / ISBN used as the title key.
* `account_id` - Store account id.
* `ignored` - When true, add to the ignore list; when false, remove.
* `reason` - Optional operator note for why the title is ignored.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::set_acquire_status

```rust
pub async fn set_acquire_status(
    &self,
    title_id: &str,
    account_id: &str,
    status: AcquireStatus,
    storage_key: Option<&str>,
    error_message: Option<&str>,
) -> Result<()>
```

Updates acquire status, storage key, and optional error message.

##### Arguments

* `title_id` - Product id / ASIN / ISBN used as the title key.
* `account_id` - Store account id.
* `status` - Acquire or request status value.
* `storage_key` - Object-storage key for the primary audio artifact.
* `error_message` - Optional failure message to persist.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::bulk_set_acquire_status

```rust
pub async fn bulk_set_acquire_status(
    &self,
    account: Option<&str>,
    asins: &[String],
    status: AcquireStatus,
) -> Result<u32>
```

Bulk-update acquire status (classic `set-status --force`).

When `asins` is non-empty, matches against uuid, product_id, isbn, or asin.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_saved_filters

```rust
pub async fn list_saved_filters(&self) -> Result<Vec<SavedFilterRecord>>
```

List saved quick-filter expressions.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_saved_filter

```rust
pub async fn upsert_saved_filter(
    &self,
    name: &str,
    query: &str,
) -> Result<SavedFilterRecord>
```

Inserts or replaces a named saved library filter query.

##### Arguments

* `name` - Logical name (filter, secret, …).
* `query` - Saved filter query string.

##### Returns

`Result<SavedFilterRecord>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_saved_filter

```rust
pub async fn get_saved_filter(&self, name: &str) -> Result<Option<SavedFilterRecord>>
```

Loads a saved filter by name, if present.

##### Arguments

* `name` - Logical name (filter, secret, …).

##### Returns

`Result<Option<SavedFilterRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::delete_saved_filter

```rust
pub async fn delete_saved_filter(&self, name: &str) -> Result<()>
```

Deletes a saved filter by name.

##### Arguments

* `name` - Logical name (filter, secret, …).

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::count_by_status

```rust
pub async fn count_by_status(&self, status: AcquireStatus) -> Result<i64>
```

Counts books whose acquire status equals `status`.

A peer writer on the same SQLite file (TRUNCATE journal) can make this
read return `SQLITE_BUSY`. The read is repeated until the lock clears
or the attempt budget is spent. `status` is unchanged.

##### Arguments

* `status` - Acquire or request status value.

##### Returns

`Result<i64>` — `Ok` on success.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::update_catalog_enrichment

```rust
pub async fn update_catalog_enrichment(
    &self,
    book_uuid: &str,
    fields: &CatalogEnrichmentFields,
) -> Result<()>
```

Persist enrichment fields without touching scan / ownership columns.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_work

```rust
pub async fn upsert_work(&self, work: &NewWork) -> Result<WorkRecord>
```

Upsert a canonical work row (COALESCE-preserve on conflict).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_work

```rust
pub async fn get_work(&self, id: &str) -> Result<Option<WorkRecord>>
```

Loads a canonical work by id, if present.

##### Arguments

* `id` - Row or work id.

##### Returns

`Result<Option<WorkRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::find_work_by_asin

```rust
pub async fn find_work_by_asin(&self, asin: &str) -> Result<Option<WorkRecord>>
```

Find work by Amazon ASIN identifier.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::find_work_by_isbn

```rust
pub async fn find_work_by_isbn(&self, isbn: &str) -> Result<Option<WorkRecord>>
```

Finds a canonical work whose preferred ISBN matches `isbn`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_works

```rust
pub async fn list_works(&self) -> Result<Vec<WorkRecord>>
```

Lists every canonical work row in the library database.


##### Returns

`Result<Vec<WorkRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::link_book_to_work

```rust
pub async fn link_book_to_work(&self, work_id: &str, book_uuid: &str) -> Result<()>
```

Associates a book UUID with a canonical work id.

##### Arguments

* `work_id` - Canonical work id.
* `book_uuid` - Library `books.uuid`.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::work_id_for_book

```rust
pub async fn work_id_for_book(&self, book_uuid: &str) -> Result<Option<String>>
```

Returns the work id linked to `book_uuid`, if any.

##### Arguments

* `book_uuid` - Library `books.uuid`.

##### Returns

`Result<Option<String>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::book_uuids_for_work

```rust
pub async fn book_uuids_for_work(&self, work_id: &str) -> Result<Vec<String>>
```

Lists book UUIDs linked to `work_id`.

##### Arguments

* `work_id` - Canonical work id.

##### Returns

`Result<Vec<String>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::upsert_listening_progress

```rust
pub async fn upsert_listening_progress(
    &self,
    row: &NewListeningProgress,
) -> Result<ListeningProgressRecord>
```

Inserts or updates listening progress for a title.

##### Arguments

* `row` - `row` argument for this call.

##### Returns

`Result<ListeningProgressRecord>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_listening_progress

```rust
pub async fn get_listening_progress(
    &self,
    provider: &str,
    external_user_id: &str,
    external_item_id: &str,
) -> Result<Option<ListeningProgressRecord>>
```

Loads listening progress for one identity and book.

##### Arguments

* `provider` - External provider id.
* `external_user_id` - User id at the external provider.
* `external_item_id` - `external_item_id` argument for this call.

##### Returns

`Result<Option<ListeningProgressRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::list_listening_progress

```rust
pub async fn list_listening_progress(
    &self,
    external_user_id: Option<&str>,
) -> Result<Vec<ListeningProgressRecord>>
```

Lists listening progress rows for an identity.

##### Arguments

* `external_user_id` - User id at the external provider.

##### Returns

`Result<Vec<ListeningProgressRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::create_title_request

```rust
pub async fn create_title_request(
    &self,
    req: &NewTitleRequest,
) -> Result<TitleRequestRecord>
```

Creates a wishlist / title-request row.

##### Arguments

* `req` - Insert / update request payload.

##### Returns

`Result<TitleRequestRecord>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::upsert_title_request_sources

```rust
pub async fn upsert_title_request_sources(
    &self,
    title_request_id: i64,
    sources: &[NewTitleRequestSource],
) -> Result<TitleRequestRecord>
```

Upsert per-storefront snapshots for a wishlist row, then return the merged record.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_title_request_sources

```rust
pub async fn list_title_request_sources(
    &self,
    title_request_id: i64,
) -> Result<Vec<TitleRequestSourceRecord>>
```

Lists per-storefront snapshots for a title request.

##### Arguments

* `title_request_id` - `title_request_id` argument for this call.

##### Returns

`Result<Vec<TitleRequestSourceRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::find_open_wishlist

```rust
pub async fn find_open_wishlist(
    &self,
    identity_id: Option<i64>,
    work_key: &str,
) -> Result<Option<TitleRequestRecord>>
```

Open wishlist row for this identity + work key, if any.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::find_open_wishlist_matching

```rust
pub async fn find_open_wishlist_matching(
    &self,
    identity_id: Option<i64>,
    work_key: &str,
    title: &str,
    authors: Option<&str>,
    asin: Option<&str>,
    isbn: Option<&str>,
) -> Result<Option<TitleRequestRecord>>
```

Open wishlist row that matches bibliographic identity even when `work_key` differs.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_wishlist

```rust
pub async fn list_wishlist(
    &self,
    identity_id: Option<i64>,
) -> Result<Vec<TitleRequestRecord>>
```

Personal open wishlist for a portal identity, or operator (`identity_id` null).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_global_request_queue

```rust
pub async fn list_global_request_queue(&self) -> Result<Vec<GlobalQueueEntry>>
```

Global request queue: open wishes grouped by `work_key`.

Sorted by wish count as a simple default; Discover re-ranks with local
taste plus a heavy per-wisher boost for the Wishlist sidebar.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_title_request_by_uuid

```rust
pub async fn get_title_request_by_uuid(
    &self,
    uuid: &str,
) -> Result<Option<TitleRequestRecord>>
```

Loads a wishlist title-request row by UUID, if present.

##### Arguments

* `uuid` - Book UUID.

##### Returns

`Result<Option<TitleRequestRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::list_title_requests

```rust
pub async fn list_title_requests(
    &self,
    status: Option<RequestStatus>,
) -> Result<Vec<TitleRequestRecord>>
```

Lists wishlist / title-request rows.

##### Arguments

* `status` - Acquire or request status value.

##### Returns

`Result<Vec<TitleRequestRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::update_title_request_status

```rust
pub async fn update_title_request_status(
    &self,
    uuid: &str,
    status: RequestStatus,
    resolved_book_uuid: Option<&str>,
) -> Result<()>
```

Updates the lifecycle status of a title request.

##### Arguments

* `uuid` - Book UUID.
* `status` - Acquire or request status value.
* `resolved_book_uuid` - `resolved_book_uuid` argument for this call.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::upsert_embedding

```rust
pub async fn upsert_embedding(
    &self,
    target_kind: &str,
    target_id: &str,
    model: &str,
    dims: i64,
    vector: &[u8],
    text_hash: &str,
) -> Result<()>
```

Inserts or replaces an embedding vector for a target.

##### Arguments

* `target_kind` - Embedding target kind.
* `target_id` - Embedding target id.
* `model` - Embedding model id.
* `dims` - Vector dimensionality.
* `vector` - Raw embedding bytes.
* `text_hash` - Hash of the embedded text.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_embedding_vector

```rust
pub async fn get_embedding_vector(
    &self,
    target_kind: &str,
    target_id: &str,
    model: &str,
) -> Result<Option<(String, Vec<u8>)>>
```

Loads the raw embedding bytes for a target, if present.

##### Arguments

* `target_kind` - Embedding target kind.
* `target_id` - Embedding target id.
* `model` - Embedding model id.

##### Returns

`Result<Option<(String, Vec<u8>)>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::list_embeddings

```rust
pub async fn list_embeddings(
    &self,
    target_kind: &str,
    model: &str,
) -> Result<Vec<(String, Vec<u8>)>>
```

Lists embedding metadata rows (optionally filtered).

##### Arguments

* `target_kind` - Embedding target kind.
* `model` - Embedding model id.

##### Returns

`Result<Vec<(String, Vec<u8>)>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::embedding_text_hash

```rust
pub async fn embedding_text_hash(
    &self,
    target_kind: &str,
    target_id: &str,
    model: &str,
) -> Result<Option<String>>
```

Returns the stored text hash for an embedding target, if any.

##### Arguments

* `target_kind` - Embedding target kind.
* `target_id` - Embedding target id.
* `model` - Embedding model id.

##### Returns

`Result<Option<String>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_user_preferences

```rust
pub async fn get_user_preferences(
    &self,
    subject_key: &str,
) -> Result<Option<UserPreferences>>
```

Load per-user GUI / Discover preferences by subject key.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_user_preferences_or_default

```rust
pub async fn get_user_preferences_or_default(
    &self,
    subject_key: &str,
    identity_id: Option<i64>,
) -> Result<UserPreferences>
```

Preferences for `subject_key`, or in-memory defaults when no row exists.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_user_preferences

```rust
pub async fn upsert_user_preferences(
    &self,
    subject_key: &str,
    identity_id: Option<i64>,
    default_view: &str,
    disabled_shelves: &[String],
    discover_sort: &str,
    discover_sort_dir: &str,
    discover_language: Option<&str>,
    discover_excluded_sources: &[String],
    theme: &str,
) -> Result<UserPreferences>
```

Insert or replace preferences for a subject (operator or portal identity).

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`

### NewBook

```rust
pub struct NewBook {
    pub uuid: Option<String>,
    pub product_id: String,
    pub plugin_key: String,
    pub source: String,
    pub account_id: String,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub marketplace: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub series_asin: Option<String>,
    pub purchased_at: Option<DateTime<Utc>>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub is_abridged: bool,
    pub content_kind: String,
    pub categories: Option<String>,
    pub subtitle: Option<String>,
    pub published_at: Option<DateTime<Utc>>,
}
```

Input for inserting / updating a book from sync.

**Fields**

- `uuid`: Stable UUID for this row (API / foreign-key identity).
- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).
- `plugin_key`: Provenance-qualified PluginKey that owns the account. Forced by the
  scope on write.
- `source`: Display alias of the owning plugin (`audible`, `libro`, …).
- `account_id`: Store or operator account id this row belongs to.
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `marketplace`: Store marketplace / locale code (for example `us`, `uk`).
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `series_asin`: Amazon series ASIN when the storefront exposes one.
- `purchased_at`: RFC 3339 purchase time from the storefront, when known.
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `is_abridged`: Whether the edition is abridged (0/1 or bool).
- `content_kind`: Title kind: `book`, `episode`, `podcast`, ….
- `categories`: Storefront category / genre path list (serialized).
- `subtitle`: Optional subtitle from bibliographic metadata.
- `published_at`: Publication date string from the storefront or enrichment.

#### NewBook::needs_html_entity_decode

```rust
pub fn needs_html_entity_decode(&self) -> bool
```

True when any human-readable field may contain an HTML entity.

#### NewBook::decode_html_entities

```rust
pub fn decode_html_entities(&mut self)
```

Decode HTML entities in human-readable metadata fields (titles, etc.).

#### NewBook::minimal

```rust
pub fn minimal(
    product_id: impl Into<String>,
    account_id: impl Into<String>,
    marketplace: impl Into<String>,
    title: impl Into<String>,
) -> Self
```

Minimal Audible book row with scan metadata defaults.

Sets `product_id` from the first argument, `asin = Some(product_id)`,
`isbn = None`, `uuid = None` (auto), and `plugin_key = source =
"audible"` (a [`crate::SourceScope`] replaces both on write).

Implements: `Clone`, `Debug`

### NewListeningProgress

```rust
pub struct NewListeningProgress {
    pub identity_id: Option<i64>,
    pub provider: String,
    pub external_user_id: String,
    pub book_uuid: Option<String>,
    pub work_id: Option<String>,
    pub external_item_id: String,
    pub title: Option<String>,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub progress: Option<f64>,
    pub current_time_seconds: Option<f64>,
    pub duration_seconds: Option<f64>,
    pub is_finished: bool,
    pub last_listened_at: Option<DateTime<Utc>>,
}
```

Input for upserting listening progress.

**Fields**

- `identity_id`: Foreign key to `portal_identities.id`.
- `provider`: External identity or integration provider id (for example ABS).
- `external_user_id`: User id at the external provider.
- `book_uuid`: Foreign key to `books.uuid`.
- `work_id`: Canonical work id this edition or request resolves to.
- `external_item_id`: Provider-native listening-progress item id.
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `progress`: Fractional progress 0.0–1.0 when the provider reports it.
- `current_time_seconds`: Current playback position within the title, in seconds.
- `duration_seconds`: Total duration in seconds when known.
- `is_finished`: Whether the listener marked the title finished (0/1 or bool).
- `last_listened_at`: RFC 3339 time of the last playback update.

Implements: `Clone`, `Debug`

### NewTitleRequest

```rust
pub struct NewTitleRequest {
    pub uuid: Option<String>,
    pub identity_id: Option<i64>,
    pub title: String,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub notes: Option<String>,
    pub status: RequestStatus,
    pub work_key: String,
    pub work_id: Option<String>,
    pub resolved_book_uuid: Option<String>,
    pub cover_url: Option<String>,
}
```

Input for creating a title request / wishlist row.

**Fields**

- `uuid`: Stable UUID for this row (API / foreign-key identity).
- `identity_id`: Foreign key to `portal_identities.id`.
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `notes`: Free-form operator or requester notes.
- `status`: Lifecycle status for the row (user, request, …).
- `work_key`: Stable bibliographic key; empty triggers [`fallback_work_key`](#fallback_work_key).
- `work_id`: Canonical work id this edition or request resolves to.
- `resolved_book_uuid`: Library `books.uuid` once the wishlist item is fulfilled.
- `cover_url`: HTTPS URL for cover art when known.

#### NewTitleRequest::needs_html_entity_decode

```rust
pub fn needs_html_entity_decode(&self) -> bool
```

Returns true when the string still contains HTML character entities.

#### NewTitleRequest::decode_html_entities

```rust
pub fn decode_html_entities(&mut self)
```

Decode HTML entities in human-readable fields.

Implements: `Clone`, `Debug`

### NewTitleRequestSource

```rust
pub struct NewTitleRequestSource {
    pub source: String,
    pub product_id: String,
    pub title: Option<String>,
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub cover_url: Option<String>,
    pub url: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub list_price_cents: Option<i64>,
    pub list_price_label: Option<String>,
    pub member_price_cents: Option<i64>,
    pub member_price_label: Option<String>,
}
```

Input for upserting a per-storefront wishlist snapshot.

**Fields**

- `source`: Content-source plugin id (`audible`, `libro`, …).
- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).
- `title`: Display title of the work or edition.
- `subtitle`: Optional subtitle from bibliographic metadata.
- `authors`: Comma-separated or JSON author list from the storefront.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `description`: Blurb / synopsis text (may contain HTML).
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `published_at`: Publication date string from the storefront or enrichment.
- `categories`: Storefront category / genre path list (serialized).
- `language`: BCP-47 or storefront language code when known.
- `cover_url`: HTTPS URL for cover art when known.
- `url`: Storefront product or purchase URL when known.
- `price_cents`: Observed price in minor currency units, when known.
- `currency`: ISO 4217 currency code for price fields.
- `price_label`: Storefront-formatted price string for display.
- `list_price_cents`: List/MSRP price in minor units, when known.
- `list_price_label`: Storefront-formatted list price for display.
- `member_price_cents`: Member/subscriber price in minor units, when known.
- `member_price_label`: Storefront-formatted member price for display.

#### NewTitleRequestSource::needs_html_entity_decode

```rust
pub fn needs_html_entity_decode(&self) -> bool
```

Returns true when the string still contains HTML character entities.

#### NewTitleRequestSource::decode_html_entities

```rust
pub fn decode_html_entities(&mut self)
```

Decode HTML entities in human-readable metadata fields.

Implements: `Clone`, `Debug`, `Default`

### NewWork

```rust
pub struct NewWork {
    pub id: Option<String>,
    pub canonical_asin: Option<String>,
    pub canonical_isbn: Option<String>,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub description: Option<String>,
    pub subjects: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub cover_url: Option<String>,
    pub openlibrary_id: Option<String>,
}
```

Input for upserting a canonical work.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `canonical_asin`: Preferred ASIN representing this canonical work.
- `canonical_isbn`: Preferred ISBN representing this canonical work.
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `narrators`: Comma-separated or JSON narrator list when present.
- `description`: Blurb / synopsis text (may contain HTML).
- `subjects`: Subject / topic tags from enrichment (serialized).
- `categories`: Storefront category / genre path list (serialized).
- `language`: BCP-47 or storefront language code when known.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `cover_url`: HTTPS URL for cover art when known.
- `openlibrary_id`: Open Library work/edition id when enrichment found one.

Implements: `Clone`, `Debug`, `Default`

### SavedFilterRecord

```rust
pub struct SavedFilterRecord {
    pub id: i64,
    pub name: String,
    pub query: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Saved Lucene-style quick filter.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `name`: Human-readable or logical name for this row.
- `query`: Saved library filter expression (UI search DSL).
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `Debug`

### TitleFile

```rust
pub struct TitleFile {
    pub book_uuid: String,
    pub writer: String,
}
```

Who holds a recorded file key.

**Fields**

- `book_uuid`: Library id of the title the file belongs to.
- `writer`: The host upload id that claimed the key last, or [`ACQUIRE_WRITER`](#acquire_writer).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### UserBookFields

```rust
pub struct UserBookFields {
    pub tags: Option<String>,
    pub rating_overall: Option<f32>,
    pub rating_performance: Option<f32>,
    pub rating_story: Option<f32>,
    pub is_finished: Option<bool>,
}
```

Partial update for user-defined book fields.

**Fields**

- `tags`: Operator or storefront tags (serialized string).
- `rating_overall`: Overall user rating from the storefront, if any.
- `rating_performance`: Narration/performance rating from the storefront, if any.
- `rating_story`: Story rating from the storefront, if any.
- `is_finished`: Whether the listener marked the title finished (0/1 or bool).

Implements: `Clone`, `Debug`, `Default`

### WishlistIdentity

```rust
pub struct WishlistIdentity<'a> {
    pub work_key: &'a str,
    pub title: &'a str,
    pub authors: Option<&'a str>,
    pub asin: Option<&'a str>,
    pub isbn: Option<&'a str>,
}
```

Bibliographic slice used for wishlist dedupe.

**Fields**

- `work_key`: Stable merge key used to group editions into a work.
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.

#### WishlistIdentity::matches

```rust
pub fn matches(self, other: WishlistIdentity<'_>) -> bool
```

Whether two wishlist rows refer to the same work despite differing keys.

Matches exact `work_key`, shared ASIN/ISBN, or the soft title+author key
from [`fallback_work_key`](#fallback_work_key). Cross-key isbn↔asin pairs require a soft title
agreement so unrelated hard keys never collapse.

Implements: `Clone`, `Copy`, `Debug`

## Enums

### DbAtomicParams

```rust
pub enum DbAtomicParams {
    DeleteUser { user_id: i64 },
    SetUserStatus { user_id: i64, status: String },
    SetUserPasswordHash { user_id: i64, password_hash: Option<String> },
    SetUserRole { user_id: i64, role: String },
    RedeemClaimTicket { token_hash: String, session_hash: String, expires_at: String, user_agent: Option<String>, device_type: Option<String>, client_label: Option<String>, new_password_hash: Option<String>, password_fingerprint: Option<String> },
    TakeOidcRpState { state_hash: String },
    TakeWebauthnChallenge { challenge_id: String, kind: String },
    EnqueueJob { kind: String, payload_json: String, priority: i64, max_attempts: i64, max_pending: i64, run_after: Option<String> },
    ClaimNextJob { resource_class: String, owner: String, lease_secs: i64 },
    ReserveJobTemp { job_id: String, path: String, reserved_bytes: i64, quota_bytes: i64 },
    ConfirmTotpEnrollment { user_id: i64, format: String, ciphertext: String, cipher_algorithm: Option<String>, cipher_nonce: Option<String>, kdf_algorithm: Option<String>, kdf_salt: Option<String>, kdf_m_cost: Option<i64>, kdf_t_cost: Option<i64>, kdf_p_cost: Option<i64>, created_at: String },
    DisableUserTotp { user_id: i64 },
    PublishDomainEvent { id: String, event_type: String, schema_version: i64, account_id: String, source: String, correlation_id: String, causation_id: String, dedup_key: String, payload: String, ordering_key: String, queue_name: String, content_type: String, deliver_after: String },
    SetAcquireStatus { book_uuid: String, status: String, storage_key: Option<String>, error_message: Option<String>, event_id: String, event_type: String, schema_version: i64, event_account_id: String, source: String, correlation_id: String, causation_id: String, dedup_key: String, payload: String, ordering_key: String },
    DispatchEventDeliveries { event_id: String, subscribers_json: String, mark_dispatched: bool },
    ClaimNextEventDelivery { owner: String, lease_secs: i64, plugin_ids_json: String, max_in_flight: i64 },
}
```

Named atomic library operation compiled by the host into a generic plan.

**Variants**

- `DeleteUser`: Delete a first-party user and personal data (last-owner guarded).
  - `user_id`: `users.id` to delete.
- `SetUserStatus`: Set `users.status` (`active` / `disabled`; last-owner guarded).
  - `user_id`: `users.id` to update.
  - `status`: Canonical status string (`active` or `disabled`).
- `SetUserPasswordHash`: Set or clear the Argon2id password hash and bump `security_version`.
  - `user_id`: `users.id` to update.
  - `password_hash`: New hash, or `null` to clear.
- `SetUserRole`: Set `users.role` (last-owner guarded on demotion).
  - `user_id`: `users.id` to update.
  - `role`: Canonical role string (`owner` / `administrator` / `member`).
- `RedeemClaimTicket`: Consume a claim ticket, optionally set a first password, mint a session.
  - `token_hash`: SHA-256 hex digest of the claim ticket.
  - `session_hash`: SHA-256 hex digest of the new portal session token.
  - `expires_at`: RFC 3339 expiry for the minted session.
  - `user_agent`: Raw User-Agent captured at session mint.
  - `device_type`: Best-effort device class.
  - `client_label`: Best-effort OS / client label.
  - `new_password_hash`: Argon2id hash to set when the local user has none.
  - `password_fingerprint`: Domain-separated HMAC of the invite password for idempotency.
- `TakeOidcRpState`: Consume a one-time OIDC RP state (`DELETE` + expiry check).
  - `state_hash`: SHA-256 hex digest of the OAuth `state` parameter.
- `TakeWebauthnChallenge`: Consume a one-time WebAuthn challenge (`DELETE` + expiry check).
  - `challenge_id`: Public ceremony id returned to the browser.
  - `kind`: `register`, `login`, or `elevate`.
- `EnqueueJob`: Admit a durable job (dedup + pending cap + insert).
  - `kind`: Job kind wire string (`scan`, `acquire`, `listen_sync`, `integration_scan`).
  - `payload_json`: Versioned command envelope JSON.
  - `priority`: Higher values are claimed first.
  - `max_attempts`: Maximum claims before a failure is terminal.
  - `max_pending`: Global cap on pending+running rows.
  - `run_after`: Optional RFC 3339 delay before the job becomes claimable.
- `ClaimNextJob`: Claim the next ready job in `resource_class` for `owner`.
  - `resource_class`: Resource class wire string (`network`, …).
  - `owner`: Worker id stored as `lease_owner`.
  - `lease_secs`: Lease length in seconds.
- `ReserveJobTemp`: Reserve scratch-quota bytes for one job path.
  - `job_id`: Owning job id.
  - `path`: Absolute filesystem path.
  - `reserved_bytes`: Bytes to reserve for this path.
  - `quota_bytes`: Global reserved-bytes cap.
- `ConfirmTotpEnrollment`: Promote a sealed TOTP secret to `primary` and set `users.totp_enabled`.
  - `user_id`: `users.id` to enroll.
  - `format`: Payload format (`sealed-v1`).
  - `ciphertext`: Sealed TOTP secret bytes (`b64:…`).
  - `cipher_algorithm`: Cipher algorithm identifier, if any.
  - `cipher_nonce`: AEAD nonce (`b64:…`), if any.
  - `kdf_algorithm`: Legacy KDF algorithm, if any.
  - `kdf_salt`: Legacy KDF salt (`b64:…`), if any.
  - `kdf_m_cost`: Legacy Argon2 memory cost in KiB, if any.
  - `kdf_t_cost`: Legacy Argon2 time cost, if any.
  - `kdf_p_cost`: Legacy Argon2 parallelism, if any.
  - `created_at`: RFC 3339 timestamp to store as `created_at` on the primary row.
- `DisableUserTotp`: Delete TOTP secrets and clear `users.totp_enabled`.
  - `user_id`: `users.id` to disable.
- `PublishDomainEvent`: Persist a domain event in the outbox (dedup on eventType+dedupKey).
  - `id`: Stable event id (UUID).
  - `event_type`: Event type (`book_acquired`).
  - `schema_version`: Payload schema version.
  - `account_id`: Tenant / account id.
  - `source`: Producer plugin id; empty when unknown.
  - `correlation_id`: Trace correlation id.
  - `causation_id`: Causing event or job id.
  - `dedup_key`: Unique with `eventType`.
  - `payload`: Bounded JSON payload.
  - `ordering_key`: FIFO key copied onto deliveries.
  - `queue_name`: Queue name; empty means the domain-event bus (`core.events`).
  - `content_type`: Body encoding (`json`, `text`, `bytes`); empty means `json`.
  - `deliver_after`: RFC 3339 earliest delivery time; empty means now.
- `SetAcquireStatus`: Update a book row and optionally publish `book_acquired` in the same batch.
  - `book_uuid`: Book UUID to update.
  - `status`: Acquire status string (`acquired`, `downloading`, …).
  - `storage_key`: Object-storage key for the primary audio artifact.
  - `error_message`: Optional failure message.
  - `event_id`: Event id when publishing; empty skips the outbox insert.
  - `event_type`: Event type when publishing (`book_acquired`).
  - `schema_version`: Payload schema version.
  - `event_account_id`: Tenant / account id on the event.
  - `source`: Producer plugin id; empty when unknown.
  - `correlation_id`: Trace correlation id.
  - `causation_id`: Causing event or job id.
  - `dedup_key`: Unique with `eventType`.
  - `payload`: Bounded JSON payload.
  - `ordering_key`: FIFO key copied onto deliveries.
- `DispatchEventDeliveries`: Create per-subscriber deliveries and mark the event dispatched.
  - `event_id`: Parent event id.
  - `subscribers_json`: JSON array of `{ "pluginId": "…" }` subscriber snapshots.
  - `mark_dispatched`: When true, this page is the last snapshot and may mark the parent dispatched.
- `ClaimNextEventDelivery`: Claim the next ready event delivery for `owner`.
  - `owner`: Worker id stored as `lease_owner`.
  - `lease_secs`: Lease length in seconds.
  - `plugin_ids_json`: JSON array of plugin ids this worker can execute (`[]` claims nothing).
  - `max_in_flight`: Cluster-wide max `running` deliveries per `(plugin_id, resource_class)`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### CanonicalObject

```rust
pub enum CanonicalObject {
    Schema { sql_contract_version: u32, statements: Vec<String> },
    TableChunk { table: String, columns: Vec<String>, rows: Vec<Vec<DbValue>> },
    Identity { entries: BTreeMap<String, IdentityHighWater> },
}
```

One immutable canonical object (hashed before compression).

**Variants**

- `Schema`: Admitted `CREATE TABLE` / `CREATE INDEX` in restore order.
  - `sql_contract_version`: Bookclerk SQL grammar/ABI contract.
  - `statements`: Canonical statements (tables in FK-safe order, then indexes).
- `TableChunk`: Bounded table rows in canonical order.
  - `table`: Folded table name.
  - `columns`: Column names in CREATE declaration order.
  - `rows`: Row-major `DbValue` cells.
- `Identity`: Identity high-water for generated columns.
  - `entries`: Folded table name → high-water.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### BackupReason

```rust
pub enum BackupReason {
    Manual,
    PreMigrate,
}
```

Why a backup was written.

**Variants**

- `Manual`: Operator-created (`bookclerk db backup create`). Never pruned automatically.
- `PreMigrate`: Written immediately before a frozen schema walk.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### BackupResolve

```rust
pub enum BackupResolve {
    Id(String),
    Archive(PathBuf),
}
```

How [`resolve_backup_spec`](#resolve_backup_spec) located a recovery point.

**Variants**

- `Id`: Catalog id under `backups/manifests/`.
- `Archive`: `.tar.gz` archive of one recovery point.

Implements: `Clone`, `Debug`

### CanonicalRestoreKind

```rust
pub enum CanonicalRestoreKind {
    Library,
    PluginBinding,
}
```

How restore applies DDL companions.

**Variants**

- `Library`: Host library: Postgres identity companions only (no plugin catalog).
- `PluginBinding`: Plugin binding: catalog + Postgres identity companions.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### DatabaseUnitKind

```rust
pub enum DatabaseUnitKind {
    Library,
    PluginBinding,
}
```

Logical kind of one database unit in a recovery point.

**Variants**

- `Library`: Host library database.
- `PluginBinding`: Plugin-owned binding (`plugin_id` + `binding`).

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LibraryError

```rust
pub enum LibraryError {
    Schema(String),
    Orm(DbErr),
    Config(ConfigError),
    Io(Error),
    NotFound(String),
    LastAdministrator,
    LastOwner,
    InvalidEmail,
    Unavailable(String),
    Conflict(String),
    Forbidden(String),
    Invalid(String),
    Other(Error),
}
```

Failures from library store, schema apply, secrets, or config.

**Variants**

- `Schema`: Schema versioning, backup, or canonical SQL failure.
- `Orm`: SeaORM / database-plugin failure (`DbErr`).
- `Config`: Configuration load or validation failure.
- `Io`: Filesystem or other I/O failure.
- `NotFound`: Requested book (or similar library row) was not found.
- `LastAdministrator`: Demoting or disabling the last active administrator is refused.
- `LastOwner`: Demoting, disabling, or deleting the last active owner is refused.
- `InvalidEmail`: Contact email failed library-backed syntax validation.
- `Unavailable`: Backend temporarily unreachable or an atomic RPC response was lost.
  
  Callers that still hold the original consume-once / session token should
  retry the same atomic operation id rather than minting a new one.
- `Conflict`: Identity or ownership conflict (for example a taken OIDC `client_id`).
- `Forbidden`: The caller may not perform this write (for example a configuration
  write without operator authority).
- `Invalid`: The request or stored document failed validation.
- `Other`: Catch-all for otherwise unclassified failures.

#### LibraryError::from_db_err

```rust
pub fn from_db_err(err: DbErr) -> Self
```

Maps a SeaORM / adapter [`sea_orm::DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) onto a typed library error.

Busy, deadlock, and lost-commit tokens become [`Self::Unavailable`](#libraryerror).
Unique/duplicate-object tokens become [`Self::Conflict`](#libraryerror). Foreign-key,
CHECK, and NOT NULL stay [`Self::Orm`](#libraryerror). Everything else stays [`Self::Orm`](#libraryerror).

#### LibraryError::is_schema_apply_retryable

```rust
pub fn is_schema_apply_retryable(&self) -> bool
```

True when host schema apply may retry after re-reading durable state.

Implements: `Debug`, `Display`, `Error`, `From<ConfigError>`, `From<DbErr>`, `From<Error>`

### AcquireStatus

```rust
pub enum AcquireStatus {
    NotAcquired,
    Queued,
    Downloading,
    Acquired,
    Error,
}
```

Acquire / download state for a title.

**Variants**

- `NotAcquired`: Title has not been queued or downloaded yet.
- `Queued`: Title is waiting in the acquire queue.
- `Downloading`: Acquire pipeline is actively downloading this title.
- `Acquired`: Primary audio artifact is stored and library status is liberated.
- `Error`: Last acquire attempt failed; see `error_message` on the book row.

#### AcquireStatus::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case / lowercase wire string.

#### AcquireStatus::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

#### AcquireStatus::to_classic

```rust
pub fn to_classic(self) -> i32
```

Classic EF `LiberatedStatus` integer.

#### AcquireStatus::from_classic

```rust
pub fn from_classic(status: i64) -> Self
```

Parse classic EF `LiberatedStatus` integer.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### EnqueueOutcome

```rust
pub enum EnqueueOutcome {
    Created { id: String },
    Duplicate { existing_id: String },
    QueueFull,
}
```

Result of admitting a job into the durable queue.

**Variants**

- `Created`: A new row was inserted.
  - `id`: New job id.
- `Duplicate`: An equivalent pending/running job already exists.
  - `existing_id`: Existing job id that satisfies the dedup key.
- `QueueFull`: `pending + running` already equals `max_pending`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### JobKind

```rust
pub enum JobKind {
    Scan,
    Acquire,
    ListenSync,
    IntegrationScan,
    PluginCopy,
    Invalid,
}
```

Kind of durable daemon work stored in the `jobs` table.

**Variants**

- `Scan`: Library scan for one account or every scan-enabled account.
- `Acquire`: Acquire pending titles (optional title / account filter).
- `ListenSync`: Sync listening progress from capable integrations.
- `IntegrationScan`: Remote integration library scan (Audiobookshelf, …).
- `PluginCopy`: ABI `JobRunner` stream-copy vertical slice (`plugin_copy`).
- `Invalid`: Terminal placeholder after an unreadable persisted command is rejected.

#### JobKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case wire string.

#### JobKind::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

#### JobKind::resource_class

```rust
pub fn resource_class(self) -> JobResourceClass
```

Resource class used for admission and worker concurrency.

#### JobKind::dedup_key

```rust
pub fn dedup_key(self, payload: &JobPayload) -> String
```

Idempotency key unique among pending/running jobs of the same work.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobResourceClass

```rust
pub enum JobResourceClass {
    Network,
    Media,
    Transcription,
    Indexing,
}
```

Concurrency class that bounds how many jobs of this type run at once.

**Variants**

- `Network`: Store HTTP / scan / acquire / listening-sync work.
- `Media`: Codec / remux work (reserved; media pool remains separate).
- `Transcription`: Speech-to-text / derived transcript work (reserved for #121).
- `Indexing`: Search-index / embedding work (reserved).

#### JobResourceClass::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case wire string.

#### JobResourceClass::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

#### JobResourceClass::ALL

```rust
pub const ALL: &'static [&'static str] = _
```

Canonical wire strings for every known class.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobState

```rust
pub enum JobState {
    Pending,
    Running,
    Succeeded,
    Failed,
    Cancelled,
}
```

Lifecycle state of a durable job row.

**Variants**

- `Pending`: Admitted and waiting to be claimed.
- `Running`: Claimed by a worker that holds a live lease.
- `Succeeded`: Finished successfully.
- `Failed`: Exhausted retries or failed without retry.
- `Cancelled`: Cancelled by an operator before or during execution.

#### JobState::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case wire string.

#### JobState::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

#### JobState::is_active

```rust
pub fn is_active(self) -> bool
```

True when the job may still be claimed or is executing.

#### JobState::is_terminal

```rust
pub fn is_terminal(self) -> bool
```

True when the job will not run again.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobTrigger

```rust
pub enum JobTrigger {
    Api,
    Scheduler,
}
```

Who admitted the job (API vs periodic scheduler).

**Variants**

- `Api`: Enqueued from the HTTP control plane, tray, or CLI daemon commands.
- `Scheduler`: Enqueued by the daemon interval scheduler.

#### JobTrigger::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case wire string.

#### JobTrigger::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PublishDomainEventOutcome

```rust
pub enum PublishDomainEventOutcome {
    Created { id: String },
    Duplicate { existing_id: String },
}
```

Result of [`crate::LibraryStore::publish_domain_event`](#librarystorepublish_domain_event).

**Variants**

- `Created`: A new outbox row was inserted.
  - `id`: Event id.
- `Duplicate`: An equivalent `(event_type, dedup_key)` row already exists.
  - `existing_id`: Existing event id.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RequestStatus

```rust
pub enum RequestStatus {
    Open,
    Cancelled,
}
```

Wishlist row status (`open` while wishlisted; `cancelled` after un-wishlist).

**Variants**

- `Open`: Wishlist item is still open (not fulfilled/cancelled).
- `Cancelled`: Request cancelled by the requester or operator.

#### RequestStatus::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case / lowercase wire string.

#### RequestStatus::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### UserRole

```rust
pub enum UserRole {
    Owner,
    Administrator,
    Host,
    Member,
}
```

First-party user role (`owner` | `administrator` | `host` | `member`).

**Variants**

- `Owner`: Cluster super-user: may elevate to Operator after password re-auth.
- `Administrator`: Library / user administration without Operator elevation.
- `Host`: Member who administers the nodes they own (budgets, drain, name).
  Not trusted with other users' work; promote to Administrator for that.
- `Member`: Library member with scoped portal access (no operator token).

#### UserRole::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case / lowercase wire string.

#### UserRole::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

#### UserRole::is_privileged

```rust
pub fn is_privileged(self) -> bool
```

Whether this role may provision users and manage the host library.

#### UserRole::can_elevate

```rust
pub fn can_elevate(self) -> bool
```

Whether this role may elevate to a short-lived Operator session.

#### UserRole::can_administer_owned_nodes

```rust
pub fn can_administer_owned_nodes(self) -> bool
```

Whether this role reaches the node administration routes even while
the user owns no node (Host, Administrator, Owner). Any other active
user reaches them once they own a node: owning a node does not
require a role.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `PartialEq`, `Serialize`

### UserStatus

```rust
pub enum UserStatus {
    Active,
    Disabled,
}
```

First-party user status.

**Variants**

- `Active`: Account may sign in and use the portal.
- `Disabled`: Account is blocked from new sessions.

#### UserStatus::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case / lowercase wire string.

#### UserStatus::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SchemaState

```rust
pub enum SchemaState {
    Uninitialized,
    Unreleased { base_version: i64, checksum: String },
    Frozen { version: i64, checksum: String },
}
```

Durable Bookclerk library schema state.

**Variants**

- `Uninitialized`: No Bookclerk bookkeeping and no host tables.
- `Unreleased`: Development pack applied on top of a frozen base (or pre-v1).
  - `base_version`: Highest frozen plan version this unreleased pack sits on (`0` = none).
  - `checksum`: SHA-256 hex of [`crate::migrations::unreleased_ops`](migrations.md#unreleased_ops).
- `Frozen`: A frozen [`crate::migrations::HostMigrationStep`](migrations.md#hostmigrationstep) is applied.
  - `version`: Frozen plan version (`>= 1`).
  - `checksum`: SHA-256 hex recorded for that frozen step.

#### SchemaState::display

```rust
pub fn display(&self) -> String
```

CLI / JSON display: `uninitialized`, `unreleased@base<n>+<checksum>`,
`frozen@<version>+<checksum>`.

#### SchemaState::frozen_version

```rust
pub fn frozen_version(&self) -> Option<i64>
```

Frozen revision when this state is [`Self::Frozen`](#schemastate); otherwise `None`.

Never use `0` as a stand-in for uninitialized or unreleased.

#### SchemaState::unreleased_base_version

```rust
pub fn unreleased_base_version(&self) -> Option<i64>
```

Frozen base recorded on an unreleased database (`0` = pre-v1).

#### SchemaState::checksum

```rust
pub fn checksum(&self) -> Option<&str>
```

Checksum when the database has applied Bookclerk schema.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

### EventPruneStep

```rust
pub enum EventPruneStep {
    TerminalDeliveries,
    DeadLetters,
    StaleSubscriberNodes,
    EmptyEvents,
}
```

One delete of [`LibraryStore::prune_event_deliveries`](#librarystoreprune_event_deliveries), in run order.

**Variants**

- `TerminalDeliveries`: Acked and rejected deliveries past `retention_days`.
- `DeadLetters`: Dead letters past `dead_letter_retention_days`.
- `StaleSubscriberNodes`: Per-node subscriber heartbeats past their TTL.
- `EmptyEvents`: Dispatched events past `retention_days` with no deliveries left.

#### EventPruneStep::ALL

```rust
pub const ALL: [Self; 4] = _
```

Every step, in the order the deletes must run (deliveries before the
events they belong to).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### KvPutOutcome

```rust
pub enum KvPutOutcome {
    Written,
    ValueTooLarge { len: usize, max: u32 },
    KeyLimitReached { max: u32 },
}
```

Why a write was refused, so the binding can map it onto a wire code.

**Variants**

- `Written`: The row was written.
- `ValueTooLarge`: `value` exceeds `max_value_bytes`.
  - `len`: Bytes offered.
  - `max`: Namespace cap.
- `KeyLimitReached`: The namespace already holds `max_keys` other live keys.
  - `max`: Namespace cap.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### migrate_library_backend

```rust
pub async fn migrate_library_backend(
    source: &DatabaseConnection,
    dest: &DatabaseConnection,
    opts: &BackendMigrateOptions,
) -> Result<BackendMigrateSummary>
```

Copy all library tables from `source` into `dest`.

The destination should be empty unless [`BackendMigrateOptions::force`](#backendmigrateoptions) is set.
Runs in a single transaction on the destination when not dry-run.

#### Arguments

* `source` - Existing library backend to read from.
* `dest` - Target backend to write into (must already be migrated).
* `opts` - Dry-run / force behavior.

#### Returns

Per-table row counts copied (or that would be copied when `dry_run`).

#### Errors

Returns [`LibraryError`](#libraryerror) when the destination is non-empty without `force`,
or when any read/write fails.

### restore_backup_unit

```rust
pub async fn restore_backup_unit(
    db: &DatabaseConnection,
    repo: &BackupRepository,
    unit: &BackupUnit,
    kind: CanonicalRestoreKind,
    opts: &CanonicalRestoreOpts,
    preserve_plugin_registry: bool,
) -> Result<()>
```

Destructively replaces one logical database with `unit`. Never merges.

Verifies every referenced object before the first `DROP`. Plugin restore
rebuilds catalog companions from admitted SQL. It does not run plugin-owned
migrations. When `preserve_plugin_registry` is true, library restore does
not drop or replay `plugin_databases` (environment-local placement).
Row inserts are parameterized canonical Bookclerk SQL (`?` + typed
`DbValue` binds); adapters lower placeholders at the execute edge.
SQLite restore defers foreign keys on the restore transaction and runs
`PRAGMA foreign_key_check` before commit. It never toggles
`PRAGMA foreign_keys` on the pool connection.

#### Errors

Returns when validation fails, the adapter cannot provide complete unit
replacement, a bind/payload/request budget is exceeded, or DDL/DML cannot
be applied.

### library_ddl_for_schema_state

```rust
pub fn library_ddl_for_schema_state(state: &SchemaState) -> Result<String>
```

Library DDL that matches `state`, not necessarily this binary's latest pack.

#### Errors

Returns when the database is uninitialized, checksums do not match, or a
frozen version is unknown to this binary.

### sort_tables_by_foreign_keys

```rust
pub fn sort_tables_by_foreign_keys(
    tables: Vec<CanonicalTableSchema>,
) -> Result<Vec<CanonicalTableSchema>>
```

FK-safe table order (parents before children). Cycles fail closed.

Column-level `REFERENCES` and table-level `FOREIGN KEY` both count.
Self-references are ignored. References to tables outside this schema
are ignored. Ready nodes keep original declaration order so a child named
`a_child` that references `z_parent` still waits for the parent.

#### Errors

Returns when the FK graph contains a cycle.

### verify_recovery_point

```rust
pub fn verify_recovery_point(repo: &BackupRepository, id: &str) -> Result<ValidatedBackup>
```

Parses the manifest, rejects unsupported formats, and verifies every
referenced object **before** the caller starts a destructive restore.

#### Errors

Returns when the manifest is malformed, an object is missing/corrupt, schema
is not fully admitted, or typed cells violate the contract.

### admit_canonical_schema

```rust
pub fn admit_canonical_schema(
    sql_contract_version: u32,
    sql: &str,
) -> Result<CanonicalDatabaseSchema>
```

Parses and fully admits a diagnostic/imported canonical SQL script.

Production capture, checksum, apply, and restore paths use already-separated
statement lists ([`admit_canonical_statements`](#admit_canonical_statements)). This helper exists for
tests and imported scripts that still arrive as one packable string.

#### Errors

Returns when the packer rejects `sql` or any statement is not admitted.

### admit_canonical_statements

```rust
pub fn admit_canonical_statements(
    sql_contract_version: u32,
    statements: &[&str],
) -> Result<CanonicalDatabaseSchema>
```

Admits already-separated canonical `CREATE TABLE` / `CREATE INDEX` statements.

#### Errors

Returns when any statement is not admitted Bookclerk SQL.

### apply_admitted_sql

```rust
pub async fn apply_admitted_sql(
    db: &DatabaseConnection,
    statements: &[&str],
    kind: CanonicalRestoreKind,
) -> Result<()>
```

Applies admitted canonical DDL plus binding companions (tests / bootstrap).

#### Errors

Returns when a statement fails.

### archive_backup

```rust
pub fn archive_backup(files_dir: &Path, id: &str, dest: &Path) -> Result<()>
```

Packages one recovery point (manifest + referenced objects) as `.tar.gz`.

#### Errors

Returns when the destination cannot be written or objects are missing.

### backup_library

```rust
pub async fn backup_library(
    db: &DatabaseConnection,
    req: &BackupRequest,
) -> Result<Option<BackupOutcome>>
```

Writes a crash-safe recovery point under `files_dir/backups/`.

Skips only [`SchemaState::Uninitialized`](#schemastate). The manifest is published only
after every referenced object exists. Pre-migrate backups prune older
automatic recovery points and garbage-collect unreferenced objects.
Manual backups are never auto-pruned.

#### Errors

Returns when the files dir cannot be written, capture fails, or a requested
plugin binding is missing from [`BackupRequest::plugin_units`](#backuprequest).

### extract_backup_archive

```rust
pub fn extract_backup_archive(archive: &Path, dest: &Path) -> Result<()>
```

Extracts a `.tar.gz` recovery-point archive into `dest` (a backups root).

Expansion is bounded by entry count, per-entry size, total expanded bytes,
and gzip-decoded tar stream size so a small archive cannot fill the disk.

#### Errors

Returns when the archive is missing, malformed, a path escapes `dest`, or
an extract budget would be exceeded.

### filter_library_pack_ddl

```rust
pub fn filter_library_pack_ddl(sql: &str) -> Result<String>
```

Library migration packs may include seed `INSERT`s. Those are table data
(captured from rows), not schema objects. This keeps admitted
`CREATE TABLE` / `CREATE INDEX` and skips seed DML. Any other statement
fails closed.

#### Errors

Returns when a non-empty statement is neither admitted DDL nor seed DML.

### library_canonical_schema

```rust
pub fn library_canonical_schema() -> Result<CanonicalDatabaseSchema>
```

Admitted library schema for this binary's current pack (frozen ups + unreleased).

#### Errors

Returns when current canonical SQL is not fully admitted.

### library_canonical_schema_for_state

```rust
pub fn library_canonical_schema_for_state(
    state: &SchemaState,
) -> Result<CanonicalDatabaseSchema>
```

Admitted library schema for `state`.

#### Errors

Returns when DDL cannot be resolved or is not fully admitted.

### list_backups

```rust
pub fn list_backups(files_dir: &Path) -> Result<Vec<BackupListEntry>>
```

Lists published recovery points oldest-first.

#### Errors

Returns when the repository cannot be opened.

### plugin_canonical_schema_from_ddl_catalog

```rust
pub async fn plugin_canonical_schema_from_ddl_catalog(
    db: &DatabaseConnection,
) -> Result<CanonicalDatabaseSchema>
```

Rebuilds plugin schema from durable `bookclerk_sql_ddl` on an open connection.

#### Errors

Returns when the DDL catalog is missing, incomplete, or malformed.

### prune_automatic_backups

```rust
pub fn prune_automatic_backups(files_dir: &Path) -> Result<usize>
```

Deletes oldest automatic pre-migrate recovery points. Never deletes `manual`.

#### Errors

Returns when a manifest cannot be unlinked.

### resolve_backup_spec

```rust
pub fn resolve_backup_spec(files_dir: &Path, spec: &str) -> Result<BackupResolve>
```

Resolves a recovery-point id, timestamp prefix, or archive path.

#### Errors

Returns when no complete backup matches or the spec is ambiguous.

### restore_backup

```rust
pub async fn restore_backup(
    db: &DatabaseConnection,
    files_dir: &Path,
    id: &str,
    opts: &CanonicalRestoreOpts,
) -> Result<RestorePlan>
```

Restores the library unit after verifying **all** objects and units.

Plugin units are returned for the host/CLI to restore onto the target
adapter. Restore replaces Bookclerk-visible schema for the library unit
and does not merge or auto-migrate. Plugin-owned migrations are not run.

The library `plugin_databases` registry is preserved when the recovery
point carries no plugin units (including `--include-plugin-databases` with
an empty registry at capture). When plugin units are present, library
restore clears the registry so leftover bindings cannot outlive the
captured set; the host/CLI rebinds while restoring those units.

#### Errors

Returns when the backup is unsupported, an object is missing/corrupt, or
library replace fails. Destructive SQL starts only after complete preflight.

### restore_backup_in_repo

```rust
pub async fn restore_backup_in_repo(
    db: &DatabaseConnection,
    repo: &BackupRepository,
    id: &str,
    opts: &CanonicalRestoreOpts,
) -> Result<RestorePlan>
```

Restores from an already-open repository (extracted archives).

Target [`crate::SchemaState`](#schemastate) is read from the canonical
`bookclerk_schema_migrations` rows. This function never infers a
versioning contract from [`sea_orm::DbBackend`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/type.DbBackend.html).

#### Errors

Same as [`restore_backup`](#restore_backup).

### apply_binding_bootstrap

```rust
pub async fn apply_binding_bootstrap(db: &DatabaseConnection) -> Result<()>
```

Applies host-owned binding bootstrap when the binding is uninitialized.

Matching [`SchemaState::Unreleased`](#schemastate) is a no-op. Restore that rewrites
`bookclerk_schema_migrations` as ordinary rows will take this no-op path and will not
re-run plugin-owned DDL.

#### Errors

Returns when schema apply or the binding state machine fails closed.

### binding_bootstrap_plan

```rust
pub fn binding_bootstrap_plan(state: &SchemaState) -> Result<Option<Vec<String>>>
```

Statements to apply for `state`, or `None` when the binding already matches.

#### Errors

Returns when `state` is a checksum/base mismatch or an unexpected frozen
binding (this binary has no frozen binding plan).

### db_atomic_operation_id

```rust
pub fn db_atomic_operation_id(op: &DbAtomicParams) -> String
```

Caller-owned idempotency key for an atomic attempt.

Consume-once keys are derived from the operation so an HTTP/RPC retry
resumes the same receipt. Claim redeem includes the session hash; callers
must derive that session from the ticket plus browser nonce (see
[`crate::derive_claim_session_token`](#derive_claim_session_token)) so a new HTTP request after a lost
reply reuses this id. Other ops mint a UUID for this attempt.

### db_atomic_request_hash

```rust
pub fn db_atomic_request_hash(op: &DbAtomicParams) -> Result<String>
```

SHA-256 hex digest of the idempotency-relevant fields of `op`.

Claim redeem omits `expires_at`, client metadata, and the randomized Argon2
`new_password_hash`. Idempotency uses `password_fingerprint` (HMAC of the
plaintext password) so a retry with a fresh Argon2 salt still matches.

#### Errors

Returns an error when the operation fails.

### execute_db_atomic

```rust
pub async fn execute_db_atomic(
    db: &DatabaseConnection,
    compiled: CompiledAtomic,
) -> Result<DbAtomicResult>
```

Runs a compiled named atomic as one native SQL transaction.

#### Errors

Returns [`LibraryError`](#libraryerror) for engine failures. Application outcomes
(`lastOwner`, `empty`, …) are returned as [`DbAtomicResult`](#dbatomicresult), not errors.

### execute_named_atomic

```rust
pub async fn execute_named_atomic(
    db: &DatabaseConnection,
    operation_id: &str,
    params: &DbAtomicParams,
) -> Result<DbAtomicResult>
```

Compiles `params` and runs the plan as one native SQL transaction.

#### Errors

Returns [`LibraryError`](#libraryerror) for engine failures or invalid named operations.

### apply_host_schema

```rust
pub async fn apply_host_schema(db: &DatabaseConnection) -> Result<()>
```

Applies pending host-authored DDL from the canonical migration plan.

In-process adapters (sqlite tests, native postgres tests) infer the engine
from the opened connection. Production plugin-host uses
[`apply_host_schema_with_batch_opts`](#apply_host_schema_with_batch_opts) so adapters lower canonical DDL after
RPC.

#### Errors

Returns [`LibraryError`](#libraryerror) when a version read or DDL statement fails, the
database is newer than this binary, or a frozen checksum does not match.

### apply_host_schema_with_batch

```rust
pub async fn apply_host_schema_with_batch<F, Fut>(
    db: &DatabaseConnection,
    run_batch: F,
) -> Result<()>
where
    F: FnMut(Vec<String>) -> Fut,
    Fut: Future<Output = Result<()>>,
```

Applies schema using `run_batch` (typed `executeAtomic`) for each version.

#### Errors

Returns [`LibraryError`](#libraryerror) when a version read, DDL statement, or batch fails.

### apply_host_schema_with_batch_opts

```rust
pub async fn apply_host_schema_with_batch_opts<F, Fut>(
    db: &DatabaseConnection,
    opts: SchemaApplyOptions,
    run_batch: F,
) -> Result<()>
where
    F: FnMut(Vec<String>) -> Fut,
    Fut: Future<Output = Result<()>>,
```

Applies schema with backups using `run_batch` for each version.

#### Errors

Returns [`LibraryError`](#libraryerror) when a version read, backup, DDL, or batch fails.

### apply_host_schema_with_options

```rust
pub async fn apply_host_schema_with_options(
    db: &DatabaseConnection,
    opts: SchemaApplyOptions,
) -> Result<()>
```

Applies pending host schema with optional in-place backups.

#### Errors

Returns [`LibraryError`](#libraryerror) when a version read, backup, or DDL statement fails.

### current_schema_state

```rust
pub async fn current_schema_state(db: &DatabaseConnection) -> Result<SchemaState>
```

Reads explicit [`SchemaState`](#schemastate) in the Bookclerk host/bootstrap namespace.

An empty `bookclerk_schema_migrations` table is uninitialized; leftover SQLite
`PRAGMA user_version` is ignored.

#### Errors

Returns [`LibraryError::Schema`](#libraryerror) on malformed, partial, or contradictory markers.

### current_schema_state_in

```rust
pub async fn current_schema_state_in(
    db: &DatabaseConnection,
    namespace: &str,
) -> Result<SchemaState>
```

Reads explicit [`SchemaState`](#schemastate) for one ledger `namespace`.

Host library and binding bootstrap use [`BOOKCLERK_SCHEMA_NAMESPACE`](#bookclerk_schema_namespace).
Frozen checksums are verified against [`host_migration_plan`](migrations.md#host_migration_plan) only for
the Bookclerk namespace. Plugin-owned history uses `bookclerk_plugin_migrations`,
not this ledger.

#### Errors

Returns [`LibraryError::Schema`](#libraryerror) on malformed, partial, or contradictory markers.

### current_schema_version

```rust
pub async fn current_schema_version(db: &DatabaseConnection) -> Result<i64>
```

Frozen revision when the database is [`SchemaState::Frozen`](#schemastate); otherwise `0`.

This is **not** a discriminator for uninitialized vs unreleased. Prefer
[`current_schema_state`](#current_schema_state). `0` means “no frozen revision,” never “applied
development schema.”

#### Errors

Returns when the state query fails or the marker is malformed.

### ensure_restore_target_is_replaceable

```rust
pub async fn ensure_restore_target_is_replaceable(db: &DatabaseConnection) -> Result<()>
```

Fails closed when this binary cannot interpret the target database's
schema history well enough to drop every unknown table before restore.

Uninitialized targets are replaceable. Frozen history newer than this
binary, unreleased checksums this binary does not know, and unreadable
markers are not.

#### Errors

Returns [`LibraryError::Schema`](#libraryerror) when the target is newer or uninterpretable.

### migrate_host_schema_to

```rust
pub async fn migrate_host_schema_to(
    db: &DatabaseConnection,
    target: i64,
    opts: SchemaApplyOptions,
) -> Result<SchemaWalk>
```

Migrates toward `target`, including reversible downs for CLI rollback.

#### Errors

Returns [`LibraryError`](#libraryerror) when the walk cannot start, a backup fails, or DDL fails.
A blocked downgrade still applies every reversible step and returns the walk
(the caller treats `blocked` / `stopped_at != target` as failure).

### migrate_host_schema_to_with_batch

```rust
pub async fn migrate_host_schema_to_with_batch<F, Fut>(
    db: &DatabaseConnection,
    target: i64,
    opts: SchemaApplyOptions,
    run_batch: F,
) -> Result<SchemaWalk>
where
    F: FnMut(Vec<String>) -> Fut,
    Fut: Future<Output = Result<()>>,
```

Migrates toward `target` using `run_batch` (typed `executeAtomic`) per version.

A walk that writes anything (backup or DDL) runs under the
[`MIGRATION_LEASE`](control_plane.md#migration_lease) cluster lease, like a start that migrates: it waits
up to [`MIGRATION_LEASE_WAIT`](#migration_lease_wait) for another migrator, renews the lease
while it works, and checks it before every DDL batch. A no-op walk only
checks the database and takes no lease.

#### Errors

Returns [`LibraryError`](#libraryerror) when the walk cannot start, a backup fails, a
batch fails, another migrator holds the lease past the wait, or the lease
is lost mid-walk.

### require_schema_migrations

```rust
pub fn require_schema_migrations(caps: &DbCapabilities) -> Result<()>
```

Requires the `schemaMigrations` versioning contract from typed
[`bookclerk_plugin_abi::DbCapabilities`](../bookclerk_plugin_abi/db_execute.md#dbcapabilities).

The host always records versions in canonical `bookclerk_schema_migrations`
rows; canonical Bookclerk SQL is [`crate::migrations::current_canonical_schema`](migrations.md#current_canonical_schema)
and adapters lower it for the live connection backend at execution. Plugin
identity and diagnostic engine names are not consulted.

#### Errors

Returns [`LibraryError::Other`](#libraryerror) when `schemaMigrations` is missing.

### schema_state_compatible

```rust
pub fn schema_state_compatible(state: &SchemaState) -> bool
```

True when `state` is exactly the schema this binary applies: a start
against it neither migrates nor fails closed.

### apply_plugin_migrations

```rust
pub async fn apply_plugin_migrations(
    db: &DatabaseConnection,
    registered: &PluginMigrationSequence,
) -> Result<PluginMigrationHistory>
```

Applies the pending registered suffix. Idempotent when history already matches.

Outer loop walks durable history (exact-prefix). Each pending migration is
one short atomic unit: serialization-slot mutation + plugin operations +
journal append, with a bounded inner retry loop that does **not** consume
later migration capacity. There is no lock held across those independent
transactions. Concurrent hosts re-read durable history before applying;
uniqueness / conflict / an ambiguous reply causes another journal read.
Matching `(id, checksum)` at the expected ordinal is success.
Contradictory history fails closed.

#### Errors

Returns when prefix verification fails, apply exhausts retries, or the
journal is contradictory.

### history_from_execute_reply

```rust
pub fn history_from_execute_reply(reply: &ExecuteReply) -> Result<PluginMigrationHistory>
```

Parses journal rows from a typed select reply.

#### Errors

Returns when row shape or ordinals are invalid.

### load_plugin_migration_history

```rust
pub async fn load_plugin_migration_history(
    db: &DatabaseConnection,
) -> Result<PluginMigrationHistory>
```

Reads the host-private plugin migration journal.

#### Errors

Returns when the journal cannot be read or ordinals are not contiguous.

### next_pending_plugin_migration

```rust
pub fn next_pending_plugin_migration<'a>(
    history: &PluginMigrationHistory,
    registered: &'a PluginMigrationSequence,
) -> Result<Option<(usize, &'a ProvenPluginMigration)>>
```

Next registered migration after a verified durable prefix, if any.

Uses [`require_history_prefix`](#require_history_prefix) then indexes `registered[history.len()]`
rather than rebuilding the pending suffix vector.

#### Errors

Returns [`require_history_prefix`](#require_history_prefix) errors.

### plugin_apply_statements

```rust
pub fn plugin_apply_statements(
    ordinal: i64,
    migration: &ProvenPluginMigration,
) -> Vec<String>
```

One atomic apply unit: serialization slot, registered ops, host journal append.

### plugin_binding_type_env

```rust
pub fn plugin_binding_type_env(
    registered: &PluginMigrationSequence,
    applied: usize,
) -> SqlTypeEnv
```

Binding catalog after host bootstrap plus schema ops from the first `applied`
registered migrations.

Plugin apply stamps journal DML and suffix data ops against this evolving
environment. `applied` is the durable prefix length (host-private ordinal
count), not a plugin version.

### plugin_history_session_matches

```rust
pub fn plugin_history_session_matches(
    expected_digest: &str,
    observed: &PluginMigrationHistory,
) -> Result<()>
```

True when a binding session's captured history digest still matches durable history.

#### Errors

Returns when the digest differs (peer advanced or rewritten history).

### plugin_journal_has_entry

```rust
pub fn plugin_journal_has_entry(
    history: &PluginMigrationHistory,
    ordinal: i64,
    id: &str,
    checksum: &str,
) -> bool
```

True when `history` already contains this `(ordinal, id, checksum)` row.

### plugin_journal_select_request

```rust
pub fn plugin_journal_select_request(
    operation_id: impl Into<String>,
    deadline_unix_ms: u64,
) -> ExecuteRequest
```

Typed SELECT used by RPC hosts to read the plugin migration journal.

### prove_plugin_migration_sequence

```rust
pub fn prove_plugin_migration_sequence(
    migrations: Vec<PluginMigration>,
) -> Result<PluginMigrationSequence>
```

Proves a complete registration with one evolving [`SqlTypeEnv`](../bookclerk_plugin_abi/sql_types.md#sqltypeenv).

Schema operations update the environment. Data operations typecheck against
the current environment. The environment is not reset between migrations.

#### Errors

Returns when resource limits are exceeded, an id is invalid, ids repeat,
an operation is empty/malformed, or BookclerkSQL proof fails.
Transport/resource validation is
[`bookclerk_plugin_abi::require_plugin_migration_registration`](../bookclerk_plugin_abi/index.md#require_plugin_migration_registration);
this function is the authoritative semantic proof afterward.

### require_history_prefix

```rust
pub fn require_history_prefix(
    history: &PluginMigrationHistory,
    registered: &PluginMigrationSequence,
) -> Result<()>
```

Fail closed unless durable history is an exact prefix of `registered`.

#### Errors

Returns when an applied id/checksum/order does not match, or durable history
is longer than the current registration (older/incompatible plugin).

### sql_string_literal

```rust
pub fn sql_string_literal(value: &str) -> String
```

SQL string literal with `'` doubled.

### catalog_subscribers_for_event

```rust
pub fn catalog_subscribers_for_event(
    catalog: &[EventSubscriberCatalogRecord],
    event: &DomainEventRecord,
) -> Vec<EventSubscriber>
```

Enabled catalog rows that match `event` type, schema version, and filter.

### collapse_live_subscriber_nodes

```rust
pub fn collapse_live_subscriber_nodes(
    rows: &[EventSubscriberNodeRecord],
) -> Vec<EventSubscriberCatalogRecord>
```

Collapse live per-node rows into one catalog record per `plugin_id`.

`enabled` is true when any live row is enabled. Subscriptions are the union
of enabled live rows (disabled nodes do not contribute filters).

### content_kind_from_classic

```rust
pub fn content_kind_from_classic(content_type: i64) -> String
```

Map classic EF `ContentType` to bookclerk `content_kind`.

### content_kind_to_classic

```rust
pub fn content_kind_to_classic(content_kind: &str) -> i32
```

Classic EF `ContentType` enum value.

### event_filter_matches

```rust
pub fn event_filter_matches(filter: Option<&Value>, payload_json: &str) -> bool
```

True when `filter` is absent/empty or every filter key equals the payload object value.

### event_matches_wake_grants

```rust
pub fn event_matches_wake_grants(
    grants_json: &str,
    legacy_filter_json: &str,
    schema_version: i64,
    payload: &str,
) -> bool
```

True when an event matches persisted wake grants, or legacy type+filter rows.

### intersect_event_filters

```rust
pub fn intersect_event_filters(
    declared: Option<&Value>,
    requested: Option<&Value>,
) -> Option<Value>
```

Merge declared and requested payload filters; `None` on a same-key conflict.

### is_downloadable

```rust
pub fn is_downloadable(content_kind: &str) -> bool
```

True when the title can be downloaded (classic `WithoutParents`).

### is_episode

```rust
pub fn is_episode(content_kind: &str) -> bool
```

True when `content_kind` is a podcast episode.

### is_podcast_parent

```rust
pub fn is_podcast_parent(content_kind: &str) -> bool
```

True when `content_kind` is a podcast parent / show (no audio to acquire).

### is_valid_queue_name

```rust
pub fn is_valid_queue_name(name: &str) -> bool
```

True when `name` is `[a-z0-9][a-z0-9._-]*` and at most [`MAX_QUEUE_NAME_LEN`](#max_queue_name_len) chars.

Host-reserved names (`core.events`, `node.<id>`) satisfy this grammar; the
manifest validator, not the outbox, keeps plugins off those prefixes.

### job_backoff_run_after

```rust
pub fn job_backoff_run_after(attempt_count: i64, now: DateTime<Utc>) -> DateTime<Utc>
```

Next `run_after` after a failed attempt, exponential from a 5s base (capped).

### normalize_theme

```rust
pub fn normalize_theme(raw: &str) -> String
```

Canonical appearance preference; unknown values become `system`.

### portal_prefs_key

```rust
pub fn portal_prefs_key(identity_id: i64) -> String
```

Subject key for a portal identity (legacy; prefer [`user_prefs_key`](#user_prefs_key)).

### push_queue_wisher

```rust
pub fn push_queue_wisher(wishers: &mut Vec<QueueWisher>, wisher: QueueWisher)
```

Adds `wisher` when they are not already present (capped at [`MAX_QUEUE_WISHERS`](#max_queue_wishers)).

Duplicate people keep the richer picture and name fields.

#### Arguments

* `wishers` - Accumulated people for this work.
* `wisher` - Person who has this title on an open wishlist.

### queue_name_or_bus

```rust
pub fn queue_name_or_bus(name: &str) -> &str
```

Resolves an optional queue name: empty means [`CORE_EVENTS_QUEUE`](#core_events_queue).

### subscription_matches_event

```rust
pub fn subscription_matches_event(
    sub: &EventCatalogSubscription,
    event: &DomainEventRecord,
) -> bool
```

True when one catalog subscription matches `event` type, schema, class, and filter.

### user_prefs_key

```rust
pub fn user_prefs_key(user_id: i64) -> String
```

Subject key for a first-party user.

### wake_grants_from_subscriptions

```rust
pub fn wake_grants_from_subscriptions(
    subs: &[EventCatalogSubscription],
    wake_type: &str,
    requested_filter_json: &str,
) -> Result<Vec<EventWakeGrant>, &'static str>
```

Build wake grants from declared subscriptions for `wake_type`.

Empty `wake_type` yields no grants (timestamp-only). A type with no matching
subscription returns `"wake event type not subscribed"`. When every matching
subscription conflicts with the requested filter, returns
`"wake filter not within subscription"`.

#### Errors

Returns a static reason when the type is undeclared or the requested filter
cannot intersect any declared filter.

### plan_schema_walk

```rust
pub fn plan_schema_walk(
    plan: &[HostMigrationStep],
    from: i64,
    to: i64,
) -> Result<SchemaWalk>
```

Plans ups or downs from `from` toward `to` using `plan`.

Downgrades walk newest-first and stop at the last reversible version when
the next step has `down: None`. Versions newer than the compiled plan
cannot be reversed by this binary. Versions older than the first frozen
step in `plan` (except empty `from == 0`) cannot be upgraded by this
binary. An empty plan means no frozen schema versions exist.

#### Errors

Returns [`LibraryError::Schema`](#libraryerror) when `to` is below zero, `from` is ahead
of every compiled step (unknown newer schema), or `from` is below the
plan's first retained frozen version.

### plan_schema_walk_from_state

```rust
pub fn plan_schema_walk_from_state(
    plan: &[HostMigrationStep],
    state: &SchemaState,
    to: i64,
) -> Result<SchemaWalk>
```

Plans a frozen walk from an explicit [`SchemaState`](#schemastate).

[`SchemaState::Unreleased`](#schemastate) is never treated as empty `from == 0`. A future
freeze must not apply version-1 ups on top of an unreleased development
database — recreate (`cargo reset --yes`) unless a later explicit promote
path is added.

#### Errors

Returns [`LibraryError::Schema`](#libraryerror) when an unreleased database is asked to
become a frozen revision, or when [`plan_schema_walk`](#plan_schema_walk) fails.

### classify_session_client

```rust
pub fn classify_session_client(
    user_agent: Option<&str>,
    is_api: bool,
) -> SessionClientInfo
```

Classify a User-Agent (or missing UA for API/bearer clients).

#### Arguments

* `user_agent` - Optional `User-Agent` header value.
* `is_api` - When true (no browser UA / bearer-style client), label as API.

#### Panics

Panics when an internal invariant does not hold.

### compile_claim_event_delivery

```rust
pub fn compile_claim_event_delivery(
    operation_id: &str,
    delivery_id: &str,
    owner: &str,
    lease_secs: i64,
    plugin_id: &str,
    resource_class: &str,
    max_in_flight: i64,
    now: &str,
) -> Result<CompiledAtomic, DbErr>
```

Compiles a host-side CAS claim for one pending `event_deliveries` row.

#### Errors

Returns `DbErr::Custom` when wrapping the plan fails.

### compile_named_request

```rust
pub fn compile_named_request(
    operation_id: &str,
    params: &DbAtomicParams,
    now: &str,
) -> Result<CompiledAtomic, DbErr>
```

Compiles a named atomic request into canonical Bookclerk SQL.

Adapters lower placeholders and functions at execute time.

#### Errors

Returns `DbErr::Custom` when the named operation is missing or invalid.

### interpret_typed_exec

```rust
pub fn interpret_typed_exec(
    compiled: &CompiledAtomic,
    reply: &ExecuteReply,
    expected_hash: &str,
) -> DbAtomicResult
```

Maps a typed [`ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#executereply) onto a host [`DbAtomicResult`](#dbatomicresult).

### validate_execute_reply

```rust
pub fn validate_execute_reply(
    req: &ExecuteRequest,
    reply: &ExecuteReply,
    caps: &DbCapabilities,
) -> Result<()>
```

Rejects a guest [`ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#executereply) that does not echo the sent request or
exceeds negotiated caps.

A mismatch after the guest reports success is treated as
[`LibraryError::Unavailable`](#libraryerror) so callers retry the same `operationId`
rather than interpreting a truncated or mismatched envelope.

#### Errors

Returns [`LibraryError::Unavailable`](#libraryerror) when the echo, statement count,
result selection shape, or byte/row bounds do not match the request.

### deliver_after_for

```rust
pub fn deliver_after_for(spec: &PublishDomainEventSpec, now: &str) -> String
```

RFC 3339 earliest delivery time for `spec`; empty when it may deliver now.

### prepare_publish_domain_event

```rust
pub fn prepare_publish_domain_event(
    spec: PublishDomainEventSpec,
) -> Result<PublishDomainEventSpec>
```

Mint an event id when empty and reject oversized or blank publish fields.

#### Errors

Returns [`LibraryError::Other`](#libraryerror) when the payload exceeds 64 KiB, when
`dedup_key` is blank, when a domain event (bus row) has no `event_type`
or a queue message has one, when the queue name or content type is not
recognised, or when `source` is not empty and fails the source grammar.

### fallback_work_key

```rust
pub fn fallback_work_key(
    title: &str,
    authors: Option<&str>,
    asin: Option<&str>,
    isbn: Option<&str>,
) -> String
```

Local fallback when callers do not supply a discover `work_map_key`.

Converts ISBN-10 → ISBN-13 when possible so 10/13 variants share a key.
Soft keys here are a simple lowercase fallback — Discover re-merges with
richer identity matching when ranking the global queue.

### inject_dispatch_page_failures

```rust
pub fn inject_dispatch_page_failures(n: u32)
```

Fail after the next `n` successful dispatch pages (tests only).

### prefer_enrichment_source

```rust
pub fn prefer_enrichment_source(books: &[BookRecord]) -> Option<&BookRecord>
```

Prefer an Audible ownership row with the richest metadata for enrichment.

### set_dispatch_chunk_for_test

```rust
pub fn set_dispatch_chunk_for_test(chunk: Option<usize>)
```

Override subscribers-per-plan for dispatch paging tests.

### wishlist_identities_match

```rust
pub fn wishlist_identities_match(a: WishlistIdentity<'_>, b: WishlistIdentity<'_>) -> bool
```

Whether two wishlist rows refer to the same work despite differing keys.

### decode_html_entities

```rust
pub fn decode_html_entities(raw: &str) -> String
```

Owned decode (allocates even when unchanged — prefer [`decode_html_entities_cow`](#decode_html_entities_cow)
or [`decode_html_entities_in_place`](#decode_html_entities_in_place) on hot paths).

### decode_html_entities_cow

```rust
pub fn decode_html_entities_cow(raw: &str) -> Cow<'_, str>
```

Decode HTML entities in storefront metadata (`Memory&#39;s Blade` → `Memory's Blade`).

Libro.fm (and occasionally other catalogs) emit entity-encoded titles and
descriptions. Decode before display, persistence, and soft identity matching.

Fast path: strings without `&` are returned borrowed (no allocation).

### decode_html_entities_in_place

```rust
pub fn decode_html_entities_in_place(value: &mut String)
```

Decode in place when the value contains `&`; otherwise leave untouched.

### decode_html_entities_opt

```rust
pub fn decode_html_entities_opt(raw: Option<&str>) -> Option<String>
```

Decode when present; leave `None` unchanged.

### decode_html_entities_opt_in_place

```rust
pub fn decode_html_entities_opt_in_place(value: &mut Option<String>)
```

Decode an optional string in place.

### str_maybe_html_entity

```rust
pub fn str_maybe_html_entity(raw: &str) -> bool
```

True when any human-readable field may contain an HTML entity.

### derive_claim_password_fingerprint

```rust
pub fn derive_claim_password_fingerprint(
    dek: &MasterKey,
    nonce: &str,
    password: &str,
) -> String
```

Stable, non-secret fingerprint of an invite/reset password for idempotency.

Argon2id storage hashes use a fresh salt on every POST, so they cannot be
the atomic request hash. This value is
`HMAC-SHA256(subkey, nonce || 0x00 || password)` where
`subkey = HMAC-SHA256(DEK, "bookclerk-claim-password-subkey-v1")`.
The randomized Argon2id hash is still what gets stored on the user row.

### derive_claim_session_token

```rust
pub fn derive_claim_session_token(
    dek: &MasterKey,
    raw_ticket: &str,
    nonce: &str,
) -> String
```

Derive the portal session token for a claim ticket plus browser nonce.

The raw session is
`HMAC-SHA256(DEK, "bookclerk-claim-session-v1:" || ticket || 0x00 || nonce)`
hex-encoded (64 characters). The same ticket, nonce, and process DEK always
yield the same session, so a browser retry after a lost atomic reply
reuses `redeemClaimTicket:{token}:{session}`. A different nonce (another
browser, or the magic link alone) cannot replay that receipt. The plaintext
bearer is never written to a receipt; only [`hash_token`](#hash_token) of this value is
stored.

#### Arguments

* `dek` - Process data-encryption key from [`crate::require_master_key`](master_key.md#require_master_key).
* `raw_ticket` - Plaintext claim ticket from the SPA / share URL.
* `nonce` - Validated 64-hex redeem nonce from [`parse_claim_redeem_nonce`](#parse_claim_redeem_nonce).

### hash_token

```rust
pub fn hash_token(raw: &str) -> String
```

SHA-256 hex digest of a raw ticket/session token for durable storage.

#### Arguments

* `raw` - Opaque plaintext token; callers must not persist this value.

#### Returns

Lowercase hex SHA-256 digest (64 characters).

### parse_claim_redeem_nonce

```rust
pub fn parse_claim_redeem_nonce(raw: &str) -> Result<&str>
```

Accept a browser-generated claim-redeem nonce.

The SPA persists a 32-byte random value as 64 hex characters across HTTP
retries (`sessionStorage`). Possession of the used magic-link URL without
this nonce cannot recover the committed session.

#### Errors

Returns an error when the value is missing, the wrong length, or not hex.

### apply_merged_sources

```rust
pub fn apply_merged_sources(row: &mut TitleRequestRecord)
```

Apply merged bib / editions / hints onto a wishlist row from its source snapshots.

Updates `row` in place from `row.sources`, clearing merged fields when no
sources remain.

#### Arguments

* `row` - Wishlist title-request record whose `sources` snapshots are merged.

### pick_better_description

```rust
pub fn pick_better_description(a: Option<&str>, b: Option<&str>) -> Option<String>
```

Prefer HTML blurbs over plain text; otherwise the longer string wins.

#### Arguments

* `a` - First candidate description (may be HTML or plain).
* `b` - Second candidate description.

#### Returns

The richer non-empty description, or `None` when both are empty.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, LibraryError>;
```

Result alias for [`LibraryError`](#libraryerror).

## Constants

### CHUNK_TARGET_UNCOMPRESSED_BYTES

```rust
pub const CHUNK_TARGET_UNCOMPRESSED_BYTES: usize = _;
```

Target uncompressed JSON size for one table-data chunk.

256 KiB sits in a 128–512 KiB band: large enough to avoid pathological
per-object overhead on typical catalog rows, small enough for a small-VPS
working set and for `maxResultBytes` paging. A single row larger than the
target still occupies its own chunk.

### BACKUPS_DIR

```rust
pub const BACKUPS_DIR: &str = "backups";
```

Subdirectory of `$BOOKCLERK_FILES_DIR` holding the backup repository.

### BACKUP_FORMAT_VERSION

```rust
pub const BACKUP_FORMAT_VERSION: u32 = 1;
```

Manifest format written by this binary.

### BACKUP_RETENTION

```rust
pub const BACKUP_RETENTION: usize = 5;
```

Automatic `pre-migrate` recovery points retained after prune.

### LIBRARY_SKIP_TABLES

```rust
pub const LIBRARY_SKIP_TABLES: &[&str] = _;
```

Library registry rows are environment-local (`unit_ref` is not portable).

### MIGRATION_LEASE_TTL

```rust
pub const MIGRATION_LEASE_TTL: Duration = _;
```

How long a schema migration lease lasts without renewal. The migrator
renews it every third of this while it works.

### MIGRATION_LEASE_WAIT

```rust
pub const MIGRATION_LEASE_WAIT: Duration = _;
```

How long a starter waits for another starter's migration before it fails
closed. Longer than [`MIGRATION_LEASE_TTL`](#migration_lease_ttl), so a migrator that died is
replaced by a waiter rather than failing every waiter.

### BOOKCLERK_SCHEMA_NAMESPACE

```rust
pub const BOOKCLERK_SCHEMA_NAMESPACE: &str = "bookclerk";
```

Ledger namespace for host library schema and Bookclerk-owned binding bootstrap.

### MAX_PLUGIN_MIGRATION_APPLY_ATTEMPTS

```rust
pub const MAX_PLUGIN_MIGRATION_APPLY_ATTEMPTS: usize = 8;
```

Bounded retries for **one** expected plugin migration apply unit.

Independent of how many migrations are registered. A fresh binding may
apply the complete registration; this only caps retries of a single
expected `(ordinal, id, checksum)` while durable history is unchanged.

### CORE_EVENTS_QUEUE

```rust
pub const CORE_EVENTS_QUEUE: &str = "core.events";
```

Queue that carries the domain-event bus (`[events]` producers and consumers).

### EVENT_DELIVERY_MAX_ATTEMPTS

```rust
pub const EVENT_DELIVERY_MAX_ATTEMPTS: i64 = 8;
```

Default max delivery attempts before dead-letter.

### EVENT_RESOURCE_CLASS_NETWORK

```rust
pub const EVENT_RESOURCE_CLASS_NETWORK: &str = "network";
```

Default delivery resource class (`network`).

### EVENT_SUBSCRIBER_HEARTBEAT_TTL_SECS

```rust
pub const EVENT_SUBSCRIBER_HEARTBEAT_TTL_SECS: i64 = 60;
```

Live catalog rows quieter than this many seconds are ignored at dispatch.

### JOB_PAYLOAD_VERSION

```rust
pub const JOB_PAYLOAD_VERSION: u32 = 1;
```

Current job command envelope version stored in [`JobPayload::v`](#jobpayload).

### MAX_QUEUE_NAME_LEN

```rust
pub const MAX_QUEUE_NAME_LEN: usize = 63;
```

Longest queue name.

### MAX_QUEUE_WISHERS

```rust
pub const MAX_QUEUE_WISHERS: usize = 16;
```

Maximum distinct people serialized on a global-queue or Discover card.

### NODE_QUEUE_PREFIX

```rust
pub const NODE_QUEUE_PREFIX: &str = "node.";
```

Prefix of the per-node queues (`node.<node_id>`) node subsystems consume.

### OPERATOR_PREFS_KEY

```rust
pub const OPERATOR_PREFS_KEY: &str = "operator";
```

Subject key for the shared operator account.

### QUEUE_CONTENT_TYPE_BYTES

```rust
pub const QUEUE_CONTENT_TYPE_BYTES: &str = "bytes";
```

Queue message body is raw bytes stored as base64 text.

### QUEUE_CONTENT_TYPE_JSON

```rust
pub const QUEUE_CONTENT_TYPE_JSON: &str = "json";
```

Queue message body is UTF-8 JSON text.

### QUEUE_CONTENT_TYPE_TEXT

```rust
pub const QUEUE_CONTENT_TYPE_TEXT: &str = "text";
```

Queue message body is plain UTF-8 text.

### SCHEMA_STATE_FROZEN

```rust
pub const SCHEMA_STATE_FROZEN: &str = "frozen";
```

On-disk `bookclerk_schema_migrations.state` for a frozen plan version.

### SCHEMA_STATE_UNRELEASED

```rust
pub const SCHEMA_STATE_UNRELEASED: &str = "unreleased";
```

On-disk `bookclerk_schema_migrations.state` for an unreleased development pack.

### KV_LIST_DEFAULT_LIMIT

```rust
pub const KV_LIST_DEFAULT_LIMIT: u32 = 100;
```

Default page size for [`LibraryStore::kv_list`](#librarystorekv_list) when the caller passes 0.

### ACQUIRE_WRITER

```rust
pub const ACQUIRE_WRITER: &str = "acquire";
```

The `writer` of a key an acquire run on this node wrote.

### BOOK_PAGE_DEFAULT_LIMIT

```rust
pub const BOOK_PAGE_DEFAULT_LIMIT: u64 = 40;
```

Default page size for [`LibraryStore::list_books_filtered_page`](#librarystorelist_books_filtered_page).

### BOOK_PAGE_MAX_LIMIT

```rust
pub const BOOK_PAGE_MAX_LIMIT: u64 = 500;
```

Upper clamp for a library book page. Matches the HTTP handler.

### CLAIM_REDEEM_NONCE_HEX_LEN

```rust
pub const CLAIM_REDEEM_NONCE_HEX_LEN: usize = 64;
```

Hex length of a browser-generated claim-redeem nonce (32 bytes).
