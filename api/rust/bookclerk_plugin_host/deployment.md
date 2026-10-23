**bookclerk_plugin_host > deployment**

# Module: deployment

## Contents

**Structs**

- [`AuthorizedLocalPackages`](#authorizedlocalpackages) - Operator-placed archives keyed by canonical plugin key.
- [`DeploymentSpawn`](#deploymentspawn) - Spawn request after config resolution.
- [`LiveDeploymentRuntime`](#livedeploymentruntime) - Production runtime: spawn through the existing host adapters and register
- [`LocalPackage`](#localpackage) - Local archive the reconciler may install when the ledger has no row.

**Enums**

- [`SpawnHealth`](#spawnhealth) - Result of spawn plus the health RPC.

**Functions**

- [`load_authorized_local_packages`](#load_authorized_local_packages) - Loads operator-placed archives under `$FILES_DIR/plugin-packages/<name>/`.
- [`reconcile_local_deployments`](#reconcile_local_deployments) - Reconciles every present deployment for `node_id`.

**Traits**

- [`DeploymentRuntime`](#deploymentruntime) - Install health and spawn for one deployment row.

**Constants**

- [`AUTHORIZED_PACKAGE_DIR`](#authorized_package_dir) - Directory under the files dir that holds operator-placed package archives.

---

## bookclerk_plugin_host::deployment::AUTHORIZED_PACKAGE_DIR

*Constant*: `&str`

Directory under the files dir that holds operator-placed package archives.



## bookclerk_plugin_host::deployment::AuthorizedLocalPackages

*Struct*

Operator-placed archives keyed by canonical plugin key.

A key that two packages both resolve to is removed from the map and
recorded in `Self::is_conflicted`. Neither package is installed.

**Methods:**

- `fn get(self: &Self, key: &str) -> Option<&LocalPackage>` - Package for `key`, when exactly one authorized archive resolved to it.
- `fn is_conflicted(self: &Self, key: &str) -> bool` - True when two archives resolved to `key` and neither was kept.
- `fn is_empty(self: &Self) -> bool` - True when no archive was loaded and no key collided.
- `fn keys(self: &Self) -> std::collections::hash_map::Keys<String, LocalPackage>` - Canonical keys that have exactly one package.
- `fn insert(self: & mut Self, key: String, package: LocalPackage)` - Inserts `package` under `key`, clearing a previous conflict on that key.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, formatter: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Default**
  - `fn default() -> AuthorizedLocalPackages`
- **Clone**
  - `fn clone(self: &Self) -> AuthorizedLocalPackages`



## bookclerk_plugin_host::deployment::DeploymentRuntime

*Trait*

Install health and spawn for one deployment row.

**Methods:**

- `spawn_and_health`: Spawn the guest and run health. Health is the deployment observation;
- `guest_still_running`: True when the guest spawned for `plugin_instance_id` is still running.
- `guest_root`: Tree the running guest for `plugin_instance_id` was spawned from.
- `retire_instance`: Stops a guest this process is running for `plugin_instance_id`.
- `tracked_instance_ids`: Instance ids this process is still tracking.



## bookclerk_plugin_host::deployment::DeploymentSpawn

*Struct*

Spawn request after config resolution.

**Fields:**
- `plugin_instance_id: String` - Instance this deployment runs. Not derived from the plugin key.
- `plugin_key: String` - Canonical plugin key.
- `plugin_root: Option<std::path::PathBuf>` - Installed tree, when discovery or install produced one.
- `config: bookclerk_plugin_sdk::ExtensibleConfig` - Granted `CONFIG` payload.
- `secrets: bookclerk_plugin_sdk::ExtensibleConfig` - Granted `SECRETS` payload.
- `config_revision: i64` - Document revision the spawn applies.
- `installed: Option<std::sync::Arc<bookclerk_library::control_plane::PluginVersion>>` - Installed version the tree was materialized from; `None` for a

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> DeploymentSpawn`



## bookclerk_plugin_host::deployment::LiveDeploymentRuntime

*Struct*

Production runtime: spawn through the existing host adapters and register
the session on the live registry.

**Fields:**
- `config: std::sync::Arc<tokio::sync::RwLock<bookclerk_config::Config>>` - Live daemon config.
- `store: std::sync::Arc<tokio::sync::RwLock<bookclerk_library::LibraryStore>>` - Open library, used for instance bindings.
- `sources: std::sync::Arc<tokio::sync::RwLock<bookclerk_source::SourceRegistry>>` - Storefront sessions the reconciler owns.
- `integrations: std::sync::Arc<tokio::sync::RwLock<bookclerk_integrations::IntegrationRegistry>>` - Integration sessions the reconciler owns.
- `destinations: std::sync::Arc<tokio::sync::RwLock<crate::DestinationRegistry>>` - Destination sessions the reconciler owns.

**Methods:**

- `fn new(config: Arc<tokio::sync::RwLock<Config>>, store: Arc<tokio::sync::RwLock<LibraryStore>>, sources: Arc<tokio::sync::RwLock<bookclerk_source::SourceRegistry>>, integrations: Arc<tokio::sync::RwLock<bookclerk_integrations::IntegrationRegistry>>, destinations: Arc<tokio::sync::RwLock<crate::DestinationRegistry>>) -> Self` - Runtime bound to the daemon's live registries.
- `fn reconcile_lock(self: &Self) -> &tokio::sync::Mutex<()>` - Lock a caller holds for the duration of one reconcile pass.
- `fn set_integration_context(self: &Self, ctx: bookclerk_integrations::IntegrationContext)` - Stores the daemon lifecycle context used when a deployment starts an integration.
- `fn retire_plugin_instance(self: &Self, plugin_instance_id: &str)` - Stops every registry entry for `plugin_instance_id` and drops the guest.
- `fn forget_guest(self: &Self, plugin_instance_id: &str)` - Drops the tracked guest for `plugin_instance_id` without taking the source registry.
- `fn opened_config_json(self: &Self, plugin_instance_id: &str) -> Option<serde_json::Value>` - `CONFIG` JSON the live `open` call delivered for `plugin_instance_id`.
- `fn tracked_guest_pid(self: &Self, plugin_instance_id: &str) -> Option<u32>` - Native guest pid for `plugin_instance_id`, when this process spawned it.

**Trait Implementations:**

- **DeploymentRuntime**
  - `fn retire_instance(self: &'life0 Self, plugin_instance_id: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn spawn_and_health(self: &'life0 Self, request: &'life1 DeploymentSpawn) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn guest_still_running(self: &'life0 Self, plugin_instance_id: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn guest_root(self: &'life0 Self, plugin_instance_id: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn tracked_instance_ids(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_host::deployment::LocalPackage

*Struct*

Local archive the reconciler may install when the ledger has no row.

**Fields:**
- `archive: std::path::PathBuf` - Archive path (`tar.gz` or `zip`).
- `manifest: bookclerk_plugin_catalog::BookclerkPackageManifest` - Install-grade package manifest for `archive`.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LocalPackage`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::deployment::SpawnHealth

*Enum*

Result of spawn plus the health RPC.

**Variants:**
- `Healthy` - Health succeeded.
- `SpawnFailed{ detail: String }` - The guest did not start. No config revision was applied.
- `HealthFailed{ detail: String }` - The guest started and health failed. The install tree stays.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SpawnHealth`
- **PartialEq**
  - `fn eq(self: &Self, other: &SpawnHealth) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::deployment::load_authorized_local_packages

*Function*

Loads operator-placed archives under `$FILES_DIR/plugin-packages/<name>/`.

Each directory holds `package.json` (a [`BookclerkPackageManifest`](../bookclerk_plugin_catalog/manifest.md#bookclerk_plugin_catalogmanifestbookclerkpackagemanifest))
and `archive.tar.gz`. Artifact URLs must be `file:` paths inside that
directory. The reconciler does not download plugins. A missing directory
is an empty map.

# Errors

Returns an error when `plugin-packages` itself cannot be read. One bad
package is skipped and does not abort the rest of the directory.

```rust
fn load_authorized_local_packages(files_dir: &std::path::Path) -> crate::Result<AuthorizedLocalPackages>
```



## bookclerk_plugin_host::deployment::reconcile_local_deployments

*Function*

Reconciles every present deployment for `node_id`.

# Errors

Returns an error when the deployment list cannot be read. A single row's
install, spawn, or observation failure is stored on that row and does not
stop the others.

```rust
fn reconcile_local_deployments(store: &bookclerk_library::LibraryStore, config: &bookclerk_config::Config, node_id: &str, packages: &AuthorizedLocalPackages, runtime: &dyn DeploymentRuntime) -> crate::Result<()>
```



