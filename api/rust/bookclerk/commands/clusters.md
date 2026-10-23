**bookclerk > commands > clusters**

# Module: commands::clusters

## Contents

**Enums**

- [`ClustersCommand`](#clusterscommand) - `bookclerk clusters` verbs.
- [`JoinAs`](#joinas) - `clusters join --as`.

**Functions**

- [`run`](#run) - Runs a `clusters` verb.

---

## bookclerk::commands::clusters::ClustersCommand

*Enum*

`bookclerk clusters` verbs.

**Variants:**
- `List` - The owner clusters this install is enrolled in as a host.
- `Join{ url: Option<String>, name: Option<String>, fingerprint: Option<String>, join_as: JoinAs, argv: Vec<std::ffi::OsString> }` - Enroll this install as a host (a branch) of an owner library.
- `Leave{ cluster: Option<String> }` - Leave an owner cluster: the owner is told (and revokes this node
- `Consents{ cluster: Option<String> }` - What each owner cluster asks to run here, and whether it may.
- `Approve{ plugin: String, cluster: Option<String>, request_digest: String, deny_domains: Vec<String>, deny_bindings: Vec<String>, deny_network: bool }` - Approve an owner plugin's request to run here, optionally narrowed.
- `Scan{ cluster: Option<String>, account: Option<String>, instance: Option<String>, no_wait: bool }` - Ask an owner cluster to scan its library for this node's owner.
- `Revoke{ plugin: String, cluster: Option<String> }` - Withdraw an approval: held jobs of the plugin are cancelled and

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



## bookclerk::commands::clusters::JoinAs

*Enum*

`clusters join --as`.

**Variants:**
- `Branch` - A branch (host node) with its own database.
- `Clerk` - A clerk (owner node) sharing the library's database and key.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &JoinAs) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> JoinAs`
- **ValueEnum**
  - `fn value_variants<'a>() -> &'a [Self]`
  - `fn to_possible_value<'a>(self: &Self) -> ::std::option::Option<clap::builder::PossibleValue>`



## bookclerk::commands::clusters::run

*Function*

Runs a `clusters` verb.

# Errors

Returns an error when the database cannot be opened, a read or write
fails, or an approval is refused.

```rust
fn run(command: ClustersCommand, config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



