**bookclerk > commands > export_cmd**

# Module: commands::export_cmd

## Contents

**Enums**

- [`ExportCommand`](#exportcommand) - `bookclerk export` subcommands: native backup, Libation, spreadsheet, or Postgres copy.

**Functions**

- [`default_postgres_format`](#default_postgres_format) - Default postgres format for the redesigned CLI (native flat schema).
- [`run`](#run) - Dispatches an export subcommand and prints a human or JSON summary.

---

## bookclerk::commands::export_cmd::ExportCommand

*Enum*

`bookclerk export` subcommands: native backup, Libation, spreadsheet, or Postgres copy.

**Variants:**
- `Native{ path: std::path::PathBuf, include_plugin_manifests: bool, include_cache: bool, include_logs: bool, include_plugin_databases: bool }` - Write a portable Bookclerk `.tar.gz` backup of the files directory.
- `Libation{ path: std::path::PathBuf, force: bool, dry_run: bool }` - Export Settings.json, AccountsSettings.json, and LibationContext.db.
- `Library{ path: std::path::PathBuf, csv: bool, json: bool, xlsx: bool, asins: Vec<String>, account: Option<String> }` - Export library rows as CSV / JSON / XLSX.
- `Postgres{ args: crate::commands::copydb::CopyDbArgs }` - Copy library.db to PostgreSQL.

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



## bookclerk::commands::export_cmd::default_postgres_format

*Function*

Default postgres format for the redesigned CLI (native flat schema).

```rust
fn default_postgres_format() -> crate::commands::copydb::CopyDbFormat
```



## bookclerk::commands::export_cmd::run

*Function*

Dispatches an export subcommand and prints a human or JSON summary.

```rust
fn run(command: ExportCommand, config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



