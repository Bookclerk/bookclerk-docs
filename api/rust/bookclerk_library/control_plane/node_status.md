**bookclerk_library > control_plane > node_status**

# Module: control_plane::node_status

## Contents

**Structs**

- [`GuestReservation`](#guestreservation) - CPU and memory one guest may use at most: its grant's ceilings after the
- [`NodeHeartbeat`](#nodeheartbeat) - One heartbeat's measurements, before budget and accounting are applied.
- [`NodeSampler`](#nodesampler) - Samples capacity and measured use for heartbeats. Keeps the previous CPU
- [`NodeStatus`](#nodestatus) - A node's latest heartbeat with budget, accounting, and availability.

**Functions**

- [`instance_demand`](#instance_demand) - What one more guest of `instance_id` is expected to need: the largest
- [`list_node_statuses`](#list_node_statuses) - Every node's latest heartbeat, by node id.
- [`load_node_status`](#load_node_status) - One node's latest heartbeat, if its daemon ever ran.
- [`next_availability`](#next_availability) - Next availability average (parts per million) after `gap_ms` since the
- [`node_reservations`](#node_reservations) - Sum of the ceilings of the guests running on `node_id`, and how many run.
- [`record_deployment_reservation`](#record_deployment_reservation) - Records the ceilings of the guest a node just started for a deployment.
- [`record_node_heartbeat`](#record_node_heartbeat) - Records this node's heartbeat: its measurements, its budget from

**Constants**

- [`AVAILABILITY_FULL_PPM`](#availability_full_ppm) - Full availability in parts per million.
- [`AVAILABILITY_TAU`](#availability_tau) - Time constant of the availability average (about a day).
- [`DEFAULT_GUEST_CPU_MILLICORES`](#default_guest_cpu_millicores) - CPU a guest is assumed to need when no ceiling is known (the grant
- [`DEFAULT_GUEST_MEMORY_BYTES`](#default_guest_memory_bytes) - Memory a guest is assumed to need when no ceiling is known (the grant
- [`NODE_HEARTBEAT_INTERVAL`](#node_heartbeat_interval) - How often a daemon heartbeats.
- [`NODE_LIVE_INTERVALS`](#node_live_intervals) - A node whose last heartbeat is older than this many intervals is not live.

---

## bookclerk_library::control_plane::node_status::AVAILABILITY_FULL_PPM

*Constant*: `i64`

Full availability in parts per million.



## bookclerk_library::control_plane::node_status::AVAILABILITY_TAU

*Constant*: `std::time::Duration`

Time constant of the availability average (about a day).



## bookclerk_library::control_plane::node_status::DEFAULT_GUEST_CPU_MILLICORES

*Constant*: `i64`

CPU a guest is assumed to need when no ceiling is known (the grant
default, 80% of a core).



## bookclerk_library::control_plane::node_status::DEFAULT_GUEST_MEMORY_BYTES

*Constant*: `i64`

Memory a guest is assumed to need when no ceiling is known (the grant
default).



## bookclerk_library::control_plane::node_status::GuestReservation

*Struct*

CPU and memory one guest may use at most: its grant's ceilings after the
node's `[plugins.jail]` limits.

**Fields:**
- `memory_bytes: i64` - Memory ceiling in bytes.
- `cpu_millicores: i64` - CPU ceiling in millicores.

**Methods:**

- `fn default_guest() -> Self` - The grant defaults, used when nothing better is known.
- `fn plus(self: Self, other: Self) -> Self` - Component-wise sum.
- `fn max(self: Self, other: Self) -> Self` - Component-wise maximum.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &GuestReservation) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> GuestReservation`
- **Clone**
  - `fn clone(self: &Self) -> GuestReservation`



## bookclerk_library::control_plane::node_status::NODE_HEARTBEAT_INTERVAL

*Constant*: `std::time::Duration`

How often a daemon heartbeats.



## bookclerk_library::control_plane::node_status::NODE_LIVE_INTERVALS

*Constant*: `i64`

A node whose last heartbeat is older than this many intervals is not live.



## bookclerk_library::control_plane::node_status::NodeHeartbeat

*Struct*

One heartbeat's measurements, before budget and accounting are applied.

**Fields:**
- `incarnation: String` - Daemon process incarnation.
- `started_at: String` - RFC 3339 time the daemon started.
- `interval: std::time::Duration` - Promised heartbeat interval.
- `cpu_millicores: i64` - Detected CPU capacity in millicores.
- `memory_bytes: Option<i64>` - Detected memory in bytes.
- `cpu_used_millicores: Option<i64>` - Measured CPU in use, in millicores.
- `memory_available_bytes: Option<i64>` - Measured memory available to new work.
- `files_dir_free_bytes: Option<i64>` - Free bytes on the files directory's filesystem.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NodeHeartbeat`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeHeartbeat) -> bool`



## bookclerk_library::control_plane::node_status::NodeSampler

*Struct*

Samples capacity and measured use for heartbeats. Keeps the previous CPU
counters so each sample reports utilization since the last one.

**Methods:**

- `fn new(files_dir: &Path) -> Self` - A sampler for this process.
- `fn with_cgroup(files_dir: &Path, cgroup_dir: Option<PathBuf>) -> Self` - A sampler reading cgroup files from `cgroup_dir` (tests point it at
- `fn sample(self: & mut Self) -> NodeHeartbeat` - Reads capacity and use now. The first sample has no CPU use yet.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::node_status::NodeStatus

*Struct*

A node's latest heartbeat with budget, accounting, and availability.

**Fields:**
- `node_id: String` - Reporting node.
- `incarnation: String` - Daemon process incarnation.
- `started_at: String` - RFC 3339 daemon start.
- `heartbeat_at: String` - RFC 3339 heartbeat time.
- `heartbeat_ms: i64` - Heartbeat time in Unix milliseconds.
- `interval_ms: i64` - Promised interval in milliseconds.
- `cpu_millicores: i64` - Detected CPU capacity in millicores.
- `memory_bytes: Option<i64>` - Detected memory.
- `cpu_budget_millicores: i64` - CPU the scheduler may commit, in millicores.
- `memory_budget_bytes: Option<i64>` - Memory the scheduler may commit.
- `schedulable: bool` - False when an operator drained the node.
- `cpu_used_millicores: Option<i64>` - Measured CPU in use.
- `memory_available_bytes: Option<i64>` - Measured memory available.
- `files_dir_free_bytes: Option<i64>` - Free bytes on the files directory's filesystem.
- `reserved: GuestReservation` - Ceilings of the guests running here.
- `running_guests: i64` - Guests running here.
- `availability_ppm: i64` - Recent online fraction in parts per million.
- `online_since_ms: i64` - Unix milliseconds the current online stretch began.

**Methods:**

- `fn is_live(self: &Self, now_ms: i64) -> bool` - True while the last heartbeat is younger than
- `fn cpu_headroom_millicores(self: &Self) -> i64` - CPU still uncommitted under the budget, in millicores. Measured use
- `fn memory_headroom_bytes(self: &Self) -> Option<i64>` - Memory still uncommitted under the budget and actually available on

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeStatus) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NodeStatus`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::control_plane::node_status::instance_demand

*Function*

What one more guest of `instance_id` is expected to need: the largest
ceiling any node recorded for it, or `None` when no node ran it yet.

# Errors

Returns an error when a read fails.

```rust
fn instance_demand(store: &crate::store::LibraryStore, instance_id: &super::instances::PluginInstanceId) -> crate::error::Result<Option<GuestReservation>>
```



## bookclerk_library::control_plane::node_status::list_node_statuses

*Function*

Every node's latest heartbeat, by node id.

# Errors

Returns an error when the read fails.

```rust
fn list_node_statuses(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<NodeStatus>>
```



## bookclerk_library::control_plane::node_status::load_node_status

*Function*

One node's latest heartbeat, if its daemon ever ran.

# Errors

Returns an error when the read fails.

```rust
fn load_node_status(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<Option<NodeStatus>>
```



## bookclerk_library::control_plane::node_status::next_availability

*Function*

Next availability average (parts per million) after `gap_ms` since the
previous heartbeat.

The node counts as online for the whole gap when it is within the live
window, else for one interval (it stopped right after the last
heartbeat). The average decays with [`AVAILABILITY_TAU`](node_status.md#bookclerk_librarycontrol_planenode_statusavailability_tau).

```rust
fn next_availability(previous_ppm: i64, gap_ms: i64, interval_ms: i64) -> i64
```



## bookclerk_library::control_plane::node_status::node_reservations

*Function*

Sum of the ceilings of the guests running on `node_id`, and how many run.
A running guest with no recorded ceiling counts as the grant defaults.

# Errors

Returns an error when a read fails.

```rust
fn node_reservations(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<(GuestReservation, i64)>
```



## bookclerk_library::control_plane::node_status::record_deployment_reservation

*Function*

Records the ceilings of the guest a node just started for a deployment.

# Errors

Returns an error when the update fails.

```rust
fn record_deployment_reservation(store: &crate::store::LibraryStore, deployment_id: &str, node_id: &str, reservation: GuestReservation) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::node_status::record_node_heartbeat

*Function*

Records this node's heartbeat: its measurements, its budget from
`node.runtime`, the reservations of its running guests, and the updated
availability.

# Errors

Returns an error when a read or the upsert fails.

```rust
fn record_node_heartbeat(store: &crate::store::LibraryStore, node_id: &str, heartbeat: &NodeHeartbeat) -> crate::error::Result<NodeStatus>
```



