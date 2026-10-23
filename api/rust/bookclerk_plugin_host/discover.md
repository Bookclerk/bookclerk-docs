**bookclerk_plugin_host > discover**

# Module: discover

## Contents

**Structs**

- [`DiscoveredPlugin`](#discoveredplugin) - A discovered plugin ready to spawn.

**Functions**

- [`discover_plugins`](#discover_plugins) - Discover plugins under the configured search directories.
- [`first_party_database_kind`](#first_party_database_kind) - First-party database adapter kind when provenance + PluginKey match.
- [`identity_matches_occupancy`](#identity_matches_occupancy) - True when `plugin_key` / `alias` is the occupant named by `spec`.
- [`is_first_party_audiobookshelf`](#is_first_party_audiobookshelf) - True when this install is the verified Bookclerk Audiobookshelf integration.
- [`is_first_party_local_output`](#is_first_party_local_output) - True when this install is the verified Bookclerk local-filesystem destination.
- [`is_first_party_local_output_key`](#is_first_party_local_output_key) - True when `key` names the Bookclerk local-filesystem destination.
- [`is_first_party_s3_output`](#is_first_party_s3_output) - True when this install is the verified Bookclerk S3 destination.
- [`occupancy_matches_alias`](#occupancy_matches_alias) - True when occupancy `spec` names the display alias (or kind token) `alias`.
- [`occupancy_names_alias`](#occupancy_names_alias) - True when occupancy `spec` names `alias` among `discovered` installs.
- [`occupancy_spec`](#occupancy_spec) - Config occupancy selector (`plugin = "…"`), or `alias` when the field is empty.
- [`plugin_matches_occupancy`](#plugin_matches_occupancy) - True when `plugin` is the occupant named by `spec` (PluginKey or alias).
- [`plugin_search_dirs`](#plugin_search_dirs) - Resolve search roots: `BOOKCLERK_PLUGIN_DIRS` then `$FILES_DIR/plugins`.
- [`resolve_plugin_ref`](#resolve_plugin_ref) - Resolves `spec` to a discovered plugin.
- [`resolve_plugin_slot`](#resolve_plugin_slot) - Resolves `spec` among `plugins` without treating a vacant slot as an error.
- [`settings_table`](#settings_table) - Bootstrap connection settings for a database adapter from `config.toml`.
- [`settings_table_for`](#settings_table_for) - [`settings_table`](discover.md#bookclerk_plugin_hostdiscoversettings_table) for one explicit handler `family`.
- [`stamp_occupancy_plugin_key`](#stamp_occupancy_plugin_key) - Records `plugin` as the `[database].plugin` occupant.
- [`upgrade_unique_alias_occupancy`](#upgrade_unique_alias_occupancy) - Rewrites a unique `[database].plugin` alias to the canonical PluginKey.

---

## bookclerk_plugin_host::discover::DiscoveredPlugin

*Struct*

A discovered plugin ready to spawn.

**Fields:**
- `manifest: crate::manifest::PluginManifest` - Parsed `plugin.toml` for this install directory.
- `root: std::path::PathBuf` - Directory containing `plugin.toml` (cwd + relative `command` base).
- `command: std::path::PathBuf` - Absolute path to the plugin executable.
- `identity: bookclerk_plugin_catalog::PluginInstallIdentity` - Host-evaluated provenance-qualified identity.
- `version_grant: Option<crate::PluginGrant>` - Consent grant recorded on the cluster release this tree was

**Methods:**

- `fn new(manifest: PluginManifest, root: PathBuf, command: PathBuf) -> Self` - Builds a discovered plugin, evaluating content hashes and provenance.
- `fn try_new(manifest: PluginManifest, root: PathBuf, command: PathBuf, files_dir: Option<&Path>) -> Result<Self>` - Production constructor: fail closed on malformed receipts, hash errors,
- `fn for_test(manifest: PluginManifest, root: PathBuf, command: PathBuf) -> Self` - Explicit test/dev constructor for incomplete plugin trees.
- `fn from_release(manifest: PluginManifest, root: PathBuf, plugin_key: &str, version: &str, payload_root_sha256: &str, archive_sha256: &str, grant: crate::PluginGrant) -> Result<Self>` - Builds a plugin from a tree materialized from a cluster release.
- `fn alias(self: &Self) -> &str` - Display / CLI alias (`plugin.toml` id).
- `fn plugin_key(self: &Self) -> &PluginKey` - Provenance-qualified key.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DiscoveredPlugin`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::discover::discover_plugins

*Function*

Discover plugins under the configured search directories.

Accepts either:
- `$dir/plugin.toml` (single plugin at root), or
- `$dir/<name>/plugin.toml` (one plugin per subdirectory).

Duplicate [`PluginKey`](../bookclerk_plugin_catalog/identity.md#bookclerk_plugin_catalogidentitypluginkey) values are a hard error. Duplicate aliases from
different PluginKeys are also a hard error (invalid installation state).

# Errors

Returns [`PluginError`](error.md#bookclerk_plugin_hosterrorpluginerror) on duplicate keys, duplicate aliases, missing binaries, or I/O failures.

```rust
fn discover_plugins(config: &bookclerk_config::Config) -> crate::Result<Vec<DiscoveredPlugin>>
```



## bookclerk_plugin_host::discover::first_party_database_kind

*Function*

First-party database adapter kind when provenance + PluginKey match.

`PlatformBundled` (platform sqlite) and `VerifiedStaged` (crates.io /
platform postgres and D1) both qualify. `LocalDevelopment` and `Modified`
never do. Alias text (`id = "sqlite"`) is not consulted.

```rust
fn first_party_database_kind(plugin: &DiscoveredPlugin) -> Option<bookclerk_config::DatabasePluginKind>
```



## bookclerk_plugin_host::discover::identity_matches_occupancy

*Function*

True when `plugin_key` / `alias` is the occupant named by `spec`.

A parseable PluginKey never falls through to an alias comparison, so a
miss cannot inherit another provenance's grant or session. Does not
detect corrupt duplicate aliases — callers that load or privilege-check
must require a unique match ([`resolve_plugin_slot`](discover.md#bookclerk_plugin_hostdiscoverresolve_plugin_slot)).

# Arguments

* `plugin_key` - Canonical [`PluginKey`](../bookclerk_plugin_catalog/identity.md#bookclerk_plugin_catalogidentitypluginkey) text.
* `alias` - Manifest display id.
* `spec` - Occupancy selector from [`occupancy_spec`](discover.md#bookclerk_plugin_hostdiscoveroccupancy_spec) or a job/CLI id.

```rust
fn identity_matches_occupancy(plugin_key: &str, alias: &str, spec: &str) -> bool
```



## bookclerk_plugin_host::discover::is_first_party_audiobookshelf

*Function*

True when this install is the verified Bookclerk Audiobookshelf integration.

Only this plugin receives the host-implied network grant for the server
its instance document names (`base_url`). The live manifest alias is not
consulted: an alias twin is a different PluginKey with its own consent.

```rust
fn is_first_party_audiobookshelf(plugin: &DiscoveredPlugin) -> bool
```



## bookclerk_plugin_host::discover::is_first_party_local_output

*Function*

True when this install is the verified Bookclerk local-filesystem destination.

```rust
fn is_first_party_local_output(plugin: &DiscoveredPlugin) -> bool
```



## bookclerk_plugin_host::discover::is_first_party_local_output_key

*Function*

True when `key` names the Bookclerk local-filesystem destination.

This is the key half of [`is_first_party_local_output`]: it does not see
an install, so it cannot check provenance. Use it only where matching
more plugins than the verified one is safe, such as routing acquires to
the nodes that hold their output disks when discovery is unavailable.

# Arguments

* `key` - Durable plugin identity to classify.

# Returns

`true` for the first-party local destination package on a first-party
provenance scheme.

```rust
fn is_first_party_local_output_key(key: &bookclerk_plugin_catalog::PluginKey) -> bool
```



## bookclerk_plugin_host::discover::is_first_party_s3_output

*Function*

True when this install is the verified Bookclerk S3 destination.

```rust
fn is_first_party_s3_output(plugin: &DiscoveredPlugin) -> bool
```



## bookclerk_plugin_host::discover::occupancy_matches_alias

*Function*

True when occupancy `spec` names the display alias (or kind token) `alias`.

Bare kind tokens (`pg` ↔ `postgres`) match without discovery. A PluginKey
occupant matches only when it is a host-controlled first-party package
whose **package name** maps to that alias — never via a `#fragment`.
Third-party keys need [`occupancy_names_alias`](discover.md#bookclerk_plugin_hostdiscoveroccupancy_names_alias) plus discovery.

A bare alias does **not** match a PluginKey string passed as `alias`.

# Arguments

* `spec` - Occupancy selector from config (`plugin = "…"`).
* `alias` - Display alias or first-party kind token being compared.

```rust
fn occupancy_matches_alias(spec: &str, alias: &str) -> bool
```



## bookclerk_plugin_host::discover::occupancy_names_alias

*Function*

True when occupancy `spec` names `alias` among `discovered` installs.

Uses [`occupancy_matches_alias`](discover.md#bookclerk_plugin_hostdiscoveroccupancy_matches_alias) first, then resolves `spec` to exactly one
PluginKey. Corrupt duplicate aliases fail closed (`false`).

```rust
fn occupancy_names_alias(spec: &str, alias: &str, discovered: &[DiscoveredPlugin]) -> bool
```



## bookclerk_plugin_host::discover::occupancy_spec

*Function*

Config occupancy selector (`plugin = "…"`), or `alias` when the field is empty.

Occupancy is how host config names the guest that may use a singleton
slot (`[database].plugin`, `[output.s3].plugin`, `[sources.<id>].plugin`).
An empty field still means the display alias, which
[`resolve_plugin_slot`](discover.md#bookclerk_plugin_hostdiscoverresolve_plugin_slot) accepts only when exactly one install uses it.

# Arguments

* `plugin_field` - Occupancy string from config (`plugin = "…"`).
* `alias` - Display alias used when `plugin_field` is empty.

```rust
fn occupancy_spec<'a>(plugin_field: &'a str, alias: &'a str) -> &'a str
```



## bookclerk_plugin_host::discover::plugin_matches_occupancy

*Function*

True when `plugin` is the occupant named by `spec` (PluginKey or alias).

Does not detect corrupt duplicate aliases — loaders must call [`resolve_plugin_slot`](discover.md#bookclerk_plugin_hostdiscoverresolve_plugin_slot).

# Arguments

* `plugin` - Candidate install.
* `spec` - Occupancy selector from [`occupancy_spec`](discover.md#bookclerk_plugin_hostdiscoveroccupancy_spec).

```rust
fn plugin_matches_occupancy(plugin: &DiscoveredPlugin, spec: &str) -> bool
```



## bookclerk_plugin_host::discover::plugin_search_dirs

*Function*

Resolve search roots: `BOOKCLERK_PLUGIN_DIRS` then `$FILES_DIR/plugins`.

```rust
fn plugin_search_dirs(config: &bookclerk_config::Config) -> Vec<std::path::PathBuf>
```



## bookclerk_plugin_host::discover::resolve_plugin_ref

*Function*

Resolves `spec` to a discovered plugin.

`spec` may be a canonical [`PluginKey`](../bookclerk_plugin_catalog/identity.md#bookclerk_plugin_catalogidentitypluginkey) or an alias unique in this host
plugin namespace. In valid installed state an alias maps to exactly one
PluginKey. Duplicate aliases fail closed rather than picking first or last.

# Errors

Returns an error when `spec` matches nothing, or when corrupt disk state
contains duplicate keys or aliases.

```rust
fn resolve_plugin_ref<'a>(plugins: &'a [DiscoveredPlugin], spec: &str) -> crate::Result<&'a DiscoveredPlugin>
```



## bookclerk_plugin_host::discover::resolve_plugin_slot

*Function*

Resolves `spec` among `plugins` without treating a vacant slot as an error.

`None` means nothing installed matches. An ambiguous alias is an error
so callers fail closed instead of spawning every twin or last-write-wins.

# Arguments

* `plugins` - Already-filtered family candidates.
* `spec` - Occupancy selector from [`occupancy_spec`](discover.md#bookclerk_plugin_hostdiscoveroccupancy_spec).

# Errors

Returns [`PluginError`](error.md#bookclerk_plugin_hosterrorpluginerror) when two installs share the alias (or PluginKey).

```rust
fn resolve_plugin_slot<'a>(plugins: &'a [DiscoveredPlugin], spec: &str) -> crate::Result<Option<&'a DiscoveredPlugin>>
```



## bookclerk_plugin_host::discover::settings_table

*Function*

Bootstrap connection settings for a database adapter from `config.toml`.

Database adapters are the one plugin family whose configuration stays in
the bootstrap file: the process needs it to open the library before the
database-backed control plane exists. Every other family receives its
settings from the plugin instance document.

```rust
fn settings_table(config: &bookclerk_config::Config, plugin: &DiscoveredPlugin) -> toml::Table
```



## bookclerk_plugin_host::discover::settings_table_for

*Function*

[`settings_table`](discover.md#bookclerk_plugin_hostdiscoversettings_table) for one explicit handler `family`.

Only `PluginFamily::Database` has a
`config.toml` table; every other family returns an empty table.

```rust
fn settings_table_for(config: &bookclerk_config::Config, plugin: &DiscoveredPlugin, family: crate::PluginFamily) -> toml::Table
```



## bookclerk_plugin_host::discover::stamp_occupancy_plugin_key

*Function*

Records `plugin` as the `[database].plugin` occupant.

Database adapters are the bootstrap exception: they stay selected in
`config.toml` because the connection is needed before the database can be
read. Every other plugin is enabled by a `plugin_deployments` row.

# Errors

Returns an error when `plugin` is not a database adapter.

```rust
fn stamp_occupancy_plugin_key(config: & mut bookclerk_config::Config, plugin: &DiscoveredPlugin) -> crate::Result<()>
```



## bookclerk_plugin_host::discover::upgrade_unique_alias_occupancy

*Function*

Rewrites a unique `[database].plugin` alias to the canonical PluginKey.

An ambiguous alias or vacant slot is left unchanged so loaders still fail
closed. An empty `discovered` slice is a no-op so a settings discovery
timeout cannot wipe occupancy.

```rust
fn upgrade_unique_alias_occupancy(config: & mut bookclerk_config::Config, discovered: &[DiscoveredPlugin])
```



