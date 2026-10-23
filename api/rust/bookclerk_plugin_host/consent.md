**bookclerk_plugin_host > consent**

# Module: consent

## Contents

**Structs**

- [`GrantedEventConsumer`](#grantedeventconsumer) - One approved event-consumer subscription (structural authority).
- [`PendingStructural`](#pendingstructural) - Newly requested structural capabilities awaiting operator consent.
- [`PluginGrant`](#plugingrant) - One approved grant snapshot for a provenance-qualified plugin.
- [`PluginGrantStore`](#plugingrantstore) - On-disk grant store.

**Functions**

- [`active_processes_for`](#active_processes_for) - Absolute Spec `active_processes` from runtime overhead + extra budget.
- [`canonical_event_filter`](#canonical_event_filter) - Minified JSON for an event filter; `null` / missing → `None`.
- [`consent_request`](#consent_request) - Build the consent request surface from a manifest and its PluginKey.
- [`consent_request_alias`](#consent_request_alias) - Build the consent request surface from a manifest (alias only; tests).
- [`consent_summary`](#consent_summary) - Human-readable consent lines for CLI/UI.
- [`consumers_cover`](#consumers_cover) - True when every requested consumer is covered by some approved consumer.
- [`cores_to_percent`](#cores_to_percent) - Convert cores to Spec/config percent-of-one-core, rounded to 0.01 core.
- [`database_binding_name`](#database_binding_name) - The binding name when a grant entry is a named database binding.
- [`effective_cpu_cores`](#effective_cpu_cores) - Clamp operator/manifest cores into `0.01..=`[`host_cpu_cores_max`](consent.md#bookclerk_plugin_hostconsenthost_cpu_cores_max).
- [`effective_cpu_rate_percent`](#effective_cpu_rate_percent) - Resolved jail Spec CPU rate as percent of one logical CPU.
- [`effective_disk_budget_bytes`](#effective_disk_budget_bytes) - Resolved disk budget in bytes for `data/` and `tmp/` checks.
- [`effective_disk_mib`](#effective_disk_mib) - Resolved disk budget in MiB (default + host clamp).
- [`effective_extra_processes`](#effective_extra_processes) - Clamp an operator/manifest extra-process budget (`0..=`[`PLUGIN_JAIL_EXTRA_PROCESSES_MAX`](consent.md#bookclerk_plugin_hostconsentplugin_jail_extra_processes_max)).
- [`effective_grant`](#effective_grant) - Spawn/delivery grant: stored approval is authoritative, host-normalized.
- [`effective_memory_mib`](#effective_memory_mib) - Resolved jail Spec memory ceiling in MiB.
- [`format_cpu_cores`](#format_cpu_cores) - Format cores for operator-facing text (`0.80`).
- [`grant_covers`](#grant_covers) - True when a stored grant is usable for enable/spawn of this PluginKey.
- [`grant_has_binding`](#grant_has_binding) - True when `grant.bindings` contains `name` (case-insensitive).
- [`grant_revision`](#grant_revision) - Canonical SHA-256 of a grant's security-relevant fields (not presentation).
- [`grant_within_ceiling`](#grant_within_ceiling) - True when an existing grant stays within the manifest request surface.
- [`granted_consumers_from_manifest`](#granted_consumers_from_manifest) - Consumers declared by `plugin.toml` as structural grant entries.
- [`granted_database_bindings`](#granted_database_bindings) - Consented database binding names on a grant, in manifest-set order.
- [`granted_jobs_from_manifest`](#granted_jobs_from_manifest) - Job trigger types declared by `[triggers].jobs`.
- [`granted_queue_consumers`](#granted_queue_consumers) - Consented queue consumer names on a grant.
- [`host_cpu_cores_max`](#host_cpu_cores_max) - Host max jail CPU in cores (`logical_cpus` as a 2 d.p. float).
- [`host_cpu_rate_max`](#host_cpu_rate_max) - Host CPU rate ceiling in one-core percent units (`logical_cpus × 100`).
- [`host_overlay_config_digest`](#host_overlay_config_digest) - SHA-256 of host config that participates in implied network overlays.
- [`inject_workerd_grant_env`](#inject_workerd_grant_env) - Injects effective-grant overrides for `bookclerk-workerd` into a guest command.
- [`is_platform_envelope`](#is_platform_envelope) - True when `grant` stays inside the platform installer envelope (deny
- [`jail_process_overhead`](#jail_process_overhead) - Fixed payload occupancy for the launcher / primary guest tree.
- [`network_compatible`](#network_compatible) - Returns true when a stored network mode can satisfy a requested network mode.
- [`node_logical_cpus`](#node_logical_cpus) - Logical CPUs visible to this process (at least 1).
- [`overlay_host_implied_network`](#overlay_host_implied_network) - Overlay TCP + CIDR implied by host configuration (not persisted).
- [`pending_structural`](#pending_structural) - Structural capabilities the current manifest requests that the stored grant
- [`percent_to_cores`](#percent_to_cores) - Convert Spec/config percent-of-one-core (100 = 1.00 core) to cores (2 d.p.).
- [`queue_consumer_grant`](#queue_consumer_grant) - Grant token for consuming `queue`.
- [`queue_producer_grant`](#queue_producer_grant) - Grant token for sending through the `[[queues.producers]]` binding `binding`.
- [`reconcile_host_overlay_authority`](#reconcile_host_overlay_authority) - Recomputes effective overlay authority and fences sessions that no longer match.
- [`require_binding`](#require_binding) - Fail closed when a delivery site needs a binding the covering grant lacks.
- [`require_grant`](#require_grant) - Require an operator grant before enable **or** every external spawn.
- [`spawn_config_for_grant`](#spawn_config_for_grant) - Spawn `config` payload: non-empty settings only when the grant includes `config`.
- [`spawn_grant`](#spawn_grant) - Resolve the covering grant for spawn: verified platform auto-grant, else [`require_grant`](consent.md#bookclerk_plugin_hostconsentrequire_grant).
- [`validate_approved_grant`](#validate_approved_grant) - Validates and normalizes an operator-supplied grant against host hard caps.
- [`validate_described_capabilities`](#validate_described_capabilities) - Reject `describe()` capability claims that exceed the manifest and the
- [`validate_described_config_options`](#validate_described_config_options) - Validates `describe().configOptions` against the manifest `[[settings]]`.
- [`windows_session_active_processes`](#windows_session_active_processes) - Windows outer session Job active-process cap: baseline 5 plus `extra`.

**Constants**

- [`GRANTS_FILE`](#grants_file) - Filename under `$BOOKCLERK_FILES_DIR` for persisted grants.
- [`GRANT_SCHEMA_VERSION`](#grant_schema_version) - Current persisted [`PluginGrant`](consent.md#bookclerk_plugin_hostconsentplugingrant) document version.
- [`KNOWN_HOST_BINDINGS`](#known_host_bindings) - Host binding names operators may grant (widen or narrow).
- [`PLUGIN_JAIL_ACTIVE_PROCESSES_MAX`](#plugin_jail_active_processes_max) - Absolute OS jail PID ceiling (`overhead + extra`, inclusive).
- [`PLUGIN_JAIL_CPU_CORES_DEFAULT`](#plugin_jail_cpu_cores_default) - Default jail CPU as cores (`PLUGIN_JAIL_CPU_RATE_DEFAULT` / 100).
- [`PLUGIN_JAIL_CPU_RATE_DEFAULT`](#plugin_jail_cpu_rate_default) - Default jail Spec CPU rate percent for confined guests (one-core units).
- [`PLUGIN_JAIL_CPU_RATE_MAX`](#plugin_jail_cpu_rate_max) - Absolute ceiling used only when host CPU detection is unavailable in docs.
- [`PLUGIN_JAIL_EXTRA_PROCESSES_DEFAULT`](#plugin_jail_extra_processes_default) - Default **extra** processes/threads beyond launcher overhead.
- [`PLUGIN_JAIL_EXTRA_PROCESSES_MAX`](#plugin_jail_extra_processes_max) - Host hard cap for per-plugin extra process budget.
- [`PLUGIN_JAIL_MEMORY_MIB_DEFAULT`](#plugin_jail_memory_mib_default) - Default jail Spec memory ceiling (MiB) for confined guests.
- [`PLUGIN_JAIL_MEMORY_MIB_MAX`](#plugin_jail_memory_mib_max) - Host hard cap for per-plugin jail memory overrides (MiB).
- [`PLUGIN_STATE_BUDGET_MIB_DEFAULT`](#plugin_state_budget_mib_default) - Default per-plugin `data/` and `tmp/` disk budget (MiB each).
- [`PLUGIN_STATE_BUDGET_MIB_MAX`](#plugin_state_budget_mib_max) - Host hard cap for per-plugin disk budget overrides (MiB).
- [`QUEUE_CONSUMER_BINDING_PREFIX`](#queue_consumer_binding_prefix) - Grant `bindings` entry prefix for one `[[queues.consumers]]` row
- [`QUEUE_PRODUCER_BINDING_PREFIX`](#queue_producer_binding_prefix) - Grant `bindings` entry prefix for one `[[queues.producers]]` binding
- [`WINDOWS_SESSION_PROCESS_BASELINE`](#windows_session_process_baseline) - Windows outer session Job processes before the granted extra budget.
- [`WORKERD_GRANT_CPU_MS_ENV`](#workerd_grant_cpu_ms_env) - Grant CPU budget (ms) for workerd logging / limit narrowing.
- [`WORKERD_GRANT_DOMAINS_ENV`](#workerd_grant_domains_env) - Comma-separated grant domain allowlist for workerd egress.
- [`WORKERD_GRANT_NETWORK_MODE_ENV`](#workerd_grant_network_mode_env) - Env keys consumed by `bookclerk-workerd` (`grant.rs`) at isolate start.
- [`WORKERD_GRANT_POLICY_ENV`](#workerd_grant_policy_env) - Canonical JSON [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#bookclerk_plugin_manifestegressegresspolicy) (preferred over piecemeal domain env vars).
- [`WORKERD_GRANT_SUBREQUESTS_ENV`](#workerd_grant_subrequests_env) - Grant subrequest budget injected into workerd `EGRESS_POLICY`.

---

## bookclerk_plugin_host::consent::GRANTS_FILE

*Constant*: `&str`

Filename under `$BOOKCLERK_FILES_DIR` for persisted grants.



## bookclerk_plugin_host::consent::GRANT_SCHEMA_VERSION

*Constant*: `u32`

Current persisted [`PluginGrant`](consent.md#bookclerk_plugin_hostconsentplugingrant) document version.

This constant versions the on-disk grant representation. Missing / `0` JSON
is a legacy document. Empty structural sets are fail-closed at **every**
schema version: they approve nothing and do not acquire capabilities merely
because a newer manifest requests them.



## bookclerk_plugin_host::consent::GrantedEventConsumer

*Struct*

One approved event-consumer subscription (structural authority).

Identity includes the event type plus the security-relevant subscription
properties that change what the plugin receives: schema versions, suspend
support, and the host-owned payload filter. Operational knobs
(`resource_class`, `max_retries`) are **not** part of authority.

**Fields:**
- `event_type: String` - Versioned event type (`book_acquired`, …).
- `schema_versions: Vec<u32>` - Schema versions this grant may deliver (sorted, unique).
- `supports_suspend: bool` - Whether `EventResult::Suspended` is authorized for this type.
- `filter: Option<String>` - Canonical JSON of the host-owned payload filter, when present.

**Methods:**

- `fn from_manifest(consumer: &EventConsumer) -> Self` - Structural consumer snapshot from a `[[events.consumers]]` row.
- `fn covers(self: &Self, requested: &Self) -> bool` - True when `requested` is the same type with equal-or-narrower schemas,
- `fn intersect(self: &Self, requested: &Self) -> Option<Self>` - Intersection used when a stored grant already covers the current manifest.

**Traits:** Eq

**Trait Implementations:**

- **Ord**
  - `fn cmp(self: &Self, other: &GrantedEventConsumer) -> $crate::cmp::Ordering`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> GrantedEventConsumer`
- **PartialEq**
  - `fn eq(self: &Self, other: &GrantedEventConsumer) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &GrantedEventConsumer) -> $crate::option::Option<$crate::cmp::Ordering>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_host::consent::KNOWN_HOST_BINDINGS

*Constant*: `&[&str]`

Host binding names operators may grant (widen or narrow).



## bookclerk_plugin_host::consent::PLUGIN_JAIL_ACTIVE_PROCESSES_MAX

*Constant*: `u32`

Absolute OS jail PID ceiling (`overhead + extra`, inclusive).



## bookclerk_plugin_host::consent::PLUGIN_JAIL_CPU_CORES_DEFAULT

*Constant*: `f64`

Default jail CPU as cores (`PLUGIN_JAIL_CPU_RATE_DEFAULT` / 100).



## bookclerk_plugin_host::consent::PLUGIN_JAIL_CPU_RATE_DEFAULT

*Constant*: `u32`

Default jail Spec CPU rate percent for confined guests (one-core units).



## bookclerk_plugin_host::consent::PLUGIN_JAIL_CPU_RATE_MAX

*Constant*: `u32`

Absolute ceiling used only when host CPU detection is unavailable in docs.

Prefer [`host_cpu_rate_max`](consent.md#bookclerk_plugin_hostconsenthost_cpu_rate_max) at runtime (`logical_cpus × 100`).



## bookclerk_plugin_host::consent::PLUGIN_JAIL_EXTRA_PROCESSES_DEFAULT

*Constant*: `u32`

Default **extra** processes/threads beyond launcher overhead.



## bookclerk_plugin_host::consent::PLUGIN_JAIL_EXTRA_PROCESSES_MAX

*Constant*: `u32`

Host hard cap for per-plugin extra process budget.



## bookclerk_plugin_host::consent::PLUGIN_JAIL_MEMORY_MIB_DEFAULT

*Constant*: `u32`

Default jail Spec memory ceiling (MiB) for confined guests.



## bookclerk_plugin_host::consent::PLUGIN_JAIL_MEMORY_MIB_MAX

*Constant*: `u32`

Host hard cap for per-plugin jail memory overrides (MiB).



## bookclerk_plugin_host::consent::PLUGIN_STATE_BUDGET_MIB_DEFAULT

*Constant*: `u32`

Default per-plugin `data/` and `tmp/` disk budget (MiB each).



## bookclerk_plugin_host::consent::PLUGIN_STATE_BUDGET_MIB_MAX

*Constant*: `u32`

Host hard cap for per-plugin disk budget overrides (MiB).



## bookclerk_plugin_host::consent::PendingStructural

*Struct*

Newly requested structural capabilities awaiting operator consent.

**Fields:**
- `entrypoints: std::collections::BTreeSet<String>` - Entrypoints declared by the current manifest but not granted.
- `producers: std::collections::BTreeSet<String>` - Event producers declared by the current manifest but not granted.
- `consumers: std::collections::BTreeSet<GrantedEventConsumer>` - Event consumers declared by the current manifest but not granted.
- `jobs: std::collections::BTreeSet<String>` - Job triggers declared by the current manifest but not granted.
- `bindings: std::collections::BTreeSet<String>` - Host / database bindings declared by the current manifest but not granted.
- `compatibility_flags: std::collections::BTreeSet<String>` - Compatibility flags declared by the current manifest but not granted.

**Methods:**

- `fn is_empty(self: &Self) -> bool` - True when the current manifest introduces no unapproved structural caps.
- `fn summary_lines(self: &Self) -> Vec<String>` - CLI/UI lines for capabilities waiting on consent.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> PendingStructural`
- **PartialEq**
  - `fn eq(self: &Self, other: &PendingStructural) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PendingStructural`



## bookclerk_plugin_host::consent::PluginGrant

*Struct*

One approved grant snapshot for a provenance-qualified plugin.

**Fields:**
- `schema_version: u32` - Persisted grant document version.
- `plugin_key: String` - Canonical [`bookclerk_plugin_catalog::PluginKey`](../bookclerk_plugin_catalog/identity.md#bookclerk_plugin_catalogidentitypluginkey).
- `plugin_id: String` - Display / CLI alias from `plugin.toml` (not security authoritative).
- `entrypoints: std::collections::BTreeSet<String>` - Approved named entrypoints (wire names: `storefront`, `storage`,
- `producers: std::collections::BTreeSet<String>` - Approved event types the plugin may publish through its events binding.
- `consumers: std::collections::BTreeSet<GrantedEventConsumer>` - Approved event-consumer subscriptions (type + schema + suspend + filter).
- `jobs: std::collections::BTreeSet<String>` - Approved `[triggers].jobs` types the plugin may handle.
- `network_mode: String` - Approved network mode: `deny` or `outbound`.
- `domains: std::collections::BTreeSet<String>` - Approved initial outbound domain patterns (**workerd** allowlist).
- `manifest_domains: std::collections::BTreeSet<String>` - Network destinations requested by the installed manifest (not operator-added).
- `operator_added_domains: std::collections::BTreeSet<String>` - Operator-added destinations beyond the current manifest request.
- `operator_denied_domains: std::collections::BTreeSet<String>` - Operator-denied destinations (even if the manifest still lists them).
- `tcp: std::collections::BTreeSet<bookclerk_plugin_manifest::TcpGrant>` - Effective raw TCP grants (manifest ∪ operator additions − denials).
- `manifest_tcp: std::collections::BTreeSet<bookclerk_plugin_manifest::TcpGrant>` - TCP grants requested by the installed manifest.
- `operator_added_tcp: std::collections::BTreeSet<bookclerk_plugin_manifest::TcpGrant>` - Operator-added TCP grants; survive upgrades of the same PluginKey.
- `operator_denied_tcp: std::collections::BTreeSet<bookclerk_plugin_manifest::TcpGrant>` - Operator-denied TCP grants (even if the current manifest still lists them).
- `address_cidrs: std::collections::BTreeSet<String>` - Effective CIDR grants beyond the public Internet.
- `manifest_cidrs: std::collections::BTreeSet<String>` - CIDRs requested by the installed manifest.
- `operator_added_cidrs: std::collections::BTreeSet<String>` - Operator-added CIDRs; survive upgrades of the same PluginKey.
- `operator_denied_cidrs: std::collections::BTreeSet<String>` - Operator-denied CIDRs (even if the current manifest still lists them).
- `allow_undeclared_public_redirects: bool` - Effective undeclared-public-redirect permission.
- `operator_allow_undeclared_public_redirects: Option<bool>` - Explicit operator override for undeclared public redirects (`None` = follow manifest).
- `bindings: std::collections::BTreeSet<String>` - Approved host binding names (`config`, `secrets`, `oauth`, …).
- `compatibility_flags: std::collections::BTreeSet<String>` - Approved workerd compatibility flags from the consent snapshot.
- `cpu_ms: Option<u32>` - Optional workerd CPU budget override in milliseconds.
- `subrequests: Option<u32>` - Optional workerd outbound fetch / subrequest budget override.
- `disk_mib: Option<u32>` - Optional per-plugin disk budget for `data/` and `tmp/` (MiB each).
- `memory_mib: Option<u32>` - Optional jail Spec memory ceiling (MiB) for **native and workerd**.
- `cpu_rate_percent: Option<u32>` - Optional jail Spec CPU rate as percent of one logical CPU for **native**
- `extra_processes: Option<u32>` - Optional **extra** process/thread budget beyond launcher overhead (**native**).
- `approved_at: String` - RFC 3339 time when the operator approved this grant.

**Methods:**

- `fn allows_event_consumer(self: &Self, event_type: &str, schema_version: u32) -> bool` - True when this grant authorizes delivering `event_type` at `schema_version`.
- `fn allows_queue_consumer(self: &Self, queue: &str) -> bool` - True when this grant authorizes delivering messages from `queue`.
- `fn allows_queue_producer(self: &Self, binding: &str) -> bool` - True when this grant authorizes sending through the queue producer
- `fn allows_job(self: &Self, job_type: &str) -> bool` - True when this grant authorizes handling `job_type`.
- `fn empty() -> Self` - Empty grant used as a struct-update base for tests and new fields.
- `fn egress_policy(self: &Self) -> EgressPolicy` - Canonical typed network policy for workerd and the native socket proxy.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginGrant`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginGrant) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_host::consent::PluginGrantStore

*Struct*

On-disk grant store.

**Fields:**
- `grants: Vec<PluginGrant>` - Persisted per-plugin grant snapshots.

**Methods:**

- `fn path(files_dir: &Path) -> PathBuf` - Absolute path to `plugin-grants.json` under `files_dir`.
- `fn load(files_dir: &Path) -> Result<Self>` - Loads grants from disk, or an empty store when the file is missing.
- `fn save(self: &Self, files_dir: &Path) -> Result<()>` - Writes this grant store to `plugin-grants.json` under `files_dir`.
- `fn get_by_plugin_key(self: &Self, plugin_key: &str) -> Option<&PluginGrant>` - Returns the grant whose canonical PluginKey matches `plugin_key`.
- `fn get(self: &Self, plugin_key: &str) -> Option<&PluginGrant>` - Returns the grant for `plugin_key` (canonical), or an unambiguous alias.
- `fn upsert(self: & mut Self, grant: PluginGrant)` - Inserts or replaces the grant for `grant.plugin_key`.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginGrantStore) -> bool`
- **Default**
  - `fn default() -> PluginGrantStore`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginGrantStore`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::consent::QUEUE_CONSUMER_BINDING_PREFIX

*Constant*: `&str`

Grant `bindings` entry prefix for one `[[queues.consumers]]` row
(`queue_consumer:<queue>`): the host may deliver that queue to the plugin.



## bookclerk_plugin_host::consent::QUEUE_PRODUCER_BINDING_PREFIX

*Constant*: `&str`

Grant `bindings` entry prefix for one `[[queues.producers]]` binding
(`queue:<BINDING>`): the plugin may send through that `env` binding.



## bookclerk_plugin_host::consent::WINDOWS_SESSION_PROCESS_BASELINE

*Constant*: `u32`

Windows outer session Job processes before the granted extra budget.

Both `bookclerk-jail` supervisors, `bookclerk-workerd`, the pinned
`workerd`, and the native guest. Payload overhead stays 3
(`GuestRuntimeKind::process_overhead`); this baseline is only the outer
Windows Job. Linux `pids.max` counts threads and must not use this number.



## bookclerk_plugin_host::consent::WORKERD_GRANT_CPU_MS_ENV

*Constant*: `&str`

Grant CPU budget (ms) for workerd logging / limit narrowing.



## bookclerk_plugin_host::consent::WORKERD_GRANT_DOMAINS_ENV

*Constant*: `&str`

Comma-separated grant domain allowlist for workerd egress.



## bookclerk_plugin_host::consent::WORKERD_GRANT_NETWORK_MODE_ENV

*Constant*: `&str`

Env keys consumed by `bookclerk-workerd` (`grant.rs`) at isolate start.



## bookclerk_plugin_host::consent::WORKERD_GRANT_POLICY_ENV

*Constant*: `&str`

Canonical JSON [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#bookclerk_plugin_manifestegressegresspolicy) (preferred over piecemeal domain env vars).



## bookclerk_plugin_host::consent::WORKERD_GRANT_SUBREQUESTS_ENV

*Constant*: `&str`

Grant subrequest budget injected into workerd `EGRESS_POLICY`.



## bookclerk_plugin_host::consent::active_processes_for

*Function*

Absolute Spec `active_processes` from runtime overhead + extra budget.

```rust
fn active_processes_for(runtime: crate::spawn_plan::GuestRuntimeKind, extra: u32) -> u32
```



## bookclerk_plugin_host::consent::canonical_event_filter

*Function*

Minified JSON for an event filter; `null` / missing → `None`.

```rust
fn canonical_event_filter(filter: &Option<serde_json::Value>) -> Option<String>
```



## bookclerk_plugin_host::consent::consent_request

*Function*

Build the consent request surface from a manifest and its PluginKey.

```rust
fn consent_request(manifest: &crate::manifest::PluginManifest, plugin_key: &bookclerk_plugin_catalog::PluginKey) -> PluginGrant
```



## bookclerk_plugin_host::consent::consent_request_alias

*Function*

Build the consent request surface from a manifest (alias only; tests).

```rust
fn consent_request_alias(manifest: &crate::manifest::PluginManifest) -> PluginGrant
```



## bookclerk_plugin_host::consent::consent_summary

*Function*

Human-readable consent lines for CLI/UI.

```rust
fn consent_summary(grant: &PluginGrant) -> Vec<String>
```



## bookclerk_plugin_host::consent::consumers_cover

*Function*

True when every requested consumer is covered by some approved consumer.

```rust
fn consumers_cover(existing: &std::collections::BTreeSet<GrantedEventConsumer>, requested: &std::collections::BTreeSet<GrantedEventConsumer>) -> bool
```



## bookclerk_plugin_host::consent::cores_to_percent

*Function*

Convert cores to Spec/config percent-of-one-core, rounded to 0.01 core.

```rust
fn cores_to_percent(cores: f64) -> u32
```



## bookclerk_plugin_host::consent::database_binding_name

*Function*

The binding name when a grant entry is a named database binding.

```rust
fn database_binding_name(binding: &str) -> Option<&str>
```



## bookclerk_plugin_host::consent::effective_cpu_cores

*Function*

Clamp operator/manifest cores into `0.01..=`[`host_cpu_cores_max`](consent.md#bookclerk_plugin_hostconsenthost_cpu_cores_max).

```rust
fn effective_cpu_cores(value: Option<f64>) -> f64
```



## bookclerk_plugin_host::consent::effective_cpu_rate_percent

*Function*

Resolved jail Spec CPU rate as percent of one logical CPU.

Clamped to `1..=`[`host_cpu_rate_max`](consent.md#bookclerk_plugin_hostconsenthost_cpu_rate_max) (100 × logical CPUs).

```rust
fn effective_cpu_rate_percent(value: Option<u32>) -> u32
```



## bookclerk_plugin_host::consent::effective_disk_budget_bytes

*Function*

Resolved disk budget in bytes for `data/` and `tmp/` checks.

```rust
fn effective_disk_budget_bytes(grant: Option<&PluginGrant>) -> u64
```



## bookclerk_plugin_host::consent::effective_disk_mib

*Function*

Resolved disk budget in MiB (default + host clamp).

```rust
fn effective_disk_mib(value: Option<u32>) -> u32
```



## bookclerk_plugin_host::consent::effective_extra_processes

*Function*

Clamp an operator/manifest extra-process budget (`0..=`[`PLUGIN_JAIL_EXTRA_PROCESSES_MAX`](consent.md#bookclerk_plugin_hostconsentplugin_jail_extra_processes_max)).

```rust
fn effective_extra_processes(value: Option<u32>) -> u32
```



## bookclerk_plugin_host::consent::effective_grant

*Function*

Spawn/delivery grant: stored approval is authoritative, host-normalized.

Structural capabilities stay at the stored operator snapshot (narrowed).
Network destinations re-evaluate the current manifest plus persisted
operator additions/denials.

```rust
fn effective_grant(existing: &PluginGrant, requested: &PluginGrant) -> PluginGrant
```



## bookclerk_plugin_host::consent::effective_memory_mib

*Function*

Resolved jail Spec memory ceiling in MiB.

```rust
fn effective_memory_mib(value: Option<u32>) -> u32
```



## bookclerk_plugin_host::consent::format_cpu_cores

*Function*

Format cores for operator-facing text (`0.80`).

```rust
fn format_cpu_cores(cores: f64) -> String
```



## bookclerk_plugin_host::consent::grant_covers

*Function*

True when a stored grant is usable for enable/spawn of this PluginKey.

Identity match is enough. Extra capabilities on a newer manifest stay
pending until the operator approves them; [`effective_grant`](consent.md#bookclerk_plugin_hostconsenteffective_grant) intersects.

```rust
fn grant_covers(existing: &PluginGrant, requested: &PluginGrant) -> bool
```



## bookclerk_plugin_host::consent::grant_has_binding

*Function*

True when `grant.bindings` contains `name` (case-insensitive).

```rust
fn grant_has_binding(grant: &PluginGrant, name: &str) -> bool
```



## bookclerk_plugin_host::consent::grant_revision

*Function*

Canonical SHA-256 of a grant's security-relevant fields (not presentation).

```rust
fn grant_revision(grant: &PluginGrant) -> String
```



## bookclerk_plugin_host::consent::grant_within_ceiling

*Function*

True when an existing grant stays within the manifest request surface.

Used for **platform auto-grants** (installer envelope). Operator approvals
are not limited to this subset — see [`validate_approved_grant`](consent.md#bookclerk_plugin_hostconsentvalidate_approved_grant).

```rust
fn grant_within_ceiling(existing: &PluginGrant, requested: &PluginGrant) -> bool
```



## bookclerk_plugin_host::consent::granted_consumers_from_manifest

*Function*

Consumers declared by `plugin.toml` as structural grant entries.

```rust
fn granted_consumers_from_manifest(manifest: &crate::manifest::PluginManifest) -> std::collections::BTreeSet<GrantedEventConsumer>
```



## bookclerk_plugin_host::consent::granted_database_bindings

*Function*

Consented database binding names on a grant, in manifest-set order.

```rust
fn granted_database_bindings(grant: &PluginGrant) -> Vec<String>
```



## bookclerk_plugin_host::consent::granted_jobs_from_manifest

*Function*

Job trigger types declared by `[triggers].jobs`.

```rust
fn granted_jobs_from_manifest(manifest: &crate::manifest::PluginManifest) -> std::collections::BTreeSet<String>
```



## bookclerk_plugin_host::consent::granted_queue_consumers

*Function*

Consented queue consumer names on a grant.

```rust
fn granted_queue_consumers(grant: &PluginGrant) -> Vec<String>
```



## bookclerk_plugin_host::consent::host_cpu_cores_max

*Function*

Host max jail CPU in cores (`logical_cpus` as a 2 d.p. float).

```rust
fn host_cpu_cores_max() -> f64
```



## bookclerk_plugin_host::consent::host_cpu_rate_max

*Function*

Host CPU rate ceiling in one-core percent units (`logical_cpus × 100`).

```rust
fn host_cpu_rate_max() -> u32
```



## bookclerk_plugin_host::consent::host_overlay_config_digest

*Function*

SHA-256 of host config that participates in implied network overlays.

Changing a Postgres URL or D1 API origin must mint a new executor
`configuration_revision` and fence live sessions. Plugin endpoints (S3,
Audiobookshelf) come from instance documents, not `config.toml`.

```rust
fn host_overlay_config_digest(config: &bookclerk_config::Config) -> String
```



## bookclerk_plugin_host::consent::inject_workerd_grant_env

*Function*

Injects effective-grant overrides for `bookclerk-workerd` into a guest command.

Must be called after `env_clear` (explicit `BOOKCLERK_*` is blocked from
inheritance). Domains are always set (possibly empty) so a subset approval
cannot fall back to the full manifest allowlist.

# Arguments

* `cmd` - Guest process command being prepared for spawn.
* `grant` - Effective covering grant for this plugin.

```rust
fn inject_workerd_grant_env(cmd: & mut tokio::process::Command, grant: &PluginGrant)
```



## bookclerk_plugin_host::consent::is_platform_envelope

*Function*

True when `grant` stays inside the platform installer envelope (deny
network, only `config` / `work_fs` bindings): the request a Bookclerk
platform plugin may receive without the operator being asked.

```rust
fn is_platform_envelope(grant: &PluginGrant) -> bool
```



## bookclerk_plugin_host::consent::jail_process_overhead

*Function*

Fixed payload occupancy for the launcher / primary guest tree.

Workerd isolate: `bookclerk-workerd` plus the `workerd` child (2).
Native behind workerd: those two plus the native guest (3). Direct native
(diagnostic transport): the guest (1). The Windows outer Job adds the two
jail supervisors on top of the native-behind payload; see
[`WINDOWS_SESSION_PROCESS_BASELINE`](consent.md#bookclerk_plugin_hostconsentwindows_session_process_baseline).

```rust
fn jail_process_overhead(runtime: crate::spawn_plan::GuestRuntimeKind) -> u32
```



## bookclerk_plugin_host::consent::network_compatible

*Function*

Returns true when a stored network mode can satisfy a requested network mode.

# Arguments

* `existing` - Network mode already approved by the operator.
* `requested` - Network mode requested by the current plugin manifest.

# Returns

`true` when the modes match, or when an existing `deny` grant is stricter
than a requested `outbound` grant.

```rust
fn network_compatible(existing: &str, requested: &str) -> bool
```



## bookclerk_plugin_host::consent::node_logical_cpus

*Function*

Logical CPUs visible to this process (at least 1).

```rust
fn node_logical_cpus() -> u32
```



## bookclerk_plugin_host::consent::overlay_host_implied_network

*Function*

Overlay TCP + CIDR implied by host configuration (not persisted).

Not operator-invented structural authority. The operator already configured
the destination URL; spawn injects matching `EgressPolicy` so the nested
native guest can `connect()` it. Covers:

- active Postgres `[database.postgres].url`
- active D1 `[database.d1].api_base`
- Audiobookshelf `[integrations.audiobookshelf].base_url`
- S3 `[output.s3].endpoint` (custom/MinIO)

Occupancy is uniquified with [`crate::resolve_plugin_slot`]: a bare alias
overlays only the unique occupant, and a PluginKey never overlays a twin.

No-op unless the covering grant is outbound.

# Arguments

* `grant` - Effective covering grant (mutated in place).
* `plugin` - Guest being spawned.
* `config` - Host config (URL already env-applied).
* `discovered` - Installs used to uniquify occupancy (must include `plugin`).

```rust
fn overlay_host_implied_network(grant: & mut PluginGrant, plugin: &crate::discover::DiscoveredPlugin, config: &bookclerk_config::Config, discovered: &[crate::discover::DiscoveredPlugin])
```



## bookclerk_plugin_host::consent::pending_structural

*Function*

Structural capabilities the current manifest requests that the stored grant
does not yet authorize.

```rust
fn pending_structural(existing: &PluginGrant, requested: &PluginGrant) -> PendingStructural
```



## bookclerk_plugin_host::consent::percent_to_cores

*Function*

Convert Spec/config percent-of-one-core (100 = 1.00 core) to cores (2 d.p.).

```rust
fn percent_to_cores(percent: u32) -> f64
```



## bookclerk_plugin_host::consent::queue_consumer_grant

*Function*

Grant token for consuming `queue`.

```rust
fn queue_consumer_grant(queue: &str) -> String
```



## bookclerk_plugin_host::consent::queue_producer_grant

*Function*

Grant token for sending through the `[[queues.producers]]` binding `binding`.

```rust
fn queue_producer_grant(binding: &str) -> String
```



## bookclerk_plugin_host::consent::reconcile_host_overlay_authority

*Function*

Recomputes effective overlay authority and fences sessions that no longer match.

Host-config destination changes do not touch `plugin-grants.json`, so the
grant watcher cannot see them. Config reload must call this.

A deployed S3 session is recomputed with the document endpoint recorded at
registration. That path does not read `[output.s3].endpoint`, so a settings
reload cannot cancel an in-flight upload because the TOML endpoint differs
or is unset. Transitional sessions still overlay the TOML endpoint.

A revision mismatch fences that session only. Several vats may share one
PluginKey with different effective revisions, and cancelling the key would
kill a deployed upload when a transitional session's TOML endpoint changes.

```rust
fn reconcile_host_overlay_authority(config: &bookclerk_config::Config)
```



## bookclerk_plugin_host::consent::require_binding

*Function*

Fail closed when a delivery site needs a binding the covering grant lacks.

# Errors

Returns an error when the operation fails.

```rust
fn require_binding(grant: &PluginGrant, name: &str) -> crate::Result<()>
```



## bookclerk_plugin_host::consent::require_grant

*Function*

Require an operator grant before enable **or** every external spawn.

# Arguments

* `files_dir` - Bookclerk files directory containing `plugin-grants.json`.
* `manifest` - Plugin manifest whose id must have a stored grant.

# Returns

Effective [`PluginGrant`](consent.md#bookclerk_plugin_hostconsentplugingrant) (operator approval, host-normalized).

# Errors

Returns [`PluginError`](error.md#bookclerk_plugin_hosterrorpluginerror) when no grant exists for the plugin id.

```rust
fn require_grant(files_dir: &std::path::Path, plugin: &crate::discover::DiscoveredPlugin) -> crate::Result<PluginGrant>
```



## bookclerk_plugin_host::consent::spawn_config_for_grant

*Function*

Spawn `config` payload: non-empty settings only when the grant includes `config`.

```rust
fn spawn_config_for_grant(grant: &PluginGrant, config_table: serde_json::Value) -> serde_json::Value
```



## bookclerk_plugin_host::consent::spawn_grant

*Function*

Resolve the covering grant for spawn: verified platform auto-grant, else [`require_grant`](consent.md#bookclerk_plugin_hostconsentrequire_grant).

# Errors

Returns an error when the operation fails.

```rust
fn spawn_grant(files_dir: &std::path::Path, plugin: &crate::discover::DiscoveredPlugin) -> crate::Result<PluginGrant>
```



## bookclerk_plugin_host::consent::validate_approved_grant

*Function*

Validates and normalizes an operator-supplied grant against host hard caps.

The manifest `baseline` supplies identity defaults and suggested values.
Operators may **narrow** structural capabilities (entrypoints, producers,
host bindings, flags) and **widen or narrow** network destinations.
Inventing a structural capability the package did not declare is rejected.
Workerd / disk limits clamp to host maximums.

# Arguments

* `approved` - Operator-supplied grant draft.
* `baseline` - Consent request generated from the current manifest.

# Returns

A normalized grant with plugin identity and a fresh `approved_at`.

# Errors

Returns [`PluginError`](error.md#bookclerk_plugin_hosterrorpluginerror) when identity mismatches, network mode is invalid,
a binding is unknown, or a domain pattern cannot be normalized.

# Panics

Panics when an internal invariant does not hold.

```rust
fn validate_approved_grant(approved: &PluginGrant, baseline: &PluginGrant) -> crate::Result<PluginGrant>
```



## bookclerk_plugin_host::consent::validate_described_capabilities

*Function*

Reject `describe()` capability claims that exceed the manifest and the
covering grant.

The guest's typed [`PluginCapabilities`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedplugincapabilities) must not widen what
`plugin.toml` declares: no extra entrypoints, event consumers, producers,
job types, database bindings, or named bindings. Every advertised
entrypoint and producer must also be present in the operator grant, and
OAuth storefronts need the `oauth` binding.

# Arguments

* `manifest` - Installed manifest for the plugin.
* `grant` - Effective covering grant.
* `described` - Capability block the guest returned from `describe()`.
* `portal_auth_mode` - Storefront connect mode from `describe()`.

# Errors

Returns an error naming the first widening capability or missing grant.

```rust
fn validate_described_capabilities(manifest: &crate::manifest::PluginManifest, grant: &PluginGrant, described: &bookclerk_plugin_abi::PluginCapabilities, portal_auth_mode: bookclerk_plugin_abi::PortalAuthMode) -> crate::Result<()>
```



## bookclerk_plugin_host::consent::validate_described_config_options

*Function*

Validates `describe().configOptions` against the manifest `[[settings]]`.

The manifest is the authority for the settings a plugin takes. An empty
list means the guest defers to the manifest. A non-empty list must name
the same keys with the same enum value ids (labels may differ), so a guest
cannot widen or narrow its own writable keys at runtime.

# Arguments

* `manifest` - Installed manifest for the plugin.
* `described` - `configOptions` the guest returned from `describe()`.

# Errors

Returns an error naming the first key or value that differs.

```rust
fn validate_described_config_options(manifest: &crate::manifest::PluginManifest, described: &[bookclerk_plugin_abi::ConfigOption]) -> crate::Result<()>
```



## bookclerk_plugin_host::consent::windows_session_active_processes

*Function*

Windows outer session Job active-process cap: baseline 5 plus `extra`.

`extra` is clamped to [`PLUGIN_JAIL_EXTRA_PROCESSES_MAX`](consent.md#bookclerk_plugin_hostconsentplugin_jail_extra_processes_max). The sum is
clamped to [`PLUGIN_JAIL_ACTIVE_PROCESSES_MAX`](consent.md#bookclerk_plugin_hostconsentplugin_jail_active_processes_max). Inner jail caps are not
inflated by this baseline: the gateway stays at 2 and the guest stays at
1 plus the same extra budget.

```rust
fn windows_session_active_processes(extra: u32) -> u32
```



