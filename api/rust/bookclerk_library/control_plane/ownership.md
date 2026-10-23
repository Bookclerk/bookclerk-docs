**bookclerk_library > control_plane > ownership**

# Module: control_plane::ownership

## Contents

**Functions**

- [`nodes_owned_by`](#nodes_owned_by) - Nodes `user_id` owns, sorted by id.
- [`set_node_owner`](#set_node_owner) - Assigns `node_id` to `owner` (a user id), or clears its owner.
- [`verify_node_owner`](#verify_node_owner) - Returns a `ConfigActor::NodeOwner` for `user_id` on `node_id` when the

---

## bookclerk_library::control_plane::ownership::nodes_owned_by

*Function*

Nodes `user_id` owns, sorted by id.

# Errors

Returns an error when the read fails.

```rust
fn nodes_owned_by(store: &crate::store::LibraryStore, user_id: i64) -> crate::error::Result<Vec<super::identity::NodeRecord>>
```



## bookclerk_library::control_plane::ownership::set_node_owner

*Function*

Assigns `node_id` to `owner` (a user id), or clears its owner.

The owner must be an active user. Any role may own a node: owning one
lets the user administer it, while the work the node is trusted with
still follows the user's role.

# Errors

Returns `LibraryError::Forbidden` unless the actor is an operator,
`LibraryError::NotFound` when the node or user is unknown,
`LibraryError::Invalid` when the user is disabled, and the write error
when the update fails.

```rust
fn set_node_owner(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, node_id: &str, owner: Option<i64>) -> crate::error::Result<super::identity::NodeRecord>
```



## bookclerk_library::control_plane::ownership::verify_node_owner

*Function*

Returns a `ConfigActor::NodeOwner` for `user_id` on `node_id` when the
node's stored owner is that user.

# Errors

Returns `LibraryError::Forbidden` when the node is unknown or owned by
someone else, and the read error when the lookup fails.

```rust
fn verify_node_owner(store: &crate::store::LibraryStore, user_id: i64, node_id: &str) -> crate::error::Result<super::documents::ConfigActor>
```



