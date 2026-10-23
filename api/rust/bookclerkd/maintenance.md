**bookclerkd > maintenance**

# Module: maintenance

## Contents

**Structs**

- [`Maintenance`](#maintenance) - This node's view of every duty.

**Enums**

- [`Duty`](#duty) - One cluster-wide maintenance duty.

**Functions**

- [`spawn_maintenance`](#spawn_maintenance) - Starts this daemon's maintenance loop.

**Constants**

- [`MAINTENANCE_INTERVAL`](#maintenance_interval) - How often the holder runs each duty.
- [`MAINTENANCE_LEASE_TTL`](#maintenance_lease_ttl) - How long a maintenance lease lasts without renewal.
- [`MAINTENANCE_TICK`](#maintenance_tick) - How often every daemon polls its maintenance duties. Well inside

---

## bookclerkd::maintenance::Duty

*Enum*

One cluster-wide maintenance duty.

**Variants:**
- `JobRetention` - Terminal jobs past `jobs.retention_days`.
- `EventRetention` - Terminal deliveries, empty events, and stale subscriber catalog rows
- `KvExpiry` - Expired plugin KV rows (they already read as missing).
- `SessionExpiry` - Expired operator sessions, OIDC RP states, and WebAuthn challenges.

**Methods:**

- `fn lease(self: Self) -> &'static str` - The cluster lease this duty runs under.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &Duty) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> Duty`



## bookclerkd::maintenance::MAINTENANCE_INTERVAL

*Constant*: `std::time::Duration`

How often the holder runs each duty.



## bookclerkd::maintenance::MAINTENANCE_LEASE_TTL

*Constant*: `std::time::Duration`

How long a maintenance lease lasts without renewal.



## bookclerkd::maintenance::MAINTENANCE_TICK

*Constant*: `std::time::Duration`

How often every daemon polls its maintenance duties. Well inside
[`MAINTENANCE_LEASE_TTL`](maintenance.md#bookclerkdmaintenancemaintenance_lease_ttl) so the holder keeps its leases.



## bookclerkd::maintenance::Maintenance

*Struct*

This node's view of every duty.

**Fields:**
- `duties: Vec<(Duty, bookclerk_library::control_plane::SingletonDuty)>` - Each duty with its lease state on this node.

**Methods:**

- `fn new(node_id: &str, ttl: Duration, interval: Duration) -> Self` - Duties for `node_id` with the given lease TTL and run interval.
- `fn pass(self: & mut Self, state: &AppState) -> Vec<Duty>` - One tick: renews or takes each lease and runs the duties that are due

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::maintenance::spawn_maintenance

*Function*

Starts this daemon's maintenance loop.

```rust
fn spawn_maintenance(state: std::sync::Arc<crate::api::AppState>)
```



