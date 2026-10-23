**bookclerkd > config_authority**

# Module: config_authority

## Contents

**Structs**

- [`PutEventsDomainRequest`](#puteventsdomainrequest) - JSON body for `PUT /api/config/domains/core.events`.
- [`PutOutputDomainRequest`](#putoutputdomainrequest) - JSON body for `PUT /api/config/domains/core.output`.

**Functions**

- [`commit_events_settings`](#commit_events_settings) - Applies `events.*` settings keys through compare-and-swap.
- [`commit_output_settings`](#commit_output_settings) - Applies `output.*` settings keys to `core.output` through compare-and-swap.
- [`delete_owner_node`](#delete_owner_node) - `DELETE /api/config/nodes/{node_id}`: remove an owner node (a clerk, or
- [`delete_plugin_instance_route`](#delete_plugin_instance_route) - `DELETE /api/config/plugin-instances/{id}`: removes an instance that has
- [`delete_plugin_placement_route`](#delete_plugin_placement_route) - `DELETE /api/config/plugin-instances/{id}/placement`: return the
- [`get_clerks`](#get_clerks) - `GET /api/config/clerks`: the owner nodes (this one and its clerks) for
- [`get_cluster_status`](#get_cluster_status) - `GET /api/cluster/status`: cluster-wide health every node administrator
- [`get_events_domain`](#get_events_domain) - `GET /api/config/domains/core.events`.
- [`get_nodes`](#get_nodes) - `GET /api/config/nodes`: every node for an operator; the caller's own
- [`get_output_domain`](#get_output_domain) - `GET /api/config/domains/core.output`.
- [`get_plugin_instance_config`](#get_plugin_instance_config) - `GET /api/config/plugin-instances/{id}/config`.
- [`get_plugin_instances`](#get_plugin_instances) - `GET /api/config/plugin-instances`.
- [`post_plugin_instance`](#post_plugin_instance) - `POST /api/config/plugin-instances`: a second (non-default) instance of
- [`put_events_domain`](#put_events_domain) - `PUT /api/config/domains/core.events`.
- [`put_node_labels`](#put_node_labels) - `PUT /api/config/nodes/{node_id}/labels`: replace a node's placement
- [`put_node_owner`](#put_node_owner) - `PUT /api/config/nodes/{node_id}/owner`: assign or clear a node's owner.
- [`put_node_runtime`](#put_node_runtime) - `PUT /api/config/nodes/{node_id}/runtime`: edit a node's label, budgets,
- [`put_output_domain`](#put_output_domain) - `PUT /api/config/domains/core.output`.
- [`put_plugin_deployment`](#put_plugin_deployment) - `PUT /api/config/plugin-instances/{id}/deployments/{node_id}`: the
- [`put_plugin_instance_config`](#put_plugin_instance_config) - `PUT /api/config/plugin-instances/{id}/config`.
- [`put_plugin_placement_route`](#put_plugin_placement_route) - `PUT /api/config/plugin-instances/{id}/placement`: hand the instance's
- [`reconcile_events`](#reconcile_events) - Applies a newer cluster events document onto the live config.
- [`reconcile_output`](#reconcile_output) - Applies a newer cluster output document onto the live config.
- [`spawn_config_reconciler`](#spawn_config_reconciler) - Re-reads `core.events` and `core.output` on each `configuration.changed`
- [`spawn_deployment_reconciler`](#spawn_deployment_reconciler) - Re-reads local plugin deployments on each `deployment.changed` or

---

## bookclerkd::config_authority::PutEventsDomainRequest

*Struct*

JSON body for `PUT /api/config/domains/core.events`.

**Fields:**
- `expected_revision: i64` - Revision the caller read. A mismatch is a conflict.
- `retention_days: u64` - Replacement `[events]` document.
- `dead_letter_retention_days: u64` - Replacement dead-letter retention.
- `concurrency: u32` - Replacement concurrency cap.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::config_authority::PutOutputDomainRequest

*Struct*

JSON body for `PUT /api/config/domains/core.output`.

**Fields:**
- `expected_revision: i64` - Revision the caller read. A mismatch is a conflict.
- `document: bookclerk_library::control_plane::OutputSettingsV1` - Complete replacement document (schema version 1).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::config_authority::commit_events_settings

*Function*

Applies `events.*` settings keys through compare-and-swap.

# Errors

Returns 400 for invalid values and 409 when `expected` revision lost.

```rust
fn commit_events_settings(state: &crate::api::AppState, updates: &[(String, String)]) -> Result<(), axum::response::Response>
```



## bookclerkd::config_authority::commit_output_settings

*Function*

Applies `output.*` settings keys to `core.output` through compare-and-swap.

Every key is applied to one copy of the current document, which is then
validated and written as a single revision.

`expected_revision` is the revision the caller read. When it is given and
the stored document has moved on, nothing is written and the call is a
conflict, so a stale editor cannot overwrite a concurrent change.

# Errors

Returns 400 for unknown keys or invalid values and 409 when
`expected_revision` is stale or the revision lost to a concurrent write.

```rust
fn commit_output_settings(state: &crate::api::AppState, updates: &[(String, String)], expected_revision: Option<i64>) -> Result<(), axum::response::Response>
```



## bookclerkd::config_authority::delete_owner_node

*Function*

`DELETE /api/config/nodes/{node_id}`: remove an owner node (a clerk, or
any node sharing this database) that is no longer running from the
library's node list. The operator or an Owner may remove any; another
user only one they own. A running node, this node, or a branch is
refused (a branch is revoked instead).

```rust
fn delete_owner_node(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::NodeAdminCaller>, __arg2: axum::extract::Path<String>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::delete_plugin_instance_route

*Function*

`DELETE /api/config/plugin-instances/{id}`: removes an instance that has
no deployment left.

```rust
fn delete_plugin_instance_route(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::delete_plugin_placement_route

*Function*

`DELETE /api/config/plugin-instances/{id}/placement`: return the
instance to hand-managed deployments; its deployments stay as they
are.

```rust
fn delete_plugin_placement_route(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::get_clerks

*Function*

`GET /api/config/clerks`: the owner nodes (this one and its clerks) for
Settings → Clerks, shaped like `GET /api/config/nodes`. The operator or
an Owner sees every one, as `DELETE /api/config/nodes/{node_id}` lets
them remove any; another user only the ones they own, which the nodes
listing already shows them. Branches are not listed.

```rust
fn get_clerks(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::NodeAdminCaller>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::get_cluster_status

*Function*

`GET /api/cluster/status`: cluster-wide health every node administrator
may see (node counts, committed capacity, active jobs, delivery backlog),
plus detail for the nodes the caller may see.

```rust
fn get_cluster_status(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::NodeAdminCaller>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::get_events_domain

*Function*

`GET /api/config/domains/core.events`.

```rust
fn get_events_domain(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::get_nodes

*Function*

`GET /api/config/nodes`: every node for an operator; the caller's own
nodes for a node owner.

```rust
fn get_nodes(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::NodeAdminCaller>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::get_output_domain

*Function*

`GET /api/config/domains/core.output`.

```rust
fn get_output_domain(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::get_plugin_instance_config

*Function*

`GET /api/config/plugin-instances/{id}/config`.

```rust
fn get_plugin_instance_config(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::get_plugin_instances

*Function*

`GET /api/config/plugin-instances`.

```rust
fn get_plugin_instances(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::post_plugin_instance

*Function*

`POST /api/config/plugin-instances`: a second (non-default) instance of
a plugin, with an empty document so schema defaults apply. With
`node_id`, the instance is local to that node.

```rust
fn post_plugin_instance(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Json<CreatePluginInstance>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::put_events_domain

*Function*

`PUT /api/config/domains/core.events`.

```rust
fn put_events_domain(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Json<PutEventsDomainRequest>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::put_node_labels

*Function*

`PUT /api/config/nodes/{node_id}/labels`: replace a node's placement
labels. The scheduler re-plans on the wakeup this publishes.

```rust
fn put_node_labels(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>, __arg2: axum::Json<PutNodeLabels>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::put_node_owner

*Function*

`PUT /api/config/nodes/{node_id}/owner`: assign or clear a node's owner.
The owner may then edit that node's runtime settings without operator
authority.

```rust
fn put_node_owner(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>, __arg2: axum::Json<PutNodeOwner>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::put_node_runtime

*Function*

`PUT /api/config/nodes/{node_id}/runtime`: edit a node's label, budgets,
or drain switch. Applies at that node's next heartbeat. A node owner may
edit only the nodes it owns.

```rust
fn put_node_runtime(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::NodeAdminCaller>, __arg2: axum::extract::Path<String>, __arg3: axum::Json<PutNodeRuntime>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::put_output_domain

*Function*

`PUT /api/config/domains/core.output`.

```rust
fn put_output_domain(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Json<PutOutputDomainRequest>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::put_plugin_deployment

*Function*

`PUT /api/config/plugin-instances/{id}/deployments/{node_id}`: the
Deployed switch. Creates the row when the node has none.

```rust
fn put_plugin_deployment(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<(String, String)>, __arg2: axum::Json<PutPluginDeployment>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::put_plugin_instance_config

*Function*

`PUT /api/config/plugin-instances/{id}/config`.

Operator principal only. The route is mounted on the operator router.
The document is validated against the plugin's pinned schema, so this
node does not need the guest installed.

```rust
fn put_plugin_instance_config(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>, __arg2: axum::Json<PutPluginInstanceConfig>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::put_plugin_placement_route

*Function*

`PUT /api/config/plugin-instances/{id}/placement`: hand the instance's
deployments to the scheduler with a replica count and constraints.

```rust
fn put_plugin_placement_route(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>, __arg2: axum::Json<PutPluginPlacement>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::config_authority::reconcile_events

*Function*

Applies a newer cluster events document onto the live config.

The read and the publication are separate so a newer revision can commit
before a stale read is released. The stale publication then leaves the
effective body and revision unchanged.

# Errors

Returns an error when the document cannot be read. The in-memory config is
left unchanged, so a bad revision is not announced as applied.

```rust
fn reconcile_events(state: &crate::api::AppState) -> anyhow::Result<()>
```



## bookclerkd::config_authority::reconcile_output

*Function*

Applies a newer cluster output document onto the live config.

# Errors

Returns an error when the document cannot be read. The in-memory config is
left unchanged.

```rust
fn reconcile_output(state: &crate::api::AppState) -> anyhow::Result<()>
```



## bookclerkd::config_authority::spawn_config_reconciler

*Function*

Re-reads `core.events` and `core.output` on each `configuration.changed`
delivery on this node's queue, plus a slow backstop pass, until the
process stops.

```rust
fn spawn_config_reconciler(state: std::sync::Arc<crate::api::AppState>)
```



## bookclerkd::config_authority::spawn_deployment_reconciler

*Function*

Re-reads local plugin deployments on each `deployment.changed` or
`configuration.changed` delivery on this node's queue, plus a slow
backstop pass, until the process stops.

```rust
fn spawn_deployment_reconciler(state: std::sync::Arc<crate::api::AppState>)
```



