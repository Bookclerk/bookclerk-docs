**bookclerk_plugin_abi > rpc_types**

# Module: rpc_types

## Contents

**Structs**

- [`CopyResult`](#copyresult) - Result of a server-side copy.
- [`DomainEvent`](#domainevent) - Domain event (not a job). Outbox-produced, at-least-once, idempotent consume.
- [`ExtensibleConfig`](#extensibleconfig) - Versioned plugin-specific config (not a substitute for typed ABI fields).
- [`HealthOk`](#healthok) - Health payload.
- [`JobCheckpoint`](#jobcheckpoint) - Bounded, versioned checkpoint attached to an invocation or suspension.
- [`JobInvocation`](#jobinvocation) - Versioned durable command envelope (not a domain event).
- [`JobInvocationLease`](#jobinvocationlease) - Claimed-lease fields used to populate a [`JobInvocation`](rpc_types.md#bookclerk_plugin_abirpc_typesjobinvocation).
- [`ListOptions`](#listoptions) - Paginated list request.
- [`ListPage`](#listpage) - One page of keys.
- [`ObjectInfo`](#objectinfo) - Object listing entry.
- [`ObjectMetadata`](#objectmetadata) - Metadata without a body.
- [`OidcClientTemplate`](#oidcclienttemplate) - Plugin-declared Bookclerk-as-IdP relying-party template.
- [`PluginDescribe`](#plugindescribe) - Guest identity returned by `PluginWorker.describe` (the adapter merges the manifest projection over the author `describe()`).
- [`PutResult`](#putresult) - Result of a streamed put.
- [`QueryPage`](#querypage) - Bounded query page for host-internal SeaORM session helpers (not Cap'n ABI).
- [`ScalarLimitsDto`](#scalarlimitsdto) - Wire form of [`ScalarLimits`](limits.md#bookclerk_plugin_abilimitsscalarlimits).
- [`WriteOptions`](#writeoptions) - Write options for `Destination.put`.

**Enums**

- [`EventResult`](#eventresult) - Per-event result of `crate::EventConsumer::event`.
- [`JobOutcome`](#joboutcome) - Outcome of `crate::JobRunner::job`.

**Constants**

- [`JSON_MEDIA_TYPE`](#json_media_type) - Media type of a JSON [`ExtensibleConfig`](rpc_types.md#bookclerk_plugin_abirpc_typesextensibleconfig) payload.
- [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes) - Maximum checkpoint payload size (bytes).

---

## bookclerk_plugin_abi::rpc_types::CopyResult

*Struct*

Result of a server-side copy.

**Fields:**
- `bytes_copied: u64` - Bytes copied when known.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> CopyResult`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CopyResult`
- **PartialEq**
  - `fn eq(self: &Self, other: &CopyResult) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::rpc_types::DomainEvent

*Struct*

Domain event (not a job). Outbox-produced, at-least-once, idempotent consume.

**Fields:**
- `event_id: String` - Unique event id.
- `event_type: String` - Event type name.
- `schema_version: u32` - Payload schema version.
- `occurred_at_unix_ms: u64` - UTC unix milliseconds.
- `account_id: String` - Tenant / account id.
- `source: String` - Producer plugin id; empty when unknown.
- `correlation_id: String` - Trace correlation id.
- `causation_id: String` - Prior event or invocation that caused this one.
- `deduplication_key: String` - Idempotent consume key.
- `delivery_attempt: u32` - 1-based delivery attempt.
- `payload: Vec<u8>` - Bounded payload.
- `checkpoint_json: String` - Checkpoint JSON from a prior `EventResult::Suspended` (≤ [`MAX_CHECKPOINT_BYTES`](rpc_types.md#bookclerk_plugin_abirpc_typesmax_checkpoint_bytes)).
- `checkpoint_schema_version: u32` - Schema version of `Self::checkpoint_json`.
- `invocation_sequence: u32` - Resume ordinal copied from the delivery row.
- `resume_pending: bool` - True when this invocation continues a prior `suspended` result.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> DomainEvent`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DomainEvent`
- **PartialEq**
  - `fn eq(self: &Self, other: &DomainEvent) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::rpc_types::EventResult

*Enum*

Per-event result of `crate::EventConsumer::event`.

**Variants:**
- `Ack` - Event processed.
- `Retry{ retry_at_unix_ms: u64, reason: String }` - Transient failure; retry after `retry_at_unix_ms`.
- `Reject{ reason: String }` - Permanent rejection.
- `DeadLetter{ reason: String }` - Poison; do not redeliver.
- `Suspended{ checkpoint_json: String, checkpoint_schema_version: u32, wake_at_unix_ms: u64, wake_on_event_type: String, wake_on_filter_json: String }` - Durable sleep: persist checkpoint, release the process, resume later.

**Methods:**

- `fn from_json_value(value: &serde_json::Value) -> crate::Result<Self>` - Decode JSON `{"kind":"ack"|…}`. Missing or unknown `kind` is an error.
- `fn from_json_str(raw: &str) -> crate::Result<Self>` - Decode a JSON object string as [`EventResult`](rpc_types.md#bookclerk_plugin_abirpc_typeseventresult).

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &EventResult) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> EventResult`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::rpc_types::ExtensibleConfig

*Struct*

Versioned plugin-specific config (not a substitute for typed ABI fields).

**Fields:**
- `schema_version: u32` - Payload schema version.
- `media_type: String` - IANA media type or `application/json`.
- `payload: Vec<u8>` - Bounded payload bytes.

**Methods:**

- `fn json(value: &serde_json::Value) -> Self` - Wraps a JSON document as an `application/json` payload (schema version 1).
- `fn json_from<T>(value: &T) -> crate::Result<Self>` - Serializes `value` as an `application/json` payload (schema version 1).
- `fn is_empty(self: &Self) -> bool` - True when no payload was supplied.
- `fn json_value(self: &Self) -> crate::Result<serde_json::Value>` - Decodes the payload as a JSON document; an empty payload is `{}`.
- `fn json_into<T>(self: &Self) -> crate::Result<T>` - Decodes the JSON payload into `T`; an empty payload decodes `{}`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ExtensibleConfig`
- **PartialEq**
  - `fn eq(self: &Self, other: &ExtensibleConfig) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> ExtensibleConfig`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::rpc_types::HealthOk

*Struct*

Health payload.

**Fields:**
- `ok: bool` - True when the role is ready.
- `detail: String` - Operator-facing detail.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> HealthOk`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> HealthOk`
- **PartialEq**
  - `fn eq(self: &Self, other: &HealthOk) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::rpc_types::JSON_MEDIA_TYPE

*Constant*: `&str`

Media type of a JSON [`ExtensibleConfig`](rpc_types.md#bookclerk_plugin_abirpc_typesextensibleconfig) payload.



## bookclerk_plugin_abi::rpc_types::JobCheckpoint

*Struct*

Bounded, versioned checkpoint attached to an invocation or suspension.

**Fields:**
- `schema_version: u32` - Checkpoint payload schema version. Unknown versions fail closed.
- `json: String` - JSON checkpoint bytes (must be ≤ [`MAX_CHECKPOINT_BYTES`](rpc_types.md#bookclerk_plugin_abirpc_typesmax_checkpoint_bytes)).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobCheckpoint`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobCheckpoint) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::rpc_types::JobInvocation

*Struct*

Versioned durable command envelope (not a domain event).

Idempotency keys are scoped to `(account, plugin, commandType)` until a
terminal fenced outcome is committed. They are not reusable across accounts.
`deadline_unix_ms` is a guest hint; the host fence/lease is authoritative
and must not be outlived (clock skew across VPS nodes).
The envelope is not independently persisted as opaque bytes across ABI
majors; command `payload_schema_version` and checkpoint schema versions
version the stored pieces.

**Fields:**
- `payload_schema_version: u32` - Command payload schema version for `Self::command_type`.
- `invocation_id: String` - Unique id for this attempt envelope.
- `command_type: String` - Command type (`stream_copy`, …).
- `payload_json: String` - JSON payload for that command type.
- `idempotency_key: String` - Idempotency key (see struct docs for scope).
- `attempt: u32` - 1-based attempt counter.
- `correlation_id: String` - Correlation id for traces.
- `causation_id: Option<String>` - Optional causation id (prior invocation that produced this one).
- `deadline_unix_ms: u64` - Guest-visible UTC deadline as unix milliseconds.
- `checkpoint: Option<JobCheckpoint>` - Bounded, versioned checkpoint from a prior suspension.
- `invocation_sequence: u32` - Resume ordinal; distinct from failure `Self::attempt`.
- `step_id: Option<String>` - Deterministic step identifier within this invocation.

**Methods:**

- `fn stream_copy<impl Into<String>, impl Into<String>>(job_id: impl Trait, payload_json: impl Trait) -> Self` - Builds a v1 `stream_copy` invocation with a far-future deadline.
- `fn stream_copy_from_lease<impl Into<String>>(lease: JobInvocationLease, payload_json: impl Trait) -> Self` - Builds a `stream_copy` envelope from a claimed job lease.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &JobInvocation) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> JobInvocation`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::rpc_types::JobInvocationLease

*Struct*

Claimed-lease fields used to populate a [`JobInvocation`](rpc_types.md#bookclerk_plugin_abirpc_typesjobinvocation).

**Fields:**
- `job_id: String` - Durable job id (`jobs.id`).
- `attempt: u32` - 1-based attempt counter from the claim.
- `generation: i64` - Lease generation from the successful claim.
- `dedup_key: String` - Queue dedup key; stable across reclaim of the same command.
- `deadline_unix_ms: u64` - Lease expiry as UTC unix milliseconds (guest deadline hint).
- `checkpoint: Option<JobCheckpoint>` - Checkpoint restored from a prior `JobOutcome::Suspended`.
- `invocation_sequence: u32` - Resume ordinal from the durable payload; distinct from `Self::attempt`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobInvocationLease) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> JobInvocationLease`



## bookclerk_plugin_abi::rpc_types::JobOutcome

*Enum*

Outcome of `crate::JobRunner::job`.

Suspension is durable only after Bookclerk atomically commits the fenced
outcome. Open streams and process memory do not survive.

**Variants:**
- `Completed{ message: String, bytes_copied: u64 }` - Handler finished successfully.
- `Retryable{ message: String, retry_after_unix_ms: Option<u64> }` - Transient failure; host may retry after `retry_after_unix_ms`.
- `Rejected{ message: String }` - Permanent rejection (do not retry the same payload).
- `Cancelled{ message: String }` - Cancelled by host fence / guest abort.
- `Suspended{ checkpoint: JobCheckpoint, wake_at_unix_ms: u64 }` - Durable pause; resume with `checkpoint` after `wake_at_unix_ms`.

**Methods:**

- `fn is_terminal(self: &Self) -> bool` - True when the host should treat the invocation as finished.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobOutcome`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobOutcome) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::rpc_types::ListOptions

*Struct*

Paginated list request.

**Fields:**
- `prefix: String` - Key prefix filter.
- `cursor: Option<String>` - Opaque continuation token.
- `limit: u32` - Requested page size (0 = default).

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> ListOptions`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ListOptions`
- **PartialEq**
  - `fn eq(self: &Self, other: &ListOptions) -> bool`



## bookclerk_plugin_abi::rpc_types::ListPage

*Struct*

One page of keys.

Cursors are opaque. Missing/stale cursors must return `invalid_cursor`,
never silently restart at page one. Concurrent mutation is weakly
consistent unless a backend snapshot is documented.

**Fields:**
- `objects: Vec<ObjectInfo>` - Objects in this page.
- `next_cursor: Option<String>` - Continuation token; `None` when this is the last page.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> ListPage`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ListPage`
- **PartialEq**
  - `fn eq(self: &Self, other: &ListPage) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::rpc_types::MAX_CHECKPOINT_BYTES

*Constant*: `u32`

Maximum checkpoint payload size (bytes).



## bookclerk_plugin_abi::rpc_types::ObjectInfo

*Struct*

Object listing entry.

**Fields:**
- `key: String` - Object key.
- `size: u64` - Size in bytes.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ObjectInfo) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ObjectInfo`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::rpc_types::ObjectMetadata

*Struct*

Metadata without a body.

**Fields:**
- `key: String` - Object key.
- `size: u64` - Size in bytes.
- `content_type: Option<String>` - MIME type when known.
- `etag: Option<String>` - Opaque etag when the backend provides one.
- `sha256: Option<Vec<u8>>` - SHA-256 digest when computed.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> ObjectMetadata`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ObjectMetadata`
- **PartialEq**
  - `fn eq(self: &Self, other: &ObjectMetadata) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::rpc_types::OidcClientTemplate

*Struct*

Plugin-declared Bookclerk-as-IdP relying-party template.

The host materializes `oidc_clients` rows. Plugins never mint tokens.
`origin_config_key` is a dotted config path such as
`integrations.audiobookshelf.base_url`.

**Fields:**
- `client_id: String` - Stable OAuth `client_id` registered with Bookclerk's authorization server.
- `display_name: String` - Operator-facing card title.
- `callback_path: String` - Path appended to the plugin origin (e.g. `/auth/openid/callback`).
- `public_client: bool` - When true, the client is public PKCE (no secret).
- `default_scopes: Vec<String>` - Scopes offered on first materialization (`openid` / `profile` typical).
- `issue_refresh_token: bool` - When true, `/oidc/token` may issue refresh tokens for new rows.
- `origin_config_key: String` - Dotted config key that supplies the player origin.

**Methods:**

- `fn scopes_or_default(self: &Self) -> Vec<String>` - Scopes used when the guest omitted `defaultScopes`.
- `fn display_name_or_id(self: &Self) -> &str` - Card title, falling back to `client_id`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &OidcClientTemplate) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> OidcClientTemplate`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::rpc_types::PluginDescribe

*Struct*

Guest identity returned by `PluginWorker.describe` (the adapter merges the manifest projection over the author `describe()`).

Every field is a typed Cap'n Proto field of `PluginDescribe`; there is no
side-channel JSON. Storefront UI extras (`brand`, `config_options`,
`portal_auth_mode`, ...) are empty / default for kinds that do not use them.

**Fields:**
- `api_version: u32` - Must equal [`crate::limits::PRODUCT_API_VERSION`](limits.md#bookclerk_plugin_abilimitsproduct_api_version).
- `id: String` - Plugin id matching `plugin.toml`.
- `display_name: Option<String>` - Optional UI name.
- `rpc_features: Vec<String>` - Accepted RPC features (intersection with the host offer).
- `scalar_limits: ScalarLimitsDto` - Effective numeric limits.
- `capabilities: crate::generated::PluginCapabilities` - Typed capability declaration (entrypoints, event consumers/producers,
- `portal_auth_mode: crate::generated::PortalAuthMode` - Portal Accounts connect mode for storefronts.
- `password_env_var: Option<String>` - Reserved wire field: hosts ignore it. Passwords reach a guest only as
- `aliases: Vec<String>` - Alternate ids accepted for config / CLI targeting.
- `sort_key: u32` - UI sort weight among peers of the same kind; lower sorts first.
- `brand: Option<crate::generated::Brand>` - Portal brand; `None` when the guest has no brand (wire `brand.id` is
- `config_options: Vec<crate::generated::ConfigOption>` - Discoverable config option groups for source UIs.
- `cli: crate::generated::CliSchema` - Embedded CLI schema (same shape as `cliDescribe`); empty when unused.

**Methods:**

- `fn has_entrypoint(self: &Self, entrypoint: Entrypoint) -> bool` - True when the guest exports the named [`Entrypoint`](generated.md#bookclerk_plugin_abigeneratedentrypoint).
- `fn consumes_event(self: &Self, event_type: &str) -> bool` - True when the guest's default entrypoint consumes `event_type`.
- `fn runs_job(self: &Self, job_type: &str) -> bool` - True when the guest's default entrypoint runs jobs of `job_type`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Self`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginDescribe) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginDescribe`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::rpc_types::PutResult

*Struct*

Result of a streamed put.

**Fields:**
- `key: String` - Object key written.
- `bytes_written: u64` - Bytes accepted.
- `etag: Option<String>` - Backend etag when available.
- `sha256: Option<Vec<u8>>` - Digest of the written body when computed.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PutResult) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> PutResult`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PutResult`



## bookclerk_plugin_abi::rpc_types::QueryPage

*Struct*

Bounded query page for host-internal SeaORM session helpers (not Cap'n ABI).

**Fields:**
- `rows_json: String` - JSON-encoded rows.
- `next_cursor: Option<String>` - Continuation token.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &QueryPage) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> QueryPage`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> QueryPage`



## bookclerk_plugin_abi::rpc_types::ScalarLimitsDto

*Struct*

Wire form of [`ScalarLimits`](limits.md#bookclerk_plugin_abilimitsscalarlimits).

**Fields:**
- `max_scalar_bytes: u32` - Max ordinary RPC value size in bytes.
- `max_stream_window_bytes: u32` - Max `ByteSource.pull` window in bytes.
- `max_list_page: u32` - Max objects per list page.

**Traits:** Copy, Eq

**Trait Implementations:**

- **From**
  - `fn from(value: ScalarLimits) -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ScalarLimitsDto`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScalarLimitsDto) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::rpc_types::WriteOptions

*Struct*

Write options for `Destination.put`.

**Fields:**
- `content_type: Option<String>` - MIME type.
- `content_length: Option<u64>` - Known length; omit when unknown.
- `sha256: Option<Vec<u8>>` - Expected SHA-256 of the body.
- `commit_token: Option<String>` - Destination-side stage-and-publish token. Empty means a one-shot put.
- `stage_only: bool` - When true, `put` stages remotely and does not publish until `commit`.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> WriteOptions`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> WriteOptions`
- **PartialEq**
  - `fn eq(self: &Self, other: &WriteOptions) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



