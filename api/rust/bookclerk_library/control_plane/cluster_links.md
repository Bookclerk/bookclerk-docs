**bookclerk_library > control_plane > cluster_links**

# Module: control_plane::cluster_links

## Contents

**Structs**

- [`ClusterLink`](#clusterlink) - One owner cluster this install enrolled into.
- [`ClusterLinkKeys`](#clusterlinkkeys) - The node keys of one cluster link: the Ed25519 signing seed and the
- [`NewClusterLink`](#newclusterlink) - What enrollment returned, to record with [`record_cluster_link`](cluster_links.md#bookclerk_librarycontrol_planecluster_linksrecord_cluster_link).

**Functions**

- [`decode_spki_pin`](#decode_spki_pin) - Decodes an SPKI pin: base64url (no padding) of a SHA-256 digest.
- [`join_cycle`](#join_cycle) - This library's own cluster id when `parent_ancestry` names it: joining
- [`join_overflows_ancestry`](#join_overflows_ancestry) - True when joining a library that reports `parent_ancestry` would give
- [`leave_cluster_link`](#leave_cluster_link) - Leaves an owner cluster on this side: marks the link
- [`library_ancestry`](#library_ancestry) - This library's ancestry (what `GET /api/node/v1/info` reports): its own
- [`list_cluster_links`](#list_cluster_links) - Every cluster link, oldest enrollment first.
- [`load_cluster_link`](#load_cluster_link) - One cluster link.
- [`load_cluster_link_keys`](#load_cluster_link_keys) - Unseals the node keys of an active `link`.
- [`mark_cluster_link_revoked`](#mark_cluster_link_revoked) - Marks an **active** link still enrolled as `remote_node_id`
- [`record_cluster_link`](#record_cluster_link) - Seals `keys` and records the link, replacing an earlier link to the same
- [`set_cluster_link_ancestry`](#set_cluster_link_ancestry) - Records the owner library's ancestry as it reported it (`info` at join,
- [`set_cluster_link_sharing`](#set_cluster_link_sharing) - Sets a link's two sharing settings: "Share this branch's books with
- [`set_cluster_link_state`](#set_cluster_link_state) - Sets a link's state. Answers false when no such link exists.

**Constants**

- [`CLUSTER_LINK_ACTIVE`](#cluster_link_active) - `cluster_links.state` of a link the host client connects.
- [`CLUSTER_LINK_KEYS_NAME`](#cluster_link_keys_name) - `encrypted_secrets.name` of a link's node keys.
- [`CLUSTER_LINK_LEFT`](#cluster_link_left) - `cluster_links.state` after this install left the cluster.
- [`CLUSTER_LINK_REVOKED`](#cluster_link_revoked) - `cluster_links.state` once the owner revoked this node (close 4403).

---

## bookclerk_library::control_plane::cluster_links::CLUSTER_LINK_ACTIVE

*Constant*: `&str`

`cluster_links.state` of a link the host client connects.



## bookclerk_library::control_plane::cluster_links::CLUSTER_LINK_KEYS_NAME

*Constant*: `&str`

`encrypted_secrets.name` of a link's node keys.



## bookclerk_library::control_plane::cluster_links::CLUSTER_LINK_LEFT

*Constant*: `&str`

`cluster_links.state` after this install left the cluster.



## bookclerk_library::control_plane::cluster_links::CLUSTER_LINK_REVOKED

*Constant*: `&str`

`cluster_links.state` once the owner revoked this node (close 4403).



## bookclerk_library::control_plane::cluster_links::ClusterLink

*Struct*

One owner cluster this install enrolled into.

**Fields:**
- `owner_cluster_id: String` - Owner cluster id.
- `owner_origin: String` - Owner origin as [`bookclerk_node_proto::canonical_origin`](../../bookclerk_node_proto/signature.md#bookclerk_node_protosignaturecanonical_origin) spells it.
- `remote_node_id: String` - Node id the owner minted for this install.
- `spki_pin: Option<String>` - SPKI pin of the owner's self-signed certificate, when the link
- `state: String` - [`CLUSTER_LINK_ACTIVE`](cluster_links.md#bookclerk_librarycontrol_planecluster_linkscluster_link_active), [`CLUSTER_LINK_REVOKED`](cluster_links.md#bookclerk_librarycontrol_planecluster_linkscluster_link_revoked), or
- `enrolled_at: String` - RFC 3339 enrollment time.
- `share_books: bool` - "Share this branch's books with *library*" (on by default).
- `include_branches: bool` - "Include books from my own branches" (on by default; off shares
- `ancestry: Vec<String>` - The owner library's ancestry as it last reported it: its own

**Methods:**

- `fn is_active(self: &Self) -> bool` - True when the host client should connect this link.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ClusterLink) -> bool`
- **From**
  - `fn from(row: cluster_links::Model) -> Self`
- **Clone**
  - `fn clone(self: &Self) -> ClusterLink`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::control_plane::cluster_links::ClusterLinkKeys

*Struct*

The node keys of one cluster link: the Ed25519 signing seed and the
X25519 box secret. Zeroized on drop; `Debug` shows only the public keys.

**Methods:**

- `fn generate() -> Self` - Fresh random keys from the OS RNG.
- `fn from_bytes(sign_seed: &[u8; 32], box_secret: &[u8; 32]) -> Self` - Keys from known bytes (tests, and reading the sealed row back).
- `fn sign_seed(self: &Self) -> &[u8; 32]` - The Ed25519 signing seed.
- `fn box_secret(self: &Self) -> &[u8; 32]` - The X25519 box secret.
- `fn sign_public_key(self: &Self) -> [u8; 32]` - The Ed25519 public key the owner stores.
- `fn box_public_key(self: &Self) -> [u8; 32]` - The X25519 public key the owner seals job secrets to.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> ClusterLinkKeys`



## bookclerk_library::control_plane::cluster_links::NewClusterLink

*Struct*

What enrollment returned, to record with [`record_cluster_link`](cluster_links.md#bookclerk_librarycontrol_planecluster_linksrecord_cluster_link).

**Fields:**
- `owner_cluster_id: String` - Owner cluster id from the enroll answer.
- `owner_origin: String` - Owner origin the host enrolled through.
- `remote_node_id: String` - Node id from the enroll answer.
- `spki_pin: Option<String>` - SPKI pin from the enrollment link.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NewClusterLink`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NewClusterLink) -> bool`



## bookclerk_library::control_plane::cluster_links::decode_spki_pin

*Function*

Decodes an SPKI pin: base64url (no padding) of a SHA-256 digest.

# Errors

Returns `LibraryError::Invalid` for anything else.

```rust
fn decode_spki_pin(pin: &str) -> crate::error::Result<[u8; 32]>
```



## bookclerk_library::control_plane::cluster_links::join_cycle

*Function*

This library's own cluster id when `parent_ancestry` names it: joining
that library would make a cycle (A ← B ← A, A ← B ← C ← A). `None` when
it does not, or this install has no cluster identity yet.

# Errors

Returns the read error.

```rust
fn join_cycle(store: &crate::store::LibraryStore, parent_ancestry: &[String]) -> crate::error::Result<Option<String>>
```



## bookclerk_library::control_plane::cluster_links::join_overflows_ancestry

*Function*

True when joining a library that reports `parent_ancestry` would give
this library more ancestors (itself and every library above it) than
[`bookclerk_node_proto::MAX_ANCESTRY`]: [`library_ancestry`](cluster_links.md#bookclerk_librarycontrol_planecluster_linkslibrary_ancestry) would then
leave the farthest out, and a library joining this one later could no
longer be checked for a loop through them. An ancestry cut at that
limit by the library itself always holds exactly the limit, so this
refuses it too. Such a join is refused rather than cut short.

# Errors

Returns the read error.

```rust
fn join_overflows_ancestry(store: &crate::store::LibraryStore, parent_ancestry: &[String]) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::cluster_links::leave_cluster_link

*Function*

Leaves an owner cluster on this side: marks the link
[`CLUSTER_LINK_LEFT`](cluster_links.md#bookclerk_librarycontrol_planecluster_linkscluster_link_left) and deletes its sealed node keys, so this install
can never sign as that node again. Answers the link as it now stands,
or `None` when there is no such link.

This only records the departure. Telling the owner (`node.leaving`,
which revokes the node there) is the caller's, before the keys go:
`bookclerk_node_client::announce_leaving` or the running client. A
running daemon stops the link's client when it next re-reads the link.
Joining again needs a new sign-in or invite.

Only one enrollment is left: `expected_node_id` when given (the node the
caller told the owner about), else the one read here. A link a join
enrolled as another node, before this call or while it runs, is refused,
so a new enrollment is never marked left with its keys still stored or
left without the owner having been told.

# Arguments

* `store` - Library store holding the link.
* `owner_cluster_id` - Owner cluster the link joins.
* `expected_node_id` - The node the caller means to leave, when it read
  the link earlier (to send `node.leaving`); `None` leaves whatever node
  the link names now.

# Errors

Returns `LibraryError::Invalid` when the link was re-enrolled as
another node, and the read or write error.

```rust
fn leave_cluster_link(store: &crate::store::LibraryStore, owner_cluster_id: &str, expected_node_id: Option<&str>) -> crate::error::Result<Option<ClusterLink>>
```



## bookclerk_library::control_plane::cluster_links::library_ancestry

*Function*

This library's ancestry (what `GET /api/node/v1/info` reports): its own
cluster id, then for each active link the owner's cluster id and the
ancestry that owner reported, without repeats, at most
[`bookclerk_node_proto::MAX_ANCESTRY`](../../bookclerk_node_proto/catalog.md#bookclerk_node_protocatalogmax_ancestry) ids. Empty when this install has
no cluster identity yet. A join that would take it past that limit is
refused ([`join_overflows_ancestry`](cluster_links.md#bookclerk_librarycontrol_planecluster_linksjoin_overflows_ancestry)); should a library above grow its
ancestry later, the list is cut at exactly the limit, which in turn
refuses every join under it.

A clerk shares its library's database, so it reports the same ancestry:
it is the same node of the tree.

# Errors

Returns the read error.

```rust
fn library_ancestry(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<String>>
```



## bookclerk_library::control_plane::cluster_links::list_cluster_links

*Function*

Every cluster link, oldest enrollment first.

# Errors

Returns the read error.

```rust
fn list_cluster_links(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<ClusterLink>>
```



## bookclerk_library::control_plane::cluster_links::load_cluster_link

*Function*

One cluster link.

# Errors

Returns the read error.

```rust
fn load_cluster_link(store: &crate::store::LibraryStore, owner_cluster_id: &str) -> crate::error::Result<Option<ClusterLink>>
```



## bookclerk_library::control_plane::cluster_links::load_cluster_link_keys

*Function*

Unseals the node keys of an active `link`.

# Errors

Returns `LibraryError::Forbidden` when the link is not active (a
revoked or departed node never signs again),
`LibraryError::NotFound` when the key row is missing, and
`LibraryError::Invalid` when it does not hold two 32-byte keys.

```rust
fn load_cluster_link_keys(store: &crate::store::LibraryStore, link: &ClusterLink) -> crate::error::Result<ClusterLinkKeys>
```



## bookclerk_library::control_plane::cluster_links::mark_cluster_link_revoked

*Function*

Marks an **active** link still enrolled as `remote_node_id`
[`CLUSTER_LINK_REVOKED`](cluster_links.md#bookclerk_librarycontrol_planecluster_linkscluster_link_revoked) (the owner closed with 4403 on that node's
connection). A link this install already left stays `left`, and one a
later join enrolled as another node keeps its new enrollment.

# Arguments

* `store` - Library store holding the link.
* `owner_cluster_id` - Owner cluster the link joins.
* `remote_node_id` - The node the revoked connection ran as.

# Returns

Whether a row changed: `false` when the link is gone, no longer active,
or enrolled as another node.

# Errors

Returns the write error.

```rust
fn mark_cluster_link_revoked(store: &crate::store::LibraryStore, owner_cluster_id: &str, remote_node_id: &str) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::cluster_links::record_cluster_link

*Function*

Seals `keys` and records the link, replacing an earlier link to the same
cluster (a re-enrollment) as `active`.

The key row is written first: a link row never exists without keys, and
a key row left by a failed write is replaced by the next enrollment.
When the re-enrollment minted a different node id, the earlier node's
key row is deleted once the new link row is stored, so the sealed
private keys of a node the owner no longer knows are not kept.

# Errors

Returns `LibraryError::Invalid` for a malformed id, origin, or pin, an
error when no master key is configured, and the write error.

```rust
fn record_cluster_link(store: &crate::store::LibraryStore, link: &NewClusterLink, keys: &ClusterLinkKeys) -> crate::error::Result<ClusterLink>
```



## bookclerk_library::control_plane::cluster_links::set_cluster_link_ancestry

*Function*

Records the owner library's ancestry as it reported it (`info` at join,
`catalog.session` on every sync). Ids that are not plain cluster ids are
refused; at most [`bookclerk_node_proto::MAX_ANCESTRY`](../../bookclerk_node_proto/catalog.md#bookclerk_node_protocatalogmax_ancestry) are kept.

# Errors

Returns `LibraryError::Invalid` for a malformed ancestry, and the
write error.

```rust
fn set_cluster_link_ancestry(store: &crate::store::LibraryStore, owner_cluster_id: &str, ancestry: &[String]) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::cluster_links::set_cluster_link_sharing

*Function*

Sets a link's two sharing settings: "Share this branch's books with
*library*" and "Include books from my own branches". Answers false when
no such link exists. The running catalog sync reports them to the owner
on its next pass.

# Errors

Returns the write error.

```rust
fn set_cluster_link_sharing(store: &crate::store::LibraryStore, owner_cluster_id: &str, share_books: bool, include_branches: bool) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::cluster_links::set_cluster_link_state

*Function*

Sets a link's state. Answers false when no such link exists.

# Arguments

* `store` - Library store holding the link.
* `owner_cluster_id` - Owner cluster the link joins.
* `state` - One of [`CLUSTER_LINK_ACTIVE`](cluster_links.md#bookclerk_librarycontrol_planecluster_linkscluster_link_active), [`CLUSTER_LINK_REVOKED`](cluster_links.md#bookclerk_librarycontrol_planecluster_linkscluster_link_revoked) or
  [`CLUSTER_LINK_LEFT`](cluster_links.md#bookclerk_librarycontrol_planecluster_linkscluster_link_left).
* `remote_node_id` - When set, only a link still enrolled as this node is
  changed, so a client running under an earlier enrollment cannot touch the
  link that replaced it. `None` changes the link whatever its node.

# Errors

Returns `LibraryError::Invalid` for an unknown state, and the write
error.

```rust
fn set_cluster_link_state(store: &crate::store::LibraryStore, owner_cluster_id: &str, state: &str, remote_node_id: Option<&str>) -> crate::error::Result<bool>
```



