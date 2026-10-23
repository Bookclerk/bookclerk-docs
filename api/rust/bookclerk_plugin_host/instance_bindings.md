**bookclerk_plugin_host > instance_bindings**

# Module: instance_bindings

## Contents

**Structs**

- [`PreparedOpen`](#preparedopen) - Bindings passed to `PluginWorker.open`, plus the spawn config table.

**Functions**

- [`binding_grant_flags`](#binding_grant_flags) - `config` / `secrets` flags of the plugin's covering grant.
- [`prepare_open_for_instance`](#prepare_open_for_instance) - Resolves open bindings for `instance_id` of `plugin`.
- [`prepared_open_from_resolved`](#prepared_open_from_resolved) - Builds open bindings from a deployment's already resolved payloads.

---

## bookclerk_plugin_host::instance_bindings::PreparedOpen

*Struct*

Bindings passed to `PluginWorker.open`, plus the spawn config table.

**Fields:**
- `bindings: bookclerk_plugin_sdk::BindingValues` - Values for `PluginWorker.open`.
- `spawn_config_table: serde_json::Value` - JSON object passed into `crate::PluginSession::spawn_with`.
- `granted_config: serde_json::Value` - JSON object the guest receives when the grant includes `config`.
- `config_revision: i64` - Document revision this open applies.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PreparedOpen`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::instance_bindings::binding_grant_flags

*Function*

`config` / `secrets` flags of the plugin's covering grant.

```rust
fn binding_grant_flags(grant: &crate::PluginGrant) -> bookclerk_library::control_plane::InstanceBindingGrant
```



## bookclerk_plugin_host::instance_bindings::prepare_open_for_instance

*Function*

Resolves open bindings for `instance_id` of `plugin`.

The covering grant comes from `plugin-grants.json`; the settings schema
from the installed manifest. A missing document, a dangling secret ref, or
a missing required setting is an error. Document keys the installed
manifest no longer declares are dropped and logged.

# Errors

Returns an error when the grant cannot be loaded, the instance is not this
plugin's, or its document cannot be resolved.

```rust
fn prepare_open_for_instance(store: &bookclerk_library::LibraryStore, files_dir: &std::path::Path, plugin: &crate::discover::DiscoveredPlugin, instance_id: &bookclerk_library::control_plane::PluginInstanceId) -> crate::Result<PreparedOpen>
```



## bookclerk_plugin_host::instance_bindings::prepared_open_from_resolved

*Function*

Builds open bindings from a deployment's already resolved payloads.

Does not look up an instance by plugin key.

```rust
fn prepared_open_from_resolved(config: bookclerk_plugin_sdk::ExtensibleConfig, secrets: bookclerk_plugin_sdk::ExtensibleConfig, config_revision: i64) -> PreparedOpen
```



