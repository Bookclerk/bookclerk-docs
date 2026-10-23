**bookclerk_plugin_manifest > types**

# Module: types

## Contents

**Structs**

- [`BindingCapabilities`](#bindingcapabilities) - Host bindings a plugin declared, resolved from the v3 manifest tables.
- [`CapabilitiesManifest`](#capabilitiesmanifest) - Full `[capabilities]` table required on every manifest.
- [`DatabaseBindingManifest`](#databasebindingmanifest) - One `[[databases]]` row — an isolated plugin-owned database binding.
- [`EffectiveWorkerdLimits`](#effectiveworkerdlimits) - Concrete limits after applying host defaults and hard caps.
- [`EventConsumer`](#eventconsumer) - One `[[events.consumers]]` row — a durable domain-event trigger for the
- [`EventProducer`](#eventproducer) - One `[[events.producers]]` row — an event type the plugin may publish
- [`EventsManifest`](#eventsmanifest) - `[events]` — consumers (triggers) and producers (publish grants).
- [`KvNamespaceManifest`](#kvnamespacemanifest) - One `[[kv_namespaces]]` row — a durable key/value namespace the host
- [`ModuleSpec`](#modulespec) - One module entry in `[[modules]]` (workerd script packages).
- [`NamedBinding`](#namedbinding) - A named host binding (`[secrets]`, `[work_fs]`, `[oauth]`, `[[kv_namespaces]]`).
- [`NetworkCapabilities`](#networkcapabilities) - `[capabilities.network]` — mode plus optional workerd host allowlist.
- [`OidcClientToml`](#oidcclienttoml) - One `[[oidc.clients]]` row (snake_case TOML).
- [`OidcManifest`](#oidcmanifest) - `[[oidc.clients]]` install-time OIDC AS templates.
- [`PluginManifest`](#pluginmanifest) - On-disk plugin descriptor (`plugin.toml`, `api_version = 3`).
- [`QueueConsumer`](#queueconsumer) - One `[[queues.consumers]]` row — a durable queue trigger for the default
- [`QueueProducer`](#queueproducer) - One `[[queues.producers]]` row — a queue the plugin may send to through
- [`QueuesManifest`](#queuesmanifest) - `[queues]` — Workers Queues producers (send bindings) and consumers
- [`TcpCapability`](#tcpcapability) - One raw TCP grant: hostname or wildcard plus allowed destination ports.
- [`TriggersManifest`](#triggersmanifest) - `[triggers]` — handlers on the default entrypoint the host invokes.
- [`WorkerdLimits`](#workerdlimits) - Optional workerd resource limits under `[workerd.limits]`.
- [`WorkerdRuntimeManifest`](#workerdruntimemanifest) - `[workerd]` — WorkerCode-equivalent isolate configuration.

**Enums**

- [`JailNetworkNeed`](#jailnetworkneed) - OS jail network capability derived from manifest network + oauth binding.
- [`NetworkMode`](#networkmode) - Network mode declared in `[capabilities.network]`.
- [`PluginFamily`](#pluginfamily) - Handler family — how the daemon, CLI, and UI group plugins and which
- [`PluginRuntimeKind`](#pluginruntimekind) - Guest runtime selection (`runtime` key in `plugin.toml`).

**Functions**

- [`entrypoint_family`](#entrypoint_family) - Handler family an [`Entrypoint`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedentrypoint) belongs to.
- [`is_host_event_type`](#is_host_event_type) - True when `event_type` is in [`HOST_EVENT_TYPES`](types.md#bookclerk_plugin_manifesttypeshost_event_types).
- [`is_reserved_queue_name`](#is_reserved_queue_name) - True when `name` starts with a [`RESERVED_QUEUE_PREFIXES`](types.md#bookclerk_plugin_manifesttypesreserved_queue_prefixes) entry.
- [`is_valid_database_binding_name`](#is_valid_database_binding_name) - True when `name` is a valid Workers-style binding name (`[A-Z][A-Z0-9_]*`).
- [`is_valid_queue_name`](#is_valid_queue_name) - True when `name` is a valid queue name: `[a-z0-9][a-z0-9._-]*`, at most

**Constants**

- [`ALL_ENTRYPOINTS`](#all_entrypoints) - Every entrypoint in declaration order (mirrors the Cap'n Proto enum).
- [`CONFIG_BINDING`](#config_binding) - Binding name the host uses for operator config (`[vars]`).
- [`DEFAULT_EVENTS_BINDING`](#default_events_binding) - Default `binding` for `[[events.producers]]`.
- [`DEFAULT_KV_BINDING`](#default_kv_binding) - Default `binding` for `[[kv_namespaces]]`.
- [`DEFAULT_KV_MAX_KEYS`](#default_kv_max_keys) - Default `max_keys` for `[[kv_namespaces]]`.
- [`DEFAULT_KV_MAX_VALUE_BYTES`](#default_kv_max_value_bytes) - Default `max_value_bytes` for `[[kv_namespaces]]`: 64 KiB.
- [`DEFAULT_OAUTH_BINDING`](#default_oauth_binding) - Default `binding` for `[oauth]`.
- [`DEFAULT_QUEUE_MAX_BATCH_SIZE`](#default_queue_max_batch_size) - Default `max_batch_size` for `[[queues.consumers]]`.
- [`DEFAULT_QUEUE_MAX_BATCH_TIMEOUT_SECS`](#default_queue_max_batch_timeout_secs) - Default `max_batch_timeout` (seconds) for `[[queues.consumers]]`.
- [`DEFAULT_QUEUE_MAX_RETRIES`](#default_queue_max_retries) - Default `max_retries` for `[[queues.consumers]]`.
- [`DEFAULT_SECRETS_BINDING`](#default_secrets_binding) - Default `binding` for `[secrets]`.
- [`DEFAULT_WORK_FS_BINDING`](#default_work_fs_binding) - Default `binding` for `[work_fs]`.
- [`HOST_EVENT_TYPES`](#host_event_types) - Domain event types only the host emits.
- [`MAX_DATABASE_BINDINGS`](#max_database_bindings) - Maximum named database bindings one plugin may declare.
- [`MAX_DATABASE_BINDING_NAME_LEN`](#max_database_binding_name_len) - Maximum length of one database binding name.
- [`MAX_KV_KEYS`](#max_kv_keys) - Largest `max_keys` a namespace may declare.
- [`MAX_KV_VALUE_BYTES`](#max_kv_value_bytes) - Largest `max_value_bytes` a namespace may declare: 1 MiB (the ABI cap).
- [`MAX_QUEUE_BATCH_SIZE`](#max_queue_batch_size) - Largest `max_batch_size` a consumer may declare.
- [`MAX_QUEUE_BATCH_TIMEOUT_SECS`](#max_queue_batch_timeout_secs) - Largest `max_batch_timeout` (seconds) a consumer may declare.
- [`MAX_QUEUE_NAME_LEN`](#max_queue_name_len) - Maximum length of a queue name.
- [`MAX_QUEUE_RETRIES`](#max_queue_retries) - Largest `max_retries` a consumer may declare.
- [`MAX_QUEUE_RETRY_DELAY_SECS`](#max_queue_retry_delay_secs) - Largest `retry_delay` (seconds) a consumer may declare: 12 hours.
- [`RESERVED_QUEUE_PREFIXES`](#reserved_queue_prefixes) - Queue-name prefixes only the host may use (`core.events`, `node.<id>`).

---

## bookclerk_plugin_manifest::types::ALL_ENTRYPOINTS

*Constant*: `[Entrypoint; 6]`

Every entrypoint in declaration order (mirrors the Cap'n Proto enum).



## bookclerk_plugin_manifest::types::BindingCapabilities

*Struct*

Host bindings a plugin declared, resolved from the v3 manifest tables.

Derived view produced by `PluginManifest::bindings`; `[vars]`,
`[secrets]`, `[[kv_namespaces]]`, `[work_fs]`, `[oauth]`, and
`[[databases]]` each map onto one flag or list here so host code can gate
jail layout, consent, and env injection uniformly.

**Fields:**
- `config: bool` - Guest may read plugin config delivered by the host (`[vars]`).
- `secrets: bool` - Guest may read sealed secrets / credentials via host bindings (`[secrets]`).
- `plugin_kv: bool` - `[[kv_namespaces]]` is declared: the host grants one durable
- `work_fs: bool` - Guest may use host-mediated work filesystem (`[work_fs]`).
- `oauth: bool` - Guest needs an OAuth-style callback tunnel (`[oauth]`).
- `databases: Vec<String>` - Named plugin-owned database bindings (`[[databases]] binding = "DB"`).

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> BindingCapabilities`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &BindingCapabilities) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> BindingCapabilities`



## bookclerk_plugin_manifest::types::CONFIG_BINDING

*Constant*: `&str`

Binding name the host uses for operator config (`[vars]`).



## bookclerk_plugin_manifest::types::CapabilitiesManifest

*Struct*

Full `[capabilities]` table required on every manifest.

Only `network` remains here in v3 (consent has no Wrangler analogue);
bindings and triggers moved to top-level Wrangler-shaped tables.

**Fields:**
- `network: NetworkCapabilities` - Network mode and optional workerd domain allowlist.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CapabilitiesManifest`
- **PartialEq**
  - `fn eq(self: &Self, other: &CapabilitiesManifest) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::types::DEFAULT_EVENTS_BINDING

*Constant*: `&str`

Default `binding` for `[[events.producers]]`.



## bookclerk_plugin_manifest::types::DEFAULT_KV_BINDING

*Constant*: `&str`

Default `binding` for `[[kv_namespaces]]`.



## bookclerk_plugin_manifest::types::DEFAULT_KV_MAX_KEYS

*Constant*: `u32`

Default `max_keys` for `[[kv_namespaces]]`.



## bookclerk_plugin_manifest::types::DEFAULT_KV_MAX_VALUE_BYTES

*Constant*: `u32`

Default `max_value_bytes` for `[[kv_namespaces]]`: 64 KiB.



## bookclerk_plugin_manifest::types::DEFAULT_OAUTH_BINDING

*Constant*: `&str`

Default `binding` for `[oauth]`.



## bookclerk_plugin_manifest::types::DEFAULT_QUEUE_MAX_BATCH_SIZE

*Constant*: `u32`

Default `max_batch_size` for `[[queues.consumers]]`.



## bookclerk_plugin_manifest::types::DEFAULT_QUEUE_MAX_BATCH_TIMEOUT_SECS

*Constant*: `u32`

Default `max_batch_timeout` (seconds) for `[[queues.consumers]]`.



## bookclerk_plugin_manifest::types::DEFAULT_QUEUE_MAX_RETRIES

*Constant*: `u32`

Default `max_retries` for `[[queues.consumers]]`.



## bookclerk_plugin_manifest::types::DEFAULT_SECRETS_BINDING

*Constant*: `&str`

Default `binding` for `[secrets]`.



## bookclerk_plugin_manifest::types::DEFAULT_WORK_FS_BINDING

*Constant*: `&str`

Default `binding` for `[work_fs]`.



## bookclerk_plugin_manifest::types::DatabaseBindingManifest

*Struct*

One `[[databases]]` row — an isolated plugin-owned database binding.

Each name binds a database provisioned by the active database adapter —
separate from the Bookclerk library and from every other plugin. Names
must be `[A-Z][A-Z0-9_]*`, unique, and at most
[`MAX_DATABASE_BINDING_NAME_LEN`](types.md#bookclerk_plugin_manifesttypesmax_database_binding_name_len) chars; the operator consents to each
binding before enable.

**Fields:**
- `binding: String` - Binding name exposed on `env` (Workers-style, e.g. `"DB"`).

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DatabaseBindingManifest`
- **PartialEq**
  - `fn eq(self: &Self, other: &DatabaseBindingManifest) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::types::EffectiveWorkerdLimits

*Struct*

Concrete limits after applying host defaults and hard caps.

Produced by `WorkerdLimits::effective`.

**Fields:**
- `cpu_ms: u32` - Effective CPU budget in milliseconds.
- `subrequests: u32` - Effective outbound fetch / subrequest budget.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &EffectiveWorkerdLimits) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> EffectiveWorkerdLimits`



## bookclerk_plugin_manifest::types::EventConsumer

*Struct*

One `[[events.consumers]]` row — a durable domain-event trigger for the
default entrypoint's `event(batch)` handler.

**Fields:**
- `event_type: String` - Versioned event type (`book_acquired`, …).
- `schema_versions: Vec<u32>` - Schema versions this guest can consume (default `[1]`).
- `supports_suspend: bool` - Whether `EventResult::Suspended` is supported for this type.
- `resource_class: String` - Concurrency class copied onto deliveries (default `"network"`).
- `filter: Option<serde_json::Value>` - Optional host-owned payload object filter (top-level key equality).
- `max_retries: Option<u32>` - Redelivery attempts before dead-lettering; `None` uses the host default.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventConsumer`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventConsumer) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_manifest::types::EventProducer

*Struct*

One `[[events.producers]]` row — an event type the plugin may publish
through its `EVENTS` binding.

**Fields:**
- `event_type: String` - Event type the plugin publishes (snake_case).
- `binding: String` - Publisher binding name on `env` (default `EVENTS`).

**Methods:**

- `fn binding_name(self: &Self) -> &str` - Resolves the publisher binding name (default [`DEFAULT_EVENTS_BINDING`](types.md#bookclerk_plugin_manifesttypesdefault_events_binding)).

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> EventProducer`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventProducer) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::types::EventsManifest

*Struct*

`[events]` — consumers (triggers) and producers (publish grants).

**Fields:**
- `consumers: Vec<EventConsumer>` - Event types delivered to the default entrypoint's `event(batch)`.
- `producers: Vec<EventProducer>` - Event types the plugin may publish.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> EventsManifest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventsManifest) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> EventsManifest`



## bookclerk_plugin_manifest::types::HOST_EVENT_TYPES

*Constant*: `[&str; 1]`

Domain event types only the host emits.

`book_acquired` is written by the host's acquire pipeline in the same
transaction as the library row once a source plugin's acquire handler
returns; the source never publishes it. Plugins consume these types but
cannot declare them under `[[events.producers]]`.



## bookclerk_plugin_manifest::types::JailNetworkNeed

*Enum*

OS jail network capability derived from manifest network + oauth binding.

Produced by `PluginManifest::jail_network_need`. Distinct from isolate
hostname allowlists ([`crate::EgressPolicy`](egress.md#bookclerk_plugin_manifestegressegresspolicy)), which apply only inside
workerd.

**Variants:**
- `None` - No IP sockets (`mode = "deny"` on native).
- `Outbound` - Native `outbound` without OAuth — coarse jail outbound (no hostname filter).
- `Listen` - Native `outbound` + `bindings.oauth`, or any workerd guest (loopback

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &JailNetworkNeed) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> JailNetworkNeed`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::types::KvNamespaceManifest

*Struct*

One `[[kv_namespaces]]` row — a durable key/value namespace the host
serves from the library database, scoped to this plugin.

The `binding` key names the property on `env` (default `KV`). The
`namespace` defaults to the binding name lowercased, so two rows are
distinct stores unless they name the same namespace on purpose. Values
live in the cluster: an instance moved to another host reads its own keys.

**Fields:**
- `binding: String` - Binding name exposed on `env`; empty means `KV`.
- `namespace: String` - Namespace the binding is scoped to (`[a-z0-9][a-z0-9._-]*`); empty
- `max_value_bytes: u32` - Largest value in bytes (default [`DEFAULT_KV_MAX_VALUE_BYTES`](types.md#bookclerk_plugin_manifesttypesdefault_kv_max_value_bytes), at
- `max_keys: u32` - Most live keys in the namespace (default [`DEFAULT_KV_MAX_KEYS`](types.md#bookclerk_plugin_manifesttypesdefault_kv_max_keys), at

**Methods:**

- `fn name_or<'a>(self: &'a Self, default: &'a str) -> &'a str` - Resolves the binding name, falling back to `default`.
- `fn binding_name(self: &Self) -> &str` - Binding name on `env` (`KV` when omitted).
- `fn namespace_name(self: &Self) -> String` - Namespace the binding is scoped to: the declared name, or the binding

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> KvNamespaceManifest`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvNamespaceManifest) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::types::MAX_DATABASE_BINDINGS

*Constant*: `usize`

Maximum named database bindings one plugin may declare.



## bookclerk_plugin_manifest::types::MAX_DATABASE_BINDING_NAME_LEN

*Constant*: `usize`

Maximum length of one database binding name.



## bookclerk_plugin_manifest::types::MAX_KV_KEYS

*Constant*: `u32`

Largest `max_keys` a namespace may declare.



## bookclerk_plugin_manifest::types::MAX_KV_VALUE_BYTES

*Constant*: `u32`

Largest `max_value_bytes` a namespace may declare: 1 MiB (the ABI cap).



## bookclerk_plugin_manifest::types::MAX_QUEUE_BATCH_SIZE

*Constant*: `u32`

Largest `max_batch_size` a consumer may declare.



## bookclerk_plugin_manifest::types::MAX_QUEUE_BATCH_TIMEOUT_SECS

*Constant*: `u32`

Largest `max_batch_timeout` (seconds) a consumer may declare.



## bookclerk_plugin_manifest::types::MAX_QUEUE_NAME_LEN

*Constant*: `usize`

Maximum length of a queue name.



## bookclerk_plugin_manifest::types::MAX_QUEUE_RETRIES

*Constant*: `u32`

Largest `max_retries` a consumer may declare.



## bookclerk_plugin_manifest::types::MAX_QUEUE_RETRY_DELAY_SECS

*Constant*: `u32`

Largest `retry_delay` (seconds) a consumer may declare: 12 hours.



## bookclerk_plugin_manifest::types::ModuleSpec

*Struct*

One module entry in `[[modules]]` (workerd script packages).

Used for packaging, Python detection ([`crate::manifest_needs_python`](egress.md#bookclerk_plugin_manifestegressmanifest_needs_python)),
and documenting the isolate module graph alongside `[workerd].main_module`.

**Fields:**
- `name: String` - Module name as known to the isolate (often matches the filename).
- `path: String` - Path relative to the plugin package root (or modules dir).
- `module_type: String` - Module type string (TOML key `type`; default `"js"`). Use `"python"`

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ModuleSpec`
- **PartialEq**
  - `fn eq(self: &Self, other: &ModuleSpec) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::types::NamedBinding

*Struct*

A named host binding (`[secrets]`, `[work_fs]`, `[oauth]`, `[[kv_namespaces]]`).

The `binding` key names the property on `env`; it defaults per table
(`SECRETS`, `WORK_FS`, `OAUTH`, `KV`) when omitted.

**Fields:**
- `binding: String` - Binding name exposed on `env`; empty means the table default.

**Methods:**

- `fn name_or<'a>(self: &'a Self, default: &'a str) -> &'a str` - Resolves the binding name, falling back to `default`.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> NamedBinding`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NamedBinding) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> NamedBinding`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_manifest::types::NetworkCapabilities

*Struct*

`[capabilities.network]` — mode plus optional workerd host allowlist.

Defaults to deny with an empty domain list. Unknown keys are rejected.

**Fields:**
- `mode: NetworkMode` - Whether the guest may open outbound connections / isolate fetches.
- `domains: Vec<String>` - Workerd-only: initial-request host allowlist for isolate egress.
- `tcp: Vec<TcpCapability>` - Raw TCP (`connect`) grants. Fetch allowlists do not imply these.
- `allow_undeclared_public_redirects: bool` - When true, fetch redirects may target undeclared **public** hosts.
- `address_cidrs: Vec<String>` - Explicit CIDR grants beyond the public Internet (no blanket `private`).

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> NetworkCapabilities`
- **PartialEq**
  - `fn eq(self: &Self, other: &NetworkCapabilities) -> bool`
- **Default**
  - `fn default() -> Self`



## bookclerk_plugin_manifest::types::NetworkMode

*Enum*

Network mode declared in `[capabilities.network]`.

Defaults to `NetworkMode::Deny`. See `docs/plugins.md` for native vs
workerd semantics.

**Variants:**
- `Deny` - No outbound network (default). Jail and isolate both refuse guest
- `Outbound` - Coarse jail outbound (native) or workerd isolate egress.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &NetworkMode) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NetworkMode`
- **Default**
  - `fn default() -> NetworkMode`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_manifest::types::OidcClientToml

*Struct*

One `[[oidc.clients]]` row (snake_case TOML).

**Fields:**
- `client_id: String` - OAuth `client_id`.
- `display_name: String` - Operator-facing card title.
- `callback_path: String` - Path appended to the plugin origin.
- `public_client: bool` - Public PKCE when true (default).
- `default_scopes: Vec<String>` - Scopes for first materialization.
- `issue_refresh_token: bool` - Whether new rows may issue refresh tokens (default true).
- `origin_config_key: String` - Dotted config key for the player origin.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> OidcClientToml`
- **PartialEq**
  - `fn eq(self: &Self, other: &OidcClientToml) -> bool`



## bookclerk_plugin_manifest::types::OidcManifest

*Struct*

`[[oidc.clients]]` install-time OIDC AS templates.

**Fields:**
- `clients: Vec<OidcClientToml>` - Client templates materialized when this plugin is installed.

**Methods:**

- `fn is_empty(self: &Self) -> bool` - True when no clients are declared.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> OidcManifest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &OidcManifest) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> OidcManifest`



## bookclerk_plugin_manifest::types::PluginFamily

*Enum*

Handler family — how the daemon, CLI, and UI group plugins and which
`config.toml` prefix holds their settings.

Derived from the exported [`Entrypoint`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedentrypoint)s and declared triggers (see
`PluginManifest::families`); never written to `plugin.toml`. Wire values
are lowercase (`source`, `integration`, `output`, `database`).

**Variants:**
- `Source` - Storefront / library source (`Storefront` entrypoint).
- `Integration` - Event consumer, job runner, remote library, CLI, or OIDC bridge.
- `Output` - Destination / output backend (`Storage` entrypoint).
- `Database` - Library database backend (`DatabaseAdapter` entrypoint).

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the lowercase wire name used in API paths and config prefixes.
- `fn parse(name: &str) -> Option<Self>` - Parses a wire name produced by `Self::as_str`.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginFamily`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Ord**
  - `fn cmp(self: &Self, other: &PluginFamily) -> $crate::cmp::Ordering`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginFamily) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &PluginFamily) -> $crate::option::Option<$crate::cmp::Ordering>`



## bookclerk_plugin_manifest::types::PluginManifest

*Struct*

On-disk plugin descriptor (`plugin.toml`, `api_version = 3`).

Root table for install / discovery, shaped after `wrangler.jsonc`: the
plugin exports named [`Entrypoint`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedentrypoint)s, declares triggers for its default
entrypoint (`[triggers]`, `[[events.consumers]]`), and lists the host
bindings it expects on `env` (`[vars]`, `[secrets]`, `[[kv_namespaces]]`,
`[work_fs]`, `[oauth]`, `[[databases]]`, `[[events.producers]]`). Parse
with `Self::parse` (deserialize + `Self::validate`). Unknown keys are
rejected.

# Validation highlights

- `api_version` must equal [`bookclerk_plugin_abi::PRODUCT_API_VERSION`](../bookclerk_plugin_abi/limits.md#bookclerk_plugin_abilimitsproduct_api_version)
- `id` must pass [`crate::validate_plugin_id`](id.md#bookclerk_plugin_manifestidvalidate_plugin_id)
- at least one entrypoint or trigger must be declared
- native requires `command`; workerd requires `[workerd]` with date + main
- `domains` forbidden on native (use `tcp` for mediated sockets); required for workerd + outbound
- optional `logo` must pass [`crate::validate_logo`](logo.md#bookclerk_plugin_manifestlogovalidate_logo)

**Fields:**
- `api_version: u32` - ABI / schema version. Must equal [`bookclerk_plugin_abi::PRODUCT_API_VERSION`](../bookclerk_plugin_abi/limits.md#bookclerk_plugin_abilimitsproduct_api_version).
- `id: String` - Display / CLI alias (`[a-z0-9_]{2,32}` grammar). Not a privilege.
- `name: Option<String>` - Optional human-readable display name for Settings / Accounts UI.
- `version: Option<String>` - Package version. Required for enrollment and `bookclerk plugin check`
- `logo: Option<String>` - Settings / UI logo: `https://…` URL or relative image path under the
- `runtime: PluginRuntimeKind` - Guest runtime (`native` default, or `workerd`). Selects the backend
- `command: Option<std::path::PathBuf>` - Native executable path relative to the install root (required when
- `args: Vec<String>` - Extra argv passed after `command` for native guests.
- `entrypoints: Vec<Entrypoint>` - Named entrypoints this plugin exports (`["storefront", "cli"]`).
- `workerd: Option<WorkerdRuntimeManifest>` - Workerd isolate config (required when `runtime = "workerd"`).
- `modules: Vec<ModuleSpec>` - Optional module list for workerd packages (`[[modules]]`).
- `triggers: TriggersManifest` - Default-entrypoint triggers other than events (`[triggers]`).
- `events: EventsManifest` - Event consumers (triggers) and producers (publish grants).
- `databases: Vec<DatabaseBindingManifest>` - Isolated plugin-owned database bindings (`[[databases]]`).
- `vars: Option<std::collections::BTreeMap<String, toml::Value>>` - Operator config binding (`[vars]`). Declares that the guest receives
- `settings: Vec<bookclerk_plugin_abi::SettingSpec>` - Operator settings schema (`[[settings]]`): the keys an instance
- `secrets: Option<NamedBinding>` - Sealed secrets binding (`[secrets]`).
- `kv_namespaces: Vec<KvNamespaceManifest>` - Per-plugin key/value storage bindings (`[[kv_namespaces]]`).
- `queues: QueuesManifest` - Workers Queues (`[queues]` with `[[queues.producers]]` /
- `work_fs: Option<NamedBinding>` - Host-mediated work filesystem binding (`[work_fs]`).
- `oauth: Option<NamedBinding>` - OAuth callback tunnel binding (`[oauth]`).
- `capabilities: CapabilitiesManifest` - Declared network capability (consent has no Wrangler analogue).
- `cli: Option<bookclerk_plugin_abi::CliSchema>` - Optional CLI schema advertised to `bookclerk plugins <id>` (from
- `oidc: OidcManifest` - Optional Bookclerk-as-IdP client templates (`[[oidc.clients]]`).

**Methods:**

- `fn declares_python(self: &Self) -> bool` - True when the manifest itself declares a Python workerd guest.
- `fn unimplemented_surface(self: &Self) -> Option<String>` - Spawn/load error when the manifest declares a surface the host cannot
- `fn settings_schema(self: &Self) -> SettingsSchema` - Operator settings schema declared by `[[settings]]`.
- `fn config_options(self: &Self) -> Vec<ConfigOption>` - `describe().configOptions` derived from `[[settings]]`.
- `fn bindings(self: &Self) -> BindingCapabilities` - Resolves the declared host bindings into one flat view.
- `fn has_entrypoint(self: &Self, entrypoint: Entrypoint) -> bool` - True when the plugin exports `entrypoint`.
- `fn consumes_events(self: &Self) -> bool` - True when the default entrypoint has an `event(batch)` trigger.
- `fn runs_jobs(self: &Self) -> bool` - True when the default entrypoint has a `job(controller)` trigger.
- `fn produces_events(self: &Self) -> bool` - True when the plugin declares any publishable event type.
- `fn consumes_queues(self: &Self) -> bool` - True when the default entrypoint has a `queue(batch)` trigger.
- `fn produces_queues(self: &Self) -> bool` - True when the plugin declares any queue send binding.
- `fn queue_consumer_names(self: &Self) -> Vec<String>` - Queue names the default entrypoint consumes (`[[queues.consumers]]`).
- `fn queue_producer_names(self: &Self) -> Vec<String>` - Queue names this plugin may send to (`[[queues.producers]]`),
- `fn families(self: &Self) -> Vec<PluginFamily>` - Handler families this plugin belongs to, in
- `fn primary_family(self: &Self) -> PluginFamily` - Primary handler family: the first of `Self::families`.
- `fn producer_types(self: &Self) -> Vec<String>` - Event types this plugin may publish (from `[[events.producers]]`).
- `fn capabilities(self: &Self) -> PluginCapabilities` - Typed capability declaration this manifest implies.
- `fn binding_names(self: &Self) -> Vec<String>` - Names of the non-database bindings this manifest exposes on `env`.
- `fn consent_domains(self: &Self) -> Vec<String>` - Returns domains for workerd network consent UI.
- `fn logo_kind(self: &Self) -> Result<Option<crate::LogoKind>>` - Classifies the optional `logo` field after validation.
- `fn parse(text: &str) -> Result<Self>` - Parses and validates a `plugin.toml` document.
- `fn package_version(self: &Self) -> Result<String>` - Runs semantic validation after deserialize.
- `fn validate(self: &Self) -> Result<()>` - Checks id grammar, `api_version`, logo, runtime-specific required
- `fn spawn_command(self: &Self) -> Option<&PathBuf>` - Resolves the process path to spawn for this guest.
- `fn jail_network_need(self: &Self) -> JailNetworkNeed` - Maps manifest network + oauth binding to OS jail network policy.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginManifest`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginManifest) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::types::PluginRuntimeKind

*Enum*

Guest runtime selection (`runtime` key in `plugin.toml`).

Defaults to `PluginRuntimeKind::Native` when omitted.

**Variants:**
- `Native` - OS binary speaking the Workers RPC ABI, spawned through `bookclerk-jail`.
- `Workerd` - Author modules loaded by first-party `bookclerk-workerd` (one jail +

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PluginRuntimeKind) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PluginRuntimeKind`
- **Default**
  - `fn default() -> PluginRuntimeKind`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_manifest::types::QueueConsumer

*Struct*

One `[[queues.consumers]]` row — a durable queue trigger for the default
entrypoint's `queue(batch)` handler.

**Fields:**
- `queue: String` - Queue name to consume.
- `max_batch_size: u32` - Most messages in one `queue(batch)` call (default
- `max_batch_timeout: u32` - Longest wait, in seconds, to fill a batch (default
- `max_retries: u32` - Redeliveries before a message is dead-lettered (default
- `dead_letter_queue: Option<String>` - Queue that receives messages after `max_retries` is exhausted.
- `retry_delay: u32` - Seconds a retried message waits before redelivery (default `0`).
- `max_concurrency: Option<u32>` - Most concurrent `queue(batch)` invocations across the cluster; `None`

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> QueueConsumer`
- **PartialEq**
  - `fn eq(self: &Self, other: &QueueConsumer) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::types::QueueProducer

*Struct*

One `[[queues.producers]]` row — a queue the plugin may send to through
the named `env` binding.

**Fields:**
- `queue: String` - Queue name (`[a-z0-9][a-z0-9._-]*`; `core.` and `node.` are reserved).
- `binding: String` - Binding name on `env` (`[A-Z][A-Z0-9_]*`); required, as in Wrangler.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> QueueProducer`
- **PartialEq**
  - `fn eq(self: &Self, other: &QueueProducer) -> bool`



## bookclerk_plugin_manifest::types::QueuesManifest

*Struct*

`[queues]` — Workers Queues producers (send bindings) and consumers
(`queue(batch)` triggers).

**Fields:**
- `producers: Vec<QueueProducer>` - Queues the plugin may send to, one `env.<BINDING>` each.
- `consumers: Vec<QueueConsumer>` - Queues delivered to the default entrypoint's `queue(batch)`.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> QueuesManifest`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> QueuesManifest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &QueuesManifest) -> bool`



## bookclerk_plugin_manifest::types::RESERVED_QUEUE_PREFIXES

*Constant*: `[&str; 2]`

Queue-name prefixes only the host may use (`core.events`, `node.<id>`).



## bookclerk_plugin_manifest::types::TcpCapability

*Struct*

One raw TCP grant: hostname or wildcard plus allowed destination ports.

**Fields:**
- `host: String` - Hostname or `*.` prefix wildcard (IDNA-normalized at validate time).
- `ports: Vec<u16>` - Destination ports that may be dialed. Empty is fail-closed (no ports).

**Traits:** Eq

**Trait Implementations:**

- **Ord**
  - `fn cmp(self: &Self, other: &TcpCapability) -> $crate::cmp::Ordering`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TcpCapability`
- **PartialEq**
  - `fn eq(self: &Self, other: &TcpCapability) -> bool`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &TcpCapability) -> $crate::option::Option<$crate::cmp::Ordering>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::types::TriggersManifest

*Struct*

`[triggers]` — handlers on the default entrypoint the host invokes.

Mirrors `wrangler.jsonc` `triggers`; Bookclerk's triggers are durable
commands (`job(controller)`) rather than crons. Event triggers live under
`[[events.consumers]]`.

**Fields:**
- `jobs: Vec<String>` - Command types the default entrypoint's `job(controller)` handles

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> TriggersManifest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &TriggersManifest) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TriggersManifest`



## bookclerk_plugin_manifest::types::WorkerdLimits

*Struct*

Optional workerd resource limits under `[workerd.limits]`.

Local workerd does **not** Cap'n Proto-emit `cpuMs` / `subRequests`. Bookclerk
clamps these values, injects `subrequests` into egress policy JSON via
`crate::EgressPolicy::from_manifest`, and maps `cpu_ms` onto jail Spec CPU
rate (plus memory/process ceilings) for OS enforcement.

**Fields:**
- `cpu_ms: Option<u32>` - Soft CPU budget in milliseconds (unset/`0` → `Self::DEFAULT_CPU_MS`,
- `subrequests: Option<u32>` - Soft outbound fetch budget (unset/`0` → `Self::DEFAULT_SUBREQUESTS`,

**Methods:**

- `fn effective(self: &Self) -> EffectiveWorkerdLimits` - Resolves concrete limits: unset/`0` → defaults, then clamp to hard caps.
- `fn clamp(self: &Self) -> EffectiveWorkerdLimits` - Alias for `Self::effective`.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> WorkerdLimits`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> WorkerdLimits`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &WorkerdLimits) -> bool`



## bookclerk_plugin_manifest::types::WorkerdRuntimeManifest

*Struct*

`[workerd]` — WorkerCode-equivalent isolate configuration.

Required when `PluginRuntimeKind::Workerd`. Maps closely to Cloudflare
Worker module config consumed by `bookclerk-workerd`.

**Fields:**
- `compatibility_date: String` - Cloudflare compatibility date (`YYYY-MM-DD`). A date newer than the pin
- `compatibility_flags: Vec<String>` - Author compatibility flags. Only the Python pair is allowed, and only
- `main_module: String` - Entrypoint module filename relative to the modules tree (e.g. `index.js`).
- `modules_dir: String` - Directory containing worker modules (default `"modules"`).
- `entrypoint: String` - Named export used as the Worker entrypoint (default `"default"`).
- `limits: WorkerdLimits` - Optional CPU / subrequest budgets (host-clamped; see [`WorkerdLimits`](types.md#bookclerk_plugin_manifesttypesworkerdlimits)).

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> WorkerdRuntimeManifest`
- **PartialEq**
  - `fn eq(self: &Self, other: &WorkerdRuntimeManifest) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::types::entrypoint_family

*Function*

Handler family an [`Entrypoint`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedentrypoint) belongs to.

# Returns

The [`PluginFamily`](types.md#bookclerk_plugin_manifesttypespluginfamily) whose `config.toml` prefix and API group hold plugins
exporting this entrypoint.

```rust
fn entrypoint_family(entrypoint: Entrypoint) -> PluginFamily
```



## bookclerk_plugin_manifest::types::is_host_event_type

*Function*

True when `event_type` is in [`HOST_EVENT_TYPES`](types.md#bookclerk_plugin_manifesttypeshost_event_types).

```rust
fn is_host_event_type(event_type: &str) -> bool
```



## bookclerk_plugin_manifest::types::is_reserved_queue_name

*Function*

True when `name` starts with a [`RESERVED_QUEUE_PREFIXES`](types.md#bookclerk_plugin_manifesttypesreserved_queue_prefixes) entry.

```rust
fn is_reserved_queue_name(name: &str) -> bool
```



## bookclerk_plugin_manifest::types::is_valid_database_binding_name

*Function*

True when `name` is a valid Workers-style binding name (`[A-Z][A-Z0-9_]*`).

```rust
fn is_valid_database_binding_name(name: &str) -> bool
```



## bookclerk_plugin_manifest::types::is_valid_queue_name

*Function*

True when `name` is a valid queue name: `[a-z0-9][a-z0-9._-]*`, at most
[`MAX_QUEUE_NAME_LEN`](types.md#bookclerk_plugin_manifesttypesmax_queue_name_len) characters.

```rust
fn is_valid_queue_name(name: &str) -> bool
```



