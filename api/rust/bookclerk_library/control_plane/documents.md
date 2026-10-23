**bookclerk_library > control_plane > documents**

# Module: control_plane::documents

## Contents

**Structs**

- [`DocumentKey`](#documentkey) - Address of one configuration document.
- [`StoredDocument`](#storeddocument) - One stored document before domain parsing.
- [`VerifiedClusterOwner`](#verifiedclusterowner) - Proof that a user was an active Owner when the write was authorized.
- [`VerifiedNodeOwner`](#verifiednodeowner) - Proof that a user owned a node when the write was authorized.

**Enums**

- [`ConfigActor`](#configactor) - Who is allowed to write a configuration document.
- [`ReplaceOutcome`](#replaceoutcome) - Result of a compare-and-swap replacement.

**Functions**

- [`audit_count`](#audit_count) - Counts audit rows for one namespace.
- [`change_count`](#change_count) - Counts committed change notices for one namespace.

**Constants**

- [`CLUSTER_SCOPE_ID`](#cluster_scope_id) - Cluster scope instance. There is one cluster document per namespace.
- [`CLUSTER_SCOPE_TYPE`](#cluster_scope_type) - Scope type stored for cluster documents.
- [`MAX_CONFIGURATION_DOCUMENT_BYTES`](#max_configuration_document_bytes) - Maximum UTF-8 size of `document_json`.
- [`NODE_SCOPE_TYPE`](#node_scope_type) - Scope type stored for node documents.
- [`PLUGIN_INSTANCE_CONFIG_NAMESPACE`](#plugin_instance_config_namespace) - Namespace of a plugin instance's typed configuration document.
- [`PLUGIN_INSTANCE_SCOPE_TYPE`](#plugin_instance_scope_type) - Scope type stored for one plugin instance's configuration.

---

## bookclerk_library::control_plane::documents::CLUSTER_SCOPE_ID

*Constant*: `&str`

Cluster scope instance. There is one cluster document per namespace.



## bookclerk_library::control_plane::documents::CLUSTER_SCOPE_TYPE

*Constant*: `&str`

Scope type stored for cluster documents.



## bookclerk_library::control_plane::documents::ConfigActor

*Enum*

Who is allowed to write a configuration document.

**Variants:**
- `Operator{ id: String }` - Control-plane operator (CLI or operator token).
- `Administrator{ id: String }` - Portal administrator. Cannot write cluster or node configuration.
- `Member{ id: String }` - Portal member. Cannot write configuration.
- `Bootstrap` - Process startup. May import a missing document and may rewrite an
- `Scheduler` - The placement scheduler (one daemon, under the `scheduler` lease). It
- `NodeOwner(VerifiedNodeOwner)` - The user who owns one node. Writes that node's `node.runtime`
- `ClusterOwner(VerifiedClusterOwner)` - An active portal Owner. Writes the cluster's host-enrollment switch

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ConfigActor) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ConfigActor`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::documents::DocumentKey

*Struct*

Address of one configuration document.

**Fields:**
- `scope_type: String` - `cluster`, `node`, or `plugin_instance`.
- `scope_id: String` - `singleton`, a node id, or a plugin instance id.
- `namespace: String` - Domain namespace.

**Methods:**

- `fn cluster(namespace: &str) -> Self` - Cluster-scoped domain.
- `fn node(node_id: &str, namespace: &str) -> Self` - Node-scoped domain.
- `fn plugin_instance(instance_id: &str) -> Self` - Plugin-instance configuration document.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DocumentKey`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &DocumentKey) -> bool`



## bookclerk_library::control_plane::documents::MAX_CONFIGURATION_DOCUMENT_BYTES

*Constant*: `usize`

Maximum UTF-8 size of `document_json`.



## bookclerk_library::control_plane::documents::NODE_SCOPE_TYPE

*Constant*: `&str`

Scope type stored for node documents.



## bookclerk_library::control_plane::documents::PLUGIN_INSTANCE_CONFIG_NAMESPACE

*Constant*: `&str`

Namespace of a plugin instance's typed configuration document.



## bookclerk_library::control_plane::documents::PLUGIN_INSTANCE_SCOPE_TYPE

*Constant*: `&str`

Scope type stored for one plugin instance's configuration.



## bookclerk_library::control_plane::documents::ReplaceOutcome

*Enum*

Result of a compare-and-swap replacement.

**Variants:**
- `Applied(StoredDocument)` - The document is the committed body.
- `Conflict{ current_revision: i64 }` - The expected revision lost. No document, audit, or change row was written.
- `Replayed{ revision: i64 }` - This `operation_id` already committed. The document was not modified again.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ReplaceOutcome) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ReplaceOutcome`



## bookclerk_library::control_plane::documents::StoredDocument

*Struct*

One stored document before domain parsing.

**Fields:**
- `key: DocumentKey` - Document address.
- `schema_version: i64` - Domain schema version column.
- `revision: i64` - Compare-and-swap revision.
- `document_json: String` - JSON body.
- `updated_at: String` - RFC 3339 commit time.
- `updated_by: String` - Audit identity of the last writer.
- `write_operation_id: String` - Operation id of the write that produced this revision.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &StoredDocument) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> StoredDocument`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::documents::VerifiedClusterOwner

*Struct*

Proof that a user was an active Owner when the write was authorized.

Only [`super::verify_cluster_owner`](enrollment.md#bookclerk_librarycontrol_planeenrollmentverify_cluster_owner) constructs one.

**Methods:**

- `fn user_id(self: &Self) -> i64` - Owner's user id.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> VerifiedClusterOwner`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &VerifiedClusterOwner) -> bool`



## bookclerk_library::control_plane::documents::VerifiedNodeOwner

*Struct*

Proof that a user owned a node when the write was authorized.

Only [`super::verify_node_owner`](ownership.md#bookclerk_librarycontrol_planeownershipverify_node_owner) constructs one.

**Methods:**

- `fn user_id(self: &Self) -> i64` - Owning user id.
- `fn node_id(self: &Self) -> &str` - Node this owner may administer.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> VerifiedNodeOwner`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &VerifiedNodeOwner) -> bool`



## bookclerk_library::control_plane::documents::audit_count

*Function*

Counts audit rows for one namespace.

# Errors

Returns an error when the read fails.

```rust
fn audit_count(store: &crate::store::LibraryStore, namespace: &str) -> crate::error::Result<u64>
```



## bookclerk_library::control_plane::documents::change_count

*Function*

Counts committed change notices for one namespace.

# Errors

Returns an error when the read fails.

```rust
fn change_count(store: &crate::store::LibraryStore, namespace: &str) -> crate::error::Result<u64>
```



