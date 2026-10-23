**bookclerk_library > control_plane > deployments**

# Module: control_plane::deployments

## Contents

**Structs**

- [`DeploymentObservation`](#deploymentobservation) - One node's observation of a deployment.
- [`PluginDeployment`](#plugindeployment) - One desired deployment row.

**Enums**

- [`DeploymentReplace`](#deploymentreplace) - Outcome of [`replace_plugin_deployment`](deployments.md#bookclerk_librarycontrol_planedeploymentsreplace_plugin_deployment).
- [`DeploymentStatus`](#deploymentstatus) - Observed deployment status.
- [`DesiredState`](#desiredstate) - What a deployment row asks its node to do, and who decided it.

**Functions**

- [`bounded_observation_detail`](#bounded_observation_detail) - Truncates detail to 512 bytes and drops ASCII control characters.
- [`delete_plugin_deployment`](#delete_plugin_deployment) - Deletes an absent deployment and its observations.
- [`ensure_plugin_deployment`](#ensure_plugin_deployment) - Inserts `(instance, node, desired)` when that pair has no row.
- [`instance_has_deployment`](#instance_has_deployment) - True when any deployment row names `instance_id`.
- [`list_all_deployments`](#list_all_deployments) - Every deployment row in the cluster.
- [`list_deployments_for_instance`](#list_deployments_for_instance) - Every deployment row for `instance_id`, on any node.
- [`list_deployments_for_node`](#list_deployments_for_node) - Every deployment row targeting `node_id`, present or absent.
- [`list_observations_for_deployment`](#list_observations_for_deployment) - Every observation row for `deployment_id`, from any node.
- [`list_present_deployments_for_node`](#list_present_deployments_for_node) - Present deployments whose `node_id` is `node_id`.
- [`load_deployment`](#load_deployment) - Loads one deployment.
- [`load_deployment_for_node`](#load_deployment_for_node) - The deployment of `instance_id` on `node_id`, if any.
- [`load_observation`](#load_observation) - Loads the observation for one deployment on one node.
- [`present_plugin_keys_for_node`](#present_plugin_keys_for_node) - Canonical plugin keys with a present deployment on `node_id`.
- [`replace_plugin_deployment`](#replace_plugin_deployment) - Replaces desired state when `expected_revision` is current.
- [`upsert_observation`](#upsert_observation) - Upserts one local observation.

**Constants**

- [`MAX_OBSERVATION_DETAIL_BYTES`](#max_observation_detail_bytes) - Maximum UTF-8 size of an observation detail string.

---

## bookclerk_library::control_plane::deployments::DeploymentObservation

*Struct*

One node's observation of a deployment.

**Fields:**
- `deployment_id: String` - Deployment id.
- `node_id: String` - Observing node.
- `incarnation: String` - Process incarnation.
- `status: DeploymentStatus` - Last status.
- `detail: String` - Bounded detail. Empty when healthy.
- `applied_config_revision: Option<i64>` - Config revision applied by spawn, if a spawn has happened.
- `observed_at: String` - RFC 3339 observation time.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &DeploymentObservation) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DeploymentObservation`



## bookclerk_library::control_plane::deployments::DeploymentReplace

*Enum*

Outcome of [`replace_plugin_deployment`](deployments.md#bookclerk_librarycontrol_planedeploymentsreplace_plugin_deployment).

**Variants:**
- `Applied(PluginDeployment)` - The desired row is the committed body.
- `Conflict{ current_revision: i64 }` - `expected_revision` was not current. No observation row was written.
- `Replayed{ revision: i64 }` - This operation id already committed at `revision`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DeploymentReplace`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &DeploymentReplace) -> bool`



## bookclerk_library::control_plane::deployments::DeploymentStatus

*Enum*

Observed deployment status.

**Variants:**
- `Materialized` - The node has the plugin's tree (materialized from the installed
- `Running` - Session spawn returned. Health is not confirmed on this tick.
- `Healthy` - Health succeeded with the applied config revision.
- `Error` - Install, grant, config, secrets, spawn, or health failed.
- `Retired` - Desired state is `absent`; the node stopped the guest.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Stored text.
- `fn parse(raw: &str) -> Result<Self>` - Parses stored text.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &DeploymentStatus) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DeploymentStatus`



## bookclerk_library::control_plane::deployments::DesiredState

*Enum*

What a deployment row asks its node to do, and who decided it.

Operators write `DesiredState::Present` and `DesiredState::Absent`
(`plugins deploy|undeploy`, the Settings switch). The placement scheduler
writes `DesiredState::Placed` and `DesiredState::Unplaced` for an
instance it owns. A node runs the instance for `present` and `placed`
and retires it for `absent` and `unplaced`; the split only records whose
decision the row carries, so "the operator turned this off" never reads
like "the scheduler put the replica elsewhere".

**Variants:**
- `Present` - An operator enabled the instance on this node.
- `Absent` - An operator disabled the instance on this node.
- `Placed` - The scheduler placed a replica on this node.
- `Unplaced` - The scheduler holds no replica on this node (scaled away, node no

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Stored text.
- `fn parse(raw: &str) -> Result<Self>` - Parses stored or API text.
- `fn runs(self: Self) -> bool` - True when the node should run the instance.
- `fn is_scheduled(self: Self) -> bool` - True for the scheduler's states.
- `fn operator(enabled: bool) -> Self` - The operator state for an enable (`true`) or disable switch.
- `fn scheduled(placed: bool) -> Self` - The scheduler state for a replica kept (`true`) or not.
- `fn handed_back(self: Self) -> Self` - The operator state that keeps what this row does today: `placed`

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &DesiredState) -> bool`
- **Display**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Ord**
  - `fn cmp(self: &Self, other: &DesiredState) -> $crate::cmp::Ordering`
- **Clone**
  - `fn clone(self: &Self) -> DesiredState`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &DesiredState) -> $crate::option::Option<$crate::cmp::Ordering>`



## bookclerk_library::control_plane::deployments::MAX_OBSERVATION_DETAIL_BYTES

*Constant*: `usize`

Maximum UTF-8 size of an observation detail string.



## bookclerk_library::control_plane::deployments::PluginDeployment

*Struct*

One desired deployment row.

**Fields:**
- `deployment_id: String` - Deployment id.
- `plugin_instance_id: super::instances::PluginInstanceId` - Instance to run.
- `node_id: String` - Target node.
- `desired: DesiredState` - What the node should do, and whose decision it is.
- `revision: i64` - Compare-and-swap revision.
- `updated_at: String` - RFC 3339 commit time.
- `updated_by: String` - Audit identity of the last desired-state writer.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PluginDeployment) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PluginDeployment`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::deployments::bounded_observation_detail

*Function*

Truncates detail to 512 bytes and drops ASCII control characters.

```rust
fn bounded_observation_detail(detail: &str) -> String
```



## bookclerk_library::control_plane::deployments::delete_plugin_deployment

*Function*

Deletes an absent deployment and its observations.

# Errors

Returns an error when the actor may not write, the row is still present,
or the delete fails.

```rust
fn delete_plugin_deployment(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, deployment_id: &str) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::deployments::ensure_plugin_deployment

*Function*

Inserts `(instance, node, desired)` when that pair has no row.

A second call returns the original row and does not bump the revision.

# Errors

Returns an error when the actor may not write `desired`, the instance is
missing, or the insert fails.

```rust
fn ensure_plugin_deployment(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, instance_id: &super::instances::PluginInstanceId, node_id: &str, desired: DesiredState) -> crate::error::Result<PluginDeployment>
```



## bookclerk_library::control_plane::deployments::instance_has_deployment

*Function*

True when any deployment row names `instance_id`.

Key-based spawn must not pick this instance. The deployment reconciler
passes the instance id and the already resolved bindings.

# Errors

Returns an error when the read fails.

```rust
fn instance_has_deployment(store: &crate::store::LibraryStore, instance_id: &super::instances::PluginInstanceId) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::deployments::list_all_deployments

*Function*

Every deployment row in the cluster.

# Errors

Returns an error when the read fails.

```rust
fn list_all_deployments(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<PluginDeployment>>
```



## bookclerk_library::control_plane::deployments::list_deployments_for_instance

*Function*

Every deployment row for `instance_id`, on any node.

# Errors

Returns an error when the read fails.

```rust
fn list_deployments_for_instance(store: &crate::store::LibraryStore, instance_id: &super::instances::PluginInstanceId) -> crate::error::Result<Vec<PluginDeployment>>
```



## bookclerk_library::control_plane::deployments::list_deployments_for_node

*Function*

Every deployment row targeting `node_id`, present or absent.

# Errors

Returns an error when the read fails.

```rust
fn list_deployments_for_node(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<Vec<PluginDeployment>>
```



## bookclerk_library::control_plane::deployments::list_observations_for_deployment

*Function*

Every observation row for `deployment_id`, from any node.

# Errors

Returns an error when the read fails.

```rust
fn list_observations_for_deployment(store: &crate::store::LibraryStore, deployment_id: &str) -> crate::error::Result<Vec<DeploymentObservation>>
```



## bookclerk_library::control_plane::deployments::list_present_deployments_for_node

*Function*

Present deployments whose `node_id` is `node_id`.

Rows for any other node are not returned.

# Errors

Returns an error when the read fails.

```rust
fn list_present_deployments_for_node(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<Vec<PluginDeployment>>
```



## bookclerk_library::control_plane::deployments::load_deployment

*Function*

Loads one deployment.

# Errors

Returns an error when the read fails.

```rust
fn load_deployment(store: &crate::store::LibraryStore, deployment_id: &str) -> crate::error::Result<Option<PluginDeployment>>
```



## bookclerk_library::control_plane::deployments::load_deployment_for_node

*Function*

The deployment of `instance_id` on `node_id`, if any.

# Errors

Returns an error when the read fails.

```rust
fn load_deployment_for_node(store: &crate::store::LibraryStore, instance_id: &super::instances::PluginInstanceId, node_id: &str) -> crate::error::Result<Option<PluginDeployment>>
```



## bookclerk_library::control_plane::deployments::load_observation

*Function*

Loads the observation for one deployment on one node.

# Errors

Returns an error when the read fails.

```rust
fn load_observation(store: &crate::store::LibraryStore, deployment_id: &str, node_id: &str) -> crate::error::Result<Option<DeploymentObservation>>
```



## bookclerk_library::control_plane::deployments::present_plugin_keys_for_node

*Function*

Canonical plugin keys with a present deployment on `node_id`.

# Errors

Returns an error when the read fails.

```rust
fn present_plugin_keys_for_node(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<Vec<String>>
```



## bookclerk_library::control_plane::deployments::replace_plugin_deployment

*Function*

Replaces desired state when `expected_revision` is current.

The batch carries a revision predicate and a `replacePluginDeployment`
receipt. It does not update observation rows.

The scheduler writes only its own states, with one exception: it may
hand a row back (`placed` to `present`, `unplaced` to `absent`) when the
row's placement is gone, finishing a deletion a racing pass outran.

# Errors

Returns an error when the actor may not write `desired`, the deployment
is missing, or the operation id was already used for a different body.

```rust
fn replace_plugin_deployment(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, deployment_id: &str, expected_revision: i64, desired: DesiredState, operation_id: &str) -> crate::error::Result<DeploymentReplace>
```



## bookclerk_library::control_plane::deployments::upsert_observation

*Function*

Upserts one local observation.

A failed upsert does not roll back an install. The caller retries on the next tick.

# Errors

Returns an error when the write fails.

```rust
fn upsert_observation(store: &crate::store::LibraryStore, observation: &DeploymentObservation) -> crate::error::Result<()>
```



