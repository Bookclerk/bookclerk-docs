**bookclerk > commands > diagnostics_cmd**

# Module: commands::diagnostics_cmd

## Contents

**Enums**

- [`DiagnosticsCommand`](#diagnosticscommand) - `bookclerk diagnostics` subcommands: print the ring buffer or request an upload.

**Functions**

- [`run`](#run) - Dispatches a diagnostics subcommand against the process-global ring buffer.

---

## bookclerk::commands::diagnostics_cmd::DiagnosticsCommand

*Enum*

`bookclerk diagnostics` subcommands: print the ring buffer or request an upload.

**Variants:**
- `Show{ limit: usize }` - Print recent diagnostics ring-buffer events.
- `Upload{ trigger: String }` - Upload the current diagnostics ring (requires share_reports + collector URL).

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



## bookclerk::commands::diagnostics_cmd::run

*Function*

Dispatches a diagnostics subcommand against the process-global ring buffer.

```rust
fn run(command: DiagnosticsCommand, _config: &bookclerk_config::Config, format: crate::format_out::OutputFormat) -> anyhow::Result<()>
```



