**bookclerk_library > models**

# Module: models

## Contents

**Structs**

- [`AccountLinkRecord`](#accountlinkrecord) - Link between a portal identity and a bookstore account.
- [`AccountRecord`](#accountrecord) - Account row stored in the Bookclerk DB.
- [`BookRecord`](#bookrecord) - Book / library item (one ownership row per store product per account).
- [`ClaimTicketRecord`](#claimticketrecord) - Claim ticket metadata (token plaintext is never stored).
- [`DomainEventRecord`](#domaineventrecord) - Immutable outbox envelope.
- [`EmbeddingRecord`](#embeddingrecord) - Stored embedding vector metadata (blob fetched separately when needed).
- [`EnqueueJobSpec`](#enqueuejobspec) - Admission request for `crate::LibraryStore::enqueue_job`.
- [`EventCatalogSubscription`](#eventcatalogsubscription) - One `[capabilities.events.subscriptions]` row persisted in `event_subscriber_nodes`.
- [`EventDeliveryFence`](#eventdeliveryfence) - Claim fence for an event delivery (mirrors [`JobFence`](models.md#bookclerk_librarymodelsjobfence)).
- [`EventDeliveryMetrics`](#eventdeliverymetrics) - Operator-visible delivery-queue counters for `GET /api/status`.
- [`EventDeliveryRecord`](#eventdeliveryrecord) - Per-subscriber delivery row.
- [`EventSubscriber`](#eventsubscriber) - Subscriber snapshot used when creating deliveries for one event.
- [`EventSubscriberCatalogRecord`](#eventsubscribercatalogrecord) - Cluster-authoritative catalog row after collapsing live nodes by `plugin_id`.
- [`EventSubscriberNodeRecord`](#eventsubscribernoderecord) - Live per-node catalog row (`event_subscriber_nodes`).
- [`EventWakeGrant`](#eventwakegrant) - Host-derived wake grant: schema versions plus an intersected payload filter.
- [`GlobalQueueEntry`](#globalqueueentry) - Aggregated global request-queue entry (one work, many wishers).
- [`JobClaimCursor`](#jobclaimcursor) - Where the next page of routed-claim candidates starts (keyset paging).
- [`JobClaimPage`](#jobclaimpage) - One page of routed-claim candidates.
- [`JobFence`](#jobfence) - Lease identity a worker must present to mutate a running job.
- [`JobPayload`](#jobpayload) - JSON command envelope stored on a job row.
- [`JobRecord`](#jobrecord) - Durable job row returned to the daemon API and worker.
- [`JobTempPath`](#jobtemppath) - Scratch path registered against a job for crash cleanup.
- [`ListeningProgressRecord`](#listeningprogressrecord) - Listening progress snapshot from an external player (e.g. AudioBookshelf).
- [`OidcClientRecord`](#oidcclientrecord) - Registered OIDC authorization-server client (Bookclerk as IdP).
- [`OperatorSessionRecord`](#operatorsessionrecord) - Durable operator session metadata (hashed token is never exposed).
- [`PendingWakeProgress`](#pendingwakeprogress) - How far one claimed wake tick got.
- [`PortalIdentity`](#portalidentity) - Portal identity bound to an external provider user (e.g. ABS).
- [`PortalSessionRecord`](#portalsessionrecord) - Portal session row for session lists (includes hash for current-session match).
- [`PublishDomainEventSpec`](#publishdomaineventspec) - Producer-side domain event (or queue message) to persist in the outbox.
- [`QueueWisher`](#queuewisher) - Compact wisher shown on the global queue (avatar + label).
- [`SecurityAuditEvent`](#securityauditevent) - Security audit event (elevate / impersonate / login / provision).
- [`StoredPasskey`](#storedpasskey) - Stored WebAuthn credential (passkey JSON plus the label shown in Settings).
- [`TitleRequestRecord`](#titlerequestrecord) - Personal wishlist row (also contributes to the shared global queue while open).
- [`TitleRequestSourceRecord`](#titlerequestsourcerecord) - Per-storefront catalog/pricing snapshot attached to a wishlist row.
- [`UserIntegrationHint`](#userintegrationhint) - Storefront / integration connection summary for admin user lists.
- [`UserInviteRecord`](#userinviterecord) - Invite ticket for provisioning a User (token plaintext never stored).
- [`UserListeningHint`](#userlisteninghint) - Recent unfinished listening hint for admin user lists.
- [`UserPreferences`](#userpreferences) - Per-user GUI / Discover preferences (operator or portal identity).
- [`UserPresenceExtras`](#userpresenceextras) - Presence + connection extras for administrator user management.
- [`UserRecord`](#userrecord) - First-party Bookclerk user (security principal for portal paths).
- [`UserSsoPicture`](#userssopicture) - IdP-supplied avatar available as a profile-picture choice.
- [`WishlistPurchaseHint`](#wishlistpurchasehint) - Snapshotted purchase link/price for a wishlist title.
- [`WishlistStoreEdition`](#wishliststoreedition) - Storefront edition key for wishlist / queue payloads.
- [`WorkRecord`](#workrecord) - Canonical work spanning one or more ownership rows / editions.

**Enums**

- [`AcquireStatus`](#acquirestatus) - Acquire / download state for a title.
- [`EnqueueOutcome`](#enqueueoutcome) - Result of admitting a job into the durable queue.
- [`JobKind`](#jobkind) - Kind of durable daemon work stored in the `jobs` table.
- [`JobResourceClass`](#jobresourceclass) - Concurrency class that bounds how many jobs of this type run at once.
- [`JobState`](#jobstate) - Lifecycle state of a durable job row.
- [`JobTrigger`](#jobtrigger) - Who admitted the job (API vs periodic scheduler).
- [`PublishDomainEventOutcome`](#publishdomaineventoutcome) - Result of `crate::LibraryStore::publish_domain_event`.
- [`RequestStatus`](#requeststatus) - Wishlist row status (`open` while wishlisted; `cancelled` after un-wishlist).
- [`UserRole`](#userrole) - First-party user role (`owner` | `administrator` | `host` | `member`).
- [`UserStatus`](#userstatus) - First-party user status.

**Functions**

- [`catalog_subscribers_for_event`](#catalog_subscribers_for_event) - Enabled catalog rows that match `event` type, schema version, and filter.
- [`collapse_live_subscriber_nodes`](#collapse_live_subscriber_nodes) - Collapse live per-node rows into one catalog record per `plugin_id`.
- [`content_kind_from_classic`](#content_kind_from_classic) - Map classic EF `ContentType` to bookclerk `content_kind`.
- [`content_kind_to_classic`](#content_kind_to_classic) - Classic EF `ContentType` enum value.
- [`event_filter_matches`](#event_filter_matches) - True when `filter` is absent/empty or every filter key equals the payload object value.
- [`event_matches_wake_grants`](#event_matches_wake_grants) - True when an event matches persisted wake grants, or legacy type+filter rows.
- [`intersect_event_filters`](#intersect_event_filters) - Merge declared and requested payload filters; `None` on a same-key conflict.
- [`is_downloadable`](#is_downloadable) - True when the title can be downloaded (classic `WithoutParents`).
- [`is_episode`](#is_episode) - True when `content_kind` is a podcast episode.
- [`is_podcast_parent`](#is_podcast_parent) - True when `content_kind` is a podcast parent / show (no audio to acquire).
- [`is_valid_queue_name`](#is_valid_queue_name) - True when `name` is `[a-z0-9][a-z0-9._-]*` and at most [`MAX_QUEUE_NAME_LEN`](models.md#bookclerk_librarymodelsmax_queue_name_len) chars.
- [`job_backoff_run_after`](#job_backoff_run_after) - Next `run_after` after a failed attempt, exponential from a 5s base (capped).
- [`normalize_theme`](#normalize_theme) - Canonical appearance preference; unknown values become `system`.
- [`portal_prefs_key`](#portal_prefs_key) - Subject key for a portal identity (legacy; prefer [`user_prefs_key`](models.md#bookclerk_librarymodelsuser_prefs_key)).
- [`push_queue_wisher`](#push_queue_wisher) - Adds `wisher` when they are not already present (capped at [`MAX_QUEUE_WISHERS`](models.md#bookclerk_librarymodelsmax_queue_wishers)).
- [`queue_name_or_bus`](#queue_name_or_bus) - Resolves an optional queue name: empty means [`CORE_EVENTS_QUEUE`](models.md#bookclerk_librarymodelscore_events_queue).
- [`subscription_matches_event`](#subscription_matches_event) - True when one catalog subscription matches `event` type, schema, class, and filter.
- [`user_prefs_key`](#user_prefs_key) - Subject key for a first-party user.
- [`wake_grants_from_subscriptions`](#wake_grants_from_subscriptions) - Build wake grants from declared subscriptions for `wake_type`.

**Constants**

- [`CORE_EVENTS_QUEUE`](#core_events_queue) - Queue that carries the domain-event bus (`[events]` producers and consumers).
- [`EVENT_DELIVERY_MAX_ATTEMPTS`](#event_delivery_max_attempts) - Default max delivery attempts before dead-letter.
- [`EVENT_RESOURCE_CLASS_NETWORK`](#event_resource_class_network) - Default delivery resource class (`network`).
- [`EVENT_SUBSCRIBER_HEARTBEAT_TTL_SECS`](#event_subscriber_heartbeat_ttl_secs) - Live catalog rows quieter than this many seconds are ignored at dispatch.
- [`JOB_PAYLOAD_VERSION`](#job_payload_version) - Current job command envelope version stored in `JobPayload::v`.
- [`MAX_QUEUE_NAME_LEN`](#max_queue_name_len) - Longest queue name.
- [`MAX_QUEUE_WISHERS`](#max_queue_wishers) - Maximum distinct people serialized on a global-queue or Discover card.
- [`NODE_QUEUE_PREFIX`](#node_queue_prefix) - Prefix of the per-node queues (`node.<node_id>`) node subsystems consume.
- [`OPERATOR_PREFS_KEY`](#operator_prefs_key) - Subject key for the shared operator account.
- [`QUEUE_CONTENT_TYPE_BYTES`](#queue_content_type_bytes) - Queue message body is raw bytes stored as base64 text.
- [`QUEUE_CONTENT_TYPE_JSON`](#queue_content_type_json) - Queue message body is UTF-8 JSON text.
- [`QUEUE_CONTENT_TYPE_TEXT`](#queue_content_type_text) - Queue message body is plain UTF-8 text.

---

## bookclerk_library::models::AccountLinkRecord

*Struct*

Link between a portal identity and a bookstore account.

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `identity_id: i64` - Foreign key to `portal_identities.id`.
- `plugin_key: String` - Provenance-qualified PluginKey that owns the account.
- `account_id: String` - Store or operator account id this row belongs to.
- `source: String` - Display alias of the owning plugin (`audible`, `libro`, …).
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AccountLinkRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::AccountRecord

*Struct*

Account row stored in the Bookclerk DB.

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `plugin_key: String` - Provenance-qualified PluginKey that owns this account. Any instance
- `account_id: String` - Store or operator account id, unique within one plugin key.
- `source: String` - Display alias of the owning plugin (`audible`, `libro`, …).
- `marketplace: String` - Store marketplace / locale code (for example `us`, `uk`).
- `label: Option<String>` - Optional operator-facing display label.
- `scan_enabled: bool` - When nonzero/true, scheduled scans include this account.
- `connection_status: String` - `active` or `revoked` (credentials removed; books retained).
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.
- `updated_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was last modified.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AccountRecord`



## bookclerk_library::models::AcquireStatus

*Enum*

Acquire / download state for a title.

**Variants:**
- `NotAcquired` - Title has not been queued or downloaded yet.
- `Queued` - Title is waiting in the acquire queue.
- `Downloading` - Acquire pipeline is actively downloading this title.
- `Acquired` - Primary audio artifact is stored and library status is liberated.
- `Error` - Last acquire attempt failed; see `error_message` on the book row.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the canonical snake_case / lowercase wire string.
- `fn parse(s: &str) -> Option<Self>` - Parses the canonical wire string; returns `None` when unknown.
- `fn to_classic(self: Self) -> i32` - Classic EF `LiberatedStatus` integer.
- `fn from_classic(status: i64) -> Self` - Parse classic EF `LiberatedStatus` integer.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &AcquireStatus) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AcquireStatus`
- **Default**
  - `fn default() -> AcquireStatus`



## bookclerk_library::models::BookRecord

*Struct*

Book / library item (one ownership row per store product per account).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `uuid: String` - Stable UUID for this row (API / foreign-key identity).
- `plugin_key: String` - Provenance-qualified PluginKey that owns the account.
- `source: String` - Display alias of the owning plugin (`audible`, `libro`, …).
- `account_id: String` - Store or operator account id this row belongs to.
- `product_id: String` - Storefront product id (ASIN, ISBN, UUID, …).
- `asin: Option<String>` - Audible ASIN when known (`None` for Libro-only rows without enrichment).
- `isbn: Option<String>` - ISBN-13 when known (Libro always; Audible when the API provides it).
- `marketplace: String` - Store marketplace / locale code (for example `us`, `uk`).
- `title: String` - Display title of the work or edition.
- `authors: Option<String>` - Comma-separated or JSON author list from the storefront.
- `narrators: Option<String>` - Comma-separated or JSON narrator list when present.
- `series: Option<String>` - Series name when the title belongs to a series.
- `series_index: Option<String>` - Position within the series (storefront string form).
- `series_asin: Option<String>` - Audible series / podcast-parent ASIN (`Series.AudibleSeriesId`).
- `acquire_status: AcquireStatus` - Download/acquire pipeline state (`not_acquired`, `queued`, …).
- `storage_key: Option<String>` - Object-storage key for the primary audio artifact, if acquired.
- `error_message: Option<String>` - Last acquire/convert failure message for operators.
- `purchased_at: Option<chrono::DateTime<chrono::Utc>>` - RFC 3339 purchase time from the storefront, when known.
- `tags: Option<String>` - Operator or storefront tags (serialized string).
- `rating_overall: Option<f32>` - Overall user rating from the storefront, if any.
- `rating_performance: Option<f32>` - Narration/performance rating from the storefront, if any.
- `rating_story: Option<f32>` - Story rating from the storefront, if any.
- `is_finished: bool` - Whether the listener marked the title finished (0/1 or bool).
- `pdf_status: AcquireStatus` - Companion PDF acquire state (`not_acquired`, `acquired`, …).
- `pdf_storage_key: Option<String>` - Object-storage key for the companion PDF, if present.
- `publisher: Option<String>` - Publisher name from metadata enrichment or the storefront.
- `length_minutes: Option<i64>` - Runtime in whole minutes when the storefront reports it.
- `is_abridged: bool` - Whether the edition is abridged (0/1 or bool).
- `content_kind: String` - Title kind: `book`, `episode`, `podcast`, ….
- `categories: Option<String>` - Storefront category / genre path list (serialized).
- `subtitle: Option<String>` - Optional subtitle from bibliographic metadata.
- `published_at: Option<chrono::DateTime<chrono::Utc>>` - Publication date string from the storefront or enrichment.
- `description: Option<String>` - Blurb / synopsis text (may contain HTML).
- `language: Option<String>` - BCP-47 or storefront language code when known.
- `cover_url: Option<String>` - HTTPS URL for cover art when known.
- `subjects: Option<String>` - Subject / topic tags (often from Open Library; `;`- or `,`-separated).
- `enrich_source: Option<String>` - Plugin or catalog that last enriched bibliographic fields.
- `enrich_confidence: Option<f64>` - 0–1 confidence score for the last enrichment pass.
- `enrich_updated_at: Option<chrono::DateTime<chrono::Utc>>` - RFC 3339 time of the last enrichment write.
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.
- `updated_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was last modified.

**Methods:**

- `fn title_id(self: &Self) -> &str` - Public stable id used for CLI / API / acquire lookups.
- `fn asin_or_isbn(self: &Self) -> &str` - Naming / display fallback: ASIN, else ISBN, else source product id.
- `fn download_product_id(self: &Self) -> &str` - Source-native id for download / fetch APIs (`product_id`).
- `fn audible_asin(self: &Self) -> Option<&str>` - Audible ASIN for Audible license / catalog APIs when one is known.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BookRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::CORE_EVENTS_QUEUE

*Constant*: `&str`

Queue that carries the domain-event bus (`[events]` producers and consumers).



## bookclerk_library::models::ClaimTicketRecord

*Struct*

Claim ticket metadata (token plaintext is never stored).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `token_hash: String` - SHA-256 hex digest of the opaque token (plaintext never stored).
- `identity_id: Option<i64>` - Foreign key to `portal_identities.id`.
- `expires_at: chrono::DateTime<chrono::Utc>` - RFC 3339 expiry for the ticket, session, or code.
- `redeemed_at: Option<chrono::DateTime<chrono::Utc>>` - RFC 3339 time when the ticket/invite was redeemed, if any.
- `created_by: String` - Actor that created the row (user id or operator label).
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ClaimTicketRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::models::DomainEventRecord

*Struct*

Immutable outbox envelope.

**Fields:**
- `id: String` - Event id.
- `event_type: String` - Event type.
- `schema_version: i64` - Payload schema version.
- `occurred_at: chrono::DateTime<chrono::Utc>` - Occurrence time.
- `account_id: String` - Tenant / account id.
- `source: String` - Producer plugin id (`audible`, …); empty when unknown.
- `correlation_id: String` - Trace correlation id.
- `causation_id: String` - Causing id.
- `dedup_key: String` - Producer idempotency key.
- `payload: String` - Bounded JSON payload.
- `ordering_key: String` - Producer FIFO key copied onto each delivery.
- `dispatch_state: String` - `pending` or `dispatched`.
- `created_at: chrono::DateTime<chrono::Utc>` - Insert time.
- `wake_pending: bool` - True until a wake pass for this event has completed.
- `queue_name: String` - Queue this row belongs to (`core.events` for domain events).
- `content_type: String` - Body encoding of `payload` (`json`, `text`, `bytes`).
- `deliver_after: Option<chrono::DateTime<chrono::Utc>>` - Earliest delivery time for a delayed queue message.

**Methods:**

- `fn is_queue_message(self: &Self) -> bool` - True for a plugin-owned queue message (not a domain event).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DomainEventRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::EVENT_DELIVERY_MAX_ATTEMPTS

*Constant*: `i64`

Default max delivery attempts before dead-letter.



## bookclerk_library::models::EVENT_RESOURCE_CLASS_NETWORK

*Constant*: `&str`

Default delivery resource class (`network`).



## bookclerk_library::models::EVENT_SUBSCRIBER_HEARTBEAT_TTL_SECS

*Constant*: `i64`

Live catalog rows quieter than this many seconds are ignored at dispatch.



## bookclerk_library::models::EmbeddingRecord

*Struct*

Stored embedding vector metadata (blob fetched separately when needed).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `target_kind: String` - Embedding target kind (`book`, `work`, …).
- `target_id: String` - Id of the embedded target within `target_kind`.
- `model: String` - Embedding model identifier used to produce `vector`.
- `dims: i64` - Dimensionality of the stored embedding vector.
- `text_hash: String` - Hash of the text that was embedded (skip re-embed when unchanged).
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.
- `updated_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was last modified.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EmbeddingRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::EnqueueJobSpec

*Struct*

Admission request for `crate::LibraryStore::enqueue_job`.

**Fields:**
- `kind: JobKind` - Job kind to run.
- `payload: JobPayload` - Kind-specific filters and trigger metadata.
- `priority: i64` - Higher values are claimed first (default 0).
- `max_attempts: i64` - Maximum claims before a failure is terminal.
- `max_pending: i64` - Global cap on pending+running rows; enqueue fails when at the cap.
- `run_after: Option<chrono::DateTime<chrono::Utc>>` - Optional delay before the job becomes claimable.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EnqueueJobSpec`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::models::EnqueueOutcome

*Enum*

Result of admitting a job into the durable queue.

**Variants:**
- `Created{ id: String }` - A new row was inserted.
- `Duplicate{ existing_id: String }` - An equivalent pending/running job already exists.
- `QueueFull` - `pending + running` already equals `max_pending`.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &EnqueueOutcome) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> EnqueueOutcome`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::models::EventCatalogSubscription

*Struct*

One `[capabilities.events.subscriptions]` row persisted in `event_subscriber_nodes`.

**Fields:**
- `event_type: String` - Versioned event type (`book_acquired`, …).
- `schema_versions: Vec<u32>` - Schema versions this guest can consume (default `[1]`).
- `supports_suspend: bool` - Whether `EventResult::Suspended` is supported for this type.
- `resource_class: String` - Concurrency class (default `"network"`).
- `filter: Option<serde_json::Value>` - Optional host-owned payload object filter (top-level key equality).
- `queue: String` - Queue this subscription consumes; empty means the domain-event bus.
- `max_retries: Option<u32>` - Redeliveries before dead-lettering; `None` uses the host default.
- `dead_letter_queue: Option<String>` - Queue that receives a message once `max_retries` is exhausted.
- `retry_delay_secs: u32` - Seconds a retried message waits before redelivery.

**Methods:**

- `fn new<impl Into<String>>(event_type: impl Trait, schema_versions: Vec<u32>) -> Self` - Build a type+schema subscription with default resource class and no filter.
- `fn queue<impl Into<String>>(queue: impl Trait) -> Self` - Build a plugin-owned queue subscription (`[[queues.consumers]]`).
- `fn queue_name(self: &Self) -> &str` - Queue this subscription consumes (`core.events` when unset).
- `fn is_queue(self: &Self) -> bool` - True for a plugin-owned queue subscription (not the domain-event bus).
- `fn max_attempts(self: &Self) -> Option<u32>` - `max_attempts` for deliveries created from this subscription.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventCatalogSubscription) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> EventCatalogSubscription`



## bookclerk_library::models::EventDeliveryFence

*Struct*

Claim fence for an event delivery (mirrors [`JobFence`](models.md#bookclerk_librarymodelsjobfence)).

**Fields:**
- `delivery_id: String` - Delivery id that was claimed.
- `owner: String` - Worker id stored in `lease_owner`.
- `generation: i64` - `lease_generation` assigned by the successful claim.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventDeliveryFence`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::EventDeliveryMetrics

*Struct*

Operator-visible delivery-queue counters for `GET /api/status`.

**Fields:**
- `pending: i64` - `pending` rows with no checkpoint (not suspended).
- `running: i64` - Currently claimed rows.
- `suspended: i64` - `pending` rows that hold a suspend checkpoint.
- `dead_letter: i64` - Dead-lettered rows.
- `acked: i64` - Successfully acked rows.
- `oldest_pending_age_secs: Option<i64>` - Age in seconds of the oldest `pending` row, when any exist.
- `retries_total: i64` - Durable retry outcomes (including reclaim-as-retry).
- `suspensions_total: i64` - Durable suspend outcomes.
- `dead_letters_total: i64` - Durable transitions into `dead_letter`.
- `dispatch_latency_ms_avg: Option<i64>` - Average first-dispatch latency in milliseconds, when sampled.
- `handler_latency_ms_avg: Option<i64>` - Average `onEvent` handler duration in milliseconds, when sampled.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventDeliveryMetrics`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventDeliveryMetrics) -> bool`
- **Default**
  - `fn default() -> EventDeliveryMetrics`



## bookclerk_library::models::EventDeliveryRecord

*Struct*

Per-subscriber delivery row.

**Fields:**
- `id: String` - Delivery id (`{event_id}:{plugin_id}`).
- `event_id: String` - Parent event id.
- `plugin_id: String` - Subscriber plugin id.
- `idempotency_key: String` - Stable idempotency key.
- `state: String` - Lifecycle state.
- `attempt_count: i64` - Claim count (not incremented on resume).
- `max_attempts: i64` - Max claims before dead-letter.
- `lease_owner: Option<String>` - Current lease owner.
- `lease_expires_at: Option<chrono::DateTime<chrono::Utc>>` - Lease expiry.
- `lease_generation: i64` - Fence generation.
- `run_after: chrono::DateTime<chrono::Utc>` - Earliest claim time.
- `invocation_sequence: i64` - Resume ordinal.
- `resume_pending: bool` - Next claim should resume rather than increment attempt.
- `checkpoint_json: Option<String>` - Suspend checkpoint JSON.
- `checkpoint_schema_version: i64` - Checkpoint schema version.
- `ordering_key: String` - FIFO key.
- `outcome: Option<String>` - Terminal outcome.
- `error_message: Option<String>` - Operator-facing error.
- `created_at: chrono::DateTime<chrono::Utc>` - Insert time.
- `updated_at: chrono::DateTime<chrono::Utc>` - Last-modified time.
- `cancel_requested: bool` - Cooperative cancel flag for a running delivery.
- `resource_class: String` - Concurrency class (`network` today).
- `wake_event_type: String` - Event type that can wake this suspended delivery; empty = timestamp-only.
- `wake_filter_json: String` - Host-owned payload object filter JSON for event-wake; empty = type only.
- `wake_grants_json: String` - Host-derived wake grants JSON; empty falls back to type + `wake_filter_json`.
- `queue_name: String` - Queue of the parent event, copied at dispatch.

**Methods:**

- `fn fence(self: &Self) -> EventDeliveryFence` - Fence for heartbeat / complete / fail / suspend.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventDeliveryRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::EventSubscriber

*Struct*

Subscriber snapshot used when creating deliveries for one event.

**Fields:**
- `plugin_id: String` - Plugin id that declared a matching subscription.
- `resource_class: String` - Concurrency class copied onto the delivery row.
- `max_attempts: Option<u32>` - Claims before the delivery dead-letters; `None` uses the host default

**Methods:**

- `fn plugin<impl Into<String>>(plugin_id: impl Trait) -> Self` - Catalog match for `plugin_id` with the default `network` resource class.
- `fn max_attempts_or_default(self: &Self) -> i64` - Effective `max_attempts` for a new delivery row.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventSubscriber) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> EventSubscriber`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::models::EventSubscriberCatalogRecord

*Struct*

Cluster-authoritative catalog row after collapsing live nodes by `plugin_id`.

**Fields:**
- `plugin_id: String` - Canonical PluginKey of the subscriber (not the display alias).
- `subscriptions: Vec<EventCatalogSubscription>` - Union of subscriptions from live enabled nodes.
- `enabled: bool` - Whether any live node has this plugin enabled.
- `updated_at: chrono::DateTime<chrono::Utc>` - Latest heartbeat among contributing live rows.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventSubscriberCatalogRecord) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> EventSubscriberCatalogRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::models::EventSubscriberNodeRecord

*Struct*

Live per-node catalog row (`event_subscriber_nodes`).

**Fields:**
- `node_id: String` - Daemon node id.
- `plugin_id: String` - Canonical PluginKey of the subscriber (not the display alias).
- `subscriptions: Vec<EventCatalogSubscription>` - Declared subscriptions on this node.
- `enabled: bool` - Whether this node wants matching events for `plugin_id`.
- `heartbeat_at: chrono::DateTime<chrono::Utc>` - Last heartbeat from this node.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventSubscriberNodeRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventSubscriberNodeRecord) -> bool`



## bookclerk_library::models::EventWakeGrant

*Struct*

Host-derived wake grant: schema versions plus an intersected payload filter.

**Fields:**
- `schema_versions: Vec<u32>` - Schema versions this grant may observe.
- `filter: Option<serde_json::Value>` - Intersected payload object filter; `None` / empty object matches any payload.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventWakeGrant) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> EventWakeGrant`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::models::GlobalQueueEntry

*Struct*

Aggregated global request-queue entry (one work, many wishers).

**Fields:**
- `work_key: String` - Stable merge key used to group editions into a work.
- `title: String` - Display title of the work or edition.
- `authors: Option<String>` - Comma-separated or JSON author list from the storefront.
- `asin: Option<String>` - Amazon ASIN when known; otherwise null.
- `isbn: Option<String>` - ISBN-10/13 when known; otherwise null.
- `cover_url: Option<String>` - HTTPS URL for cover art when known.
- `description: Option<String>` - Blurb / synopsis text (may contain HTML).
- `subtitle: Option<String>` - Optional subtitle from bibliographic metadata.
- `narrators: Option<String>` - Comma-separated or JSON narrator list when present.
- `series: Option<String>` - Series name when the title belongs to a series.
- `series_index: Option<String>` - Position within the series (storefront string form).
- `publisher: Option<String>` - Publisher name from metadata enrichment or the storefront.
- `length_minutes: Option<i64>` - Runtime in whole minutes when the storefront reports it.
- `published_at: Option<String>` - Publication date string from the storefront or enrichment.
- `genres: Option<String>` - Merged genre / category list for display.
- `language: Option<String>` - BCP-47 or storefront language code when known.
- `store_editions: Vec<WishlistStoreEdition>` - Distinct storefront editions contributing to this wishlist item.
- `purchase_hints: Vec<WishlistPurchaseHint>` - Purchase URLs and prices observed per storefront.
- `wish_count: i64` - Number of users who requested this title.
- `wishers: Vec<QueueWisher>` - Distinct people who have this title on an open wishlist.
- `sample_uuids: Vec<String>` - Sample title-request UUIDs represented in this aggregate row.
- `first_requested_at: chrono::DateTime<chrono::Utc>` - Earliest request timestamp in this aggregate group.
- `last_requested_at: chrono::DateTime<chrono::Utc>` - Most recent request timestamp in this aggregate group.

**Methods:**

- `fn push_wisher(self: & mut Self, wisher: QueueWisher)` - Adds `wisher` when they are not already present (capped at 16).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> GlobalQueueEntry`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::JOB_PAYLOAD_VERSION

*Constant*: `u32`

Current job command envelope version stored in `JobPayload::v`.



## bookclerk_library::models::JobClaimCursor

*Struct*

Where the next page of routed-claim candidates starts (keyset paging).

Candidates are ordered by `priority DESC, created_at ASC, id ASC`; the
cursor is the last row of the previous page, so rows that leave the
pending state between pages never shift the window.

**Fields:**
- `priority: i64` - Priority of the last row read.
- `created_at: String` - `created_at` (RFC 3339 text) of the last row read.
- `id: String` - Id of the last row read.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobClaimCursor`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobClaimCursor) -> bool`



## bookclerk_library::models::JobClaimPage

*Struct*

One page of routed-claim candidates.

**Fields:**
- `jobs: Vec<JobRecord>` - Readable, not cancel-requested pending jobs, best first.
- `next: Option<JobClaimCursor>` - Where the next page starts, or `None` when no row remains.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> JobClaimPage`
- **Clone**
  - `fn clone(self: &Self) -> JobClaimPage`



## bookclerk_library::models::JobFence

*Struct*

Lease identity a worker must present to mutate a running job.

**Fields:**
- `job_id: String` - Job id that was claimed.
- `owner: String` - Worker id stored in `lease_owner`.
- `generation: i64` - `lease_generation` assigned by the successful claim.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobFence`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobFence) -> bool`



## bookclerk_library::models::JobKind

*Enum*

Kind of durable daemon work stored in the `jobs` table.

**Variants:**
- `Scan` - Library scan for one account or every scan-enabled account.
- `Acquire` - Acquire pending titles (optional title / account filter).
- `ListenSync` - Sync listening progress from capable integrations.
- `IntegrationScan` - Remote integration library scan (Audiobookshelf, …).
- `PluginCopy` - ABI `JobRunner` stream-copy vertical slice (`plugin_copy`).
- `Invalid` - Terminal placeholder after an unreadable persisted command is rejected.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the canonical snake_case wire string.
- `fn parse(s: &str) -> Option<Self>` - Parses the canonical wire string; returns `None` when unknown.
- `fn resource_class(self: Self) -> JobResourceClass` - Resource class used for admission and worker concurrency.
- `fn dedup_key(self: Self, payload: &JobPayload) -> String` - Idempotency key unique among pending/running jobs of the same work.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> JobKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobKind) -> bool`



## bookclerk_library::models::JobPayload

*Struct*

JSON command envelope stored on a job row.

**Fields:**
- `v: u32` - Envelope schema version; must equal [`JOB_PAYLOAD_VERSION`](models.md#bookclerk_librarymodelsjob_payload_version).
- `account: Option<String>` - Optional account filter (`None` = all eligible accounts).
- `title: Option<String>` - Optional title filter (uuid / ASIN / ISBN / product id).
- `plugin_instance_id: Option<String>` - Pins the plugin instance that serves the job. `None` routes to the
- `requires: Vec<String>` - Plugin instance ids the claiming node must run, with its guest up.
- `trigger: JobTrigger` - Admission source (`api` or `scheduler`).
- `integration_id: Option<String>` - Integration plugin id for `JobKind::IntegrationScan`.
- `force: bool` - When true, an integration scan asks the remote to rescan, and an
- `plugin_id: Option<String>` - Destination plugin id for `JobKind::PluginCopy`.
- `source_key: Option<String>` - Source object key for `JobKind::PluginCopy`.
- `dest_key: Option<String>` - Destination object key for `JobKind::PluginCopy`.
- `checkpoint: Option<bookclerk_plugin_abi::JobCheckpoint>` - Bounded checkpoint restored after `bookclerk_plugin_abi::JobOutcome::Suspended`.
- `invocation_sequence: Option<u32>` - Resume ordinal; distinct from failure [`Self`](models.md#bookclerk_librarymodelsjobpayload) attempt_count on the job row.
- `resume_pending: bool` - One-shot: the next claim is a suspend-resume and must not consume a failure attempt.
- `pdf_only: bool` - For [`JobKind::Acquire`]: fetch only each title's companion PDF.
- `requires_unsettled_since_ms: Option<i64>` - Unix milliseconds when the requirements sweep first found this pending

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobPayload) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> JobPayload`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> Self`



## bookclerk_library::models::JobRecord

*Struct*

Durable job row returned to the daemon API and worker.

**Fields:**
- `id: String` - Stable job id (`scan-{uuid}`, …).
- `kind: JobKind` - Job kind (`scan`, `acquire`, `listen_sync`).
- `state: JobState` - Lifecycle state.
- `priority: i64` - Higher values are claimed first.
- `resource_class: JobResourceClass` - Concurrency class.
- `payload: JobPayload` - Kind-specific filters and trigger metadata.
- `progress: Option<String>` - Human-readable progress string.
- `attempt_count: i64` - Number of times a worker has claimed this row.
- `max_attempts: i64` - Maximum claims before a failure becomes terminal.
- `run_after: chrono::DateTime<chrono::Utc>` - Earliest time a pending job may be claimed.
- `lease_owner: Option<String>` - Worker id that currently holds the lease.
- `lease_expires_at: Option<chrono::DateTime<chrono::Utc>>` - Lease expiry; stale running rows are reclaimed after this.
- `dedup_key: String` - Idempotency key among pending/running rows.
- `error_kind: Option<String>` - Structured error kind when failed or cancelled.
- `error_message: Option<String>` - Operator-facing error text.
- `cancel_requested: bool` - Cooperative cancel flag.
- `created_at: chrono::DateTime<chrono::Utc>` - When the row was inserted.
- `updated_at: chrono::DateTime<chrono::Utc>` - When the row was last modified.
- `started_at: Option<chrono::DateTime<chrono::Utc>>` - When a worker first claimed the row.
- `finished_at: Option<chrono::DateTime<chrono::Utc>>` - When the row reached a terminal state.
- `lease_generation: i64` - Per-claim generation used to fence heartbeat and finalization.

**Methods:**

- `fn fence(self: &Self) -> Option<JobFence>` - Lease identity for heartbeat and finalization, when this row is claimed.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::JobResourceClass

*Enum*

Concurrency class that bounds how many jobs of this type run at once.

**Variants:**
- `Network` - Store HTTP / scan / acquire / listening-sync work.
- `Media` - Codec / remux work (reserved; media pool remains separate).
- `Transcription` - Speech-to-text / derived transcript work (reserved for #121).
- `Indexing` - Search-index / embedding work (reserved).

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the canonical snake_case wire string.
- `fn parse(s: &str) -> Option<Self>` - Parses the canonical wire string; returns `None` when unknown.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> JobResourceClass`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobResourceClass) -> bool`



## bookclerk_library::models::JobState

*Enum*

Lifecycle state of a durable job row.

**Variants:**
- `Pending` - Admitted and waiting to be claimed.
- `Running` - Claimed by a worker that holds a live lease.
- `Succeeded` - Finished successfully.
- `Failed` - Exhausted retries or failed without retry.
- `Cancelled` - Cancelled by an operator before or during execution.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the canonical snake_case wire string.
- `fn parse(s: &str) -> Option<Self>` - Parses the canonical wire string; returns `None` when unknown.
- `fn is_active(self: Self) -> bool` - True when the job may still be claimed or is executing.
- `fn is_terminal(self: Self) -> bool` - True when the job will not run again.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> JobState`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobState) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::models::JobTempPath

*Struct*

Scratch path registered against a job for crash cleanup.

**Fields:**
- `id: i64` - Surrogate primary key.
- `job_id: String` - Owning job id.
- `path: String` - Absolute filesystem path.
- `created_at: chrono::DateTime<chrono::Utc>` - When the path was registered.
- `reserved_bytes: u64` - Bytes reserved against the job temp quota for this path.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobTempPath`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::JobTrigger

*Enum*

Who admitted the job (API vs periodic scheduler).

**Variants:**
- `Api` - Enqueued from the HTTP control plane, tray, or CLI daemon commands.
- `Scheduler` - Enqueued by the daemon interval scheduler.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the canonical snake_case wire string.
- `fn parse(s: &str) -> Option<Self>` - Parses the canonical wire string; returns `None` when unknown.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> JobTrigger`
- **Clone**
  - `fn clone(self: &Self) -> JobTrigger`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobTrigger) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::ListeningProgressRecord

*Struct*

Listening progress snapshot from an external player (e.g. AudioBookshelf).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
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
- `updated_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was last modified.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ListeningProgressRecord`



## bookclerk_library::models::MAX_QUEUE_NAME_LEN

*Constant*: `usize`

Longest queue name.



## bookclerk_library::models::MAX_QUEUE_WISHERS

*Constant*: `usize`

Maximum distinct people serialized on a global-queue or Discover card.



## bookclerk_library::models::NODE_QUEUE_PREFIX

*Constant*: `&str`

Prefix of the per-node queues (`node.<node_id>`) node subsystems consume.



## bookclerk_library::models::OPERATOR_PREFS_KEY

*Constant*: `&str`

Subject key for the shared operator account.



## bookclerk_library::models::OidcClientRecord

*Struct*

Registered OIDC authorization-server client (Bookclerk as IdP).

**Fields:**
- `id: i64` - Surrogate primary key.
- `client_id: String` - Public client_id used at authorize / token.
- `client_secret_hash: Option<String>` - Hash of the client secret; `None` for public PKCE clients.
- `redirect_uris: Vec<String>` - Allowed OAuth redirect URIs.
- `name: Option<String>` - Operator-facing display name.
- `issue_refresh_token: bool` - When true, token responses include a refresh token.
- `allowed_scopes: Vec<String>` - Scopes this client may be granted (`openid`, `profile`, `email`, …).
- `enabled: bool` - When true, authorize and token endpoints accept this client.
- `plugin_id: Option<String>` - Plugin id that owns this client; `None` for operator-created clients.
- `created_at: String` - RFC 3339 timestamp when the row was inserted.

**Methods:**

- `fn has_secret(self: &Self) -> bool` - True when a client secret hash is stored (confidential client).
- `fn is_plugin_provided(self: &Self) -> bool` - True when a plugin owns this client's identity and redirect URIs.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> OidcClientRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::models::OperatorSessionRecord

*Struct*

Durable operator session metadata (hashed token is never exposed).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `expires_at: chrono::DateTime<chrono::Utc>` - RFC 3339 expiry for the ticket, session, or code.
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.
- `last_used_at: Option<chrono::DateTime<chrono::Utc>>` - RFC 3339 time of the last authenticated use of this session.
- `elevated_from_user_id: Option<i64>` - User id that elevated into this operator session, if any.
- `impersonating_user_id: Option<i64>` - User id being impersonated by this operator session, if any.
- `user_agent: Option<String>` - Raw User-Agent captured at session mint (optional).
- `device_type: Option<String>` - Best-effort device class (`desktop` / `mobile` / `tablet` / `api`).
- `client_label: Option<String>` - Best-effort OS / client label (`Windows`, `Android`, `API`, …).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> OperatorSessionRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::PendingWakeProgress

*Struct*

How far one claimed wake tick got.

**Fields:**
- `claimed: u32` - Events this tick claimed and processed one delivery page for.
- `still_pending: bool` - True when another tick should run (saved cursor or remaining flags).

**Traits:** Copy, Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> PendingWakeProgress`
- **Clone**
  - `fn clone(self: &Self) -> PendingWakeProgress`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PendingWakeProgress) -> bool`



## bookclerk_library::models::PortalIdentity

*Struct*

Portal identity bound to an external provider user (e.g. ABS).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `provider: String` - External identity or integration provider id (for example ABS).
- `external_user_id: String` - User id at the external provider.
- `label: Option<String>` - Optional operator-facing display label.
- `user_id: Option<i64>` - First-party user this external identity is linked to (Phase 1).
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.
- `picture_url: Option<String>` - Last HTTPS profile picture URL from the identity provider, when any.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PortalIdentity`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::PortalSessionRecord

*Struct*

Portal session row for session lists (includes hash for current-session match).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `token_hash: String` - SHA-256 hex of the session token (never serialized to clients).
- `created_at: String` - RFC 3339 created timestamp.
- `expires_at: String` - RFC 3339 expiry timestamp.
- `last_used_at: Option<String>` - RFC 3339 last-used timestamp, when known.
- `user_agent: Option<String>` - Raw User-Agent captured at session mint (optional).
- `device_type: Option<String>` - Best-effort device class.
- `client_label: Option<String>` - Best-effort OS / client label.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> PortalSessionRecord`



## bookclerk_library::models::PublishDomainEventOutcome

*Enum*

Result of `crate::LibraryStore::publish_domain_event`.

**Variants:**
- `Created{ id: String }` - A new outbox row was inserted.
- `Duplicate{ existing_id: String }` - An equivalent `(event_type, dedup_key)` row already exists.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PublishDomainEventOutcome) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PublishDomainEventOutcome`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::models::PublishDomainEventSpec

*Struct*

Producer-side domain event (or queue message) to persist in the outbox.

A domain event leaves `queue_name` empty (the bus, [`CORE_EVENTS_QUEUE`](models.md#bookclerk_librarymodelscore_events_queue))
and sets `event_type`. A queue message names its queue and leaves
`event_type` empty; `content_type` says how `payload` is encoded and
`delay_seconds` holds its deliveries back.

**Fields:**
- `id: String` - Stable event id (UUID). Generated when empty.
- `event_type: String` - Event type (`book_acquired`); empty for queue messages.
- `schema_version: i64` - Payload schema version.
- `account_id: String` - Tenant / account id.
- `source: String` - Producer plugin id (`audible`, …); empty when unknown.
- `correlation_id: String` - Trace correlation id.
- `causation_id: String` - Causing event or job id.
- `dedup_key: String` - Unique with `event_type` (duplicate publishes coalesce).
- `payload: String` - Bounded JSON payload (never media bytes).
- `ordering_key: String` - FIFO key copied onto each delivery (`book uuid` for `book_acquired`).
- `queue_name: String` - Queue name; empty means the domain-event bus ([`CORE_EVENTS_QUEUE`](models.md#bookclerk_librarymodelscore_events_queue)).
- `content_type: String` - Body encoding (`json`, `text`, `bytes`); empty means `json`.
- `delay_seconds: u32` - Seconds before the first delivery may be claimed (queue messages).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PublishDomainEventSpec`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> PublishDomainEventSpec`



## bookclerk_library::models::QUEUE_CONTENT_TYPE_BYTES

*Constant*: `&str`

Queue message body is raw bytes stored as base64 text.



## bookclerk_library::models::QUEUE_CONTENT_TYPE_JSON

*Constant*: `&str`

Queue message body is UTF-8 JSON text.



## bookclerk_library::models::QUEUE_CONTENT_TYPE_TEXT

*Constant*: `&str`

Queue message body is plain UTF-8 text.



## bookclerk_library::models::QueueWisher

*Struct*

Compact wisher shown on the global queue (avatar + label).

**Fields:**
- `user_id: Option<i64>` - First-party `users.id` when the portal identity is linked.
- `identity_id: Option<i64>` - Portal identity that created the wish (dedupe for unlinked identities).
- `display_name: Option<String>` - Display name, identity label, or `"Operator"`.
- `login_name: Option<String>` - Local login name when the first-party user has one.
- `operator: bool` - True when the wish came from the operator token (no portal user).
- `has_avatar: bool` - True when an uploaded avatar file exists (filled by the daemon).
- `avatar_source: Option<String>` - Explicit picture choice when the user has set one.
- `gravatar_hash: Option<String>` - SHA-256 hex of the contact email for Gravatar.
- `picture_url: Option<String>` - HTTPS picture from the wishing identity, when any.

**Methods:**

- `fn same_person(self: &Self, other: &Self) -> bool` - Returns true when both rows represent the same person for aggregation.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &QueueWisher) -> bool`
- **Default**
  - `fn default() -> QueueWisher`
- **Clone**
  - `fn clone(self: &Self) -> QueueWisher`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::RequestStatus

*Enum*

Wishlist row status (`open` while wishlisted; `cancelled` after un-wishlist).

**Variants:**
- `Open` - Wishlist item is still open (not fulfilled/cancelled).
- `Cancelled` - Request cancelled by the requester or operator.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the canonical snake_case / lowercase wire string.
- `fn parse(s: &str) -> Option<Self>` - Parses the canonical wire string; returns `None` when unknown.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> RequestStatus`
- **Default**
  - `fn default() -> RequestStatus`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RequestStatus) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::models::SecurityAuditEvent

*Struct*

Security audit event (elevate / impersonate / login / provision).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp of the audit event.
- `actor: String` - Actor label (user id, operator, or system).
- `action: String` - Audit action verb (for example `login`, `rotate_token`).
- `detail_json: Option<String>` - JSON object with structured event details (no secrets).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SecurityAuditEvent`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::StoredPasskey

*Struct*

Stored WebAuthn credential (passkey JSON plus the label shown in Settings).

**Fields:**
- `id: i64` - Surrogate primary key.
- `credential_id: String` - Base64url credential id.
- `passkey_json: String` - Serialized `webauthn_rs` passkey JSON.
- `name: Option<String>` - Label chosen at registration; `None` for rows created before names existed.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> StoredPasskey`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::models::TitleRequestRecord

*Struct*

Personal wishlist row (also contributes to the shared global queue while open).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `uuid: String` - Stable UUID for this row (API / foreign-key identity).
- `identity_id: Option<i64>` - Foreign key to `portal_identities.id`.
- `title: String` - Display title of the work or edition.
- `authors: Option<String>` - Comma-separated or JSON author list from the storefront.
- `asin: Option<String>` - Amazon ASIN when known; otherwise null.
- `isbn: Option<String>` - ISBN-10/13 when known; otherwise null.
- `notes: Option<String>` - Free-form operator or requester notes.
- `status: RequestStatus` - Lifecycle status for the row (user, request, …).
- `work_key: String` - Stable bibliographic key (`isbn:…` / `asin:…` / `soft:…`) for aggregation.
- `work_id: Option<String>` - Canonical work id this edition or request resolves to.
- `resolved_book_uuid: Option<String>` - Library `books.uuid` once the wishlist item is fulfilled.
- `cover_url: Option<String>` - HTTPS URL for cover art when known.
- `sources: Vec<TitleRequestSourceRecord>` - Per-storefront snapshots (empty for legacy rows).
- `description: Option<String>` - Blurb / synopsis text (may contain HTML).
- `subtitle: Option<String>` - Optional subtitle from bibliographic metadata.
- `narrators: Option<String>` - Comma-separated or JSON narrator list when present.
- `series: Option<String>` - Series name when the title belongs to a series.
- `series_index: Option<String>` - Position within the series (storefront string form).
- `publisher: Option<String>` - Publisher name from metadata enrichment or the storefront.
- `length_minutes: Option<i64>` - Runtime in whole minutes when the storefront reports it.
- `published_at: Option<String>` - Publication date string from the storefront or enrichment.
- `genres: Option<String>` - Merged genre / category list for display.
- `language: Option<String>` - BCP-47 or storefront language code when known.
- `store_editions: Vec<WishlistStoreEdition>` - Distinct storefront editions contributing to this wishlist item.
- `purchase_hints: Vec<WishlistPurchaseHint>` - Purchase URLs and prices observed per storefront.
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.
- `updated_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was last modified.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> TitleRequestRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::TitleRequestSourceRecord

*Struct*

Per-storefront catalog/pricing snapshot attached to a wishlist row.

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `title_request_id: i64` - Foreign key to `title_requests.id`.
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
- `observed_at: Option<chrono::DateTime<chrono::Utc>>` - RFC 3339 time when this storefront snapshot was observed.
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.
- `updated_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was last modified.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TitleRequestSourceRecord`
- **Default**
  - `fn default() -> TitleRequestSourceRecord`



## bookclerk_library::models::UserIntegrationHint

*Struct*

Storefront / integration connection summary for admin user lists.

**Fields:**
- `source: String` - Content-source or integration plugin id (`audible`, `libro`, …).
- `account_id: String` - Store or operator account id this row belongs to.
- `label: Option<String>` - Optional operator-facing account label.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UserIntegrationHint`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::UserInviteRecord

*Struct*

Invite ticket for provisioning a User (token plaintext never stored).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `token_hash: String` - SHA-256 hex digest of the opaque token (plaintext never stored).
- `role: UserRole` - First-party role (`administrator` or `member`).
- `login_name: Option<String>` - Local username for password login, when set.
- `display_name: Option<String>` - Human-readable name shown in the UI.
- `expires_at: chrono::DateTime<chrono::Utc>` - RFC 3339 expiry for the ticket, session, or code.
- `redeemed_at: Option<chrono::DateTime<chrono::Utc>>` - RFC 3339 time when the ticket/invite was redeemed, if any.
- `created_by: String` - Actor that created the row (user id or operator label).
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UserInviteRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::UserListeningHint

*Struct*

Recent unfinished listening hint for admin user lists.

**Fields:**
- `title: Option<String>` - Display title when the provider reported one.
- `provider: String` - Integration / storefront provider id.
- `last_listened_at: chrono::DateTime<chrono::Utc>` - RFC 3339 time of the last playback update.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UserListeningHint`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::UserPreferences

*Struct*

Per-user GUI / Discover preferences (operator or portal identity).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `subject_key: String` - Preference subject key (`operator`, `user:<id>`, portal key).
- `identity_id: Option<i64>` - Foreign key to `portal_identities.id`.
- `default_view: String` - Preferred library SPA view identifier (for example `grid` or `list`).
- `disabled_shelves: Vec<String>` - Shelf kind ids to hide (`author`, `chirp_deals`, …). Empty = all on.
- `discover_sort: String` - Preferred Discover sort column (for example `title` or `added`).
- `discover_sort_dir: String` - Discover sort direction (`asc` / `desc`).
- `discover_language: Option<String>` - Preferred content language (`en`, `__all__`, …). `None` = browser default.
- `discover_excluded_sources: Vec<String>` - Store ids to hide in Discover. Empty = all sources (including future).
- `theme: String` - Appearance preference (`system`, `light`, or `dark`).
- `updated_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was last modified.

**Methods:**

- `fn defaults_for(subject_key: &str, identity_id: Option<i64>) -> Self` - Defaults when no row exists yet.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> UserPreferences`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::models::UserPresenceExtras

*Struct*

Presence + connection extras for administrator user management.

**Fields:**
- `online: bool` - True when the user has a non-expired portal session.
- `listening: Option<UserListeningHint>` - Most recent unfinished listen within the listening window, if any.
- `integrations: Vec<UserIntegrationHint>` - Linked storefront / integration accounts across portal identities.
- `last_active_at: Option<chrono::DateTime<chrono::Utc>>` - Most recent *unexpired* portal session activity (RFC 3339), when known.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UserPresenceExtras`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::UserRecord

*Struct*

First-party Bookclerk user (security principal for portal paths).

**Fields:**
- `id: i64` - Surrogate primary key assigned by the database.
- `role: UserRole` - First-party role (`owner`, `administrator`, `host`, or `member`).
- `status: UserStatus` - Lifecycle status for the row (user, request, …).
- `display_name: Option<String>` - Human-readable name shown in the UI.
- `login_name: Option<String>` - Local username for password login, when set.
- `email: Option<String>` - Optional contact email (notifications / magic-link invites later).
- `has_password: bool` - Whether a local password hash is stored for this user.
- `security_version: i64` - Incremented to invalidate existing sessions after security changes.
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.
- `updated_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was last modified.
- `last_seen_at: Option<chrono::DateTime<chrono::Utc>>` - Last authenticated portal activity (RFC 3339); survives session expiry.
- `avatar_source: Option<String>` - Explicit picture choice (`monogram` / `gravatar` / `upload` / `sso:{id}`); `None` is auto.
- `totp_enabled: bool` - True when a TOTP authenticator has been confirmed for local password login.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UserRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::UserRole

*Enum*

First-party user role (`owner` | `administrator` | `host` | `member`).

**Variants:**
- `Owner` - Cluster super-user: may elevate to Operator after password re-auth.
- `Administrator` - Library / user administration without Operator elevation.
- `Host` - Member who administers the nodes they own (budgets, drain, name).
- `Member` - Library member with scoped portal access (no operator token).

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the canonical snake_case / lowercase wire string.
- `fn parse(s: &str) -> Option<Self>` - Parses the canonical wire string; returns `None` when unknown.
- `fn is_privileged(self: Self) -> bool` - Whether this role may provision users and manage the host library.
- `fn can_elevate(self: Self) -> bool` - Whether this role may elevate to a short-lived Operator session.
- `fn can_administer_owned_nodes(self: Self) -> bool` - Whether this role reaches the node administration routes even while

**Traits:** Eq, Copy

**Trait Implementations:**

- **Display**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &UserRole) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> UserRole`



## bookclerk_library::models::UserSsoPicture

*Struct*

IdP-supplied avatar available as a profile-picture choice.

**Fields:**
- `identity_id: i64` - `portal_identities.id` this picture belongs to.
- `provider: String` - Identity-broker id (`oidc:google`, `oidc:github`, …).
- `picture_url: String` - HTTPS URL to display (hotlinked from the IdP / CDN).
- `last_used_at: Option<chrono::DateTime<chrono::Utc>>` - Most recent portal session activity for this identity, when any.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UserSsoPicture`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::UserStatus

*Enum*

First-party user status.

**Variants:**
- `Active` - Account may sign in and use the portal.
- `Disabled` - Account is blocked from new sessions.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the canonical snake_case / lowercase wire string.
- `fn parse(s: &str) -> Option<Self>` - Parses the canonical wire string; returns `None` when unknown.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> UserStatus`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &UserStatus) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::models::WishlistPurchaseHint

*Struct*

Snapshotted purchase link/price for a wishlist title.

**Fields:**
- `source: String` - Content-source plugin id (`audible`, `libro`, …).
- `product_id: String` - Storefront product id (ASIN, ISBN, UUID, …).
- `title: Option<String>` - Display title of the work or edition.
- `url: Option<String>` - Storefront product or purchase URL when known.
- `price_cents: Option<i64>` - Observed price in minor currency units, when known.
- `currency: Option<String>` - ISO 4217 currency code for price fields.
- `price_label: Option<String>` - Storefront-formatted price string for display.
- `list_price_cents: Option<i64>` - List/MSRP price in minor units, when known.
- `list_price_label: Option<String>` - Storefront-formatted list price for display.
- `member_price_cents: Option<i64>` - Member/subscriber price in minor units, when known.
- `member_price_label: Option<String>` - Storefront-formatted member price for display.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> WishlistPurchaseHint`
- **Default**
  - `fn default() -> WishlistPurchaseHint`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::models::WishlistStoreEdition

*Struct*

Storefront edition key for wishlist / queue payloads.

**Fields:**
- `source: String` - Content-source plugin id (`audible`, `libro`, …).
- `product_id: String` - Storefront product id (ASIN, ISBN, UUID, …).

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &WishlistStoreEdition) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> WishlistStoreEdition`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::models::WorkRecord

*Struct*

Canonical work spanning one or more ownership rows / editions.

**Fields:**
- `id: String` - Surrogate primary key assigned by the database.
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
- `created_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was inserted.
- `updated_at: chrono::DateTime<chrono::Utc>` - RFC 3339 timestamp when the row was last modified.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> WorkRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::models::catalog_subscribers_for_event

*Function*

Enabled catalog rows that match `event` type, schema version, and filter.

```rust
fn catalog_subscribers_for_event(catalog: &[EventSubscriberCatalogRecord], event: &DomainEventRecord) -> Vec<EventSubscriber>
```



## bookclerk_library::models::collapse_live_subscriber_nodes

*Function*

Collapse live per-node rows into one catalog record per `plugin_id`.

`enabled` is true when any live row is enabled. Subscriptions are the union
of enabled live rows (disabled nodes do not contribute filters).

```rust
fn collapse_live_subscriber_nodes(rows: &[EventSubscriberNodeRecord]) -> Vec<EventSubscriberCatalogRecord>
```



## bookclerk_library::models::content_kind_from_classic

*Function*

Map classic EF `ContentType` to bookclerk `content_kind`.

```rust
fn content_kind_from_classic(content_type: i64) -> String
```



## bookclerk_library::models::content_kind_to_classic

*Function*

Classic EF `ContentType` enum value.

```rust
fn content_kind_to_classic(content_kind: &str) -> i32
```



## bookclerk_library::models::event_filter_matches

*Function*

True when `filter` is absent/empty or every filter key equals the payload object value.

```rust
fn event_filter_matches(filter: Option<&serde_json::Value>, payload_json: &str) -> bool
```



## bookclerk_library::models::event_matches_wake_grants

*Function*

True when an event matches persisted wake grants, or legacy type+filter rows.

```rust
fn event_matches_wake_grants(grants_json: &str, legacy_filter_json: &str, schema_version: i64, payload: &str) -> bool
```



## bookclerk_library::models::intersect_event_filters

*Function*

Merge declared and requested payload filters; `None` on a same-key conflict.

```rust
fn intersect_event_filters(declared: Option<&serde_json::Value>, requested: Option<&serde_json::Value>) -> Option<serde_json::Value>
```



## bookclerk_library::models::is_downloadable

*Function*

True when the title can be downloaded (classic `WithoutParents`).

```rust
fn is_downloadable(content_kind: &str) -> bool
```



## bookclerk_library::models::is_episode

*Function*

True when `content_kind` is a podcast episode.

```rust
fn is_episode(content_kind: &str) -> bool
```



## bookclerk_library::models::is_podcast_parent

*Function*

True when `content_kind` is a podcast parent / show (no audio to acquire).

```rust
fn is_podcast_parent(content_kind: &str) -> bool
```



## bookclerk_library::models::is_valid_queue_name

*Function*

True when `name` is `[a-z0-9][a-z0-9._-]*` and at most [`MAX_QUEUE_NAME_LEN`](models.md#bookclerk_librarymodelsmax_queue_name_len) chars.

Host-reserved names (`core.events`, `node.<id>`) satisfy this grammar; the
manifest validator, not the outbox, keeps plugins off those prefixes.

```rust
fn is_valid_queue_name(name: &str) -> bool
```



## bookclerk_library::models::job_backoff_run_after

*Function*

Next `run_after` after a failed attempt, exponential from a 5s base (capped).

```rust
fn job_backoff_run_after(attempt_count: i64, now: chrono::DateTime<chrono::Utc>) -> chrono::DateTime<chrono::Utc>
```



## bookclerk_library::models::normalize_theme

*Function*

Canonical appearance preference; unknown values become `system`.

```rust
fn normalize_theme(raw: &str) -> String
```



## bookclerk_library::models::portal_prefs_key

*Function*

Subject key for a portal identity (legacy; prefer [`user_prefs_key`](models.md#bookclerk_librarymodelsuser_prefs_key)).

```rust
fn portal_prefs_key(identity_id: i64) -> String
```



## bookclerk_library::models::push_queue_wisher

*Function*

Adds `wisher` when they are not already present (capped at [`MAX_QUEUE_WISHERS`](models.md#bookclerk_librarymodelsmax_queue_wishers)).

Duplicate people keep the richer picture and name fields.

# Arguments

* `wishers` - Accumulated people for this work.
* `wisher` - Person who has this title on an open wishlist.

```rust
fn push_queue_wisher(wishers: & mut Vec<QueueWisher>, wisher: QueueWisher)
```



## bookclerk_library::models::queue_name_or_bus

*Function*

Resolves an optional queue name: empty means [`CORE_EVENTS_QUEUE`](models.md#bookclerk_librarymodelscore_events_queue).

```rust
fn queue_name_or_bus(name: &str) -> &str
```



## bookclerk_library::models::subscription_matches_event

*Function*

True when one catalog subscription matches `event` type, schema, class, and filter.

```rust
fn subscription_matches_event(sub: &EventCatalogSubscription, event: &DomainEventRecord) -> bool
```



## bookclerk_library::models::user_prefs_key

*Function*

Subject key for a first-party user.

```rust
fn user_prefs_key(user_id: i64) -> String
```



## bookclerk_library::models::wake_grants_from_subscriptions

*Function*

Build wake grants from declared subscriptions for `wake_type`.

Empty `wake_type` yields no grants (timestamp-only). A type with no matching
subscription returns `"wake event type not subscribed"`. When every matching
subscription conflicts with the requested filter, returns
`"wake filter not within subscription"`.

# Errors

Returns a static reason when the type is undeclared or the requested filter
cannot intersect any declared filter.

```rust
fn wake_grants_from_subscriptions(subs: &[EventCatalogSubscription], wake_type: &str, requested_filter_json: &str) -> std::result::Result<Vec<EventWakeGrant>, &'static str>
```



