**bookclerk > commands > queues**

# Module: commands::queues

## Contents

**Enums**

- [`QueuesCommand`](#queuescommand) - `bookclerk queues` verbs that call bookclerkd's `/api/queues` surface.

**Functions**

- [`run`](#run) - Dispatches a queues verb against a running bookclerkd.

---

## bookclerk::commands::queues::QueuesCommand

*Enum*

`bookclerk queues` verbs that call bookclerkd's `/api/queues` surface.

**Variants:**
- `List` - GET /api/queues — every queue with delivery counters and consumers.
- `Inspect{ name: String }` - GET /api/queues/{name} — counters, consumers, recent messages and deliveries.

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



## bookclerk::commands::queues::run

*Function*

Dispatches a queues verb against a running bookclerkd.

# Errors

Returns an error when the daemon is unreachable or answers with a failure.

```rust
fn run(command: QueuesCommand, config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



