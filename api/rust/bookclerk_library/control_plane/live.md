**bookclerk_library > control_plane > live**

# Module: control_plane::live

## Contents

**Structs**

- [`LiveRevision`](#liverevision) - One topic's revision: its row count and newest change stamp.

**Enums**

- [`LiveTopic`](#livetopic) - A topic of the live socket.

**Functions**

- [`changed_topics`](#changed_topics) - The topics whose revision differs between `before` and `after`.
- [`live_revisions`](#live_revisions) - Reads every topic's revision.

**Type Aliases**

- [`LiveRevisions`](#liverevisions) - Every topic's revision, in `LiveTopic::ALL` order.

---

## bookclerk_library::control_plane::live::LiveRevision

*Struct*

One topic's revision: its row count and newest change stamp.

**Fields:**
- `rows: u64` - Rows behind the topic.
- `newest: Option<String>` - The newest change stamp (RFC 3339 text or a sequence number), or a

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LiveRevision`
- **PartialEq**
  - `fn eq(self: &Self, other: &LiveRevision) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> LiveRevision`



## bookclerk_library::control_plane::live::LiveRevisions

*Type Alias*: `Vec<(LiveTopic, LiveRevision)>`

Every topic's revision, in `LiveTopic::ALL` order.



## bookclerk_library::control_plane::live::LiveTopic

*Enum*

A topic of the live socket.

**Variants:**
- `Library` - The library's titles (`books`).
- `Jobs` - Jobs (`jobs`).
- `Deliveries` - Event deliveries (`event_deliveries`).
- `Nodes` - Node heartbeats (`node_status`).
- `Config` - Cluster configuration documents (`configuration_changes`).
- `Consents` - Owner-plugin consent on this host (`remote_plugin_consents`).
- `Clusters` - Owner clusters this host joined, and their state (`cluster_links`).
- `Users` - Users (`users`).

**Methods:**

- `fn as_str(self: Self) -> &'static str` - The topic's wire name.

**Traits:** Copy, Eq

**Trait Implementations:**

- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &LiveTopic) -> $crate::option::Option<$crate::cmp::Ordering>`
- **Clone**
  - `fn clone(self: &Self) -> LiveTopic`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &LiveTopic) -> bool`
- **Ord**
  - `fn cmp(self: &Self, other: &LiveTopic) -> $crate::cmp::Ordering`



## bookclerk_library::control_plane::live::changed_topics

*Function*

The topics whose revision differs between `before` and `after`.

```rust
fn changed_topics(before: &LiveRevisions, after: &LiveRevisions) -> Vec<LiveTopic>
```



## bookclerk_library::control_plane::live::live_revisions

*Function*

Reads every topic's revision.

# Errors

Returns the read error.

```rust
fn live_revisions(store: &crate::store::LibraryStore) -> crate::error::Result<LiveRevisions>
```



