**bookclerk > registry**

# Module: registry

## Contents

**Functions**

- [`destinations`](#destinations) - Spawns every storage deployment present on this node.
- [`integrations`](#integrations) - Spawns every integration deployment present on this node.
- [`open_host`](#open_host) - Opens the library and enrolls local plugins. Loads no guest.
- [`open_library`](#open_library) - Opens the library and aligns the cluster secret root, without enrolling
- [`resolve_source`](#resolve_source) - Resolves `--source` (plugin instance id, PluginKey, or alias) and an
- [`sources`](#sources) - Spawns every storefront deployment present on this node.

---

## bookclerk::registry::destinations

*Function*

Spawns every storage deployment present on this node.

# Errors

Returns an error when a present destination guest cannot start.

```rust
fn destinations(host: &bookclerk_plugin_host::HostControlPlane) -> anyhow::Result<bookclerk_plugin_host::DestinationRegistry>
```



## bookclerk::registry::integrations

*Function*

Spawns every integration deployment present on this node.

# Errors

Returns an error when the files directory is unavailable.

```rust
fn integrations(host: &bookclerk_plugin_host::HostControlPlane) -> anyhow::Result<bookclerk_integrations::IntegrationRegistry>
```



## bookclerk::registry::open_host

*Function*

Opens the library and enrolls local plugins. Loads no guest.

# Errors

Returns an error when the database cannot be opened or the control plane
cannot be bootstrapped.

```rust
fn open_host(config: &bookclerk_config::Config) -> anyhow::Result<bookclerk_plugin_host::HostControlPlane>
```



## bookclerk::registry::open_library

*Function*

Opens the library and aligns the cluster secret root, without enrolling
plugins. For commands that never load a guest.

Alignment runs before the caller can cache a data-encryption key or seal a
secret. An enrolled database with a missing or different `master.key` fails
here, and this function does not leave a replacement key behind.

# Errors

Returns an error when the database cannot be opened or the secret root
does not align.

```rust
fn open_library(config: &bookclerk_config::Config) -> anyhow::Result<bookclerk_library::LibraryStore>
```



## bookclerk::registry::resolve_source

*Function*

Resolves `--source` (plugin instance id, PluginKey, or alias) and an
optional `--instance` pin to one registered source.

A bare key or alias routes to the plugin's default instance. An unknown
needle lists what is registered; a pin that is not an instance of that
plugin fails rather than falling back.

# Errors

Returns an error when nothing matches.

```rust
fn resolve_source(registry: &bookclerk_source::SourceRegistry, needle: &str, instance: Option<&str>) -> anyhow::Result<std::sync::Arc<dyn ContentSource>>
```



## bookclerk::registry::sources

*Function*

Spawns every storefront deployment present on this node.

# Errors

Returns an error when the files directory is unavailable.

```rust
fn sources(host: &bookclerk_plugin_host::HostControlPlane) -> anyhow::Result<bookclerk_source::SourceRegistry>
```



