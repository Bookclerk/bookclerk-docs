**bookclerk_library > control_plane > enrollment**

# Module: control_plane::enrollment

## Contents

**Structs**

- [`EnrolledNode`](#enrollednode) - Result of a successful redeem.
- [`EnrollmentLink`](#enrollmentlink) - One link as listings show it. The secret and its hash are not included.
- [`EnrollmentSecret`](#enrollmentsecret) - A link's plaintext secret. Shown once at mint and never stored; `Debug`
- [`EnrollmentToken`](#enrollmenttoken) - A parsed enrollment token.
- [`HostEnrollmentSettings`](#hostenrollmentsettings) - The host-enrollment switch as stored, or its default when no document
- [`HostEnrollmentSettingsV1`](#hostenrollmentsettingsv1) - Typed `core.host_enrollment` document, schema version 1.
- [`MintedLink`](#mintedlink) - A freshly minted link and its secret, which is returned only here.
- [`NodeCredential`](#nodecredential) - A host node's stored credential.
- [`RedeemRequest`](#redeemrequest) - What a host submits to redeem a link.

**Enums**

- [`EnrollmentActor`](#enrollmentactor) - Who is acting on enrollment links and node credentials.

**Functions**

- [`authorized_enrollment_link`](#authorized_enrollment_link) - The link a branch or clerk sign-in redeems through: its id and secret are
- [`check_redeem_request`](#check_redeem_request) - Checks what a redeem would refuse as malformed before anything is
- [`format_enrollment_token`](#format_enrollment_token) - Formats `bcn1.<link_id>.<secret>[.<spki_pin>]`.
- [`list_enrollment_links`](#list_enrollment_links) - Links `actor` may see, newest first: every link for the operator or an
- [`load_host_enrollment`](#load_host_enrollment) - Reads the cluster's host-enrollment switch. A missing document means the
- [`load_node_credential`](#load_node_credential) - Reads a host node's credential.
- [`mint_enrollment_link`](#mint_enrollment_link) - Mints a link that enrolls one host node for `for_user_id`.
- [`parse_enrollment_token`](#parse_enrollment_token) - Parses a token, or a whole link whose fragment is the token.
- [`record_authorized_enrollment_link`](#record_authorized_enrollment_link) - Records the link an approved branch sign-in redeems through
- [`record_node_connected`](#record_node_connected) - Stamps `last_connected_at` after a host node authenticates. Returns
- [`record_node_left`](#record_node_left) - A host announced it is leaving (`node.leaving`): revokes its credential
- [`redeem_enrollment_link`](#redeem_enrollment_link) - Redeems a link: one canonical batch marks it redeemed, inserts a host
- [`redeem_enrollment_link_as`](#redeem_enrollment_link_as) - [`redeem_enrollment_link`](enrollment.md#bookclerk_librarycontrol_planeenrollmentredeem_enrollment_link) for a node of `kind`: the same canonical
- [`revoke_enrollment_link`](#revoke_enrollment_link) - Revokes an unredeemed link. Returns the link as stored afterwards.
- [`revoke_node_credential`](#revoke_node_credential) - Revokes a host node's credential and tells every node, so the node
- [`set_host_enrollment_enabled`](#set_host_enrollment_enabled) - Turns host enrollment on or off for the cluster.
- [`verify_cluster_owner`](#verify_cluster_owner) - Returns a `ConfigActor::ClusterOwner` when `user_id` is an active Owner.

**Constants**

- [`AUTHORIZED_LINK_CREATED_BY`](#authorized_link_created_by) - Prefix of the `created_by` audit id of a link a sign-in created.
- [`DEFAULT_LINK_TTL`](#default_link_ttl) - Default lifetime of a link.
- [`ENROLLMENT_TOKEN_PREFIX`](#enrollment_token_prefix) - Version tag that starts every enrollment token.
- [`HOST_ENROLLMENT_NAMESPACE`](#host_enrollment_namespace) - Namespace of the cluster's host-enrollment switch.
- [`HOST_ENROLLMENT_SCHEMA_VERSION`](#host_enrollment_schema_version) - Schema version of [`HostEnrollmentSettingsV1`](enrollment.md#bookclerk_librarycontrol_planeenrollmenthostenrollmentsettingsv1).
- [`MAX_LINK_LABEL_BYTES`](#max_link_label_bytes) - Longest `label` stored on a link.
- [`MAX_LINK_TTL`](#max_link_ttl) - Longest lifetime a link may be minted with.
- [`MIN_LINK_TTL`](#min_link_ttl) - Shortest lifetime a link may be minted with.
- [`NODE_PUBLIC_KEY_LEN`](#node_public_key_len) - Size of an Ed25519 or X25519 public key.

---

## bookclerk_library::control_plane::enrollment::AUTHORIZED_LINK_CREATED_BY

*Constant*: `&str`

Prefix of the `created_by` audit id of a link a sign-in created.



## bookclerk_library::control_plane::enrollment::DEFAULT_LINK_TTL

*Constant*: `std::time::Duration`

Default lifetime of a link.



## bookclerk_library::control_plane::enrollment::ENROLLMENT_TOKEN_PREFIX

*Constant*: `&str`

Version tag that starts every enrollment token.



## bookclerk_library::control_plane::enrollment::EnrolledNode

*Struct*

Result of a successful redeem.

**Fields:**
- `cluster_id: String` - Owner cluster id.
- `node_id: String` - Node id minted for the host.
- `owner_user_id: i64` - User who owns the node.
- `replayed: bool` - True when this answered a retry of an earlier successful redeem.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &EnrolledNode) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> EnrolledNode`



## bookclerk_library::control_plane::enrollment::EnrollmentActor

*Enum*

Who is acting on enrollment links and node credentials.

A `User` is resolved from the database on every call: the role and
status the library reads, not what the caller claims, decide what it may
do.

**Variants:**
- `Operator{ id: String }` - Control-plane operator (CLI or operator token).
- `User{ user_id: i64 }` - A signed-in portal user.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &EnrollmentActor) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> EnrollmentActor`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::enrollment::EnrollmentLink

*Struct*

One link as listings show it. The secret and its hash are not included.

**Fields:**
- `link_id: String` - Public link id.
- `for_user_id: i64` - User the enrolled node will belong to.
- `label: String` - Free-form label.
- `created_by: String` - Audit id of the minter.
- `created_at: String` - RFC 3339 mint time.
- `expires_at: String` - RFC 3339 expiry.
- `redeemed_at: Option<String>` - RFC 3339 redeem time.
- `redeemed_node_id: Option<String>` - Node the redeem created.
- `revoked_at: Option<String>` - RFC 3339 revoke time.

**Methods:**

- `fn state(self: &Self, now: DateTime<Utc>) -> &'static str` - `pending`, `redeemed`, `revoked`, or `expired` at `now`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &EnrollmentLink) -> bool`
- **From**
  - `fn from(row: node_enrollment_links::Model) -> Self`
- **Clone**
  - `fn clone(self: &Self) -> EnrollmentLink`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::control_plane::enrollment::EnrollmentSecret

*Struct*

A link's plaintext secret. Shown once at mint and never stored; `Debug`
does not print it.

**Tuple Struct**: `()`

**Methods:**

- `fn from_text(secret: String) -> Self` - Wraps a secret a host submitted.
- `fn expose(self: &Self) -> &str` - The secret text (base64url, no padding).

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &EnrollmentSecret) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> EnrollmentSecret`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_library::control_plane::enrollment::EnrollmentToken

*Struct*

A parsed enrollment token.

**Fields:**
- `link_id: String` - Link id.
- `secret: EnrollmentSecret` - Link secret.
- `spki_pin: Option<String>` - SPKI pin of the owner's self-signed certificate, when present.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EnrollmentToken`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &EnrollmentToken) -> bool`



## bookclerk_library::control_plane::enrollment::HOST_ENROLLMENT_NAMESPACE

*Constant*: `&str`

Namespace of the cluster's host-enrollment switch.



## bookclerk_library::control_plane::enrollment::HOST_ENROLLMENT_SCHEMA_VERSION

*Constant*: `i64`

Schema version of [`HostEnrollmentSettingsV1`](enrollment.md#bookclerk_librarycontrol_planeenrollmenthostenrollmentsettingsv1).



## bookclerk_library::control_plane::enrollment::HostEnrollmentSettings

*Struct*

The host-enrollment switch as stored, or its default when no document
was ever written.

**Fields:**
- `body: HostEnrollmentSettingsV1` - Effective settings.
- `revision: Option<i64>` - Stored revision; `None` while the default applies.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> HostEnrollmentSettings`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &HostEnrollmentSettings) -> bool`



## bookclerk_library::control_plane::enrollment::HostEnrollmentSettingsV1

*Struct*

Typed `core.host_enrollment` document, schema version 1.

**Fields:**
- `enabled: bool` - When false, no link may be minted or redeemed.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> HostEnrollmentSettingsV1`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Self`
- **PartialEq**
  - `fn eq(self: &Self, other: &HostEnrollmentSettingsV1) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::control_plane::enrollment::MAX_LINK_LABEL_BYTES

*Constant*: `usize`

Longest `label` stored on a link.



## bookclerk_library::control_plane::enrollment::MAX_LINK_TTL

*Constant*: `std::time::Duration`

Longest lifetime a link may be minted with.



## bookclerk_library::control_plane::enrollment::MIN_LINK_TTL

*Constant*: `std::time::Duration`

Shortest lifetime a link may be minted with.



## bookclerk_library::control_plane::enrollment::MintedLink

*Struct*

A freshly minted link and its secret, which is returned only here.

**Fields:**
- `link: EnrollmentLink` - Stored link.
- `secret: EnrollmentSecret` - Plaintext secret; put it in the URL fragment and drop it.

**Methods:**

- `fn token(self: &Self, spki_pin: Option<&str>) -> Zeroizing<String>` - The `bcn1.<link_id>.<secret>[.<spki_pin>]` token.
- `fn url(self: &Self, origin: &str, spki_pin: Option<&str>) -> Zeroizing<String>` - `<origin>/enroll#<token>`: the secret stays in the fragment.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> MintedLink`



## bookclerk_library::control_plane::enrollment::NODE_PUBLIC_KEY_LEN

*Constant*: `usize`

Size of an Ed25519 or X25519 public key.



## bookclerk_library::control_plane::enrollment::NodeCredential

*Struct*

A host node's stored credential.

**Fields:**
- `node_id: String` - Host node.
- `sign_public_key: Vec<u8>` - Ed25519 public signing key.
- `box_public_key: Vec<u8>` - X25519 public box key.
- `link_id: String` - Link that enrolled it.
- `enrolled_at: String` - RFC 3339 enrollment time.
- `revoked_at: Option<String>` - RFC 3339 revoke time.
- `last_connected_at: Option<String>` - RFC 3339 time of the last authenticated connection.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NodeCredential`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeCredential) -> bool`
- **From**
  - `fn from(row: node_credentials::Model) -> Self`



## bookclerk_library::control_plane::enrollment::RedeemRequest

*Struct*

What a host submits to redeem a link.

**Fields:**
- `link_id: String` - Link id from the token.
- `secret: EnrollmentSecret` - Link secret from the token.
- `sign_public_key: Vec<u8>` - Ed25519 public signing key (32 bytes).
- `box_public_key: Vec<u8>` - X25519 public box key (32 bytes).
- `software_version: String` - Bookclerk version of the host.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &RedeemRequest) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> RedeemRequest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_library::control_plane::enrollment::authorized_enrollment_link

*Function*

The link a branch or clerk sign-in redeems through: its id and secret are
derived from the authorization code and the PKCE verifier, so a retry of
the same exchange (same code, verifier and keys) finds the same link and
answers from its receipt, and a wrong verifier can never redeem it.

The secret also binds the node kind the code was approved for, so a link
recorded for one kind never redeems as the other, even when the first
redeem failed and left the link unredeemed.

The id is `hex(SHA-256("bookclerk-branch-link-v1\0" || code))[..32]`; the
secret is base64url `SHA-256("bookclerk-branch-secret-v1\0" || code ||
"\0" || verifier || "\0" || kind)`.

```rust
fn authorized_enrollment_link(code: &str, verifier: &str, kind: super::NodeKind) -> (String, EnrollmentSecret)
```



## bookclerk_library::control_plane::enrollment::check_redeem_request

*Function*

Checks what a redeem would refuse as malformed before anything is
written: its keys (canonical Curve25519 points) and `software_version`. A
sign-in's redeem runs this before it spends its code, so a malformed
request leaves the code for a corrected retry.

# Errors

Returns `LibraryError::Invalid` for malformed keys or a version that is
too long or holds control characters.

```rust
fn check_redeem_request(req: &RedeemRequest) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::enrollment::format_enrollment_token

*Function*

Formats `bcn1.<link_id>.<secret>[.<spki_pin>]`.

```rust
fn format_enrollment_token(link_id: &str, secret: &EnrollmentSecret, spki_pin: Option<&str>) -> zeroize::Zeroizing<String>
```



## bookclerk_library::control_plane::enrollment::list_enrollment_links

*Function*

Links `actor` may see, newest first: every link for the operator or an
Owner, the caller's own links for anyone else.

# Errors

Returns an error when the actor is not an active user or the read fails.

```rust
fn list_enrollment_links(store: &crate::store::LibraryStore, actor: &EnrollmentActor) -> crate::error::Result<Vec<EnrollmentLink>>
```



## bookclerk_library::control_plane::enrollment::load_host_enrollment

*Function*

Reads the cluster's host-enrollment switch. A missing document means the
default: enabled.

# Errors

Returns an error when the read fails or the stored document has an
unsupported schema or an invalid body.

```rust
fn load_host_enrollment(store: &crate::store::LibraryStore) -> crate::error::Result<HostEnrollmentSettings>
```



## bookclerk_library::control_plane::enrollment::load_node_credential

*Function*

Reads a host node's credential.

# Errors

Returns an error when the read fails.

```rust
fn load_node_credential(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<Option<NodeCredential>>
```



## bookclerk_library::control_plane::enrollment::mint_enrollment_link

*Function*

Mints a link that enrolls one host node for `for_user_id`.

`ttl` defaults to 24 hours and must be 5 minutes to 7 days. The insert
is predicated on the enrollment switch revision read as enabled, so a
disable that commits while the link is minted wins.

# Errors

Returns `LibraryError::Forbidden` when enrollment is disabled, the
actor is disabled, or a non-Owner mints for someone else;
`LibraryError::NotFound` / `LibraryError::Invalid` when the target
user is unknown or disabled or the ttl or label is out of range; and the
write error when the insert fails.

```rust
fn mint_enrollment_link(store: &crate::store::LibraryStore, actor: &EnrollmentActor, for_user_id: i64, label: &str, ttl: Option<std::time::Duration>) -> crate::error::Result<MintedLink>
```



## bookclerk_library::control_plane::enrollment::parse_enrollment_token

*Function*

Parses a token, or a whole link whose fragment is the token.

The secret is registered for redaction, so logs and diagnostics scrub
it in the process that parsed it.

# Errors

Returns `LibraryError::Invalid` when the text is not a `bcn1` token.

```rust
fn parse_enrollment_token(raw: &str) -> crate::error::Result<EnrollmentToken>
```



## bookclerk_library::control_plane::enrollment::record_authorized_enrollment_link

*Function*

Records the link an approved branch sign-in redeems through
([`authorized_enrollment_link`](enrollment.md#bookclerk_librarycontrol_planeenrollmentauthorized_enrollment_link)), for `for_user_id`, the user who signed
in and consented. Called only after the authorization code's client,
redirect and PKCE verifier were checked, before the code is consumed, so
a failed write leaves the code usable; the redeem itself is
[`redeem_enrollment_link`](enrollment.md#bookclerk_librarycontrol_planeenrollmentredeem_enrollment_link), the same canonical batch a minted link takes.

A second call for the same link id leaves the first row as it is.

# Errors

Returns `LibraryError::Forbidden` while host enrollment is off or the
user is not active, `LibraryError::Invalid` for a malformed label, and
the write error.

```rust
fn record_authorized_enrollment_link(store: &crate::store::LibraryStore, link_id: &str, secret: &EnrollmentSecret, for_user_id: i64, label: &str) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::enrollment::record_node_connected

*Function*

Stamps `last_connected_at` after a host node authenticates. Returns
false (and stamps nothing) when the credential is missing or revoked,
so a revoke that lands mid-handshake still refuses the connection.

# Errors

Returns an error when the update fails.

```rust
fn record_node_connected(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::enrollment::record_node_left

*Function*

A host announced it is leaving (`node.leaving`): revokes its credential
at once and drops its live status, so it stops counting as a node
without waiting for missed heartbeats, and tells every node so the
connection is closed wherever it is held. Answers false when the node
has no unrevoked credential; for one already revoked, the cleanup is
still finished, so a retried leave completes one that failed part way.

# Errors

Returns the write error.

```rust
fn record_node_left(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::enrollment::redeem_enrollment_link

*Function*

Redeems a link: one canonical batch marks it redeemed, inserts a host
`nodes` row owned by the link's user, inserts its `node_credentials`,
and records a receipt.

The batch's predicates re-check the link hash, expiry, single use,
revocation, the user's status, and that the enrollment switch is still
the revision read as enabled, so a concurrent disable or second redeem
fails closed. A retry with the same link, secret, and keys after a lost
reply returns the same node (`replayed`).

# Errors

Returns `LibraryError::Invalid` for malformed keys,
`LibraryError::Forbidden` when enrollment is disabled or the link does
not redeem (one message for every reason), and the write error when the
batch fails.

```rust
fn redeem_enrollment_link(store: &crate::store::LibraryStore, req: &RedeemRequest) -> crate::error::Result<EnrolledNode>
```



## bookclerk_library::control_plane::enrollment::redeem_enrollment_link_as

*Function*

[`redeem_enrollment_link`](enrollment.md#bookclerk_librarycontrol_planeenrollmentredeem_enrollment_link) for a node of `kind`: the same canonical
batch, recording `kind` on the `nodes` row.

A `super::NodeKind::Owner` node (a clerk, which joined with a sign-in
an Owner approved as such) shares the library's database, so it gets no
`node_credentials` row: it never dials the node socket. The kind is part
of the request hash, so a retry of a branch redeem as a clerk (or the
other way round) is refused as a spent link.

# Errors

As [`redeem_enrollment_link`](enrollment.md#bookclerk_librarycontrol_planeenrollmentredeem_enrollment_link).

```rust
fn redeem_enrollment_link_as(store: &crate::store::LibraryStore, req: &RedeemRequest, kind: super::NodeKind) -> crate::error::Result<EnrolledNode>
```



## bookclerk_library::control_plane::enrollment::revoke_enrollment_link

*Function*

Revokes an unredeemed link. Returns the link as stored afterwards.

# Errors

Returns `LibraryError::NotFound` when the link is unknown or not the
caller's (the two read the same), `LibraryError::Conflict` when it was
already redeemed, and the write error when the update fails.

```rust
fn revoke_enrollment_link(store: &crate::store::LibraryStore, actor: &EnrollmentActor, link_id: &str) -> crate::error::Result<EnrollmentLink>
```



## bookclerk_library::control_plane::enrollment::revoke_node_credential

*Function*

Revokes a host node's credential and tells every node, so the node
holding that host's connection can close it.

The operator and an Owner may revoke any host node; any other active
user may revoke the nodes they own. Revoking twice keeps the first time.

# Errors

Returns `LibraryError::NotFound` when the node is unknown or not the
caller's (the two read the same), `LibraryError::Invalid` when the
node has no host credential (an owner node), and the write error when
the update fails.

```rust
fn revoke_node_credential(store: &crate::store::LibraryStore, actor: &EnrollmentActor, node_id: &str) -> crate::error::Result<NodeCredential>
```



## bookclerk_library::control_plane::enrollment::set_host_enrollment_enabled

*Function*

Turns host enrollment on or off for the cluster.

Only the operator or a verified Owner ([`verify_cluster_owner`](enrollment.md#bookclerk_librarycontrol_planeenrollmentverify_cluster_owner)) may write
it. Retries a few times on a concurrent edit.

# Errors

Returns `LibraryError::Forbidden` for any other actor, and the write
error when the document cannot be stored.

```rust
fn set_host_enrollment_enabled(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, enabled: bool) -> crate::error::Result<super::ConfigurationDocument<HostEnrollmentSettingsV1>>
```



## bookclerk_library::control_plane::enrollment::verify_cluster_owner

*Function*

Returns a `ConfigActor::ClusterOwner` when `user_id` is an active Owner.

# Errors

Returns `LibraryError::Forbidden` for anyone else (including an unknown
user), and the read error when the lookup fails.

```rust
fn verify_cluster_owner(store: &crate::store::LibraryStore, user_id: i64) -> crate::error::Result<super::documents::ConfigActor>
```



