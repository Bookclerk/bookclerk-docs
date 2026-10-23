**bookclerk_node_client > enroll**

# Module: enroll

## Contents

**Enums**

- [`JoinError`](#joinerror) - Why joining failed.

**Functions**

- [`check_join_ancestry`](#check_join_ancestry) - Reads the library's `info` over `endpoint` and refuses a library whose
- [`join_cluster`](#join_cluster) - Enrolls this install into the owner cluster named by `link`.
- [`refuse_cycle`](#refuse_cycle) - Refuses `ancestry` when it names this install's own cluster id, is

---

## bookclerk_node_client::enroll::JoinError

*Enum*

Why joining failed.

**Variants:**
- `Link(String)` - The text is not a whole `https://<owner>/enroll#bcn1...` link.
- `Dial(crate::endpoint::DialError)` - The owner could not be reached under the transport policy.
- `Refused` - The owner refused the link (spent, expired, revoked, or unknown), or
- `OriginMismatch{ origin: String }` - The library names another origin as its own; join at that one.
- `Http{ status: u16, error: String }` - The owner answered with another status.
- `Record(String)` - The enroll request could not be built, or the library's `info`
- `Cycle` - The library is this one, or one of its own branches (its ancestry
- `AncestryTooDeep` - Joining the library would give this one more ancestors than an
- `Unsaved{ node_id: Option<String>, reason: String }` - The library accepted the join (an invite link or a sign-in code),
- `SaveFailed{ node_id: String, reason: String }` - A sign-in's code was redeemed and the library enrolled this host,

**Methods:**

- `fn is_retryable(self: &Self) -> bool` - True for a sign-in redeem failure that a repeat of the same exchange

**Trait Implementations:**

- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **From**
  - `fn from(source: DialError) -> Self`



## bookclerk_node_client::enroll::check_join_ancestry

*Function*

Reads the library's `info` over `endpoint` and refuses a library whose
ancestry names this install's own cluster id (A ← B ← A). Answers its
ancestry, to record on the link.

# Errors

Returns `JoinError::Cycle` for a cycle, and [`JoinError`](enroll.md#bookclerk_node_clientenrolljoinerror) when the
library cannot be read (fail closed: no join without the check).

```rust
fn check_join_ancestry(store: &bookclerk_library::LibraryStore, endpoint: &crate::endpoint::OwnerEndpoint) -> Result<Vec<String>, JoinError>
```



## bookclerk_node_client::enroll::join_cluster

*Function*

Enrolls this install into the owner cluster named by `link`.

Generates fresh node keys, redeems the link at the owner (under the
transport policy and the link's SPKI pin), then seals the keys into this
install's `encrypted_secrets` and records the `cluster_links` row. The
link secret is sent once in the request body and never logged.

`link` is the whole link as minted (`https://owner/enroll#bcn1...`):
the origin it names is the one this host dials and signs from then on.

# Errors

Returns [`JoinError`](enroll.md#bookclerk_node_clientenrolljoinerror) when the link is malformed, the owner is
unreachable or refuses, or the local write fails.

```rust
fn join_cluster(store: &bookclerk_library::LibraryStore, link: &str, software_version: &str) -> Result<bookclerk_library::control_plane::ClusterLink, JoinError>
```



## bookclerk_node_client::enroll::refuse_cycle

*Function*

Refuses `ancestry` when it names this install's own cluster id, is
malformed, or would take this library's ancestry past what one can name
([`join_overflows_ancestry`](../bookclerk_library/control_plane/cluster_links.md#bookclerk_librarycontrol_planecluster_linksjoin_overflows_ancestry)).

# Errors

Returns `JoinError::Cycle`, `JoinError::AncestryTooDeep` or
`JoinError::Record`.

```rust
fn refuse_cycle(store: &bookclerk_library::LibraryStore, ancestry: &[String]) -> Result<(), JoinError>
```



