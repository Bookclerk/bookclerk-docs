**bookclerk > commands > db**

# Module: commands::db

## Contents

**Enums**

- [`BackupCommand`](#backupcommand) - `bookclerk db backup` verbs.
- [`DbCommand`](#dbcommand) - Host schema versioning, backups, and explicit ups/downs (`migrate --to`).

**Functions**

- [`run`](#run) - Dispatches a `bookclerk db` verb.

---

## bookclerk::commands::db::BackupCommand

*Enum*

`bookclerk db backup` verbs.

**Variants:**
- `Create{ path: Option<std::path::PathBuf>, include_plugin_databases: bool }` - Write a manual recovery point under `files_dir/backups/`.
- `List` - List recovery points oldest-first.
- `Verify{ from: String }` - Verify a recovery point (objects, schema, typed rows) without restoring.
- `Prune` - Prune automatic pre-migrate recovery points and unreferenced objects.

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



## bookclerk::commands::db::DbCommand

*Enum*

Host schema versioning, backups, and explicit ups/downs (`migrate --to`).

**Variants:**
- `Version` - Show this binary's frozen plan and the database's explicit schema state.
- `Backup{ command: BackupCommand }` - Canonical Bookclerk backup (logically full recovery points).
- `Restore{ from: String }` - Restore schema and data from a recovery-point id, archive, or timestamp.
- `Migrate{ to: Option<i64>, include_plugin_databases: bool }` - Apply frozen ups or reversible downs to reach `--to`.

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



## bookclerk::commands::db::run

*Function*

Dispatches a `bookclerk db` verb.

```rust
fn run(command: DbCommand, config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



