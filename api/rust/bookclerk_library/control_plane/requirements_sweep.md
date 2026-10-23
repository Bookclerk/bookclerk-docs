**bookclerk_library > control_plane > requirements_sweep**

# Module: control_plane::requirements_sweep

## Contents

**Structs**

- [`RequirementsSweepReport`](#requirementssweepreport) - What one sweep did.

**Functions**

- [`sweep_job_requirements`](#sweep_job_requirements) - Re-resolves the requirements of every pending routed job (see the module

**Constants**

- [`REQUIREMENTS_GRACE`](#requirements_grace) - How long a pending job's requirements may stay unsettled before the
- [`REQUIREMENTS_SWEEP_LEASE`](#requirements_sweep_lease) - Cluster lease the daemon holds while it sweeps.
- [`REQUIREMENTS_SWEEP_LEASE_TTL`](#requirements_sweep_lease_ttl) - Lease length; the holder renews it on every pass.
- [`UNROUTABLE_ERROR_KIND`](#unroutable_error_kind) - `error_kind` of a job the sweep failed.

---

## bookclerk_library::control_plane::requirements_sweep::REQUIREMENTS_GRACE

*Constant*: `std::time::Duration`

How long a pending job's requirements may stay unsettled before the
sweep acts on them. Several scheduler passes (20 s apart) and guest
restarts fit inside it.



## bookclerk_library::control_plane::requirements_sweep::REQUIREMENTS_SWEEP_LEASE

*Constant*: `&str`

Cluster lease the daemon holds while it sweeps.



## bookclerk_library::control_plane::requirements_sweep::REQUIREMENTS_SWEEP_LEASE_TTL

*Constant*: `std::time::Duration`

Lease length; the holder renews it on every pass.



## bookclerk_library::control_plane::requirements_sweep::RequirementsSweepReport

*Struct*

What one sweep did.

**Fields:**
- `examined: u32` - Pending jobs whose requirements were re-resolved.
- `rewritten: u32` - Jobs whose requirement set was rewritten.
- `unsettled: u32` - Jobs newly marked, or still, unsettled within the grace period.
- `settled: u32` - Jobs whose unsettled mark was cleared without another change.
- `failed: u32` - Jobs failed as unroutable after the grace period.
- `lost: u32` - Writes that lost their compare-and-swap (the job was claimed).

**Methods:**

- `fn changed(self: &Self) -> bool` - True when the sweep wrote anything.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> RequirementsSweepReport`
- **Clone**
  - `fn clone(self: &Self) -> RequirementsSweepReport`
- **PartialEq**
  - `fn eq(self: &Self, other: &RequirementsSweepReport) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::requirements_sweep::UNROUTABLE_ERROR_KIND

*Constant*: `&str`

`error_kind` of a job the sweep failed.



## bookclerk_library::control_plane::requirements_sweep::sweep_job_requirements

*Function*

Re-resolves the requirements of every pending routed job (see the module
docs for the rules).

`node_bound_plugin_keys` are the plugin keys of node-bound outputs. A
recorded instance whose key is missing from them is treated as no longer
required, so the caller must pass a set that does not depend on what the
sweeping node has installed, and must not sweep with a set it could not
read. `grace` is normally [`REQUIREMENTS_GRACE`](requirements_sweep.md#bookclerk_librarycontrol_planerequirements_sweeprequirements_grace); `now` is the sweep's
clock.

# Errors

Returns an error when a read or write fails; jobs handled before the
failure keep their writes.

```rust
fn sweep_job_requirements(store: &crate::store::LibraryStore, node_bound_plugin_keys: &std::collections::BTreeSet<String>, grace: std::time::Duration, now: chrono::DateTime<chrono::Utc>) -> crate::error::Result<RequirementsSweepReport>
```



