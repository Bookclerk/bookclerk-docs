**bookclerk_library > control_plane > routing**

# Module: control_plane::routing

## Contents

**Structs**

- [`RouteRank`](#routerank) - This node's position among the live nodes running one instance.

**Functions**

- [`rank_nodes`](#rank_nodes) - Ranks `running` nodes for `demand`, best first (most room, then node id).
- [`route_ranks`](#route_ranks) - For each claim id that is a plugin instance id, `node_id`'s rank among

**Constants**

- [`ROUTE_DEFER_PER_RANK`](#route_defer_per_rank) - How long each rank waits behind the one above it.

---

## bookclerk_library::control_plane::routing::ROUTE_DEFER_PER_RANK

*Constant*: `std::time::Duration`

How long each rank waits behind the one above it.



## bookclerk_library::control_plane::routing::RouteRank

*Struct*

This node's position among the live nodes running one instance.

**Fields:**
- `rank: usize` - 0 for the best node.
- `nodes: Vec<String>` - Live nodes running the instance, best first.

**Methods:**

- `fn defer(self: &Self) -> Duration` - How long this node lets a delivery wait before claiming it.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RouteRank`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RouteRank) -> bool`



## bookclerk_library::control_plane::routing::rank_nodes

*Function*

Ranks `running` nodes for `demand`, best first (most room, then node id).

```rust
fn rank_nodes(running: &[super::placement::NodeCandidate], demand: super::node_status::GuestReservation) -> Vec<String>
```



## bookclerk_library::control_plane::routing::route_ranks

*Function*

For each claim id that is a plugin instance id, `node_id`'s rank among
the live nodes whose guest of that instance is up (`running` or
`healthy`). Ids that are not instance ids (plugin keys, aliases, node
consumers) are left out: they are not routed. A node missing from the
list (no heartbeat yet) ranks after every live node.

# Errors

Returns an error when a read fails.

```rust
fn route_ranks(store: &crate::store::LibraryStore, node_id: &str, claim_ids: &[String]) -> crate::error::Result<std::collections::BTreeMap<String, RouteRank>>
```



