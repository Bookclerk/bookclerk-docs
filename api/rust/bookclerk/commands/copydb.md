**bookclerk > commands > copydb**

# Module: commands::copydb

## Contents

**Structs**

- [`CopyDbArgs`](#copydbargs) - CLI flags for copying `library.db` into PostgreSQL.

**Enums**

- [`CopyDbFormat`](#copydbformat) - Destination schema for `bookclerk export postgres` (classic Libation vs native flat).

**Functions**

- [`run`](#run) - Opens the source SQLite read-only and writes the chosen schema to PostgreSQL.

---

## bookclerk::commands::copydb::CopyDbArgs

*Struct*

CLI flags for copying `library.db` into PostgreSQL.

**Fields:**
- `connection: String` - PostgreSQL connection string.
- `source: Option<std::path::PathBuf>` - Source SQLite path (default: `{files_dir}/library.db`).
- `format: CopyDbFormat` - Output schema format.

**Trait Implementations:**

- **Args**
  - `fn group_id() -> Option<clap::Id>`
  - `fn augment_args<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn augment_args_for_update<'b>(__clap_app: clap::Command) -> clap::Command`
- **FromArgMatches**
  - `fn from_arg_matches(__clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn from_arg_matches_mut(__clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn update_from_arg_matches(self: & mut Self, __clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
  - `fn update_from_arg_matches_mut(self: & mut Self, __clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk::commands::copydb::CopyDbFormat

*Enum*

Destination schema for `bookclerk export postgres` (classic Libation vs native flat).

**Variants:**
- `Classic` - Classic Libation EF / Postgres schema.
- `Flat` - Native bookclerk flat schema.

**Traits:** Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> CopyDbFormat`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **ValueEnum**
  - `fn value_variants<'a>() -> &'a [Self]`
  - `fn to_possible_value<'a>(self: &Self) -> ::std::option::Option<clap::builder::PossibleValue>`
- **Clone**
  - `fn clone(self: &Self) -> CopyDbFormat`



## bookclerk::commands::copydb::run

*Function*

Opens the source SQLite read-only and writes the chosen schema to PostgreSQL.

```rust
fn run(args: CopyDbArgs, config: &bookclerk_config::Config) -> anyhow::Result<()>
```



