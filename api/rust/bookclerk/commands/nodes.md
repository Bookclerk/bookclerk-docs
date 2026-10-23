**bookclerk > commands > nodes**

# Module: commands::nodes

## Contents

**Enums**

- [`InvitesCommand`](#invitescommand) - `bookclerk nodes invites` verbs.
- [`NodesCommand`](#nodescommand) - `bookclerk nodes` verbs.

**Functions**

- [`run`](#run) - Dispatches a nodes verb.

---

## bookclerk::commands::nodes::InvitesCommand

*Enum*

`bookclerk nodes invites` verbs.

**Variants:**
- `List` - Every enrollment link and its state. Secrets are never shown again.
- `Revoke{ link_id: String }` - Revoke a link that was not redeemed yet.

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



## bookclerk::commands::nodes::NodesCommand

*Enum*

`bookclerk nodes` verbs.

**Variants:**
- `List` - Every registered node: liveness, artifact target, and labels.
- `Set{ node: Option<String>, cpu_budget: Option<u8>, memory_budget: Option<u8>, drain: bool, undrain: bool, label: Option<String> }` - Set a node's scheduling budget or drain it. Applies at that node's
- `Label{ node: Option<String>, add: Vec<String>, remove: Vec<String>, clear: bool }` - Add or remove placement labels on a node (`nas`, `zone=eu`).
- `Owner{ node: Option<String>, user: Option<String>, clear: bool }` - Assign a node to the user who administers it, or clear its owner. The
- `Invite{ user: String, ttl: String, label: String }` - Mint a single-use link that enrolls one host node for a user. The
- `Invites{ command: InvitesCommand }` - List or revoke enrollment links.
- `Revoke{ node: String }` - Revoke an enrolled host node's credential. Its connection closes and

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



## bookclerk::commands::nodes::run

*Function*

Dispatches a nodes verb.

# Errors

Returns an error when the library cannot be opened, the node is unknown,
or a label is invalid.

```rust
fn run(command: NodesCommand, config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



