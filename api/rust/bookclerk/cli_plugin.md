**bookclerk > cli_plugin**

# Module: cli_plugin

## Contents

**Functions**

- [`find_command`](#find_command) - Find a command spec by name.
- [`matches_to_invoke_args`](#matches_to_invoke_args) - Extract invoke args for `command` from clap matches under that subcommand.
- [`schema_to_command`](#schema_to_command) - Convert a plugin CLI schema into a clap command named `plugin_id`.

**Constants**

- [`RESERVED_PLUGIN_SUBCOMMANDS`](#reserved_plugin_subcommands) - Reserved host subcommands under `bookclerk plugins` (cannot be plugin ids).

---

## bookclerk::cli_plugin::RESERVED_PLUGIN_SUBCOMMANDS

*Constant*: `&[&str]`

Reserved host subcommands under `bookclerk plugins` (cannot be plugin ids).



## bookclerk::cli_plugin::find_command

*Function*

Find a command spec by name.

```rust
fn find_command<'a>(schema: &'a bookclerk_plugin_host::CliSchema, name: &str) -> Option<&'a bookclerk_plugin_host::CliCommandSpec>
```



## bookclerk::cli_plugin::matches_to_invoke_args

*Function*

Extract invoke args for `command` from clap matches under that subcommand.

Values travel as their string form (`CliArg::value`); the guest parses
per `CliArgSpec::kind`.

```rust
fn matches_to_invoke_args(spec: &bookclerk_plugin_host::CliCommandSpec, matches: &clap::ArgMatches) -> anyhow::Result<Vec<bookclerk_plugin_host::CliArg>>
```



## bookclerk::cli_plugin::schema_to_command

*Function*

Convert a plugin CLI schema into a clap command named `plugin_id`.

```rust
fn schema_to_command(plugin_id: &str, about: Option<&str>, schema: &bookclerk_plugin_host::CliSchema) -> clap::Command
```



