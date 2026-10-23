**bookclerk > commands > import_cmd**

# Module: commands::import_cmd

## Contents

**Enums**

- [`ImportCommand`](#importcommand) - `bookclerk import` subcommands: native `.tar.gz` restore or classic Libation Files.

**Functions**

- [`run`](#run) - Runs native or Libation import and prints a summary (or JSON via `--format`).

---

## bookclerk::commands::import_cmd::ImportCommand

*Enum*

`bookclerk import` subcommands: native `.tar.gz` restore or classic Libation Files.

**Variants:**
- `Native{ from: std::path::PathBuf, force: bool, dry_run: bool }` - Restore a native Bookclerk `.tar.gz` backup into the files directory.
- `Libation{ from: std::path::PathBuf, force: bool, skip_auth: bool, dry_run: bool }` - Import classic Libation Files (Settings / accounts / DB).

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



## bookclerk::commands::import_cmd::run

*Function*

Runs native or Libation import and prints a summary (or JSON via `--format`).

```rust
fn run(command: ImportCommand, config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



