# bookclerk_workerd::grant

Operator consent grant overrides delivered to `bookclerk-workerd` via env.

The plugin host injects these after resolving the effective grant at spawn.
Values always clamp to host [`WorkerdLimits`](../bookclerk_plugin_manifest/types.md#workerdlimits) caps and never widen beyond
the manifest-derived egress policy / limits.

| Item | Kind | Summary |
| --- | --- | --- |
| [`OperatorGrantEnv`](#operatorgrantenv) | struct | Optional operator grant fields parsed from the process environment. |
| [`GRANT_NETWORK_MODE_ENV`](#grant_network_mode_env) | constant | Env: network mode override (`deny` or `outbound`). |
| [`GRANT_DOMAINS_ENV`](#grant_domains_env) | constant | Env: comma-separated initial-host domain patterns (grant subset). |
| [`GRANT_CPU_MS_ENV`](#grant_cpu_ms_env) | constant | Env: workerd CPU budget override in milliseconds. |
| [`GRANT_SUBREQUESTS_ENV`](#grant_subrequests_env) | constant | Env: workerd subrequest budget override. |
| [`GRANT_POLICY_ENV`](#grant_policy_env) | constant | Env: canonical JSON `EgressPolicy` (preferred over piecemeal domain vars). |

## Structs

### OperatorGrantEnv

```rust
pub struct OperatorGrantEnv {
    pub network_mode: Option<NetworkMode>,
    pub domains: Option<Vec<String>>,
    pub policy: Option<EgressPolicy>,
    pub cpu_ms: Option<u32>,
    pub subrequests: Option<u32>,
}
```

Optional operator grant fields parsed from the process environment.

**Fields**

- `network_mode`: Stricter-or-equal network mode from the stored grant.
- `domains`: Domain allowlist subset (`Some` even when empty).
- `policy`: Canonical policy JSON when the host injected [`GRANT_POLICY_ENV`](#grant_policy_env).
- `cpu_ms`: CPU budget override (milliseconds).
- `subrequests`: Subrequest budget override.

#### OperatorGrantEnv::from_env

```rust
pub fn from_env() -> Self
```

Reads grant override env vars set by the plugin host at spawn.

Missing vars leave the corresponding field unset (manifest policy wins).
Invalid mode / non-numeric limits are ignored (fail open to manifest).

#### OperatorGrantEnv::is_empty

```rust
pub fn is_empty(&self) -> bool
```

Returns true when no grant override env vars were present.

#### OperatorGrantEnv::apply_egress

```rust
pub fn apply_egress(&self, manifest: &PluginManifest, egress: EgressProxy) -> EgressProxy
```

Narrows a manifest-derived egress proxy to the operator grant.

Domain patterns are IDNA-normalized; invalid entries are dropped.
Python + outbound guests still receive Pyodide CDN hosts.

#### OperatorGrantEnv::apply_limits

```rust
pub fn apply_limits(&self, limits: EffectiveWorkerdLimits) -> EffectiveWorkerdLimits
```

Narrows effective workerd limits to the operator grant (host-capped).

When a grant override is present it is authoritative within
[`WorkerdLimits`](../bookclerk_plugin_manifest/types.md#workerdlimits) hard maxes and may be **higher** than the manifest
defaults.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

## Constants

### GRANT_NETWORK_MODE_ENV

```rust
pub const GRANT_NETWORK_MODE_ENV: &str = "BOOKCLERK_WORKERD_GRANT_NETWORK_MODE";
```

Env: network mode override (`deny` or `outbound`).

Must stay aligned with `bookclerk_plugin_host::consent::WORKERD_GRANT_*_ENV`.

### GRANT_DOMAINS_ENV

```rust
pub const GRANT_DOMAINS_ENV: &str = "BOOKCLERK_WORKERD_GRANT_DOMAINS";
```

Env: comma-separated initial-host domain patterns (grant subset).

### GRANT_CPU_MS_ENV

```rust
pub const GRANT_CPU_MS_ENV: &str = "BOOKCLERK_WORKERD_GRANT_CPU_MS";
```

Env: workerd CPU budget override in milliseconds.

### GRANT_SUBREQUESTS_ENV

```rust
pub const GRANT_SUBREQUESTS_ENV: &str = "BOOKCLERK_WORKERD_GRANT_SUBREQUESTS";
```

Env: workerd subrequest budget override.

### GRANT_POLICY_ENV

```rust
pub const GRANT_POLICY_ENV: &str = "BOOKCLERK_WORKERD_GRANT_POLICY";
```

Env: canonical JSON [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#egresspolicy) (preferred over piecemeal domain vars).
