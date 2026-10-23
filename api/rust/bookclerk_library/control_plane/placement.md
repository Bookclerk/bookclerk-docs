**bookclerk_library > control_plane > placement**

# Module: control_plane::placement

## Contents

**Structs**

- [`NodeCandidate`](#nodecandidate) - One node as the planner sees it: identity, liveness, and the budget
- [`PlacementPlan`](#placementplan) - What the planner decided for one instance.
- [`PlacementSpec`](#placementspec) - What an operator asks for: how many nodes, and which may qualify.
- [`PluginPlacement`](#pluginplacement) - A stored placement: the operator's intent plus the last pass's status.
- [`ScheduleReport`](#schedulereport) - What one scheduler pass changed.

**Enums**

- [`PlacementWrite`](#placementwrite) - Outcome of [`put_plugin_placement`](placement.md#bookclerk_librarycontrol_planeplacementput_plugin_placement).
- [`Replicas`](#replicas) - How many nodes should run a placed instance.

**Functions**

- [`delete_plugin_placement`](#delete_plugin_placement) - Deletes an instance's placement and hands its deployments back to the
- [`effective_node_labels`](#effective_node_labels) - A node's operator labels plus `node=<id>` and `target=<target>`.
- [`ensure_hand_managed`](#ensure_hand_managed) - Fails when the scheduler owns `instance_id`'s deployments, so a
- [`labels_satisfy`](#labels_satisfy) - True when `labels` satisfy every constraint.
- [`list_plugin_placements`](#list_plugin_placements) - Every placement, by instance id.
- [`live_node_ids`](#live_node_ids) - Nodes whose daemon is live: its last heartbeat in `node_status` is
- [`load_plugin_placement`](#load_plugin_placement) - The placement of `instance_id`, if it has one.
- [`plan_placement`](#plan_placement) - Plans one instance.
- [`put_plugin_placement`](#put_plugin_placement) - Writes placement intent for an instance.
- [`schedule_placements`](#schedule_placements) - One scheduler pass: plans every placement against the nodes' last
- [`set_node_labels`](#set_node_labels) - Replaces a node's operator labels and wakes the scheduler.
- [`undeploy_instance`](#undeploy_instance) - Stops `instance_id` everywhere: drops its placement (handing its rows
- [`validate_constraint`](#validate_constraint) - Checks a constraint: a label, optionally prefixed with `!` to exclude it.
- [`validate_label`](#validate_label) - Checks the label grammar: `name` or `name=value`, lowercase letters,
- [`validate_node_label`](#validate_node_label) - Checks an operator label: the grammar of [`validate_label`](placement.md#bookclerk_librarycontrol_planeplacementvalidate_label), and not one

**Constants**

- [`MAX_NODE_LABELS`](#max_node_labels) - Most operator labels one node may carry.
- [`MAX_PLACEMENT_CONSTRAINTS`](#max_placement_constraints) - Most constraints one placement may carry.
- [`MAX_REPLICAS`](#max_replicas) - Largest replica count an operator may ask for.
- [`NODE_LABEL`](#node_label) - Implicit label name every node carries with its own id.
- [`SCHEDULER_LEASE`](#scheduler_lease) - Name of the cluster lease the scheduler runs under.
- [`SCHEDULER_LEASE_TTL`](#scheduler_lease_ttl) - How long a scheduler lease lasts without renewal.
- [`TARGET_LABEL`](#target_label) - Implicit label name every node carries with its artifact target.

---

## bookclerk_library::control_plane::placement::MAX_NODE_LABELS

*Constant*: `usize`

Most operator labels one node may carry.



## bookclerk_library::control_plane::placement::MAX_PLACEMENT_CONSTRAINTS

*Constant*: `usize`

Most constraints one placement may carry.



## bookclerk_library::control_plane::placement::MAX_REPLICAS

*Constant*: `u32`

Largest replica count an operator may ask for.



## bookclerk_library::control_plane::placement::NODE_LABEL

*Constant*: `&str`

Implicit label name every node carries with its own id.



## bookclerk_library::control_plane::placement::NodeCandidate

*Struct*

One node as the planner sees it: identity, liveness, and the budget
headroom its last heartbeat reported.

**Fields:**
- `node_id: String` - Node id.
- `labels: std::collections::BTreeSet<String>` - Effective labels ([`effective_node_labels`](placement.md#bookclerk_librarycontrol_planeplacementeffective_node_labels)).
- `target: String` - Artifact target; empty when unknown.
- `live: bool` - True when the node's daemon heartbeated recently.
- `schedulable: bool` - False when an operator drained the node.
- `cpu_headroom_millicores: i64` - CPU still uncommitted under the node's budget, in millicores
- `memory_headroom_bytes: Option<i64>` - Memory still uncommitted under the budget and free on the machine;
- `availability_ppm: i64` - Recent online fraction in parts per million.

**Methods:**

- `fn from_status(node: &NodeRecord, status: Option<&NodeStatus>, now_ms: i64) -> Self` - A node as of its last heartbeat (`None`: its daemon never ran, so it
- `fn fits(self: &Self, demand: GuestReservation) -> bool` - True when one more guest needing `demand` fits the memory budget.
- `fn over_budget(self: &Self) -> bool` - True when the node has committed more memory than its budget, or
- `fn score(self: &Self, demand: GuestReservation) -> i64` - How many more guests like `demand` the node has room for, in
- `fn take(self: & mut Self, demand: GuestReservation)` - Commits `demand` on this node for the rest of a pass.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeCandidate) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NodeCandidate`



## bookclerk_library::control_plane::placement::PlacementPlan

*Struct*

What the planner decided for one instance.

**Fields:**
- `add: Vec<String>` - Nodes that should gain a replica.
- `remove: Vec<String>` - Nodes whose replica should be removed.
- `placed: u32` - Replicas placed after the plan applies.
- `detail: String` - Why fewer than requested are placed; empty when satisfied.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PlacementPlan) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PlacementPlan`
- **Default**
  - `fn default() -> PlacementPlan`



## bookclerk_library::control_plane::placement::PlacementSpec

*Struct*

What an operator asks for: how many nodes, and which may qualify.

**Fields:**
- `replicas: Replicas` - Nodes that should run the instance.
- `constraints: Vec<String>` - Label constraints every chosen node must satisfy.

**Methods:**

- `fn normalized(self: &Self) -> Result<Self>` - Validated copy with constraints trimmed, sorted, and deduplicated.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PlacementSpec) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PlacementSpec`
- **Default**
  - `fn default() -> PlacementSpec`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::placement::PlacementWrite

*Enum*

Outcome of [`put_plugin_placement`](placement.md#bookclerk_librarycontrol_planeplacementput_plugin_placement).

**Variants:**
- `Applied(PluginPlacement)` - The intent is stored at the returned revision.
- `Conflict{ current_revision: i64 }` - `expected_revision` was not current (0 when no placement exists).

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PlacementWrite) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PlacementWrite`



## bookclerk_library::control_plane::placement::PluginPlacement

*Struct*

A stored placement: the operator's intent plus the last pass's status.

**Fields:**
- `plugin_instance_id: super::instances::PluginInstanceId` - Placed instance.
- `spec: PlacementSpec` - Requested replicas and constraints.
- `revision: i64` - Compare-and-swap revision of the intent.
- `updated_at: String` - RFC 3339 time of the last intent write.
- `updated_by: String` - Audit identity of the last intent writer.
- `placed: u32` - Replicas the last scheduler pass placed.
- `detail: String` - Why the last pass placed fewer than requested; empty when satisfied.
- `scheduled_at: String` - RFC 3339 time the scheduler last changed the status; empty before

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginPlacement`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginPlacement) -> bool`



## bookclerk_library::control_plane::placement::Replicas

*Enum*

How many nodes should run a placed instance.

**Variants:**
- `Count(u32)` - Exactly this many nodes (zero keeps the placement and stops every
- `All` - Every node that qualifies and has room; calls then go to the best of

**Traits:** Copy, Eq

**Trait Implementations:**

- **FromStr**
  - `fn from_str(raw: &str) -> Result<Self>`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Deserialize**
  - `fn deserialize<D>(deserializer: D) -> std::result::Result<Self, <D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Replicas`
- **Display**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Serialize**
  - `fn serialize<S>(self: &Self, serializer: S) -> std::result::Result<<S as >::Ok, <S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &Replicas) -> bool`
- **Default**
  - `fn default() -> Self`



## bookclerk_library::control_plane::placement::SCHEDULER_LEASE

*Constant*: `&str`

Name of the cluster lease the scheduler runs under.



## bookclerk_library::control_plane::placement::SCHEDULER_LEASE_TTL

*Constant*: `std::time::Duration`

How long a scheduler lease lasts without renewal.



## bookclerk_library::control_plane::placement::ScheduleReport

*Struct*

What one scheduler pass changed.

**Fields:**
- `placements: usize` - Placements considered.
- `added: usize` - Deployments set `placed`.
- `removed: usize` - Deployments set `unplaced`.
- `conflicts: usize` - Deployment writes that lost a compare-and-swap (retried next pass).
- `handed_back: usize` - Scheduler-owned rows of instances with no placement, handed back to

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScheduleReport) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ScheduleReport`
- **Default**
  - `fn default() -> ScheduleReport`



## bookclerk_library::control_plane::placement::TARGET_LABEL

*Constant*: `&str`

Implicit label name every node carries with its artifact target.



## bookclerk_library::control_plane::placement::delete_plugin_placement

*Function*

Deletes an instance's placement and hands its deployments back to the
operator: each `placed` row becomes `present` and each `unplaced` row
`absent`, so every node keeps doing what it does now. Returns false when
there was no placement.

The placement row goes first, so a scheduler pass that starts now finds
no placement and leaves these rows alone. A pass already in flight skips
an instance whose placement is gone, and a row it still manages to write
is handed back by the next pass (see [`schedule_placements`](placement.md#bookclerk_librarycontrol_planeplacementschedule_placements)).

# Errors

Returns an error when the actor is not an operator, a write fails, or a
row keeps losing its compare-and-swap (`LibraryError::Conflict`).

```rust
fn delete_plugin_placement(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, instance_id: &super::instances::PluginInstanceId) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::placement::effective_node_labels

*Function*

A node's operator labels plus `node=<id>` and `target=<target>`.

```rust
fn effective_node_labels(node: &super::identity::NodeRecord) -> std::collections::BTreeSet<String>
```



## bookclerk_library::control_plane::placement::ensure_hand_managed

*Function*

Fails when the scheduler owns `instance_id`'s deployments, so a
hand-written enable or disable is not silently undone on its next pass.

# Errors

Returns a conflict naming the placement when one exists.

```rust
fn ensure_hand_managed(store: &crate::store::LibraryStore, instance_id: &super::instances::PluginInstanceId) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::placement::labels_satisfy

*Function*

True when `labels` satisfy every constraint.

```rust
fn labels_satisfy(labels: &std::collections::BTreeSet<String>, constraints: &[String]) -> bool
```



## bookclerk_library::control_plane::placement::list_plugin_placements

*Function*

Every placement, by instance id.

# Errors

Returns an error when the read fails.

```rust
fn list_plugin_placements(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<PluginPlacement>>
```



## bookclerk_library::control_plane::placement::live_node_ids

*Function*

Nodes whose daemon is live: its last heartbeat in `node_status` is
younger than three heartbeat intervals.

# Errors

Returns an error when the read fails.

```rust
fn live_node_ids(store: &crate::store::LibraryStore) -> crate::error::Result<std::collections::BTreeSet<String>>
```



## bookclerk_library::control_plane::placement::load_plugin_placement

*Function*

The placement of `instance_id`, if it has one.

# Errors

Returns an error when the read fails.

```rust
fn load_plugin_placement(store: &crate::store::LibraryStore, instance_id: &super::instances::PluginInstanceId) -> crate::error::Result<Option<PluginPlacement>>
```



## bookclerk_library::control_plane::placement::plan_placement

*Function*

Plans one instance.

`version_targets` is the active release's artifact targets (`None` for a
plugin with no release), `demand` what one guest of the instance needs,
and `present` the nodes that already run it.

A node qualifies when it is live, schedulable, satisfies the
constraints, and has an artifact for its target. Replicas stay where
they run while their node qualifies. New replicas go to the qualifying
nodes with the most room for this demand, and only to nodes where its
memory fits the budget. A scale-down drops the replicas on the nodes
with the least room first. When a replica's node is over its memory
budget and another qualifying node has room, the replica moves there
(one move per pass, so the cluster does not thrash).

```rust
fn plan_placement(spec: &PlacementSpec, version_targets: Option<&std::collections::BTreeSet<String>>, demand: super::node_status::GuestReservation, nodes: &[NodeCandidate], present: &std::collections::BTreeSet<String>) -> PlacementPlan
```



## bookclerk_library::control_plane::placement::put_plugin_placement

*Function*

Writes placement intent for an instance.

`expected_revision` is `None` (or `Some(0)`) to create a placement and the
current revision to replace one. Every node is woken so the scheduler,
wherever it runs, applies the change.

# Errors

Returns an error when the actor is not an operator, the spec is invalid,
the instance is missing, or the write fails.

```rust
fn put_plugin_placement(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, instance_id: &super::instances::PluginInstanceId, spec: &PlacementSpec, expected_revision: Option<i64>) -> crate::error::Result<PlacementWrite>
```



## bookclerk_library::control_plane::placement::schedule_placements

*Function*

One scheduler pass: plans every placement against the nodes' last
heartbeats and writes the deployments the plans call for. Call it only
while holding the [`SCHEDULER_LEASE`](placement.md#bookclerk_librarycontrol_planeplacementscheduler_lease).

Headroom is tracked across the pass: a replica placed for one instance
is committed against its node before the next instance is planned.

# Errors

Returns an error when a read fails or a deployment write is refused.

```rust
fn schedule_placements(store: &crate::store::LibraryStore) -> crate::error::Result<ScheduleReport>
```



## bookclerk_library::control_plane::placement::set_node_labels

*Function*

Replaces a node's operator labels and wakes the scheduler.

# Errors

Returns an error when the actor is not an operator, a label is invalid or
reserved, there are too many labels, or the node is unknown.

```rust
fn set_node_labels(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, node_id: &str, labels: &[String]) -> crate::error::Result<super::identity::NodeRecord>
```



## bookclerk_library::control_plane::placement::undeploy_instance

*Function*

Stops `instance_id` everywhere: drops its placement (handing its rows
back) and sets every running deployment `absent`. Returns how many
deployments were stopped. Uninstall runs this for each instance before
it clears the active version.

# Errors

Returns an error when the actor is not an operator, a write fails, or a
row keeps losing its compare-and-swap (`LibraryError::Conflict`); the
caller must then not treat the instance as stopped.

```rust
fn undeploy_instance(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, instance_id: &super::instances::PluginInstanceId) -> crate::error::Result<usize>
```



## bookclerk_library::control_plane::placement::validate_constraint

*Function*

Checks a constraint: a label, optionally prefixed with `!` to exclude it.

# Errors

Returns an error when the label part does not fit the grammar.

```rust
fn validate_constraint(constraint: &str) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::placement::validate_label

*Function*

Checks the label grammar: `name` or `name=value`, lowercase letters,
digits, and `._/-`, the name starting with a letter or digit, at most 63
bytes.

# Errors

Returns an error when `label` does not fit the grammar.

```rust
fn validate_label(label: &str) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::placement::validate_node_label

*Function*

Checks an operator label: the grammar of [`validate_label`](placement.md#bookclerk_librarycontrol_planeplacementvalidate_label), and not one
of the implicit names every node already carries.

# Errors

Returns an error for a malformed or reserved label.

```rust
fn validate_node_label(label: &str) -> crate::error::Result<()>
```



