**bookclerk > commands > migrate**

# Module: commands::migrate

## Contents

**Enums**

- [`MigrateCommand`](#migratecommand) - Hidden `bookclerk migrate` alias of `import libation` (classic Files directory).

**Functions**

- [`run`](#run) - Forwards `migrate import` to [`import_cmd::run`](import_cmd.md#bookclerkcommandsimport_cmdrun) with text output.

---

## bookclerk::commands::migrate::MigrateCommand

*Enum*

Hidden `bookclerk migrate` alias of `import libation` (classic Files directory).

**Variants:**
- `Import{ from: std::path::PathBuf, force: bool, skip_auth: bool, dry_run: bool }` - Import Settings.json, AccountsSettings.json, and LibationContext.db.

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



## bookclerk::commands::migrate::run

*Function*

Forwards `migrate import` to [`import_cmd::run`](import_cmd.md#bookclerkcommandsimport_cmdrun) with text output.

```rust
fn run(command: MigrateCommand, config: &bookclerk_config::Config) -> anyhow::Result<()>
```



