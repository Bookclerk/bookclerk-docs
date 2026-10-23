**bookclerk_library > control_plane > remote_consents**

# Module: control_plane::remote_consents

## Contents

**Structs**

- [`RemoteApproval`](#remoteapproval) - What an operator approves.
- [`RemoteArtifactRef`](#remoteartifactref) - The archive an owner asks this host to fetch: the version's artifact for
- [`RemotePluginConsent`](#remotepluginconsent) - One owner plugin of one owner cluster, as this install sees it.

**Enums**

- [`RemoteApprover`](#remoteapprover) - Who gives an approval.

**Functions**

- [`approve_remote_plugin`](#approve_remote_plugin) - Stores an approval, but only when the recorded request is still the
- [`approved_remote_payload_digests`](#approved_remote_payload_digests) - Payload-root digests of every owner tree an operator approval still
- [`list_remote_plugin_consents`](#list_remote_plugin_consents) - Every consent row, of one cluster when `cluster_id` is given, by
- [`load_remote_plugin_consent`](#load_remote_plugin_consent) - The consent row for `plugin_key` of `cluster_id`.
- [`record_remote_plugin_request`](#record_remote_plugin_request) - Records what the owner asks to run here: `version` of `plugin_key` with
- [`remote_request_digest`](#remote_request_digest) - The digest an operator approves a request by: SHA-256 (lowercase hex)
- [`revoke_remote_plugin`](#revoke_remote_plugin) - Clears the approval of `plugin_key` of `cluster_id`, keeping its

**Constants**

- [`REMOTE_CONSENT_PLATFORM`](#remote_consent_platform) - `approved_by` of an automatic approval: a bundled platform plugin whose

---

## bookclerk_library::control_plane::remote_consents::REMOTE_CONSENT_PLATFORM

*Constant*: `&str`

`approved_by` of an automatic approval: a bundled platform plugin whose
request stays inside the platform envelope.



## bookclerk_library::control_plane::remote_consents::RemoteApproval

*Struct*

What an operator approves.

**Generic Parameters:**
- 'a

**Fields:**
- `cluster_id: &'a str` - Owner cluster id.
- `plugin_key: &'a str` - Canonical plugin key.
- `version: &'a str` - The requested version the operator reviewed.
- `reviewed_request_json: &'a str` - The owner's grant the operator reviewed, exactly as recorded.
- `reviewed_artifact: RemoteArtifactRef<'a>` - The archive the operator reviewed, exactly as recorded.
- `grant_json: &'a str` - The grant consented to (already checked to be no wider than the
- `approver: RemoteApprover<'a>` - Who approves.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RemoteApproval<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteApproval<'a>) -> bool`



## bookclerk_library::control_plane::remote_consents::RemoteApprover

*Enum*

Who gives an approval.

**Generic Parameters:**
- 'a

**Variants:**
- `Operator(&'a str)` - This install's operator, by audit identity (never
- `Platform` - The automatic approval of a bundled platform plugin. Refused while

**Methods:**

- `fn audit_identity(self: &Self) -> &str` - The audit identity stored as `approved_by`.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteApprover<'a>) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> RemoteApprover<'a>`



## bookclerk_library::control_plane::remote_consents::RemoteArtifactRef

*Struct*

The archive an owner asks this host to fetch: the version's artifact for
this host's target.

**Generic Parameters:**
- 'a

**Fields:**
- `url: &'a str` - Where the archive is fetched from (`blob:sha256:` over the owner
- `archive_sha256: &'a str` - SHA-256 (lowercase hex) of the archive.
- `payload_root_sha256: &'a str` - Payload-root digest of the tree the archive unpacks to: the key of

**Methods:**

- `fn of(artifact: &'a PluginArtifact) -> Self` - The consent view of a version artifact.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RemoteArtifactRef<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteArtifactRef<'a>) -> bool`



## bookclerk_library::control_plane::remote_consents::RemotePluginConsent

*Struct*

One owner plugin of one owner cluster, as this install sees it.

**Fields:**
- `owner_cluster_id: String` - Owner cluster id.
- `plugin_key: String` - Canonical plugin key.
- `requested_version: String` - Version the owner deploys here now.
- `requested_grant_json: String` - The owner's approved grant for that version (JSON).
- `requested_artifact_url: String` - URL of the archive the owner deploys to this host.
- `requested_artifact_sha256: String` - SHA-256 of that archive.
- `requested_artifact_payload_sha256: String` - Payload-root digest of the tree that archive unpacks to.
- `version: Option<String>` - Version this install's operator approved.
- `approved_request_json: Option<String>` - The owner's grant when the operator approved (JSON).
- `approved_artifact_url: Option<String>` - The archive URL when the operator approved.
- `approved_artifact_sha256: Option<String>` - The archive SHA-256 when the operator approved.
- `approved_artifact_payload_sha256: Option<String>` - The payload-root digest when the operator approved.
- `grant_json: Option<String>` - The grant the operator consented to (JSON).
- `approved_at: Option<String>` - RFC 3339 approval time.
- `approved_by: Option<String>` - Who approved ([`REMOTE_CONSENT_PLATFORM`](remote_consents.md#bookclerk_librarycontrol_planeremote_consentsremote_consent_platform) for an automatic approval).
- `revoked_at: Option<String>` - RFC 3339 time the operator revoked the approval; cleared only by an
- `updated_at: String` - RFC 3339 time of the last write.

**Methods:**

- `fn is_approved(self: &Self) -> bool` - True when an approval is stored. Whether it still covers the
- `fn is_revoked(self: &Self) -> bool` - True when the operator revoked the approval and has not approved
- `fn request_digest(self: &Self) -> String` - The digest of the current request ([`remote_request_digest`](remote_consents.md#bookclerk_librarycontrol_planeremote_consentsremote_request_digest)); an
- `fn requested_artifact(self: &Self) -> RemoteArtifactRef` - The archive the current request asks this host to fetch.
- `fn approved_artifact_matches(self: &Self, artifact: RemoteArtifactRef) -> bool` - True when an approval is stored for exactly `artifact` (URL, archive

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemotePluginConsent) -> bool`
- **From**
  - `fn from(row: remote_plugin_consents::Model) -> Self`
- **Clone**
  - `fn clone(self: &Self) -> RemotePluginConsent`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::control_plane::remote_consents::approve_remote_plugin

*Function*

Stores an approval, but only when the recorded request is still the
one the operator reviewed: a request that changed in between (a new
version, a wider grant, another archive) answers `LibraryError::Conflict` and nothing
is written. An automatic approval (`RemoteApprover::Platform`) is
likewise refused while a revoke is recorded (an operator approval
clears it) and over an operator's approval, which it would replace
along with its narrowing.

# Errors

Returns `LibraryError::NotFound` when no request is recorded,
`LibraryError::Conflict` when it changed or an automatic approval
meets a revoke or an operator's approval, `LibraryError::Invalid` for a malformed field (an
operator identity of [`REMOTE_CONSENT_PLATFORM`](remote_consents.md#bookclerk_librarycontrol_planeremote_consentsremote_consent_platform) among them), and the
write error.

```rust
fn approve_remote_plugin(store: &crate::store::LibraryStore, approval: &RemoteApproval) -> crate::error::Result<RemotePluginConsent>
```



## bookclerk_library::control_plane::remote_consents::approved_remote_payload_digests

*Function*

Payload-root digests of every owner tree an operator approval still
covers, which a sweep of the shared version cache must keep: owner
versions have no local deployment row to reference them.

# Errors

Returns the read error.

```rust
fn approved_remote_payload_digests(store: &crate::store::LibraryStore) -> crate::error::Result<std::collections::BTreeSet<String>>
```



## bookclerk_library::control_plane::remote_consents::list_remote_plugin_consents

*Function*

Every consent row, of one cluster when `cluster_id` is given, by
cluster then plugin key.

# Errors

Returns an error when the read fails.

```rust
fn list_remote_plugin_consents(store: &crate::store::LibraryStore, cluster_id: Option<&str>) -> crate::error::Result<Vec<RemotePluginConsent>>
```



## bookclerk_library::control_plane::remote_consents::load_remote_plugin_consent

*Function*

The consent row for `plugin_key` of `cluster_id`.

# Errors

Returns an error when the read fails.

```rust
fn load_remote_plugin_consent(store: &crate::store::LibraryStore, cluster_id: &str, plugin_key: &str) -> crate::error::Result<Option<RemotePluginConsent>>
```



## bookclerk_library::control_plane::remote_consents::record_remote_plugin_request

*Function*

Records what the owner asks to run here: `version` of `plugin_key` with
the owner's approved grant, fetched from `artifact`. A stored approval
is kept; whether it still covers the new request is decided when the
grant is evaluated (and an approval of another archive never does,
`RemotePluginConsent::approved_artifact_matches`). Writes nothing
when the request is unchanged.

# Errors

Returns `LibraryError::Invalid` for an empty or oversized field or a
grant that is not a JSON object, and the write error.

```rust
fn record_remote_plugin_request(store: &crate::store::LibraryStore, cluster_id: &str, plugin_key: &str, version: &str, requested_grant_json: &str, artifact: RemoteArtifactRef) -> crate::error::Result<RemotePluginConsent>
```



## bookclerk_library::control_plane::remote_consents::remote_request_digest

*Function*

The digest an operator approves a request by: SHA-256 (lowercase hex)
over the requested version, the owner's grant exactly as recorded and
the archive's URL, digest and payload-root digest, each length-prefixed,
so any change to any of them gives another digest.

# Arguments

* `version` - The requested version.
* `requested_grant_json` - The owner's grant as recorded.
* `artifact` - The archive this host would fetch.

# Returns

64 lowercase hex characters.

```rust
fn remote_request_digest(version: &str, requested_grant_json: &str, artifact: RemoteArtifactRef) -> String
```



## bookclerk_library::control_plane::remote_consents::revoke_remote_plugin

*Function*

Clears the approval of `plugin_key` of `cluster_id`, keeping its
request, and records the revoke so no automatic approval brings it
back. Answers the row, or `None` when nothing is recorded.

# Errors

Returns an error when a read or the write fails.

```rust
fn revoke_remote_plugin(store: &crate::store::LibraryStore, cluster_id: &str, plugin_key: &str) -> crate::error::Result<Option<RemotePluginConsent>>
```



