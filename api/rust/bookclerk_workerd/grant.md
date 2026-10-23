**bookclerk_workerd > grant**

# Module: grant

## Contents

**Structs**

- [`OperatorGrantEnv`](#operatorgrantenv) - Optional operator grant fields parsed from the process environment.

**Constants**

- [`GRANT_CPU_MS_ENV`](#grant_cpu_ms_env) - Env: workerd CPU budget override in milliseconds.
- [`GRANT_DOMAINS_ENV`](#grant_domains_env) - Env: comma-separated initial-host domain patterns (grant subset).
- [`GRANT_NETWORK_MODE_ENV`](#grant_network_mode_env) - Env: network mode override (`deny` or `outbound`).
- [`GRANT_POLICY_ENV`](#grant_policy_env) - Env: canonical JSON [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#bookclerk_plugin_manifestegressegresspolicy) (preferred over piecemeal domain vars).
- [`GRANT_SUBREQUESTS_ENV`](#grant_subrequests_env) - Env: workerd subrequest budget override.

---

## bookclerk_workerd::grant::GRANT_CPU_MS_ENV

*Constant*: `&str`

Env: workerd CPU budget override in milliseconds.



## bookclerk_workerd::grant::GRANT_DOMAINS_ENV

*Constant*: `&str`

Env: comma-separated initial-host domain patterns (grant subset).



## bookclerk_workerd::grant::GRANT_NETWORK_MODE_ENV

*Constant*: `&str`

Env: network mode override (`deny` or `outbound`).

Must stay aligned with `bookclerk_plugin_host::consent::WORKERD_GRANT_*_ENV`.



## bookclerk_workerd::grant::GRANT_POLICY_ENV

*Constant*: `&str`

Env: canonical JSON [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#bookclerk_plugin_manifestegressegresspolicy) (preferred over piecemeal domain vars).



## bookclerk_workerd::grant::GRANT_SUBREQUESTS_ENV

*Constant*: `&str`

Env: workerd subrequest budget override.



## bookclerk_workerd::grant::OperatorGrantEnv

*Struct*

Optional operator grant fields parsed from the process environment.

**Fields:**
- `network_mode: Option<bookclerk_plugin_manifest::NetworkMode>` - Stricter-or-equal network mode from the stored grant.
- `domains: Option<Vec<String>>` - Domain allowlist subset (`Some` even when empty).
- `policy: Option<bookclerk_plugin_manifest::EgressPolicy>` - Canonical policy JSON when the host injected [`GRANT_POLICY_ENV`](grant.md#bookclerk_workerdgrantgrant_policy_env).
- `cpu_ms: Option<u32>` - CPU budget override (milliseconds).
- `subrequests: Option<u32>` - Subrequest budget override.

**Methods:**

- `fn from_env() -> Self` - Reads grant override env vars set by the plugin host at spawn.
- `fn is_empty(self: &Self) -> bool` - Returns true when no grant override env vars were present.
- `fn apply_egress(self: &Self, manifest: &PluginManifest, egress: EgressProxy) -> EgressProxy` - Narrows a manifest-derived egress proxy to the operator grant.
- `fn apply_limits(self: &Self, limits: EffectiveWorkerdLimits) -> EffectiveWorkerdLimits` - Narrows effective workerd limits to the operator grant (host-capped).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> OperatorGrantEnv`
- **PartialEq**
  - `fn eq(self: &Self, other: &OperatorGrantEnv) -> bool`
- **Default**
  - `fn default() -> OperatorGrantEnv`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



