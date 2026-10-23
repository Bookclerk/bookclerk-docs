**bookclerk > commands > events**

# Module: commands::events

## Contents

**Enums**

- [`EventsCommand`](#eventscommand) - `bookclerk events` verbs that call bookclerkd's `/api/events` surface.

**Functions**

- [`run`](#run) - Dispatches an events verb against a running bookclerkd.

---

## bookclerk::commands::events::EventsCommand

*Enum*

`bookclerk events` verbs that call bookclerkd's `/api/events` surface.

**Variants:**
- `List` - GET /api/events — recent outbox envelopes.
- `DeadLetters` - GET /api/events/deliveries?state=dead_letter
- `Retry{ id: String }` - POST /api/events/deliveries/{id}/retry
- `Ack{ id: String }` - POST /api/events/deliveries/{id}/acknowledge
- `Cancel{ id: String }` - POST /api/events/deliveries/{id}/cancel
- `Resume{ id: String }` - POST /api/events/deliveries/{id}/resume

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



## bookclerk::commands::events::run

*Function*

Dispatches an events verb against a running bookclerkd.

```rust
fn run(command: EventsCommand, config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



