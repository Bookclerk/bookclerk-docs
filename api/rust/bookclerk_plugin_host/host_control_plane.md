**bookclerk_plugin_host > host_control_plane**

# Module: host_control_plane

## Contents

**Structs**

- [`HostControlPlane`](#hostcontrolplane) - Open control plane for this process.
- [`NodeEnrollment`](#nodeenrollment) - Result of [`enroll_node`]: the bootstrap half of a host control plane.
- [`PresentDeployment`](#presentdeployment) - A deployment the database wants present on this node, joined with the

**Functions**

- [`allow_credential_login`](#allow_credential_login) - `allow_credential_login` from the resolved `CONFIG`; absent means allowed.
- [`enroll_node`](#enroll_node) - Bootstraps the control plane on an open library and enrolls every local
- [`open_host_control_plane`](#open_host_control_plane) - Opens the library database and the control plane for `config`.
- [`open_host_control_plane_with_store`](#open_host_control_plane_with_store) - [`open_host_control_plane`](host_control_plane.md#bookclerk_plugin_hosthost_control_planeopen_host_control_plane) on a library that is already open.
- [`refresh_present`](#refresh_present) - Loads an already-open host control plane's deployments again, for callers

---

## bookclerk_plugin_host::host_control_plane::HostControlPlane

*Struct*

Open control plane for this process.

**Fields:**
- `config: bookclerk_config::Config` - Bootstrap config with the database-backed domains overlaid.
- `database_registry: crate::host::DatabaseRegistry` - Database adapters discovered for bootstrap.
- `store: bookclerk_library::LibraryStore` - Open library.
- `session: bookclerk_library::control_plane::ControlPlaneSession` - Node identity, cluster id, and core documents.
- `enrolled: Vec<crate::enrollment::EnrolledPlugin>` - Plugins enrolled on this pass.
- `present: Vec<PresentDeployment>` - Deployments desired present on this node, in deployment-id order.
- `default_instances: std::collections::HashMap<String, String>` - Default instance id per plugin key, for routing a bare key.

**Methods:**

- `fn node_id(self: &Self) -> &str` - This node's id.
- `fn load_sources(self: &Self, services: &SessionServices) -> Result<SourceRegistry>` - Spawns every present storefront deployment and registers it under its
- `fn load_integrations(self: &Self, services: &SessionServices) -> Result<IntegrationRegistry>` - Spawns every present integration deployment.
- `fn load_destinations(self: &Self) -> Result<DestinationRegistry>` - Spawns every present storage deployment.



## bookclerk_plugin_host::host_control_plane::NodeEnrollment

*Struct*

Result of [`enroll_node`]: the bootstrap half of a host control plane.

**Fields:**
- `config: bookclerk_config::Config` - Bootstrap config with the database-backed domains overlaid.
- `session: bookclerk_library::control_plane::ControlPlaneSession` - Node identity, cluster id, and core documents.
- `enrolled: Vec<crate::enrollment::EnrolledPlugin>` - Plugins enrolled on this pass.



## bookclerk_plugin_host::host_control_plane::PresentDeployment

*Struct*

A deployment the database wants present on this node, joined with the
instance row and the local install it will run.

**Fields:**
- `deployment: bookclerk_library::control_plane::PluginDeployment` - Desired row.
- `instance: bookclerk_library::control_plane::PluginInstance` - Instance the row runs.
- `plugin: Option<crate::discover::DiscoveredPlugin>` - Install discovered on this node, when there is one.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> PresentDeployment`



## bookclerk_plugin_host::host_control_plane::allow_credential_login

*Function*

`allow_credential_login` from the resolved `CONFIG`; absent means allowed.

```rust
fn allow_credential_login(granted_config: &serde_json::Value) -> bool
```



## bookclerk_plugin_host::host_control_plane::enroll_node

*Function*

Bootstraps the control plane on an open library and enrolls every local
plugin, without loading any guest.

The daemon's config reload uses this: guests stay with the deployment
reconciler, which picks up the enrolled deployments on its next pass.

# Errors

Returns an error when the secret root or node identity cannot be
established, or enrollment fails as a whole.

```rust
fn enroll_node(config: &bookclerk_config::Config, store: &bookclerk_library::LibraryStore) -> crate::Result<NodeEnrollment>
```



## bookclerk_plugin_host::host_control_plane::open_host_control_plane

*Function*

Opens the library database and the control plane for `config`.

# Errors

Returns an error when the database cannot be opened, the secret root or
node identity cannot be established, or enrollment fails as a whole.

```rust
fn open_host_control_plane(config: &bookclerk_config::Config) -> crate::Result<HostControlPlane>
```



## bookclerk_plugin_host::host_control_plane::open_host_control_plane_with_store

*Function*

[`open_host_control_plane`](host_control_plane.md#bookclerk_plugin_hosthost_control_planeopen_host_control_plane) on a library that is already open.

The daemon's config reload uses this so it does not reopen the database.

# Errors

Returns an error when bootstrap or enrollment fails.

```rust
fn open_host_control_plane_with_store(config: &bookclerk_config::Config, database_registry: crate::host::DatabaseRegistry, store: bookclerk_library::LibraryStore) -> crate::Result<HostControlPlane>
```



## bookclerk_plugin_host::host_control_plane::refresh_present

*Function*

Loads an already-open host control plane's deployments again, for callers
that changed desired state and want a fresh view without re-enrolling.

# Errors

Returns an error when the deployment table cannot be read.

```rust
fn refresh_present(store: &bookclerk_library::LibraryStore, node_id: &str, enrolled: &[crate::enrollment::EnrolledPlugin]) -> crate::Result<(Vec<PresentDeployment>, std::collections::HashMap<String, String>)>
```



