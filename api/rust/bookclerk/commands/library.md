**bookclerk > commands > library**

# Module: commands::library

## Contents

**Enums**

- [`LibraryCommand`](#librarycommand) - `bookclerk library` subcommands for scan, acquire, accounts, and export.

**Functions**

- [`run`](#run) - Dispatches a `library` subcommand against the opened library store.

---

## bookclerk::commands::library::LibraryCommand

*Enum*

`bookclerk library` subcommands for scan, acquire, accounts, and export.

**Variants:**
- `Accounts{ source: Option<String>, bare: bool }` - List configured bookstore accounts.
- `SetScan{ account: String, source: Option<String>, scan: bool }` - Enable or disable an account for library scans (GUI: Include in library scan).
- `Status{ source: Option<String> }` - Show token validity / refresh health across sources.
- `RevokeAccount{ account: String, source: Option<String> }` - Remove store credentials but keep acquired books and account rows.
- `Scan{ account: Option<String>, accounts: Vec<String>, source: Option<String>, instance: Option<String>, match_storage: bool, fix_layout: bool, no_wait: bool }` - Sync the library from the content sources: queues one scan job per
- `Acquire{ asin: Option<String>, isbn: Option<String>, asins: Vec<String>, account: Option<String>, instance: Option<String>, dry_run: bool, force: bool, pdf: bool, no_wait: bool }` - Download + decrypt + store titles: queues one acquire job per title
- `SetStatus{ account: Option<String>, keep_missing: bool, downloaded: bool, not_downloaded: bool, force: bool, fix_layout: bool, asins: Vec<String> }` - Match storage files to library rows and update acquire status.
- `GetLicense{ asin: String, account: Option<String>, json: bool, full: bool }` - Fetch a content license for an ASIN.
- `Search{ query: String, limit: usize, bare: bool, rebuild_index: bool, filter: Option<String> }` - Search the library index.
- `Export{ path: std::path::PathBuf, csv: bool, json: bool, xlsx: bool, asins: Vec<String>, account: Option<String> }` - Export library rows as CSV / JSON / XLSX (alias of `export library`).
- `List{ account: Option<String>, status: Option<String> }` - List books in the local library DB.
- `Convert{ account: Option<String>, force: bool, asins: Vec<String> }` - Convert acquired m4b/m4a to mp3.
- `Filters{ command: FilterCommand }` - Manage saved quick filters.

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



## bookclerk::commands::library::run

*Function*

Dispatches a `library` subcommand against the opened library store.

```rust
fn run(command: LibraryCommand, config: &bookclerk_config::Config) -> anyhow::Result<()>
```



