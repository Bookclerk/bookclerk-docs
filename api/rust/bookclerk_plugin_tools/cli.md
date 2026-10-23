**bookclerk_plugin_tools > cli**

# Module: cli

## Contents

**Functions**

- [`run`](#run) - Runs the authoring CLI (`check` / `fmt` / `sync-embed` / `package` / `smoke`).

---

## bookclerk_plugin_tools::cli::run

*Function*

Runs the authoring CLI (`check` / `fmt` / `sync-embed` / `package` / `smoke`).

Reads `std::env::args` after the binary name. Unknown commands or missing
required flags print usage to stderr and return exit code `2`.

# Returns

Process [`ExitCode`]: `0` on success, `1` on command failure, `2` on usage
errors.

# Examples

```ignore
// from src/bin/bookclerk-plugin.rs
fn main() -> std::process::ExitCode {
    bookclerk_plugin_tools::run_tools_cli()
}
```

```rust
fn run() -> std::process::ExitCode
```



