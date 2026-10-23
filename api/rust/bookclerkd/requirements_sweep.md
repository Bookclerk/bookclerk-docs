**bookclerkd > requirements_sweep**

# Module: requirements_sweep

## Contents

**Functions**

- [`run_requirements_sweep`](#run_requirements_sweep) - One pass: take or renew the lease, then sweep. Returns `None` when
- [`spawn_requirements_sweep`](#spawn_requirements_sweep) - Starts the sweep loop.

**Constants**

- [`SWEEP_INTERVAL`](#sweep_interval) - Longest gap between sweeps; well inside [`REQUIREMENTS_SWEEP_LEASE_TTL`](../bookclerk_library/control_plane/requirements_sweep.md#bookclerk_librarycontrol_planerequirements_sweeprequirements_sweep_lease_ttl)

---

## bookclerkd::requirements_sweep::SWEEP_INTERVAL

*Constant*: `std::time::Duration`

Longest gap between sweeps; well inside [`REQUIREMENTS_SWEEP_LEASE_TTL`](../bookclerk_library/control_plane/requirements_sweep.md#bookclerk_librarycontrol_planerequirements_sweeprequirements_sweep_lease_ttl)
so the holder keeps its lease.



## bookclerkd::requirements_sweep::run_requirements_sweep

*Function*

One pass: take or renew the lease, then sweep. Returns `None` when
another node holds the lease or the pass failed.

```rust
fn run_requirements_sweep(state: &crate::api::AppState) -> Option<bookclerk_library::control_plane::RequirementsSweepReport>
```



## bookclerkd::requirements_sweep::spawn_requirements_sweep

*Function*

Starts the sweep loop.

```rust
fn spawn_requirements_sweep(state: std::sync::Arc<crate::api::AppState>)
```



