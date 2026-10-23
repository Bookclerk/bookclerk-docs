**bookclerk_library > control_plane > clerks**

# Module: control_plane::clerks

## Contents

**Structs**

- [`LibraryContents`](#librarycontents) - What library data an install holds of its own.

**Functions**

- [`library_contents`](#library_contents) - Counts the library data this install holds of its own.
- [`owner_nodes_visible_to`](#owner_nodes_visible_to) - The owner nodes (`kind = owner`: this one and its clerks) `actor` may
- [`remove_owner_node`](#remove_owner_node) - Removes owner node `node_id` from the library's node list, in one

---

## bookclerk_library::control_plane::clerks::LibraryContents

*Struct*

What library data an install holds of its own.

**Fields:**
- `users: u64` - Users of any role.
- `accounts: u64` - Store accounts.
- `books: u64` - Books.
- `libraries_joined: u64` - Libraries this install is a branch of (not left).

**Methods:**

- `fn is_empty(self: &Self) -> bool` - True when the install holds no library data: it may join as a clerk.
- `fn reasons(self: &Self) -> Vec<String>` - Why the install is not empty, for the operator.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LibraryContents`
- **PartialEq**
  - `fn eq(self: &Self, other: &LibraryContents) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> LibraryContents`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::control_plane::clerks::library_contents

*Function*

Counts the library data this install holds of its own.

# Errors

Returns the read error.

```rust
fn library_contents(store: &crate::store::LibraryStore) -> crate::error::Result<LibraryContents>
```



## bookclerk_library::control_plane::clerks::owner_nodes_visible_to

*Function*

The owner nodes (`kind = owner`: this one and its clerks) `actor` may
see, by the rule [`remove_owner_node`](clerks.md#bookclerk_librarycontrol_planeclerksremove_owner_node) removes them: the operator or an
active Owner sees every one; another active user only the ones they own;
an unknown or disabled user none.

# Errors

Returns the read error.

```rust
fn owner_nodes_visible_to(store: &crate::store::LibraryStore, actor: &super::enrollment::EnrollmentActor) -> crate::error::Result<Vec<super::identity::NodeRecord>>
```



## bookclerk_library::control_plane::clerks::remove_owner_node

*Function*

Removes owner node `node_id` from the library's node list, in one
transaction: its `nodes` row (and with it its labels and credentials),
status, event-subscriber registrations, secret handoffs, and plugin
deployments with their observations. Only a node that is not
running may go, and never `this_node`. The liveness check and the
removal are one step: the node's status row is deleted only if it still
holds the stale heartbeat that was checked, so a heartbeat that lands
in between refuses the removal. A node that was only cut off from the
database for longer than its liveness window can still be removed while
it runs; its next heartbeat writes a status row again, but not its node
row. Removing a node does not take back the
database credentials or DEK it held; rotate those when it is not
trusted any more.

The operator or an Owner may remove any owner node; another user only
one they own.

# Errors

Returns `LibraryError::NotFound` for an unknown node or one the
caller may not see, `LibraryError::Invalid` for a host node (revoke
it instead) or this node, `LibraryError::Conflict` while it is live,
and the write error.

```rust
fn remove_owner_node(store: &crate::store::LibraryStore, actor: &super::enrollment::EnrollmentActor, node_id: &str, this_node: &str) -> crate::error::Result<()>
```



