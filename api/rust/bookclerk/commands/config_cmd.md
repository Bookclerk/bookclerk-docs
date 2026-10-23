**bookclerk > commands > config_cmd**

# Module: commands::config_cmd

## Contents

**Enums**

- [`ConfigCommand`](#configcommand) - `bookclerk config` subcommands for settings, secrets, templates, and database migrate.
- [`DatabaseCommand`](#databasecommand) - Nested `config database` actions for copying rows between backend plugins.
- [`MasterKeyCommand`](#masterkeycommand) - Nested `config master-key` actions for inspecting or wrapping `master.key`.
- [`TemplateCommand`](#templatecommand) - Nested `config template` actions for naming tags, profiles, and path preview.

**Functions**

- [`run`](#run) - Dispatches a `config` subcommand against the loaded configuration.

---

## bookclerk::commands::config_cmd::ConfigCommand

*Enum*

`bookclerk config` subcommands for settings, secrets, templates, and database migrate.

**Variants:**
- `Get{ key: Option<String>, bare: bool }` - Print a configuration value by dotted key or classic Settings.json name.
- `Set{ key: String, value: String }` - Set a configuration value and write `config.toml`.
- `Show` - Print the effective configuration as TOML-ish summary.
- `Paths` - Print resolved filesystem paths.
- `MasterKey{ command: MasterKeyCommand }` - Inspect or wrap `{files_dir}/master.key` (BCK1 ↔ BCK2).
- `Template{ command: TemplateCommand }` - Naming template helpers.
- `Database{ command: DatabaseCommand }` - Database backend helpers (plugin switch / migration).

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



## bookclerk::commands::config_cmd::DatabaseCommand

*Enum*

Nested `config database` actions for copying rows between backend plugins.

**Variants:**
- `Migrate{ from: Option<String>, to: String, dry_run: bool, force: bool, apply: bool }` - Copy library data from one `[database].plugin` backend to another.

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



## bookclerk::commands::config_cmd::MasterKeyCommand

*Enum*

Nested `config master-key` actions for inspecting or wrapping `master.key`.

**Variants:**
- `Status` - Show whether `master.key` exists and if it is raw (BCK1) or wrapped (BCK2).
- `Wrap` - Wrap a BCK1 `master.key` with a passphrase (no-op unlock if already BCK2).

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



## bookclerk::commands::config_cmd::TemplateCommand

*Enum*

Nested `config template` actions for naming tags, profiles, and path preview.

**Variants:**
- `Tags` - List supported naming template property tags.
- `Profiles` - List built-in naming profiles and their templates.
- `Preview{ asin: String, account: Option<String>, profile: Option<String>, folder: Option<String>, file: Option<String>, ext: String }` - Preview folder/file templates for a library title.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Subcommand**
  - `fn augment_subcommands<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn augment_subcommands_for_update<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn has_subcommand(__clap_name: &str) -> bool`
- **FromArgMatches**
  - `fn from_arg_matches(__clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn from_arg_matches_mut(__clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn update_from_arg_matches(self: & mut Self, __clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
  - `fn update_from_arg_matches_mut<'b>(self: & mut Self, __clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`



## bookclerk::commands::config_cmd::run

*Function*

Dispatches a `config` subcommand against the loaded configuration.

```rust
fn run(command: ConfigCommand, config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



