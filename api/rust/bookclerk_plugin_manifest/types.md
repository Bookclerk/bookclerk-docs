# bookclerk_plugin_manifest::types

`plugin.toml` types: manifest root and nested capability / workerd tables.

These structs are the serde projection of the install descriptor. Unknown
keys are rejected (`deny_unknown_fields`) so typos fail at parse time.
Semantic rules that cannot be expressed in serde alone live in
[`PluginManifest::validate`](#pluginmanifestvalidate). Product narrative: `docs/plugins.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`TcpCapability`](#tcpcapability) | struct | One raw TCP grant: hostname or wildcard plus allowed destination ports. |
| [`NetworkCapabilities`](#networkcapabilities) | struct | `[capabilities.network]` — mode plus optional workerd host allowlist. |
| [`BindingCapabilities`](#bindingcapabilities) | struct | Host bindings a plugin declared, resolved from the v3 manifest tables. |
| [`DatabaseBindingManifest`](#databasebindingmanifest) | struct | One `[[databases]]` row — an isolated plugin-owned database binding. |
| [`NamedBinding`](#namedbinding) | struct | A named host binding (`[secrets]`, `[work_fs]`, `[oauth]`, `[[kv_namespaces]]`). |
| [`KvNamespaceManifest`](#kvnamespacemanifest) | struct | One `[[kv_namespaces]]` row — a durable key/value namespace the host serves from the library database, scoped to this plugin. |
| [`TriggersManifest`](#triggersmanifest) | struct | `[triggers]` — handlers on the default entrypoint the host invokes. |
| [`CapabilitiesManifest`](#capabilitiesmanifest) | struct | Full `[capabilities]` table required on every manifest. |
| [`EventConsumer`](#eventconsumer) | struct | One `[[events.consumers]]` row — a durable domain-event trigger for the default entrypoint's `event(batch)` handler. |
| [`EventProducer`](#eventproducer) | struct | One `[[events.producers]]` row — an event type the plugin may publish through its `EVENTS` binding. |
| [`EventsManifest`](#eventsmanifest) | struct | `[events]` — consumers (triggers) and producers (publish grants). |
| [`QueuesManifest`](#queuesmanifest) | struct | `[queues]` — Workers Queues producers (send bindings) and consumers (`queue(batch)` triggers). |
| [`QueueProducer`](#queueproducer) | struct | One `[[queues.producers]]` row — a queue the plugin may send to through the named `env` binding. |
| [`QueueConsumer`](#queueconsumer) | struct | One `[[queues.consumers]]` row — a durable queue trigger for the default entrypoint's `queue(batch)` handler. |
| [`WorkerdRuntimeManifest`](#workerdruntimemanifest) | struct | `[workerd]` — WorkerCode-equivalent isolate configuration. |
| [`WorkerdLimits`](#workerdlimits) | struct | Optional workerd resource limits under `[workerd.limits]`. |
| [`EffectiveWorkerdLimits`](#effectiveworkerdlimits) | struct | Concrete limits after applying host defaults and hard caps. |
| [`ModuleSpec`](#modulespec) | struct | One module entry in `[[modules]]` (workerd script packages). |
| [`PluginManifest`](#pluginmanifest) | struct | On-disk plugin descriptor (`plugin.toml`, `api_version = 3`). |
| [`OidcManifest`](#oidcmanifest) | struct | `[[oidc.clients]]` install-time OIDC AS templates. |
| [`OidcClientToml`](#oidcclienttoml) | struct | One `[[oidc.clients]]` row (snake_case TOML). |
| [`PluginFamily`](#pluginfamily) | enum | Handler family — how the daemon, CLI, and UI group plugins and which `config.toml` prefix holds their settings. |
| [`PluginRuntimeKind`](#pluginruntimekind) | enum | Guest runtime selection (`runtime` key in `plugin.toml`). |
| [`NetworkMode`](#networkmode) | enum | Network mode declared in `[capabilities.network]`. |
| [`JailNetworkNeed`](#jailnetworkneed) | enum | OS jail network capability derived from manifest network + oauth binding. |
| [`entrypoint_family`](#entrypoint_family) | function | Handler family an `Entrypoint` belongs to. |
| [`is_valid_database_binding_name`](#is_valid_database_binding_name) | function | True when `name` is a valid Workers-style binding name (`A-Z*`). |
| [`is_host_event_type`](#is_host_event_type) | function | True when `event_type` is in `HOST_EVENT_TYPES`. |
| [`is_valid_queue_name`](#is_valid_queue_name) | function | True when `name` is a valid queue name: `a-z0-9*`, at most `MAX_QUEUE_NAME_LEN` characters. |
| [`is_reserved_queue_name`](#is_reserved_queue_name) | function | True when `name` starts with a `RESERVED_QUEUE_PREFIXES` entry. |
| [`ALL_ENTRYPOINTS`](#all_entrypoints) | constant | Every entrypoint in declaration order (mirrors the Cap'n Proto enum). |
| [`MAX_DATABASE_BINDINGS`](#max_database_bindings) | constant | Maximum named database bindings one plugin may declare. |
| [`MAX_DATABASE_BINDING_NAME_LEN`](#max_database_binding_name_len) | constant | Maximum length of one database binding name. |
| [`DEFAULT_KV_MAX_VALUE_BYTES`](#default_kv_max_value_bytes) | constant | Default `max_value_bytes` for `[[kv_namespaces]]`: 64 KiB. |
| [`MAX_KV_VALUE_BYTES`](#max_kv_value_bytes) | constant | Largest `max_value_bytes` a namespace may declare: 1 MiB (the ABI cap). |
| [`DEFAULT_KV_MAX_KEYS`](#default_kv_max_keys) | constant | Default `max_keys` for `[[kv_namespaces]]`. |
| [`MAX_KV_KEYS`](#max_kv_keys) | constant | Largest `max_keys` a namespace may declare. |
| [`DEFAULT_SECRETS_BINDING`](#default_secrets_binding) | constant | Default `binding` for `[secrets]`. |
| [`DEFAULT_WORK_FS_BINDING`](#default_work_fs_binding) | constant | Default `binding` for `[work_fs]`. |
| [`DEFAULT_OAUTH_BINDING`](#default_oauth_binding) | constant | Default `binding` for `[oauth]`. |
| [`DEFAULT_KV_BINDING`](#default_kv_binding) | constant | Default `binding` for `[[kv_namespaces]]`. |
| [`DEFAULT_EVENTS_BINDING`](#default_events_binding) | constant | Default `binding` for `[[events.producers]]`. |
| [`CONFIG_BINDING`](#config_binding) | constant | Binding name the host uses for operator config (`[vars]`). |
| [`HOST_EVENT_TYPES`](#host_event_types) | constant | Domain event types only the host emits. |
| [`MAX_QUEUE_NAME_LEN`](#max_queue_name_len) | constant | Maximum length of a queue name. |
| [`RESERVED_QUEUE_PREFIXES`](#reserved_queue_prefixes) | constant | Queue-name prefixes only the host may use (`core.events`, `node.<id>`). |
| [`DEFAULT_QUEUE_MAX_BATCH_SIZE`](#default_queue_max_batch_size) | constant | Default `max_batch_size` for `[[queues.consumers]]`. |
| [`MAX_QUEUE_BATCH_SIZE`](#max_queue_batch_size) | constant | Largest `max_batch_size` a consumer may declare. |
| [`DEFAULT_QUEUE_MAX_BATCH_TIMEOUT_SECS`](#default_queue_max_batch_timeout_secs) | constant | Default `max_batch_timeout` (seconds) for `[[queues.consumers]]`. |
| [`MAX_QUEUE_BATCH_TIMEOUT_SECS`](#max_queue_batch_timeout_secs) | constant | Largest `max_batch_timeout` (seconds) a consumer may declare. |
| [`DEFAULT_QUEUE_MAX_RETRIES`](#default_queue_max_retries) | constant | Default `max_retries` for `[[queues.consumers]]`. |
| [`MAX_QUEUE_RETRIES`](#max_queue_retries) | constant | Largest `max_retries` a consumer may declare. |
| [`MAX_QUEUE_RETRY_DELAY_SECS`](#max_queue_retry_delay_secs) | constant | Largest `retry_delay` (seconds) a consumer may declare: 12 hours. |

## Re-exports

- [`bookclerk_plugin_abi::Entrypoint`](../bookclerk_plugin_abi/generated.md#entrypoint)
- [`bookclerk_plugin_abi::EventConsumerSpec`](../bookclerk_plugin_abi/generated.md#eventconsumerspec)
- [`bookclerk_plugin_abi::PluginCapabilities`](../bookclerk_plugin_abi/generated.md#plugincapabilities)

## Structs

### TcpCapability

```rust
pub struct TcpCapability {
    pub host: String,
    pub ports: Vec<u16>,
}
```

One raw TCP grant: hostname or wildcard plus allowed destination ports.

**Fields**

- `host`: Hostname or `*.` prefix wildcard (IDNA-normalized at validate time).
- `ports`: Destination ports that may be dialed. Empty is fail-closed (no ports).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### NetworkCapabilities

```rust
pub struct NetworkCapabilities {
    pub mode: NetworkMode,
    pub domains: Vec<String>,
    pub tcp: Vec<TcpCapability>,
    pub allow_undeclared_public_redirects: bool,
    pub address_cidrs: Vec<String>,
}
```

`[capabilities.network]` — mode plus optional workerd host allowlist.

Defaults to deny with an empty domain list. Unknown keys are rejected.

**Fields**

- `mode`: Whether the guest may open outbound connections / isolate fetches.
- `domains`: Workerd-only: initial-request host allowlist for isolate egress.
  
  Entries are exact hosts or `*.` prefix wildcards; validated with IDNA
  ToASCII at [`PluginManifest::validate`](#pluginmanifestvalidate). Must be empty for
  `runtime = "native"` and non-empty for workerd + outbound.
- `tcp`: Raw TCP (`connect`) grants. Fetch allowlists do not imply these.
- `allow_undeclared_public_redirects`: When true, fetch redirects may target undeclared **public** hosts.
  
  Address-space policy still applies: loopback / RFC1918 / link-local /
  metadata never piggy-back on this flag.
- `address_cidrs`: Explicit CIDR grants beyond the public Internet (no blanket `private`).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### BindingCapabilities

```rust
pub struct BindingCapabilities {
    pub config: bool,
    pub secrets: bool,
    pub plugin_kv: bool,
    pub work_fs: bool,
    pub oauth: bool,
    pub databases: Vec<String>,
}
```

Host bindings a plugin declared, resolved from the v3 manifest tables.

Derived view produced by [`PluginManifest::bindings`](#pluginmanifestbindings); `[vars]`,
`[secrets]`, `[[kv_namespaces]]`, `[work_fs]`, `[oauth]`, and
`[[databases]]` each map onto one flag or list here so host code can gate
jail layout, consent, and env injection uniformly.

**Fields**

- `config`: Guest may read plugin config delivered by the host (`[vars]`).
- `secrets`: Guest may read sealed secrets / credentials via host bindings (`[secrets]`).
- `plugin_kv`: `[[kv_namespaces]]` is declared: the host grants one durable
  key/value namespace per row, served from the library database. This
  is a different capability from [`Self::databases`](#bindingcapabilities).
- `work_fs`: Guest may use host-mediated work filesystem (`[work_fs]`).
- `oauth`: Guest needs an OAuth-style callback tunnel (`[oauth]`).
  
  With native outbound, this upgrades jail network need to
  [`JailNetworkNeed::Listen`](#jailnetworkneed).
- `databases`: Named plugin-owned database bindings (`[[databases]] binding = "DB"`).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### DatabaseBindingManifest

```rust
pub struct DatabaseBindingManifest {
    pub binding: String,
}
```

One `[[databases]]` row — an isolated plugin-owned database binding.

Each name binds a database provisioned by the active database adapter —
separate from the Bookclerk library and from every other plugin. Names
must be `[A-Z][A-Z0-9_]*`, unique, and at most
[`MAX_DATABASE_BINDING_NAME_LEN`](#max_database_binding_name_len) chars; the operator consents to each
binding before enable.

**Fields**

- `binding`: Binding name exposed on `env` (Workers-style, e.g. `"DB"`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NamedBinding

```rust
pub struct NamedBinding {
    pub binding: String,
}
```

A named host binding (`[secrets]`, `[work_fs]`, `[oauth]`, `[[kv_namespaces]]`).

The `binding` key names the property on `env`; it defaults per table
(`SECRETS`, `WORK_FS`, `OAUTH`, `KV`) when omitted.

**Fields**

- `binding`: Binding name exposed on `env`; empty means the table default.

#### NamedBinding::name_or

```rust
pub fn name_or<'a>(&'a self, default: &'a str) -> &'a str
```

Resolves the binding name, falling back to `default`.

##### Returns

The author's `binding` when non-empty, else `default`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### KvNamespaceManifest

```rust
pub struct KvNamespaceManifest {
    pub binding: String,
    pub namespace: String,
    pub max_value_bytes: u32,
    pub max_keys: u32,
}
```

One `[[kv_namespaces]]` row — a durable key/value namespace the host
serves from the library database, scoped to this plugin.

The `binding` key names the property on `env` (default `KV`). The
`namespace` defaults to the binding name lowercased, so two rows are
distinct stores unless they name the same namespace on purpose. Values
live in the cluster: an instance moved to another host reads its own keys.

**Fields**

- `binding`: Binding name exposed on `env`; empty means `KV`.
- `namespace`: Namespace the binding is scoped to (`[a-z0-9][a-z0-9._-]*`); empty
  means the binding name lowercased.
- `max_value_bytes`: Largest value in bytes (default [`DEFAULT_KV_MAX_VALUE_BYTES`](#default_kv_max_value_bytes), at
  most [`MAX_KV_VALUE_BYTES`](#max_kv_value_bytes)).
- `max_keys`: Most live keys in the namespace (default [`DEFAULT_KV_MAX_KEYS`](#default_kv_max_keys), at
  most [`MAX_KV_KEYS`](#max_kv_keys)); a `put` past the cap fails with `forbidden`.

#### KvNamespaceManifest::name_or

```rust
pub fn name_or<'a>(&'a self, default: &'a str) -> &'a str
```

Resolves the binding name, falling back to `default`.

#### KvNamespaceManifest::binding_name

```rust
pub fn binding_name(&self) -> &str
```

Binding name on `env` (`KV` when omitted).

#### KvNamespaceManifest::namespace_name

```rust
pub fn namespace_name(&self) -> String
```

Namespace the binding is scoped to: the declared name, or the binding
name lowercased.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### TriggersManifest

```rust
pub struct TriggersManifest {
    pub jobs: Vec<String>,
}
```

`[triggers]` — handlers on the default entrypoint the host invokes.

Mirrors `wrangler.jsonc` `triggers`; Bookclerk's triggers are durable
commands (`job(controller)`) rather than crons. Event triggers live under
`[[events.consumers]]`.

**Fields**

- `jobs`: Command types the default entrypoint's `job(controller)` handles
  (snake_case, e.g. `stream_copy`). Empty means no job trigger.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CapabilitiesManifest

```rust
pub struct CapabilitiesManifest {
    pub network: NetworkCapabilities,
}
```

Full `[capabilities]` table required on every manifest.

Only `network` remains here in v3 (consent has no Wrangler analogue);
bindings and triggers moved to top-level Wrangler-shaped tables.

**Fields**

- `network`: Network mode and optional workerd domain allowlist.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventConsumer

```rust
pub struct EventConsumer {
    pub event_type: String,
    pub schema_versions: Vec<u32>,
    pub supports_suspend: bool,
    pub resource_class: String,
    pub filter: Option<Value>,
    pub max_retries: Option<u32>,
}
```

One `[[events.consumers]]` row — a durable domain-event trigger for the
default entrypoint's `event(batch)` handler.

**Fields**

- `event_type`: Versioned event type (`book_acquired`, …).
- `schema_versions`: Schema versions this guest can consume (default `[1]`).
- `supports_suspend`: Whether `EventResult::Suspended` is supported for this type.
- `resource_class`: Concurrency class copied onto deliveries (default `"network"`).
- `filter`: Optional host-owned payload object filter (top-level key equality).
- `max_retries`: Redelivery attempts before dead-lettering; `None` uses the host default.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventProducer

```rust
pub struct EventProducer {
    pub event_type: String,
    pub binding: String,
}
```

One `[[events.producers]]` row — an event type the plugin may publish
through its `EVENTS` binding.

**Fields**

- `event_type`: Event type the plugin publishes (snake_case).
- `binding`: Publisher binding name on `env` (default `EVENTS`).

#### EventProducer::binding_name

```rust
pub fn binding_name(&self) -> &str
```

Resolves the publisher binding name (default [`DEFAULT_EVENTS_BINDING`](#default_events_binding)).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventsManifest

```rust
pub struct EventsManifest {
    pub consumers: Vec<EventConsumer>,
    pub producers: Vec<EventProducer>,
}
```

`[events]` — consumers (triggers) and producers (publish grants).

**Fields**

- `consumers`: Event types delivered to the default entrypoint's `event(batch)`.
- `producers`: Event types the plugin may publish.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QueuesManifest

```rust
pub struct QueuesManifest {
    pub producers: Vec<QueueProducer>,
    pub consumers: Vec<QueueConsumer>,
}
```

`[queues]` — Workers Queues producers (send bindings) and consumers
(`queue(batch)` triggers).

**Fields**

- `producers`: Queues the plugin may send to, one `env.<BINDING>` each.
- `consumers`: Queues delivered to the default entrypoint's `queue(batch)`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QueueProducer

```rust
pub struct QueueProducer {
    pub queue: String,
    pub binding: String,
}
```

One `[[queues.producers]]` row — a queue the plugin may send to through
the named `env` binding.

**Fields**

- `queue`: Queue name (`[a-z0-9][a-z0-9._-]*`; `core.` and `node.` are reserved).
- `binding`: Binding name on `env` (`[A-Z][A-Z0-9_]*`); required, as in Wrangler.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QueueConsumer

```rust
pub struct QueueConsumer {
    pub queue: String,
    pub max_batch_size: u32,
    pub max_batch_timeout: u32,
    pub max_retries: u32,
    pub dead_letter_queue: Option<String>,
    pub retry_delay: u32,
    pub max_concurrency: Option<u32>,
}
```

One `[[queues.consumers]]` row — a durable queue trigger for the default
entrypoint's `queue(batch)` handler.

**Fields**

- `queue`: Queue name to consume.
- `max_batch_size`: Most messages in one `queue(batch)` call (default
  [`DEFAULT_QUEUE_MAX_BATCH_SIZE`](#default_queue_max_batch_size), at most [`MAX_QUEUE_BATCH_SIZE`](#max_queue_batch_size)).
- `max_batch_timeout`: Longest wait, in seconds, to fill a batch (default
  [`DEFAULT_QUEUE_MAX_BATCH_TIMEOUT_SECS`](#default_queue_max_batch_timeout_secs)).
- `max_retries`: Redeliveries before a message is dead-lettered (default
  [`DEFAULT_QUEUE_MAX_RETRIES`](#default_queue_max_retries)).
- `dead_letter_queue`: Queue that receives messages after `max_retries` is exhausted.
- `retry_delay`: Seconds a retried message waits before redelivery (default `0`).
- `max_concurrency`: Most concurrent `queue(batch)` invocations across the cluster; `None`
  lets the host choose.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WorkerdRuntimeManifest

```rust
pub struct WorkerdRuntimeManifest {
    pub compatibility_date: String,
    pub compatibility_flags: Vec<String>,
    pub main_module: String,
    pub modules_dir: String,
    pub entrypoint: String,
    pub limits: WorkerdLimits,
}
```

`[workerd]` — WorkerCode-equivalent isolate configuration.

Required when [`PluginRuntimeKind::Workerd`](#pluginruntimekind). Maps closely to Cloudflare
Worker module config consumed by `bookclerk-workerd`.

**Fields**

- `compatibility_date`: Cloudflare compatibility date (`YYYY-MM-DD`). A date newer than the pin
  ([`crate::WORKERD_PIN_COMPAT_DATE`](workerd_contract.md#workerd_pin_compat_date)) warns and loads at that pin.
- `compatibility_flags`: Author compatibility flags. Only the Python pair is allowed, and only
  when the guest has a Python module. See
  [`crate::PYTHON_COMPATIBILITY_FLAGS`](workerd_contract.md#python_compatibility_flags).
- `main_module`: Entrypoint module filename relative to the modules tree (e.g. `index.js`).
- `modules_dir`: Directory containing worker modules (default `"modules"`).
- `entrypoint`: Named export used as the Worker entrypoint (default `"default"`).
- `limits`: Optional CPU / subrequest budgets (host-clamped; see [`WorkerdLimits`](#workerdlimits)).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WorkerdLimits

```rust
pub struct WorkerdLimits {
    pub cpu_ms: Option<u32>,
    pub subrequests: Option<u32>,
}
```

Optional workerd resource limits under `[workerd.limits]`.

Local workerd does **not** Cap'n Proto-emit `cpuMs` / `subRequests`. Bookclerk
clamps these values, injects `subrequests` into egress policy JSON via
[`crate::EgressPolicy::from_manifest`](egress.md#egresspolicyfrom_manifest), and maps `cpu_ms` onto jail Spec CPU
rate (plus memory/process ceilings) for OS enforcement.

**Fields**

- `cpu_ms`: Soft CPU budget in milliseconds (unset/`0` → [`Self::DEFAULT_CPU_MS`](#workerdlimitsdefault_cpu_ms),
  then clamped to [`Self::MAX_CPU_MS`](#workerdlimitsmax_cpu_ms)).
- `subrequests`: Soft outbound fetch budget (unset/`0` → [`Self::DEFAULT_SUBREQUESTS`](#workerdlimitsdefault_subrequests),
  then clamped to [`Self::MAX_SUBREQUESTS`](#workerdlimitsmax_subrequests)). Injected into
  `EGRESS_POLICY.subrequests` for workerd guests.

#### WorkerdLimits::DEFAULT_CPU_MS

```rust
pub const DEFAULT_CPU_MS: u32 = 30_000
```

Default CPU budget when unset or `0` (matches Echo examples): 30_000 ms.

#### WorkerdLimits::DEFAULT_SUBREQUESTS

```rust
pub const DEFAULT_SUBREQUESTS: u32 = 50
```

Default outbound fetch budget when unset or `0` (matches Echo examples): 50.

#### WorkerdLimits::MAX_CPU_MS

```rust
pub const MAX_CPU_MS: u32 = 120_000
```

Hard host cap for CPU budget (ms): 120_000.

#### WorkerdLimits::MAX_SUBREQUESTS

```rust
pub const MAX_SUBREQUESTS: u32 = 1_000
```

Hard host cap for outbound fetch budget: 1_000.

#### WorkerdLimits::effective

```rust
pub fn effective(&self) -> EffectiveWorkerdLimits
```

Resolves concrete limits: unset/`0` → defaults, then clamp to hard caps.

##### Returns

[`EffectiveWorkerdLimits`](#effectiveworkerdlimits) with `cpu_ms` and `subrequests` in range
`[default, max]` (or exactly the author value when already in range).

#### WorkerdLimits::clamp

```rust
pub fn clamp(&self) -> EffectiveWorkerdLimits
```

Alias for [`Self::effective`](#workerdlimitseffective).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EffectiveWorkerdLimits

```rust
pub struct EffectiveWorkerdLimits {
    pub cpu_ms: u32,
    pub subrequests: u32,
}
```

Concrete limits after applying host defaults and hard caps.

Produced by [`WorkerdLimits::effective`](#workerdlimitseffective).

**Fields**

- `cpu_ms`: Effective CPU budget in milliseconds.
- `subrequests`: Effective outbound fetch / subrequest budget.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ModuleSpec

```rust
pub struct ModuleSpec {
    pub name: String,
    pub path: String,
    pub module_type: String,
}
```

One module entry in `[[modules]]` (workerd script packages).

Used for packaging, Python detection ([`crate::manifest_needs_python`](egress.md#manifest_needs_python)),
and documenting the isolate module graph alongside `[workerd].main_module`.

**Fields**

- `name`: Module name as known to the isolate (often matches the filename).
- `path`: Path relative to the plugin package root (or modules dir).
  
  When omitted or empty, [`Self::name`](#modulespec) is the source file in the modules
  directory. An explicit path is the only load-set key.
- `module_type`: Module type string (TOML key `type`; default `"js"`). Use `"python"`
  for Pyodide modules.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginManifest

```rust
pub struct PluginManifest {
    pub api_version: u32,
    pub id: String,
    pub name: Option<String>,
    pub version: Option<String>,
    pub logo: Option<String>,
    pub runtime: PluginRuntimeKind,
    pub command: Option<PathBuf>,
    pub args: Vec<String>,
    pub entrypoints: Vec<Entrypoint>,
    pub workerd: Option<WorkerdRuntimeManifest>,
    pub modules: Vec<ModuleSpec>,
    pub triggers: TriggersManifest,
    pub events: EventsManifest,
    pub databases: Vec<DatabaseBindingManifest>,
    pub vars: Option<BTreeMap<String, Value>>,
    pub settings: Vec<SettingSpec>,
    pub secrets: Option<NamedBinding>,
    pub kv_namespaces: Vec<KvNamespaceManifest>,
    pub queues: QueuesManifest,
    pub work_fs: Option<NamedBinding>,
    pub oauth: Option<NamedBinding>,
    pub capabilities: CapabilitiesManifest,
    pub cli: Option<CliSchema>,
    pub oidc: OidcManifest,
}
```

On-disk plugin descriptor (`plugin.toml`, `api_version = 3`).

Root table for install / discovery, shaped after `wrangler.jsonc`: the
plugin exports named [`Entrypoint`](../bookclerk_plugin_abi/generated.md#entrypoint)s, declares triggers for its default
entrypoint (`[triggers]`, `[[events.consumers]]`), and lists the host
bindings it expects on `env` (`[vars]`, `[secrets]`, `[[kv_namespaces]]`,
`[work_fs]`, `[oauth]`, `[[databases]]`, `[[events.producers]]`). Parse
with [`Self::parse`](#pluginmanifestparse) (deserialize + [`Self::validate`](#pluginmanifestvalidate)). Unknown keys are
rejected.

#### Validation highlights

- `api_version` must equal [`bookclerk_plugin_abi::PRODUCT_API_VERSION`](../bookclerk_plugin_abi/index.md#product_api_version)
- `id` must pass [`crate::validate_plugin_id`](id.md#validate_plugin_id)
- at least one entrypoint or trigger must be declared
- native requires `command`; workerd requires `[workerd]` with date + main
- `domains` forbidden on native (use `tcp` for mediated sockets); required for workerd + outbound
- optional `logo` must pass [`crate::validate_logo`](logo.md#validate_logo)

**Fields**

- `api_version`: ABI / schema version. Must equal [`bookclerk_plugin_abi::PRODUCT_API_VERSION`](../bookclerk_plugin_abi/index.md#product_api_version).
- `id`: Display / CLI alias (`[a-z0-9_]{2,32}` grammar). Not a privilege.
- `name`: Optional human-readable display name for Settings / Accounts UI.
- `version`: Package version. Required for enrollment and `bookclerk plugin check`
  and must be semver: the host pins the `[[settings]]` schema with it
  (see [`Self::package_version`](#pluginmanifestpackage_version)).
- `logo`: Settings / UI logo: `https://…` URL or relative image path under the
  plugin root. Validated by [`crate::validate_logo`](logo.md#validate_logo) when present.
- `runtime`: Guest runtime (`native` default, or `workerd`). Selects the backend
  behind `bookclerk-workerd`; it does not change the ABI.
- `command`: Native executable path relative to the install root (required when
  `runtime = "native"`).
- `args`: Extra argv passed after `command` for native guests.
- `entrypoints`: Named entrypoints this plugin exports (`["storefront", "cli"]`).
- `workerd`: Workerd isolate config (required when `runtime = "workerd"`).
- `modules`: Optional module list for workerd packages (`[[modules]]`).
- `triggers`: Default-entrypoint triggers other than events (`[triggers]`).
- `events`: Event consumers (triggers) and producers (publish grants).
- `databases`: Isolated plugin-owned database bindings (`[[databases]]`).
- `vars`: Operator config binding (`[vars]`). Declares that the guest receives
  `CONFIG`; an empty table is the usual form. Values come from
  `[[settings]]` defaults and the plugin instance document.
- `settings`: Operator settings schema (`[[settings]]`): the keys an instance
  document may set, their types, enum values, and defaults. Non-secret
  entries need `[vars]`; `type = "secret"` entries need `[secrets]`.
- `secrets`: Sealed secrets binding (`[secrets]`).
- `kv_namespaces`: Per-plugin key/value storage bindings (`[[kv_namespaces]]`).
  
  Durable `env.<BINDING>` (default `KV`) served from the library
  database, distinct from [`Self::databases`]: a Workers `KVNamespace`
  surface (`get` / `put` / `delete` / `list`, expiration, metadata) that
  follows the instance across hosts.
- `queues`: Workers Queues (`[queues]` with `[[queues.producers]]` /
  `[[queues.consumers]]`): send bindings and `queue(batch)` triggers on
  the host outbox. The domain-event bus (`[events]`) is the host's own
  `core.events` queue; `[queues]` names plugin-owned queues.
- `work_fs`: Host-mediated work filesystem binding (`[work_fs]`).
- `oauth`: OAuth callback tunnel binding (`[oauth]`).
- `capabilities`: Declared network capability (consent has no Wrangler analogue).
- `cli`: Optional CLI schema advertised to `bookclerk plugins <id>` (from
  `bookclerk-plugin-abi::CliSchema`). Requires the `Cli` entrypoint.
- `oidc`: Optional Bookclerk-as-IdP client templates (`[[oidc.clients]]`).

#### PluginManifest::declares_python

```rust
pub fn declares_python(&self) -> bool
```

True when the manifest itself declares a Python workerd guest.

Python means any of: `main_module` ending in `.py`, or a `[[modules]]`
row with `type = "python"` or a `.py` load-set key. An explicit `path`
is that key; `name` is the key only when `path` is omitted. Compatibility
flags are not evidence. A `.py` file that exists only on disk is not a
declaration: check and materialize reject it.

##### Returns

`true` when Python flags are required and Pyodide consent hosts apply
for outbound guests.

#### PluginManifest::unimplemented_surface

```rust
pub fn unimplemented_surface(&self) -> Option<String>
```

Spawn/load error when the manifest declares a surface the host cannot
run yet.

Every manifest surface is implemented today (`[[kv_namespaces]]` and
`[queues]` included); the hook stays so a future declared-but-unbuilt
table can refuse load with "not implemented yet" without touching the
spawn sites.

##### Returns

The first unimplemented surface, or `None` when load may continue.

#### PluginManifest::settings_schema

```rust
pub fn settings_schema(&self) -> SettingsSchema
```

Operator settings schema declared by `[[settings]]`.

##### Returns

The declared entries in order; empty when the plugin takes no
operator settings.

#### PluginManifest::config_options

```rust
pub fn config_options(&self) -> Vec<ConfigOption>
```

`describe().configOptions` derived from `[[settings]]`.

A guest whose `describe()` returns a non-empty list must return this
list; an empty list means "use the manifest".

#### PluginManifest::bindings

```rust
pub fn bindings(&self) -> BindingCapabilities
```

Resolves the declared host bindings into one flat view.

##### Returns

[`BindingCapabilities`](#bindingcapabilities) with one flag per named-binding table and the
`[[databases]]` binding names in declaration order.

#### PluginManifest::has_entrypoint

```rust
pub fn has_entrypoint(&self, entrypoint: Entrypoint) -> bool
```

True when the plugin exports `entrypoint`.

#### PluginManifest::consumes_events

```rust
pub fn consumes_events(&self) -> bool
```

True when the default entrypoint has an `event(batch)` trigger.

#### PluginManifest::runs_jobs

```rust
pub fn runs_jobs(&self) -> bool
```

True when the default entrypoint has a `job(controller)` trigger.

#### PluginManifest::produces_events

```rust
pub fn produces_events(&self) -> bool
```

True when the plugin declares any publishable event type.

#### PluginManifest::consumes_queues

```rust
pub fn consumes_queues(&self) -> bool
```

True when the default entrypoint has a `queue(batch)` trigger.

#### PluginManifest::produces_queues

```rust
pub fn produces_queues(&self) -> bool
```

True when the plugin declares any queue send binding.

#### PluginManifest::queue_consumer_names

```rust
pub fn queue_consumer_names(&self) -> Vec<String>
```

Queue names the default entrypoint consumes (`[[queues.consumers]]`).

#### PluginManifest::queue_producer_names

```rust
pub fn queue_producer_names(&self) -> Vec<String>
```

Queue names this plugin may send to (`[[queues.producers]]`),
deduplicated in declaration order.

#### PluginManifest::families

```rust
pub fn families(&self) -> Vec<PluginFamily>
```

Handler families this plugin belongs to, in
[`PluginFamily::ALL`](#pluginfamilyall) priority order.

A plugin joins the `Source` / `Output` / `Database` families through
the matching named entrypoint and the `Integration` family through
event consumers, queue consumers, job triggers, `RemoteLibrary`,
`Cli`, or `Oidc`.

##### Returns

Deduplicated families; never empty for a validated manifest.

#### PluginManifest::primary_family

```rust
pub fn primary_family(&self) -> PluginFamily
```

Primary handler family: the first of [`Self::families`](#pluginmanifestfamilies).

Picks the `config.toml` settings prefix (`database.*` > `sources.*` >
`output.*` > `integrations.*`) and the API grouping.

##### Returns

The highest-priority family; `Integration` for a manifest with no
entrypoint (only reachable before validation).

#### PluginManifest::producer_types

```rust
pub fn producer_types(&self) -> Vec<String>
```

Event types this plugin may publish (from `[[events.producers]]`).

#### PluginManifest::capabilities

```rust
pub fn capabilities(&self) -> PluginCapabilities
```

Typed capability declaration this manifest implies.

Guests return exactly this value from `describe()` so the host's
widening check (manifest vs describe vs grant) is a plain equality
comparison. Native Rust guests typically call it on the manifest
embedded with `include_str!("../plugin.toml")`.

##### Returns

Entrypoints, event consumers/producers, job types, database binding
names, and named binding names (`CONFIG`, `SECRETS`, `WORK_FS`,
`OAUTH`, KV, event, and queue bindings) in manifest order.

#### PluginManifest::binding_names

```rust
pub fn binding_names(&self) -> Vec<String>
```

Names of the non-database bindings this manifest exposes on `env`.

##### Returns

`CONFIG` (when `[vars]` is present), the resolved `[secrets]`,
`[work_fs]`, `[oauth]` names, every `[[kv_namespaces]]` binding,
every distinct `[[events.producers]]` binding, and every
`[[queues.producers]]` binding, in that order.

#### PluginManifest::consent_domains

```rust
pub fn consent_domains(&self) -> Vec<String>
```

Returns domains for workerd network consent UI.

IDNA-normalizes author domains and includes Pyodide CDN hosts when this
is a Python + outbound guest. On normalization failure, falls back to
filtering author domains individually (invalid entries dropped).

##### Returns

Deduplicated host patterns suitable for operator consent display.

#### PluginManifest::logo_kind

```rust
pub fn logo_kind(&self) -> Result<Option<LogoKind>>
```

Classifies the optional `logo` field after validation.

##### Returns

`Ok(None)` when `logo` is unset; `Ok(Some(kind))` when valid.

##### Errors

Returns [`Error`](error.md#error) from [`crate::validate_logo`](logo.md#validate_logo) when `logo` is set but
invalid.

#### PluginManifest::parse

```rust
pub fn parse(text: &str) -> Result<Self>
```

Parses and validates a `plugin.toml` document.

Deserializes with `toml::from_str` then runs [`Self::validate`](#pluginmanifestvalidate).

##### Arguments

* `text` - Full UTF-8 contents of `plugin.toml`.

##### Returns

A semantically validated [`PluginManifest`](#pluginmanifest).

##### Errors

Returns [`Error::TomlDe`](error.md#error) on malformed TOML, or [`Error::Message`](error.md#error) from
[`Self::validate`](#pluginmanifestvalidate).

##### Examples

```rust
use bookclerk_plugin_manifest::PluginManifest;

let m = PluginManifest::parse(r#"
api_version = 3
id = "echo"
runtime = "native"
command = "./echo"
entrypoints = ["cli"]

[capabilities.network]
mode = "deny"
"#).unwrap();
assert_eq!(m.id, "echo");
```

#### PluginManifest::package_version

```rust
pub fn package_version(&self) -> Result<String>
```

Runs semantic validation after deserialize.

The declared `version`, validated as a semantic version.

Enrollment pins the settings schema under this version and refuses a
plugin whose version is older than the pin, so the field is required
even though the parser accepts its absence.

##### Errors

Returns [`Error::Message`](error.md#error) when `version` is missing or not semver.

#### PluginManifest::validate

```rust
pub fn validate(&self) -> Result<()>
```

Checks id grammar, `api_version`, logo, runtime-specific required
fields, and network domain rules (native forbid / workerd outbound
require / IDNA).

##### Returns

`Ok(())` when all rules pass.

##### Errors

Returns [`Error::Message`](error.md#error) describing the first failed rule.

#### PluginManifest::spawn_command

```rust
pub fn spawn_command(&self) -> Option<&PathBuf>
```

Resolves the process path to spawn for this guest.

##### Returns

`Some(command)` for native guests; `None` for workerd (the host
resolves the `bookclerk-workerd` helper beside itself).

#### PluginManifest::jail_network_need

```rust
pub fn jail_network_need(&self) -> JailNetworkNeed
```

Maps manifest network + oauth binding to OS jail network policy.

Native-behind-workerd guests are nested under `NetPolicy::Deny`; they
must use the SDK socket capability, not ambient `AF_INET`. Direct-native
diagnostics still map `mode = "outbound"` to
[`JailNetworkNeed::Outbound`](#jailnetworkneed). Fetch hostname allowlists (`domains`)
remain workerd-only. With `[oauth]`, native guests also need loopback
listen for the host callback tunnel ([`JailNetworkNeed::Listen`](#jailnetworkneed)).
Workerd always needs loopback listen/connect to its Cloudflare child.

##### Returns

The jail network need the host should apply when spawning the guest.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OidcManifest

```rust
pub struct OidcManifest {
    pub clients: Vec<OidcClientToml>,
}
```

`[[oidc.clients]]` install-time OIDC AS templates.

**Fields**

- `clients`: Client templates materialized when this plugin is installed.

#### OidcManifest::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no clients are declared.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OidcClientToml

```rust
pub struct OidcClientToml {
    pub client_id: String,
    pub display_name: String,
    pub callback_path: String,
    pub public_client: bool,
    pub default_scopes: Vec<String>,
    pub issue_refresh_token: bool,
    pub origin_config_key: String,
}
```

One `[[oidc.clients]]` row (snake_case TOML).

**Fields**

- `client_id`: OAuth `client_id`.
- `display_name`: Operator-facing card title.
- `callback_path`: Path appended to the plugin origin.
- `public_client`: Public PKCE when true (default).
- `default_scopes`: Scopes for first materialization.
- `issue_refresh_token`: Whether new rows may issue refresh tokens (default true).
- `origin_config_key`: Dotted config key for the player origin.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### PluginFamily

```rust
pub enum PluginFamily {
    Source,
    Integration,
    Output,
    Database,
}
```

Handler family — how the daemon, CLI, and UI group plugins and which
`config.toml` prefix holds their settings.

Derived from the exported [`Entrypoint`](../bookclerk_plugin_abi/generated.md#entrypoint)s and declared triggers (see
[`PluginManifest::families`](#pluginmanifestfamilies)); never written to `plugin.toml`. Wire values
are lowercase (`source`, `integration`, `output`, `database`).

**Variants**

- `Source`: Storefront / library source (`Storefront` entrypoint).
- `Integration`: Event consumer, job runner, remote library, CLI, or OIDC bridge.
- `Output`: Destination / output backend (`Storage` entrypoint).
- `Database`: Library database backend (`DatabaseAdapter` entrypoint).

#### PluginFamily::ALL

```rust
pub const ALL: [Self; 4] = _
```

Every family in settings-prefix priority order.

#### PluginFamily::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the lowercase wire name used in API paths and config prefixes.

##### Returns

One of `"source"`, `"integration"`, `"output"`, or `"database"`.

#### PluginFamily::parse

```rust
pub fn parse(name: &str) -> Option<Self>
```

Parses a wire name produced by [`Self::as_str`](#pluginfamilyas_str).

##### Returns

`Some(family)` for a known name; `None` otherwise.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### PluginRuntimeKind

```rust
pub enum PluginRuntimeKind {
    Native,
    Workerd,
}
```

Guest runtime selection (`runtime` key in `plugin.toml`).

Defaults to [`PluginRuntimeKind::Native`](#pluginruntimekind) when omitted.

**Variants**

- `Native`: OS binary speaking the Workers RPC ABI, spawned through `bookclerk-jail`.
  
  Requires a non-empty `command`. Must not declare
  `capabilities.network.domains` (fetch allowlists are workerd-only).
  Raw TCP uses `capabilities.network.tcp` via the Bookclerk socket
  capability; ambient `AF_INET` is not a grant.
- `Workerd`: Author modules loaded by first-party `bookclerk-workerd` (one jail +
  one isolate per plugin).
  
  Requires a `[workerd]` table. Outbound mode requires non-empty
  `capabilities.network.domains`.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NetworkMode

```rust
pub enum NetworkMode {
    Deny,
    Outbound,
}
```

Network mode declared in `[capabilities.network]`.

Defaults to [`NetworkMode::Deny`](#networkmode). See `docs/plugins.md` for native vs
workerd semantics.

**Variants**

- `Deny`: No outbound network (default). Jail and isolate both refuse guest
  fetches / coarse outbound.
- `Outbound`: Coarse jail outbound (native) or workerd isolate egress.
  
  For **workerd**, pair with `domains` (isolate hostname allowlist;
  required and validated). For **native**, do **not** set `domains`;
  declare `tcp` for mediated `connect()`. Ambient internet sockets are
  not a grant (native-behind-workerd sibling jail is `NetPolicy::Deny`).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JailNetworkNeed

```rust
pub enum JailNetworkNeed {
    None,
    Outbound,
    Listen,
}
```

OS jail network capability derived from manifest network + oauth binding.

Produced by [`PluginManifest::jail_network_need`](#pluginmanifestjail_network_need). Distinct from isolate
hostname allowlists ([`crate::EgressPolicy`](egress.md#egresspolicy)), which apply only inside
workerd.

**Variants**

- `None`: No IP sockets (`mode = "deny"` on native).
- `Outbound`: Native `outbound` without OAuth — coarse jail outbound (no hostname filter).
- `Listen`: Native `outbound` + `bindings.oauth`, or any workerd guest (loopback
  listen/connect to the Cloudflare child / OAuth callback tunnel).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### entrypoint_family

```rust
pub fn entrypoint_family(entrypoint: Entrypoint) -> PluginFamily
```

Handler family an [`Entrypoint`](../bookclerk_plugin_abi/generated.md#entrypoint) belongs to.

#### Returns

The [`PluginFamily`](#pluginfamily) whose `config.toml` prefix and API group hold plugins
exporting this entrypoint.

### is_valid_database_binding_name

```rust
pub fn is_valid_database_binding_name(name: &str) -> bool
```

True when `name` is a valid Workers-style binding name (`[A-Z][A-Z0-9_]*`).

### is_host_event_type

```rust
pub fn is_host_event_type(event_type: &str) -> bool
```

True when `event_type` is in [`HOST_EVENT_TYPES`](#host_event_types).

### is_valid_queue_name

```rust
pub fn is_valid_queue_name(name: &str) -> bool
```

True when `name` is a valid queue name: `[a-z0-9][a-z0-9._-]*`, at most
[`MAX_QUEUE_NAME_LEN`](#max_queue_name_len) characters.

### is_reserved_queue_name

```rust
pub fn is_reserved_queue_name(name: &str) -> bool
```

True when `name` starts with a [`RESERVED_QUEUE_PREFIXES`](#reserved_queue_prefixes) entry.

## Constants

### ALL_ENTRYPOINTS

```rust
pub const ALL_ENTRYPOINTS: [Entrypoint; 6] = _;
```

Every entrypoint in declaration order (mirrors the Cap'n Proto enum).

### MAX_DATABASE_BINDINGS

```rust
pub const MAX_DATABASE_BINDINGS: usize = 8;
```

Maximum named database bindings one plugin may declare.

### MAX_DATABASE_BINDING_NAME_LEN

```rust
pub const MAX_DATABASE_BINDING_NAME_LEN: usize = 32;
```

Maximum length of one database binding name.

### DEFAULT_KV_MAX_VALUE_BYTES

```rust
pub const DEFAULT_KV_MAX_VALUE_BYTES: u32 = 65_536;
```

Default `max_value_bytes` for `[[kv_namespaces]]`: 64 KiB.

### MAX_KV_VALUE_BYTES

```rust
pub const MAX_KV_VALUE_BYTES: u32 = 1_048_576;
```

Largest `max_value_bytes` a namespace may declare: 1 MiB (the ABI cap).

### DEFAULT_KV_MAX_KEYS

```rust
pub const DEFAULT_KV_MAX_KEYS: u32 = 10_000;
```

Default `max_keys` for `[[kv_namespaces]]`.

### MAX_KV_KEYS

```rust
pub const MAX_KV_KEYS: u32 = 1_000_000;
```

Largest `max_keys` a namespace may declare.

### DEFAULT_SECRETS_BINDING

```rust
pub const DEFAULT_SECRETS_BINDING: &str = "SECRETS";
```

Default `binding` for `[secrets]`.

### DEFAULT_WORK_FS_BINDING

```rust
pub const DEFAULT_WORK_FS_BINDING: &str = "WORK_FS";
```

Default `binding` for `[work_fs]`.

### DEFAULT_OAUTH_BINDING

```rust
pub const DEFAULT_OAUTH_BINDING: &str = "OAUTH";
```

Default `binding` for `[oauth]`.

### DEFAULT_KV_BINDING

```rust
pub const DEFAULT_KV_BINDING: &str = "KV";
```

Default `binding` for `[[kv_namespaces]]`.

### DEFAULT_EVENTS_BINDING

```rust
pub const DEFAULT_EVENTS_BINDING: &str = "EVENTS";
```

Default `binding` for `[[events.producers]]`.

### CONFIG_BINDING

```rust
pub const CONFIG_BINDING: &str = "CONFIG";
```

Binding name the host uses for operator config (`[vars]`).

### HOST_EVENT_TYPES

```rust
pub const HOST_EVENT_TYPES: [&str; 1] = _;
```

Domain event types only the host emits.

`book_acquired` is written by the host's acquire pipeline in the same
transaction as the library row once a source plugin's acquire handler
returns; the source never publishes it. Plugins consume these types but
cannot declare them under `[[events.producers]]`.

### MAX_QUEUE_NAME_LEN

```rust
pub const MAX_QUEUE_NAME_LEN: usize = 63;
```

Maximum length of a queue name.

### RESERVED_QUEUE_PREFIXES

```rust
pub const RESERVED_QUEUE_PREFIXES: [&str; 2] = _;
```

Queue-name prefixes only the host may use (`core.events`, `node.<id>`).

### DEFAULT_QUEUE_MAX_BATCH_SIZE

```rust
pub const DEFAULT_QUEUE_MAX_BATCH_SIZE: u32 = 10;
```

Default `max_batch_size` for `[[queues.consumers]]`.

### MAX_QUEUE_BATCH_SIZE

```rust
pub const MAX_QUEUE_BATCH_SIZE: u32 = 100;
```

Largest `max_batch_size` a consumer may declare.

### DEFAULT_QUEUE_MAX_BATCH_TIMEOUT_SECS

```rust
pub const DEFAULT_QUEUE_MAX_BATCH_TIMEOUT_SECS: u32 = 5;
```

Default `max_batch_timeout` (seconds) for `[[queues.consumers]]`.

### MAX_QUEUE_BATCH_TIMEOUT_SECS

```rust
pub const MAX_QUEUE_BATCH_TIMEOUT_SECS: u32 = 60;
```

Largest `max_batch_timeout` (seconds) a consumer may declare.

### DEFAULT_QUEUE_MAX_RETRIES

```rust
pub const DEFAULT_QUEUE_MAX_RETRIES: u32 = 3;
```

Default `max_retries` for `[[queues.consumers]]`.

### MAX_QUEUE_RETRIES

```rust
pub const MAX_QUEUE_RETRIES: u32 = 100;
```

Largest `max_retries` a consumer may declare.

### MAX_QUEUE_RETRY_DELAY_SECS

```rust
pub const MAX_QUEUE_RETRY_DELAY_SECS: u32 = 43_200;
```

Largest `retry_delay` (seconds) a consumer may declare: 12 hours.
