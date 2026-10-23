**bookclerk > commands > integrations**

# Module: commands::integrations

## Contents

**Enums**

- [`IntegrationsCommand`](#integrationscommand) - `bookclerk integrations` subcommands (status, test, tickets, remote scan).
- [`TicketsCommand`](#ticketscommand) - Claim-ticket subcommands under `bookclerk integrations tickets`.

**Functions**

- [`run`](#run) - Dispatches an integrations CLI command against the loaded registry and library.

---

## bookclerk::commands::integrations::IntegrationsCommand

*Enum*

`bookclerk integrations` subcommands (status, test, tickets, remote scan).

**Variants:**
- `Status` - Show health of configured integrations.
- `Test{ integration: Option<String> }` - Probe connectivity for one (or all) enabled integrations.
- `Tickets{ command: TicketsCommand }` - Claim ticket management.
- `Scan{ integration: String, force: bool }` - Trigger a remote library scan on an integration that supports it.

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



## bookclerk::commands::integrations::TicketsCommand

*Enum*

Claim-ticket subcommands under `bookclerk integrations tickets`.

**Variants:**
- `Create{ provider: String, external_user_id: String, label: Option<String> }` - Mint a claim ticket for an external identity.
- `List` - List open (unredeemed, unexpired) claim tickets.

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



## bookclerk::commands::integrations::run

*Function*

Dispatches an integrations CLI command against the loaded registry and library.

```rust
fn run(command: IntegrationsCommand, config: &bookclerk_config::Config) -> anyhow::Result<()>
```



