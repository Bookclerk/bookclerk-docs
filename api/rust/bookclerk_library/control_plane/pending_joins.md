**bookclerk_library > control_plane > pending_joins**

# Module: control_plane::pending_joins

## Contents

**Structs**

- [`NewPendingJoin`](#newpendingjoin) - What a joining daemon records when it starts a sign-in.
- [`PendingJoin`](#pendingjoin) - A pending join as stored (no secrets).
- [`TakenJoin`](#takenjoin) - A consumed pending join with its secrets. `Debug` prints no secret.

**Functions**

- [`create_pending_join`](#create_pending_join) - Records a sign-in in progress: seals `verifier` and `keys` and writes the
- [`finish_pending_join`](#finish_pending_join) - Ends the pending join for `state`: deletes its row and sealed material.
- [`open_pending_join`](#open_pending_join) - Reads the pending join for `state` without consuming it: its facts and
- [`purge_expired_pending_joins`](#purge_expired_pending_joins) - Deletes every expired pending join and its sealed material. Answers how
- [`take_pending_join`](#take_pending_join) - Consumes the pending join for `state`: answers its facts and secrets once

**Constants**

- [`MAX_JOIN_NAME_BYTES`](#max_join_name_bytes) - Longest branch or library name stored.
- [`PENDING_JOIN_TTL`](#pending_join_ttl) - How long a sign-in may take before its pending row is refused.

---

## bookclerk_library::control_plane::pending_joins::MAX_JOIN_NAME_BYTES

*Constant*: `usize`

Longest branch or library name stored.



## bookclerk_library::control_plane::pending_joins::NewPendingJoin

*Struct*

What a joining daemon records when it starts a sign-in.

**Fields:**
- `owner_origin: String` - Owner origin the sign-in goes to.
- `spki_pin: Option<String>` - SPKI pin the operator confirmed for a self-signed owner.
- `owner_cluster_id: String` - Owner cluster id `GET /api/node/v1/info` reported.
- `library_name: String` - Library name the owner reported.
- `branch_name: String` - Name this install joins as.
- `redirect_uri: String` - Redirect URI of the authorization request.
- `allow_clerk: bool` - True when the operator offered to join as a clerk (the install was

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NewPendingJoin) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NewPendingJoin`



## bookclerk_library::control_plane::pending_joins::PENDING_JOIN_TTL

*Constant*: `std::time::Duration`

How long a sign-in may take before its pending row is refused.



## bookclerk_library::control_plane::pending_joins::PendingJoin

*Struct*

A pending join as stored (no secrets).

**Fields:**
- `owner_origin: String` - Owner origin.
- `spki_pin: Option<String>` - Confirmed SPKI pin.
- `owner_cluster_id: String` - Owner cluster id.
- `library_name: String` - Library name.
- `branch_name: String` - Branch name.
- `redirect_uri: String` - Redirect URI.
- `allow_clerk: bool` - True when the operator offered to join as a clerk.
- `expires_at: String` - RFC 3339 expiry.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PendingJoin) -> bool`
- **From**
  - `fn from(row: pending_joins::Model) -> Self`
- **Clone**
  - `fn clone(self: &Self) -> PendingJoin`



## bookclerk_library::control_plane::pending_joins::TakenJoin

*Struct*

A consumed pending join with its secrets. `Debug` prints no secret.

**Fields:**
- `join: PendingJoin` - The stored facts.
- `verifier: zeroize::Zeroizing<String>` - The PKCE verifier.
- `keys: super::cluster_links::ClusterLinkKeys` - The node keys generated for the join.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_library::control_plane::pending_joins::create_pending_join

*Function*

Records a sign-in in progress: seals `verifier` and `keys` and writes the
row under `hash_token(state)`. Expired rows are purged first.

# Errors

Returns `LibraryError::Invalid` for a malformed origin, pin, name or
verifier, an error when no master key is configured, and the write
error.

```rust
fn create_pending_join(store: &crate::store::LibraryStore, join: &NewPendingJoin, state: &str, verifier: &str, keys: &super::cluster_links::ClusterLinkKeys) -> crate::error::Result<PendingJoin>
```



## bookclerk_library::control_plane::pending_joins::finish_pending_join

*Function*

Ends the pending join for `state`: deletes its row and sealed material.

# Returns

Whether this call removed the row (`false` when another call already
did, or there was none).

# Errors

Returns the write error.

```rust
fn finish_pending_join(store: &crate::store::LibraryStore, state: &str) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::pending_joins::open_pending_join

*Function*

Reads the pending join for `state` without consuming it: its facts and
unsealed secrets, so a redeem that fails for a reason a retry can fix
(the library unreachable, or its answer lost) can be tried again with
the same keys. `None` for an unknown, finished or expired state.

The caller serializes attempts for one state and calls
[`finish_pending_join`](pending_joins.md#bookclerk_librarycontrol_planepending_joinsfinish_pending_join) once the outcome is final.

# Errors

Returns the read error, or `LibraryError::Invalid` when the sealed
material is malformed.

```rust
fn open_pending_join(store: &crate::store::LibraryStore, state: &str) -> crate::error::Result<Option<TakenJoin>>
```



## bookclerk_library::control_plane::pending_joins::purge_expired_pending_joins

*Function*

Deletes every expired pending join and its sealed material. Answers how
many rows went.

# Errors

Returns the read or write error.

```rust
fn purge_expired_pending_joins(store: &crate::store::LibraryStore) -> crate::error::Result<u64>
```



## bookclerk_library::control_plane::pending_joins::take_pending_join

*Function*

Consumes the pending join for `state`: answers its facts and secrets once
and deletes both rows. `None` for an unknown, already consumed, or
expired state.

# Errors

Returns the read or write error, or `LibraryError::Invalid` when the
sealed material is malformed.

```rust
fn take_pending_join(store: &crate::store::LibraryStore, state: &str) -> crate::error::Result<Option<TakenJoin>>
```



