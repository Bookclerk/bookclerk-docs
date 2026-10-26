# bookclerk_plugin_manifest::egress

Shared host/egress allowlist policy (workerd JS bridge + native mediator).

Semantics match `bookclerk-workerd` bridge `egress.js` and the product rules
in `docs/plugins/README.md`:

- **Initial** request hosts must match [`EgressPolicy::domains`](#egresspolicydomains) (`*.`
  prefix wildcards). Hosts and patterns are IDNA ToASCII–normalized;
  percent-encoded hosts fail closed.
- **Redirect** hops must stay on the fetch allowlist unless the operator
  grants [`EgressPolicy::allow_undeclared_public_redirects`](#egresspolicy). That flag
  never implies loopback, RFC1918, link-local, or metadata.
- Hop count is capped by [`EgressPolicy::max_redirects`](#egresspolicymax_redirects) (wire
  `maxRedirects`).
- **Python + outbound** workerd guests also receive
  [`PYODIDE_EGRESS_HOSTS`](#pyodide_egress_hosts) in consent and egress lists.

Native-behind-workerd guests use the same policy through the inherited
CONNECT mux; ambient `AF_INET`/`AF_INET6` is denied by the sibling jail. See
[`crate::NetworkMode`](types.md#networkmode) and [`crate::PluginManifest::validate`](types.md#pluginmanifestvalidate).

| Item | Kind | Summary |
| --- | --- | --- |
| [`EgressPolicy`](#egresspolicy) | struct | Initial-host allowlist with redirect, TCP, and address-space policy. |
| [`TcpGrant`](#tcpgrant) | struct | One TCP grant in the injected policy (`host` + `ports`). |
| [`manifest_needs_python`](#manifest_needs_python) | function | Returns `true` when the manifest declares a Python workerd guest. |
| [`with_python_runtime_hosts`](#with_python_runtime_hosts) | function | Appends `PYODIDE_EGRESS_HOSTS` when `needs_python` and mode is outbound. |
| [`consent_domains_for`](#consent_domains_for) | function | Builds the consent / egress domain list for a manifest. |
| [`normalize_hostname`](#normalize_hostname) | function | Normalizes a hostname with IDNA ToASCII (fail closed). |
| [`normalize_domain_pattern`](#normalize_domain_pattern) | function | Normalizes an allowlist pattern (`*.example.com` or exact host) via IDNA. |
| [`host_matches`](#host_matches) | function | True when an IDNA-normalized request host is covered by an allowlist pattern. |
| [`DEFAULT_MAX_REDIRECTS`](#default_max_redirects) | constant | Default redirect budget injected when building policy from network caps. |
| [`PYODIDE_EGRESS_HOSTS`](#pyodide_egress_hosts) | constant | Hosts Pyodide / Cloudflare Python Workers commonly fetch on first boot (runtime index + micropip). |

## Structs

### EgressPolicy

```rust
pub struct EgressPolicy {
    pub mode: NetworkMode,
    pub domains: Vec<String>,
    pub max_redirects: u32,
    pub subrequests: Option<u32>,
    pub allow_undeclared_public_redirects: bool,
    pub tcp: Vec<TcpGrant>,
    pub address_cidrs: Vec<String>,
}
```

Initial-host allowlist with redirect, TCP, and address-space policy.

Serialized as camelCase for injection into workerd `EGRESS_POLICY`.

#### Matching rules

- [`Self::allows_initial`](#egresspolicyallows_initial) — hostname must match [`Self::domains`](#egresspolicydomains) after
  IDNA normalization. IP literals also pass [`Self::allows_ip`](#egresspolicyallows_ip).
- [`Self::allows_redirect`](#egresspolicyallows_redirect) — hop budget plus either the domain allowlist
  or [`Self::allow_undeclared_public_redirects`](#egresspolicy) for **public** targets.
- [`Self::allows_tcp`](#egresspolicyallows_tcp) — host + port must match a TCP grant. Fetch grants
  never imply TCP.
- [`Self::allows_ip`](#egresspolicyallows_ip) — public Internet by default; extra CIDRs are
  explicit. Redirect-to-undeclared-public never implies private ranges.

**Fields**

- `mode`: Network mode (`deny` or `outbound`). Wire name: `mode`.
- `domains`: IDNA-normalized initial-request host patterns (`*.cdn.example.com` or
  exact hosts). Empty when mode is deny. Wire name: `domains`.
- `max_redirects`: Maximum redirect hops after an allowed initial request (0-based hop
  index must be `< max_redirects`). Wire name: `maxRedirects`. Defaults
  to [`DEFAULT_MAX_REDIRECTS`](#default_max_redirects).
- `subrequests`: Max outbound `fetch` calls (initial request plus redirect hops that
  actually fetch).
  
  When present and finite, the workerd egress bridge enforces this
  counter **per egress invocation**. Absent means unlimited (tests /
  native mediator). Workerd guests always inject the clamped
  `[workerd].limits.subrequests` value via [`Self::from_manifest`](#egresspolicyfrom_manifest).
  Wire name: `subrequests`.
- `allow_undeclared_public_redirects`: When true, redirect hops may target undeclared public hosts.
  
  Private / loopback / link-local / metadata destinations still require
  an address-space grant.
- `tcp`: Raw TCP grants (`connect`). Fetch allowlists do not imply these.
- `address_cidrs`: Explicit CIDR grants beyond the public Internet.

#### EgressPolicy::from_network

```rust
pub fn from_network(caps: &NetworkCapabilities) -> Self
```

Builds a policy from network capabilities, normalizing domains.

Invalid allowlist entries cause a fail-closed [`Self::deny`](#egresspolicydeny) policy
(prefer [`Self::try_from_network`](#egresspolicytry_from_network) when you need the error).

##### Arguments

* `caps` - `[capabilities.network]` table from the manifest.

##### Returns

An outbound or deny policy with normalized domains and
[`DEFAULT_MAX_REDIRECTS`](#default_max_redirects). `subrequests` is unset.

#### EgressPolicy::try_from_network

```rust
pub fn try_from_network(caps: &NetworkCapabilities) -> Result<Self, String>
```

Like [`Self::from_network`](#egresspolicyfrom_network) but returns an error when any allowlist
entry fails IDNA / percent-encoding checks.

##### Arguments

* `caps` - `[capabilities.network]` table from the manifest.

##### Returns

Policy with deduplicated, IDNA-normalized domain patterns.

##### Errors

Returns a string describing the first invalid domain pattern.

#### EgressPolicy::from_manifest

```rust
pub fn from_manifest(manifest: &PluginManifest) -> Self
```

Builds policy from a full plugin manifest.

Appends [`PYODIDE_EGRESS_HOSTS`](#pyodide_egress_hosts) for Python + outbound workerd guests
and sets `subrequests` from clamped `[workerd].limits` when present.
Invalid allowlist entries fail closed to [`Self::deny`](#egresspolicydeny).

##### Arguments

* `manifest` - Validated (or at least deserialized) plugin descriptor.

##### Returns

Egress policy ready for workerd injection or consent materialization.

#### EgressPolicy::try_from_manifest

```rust
pub fn try_from_manifest(manifest: &PluginManifest) -> Result<Self, String>
```

Like [`Self::from_manifest`](#egresspolicyfrom_manifest) but fails closed on invalid allowlist entries.

##### Arguments

* `manifest` - Plugin descriptor providing network caps and optional
  `[workerd]` limits.

##### Returns

Policy with Python runtime hosts (when applicable) and clamped
subrequest budget when `[workerd]` is present.

##### Errors

Returns a string describing the first invalid domain in
`capabilities.network.domains`.

#### EgressPolicy::with_subrequests

```rust
pub fn with_subrequests(self, subrequests: u32) -> Self
```

Attaches a clamped subrequest budget (typically from
[`crate::WorkerdLimits::effective`](types.md#workerdlimitseffective)).

##### Arguments

* `subrequests` - Maximum fetch count for the workerd egress bridge.

##### Returns

`self` with `subrequests` set to `Some(subrequests)`.

#### EgressPolicy::deny

```rust
pub fn deny() -> Self
```

Returns a deny-all policy (no domains, default redirect budget, no
subrequest cap).

Used when allowlist normalization fails (`from_*` helpers) or when
network mode is deny.

#### EgressPolicy::mode

```rust
pub fn mode(&self) -> NetworkMode
```

Returns the configured [`NetworkMode`](types.md#networkmode).

#### EgressPolicy::domains

```rust
pub fn domains(&self) -> &[String]
```

Returns the IDNA-normalized domain allowlist (may include Pyodide hosts).

#### EgressPolicy::max_redirects

```rust
pub fn max_redirects(&self) -> u32
```

Returns the redirect hop budget (`maxRedirects` on the wire).

#### EgressPolicy::allows_initial

```rust
pub fn allows_initial(&self, host: &str) -> bool
```

Returns whether a *direct* (non-redirect) request host is permitted.

Deny mode always returns `false`. Outbound mode normalizes `host` via
[`normalize_hostname`](#normalize_hostname) and matches against [`Self::domains`](#egresspolicydomains).

##### Arguments

* `host` - Request hostname (may include a trailing dot; Unicode or
  Punycode). Percent-encoded hosts never match.

##### Returns

`true` when mode is outbound and the host matches an allowlist entry.

#### EgressPolicy::allows_ip

```rust
pub fn allows_ip(&self, ip: IpAddr) -> bool
```

True when `ip` is the public Internet or an explicit CIDR grant.

#### EgressPolicy::allows_tcp

```rust
pub fn allows_tcp(&self, host: &str, port: u16) -> bool
```

Raw TCP `connect` to `host:port`. Fetch domain grants do not apply.

#### EgressPolicy::allows_redirect

```rust
pub fn allows_redirect(&self, host: &str, hop_index: u32) -> bool
```

Returns whether a redirect hop is allowed.

Deny mode always returns `false`. Outbound mode requires
`hop_index < max_redirects`. The hop host must match the fetch
allowlist unless [`Self::allow_undeclared_public_redirects`](#egresspolicy) is set,
in which case only public destinations (plus explicit CIDRs) pass.

#### EgressPolicy::to_policy_json

```rust
pub fn to_policy_json(&self) -> Value
```

Serializes this policy to the JSON value injected as workerd
`EGRESS_POLICY` (and the native helper wire form).

On unlikely serialize failure, returns a deny-shaped object with
`maxRedirects` set to [`DEFAULT_MAX_REDIRECTS`](#default_max_redirects).

##### Returns

CamelCase JSON (`mode`, `domains`, `maxRedirects`, optional
`subrequests`).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### TcpGrant

```rust
pub struct TcpGrant {
    pub host: String,
    pub ports: Vec<u16>,
}
```

One TCP grant in the injected policy (`host` + `ports`).

**Fields**

- `host`: Hostname or `*.` wildcard (IDNA-normalized).
- `ports`: Destination ports. Empty is fail-closed.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

## Functions

### manifest_needs_python

```rust
pub fn manifest_needs_python(manifest: &PluginManifest) -> bool
```

Returns `true` when the manifest declares a Python workerd guest.

Detection matches [`PluginManifest::declares_python`]: a `.py` main module
or a `[[modules]]` row with `type = "python"` or a `.py` load-set key
(explicit `path`, otherwise `name`). Compatibility
flags are not evidence, so a JS guest cannot pull Pyodide hosts in by
listing `python_workers`. A `.py` file that exists only on disk does not
widen this list; materialize must use the same set as consent.

Native runtimes always return `false`.

#### Arguments

* `manifest` - Plugin descriptor to inspect.

#### Returns

`true` when Pyodide CDN hosts should be merged into consent/egress lists
for outbound guests.

### with_python_runtime_hosts

```rust
pub fn with_python_runtime_hosts(
    needs_python: bool,
    mode: NetworkMode,
    base: &[String],
) -> Vec<String>
```

Appends [`PYODIDE_EGRESS_HOSTS`](#pyodide_egress_hosts) when `needs_python` and mode is outbound.

Existing entries that already match a Pyodide host (case-insensitive) are
not duplicated.

#### Arguments

* `needs_python` - Typically from [`manifest_needs_python`](#manifest_needs_python).
* `mode` - Network mode; hosts are only appended for
  [`NetworkMode::Outbound`](types.md#networkmode).
* `base` - Already-normalized author allowlist.

#### Returns

A new `Vec` with base domains plus any missing Pyodide hosts.

### consent_domains_for

```rust
pub fn consent_domains_for(manifest: &PluginManifest) -> Result<Vec<String>, String>
```

Builds the consent / egress domain list for a manifest.

Normalizes `capabilities.network.domains` via IDNA and appends Pyodide
hosts when [`manifest_needs_python`](#manifest_needs_python) and mode is outbound.

#### Arguments

* `manifest` - Plugin descriptor providing network caps and runtime hints.

#### Returns

Deduplicated, IDNA-normalized host patterns for operator consent UI and
materialization.

#### Errors

Returns a string when any declared domain fails
[`normalize_domain_pattern`](#normalize_domain_pattern).

### normalize_hostname

```rust
pub fn normalize_hostname(host: &str) -> Option<String>
```

Normalizes a hostname with IDNA ToASCII (fail closed).

Trims ASCII whitespace and a single trailing dot. Rejects empty strings
and any host containing `%` (percent-encoding bypass).

#### Arguments

* `host` - Raw request or allowlist hostname.

#### Returns

ASCII/Punycode hostname, or `None` when IDNA fails or the input is
percent-encoded / empty.

### normalize_domain_pattern

```rust
pub fn normalize_domain_pattern(pattern: &str) -> Option<String>
```

Normalizes an allowlist pattern (`*.example.com` or exact host) via IDNA.

The `*.` prefix is preserved; only the suffix is ToASCII-normalized.
Percent-encoded patterns and empty patterns return `None`.

#### Arguments

* `pattern` - Author-supplied domain entry from
  `capabilities.network.domains`.

#### Returns

Normalized pattern suitable for storage on [`EgressPolicy::domains`](#egresspolicydomains), or
`None` on failure.

### host_matches

```rust
pub fn host_matches(host: &str, pattern: &str) -> bool
```

True when an IDNA-normalized request host is covered by an allowlist pattern.

Both sides are IDNA-normalized first; invalid hosts or patterns never
match. Wildcard patterns (`*.cdn.example.com`) match the bare suffix and
any subdomain (`cdn.example.com`, `a.cdn.example.com`) but not
`notcdn.example.com`.

#### Arguments

* `host` - Request hostname to test.
* `pattern` - Exact host or `*.` prefix wildcard pattern.

#### Returns

`true` when both normalize successfully and the host is covered by the
pattern.

#### Examples

```rust
use bookclerk_plugin_manifest::host_matches;

assert!(host_matches("API.Example.COM", "api.example.com"));
assert!(host_matches("www.cdn.example.com", "*.cdn.example.com"));
assert!(!host_matches("evil.com", "api.example.com"));
```

## Constants

### DEFAULT_MAX_REDIRECTS

```rust
pub const DEFAULT_MAX_REDIRECTS: u32 = 10;
```

Default redirect budget injected when building policy from network caps.

Matches workerd policy injection (`maxRedirects = 10`). Wire JSON uses
camelCase `maxRedirects`.

### PYODIDE_EGRESS_HOSTS

```rust
pub const PYODIDE_EGRESS_HOSTS: &[&str] = _;
```

Hosts Pyodide / Cloudflare Python Workers commonly fetch on first boot
(runtime index + micropip).

Included in consent UI lists and [`EgressPolicy`](#egresspolicy) domain allowlists when a
workerd guest is Python **and** [`NetworkMode::Outbound`](types.md#networkmode). See Cloudflare
Python packages docs. Values:
`cdn.jsdelivr.net`, `pypi.org`, `files.pythonhosted.org`.
