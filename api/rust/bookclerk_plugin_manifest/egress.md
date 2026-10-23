**bookclerk_plugin_manifest > egress**

# Module: egress

## Contents

**Structs**

- [`EgressPolicy`](#egresspolicy) - Initial-host allowlist with redirect, TCP, and address-space policy.
- [`TcpGrant`](#tcpgrant) - One TCP grant in the injected policy (`host` + `ports`).

**Functions**

- [`consent_domains_for`](#consent_domains_for) - Builds the consent / egress domain list for a manifest.
- [`host_matches`](#host_matches) - True when an IDNA-normalized request host is covered by an allowlist pattern.
- [`manifest_needs_python`](#manifest_needs_python) - Returns `true` when the manifest declares a Python workerd guest.
- [`normalize_domain_pattern`](#normalize_domain_pattern) - Normalizes an allowlist pattern (`*.example.com` or exact host) via IDNA.
- [`normalize_hostname`](#normalize_hostname) - Normalizes a hostname with IDNA ToASCII (fail closed).
- [`with_python_runtime_hosts`](#with_python_runtime_hosts) - Appends [`PYODIDE_EGRESS_HOSTS`](egress.md#bookclerk_plugin_manifestegresspyodide_egress_hosts) when `needs_python` and mode is outbound.

**Constants**

- [`DEFAULT_MAX_REDIRECTS`](#default_max_redirects) - Default redirect budget injected when building policy from network caps.
- [`PYODIDE_EGRESS_HOSTS`](#pyodide_egress_hosts) - Hosts Pyodide / Cloudflare Python Workers commonly fetch on first boot

---

## bookclerk_plugin_manifest::egress::DEFAULT_MAX_REDIRECTS

*Constant*: `u32`

Default redirect budget injected when building policy from network caps.

Matches workerd policy injection (`maxRedirects = 10`). Wire JSON uses
camelCase `maxRedirects`.



## bookclerk_plugin_manifest::egress::EgressPolicy

*Struct*

Initial-host allowlist with redirect, TCP, and address-space policy.

Serialized as camelCase for injection into workerd `EGRESS_POLICY`.

# Matching rules

- `Self::allows_initial` — hostname must match `Self::domains` after
  IDNA normalization. IP literals also pass `Self::allows_ip`.
- `Self::allows_redirect` — hop budget plus either the domain allowlist
  or `Self::allow_undeclared_public_redirects` for **public** targets.
- `Self::allows_tcp` — host + port must match a TCP grant. Fetch grants
  never imply TCP.
- `Self::allows_ip` — public Internet by default; extra CIDRs are
  explicit. Redirect-to-undeclared-public never implies private ranges.

**Fields:**
- `mode: crate::types::NetworkMode` - Network mode (`deny` or `outbound`). Wire name: `mode`.
- `domains: Vec<String>` - IDNA-normalized initial-request host patterns (`*.cdn.example.com` or
- `max_redirects: u32` - Maximum redirect hops after an allowed initial request (0-based hop
- `subrequests: Option<u32>` - Max outbound `fetch` calls (initial request plus redirect hops that
- `allow_undeclared_public_redirects: bool` - When true, redirect hops may target undeclared public hosts.
- `tcp: Vec<TcpGrant>` - Raw TCP grants (`connect`). Fetch allowlists do not imply these.
- `address_cidrs: Vec<String>` - Explicit CIDR grants beyond the public Internet.

**Methods:**

- `fn from_network(caps: &NetworkCapabilities) -> Self` - Builds a policy from network capabilities, normalizing domains.
- `fn try_from_network(caps: &NetworkCapabilities) -> Result<Self, String>` - Like `Self::from_network` but returns an error when any allowlist
- `fn from_manifest(manifest: &PluginManifest) -> Self` - Builds policy from a full plugin manifest.
- `fn try_from_manifest(manifest: &PluginManifest) -> Result<Self, String>` - Like `Self::from_manifest` but fails closed on invalid allowlist entries.
- `fn with_subrequests(self: Self, subrequests: u32) -> Self` - Attaches a clamped subrequest budget (typically from
- `fn deny() -> Self` - Returns a deny-all policy (no domains, default redirect budget, no
- `fn mode(self: &Self) -> NetworkMode` - Returns the configured [`NetworkMode`](types.md#bookclerk_plugin_manifesttypesnetworkmode).
- `fn domains(self: &Self) -> &[String]` - Returns the IDNA-normalized domain allowlist (may include Pyodide hosts).
- `fn max_redirects(self: &Self) -> u32` - Returns the redirect hop budget (`maxRedirects` on the wire).
- `fn allows_initial(self: &Self, host: &str) -> bool` - Returns whether a *direct* (non-redirect) request host is permitted.
- `fn allows_ip(self: &Self, ip: std::net::IpAddr) -> bool` - True when `ip` is the public Internet or an explicit CIDR grant.
- `fn allows_tcp(self: &Self, host: &str, port: u16) -> bool` - Raw TCP `connect` to `host:port`. Fetch domain grants do not apply.
- `fn allows_redirect(self: &Self, host: &str, hop_index: u32) -> bool` - Returns whether a redirect hop is allowed.
- `fn to_policy_json(self: &Self) -> serde_json::Value` - Serializes this policy to the JSON value injected as workerd

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> EgressPolicy`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &EgressPolicy) -> bool`



## bookclerk_plugin_manifest::egress::PYODIDE_EGRESS_HOSTS

*Constant*: `&[&str]`

Hosts Pyodide / Cloudflare Python Workers commonly fetch on first boot
(runtime index + micropip).

Included in consent UI lists and [`EgressPolicy`](egress.md#bookclerk_plugin_manifestegressegresspolicy) domain allowlists when a
workerd guest is Python **and** `NetworkMode::Outbound`. See Cloudflare
Python packages docs. Values:
`cdn.jsdelivr.net`, `pypi.org`, `files.pythonhosted.org`.



## bookclerk_plugin_manifest::egress::TcpGrant

*Struct*

One TCP grant in the injected policy (`host` + `ports`).

**Fields:**
- `host: String` - Hostname or `*.` wildcard (IDNA-normalized).
- `ports: Vec<u16>` - Destination ports. Empty is fail-closed.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> TcpGrant`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &TcpGrant) -> $crate::option::Option<$crate::cmp::Ordering>`
- **PartialEq**
  - `fn eq(self: &Self, other: &TcpGrant) -> bool`
- **Ord**
  - `fn cmp(self: &Self, other: &TcpGrant) -> $crate::cmp::Ordering`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_manifest::egress::consent_domains_for

*Function*

Builds the consent / egress domain list for a manifest.

Normalizes `capabilities.network.domains` via IDNA and appends Pyodide
hosts when [`manifest_needs_python`](egress.md#bookclerk_plugin_manifestegressmanifest_needs_python) and mode is outbound.

# Arguments

* `manifest` - Plugin descriptor providing network caps and runtime hints.

# Returns

Deduplicated, IDNA-normalized host patterns for operator consent UI and
materialization.

# Errors

Returns a string when any declared domain fails
[`normalize_domain_pattern`](egress.md#bookclerk_plugin_manifestegressnormalize_domain_pattern).

```rust
fn consent_domains_for(manifest: &crate::types::PluginManifest) -> Result<Vec<String>, String>
```



## bookclerk_plugin_manifest::egress::host_matches

*Function*

True when an IDNA-normalized request host is covered by an allowlist pattern.

Both sides are IDNA-normalized first; invalid hosts or patterns never
match. Wildcard patterns (`*.cdn.example.com`) match the bare suffix and
any subdomain (`cdn.example.com`, `a.cdn.example.com`) but not
`notcdn.example.com`.

# Arguments

* `host` - Request hostname to test.
* `pattern` - Exact host or `*.` prefix wildcard pattern.

# Returns

`true` when both normalize successfully and the host is covered by the
pattern.

# Examples

```
use bookclerk_plugin_manifest::host_matches;

assert!(host_matches("API.Example.COM", "api.example.com"));
assert!(host_matches("www.cdn.example.com", "*.cdn.example.com"));
assert!(!host_matches("evil.com", "api.example.com"));
```

```rust
fn host_matches(host: &str, pattern: &str) -> bool
```



## bookclerk_plugin_manifest::egress::manifest_needs_python

*Function*

Returns `true` when the manifest declares a Python workerd guest.

Detection matches [`PluginManifest::declares_python`]: a `.py` main module
or a `[[modules]]` row with `type = "python"` or a `.py` load-set key
(explicit `path`, otherwise `name`). Compatibility
flags are not evidence, so a JS guest cannot pull Pyodide hosts in by
listing `python_workers`. A `.py` file that exists only on disk does not
widen this list; materialize must use the same set as consent.

Native runtimes always return `false`.

# Arguments

* `manifest` - Plugin descriptor to inspect.

# Returns

`true` when Pyodide CDN hosts should be merged into consent/egress lists
for outbound guests.

```rust
fn manifest_needs_python(manifest: &crate::types::PluginManifest) -> bool
```



## bookclerk_plugin_manifest::egress::normalize_domain_pattern

*Function*

Normalizes an allowlist pattern (`*.example.com` or exact host) via IDNA.

The `*.` prefix is preserved; only the suffix is ToASCII-normalized.
Percent-encoded patterns and empty patterns return `None`.

# Arguments

* `pattern` - Author-supplied domain entry from
  `capabilities.network.domains`.

# Returns

Normalized pattern suitable for storage on `EgressPolicy::domains`, or
`None` on failure.

```rust
fn normalize_domain_pattern(pattern: &str) -> Option<String>
```



## bookclerk_plugin_manifest::egress::normalize_hostname

*Function*

Normalizes a hostname with IDNA ToASCII (fail closed).

Trims ASCII whitespace and a single trailing dot. Rejects empty strings
and any host containing `%` (percent-encoding bypass).

# Arguments

* `host` - Raw request or allowlist hostname.

# Returns

ASCII/Punycode hostname, or `None` when IDNA fails or the input is
percent-encoded / empty.

```rust
fn normalize_hostname(host: &str) -> Option<String>
```



## bookclerk_plugin_manifest::egress::with_python_runtime_hosts

*Function*

Appends [`PYODIDE_EGRESS_HOSTS`](egress.md#bookclerk_plugin_manifestegresspyodide_egress_hosts) when `needs_python` and mode is outbound.

Existing entries that already match a Pyodide host (case-insensitive) are
not duplicated.

# Arguments

* `needs_python` - Typically from [`manifest_needs_python`](egress.md#bookclerk_plugin_manifestegressmanifest_needs_python).
* `mode` - Network mode; hosts are only appended for
  `NetworkMode::Outbound`.
* `base` - Already-normalized author allowlist.

# Returns

A new `Vec` with base domains plus any missing Pyodide hosts.

```rust
fn with_python_runtime_hosts(needs_python: bool, mode: crate::types::NetworkMode, base: &[String]) -> Vec<String>
```



