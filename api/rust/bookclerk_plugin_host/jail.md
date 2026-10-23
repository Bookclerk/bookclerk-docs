**bookclerk_plugin_host > jail**

# Module: jail

## Contents

**Functions**

- [`guest_reservation`](#guest_reservation) - CPU and memory one guest may use at most on this host: its grant's
- [`plugin_data_dir`](#plugin_data_dir) - The private directory a plugin keeps state in.

---

## bookclerk_plugin_host::jail::guest_reservation

*Function*

CPU and memory one guest may use at most on this host: its grant's
ceilings after `[plugins.jail]`, the numbers its jail enforces. The
cluster scheduler accounts host budgets against these.

```rust
fn guest_reservation(manifest: &bookclerk_plugin_manifest::PluginManifest, grant: Option<&crate::PluginGrant>, jail: &bookclerk_config::PluginsJailConfig) -> bookclerk_library::control_plane::GuestReservation
```



## bookclerk_plugin_host::jail::plugin_data_dir

*Function*

The private directory a plugin keeps state in.

State is keyed by [`bookclerk_plugin_catalog::PluginKey`](../bookclerk_plugin_catalog/identity.md#bookclerk_plugin_catalogidentitypluginkey) (`plugin-state/<fs_id>/data`), not the
display alias. Invalid keys cannot be constructed.

# Errors

Returns an error when the operation fails.

```rust
fn plugin_data_dir(config: &bookclerk_config::Config, plugin: &crate::discover::DiscoveredPlugin) -> crate::Result<std::path::PathBuf>
```



