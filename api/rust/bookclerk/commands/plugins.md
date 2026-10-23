**bookclerk > commands > plugins**

# Module: commands::plugins

## Contents

**Enums**

- [`InstancesCommand`](#instancescommand) - `plugins instances` subcommands.
- [`PluginConfigCommand`](#pluginconfigcommand) - `plugins config` subcommands.
- [`PluginDbCommand`](#plugindbcommand) - Nested `plugins db` actions over the `plugin_databases` registry.
- [`PluginsCommand`](#pluginscommand) - `bookclerk plugins` subcommands: install plugins into the cluster, deploy
- [`RegistryCommand`](#registrycommand) - Nested `plugins registry` actions that edit `[[plugins.registries]]`.
- [`RegistryKindArg`](#registrykindarg) - Registry adapter kind accepted by `plugins registry add`.

**Functions**

- [`augment_plugins_command`](#augment_plugins_command) - Augment the `plugins` clap command with discovered plugin ids (manifest CLI).
- [`run`](#run) - Dispatches a `plugins` subcommand against the loaded config.
- [`run_plugin_cli`](#run_plugin_cli) - Invoke `bookclerk plugins <plugin-id> <command> …` via JSON-RPC `cli.invoke`.

---

## bookclerk::commands::plugins::InstancesCommand

*Enum*

`plugins instances` subcommands.

**Variants:**
- `List{ plugin: Option<String> }` - List instances with their deployments on every node.
- `Create{ plugin: String, name: Option<String>, node: Option<String> }` - Create another instance of a plugin that has a pinned schema.
- `Delete{ instance: String }` - Delete an instance that has no deployment on any node.

**Trait Implementations:**

- **Subcommand**
  - `fn augment_subcommands<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn augment_subcommands_for_update<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn has_subcommand(__clap_name: &str) -> bool`
- **FromArgMatches**
  - `fn from_arg_matches(__clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn from_arg_matches_mut(__clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn update_from_arg_matches(self: & mut Self, __clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
  - `fn update_from_arg_matches_mut<'b>(self: & mut Self, __clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk::commands::plugins::PluginConfigCommand

*Enum*

`plugins config` subcommands.

**Variants:**
- `Get{ instance: String }` - Show an instance's effective settings: schema defaults under the
- `Set{ instance: String, key: String, value: Option<String>, secret: bool }` - Set one setting. A `secret` setting takes `--secret` and reads the
- `Unset{ instance: String, key: String }` - Unset one setting so the schema default applies, or drop a secret.

**Trait Implementations:**

- **FromArgMatches**
  - `fn from_arg_matches(__clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn from_arg_matches_mut(__clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn update_from_arg_matches(self: & mut Self, __clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
  - `fn update_from_arg_matches_mut<'b>(self: & mut Self, __clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Subcommand**
  - `fn augment_subcommands<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn augment_subcommands_for_update<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn has_subcommand(__clap_name: &str) -> bool`



## bookclerk::commands::plugins::PluginDbCommand

*Enum*

Nested `plugins db` actions over the `plugin_databases` registry.

**Variants:**
- `List{ plugin: Option<String> }` - List provisioned plugin database bindings.
- `Drop{ plugin: String, binding: Option<String>, yes: bool }` - Drop provisioned binding units and their registry rows.

**Trait Implementations:**

- **FromArgMatches**
  - `fn from_arg_matches(__clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn from_arg_matches_mut(__clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn update_from_arg_matches(self: & mut Self, __clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
  - `fn update_from_arg_matches_mut<'b>(self: & mut Self, __clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Subcommand**
  - `fn augment_subcommands<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn augment_subcommands_for_update<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn has_subcommand(__clap_name: &str) -> bool`



## bookclerk::commands::plugins::PluginsCommand

*Enum*

`bookclerk plugins` subcommands: install plugins into the cluster, deploy
them onto nodes, configure instances, and manage registries.

**Variants:**
- `List` - List plugins found under plugin search directories.
- `Search{ query: Option<String>, limit: u32 }` - Search configured registries (and crates.io) for Bookclerk plugins.
- `Install{ coordinate: String, archive: Option<std::path::PathBuf>, manifest: Option<std::path::PathBuf>, target: Option<String>, activate: bool, approve_capabilities: bool, allow_unverified_publisher: bool, dry_run: bool, offline: bool }` - Install a plugin version into the cluster from a source-qualified
- `Update{ id: Option<String>, to: Option<String>, allow_unverified_publisher: bool, approve_capabilities: bool, dry_run: bool }` - Install and activate a newer version of an installed plugin (or
- `Rollback{ id: String, version: String }` - Activate an earlier installed version of a plugin.
- `Versions{ id: Option<String> }` - List installed versions (the active one is starred).
- `Uninstall{ id: String, purge_state: bool, discard_acl_journals: bool }` - Uninstall a plugin: clear its active version and undeploy it from
- `Info{ id: String }` - Show details for one discovered plugin (local + receipt).
- `Diagnose{ id: Option<String> }` - Run diagnose probes for one (or all) plugins that support it.
- `Doctor{ id: Option<String> }` - Check install integrity, target, jail, and describe/health status.
- `Deploy{ id: String, node: Option<String>, replicas: Option<bookclerk_library::control_plane::Replicas>, on: Vec<String>, manual: bool }` - Deploy a plugin instance. With no flags it runs on this node (or on
- `Undeploy{ id: String, node: Option<String> }` - Undeploy a plugin instance from this node (or `--node`): its
- `Instances{ command: InstancesCommand }` - Plugin instances: one per plugin by default, more when an operator
- `Config{ command: PluginConfigCommand }` - Instance settings, validated against the plugin's declared schema.
- `Approve{ id: String, yes: bool }` - Approve network domains and host bindings for a plugin.
- `Registry{ command: RegistryCommand }` - Manage configured plugin registries.
- `Db{ command: PluginDbCommand }` - Manage isolated plugin database bindings (`capabilities.bindings.databases`).

**Trait Implementations:**

- **FromArgMatches**
  - `fn from_arg_matches(__clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn from_arg_matches_mut(__clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn update_from_arg_matches(self: & mut Self, __clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
  - `fn update_from_arg_matches_mut<'b>(self: & mut Self, __clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Subcommand**
  - `fn augment_subcommands<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn augment_subcommands_for_update<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn has_subcommand(__clap_name: &str) -> bool`



## bookclerk::commands::plugins::RegistryCommand

*Enum*

Nested `plugins registry` actions that edit `[[plugins.registries]]`.

**Variants:**
- `List` - List configured registries.
- `Add{ kind: RegistryKindArg, url: Option<String>, name: Option<String> }` - Add a registry source to `config.toml`.
- `Remove{ index: Option<usize>, url: Option<String> }` - Remove a registry by index (from `registry list`) or URL.

**Trait Implementations:**

- **FromArgMatches**
  - `fn from_arg_matches(__clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn from_arg_matches_mut(__clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn update_from_arg_matches(self: & mut Self, __clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
  - `fn update_from_arg_matches_mut<'b>(self: & mut Self, __clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Subcommand**
  - `fn augment_subcommands<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn augment_subcommands_for_update<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn has_subcommand(__clap_name: &str) -> bool`



## bookclerk::commands::plugins::RegistryKindArg

*Enum*

Registry adapter kind accepted by `plugins registry add`.

**Variants:**
- `Static` - HTTPS index of Bookclerk package manifests.
- `Cargo` - crates.io (or a Cargo-compatible registry URL).
- `Npm` - npm registry (default `registry.npmjs.org`).
- `Pypi` - PyPI (or a compatible simple-index URL).

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RegistryKindArg`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **ValueEnum**
  - `fn value_variants<'a>() -> &'a [Self]`
  - `fn to_possible_value<'a>(self: &Self) -> ::std::option::Option<clap::builder::PossibleValue>`



## bookclerk::commands::plugins::augment_plugins_command

*Function*

Augment the `plugins` clap command with discovered plugin ids (manifest CLI).

```rust
fn augment_plugins_command(plugins_cmd: clap::Command, config: &bookclerk_config::Config) -> clap::Command
```



## bookclerk::commands::plugins::run

*Function*

Dispatches a `plugins` subcommand against the loaded config.

```rust
fn run(command: PluginsCommand, config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



## bookclerk::commands::plugins::run_plugin_cli

*Function*

Invoke `bookclerk plugins <plugin-id> <command> …` via JSON-RPC `cli.invoke`.

```rust
fn run_plugin_cli(config: &bookclerk_config::Config, plugin_id: &str, argv: &[String], format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



