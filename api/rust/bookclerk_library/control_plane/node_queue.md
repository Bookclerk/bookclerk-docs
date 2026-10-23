**bookclerk_library > control_plane > node_queue**

# Module: control_plane::node_queue

## Contents

**Enums**

- [`NodeMessage`](#nodemessage) - One message on a `node.<node_id>` queue. The body is the JSON of this

**Functions**

- [`is_node_consumer_id`](#is_node_consumer_id) - True when a catalog / delivery `plugin_id` is a node consumer rather than
- [`is_valid_node_id`](#is_valid_node_id) - True when `node_id` can name a node: `node.<node_id>` is a valid queue
- [`node_consumer_id`](#node_consumer_id) - Catalog id of a node's own consumer (`node:<node_id>`).
- [`node_queue_name`](#node_queue_name) - Queue a node consumes: `node.<node_id>`. Registration accepts only node
- [`publish_node_message`](#publish_node_message) - Publishes `message` onto `node.<node_id>`.
- [`publish_to_all_nodes`](#publish_to_all_nodes) - Publishes `message` onto every registered node's queue.

**Constants**

- [`NODE_CONSUMER_PREFIX`](#node_consumer_prefix) - Catalog `plugin_id` prefix of a node's own consumer (`node:<node_id>`).

---

## bookclerk_library::control_plane::node_queue::NODE_CONSUMER_PREFIX

*Constant*: `&str`

Catalog `plugin_id` prefix of a node's own consumer (`node:<node_id>`).



## bookclerk_library::control_plane::node_queue::NodeMessage

*Enum*

One message on a `node.<node_id>` queue. The body is the JSON of this
enum (`{"type": "deployment.changed", ...}`).

**Variants:**
- `DeploymentChanged{ deployment_id: String, plugin_instance_id: String, desired: String, revision: i64 }` - A deployment targeting this node was created, replaced, or deleted.
- `InstallChanged{ plugin_key: String, version: String, revision: i64 }` - The active version of a plugin moved (install, update, rollback,
- `PlacementChanged{ subject: String, revision: i64 }` - Placement intent, or a fact it depends on, changed: an instance's
- `ConfigurationChanged{ scope_type: String, scope_id: String, namespace: String, revision: i64 }` - A configuration document changed (cluster settings or an instance
- `NodeRevoked{ node_id: String }` - A host node's credential was revoked. The node holding that host's

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NodeMessage`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeMessage) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::control_plane::node_queue::is_node_consumer_id

*Function*

True when a catalog / delivery `plugin_id` is a node consumer rather than
a plugin.

```rust
fn is_node_consumer_id(plugin_id: &str) -> bool
```



## bookclerk_library::control_plane::node_queue::is_valid_node_id

*Function*

True when `node_id` can name a node: `node.<node_id>` is a valid queue
name. Minted ids are lowercase UUIDs, which always qualify.

```rust
fn is_valid_node_id(node_id: &str) -> bool
```



## bookclerk_library::control_plane::node_queue::node_consumer_id

*Function*

Catalog id of a node's own consumer (`node:<node_id>`).

```rust
fn node_consumer_id(node_id: &str) -> String
```



## bookclerk_library::control_plane::node_queue::node_queue_name

*Function*

Queue a node consumes: `node.<node_id>`. Registration accepts only node
ids for which this is a valid queue name ([`is_valid_node_id`](node_queue.md#bookclerk_librarycontrol_planenode_queueis_valid_node_id)).

```rust
fn node_queue_name(node_id: &str) -> String
```



## bookclerk_library::control_plane::node_queue::publish_node_message

*Function*

Publishes `message` onto `node.<node_id>`.

# Errors

Returns an error when the outbox insert fails.

```rust
fn publish_node_message(store: &crate::LibraryStore, node_id: &str, message: &NodeMessage) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::node_queue::publish_to_all_nodes

*Function*

Publishes `message` onto every registered node's queue.

# Errors

Returns an error when the node list or an outbox insert fails.

```rust
fn publish_to_all_nodes(store: &crate::LibraryStore, message: &NodeMessage) -> crate::error::Result<()>
```



