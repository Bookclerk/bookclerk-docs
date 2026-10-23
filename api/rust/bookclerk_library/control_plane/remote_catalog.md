**bookclerk_library > control_plane > remote_catalog**

# Module: control_plane::remote_catalog

## Contents

**Structs**

- [`RemoteDeploymentEntry`](#remotedeploymententry) - One deployment a Host should run, with what it runs.
- [`RemoteInstance`](#remoteinstance) - A deployed instance a Host asks about, with the node's trust.
- [`RemoteObservation`](#remoteobservation) - One observation a Host reports.

**Functions**

- [`artifact_served_to_node`](#artifact_served_to_node) - True when the archive `archive_sha256` is stored in the library and is
- [`record_remote_observation`](#record_remote_observation) - Records a Host's observation of one of its deployments. Answers `false`
- [`remote_deployments_for_node`](#remote_deployments_for_node) - The deployments that target `node_id` and should run, in deployment-id
- [`remote_instance_for_node`](#remote_instance_for_node) - `instance_id` when it has a deployment on `node_id` that should run and

---

## bookclerk_library::control_plane::remote_catalog::RemoteDeploymentEntry

*Struct*

One deployment a Host should run, with what it runs.

**Fields:**
- `deployment: super::deployments::PluginDeployment` - The desired row (targets the Host and should run).
- `plugin_key: String` - The instance's plugin key.
- `default_instance: bool` - True when the instance is its plugin's default instance, which
- `version: Option<super::versions::PluginVersion>` - The plugin's active version; `None` when nothing is installed.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteDeploymentEntry) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> RemoteDeploymentEntry`



## bookclerk_library::control_plane::remote_catalog::RemoteInstance

*Struct*

A deployed instance a Host asks about, with the node's trust.

**Fields:**
- `deployment: super::deployments::PluginDeployment` - The deployment on the Host.
- `plugin_key: String` - The instance's plugin key.
- `version: Option<super::versions::PluginVersion>` - The plugin's active version.
- `trust: super::trust::NodeTrust` - What the node is trusted with now. Instance secrets belong to the

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RemoteInstance`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteInstance) -> bool`



## bookclerk_library::control_plane::remote_catalog::RemoteObservation

*Struct*

One observation a Host reports.

**Generic Parameters:**
- 'a

**Fields:**
- `deployment_id: &'a str` - Deployment id.
- `status: super::deployments::DeploymentStatus` - Observed status.
- `detail: &'a str` - Detail (bounded on write).
- `applied_config_revision: Option<i64>` - Config revision the Host applied, if any.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteObservation<'a>) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> RemoteObservation<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::remote_catalog::artifact_served_to_node

*Function*

True when the archive `archive_sha256` is stored in the library and is
an artifact of the active version of a plugin deployed to `node_id`.

# Errors

Returns an error when a read fails.

```rust
fn artifact_served_to_node(store: &crate::store::LibraryStore, node_id: &str, archive_sha256: &str) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::remote_catalog::record_remote_observation

*Function*

Records a Host's observation of one of its deployments. Answers `false`
(and writes nothing) for a deployment that does not target the node, a
status a Host may not report, or a node trusted with nothing.

# Errors

Returns an error when a read or the write fails.

```rust
fn record_remote_observation(store: &crate::store::LibraryStore, node_id: &str, incarnation: &str, observation: &RemoteObservation) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::remote_catalog::remote_deployments_for_node

*Function*

The deployments that target `node_id` and should run, in deployment-id
order. Empty for a node trusted with nothing.

# Errors

Returns an error when a read fails.

```rust
fn remote_deployments_for_node(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<Vec<RemoteDeploymentEntry>>
```



## bookclerk_library::control_plane::remote_catalog::remote_instance_for_node

*Function*

`instance_id` when it has a deployment on `node_id` that should run and
the node is trusted with something; `None` otherwise (an unknown
instance and one deployed elsewhere read the same).

# Errors

Returns an error when a read fails.

```rust
fn remote_instance_for_node(store: &crate::store::LibraryStore, node_id: &str, instance_id: &str) -> crate::error::Result<Option<RemoteInstance>>
```



