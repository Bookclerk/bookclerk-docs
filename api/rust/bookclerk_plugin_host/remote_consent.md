**bookclerk_plugin_host > remote_consent**

# Module: remote_consent

## Contents

**Structs**

- [`RemoteNarrowing`](#remotenarrowing) - What a host operator takes out of an owner plugin's request.

**Enums**

- [`RemoteApprovalError`](#remoteapprovalerror) - Why [`approve_remote_request`](remote_consent.md#bookclerk_plugin_hostremote_consentapprove_remote_request) refused, by cause, so a caller (the
- [`RemoteConsentState`](#remoteconsentstate) - How this host stands on one owner plugin's request.

**Functions**

- [`approve_remote_request`](#approve_remote_request) - Approves the request recorded from `cluster_id` for `plugin_key`,
- [`narrow_remote_grant`](#narrow_remote_grant) - The grant a guest runs with: `requested ∩ consent`.
- [`remote_consent`](#remote_consent) - The consent an operator gives to `requested` with `narrowing` applied.
- [`remote_consent_state`](#remote_consent_state) - Evaluates a stored consent row's grants: the current request
- [`remote_consent_view`](#remote_consent_view) - The one JSON view of a consent row, shared by the operator API and the
- [`remote_platform_auto_approval`](#remote_platform_auto_approval) - True when a platform plugin's request may be approved without asking:
- [`remote_request_within`](#remote_request_within) - True when `requested` asks for nothing outside `reviewed`: the same
- [`remote_row_consent_state`](#remote_row_consent_state) - Evaluates a stored consent row: [`remote_consent_state`](remote_consent.md#bookclerk_plugin_hostremote_consentremote_consent_state) over its

---

## bookclerk_plugin_host::remote_consent::RemoteApprovalError

*Enum*

Why [`approve_remote_request`](remote_consent.md#bookclerk_plugin_hostremote_consentapprove_remote_request) refused, by cause, so a caller (the
operator API) can tell "review again" from "fix the input" from "retry
later".

**Variants:**
- `NotRequested(String)` - No request is recorded for that cluster and plugin.
- `Conflict(String)` - The recorded request is not the reviewed one (its digest differs),
- `Invalid(String)` - The recorded request does not parse, the narrowing names something
- `Store(String)` - The store failed (a fault that may pass).

**Traits:** Error

**Trait Implementations:**

- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **From**
  - `fn from(err: bookclerk_library::LibraryError) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::remote_consent::RemoteConsentState

*Enum*

How this host stands on one owner plugin's request.

**Variants:**
- `Consented(Box<crate::consent::PluginGrant>)` - Approved and the request is within what was reviewed: the guest
- `Pending` - Never approved (or revoked): waiting for the host's operator.
- `NeedsReapproval` - Approved once, but the request now asks for more than was reviewed
- `Invalid` - The request or the stored approval does not parse; nothing runs.

**Methods:**

- `fn grant(self: &Self) -> Option<&PluginGrant>` - The grant to run with, when consented.
- `fn as_str(self: &Self) -> &'static str` - Stable text for observations and listings.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteConsentState) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> RemoteConsentState`



## bookclerk_plugin_host::remote_consent::RemoteNarrowing

*Struct*

What a host operator takes out of an owner plugin's request.

**Fields:**
- `deny_domains: std::collections::BTreeSet<String>` - Outbound domain patterns to deny.
- `deny_bindings: std::collections::BTreeSet<String>` - Host bindings to deny (`secrets`, `oauth`, `kv`, ...).
- `deny_network: bool` - Turn outbound network off altogether.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RemoteNarrowing`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> RemoteNarrowing`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteNarrowing) -> bool`



## bookclerk_plugin_host::remote_consent::approve_remote_request

*Function*

Approves the request recorded from `cluster_id` for `plugin_key`,
exactly as it was reviewed: `reviewed_digest` must be the recorded
request's digest
(`bookclerk_library::control_plane::RemotePluginConsent::request_digest`,
shown with the request), and the consent is that request minus
`narrowing` ([`remote_consent`](remote_consent.md#bookclerk_plugin_hostremote_consentremote_consent)). The write is a compare-and-swap on the
recorded request, so a request that changed after it was read is
refused rather than approved unseen.

# Arguments

* `store` - This host's library.
* `cluster_id` - Owner cluster id.
* `plugin_key` - Canonical plugin key.
* `reviewed_digest` - Digest of the request as the approver reviewed it.
* `narrowing` - What the approver takes out of the request.
* `approver` - Who approves (an automatic approval never overrides an
  operator's revoke).

# Errors

Returns `RemoteApprovalError::NotRequested` when nothing is recorded,
`RemoteApprovalError::Conflict` when the recorded request is not the
reviewed one (its digest differs) or the store refuses the write (a
request that changed in between, or an automatic approval of a revoked
one), `RemoteApprovalError::Invalid` when it does not parse or the
narrowing names something it does not ask for, and
`RemoteApprovalError::Store` when a read or write fails.

```rust
fn approve_remote_request(store: &bookclerk_library::LibraryStore, cluster_id: &str, plugin_key: &str, reviewed_digest: &str, narrowing: &RemoteNarrowing, approver: bookclerk_library::control_plane::RemoteApprover) -> std::result::Result<bookclerk_library::control_plane::RemotePluginConsent, RemoteApprovalError>
```



## bookclerk_plugin_host::remote_consent::narrow_remote_grant

*Function*

The grant a guest runs with: `requested ∩ consent`.

Every structural set and network destination is intersected; network
is outbound only when both are; undeclared redirects only when both
allow; budgets take the smaller. Destinations the request asks for but
the result lacks are recorded as operator denials, so the spawn path's
manifest re-evaluation ([`crate::effective_grant`](consent.md#bookclerk_plugin_hostconsenteffective_grant)) keeps them out.

```rust
fn narrow_remote_grant(requested: &crate::consent::PluginGrant, consent: &crate::consent::PluginGrant) -> crate::consent::PluginGrant
```



## bookclerk_plugin_host::remote_consent::remote_consent

*Function*

The consent an operator gives to `requested` with `narrowing` applied.

# Errors

Returns an error when the narrowing names a domain or binding the
request does not ask for (it would deny nothing, and a typo must not
read as a narrowing that did not happen).

```rust
fn remote_consent(requested: &crate::consent::PluginGrant, narrowing: &RemoteNarrowing) -> crate::Result<crate::consent::PluginGrant>
```



## bookclerk_plugin_host::remote_consent::remote_consent_state

*Function*

Evaluates a stored consent row's grants: the current request
(`requested_json`), the request as reviewed and the consent (both
`None` while unapproved).

```rust
fn remote_consent_state(plugin_key: &str, requested_json: &str, reviewed_json: Option<&str>, consent_json: Option<&str>) -> RemoteConsentState
```



## bookclerk_plugin_host::remote_consent::remote_consent_view

*Function*

The one JSON view of a consent row, shared by the operator API and the
CLI so both answer the same fields: the request (version, grant, the
archive this host would fetch, and its `request_digest`), the approval
(`approved_version`, `grant`, `approved_at`, `approved_by`), `revoked_at`,
`updated_at`, and the `state` ([`remote_row_consent_state`](remote_consent.md#bookclerk_plugin_hostremote_consentremote_row_consent_state)). A stored
grant that does not parse shows as `null`.

```rust
fn remote_consent_view(row: &bookclerk_library::control_plane::RemotePluginConsent) -> serde_json::Value
```



## bookclerk_plugin_host::remote_consent::remote_platform_auto_approval

*Function*

True when a platform plugin's request may be approved without asking:
the key is a Bookclerk platform key, this host's own install of that key
is platform-bundled with the same payload digest as the owner's
artifact (`local_bundled_payload`, from this host's provenance check),
and the request stays inside the platform envelope (deny network, only
`config` / `work_fs`).

```rust
fn remote_platform_auto_approval(requested: &crate::consent::PluginGrant, owner_payload_root_sha256: &str, local_bundled_payload: Option<&str>) -> bool
```



## bookclerk_plugin_host::remote_consent::remote_request_within

*Function*

True when `requested` asks for nothing outside `reviewed`: the same
plugin, every capability set a subset, network no wider (outbound only
if reviewed outbound), undeclared redirects only if reviewed, and no
larger budget.

```rust
fn remote_request_within(reviewed: &crate::consent::PluginGrant, requested: &crate::consent::PluginGrant) -> bool
```



## bookclerk_plugin_host::remote_consent::remote_row_consent_state

*Function*

Evaluates a stored consent row: [`remote_consent_state`](remote_consent.md#bookclerk_plugin_hostremote_consentremote_consent_state) over its
grants, except that an approval of another archive (the owner changed
the artifact URL or digest since the operator approved) needs approval
again, whatever the grant.

# Arguments

* `row` - The consent row as stored.

```rust
fn remote_row_consent_state(row: &bookclerk_library::control_plane::RemotePluginConsent) -> RemoteConsentState
```



