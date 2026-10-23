**bookclerk > commands > plugin_installs**

# Module: commands::plugin_installs

## Contents

**Structs**

- [`InstallArgs`](#installargs) - What `plugins install` was asked to install.

**Functions**

- [`active_version_for`](#active_version_for) - The active version a plugin alias or key names, if any.
- [`run_approve`](#run_approve) - `plugins approve` for an installed plugin. Returns false when `id` names
- [`run_install`](#run_install) - `plugins install`: install a version.
- [`run_list`](#run_list) - `plugins versions [plugin]`: installed versions and the active pointer.
- [`run_rollback`](#run_rollback) - `plugins rollback <plugin> <version>`: move the active pointer back.
- [`run_uninstall`](#run_uninstall) - `plugins uninstall` for a installed plugin. Returns false when `id` names
- [`run_update`](#run_update) - `plugins update` for installed plugins. Returns false when `id` names no

---

## bookclerk::commands::plugin_installs::InstallArgs

*Struct*

What `plugins install` was asked to install.

**Generic Parameters:**
- 'a

**Fields:**
- `coordinate: &'a str` - Coordinate text (ignored with `--archive`).
- `archive: Option<&'a std::path::Path>` - Local archive.
- `manifest: Option<&'a std::path::Path>` - Package manifest for `--archive`.
- `target: Option<String>` - Target of the local archive.
- `activate: bool` - Make this the active version even when another is active.
- `approve_capabilities: bool` - Record the requested consent as approved.
- `allow_unverified_publisher: bool` - Accept a publisher without independent authenticity.
- `dry_run: bool` - Resolve and print only.
- `offline: bool` - Refuse remote downloads.



## bookclerk::commands::plugin_installs::active_version_for

*Function*

The active version a plugin alias or key names, if any.

# Errors

Returns an error when a library read fails.

```rust
fn active_version_for(store: &bookclerk_library::LibraryStore, needle: &str) -> anyhow::Result<Option<bookclerk_library::control_plane::PluginVersion>>
```



## bookclerk::commands::plugin_installs::run_approve

*Function*

`plugins approve` for an installed plugin. Returns false when `id` names
no installed plugin.

# Errors

Returns an error when approval is declined or a write fails.

```rust
fn run_approve(config: &bookclerk_config::Config, id: &str, yes: bool, format: crate::format_out::OutputFormat) -> anyhow::Result<bool>
```



## bookclerk::commands::plugin_installs::run_install

*Function*

`plugins install`: install a version.

# Errors

Returns an error when trust policy refuses the publisher, the package
does not resolve or verify, or the alias belongs to another plugin.

```rust
fn run_install(config: &bookclerk_config::Config, args: InstallArgs, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



## bookclerk::commands::plugin_installs::run_list

*Function*

`plugins versions [plugin]`: installed versions and the active pointer.

# Errors

Returns an error when a library read fails.

```rust
fn run_list(config: &bookclerk_config::Config, id: Option<&str>, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



## bookclerk::commands::plugin_installs::run_rollback

*Function*

`plugins rollback <plugin> <version>`: move the active pointer back.

# Errors

Returns an error when the plugin or version is not installed.

```rust
fn run_rollback(config: &bookclerk_config::Config, id: &str, version: &str, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



## bookclerk::commands::plugin_installs::run_uninstall

*Function*

`plugins uninstall` for a installed plugin. Returns false when `id` names
no installed plugin.

# Errors

Returns an error when a library write fails.

```rust
fn run_uninstall(config: &bookclerk_config::Config, id: &str, format: crate::format_out::OutputFormat) -> anyhow::Result<bool>
```



## bookclerk::commands::plugin_installs::run_update

*Function*

`plugins update` for installed plugins. Returns false when `id` names no
installed plugin (the caller falls back to host-local receipts).

# Errors

Returns an error when resolution, installing, or activation fails.

```rust
fn run_update(config: &bookclerk_config::Config, id: Option<&str>, to: Option<&str>, allow_unverified_publisher: bool, approve_capabilities: bool, dry_run: bool, format: crate::format_out::OutputFormat) -> anyhow::Result<bool>
```



