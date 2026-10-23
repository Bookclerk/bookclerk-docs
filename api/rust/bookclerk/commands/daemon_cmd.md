**bookclerk > commands > daemon_cmd**

# Module: commands::daemon_cmd

## Contents

**Enums**

- [`DaemonCommand`](#daemoncommand) - `bookclerk daemon` verbs that call bookclerkd or manage the operator token.
- [`TokenCommand`](#tokencommand) - Operator-token subcommands: print the effective token or mint a new one.

**Functions**

- [`run`](#run) - Dispatches a daemon verb: token ops stay local; others call the control plane.
- [`run_login`](#run_login) - Print a loopback operator sign-in URL (requires a running daemon).

---

## bookclerk::commands::daemon_cmd::DaemonCommand

*Enum*

`bookclerk daemon` verbs that call bookclerkd or manage the operator token.

**Variants:**
- `Health` - GET /health
- `Status` - GET /status (or /api/status)
- `Scan{ account: Option<String> }` - POST /scan (or /api/library/scan)
- `Acquire{ asin: Option<String>, account: Option<String> }` - POST /acquire (or /api/library/acquire)
- `Jobs` - GET /jobs (or /api/jobs)
- `Token{ command: Option<TokenCommand> }` - Show or rotate the operator API token (DB-backed; env override wins).

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



## bookclerk::commands::daemon_cmd::TokenCommand

*Enum*

Operator-token subcommands: print the effective token or mint a new one.

**Variants:**
- `Show` - Print the current effective operator token (default).
- `Rotate` - Mint a new token, store it in encrypted_secrets, and reload the daemon when reachable.

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



## bookclerk::commands::daemon_cmd::run

*Function*

Dispatches a daemon verb: token ops stay local; others call the control plane.

```rust
fn run(command: DaemonCommand, config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



## bookclerk::commands::daemon_cmd::run_login

*Function*

Print a loopback operator sign-in URL (requires a running daemon).

Uses the same `POST /api/auth/tray-handoff/prepare` ticket as the tray Copy
sign-in link. Does not print the durable operator token.

```rust
fn run_login(config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



