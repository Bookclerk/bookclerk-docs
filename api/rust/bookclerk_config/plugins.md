**bookclerk_config > plugins**

# Module: plugins

## Contents

**Structs**

- [`IntegrationsConfig`](#integrationsconfig) - Connect-portal settings under `[integrations]`.
- [`PluginRegistryEntry`](#pluginregistryentry) - One configured plugin registry source (`[[plugins.registries]]`).
- [`PluginsConfig`](#pluginsconfig) - `[plugins]` — how external plugin guests are run.
- [`PluginsJailConfig`](#pluginsjailconfig) - Optional global resource ceilings for plugin guest jails under `[plugins.jail]`.

**Functions**

- [`host_cpu_rate_max`](#host_cpu_rate_max) - Host CPU rate ceiling in one-core percent units (`logical_cpus × 100`).
- [`node_logical_cpus`](#node_logical_cpus) - Logical CPUs visible to this process (at least 1).

---

## bookclerk_config::plugins::IntegrationsConfig

*Struct*

Connect-portal settings under `[integrations]`.

Integration plugins themselves (Audiobookshelf, …) are deployed plugin
instances; their settings live on the instance, not here.

**Fields:**
- `claim_ticket_ttl_hours: u64` - Hours a connect-portal claim ticket remains valid (default 72).
- `public_origin: Option<String>` - Public HTTPS origin for portal redirects when behind a reverse proxy
- `portal_session_ttl_hours: u64` - Hours a portal browser session cookie remains valid (default 12).

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> IntegrationsConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &IntegrationsConfig) -> bool`
- **Default**
  - `fn default() -> Self`



## bookclerk_config::plugins::PluginRegistryEntry

*Struct*

One configured plugin registry source (`[[plugins.registries]]`).

**Fields:**
- `kind: String` - Adapter kind: `static`, `cargo`, `npm`, or `pypi`.
- `url: Option<String>` - Index / registry URL (required for `static`; optional overrides for others).
- `name: Option<String>` - Optional operator-facing name.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginRegistryEntry`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginRegistryEntry) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_config::plugins::PluginsConfig

*Struct*

`[plugins]` — how external plugin guests are run.

Separate from `[sources.*]` / `[integrations.*]`, which say *which* plugins
to load and pass them their own knobs. This says how the host runs whichever
ones it loads.

**Fields:**
- `isolation: crate::Isolation` - How strictly guest processes must be confined.
- `jail_bin: Option<std::path::PathBuf>` - Explicit path to `bookclerk-jail`. Normally left unset, in which case the
- `jail: PluginsJailConfig` - Optional global jail resource ceilings for all plugin guests.
- `registries: Vec<PluginRegistryEntry>` - Federated discovery sources (static indexes, cargo, npm, pypi).
- `allow_unverified_publisher: bool` - When true, community packages without independent publisher authenticity

**Methods:**

- `fn apply_env_overrides(self: & mut Self)` - Apply `BOOKCLERK_PLUGIN_*` environment overrides.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginsConfig`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginsConfig) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> PluginsConfig`



## bookclerk_config::plugins::PluginsJailConfig

*Struct*

Optional global resource ceilings for plugin guest jails under `[plugins.jail]`.

These override label defaults when building a guest jail spec.
Guest filesystem access remains install read-only plus host-managed data/tmp —
not a free-form path widen.

`cpu_rate_percent` is percent of **one logical CPU** (100 = one core) and is a
**per-jail ceiling** only (`min(grant, this, host_max)`). Quotas are rate
limits, not reservations: if many plugins’ ceilings sum above host capacity,
the OS scheduler shares CPU among runnable guests. Bookclerk does not keep a
cumulative Σ ledger or throttle later spawns.

**Fields:**
- `memory_mib: Option<u64>` - Soft memory ceiling in mebibytes (mapped to Spec `memory_bytes`).
- `cpu_rate_percent: Option<u32>` - Per-jail CPU rate ceiling (one-core percent units). Defaults to 80.
- `extra_processes: Option<u32>` - Ceiling on **extra** processes/threads beyond launcher overhead. Defaults to 2.

**Methods:**

- `fn clamp(self: & mut Self)` - Clamp CPU / extra-process ceilings into host-safe ranges when set.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PluginsJailConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginsJailConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_config::plugins::host_cpu_rate_max

*Function*

Host CPU rate ceiling in one-core percent units (`logical_cpus × 100`).

```rust
fn host_cpu_rate_max() -> u32
```



## bookclerk_config::plugins::node_logical_cpus

*Function*

Logical CPUs visible to this process (at least 1).

```rust
fn node_logical_cpus() -> u32
```



