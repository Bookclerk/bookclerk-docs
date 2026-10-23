**bookclerk_plugin_host > install_preflight**

# Module: install_preflight

## Contents

**Functions**

- [`configured_alias_occupants`](#configured_alias_occupants) - Occupancy (PluginKey, alias, path) under every configured discovery root.
- [`install_from_manifest_with_configured_aliases`](#install_from_manifest_with_configured_aliases) - Catalog install after [`reject_configured_alias_collision`](install_preflight.md#bookclerk_plugin_hostinstall_preflightreject_configured_alias_collision).
- [`install_local_archive_with_configured_aliases`](#install_local_archive_with_configured_aliases) - Local-archive install after [`reject_configured_alias_collision`](install_preflight.md#bookclerk_plugin_hostinstall_preflightreject_configured_alias_collision).
- [`reject_configured_alias_collision`](#reject_configured_alias_collision) - Rejects installing `incoming_alias` when a different PluginKey already owns

---

## bookclerk_plugin_host::install_preflight::configured_alias_occupants

*Function*

Occupancy (PluginKey, alias, path) under every configured discovery root.

Walks [`plugin_search_dirs`](discover.md#bookclerk_plugin_hostdiscoverplugin_search_dirs) without hashing payloads. A receipt PluginKey
is preferred; otherwise `PluginKey::from_install_path`.

# Errors

Returns when a `plugin.toml` cannot be read or parsed, or a PluginKey
cannot be formed for an occupant.

```rust
fn configured_alias_occupants(config: &bookclerk_config::Config) -> crate::Result<Vec<(bookclerk_plugin_catalog::PluginKey, String, std::path::PathBuf)>>
```



## bookclerk_plugin_host::install_preflight::install_from_manifest_with_configured_aliases

*Function*

Catalog install after [`reject_configured_alias_collision`](install_preflight.md#bookclerk_plugin_hostinstall_preflightreject_configured_alias_collision).

`lock` must already cover `opts.plugins_root` so alias preflight and the
catalog transaction share one host-local mutation lock.

# Errors

Returns a catalog error when the lock does not cover this plugins root,
preflight rejects the alias, or install fails.

```rust
fn install_from_manifest_with_configured_aliases(config: &bookclerk_config::Config, lock: &bookclerk_plugin_catalog::PluginMutationLock, manifest: &bookclerk_plugin_catalog::BookclerkPackageManifest, coordinate: &bookclerk_plugin_catalog::PackageCoordinate, opts: &bookclerk_plugin_catalog::InstallOptions) -> bookclerk_plugin_catalog::Result<bookclerk_plugin_catalog::InstallOutcome>
```



## bookclerk_plugin_host::install_preflight::install_local_archive_with_configured_aliases

*Function*

Local-archive install after [`reject_configured_alias_collision`](install_preflight.md#bookclerk_plugin_hostinstall_preflightreject_configured_alias_collision).

`lock` must already cover `opts.plugins_root`.

# Errors

Returns a catalog error when the lock does not cover this plugins root,
preflight rejects the alias, or install fails.

```rust
fn install_local_archive_with_configured_aliases(config: &bookclerk_config::Config, lock: &bookclerk_plugin_catalog::PluginMutationLock, archive: &std::path::Path, manifest: &bookclerk_plugin_catalog::BookclerkPackageManifest, opts: &bookclerk_plugin_catalog::InstallOptions) -> bookclerk_plugin_catalog::Result<bookclerk_plugin_catalog::InstallOutcome>
```



## bookclerk_plugin_host::install_preflight::reject_configured_alias_collision

*Function*

Rejects installing `incoming_alias` when a different PluginKey already owns
it anywhere in the configured discovery universe.

Same PluginKey remains eligible for update/replace at the **same** alias.
`--replace` cannot seize a foreign alias or rename an installed alias.

# Errors

Returns [`PluginError`](error.md#bookclerk_plugin_hosterrorpluginerror) when the alias is occupied by another PluginKey or
occupancy cannot be read.

```rust
fn reject_configured_alias_collision(config: &bookclerk_config::Config, incoming_key: &bookclerk_plugin_catalog::PluginKey, incoming_alias: &str) -> crate::Result<()>
```



