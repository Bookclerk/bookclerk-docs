# bookclerk_plugin_echo_native_rust::cli

The `cli` entrypoint: `ping` plus the notes commands on the `DB` binding.

| Item | Kind | Summary |
| --- | --- | --- |
| [`NotesCli`](#notescli) | struct | The `cli` entrypoint, holding the `DB` binding the host granted. |
| [`cli_schema`](#cli_schema) | function | CLI schema advertised at `PluginCli.describe`. |
| [`whole_number`](#whole_number) | function | Parses a whole-number CLI argument within `1..=maximum`. |
| [`database_failure`](#database_failure) | function | A CLI reply for a failed database call. |
| [`MAX_NOTES_PER_ADD`](#max_notes_per_add) | constant | Largest `--count` `notes-add` accepts, so one command stays a modest write. |
| [`MAX_NOTES_PER_LIST`](#max_notes_per_list) | constant | Largest `--limit` `notes-list` accepts. |

## Structs

### NotesCli

```rust
pub struct NotesCli {
    /* private fields */
}
```

The `cli` entrypoint, holding the `DB` binding the host granted.

#### NotesCli::new

```rust
pub fn new(db: Option<DatabaseBinding>) -> Self
```

The entrypoint over `db`.

Implements: `PluginCli`

## Functions

### cli_schema

```rust
pub fn cli_schema() -> CliSchema
```

CLI schema advertised at `PluginCli.describe`.

### whole_number

```rust
pub fn whole_number(value: &str, maximum: usize) -> Option<usize>
```

Parses a whole-number CLI argument within `1..=maximum`.

#### Returns

The number, or `None` when `value` is not one to six plain decimal digits
in range.

### database_failure

```rust
pub fn database_failure(
    err: PluginError,
    committed: usize,
) -> Result<CliInvokeResult, PluginError>
```

A CLI reply for a failed database call.

A typed limit names the cap (exit status 1); any other error is passed on
so the host reports it, keeping its code.

#### Arguments

* `err` - What the binding returned.
* `committed` - Notes already stored by batches that committed before the
  failure; they stay stored.

#### Errors

Returns `err` itself when it is not a `limit_exceeded` error and nothing
was stored; otherwise an error with `err`'s code and details whose
message says how many notes were.

## Constants

### MAX_NOTES_PER_ADD

```rust
pub const MAX_NOTES_PER_ADD: usize = 10_000;
```

Largest `--count` `notes-add` accepts, so one command stays a modest write.

### MAX_NOTES_PER_LIST

```rust
pub const MAX_NOTES_PER_LIST: usize = 1_000;
```

Largest `--limit` `notes-list` accepts.

The listing goes back in one CLI reply, so it is bounded even though
[`NotesReader`](notes.md#notesreader) itself streams.
