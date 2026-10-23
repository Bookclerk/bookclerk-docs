**bookclerk_plugin_host > enrollment**

# Module: enrollment

## Contents

**Structs**

- [`EnrolledPlugin`](#enrolledplugin) - One plugin after enrollment on this node.

**Functions**

- [`enroll_local_plugins`](#enroll_local_plugins) - Enrolls every discovered plugin on `node_id`.
- [`enroll_one`](#enroll_one) - Enrolls one discovered plugin. See the module docs for the steps.
- [`has_covering_grant`](#has_covering_grant) - True when the operator already consented to this plugin, or it is a
- [`is_bootstrap_only`](#is_bootstrap_only) - True when the plugin is a database adapter and nothing else: those stay

---

## bookclerk_plugin_host::enrollment::EnrolledPlugin

*Struct*

One plugin after enrollment on this node.

**Fields:**
- `plugin: crate::discover::DiscoveredPlugin` - Discovered install.
- `instance: bookclerk_library::control_plane::PluginInstance` - Default instance for the plugin key.
- `deployment: bookclerk_library::control_plane::PluginDeployment` - This node's deployment of the default instance.
- `schema: bookclerk_library::control_plane::PinnedPluginSchema` - Pinned settings schema.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EnrolledPlugin`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::enrollment::enroll_local_plugins

*Function*

Enrolls every discovered plugin on `node_id`.

A plugin that fails to enroll is logged and skipped so one bad install
does not keep the others from starting.

# Errors

Returns an error when discovery or the grant store cannot be read.

```rust
fn enroll_local_plugins(store: &bookclerk_library::LibraryStore, config: &bookclerk_config::Config, node_id: &str) -> crate::Result<Vec<EnrolledPlugin>>
```



## bookclerk_plugin_host::enrollment::enroll_one

*Function*

Enrolls one discovered plugin. See the module docs for the steps.

# Errors

Returns an error when a schema, instance, document, or deployment write fails.

```rust
fn enroll_one(store: &bookclerk_library::LibraryStore, node_id: &str, plugin: &crate::discover::DiscoveredPlugin, grants: &crate::consent::PluginGrantStore) -> crate::Result<EnrolledPlugin>
```



## bookclerk_plugin_host::enrollment::has_covering_grant

*Function*

True when the operator already consented to this plugin, or it is a
verified platform guest that grants itself on first spawn.

```rust
fn has_covering_grant(plugin: &crate::discover::DiscoveredPlugin, grants: &crate::consent::PluginGrantStore) -> bool
```



## bookclerk_plugin_host::enrollment::is_bootstrap_only

*Function*

True when the plugin is a database adapter and nothing else: those stay
bootstrap (`[database]` in `config.toml`) in this phase.

```rust
fn is_bootstrap_only(plugin: &crate::discover::DiscoveredPlugin) -> bool
```



