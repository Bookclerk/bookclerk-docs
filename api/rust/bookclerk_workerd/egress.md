**bookclerk_workerd > egress**

# Module: egress

## Contents

**Structs**

- [`EgressProxy`](#egressproxy) - Initial-host allowlist with redirect-follow semantics.

---

## bookclerk_workerd::egress::EgressProxy

*Struct*

Initial-host allowlist with redirect-follow semantics.

Thin wrapper kept for call sites; core matching lives in
[`bookclerk_plugin_manifest::EgressPolicy`](../bookclerk_plugin_manifest/egress.md#bookclerk_plugin_manifestegressegresspolicy).

**Methods:**

- `fn from_manifest(manifest: &PluginManifest) -> Self` - Builds egress constraints from a plugin manifest network section.
- `fn from_policy(inner: EgressPolicy) -> Self` - Builds egress constraints from an already-approved host policy.
- `fn policy(self: &Self) -> &EgressPolicy` - Returns the underlying egress policy for inspection / logging.
- `fn mode(self: &Self) -> NetworkMode` - Network mode from the plugin manifest.
- `fn max_redirects(self: &Self) -> u32` - Maximum redirect hops after an allowed initial request.
- `fn allowed_initial_hosts(self: &Self) -> &[String]` - Hostnames the guest may dial before redirects.
- `fn allows_initial_host(self: &Self, host: &str) -> bool` - Whether a *direct* (non-redirect) request host is permitted.
- `fn allows_redirect_hop(self: &Self, host: &str, hop_index: u32) -> bool` - Redirect hops must stay in the fetch allowlist unless

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EgressProxy`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



