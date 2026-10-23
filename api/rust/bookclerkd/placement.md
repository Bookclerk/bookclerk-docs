**bookclerkd > placement**

# Module: placement

## Contents

**Functions**

- [`run_placement_pass`](#run_placement_pass) - One pass: take or renew the lease, then schedule. Returns `None` when
- [`spawn_placement_scheduler`](#spawn_placement_scheduler) - Starts the scheduler loop.

**Constants**

- [`PLACEMENT_INTERVAL`](#placement_interval) - Longest gap between scheduler passes; well inside

---

## bookclerkd::placement::PLACEMENT_INTERVAL

*Constant*: `std::time::Duration`

Longest gap between scheduler passes; well inside
[`SCHEDULER_LEASE_TTL`](../bookclerk_library/control_plane/placement.md#bookclerk_librarycontrol_planeplacementscheduler_lease_ttl) so the holder keeps its lease.



## bookclerkd::placement::run_placement_pass

*Function*

One pass: take or renew the lease, then schedule. Returns `None` when
another node holds the lease or the pass failed.

```rust
fn run_placement_pass(state: &crate::api::AppState) -> Option<bookclerk_library::control_plane::ScheduleReport>
```



## bookclerkd::placement::spawn_placement_scheduler

*Function*

Starts the scheduler loop.

```rust
fn spawn_placement_scheduler(state: std::sync::Arc<crate::api::AppState>)
```



