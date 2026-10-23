**bookclerk_library > control_plane > instances**

# Module: control_plane::instances

## Contents

**Structs**

- [`PluginInstance`](#plugininstance) - One `plugin_instances` row.
- [`PluginInstanceId`](#plugininstanceid) - Stable id of one plugin instance.

**Functions**

- [`create_node_local_instance`](#create_node_local_instance) - Inserts a node-local instance of `plugin_key` owned by `node_id` (H-2):
- [`create_plugin_instance`](#create_plugin_instance) - Inserts a new instance for `plugin_key`.
- [`default_instance_for_key`](#default_instance_for_key) - The instance a host routes to when a job does not pin one: the
- [`delete_plugin_instance`](#delete_plugin_instance) - Deletes an instance that has no deployments. Its configuration document,
- [`ensure_plugin_instance`](#ensure_plugin_instance) - Returns the existing instance for `plugin_key`, or inserts one.
- [`find_plugin_instance_by_key`](#find_plugin_instance_by_key) - Earliest instance row for `plugin_key`, if any.
- [`list_all_plugin_instances`](#list_all_plugin_instances) - Every instance row, ordered by plugin key then creation time.
- [`list_plugin_instances_for_key`](#list_plugin_instances_for_key) - Every instance row for `plugin_key`, oldest first.
- [`load_plugin_instance`](#load_plugin_instance) - Loads one instance by id.

---

## bookclerk_library::control_plane::instances::PluginInstance

*Struct*

One `plugin_instances` row.

**Fields:**
- `id: PluginInstanceId` - Stable id.
- `plugin_key: String` - Canonical plugin key. Immutable after create.
- `display_name: String` - Operator-facing name; empty for the default instance.
- `created_at: String` - RFC 3339 insert time.
- `created_by: String` - Audit identity of the creator.
- `owner_node_id: Option<String>` - The node that owns this instance (a node-local output, H-2); it may

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginInstance) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PluginInstance`



## bookclerk_library::control_plane::instances::PluginInstanceId

*Struct*

Stable id of one plugin instance.

A hyphenated UUID version 4. It does not parse as a plugin key.

**Tuple Struct**: `()`

**Methods:**

- `fn mint() -> Self` - Mints a new random id.
- `fn parse(raw: &str) -> Result<Self>` - Parses a hyphenated UUID.
- `fn as_str(self: &Self) -> &str` - Hyphenated UUID text stored in the database.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PluginInstanceId) -> bool`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &PluginInstanceId) -> $crate::option::Option<$crate::cmp::Ordering>`
- **Clone**
  - `fn clone(self: &Self) -> PluginInstanceId`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Display**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Ord**
  - `fn cmp(self: &Self, other: &PluginInstanceId) -> $crate::cmp::Ordering`



## bookclerk_library::control_plane::instances::create_node_local_instance

*Function*

Inserts a node-local instance of `plugin_key` owned by `node_id` (H-2):
it adds to the cluster's outputs on that node only, and a deployment of
it to any other node, or a placement, is refused.

# Errors

Returns an error when the actor may not create an instance, the node id
is malformed or has no `nodes` row, or the insert fails.

```rust
fn create_node_local_instance(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, plugin_key: &str, node_id: &str, display_name: &str) -> crate::error::Result<PluginInstance>
```



## bookclerk_library::control_plane::instances::create_plugin_instance

*Function*

Inserts a new instance for `plugin_key`.

A second call mints a different id even when `plugin_key` matches.

# Errors

Returns an error when the actor may not create an instance or the insert fails.

```rust
fn create_plugin_instance(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, plugin_key: &str, display_name: &str) -> crate::error::Result<PluginInstance>
```



## bookclerk_library::control_plane::instances::default_instance_for_key

*Function*

The instance a host routes to when a job does not pin one: the
`plugin_instance_defaults` claim for `plugin_key`, else the oldest row.

# Errors

Returns an error when the read fails.

```rust
fn default_instance_for_key(store: &crate::store::LibraryStore, plugin_key: &str) -> crate::error::Result<Option<PluginInstance>>
```



## bookclerk_library::control_plane::instances::delete_plugin_instance

*Function*

Deletes an instance that has no deployments. Its configuration document,
audit rows, and sealed secrets are left for retention cleanup.

# Errors

Returns an error when the actor may not write, a deployment still
references the instance, or the delete fails.

```rust
fn delete_plugin_instance(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, id: &PluginInstanceId) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::instances::ensure_plugin_instance

*Function*

Returns the existing instance for `plugin_key`, or inserts one.

GraphicAudio startup uses this so a second process start keeps the original id.
Concurrent calls, including two connections to the same file, share one
inserted row. The claim is one canonical host batch (no interactive
`BEGIN`), so D1 and other non-interactive adapters can run it. A lock or
uniqueness failure retries that batch. Explicit [`create_plugin_instance`](instances.md#bookclerk_librarycontrol_planeinstancescreate_plugin_instance)
can still mint another row for the same key; that path does not claim the
default enrollment.

# Errors

Returns an error when the actor may not create an instance or the write fails.

```rust
fn ensure_plugin_instance(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, plugin_key: &str) -> crate::error::Result<PluginInstance>
```



## bookclerk_library::control_plane::instances::find_plugin_instance_by_key

*Function*

Earliest instance row for `plugin_key`, if any.

Oldest-wins. Prefer [`default_instance_for_key`](instances.md#bookclerk_librarycontrol_planeinstancesdefault_instance_for_key), which honours the
`plugin_instance_defaults` claim and falls back to this order.

# Errors

Returns an error when the read fails.

```rust
fn find_plugin_instance_by_key(store: &crate::store::LibraryStore, plugin_key: &str) -> crate::error::Result<Option<PluginInstance>>
```



## bookclerk_library::control_plane::instances::list_all_plugin_instances

*Function*

Every instance row, ordered by plugin key then creation time.

# Errors

Returns an error when the read fails.

```rust
fn list_all_plugin_instances(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<PluginInstance>>
```



## bookclerk_library::control_plane::instances::list_plugin_instances_for_key

*Function*

Every instance row for `plugin_key`, oldest first.

# Errors

Returns an error when the read fails.

```rust
fn list_plugin_instances_for_key(store: &crate::store::LibraryStore, plugin_key: &str) -> crate::error::Result<Vec<PluginInstance>>
```



## bookclerk_library::control_plane::instances::load_plugin_instance

*Function*

Loads one instance by id.

# Errors

Returns an error when the read fails or `id` is not a UUID.

```rust
fn load_plugin_instance(store: &crate::store::LibraryStore, id: &PluginInstanceId) -> crate::error::Result<Option<PluginInstance>>
```



