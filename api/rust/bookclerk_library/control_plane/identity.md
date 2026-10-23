**bookclerk_library > control_plane > identity**

# Module: control_plane::identity

## Contents

**Structs**

- [`NodeIdentity`](#nodeidentity) - Bootstrap identity loaded from [`NODE_IDENTITY_FILE`](identity.md#bookclerk_librarycontrol_planeidentitynode_identity_file).
- [`NodeRecord`](#noderecord) - Durable node row plus the latest observation columns.

**Enums**

- [`NodeKind`](#nodekind) - How a node joined the cluster.

**Functions**

- [`current_process_incarnation`](#current_process_incarnation) - This process's incarnation. Stable until the process exits.
- [`encode_node_identity_file`](#encode_node_identity_file) - The [`NODE_IDENTITY_FILE`](identity.md#bookclerk_librarycontrol_planeidentitynode_identity_file) contents [`install_node_identity`](identity.md#bookclerk_librarycontrol_planeidentityinstall_node_identity) writes, for
- [`install_node_identity`](#install_node_identity) - Writes the identity file for a node id the cluster already recorded,
- [`list_node_ids`](#list_node_ids) - Lists node ids, sorted.
- [`list_nodes`](#list_nodes) - Every node row, sorted by id.
- [`load_node`](#load_node) - Loads one node row.
- [`load_or_create_node_identity`](#load_or_create_node_identity) - Loads the identity file or creates a new node id.
- [`node_identity_path`](#node_identity_path) - Path of the node identity file.
- [`node_target`](#node_target) - Bookclerk artifact target this binary runs as (`linux-x64-gnu`,

**Constants**

- [`NODE_IDENTITY_FILE`](#node_identity_file) - Bootstrap file under `$BOOKCLERK_FILES_DIR` that stores the stable node id.

---

## bookclerk_library::control_plane::identity::NODE_IDENTITY_FILE

*Constant*: `&str`

Bootstrap file under `$BOOKCLERK_FILES_DIR` that stores the stable node id.



## bookclerk_library::control_plane::identity::NodeIdentity

*Struct*

Bootstrap identity loaded from [`NODE_IDENTITY_FILE`](identity.md#bookclerk_librarycontrol_planeidentitynode_identity_file).

**Fields:**
- `node_id: String` - Stable node id.
- `cluster_id: Option<String>` - Cluster binding, once enrollment has succeeded.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeIdentity) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NodeIdentity`



## bookclerk_library::control_plane::identity::NodeKind

*Enum*

How a node joined the cluster.

**Variants:**
- `Owner` - Shares the library database and `master.key`. Every node is one today.
- `Host` - Keeps its own database and operator, and enrolled to contribute work.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Stored text.
- `fn parse(raw: &str) -> Self` - Parses stored text; anything unknown reads as `Self::Owner`.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> NodeKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeKind) -> bool`



## bookclerk_library::control_plane::identity::NodeRecord

*Struct*

Durable node row plus the latest observation columns.

**Fields:**
- `node_id: String` - Stable node id.
- `cluster_id: String` - Cluster id stored with the node.
- `created_at: String` - RFC 3339 creation time. Heartbeats do not change it.
- `incarnation: String` - Process incarnation from the latest heartbeat.
- `heartbeat_at: String` - RFC 3339 heartbeat time.
- `software_version: String` - Software version from the latest heartbeat.
- `schema_state: String` - Schema state display from the latest heartbeat.
- `compatible: bool` - True when the reporting binary accepted the library schema.
- `logical_cpus: Option<i64>` - `available_parallelism`, empty when the probe failed.
- `cpu_max_quota_us: Option<i64>` - Process cgroup `cpu.max` quota in microseconds. Empty when unlimited or missing.
- `cpu_max_period_us: Option<i64>` - Process cgroup `cpu.max` period in microseconds. Empty when unlimited or missing.
- `memory_max_bytes: Option<i64>` - Process cgroup `memory.max` in bytes. Empty when unlimited or missing.
- `memory_current_bytes: Option<i64>` - Process cgroup `memory.current` in bytes.
- `memory_anon_bytes: Option<i64>` - Process cgroup `memory.stat` field `anon`.
- `files_dir_free_bytes: Option<i64>` - Free bytes on the filesystem that holds the files directory.
- `scratch_bytes: Option<i64>` - Byte size of the acquire scratch tree under the download cache.
- `target: String` - Bookclerk artifact target of the reporting binary; empty on a row
- `labels: Vec<String>` - Operator-assigned placement labels, sorted.
- `kind: NodeKind` - How the node joined the cluster.
- `owner_user_id: Option<i64>` - User who owns this node and may administer it, when one is assigned.
- `share_books: Option<bool>` - For a branch: "Share this branch's books" as it last reported it;
- `include_branches: Option<bool>` - For a branch: "Include books from my own branches", as reported.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeRecord) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NodeRecord`



## bookclerk_library::control_plane::identity::current_process_incarnation

*Function*

This process's incarnation. Stable until the process exits.

```rust
fn current_process_incarnation() -> &'static str
```



## bookclerk_library::control_plane::identity::encode_node_identity_file

*Function*

The [`NODE_IDENTITY_FILE`](identity.md#bookclerk_librarycontrol_planeidentitynode_identity_file) contents [`install_node_identity`](identity.md#bookclerk_librarycontrol_planeidentityinstall_node_identity) writes, for
a caller that stages the file itself (a clerk applying its bootstrap as
one recoverable step) with mode `0600`.

# Errors

Returns `LibraryError::Invalid` for a node id that is not a UUID or an
empty cluster id, and the encode error.

```rust
fn encode_node_identity_file(node_id: &str, cluster_id: &str) -> crate::error::Result<Vec<u8>>
```



## bookclerk_library::control_plane::identity::install_node_identity

*Function*

Writes the identity file for a node id the cluster already recorded,
bound to `cluster_id`, replacing any file there atomically. A clerk
joining a library takes the node id the library minted for it at
redeem, so its first heartbeat lands on that `nodes` row.

# Errors

Returns `LibraryError::Invalid` for a node id that is not a UUID or an
empty cluster id, and the write error.

```rust
fn install_node_identity(files_dir: &std::path::Path, node_id: &str, cluster_id: &str) -> crate::error::Result<NodeIdentity>
```



## bookclerk_library::control_plane::identity::list_node_ids

*Function*

Lists node ids, sorted.

# Errors

Returns an error when the read fails.

```rust
fn list_node_ids(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<String>>
```



## bookclerk_library::control_plane::identity::list_nodes

*Function*

Every node row, sorted by id.

# Errors

Returns an error when the read fails.

```rust
fn list_nodes(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<NodeRecord>>
```



## bookclerk_library::control_plane::identity::load_node

*Function*

Loads one node row.

# Errors

Returns an error when the row is missing or the read fails.

```rust
fn load_node(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<NodeRecord>
```



## bookclerk_library::control_plane::identity::load_or_create_node_identity

*Function*

Loads the identity file or creates a new node id.

Concurrent creators of the same path share the winner's node id.
Copying the file copies the node id. A new file is a distinct node.

# Errors

Returns an error when the file is unreadable, has an unknown version, or
cannot be created.

```rust
fn load_or_create_node_identity(files_dir: &std::path::Path) -> crate::error::Result<NodeIdentity>
```



## bookclerk_library::control_plane::identity::node_identity_path

*Function*

Path of the node identity file.

```rust
fn node_identity_path(files_dir: &std::path::Path) -> std::path::PathBuf
```



## bookclerk_library::control_plane::identity::node_target

*Function*

Bookclerk artifact target this binary runs as (`linux-x64-gnu`,
`macos-arm64`, …). Matches `bookclerk_plugin_catalog::host_bookclerk_target`
(a plugin-host test pins the two together).

```rust
fn node_target() -> &'static str
```



