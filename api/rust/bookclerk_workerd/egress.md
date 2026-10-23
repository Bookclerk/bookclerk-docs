# bookclerk_workerd::egress

Host egress policy for workerd guests (wraps shared [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#egresspolicy)).

## Structs

### EgressProxy

```rust
pub struct EgressProxy {
    /* private fields */
}
```

Initial-host allowlist with redirect-follow semantics.

Thin wrapper kept for call sites; core matching lives in
[`bookclerk_plugin_manifest::EgressPolicy`](../bookclerk_plugin_manifest/egress.md#egresspolicy).

#### EgressProxy::from_manifest

```rust
pub fn from_manifest(manifest: &PluginManifest) -> Self
```

Builds egress constraints from a plugin manifest network section.

##### Arguments

* `manifest` - Parsed plugin manifest.

##### Returns

Updated `Self` for chaining.

#### EgressProxy::from_policy

```rust
pub fn from_policy(inner: EgressPolicy) -> Self
```

Builds egress constraints from an already-approved host policy.

##### Arguments

* `inner` - `inner` input for this call.

##### Returns

Updated `Self` for chaining.

#### EgressProxy::policy

```rust
pub fn policy(&self) -> &EgressPolicy
```

Returns the underlying egress policy for inspection / logging.

#### EgressProxy::mode

```rust
pub fn mode(&self) -> NetworkMode
```

Network mode from the plugin manifest.

#### EgressProxy::max_redirects

```rust
pub fn max_redirects(&self) -> u32
```

Maximum redirect hops after an allowed initial request.

#### EgressProxy::allowed_initial_hosts

```rust
pub fn allowed_initial_hosts(&self) -> &[String]
```

Hostnames the guest may dial before redirects.

#### EgressProxy::allows_initial_host

```rust
pub fn allows_initial_host(&self, host: &str) -> bool
```

Whether a *direct* (non-redirect) request host is permitted.

##### Arguments

* `host` - Hostname being checked for egress.

##### Returns

`true` when the predicate holds.

#### EgressProxy::allows_redirect_hop

```rust
pub fn allows_redirect_hop(&self, host: &str, hop_index: u32) -> bool
```

Redirect hops must stay in the fetch allowlist unless
`allowUndeclaredPublicRedirects` is set. Address-space policy still
applies (loopback / RFC1918 / link-local / metadata stay denied).

##### Arguments

* `host` - Redirect target hostname.
* `hop_index` - Numeric hop index after the initial request.

##### Returns

`true` when the predicate holds.

Implements: `Clone`, `Debug`
