# bookclerk_plugin_echo_native_rust

Reference Echo integration (native Rust): the parts its tests drive.

[`notes`](notes.md) keeps a `notes` table on the plugin's own `[[databases]]`
binding, written against the database limits; [`cli`](cli.md) is the `cli`
entrypoint that exposes it. `src/main.rs` serves them with the other
entrypoints.

## Modules

- [`cli`](cli.md): The `cli` entrypoint: `ping` plus the notes commands on the `DB` binding.
- [`notes`](notes.md): A small `notes` table on the plugin's own `[[databases]]` binding.
