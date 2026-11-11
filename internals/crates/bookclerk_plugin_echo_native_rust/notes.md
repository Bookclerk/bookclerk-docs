# bookclerk_plugin_echo_native_rust::notes

A small `notes` table on the plugin's own `[[databases]]` binding.

Written the way plugin database code should be: sized from `limits()`,
split with `chunk()`, read with `iterate()` when the session streams, and
paged by key when it does not. Typed `limit_exceeded` errors are handled
by the cap they name (`PluginError::limit`), never by matching the
message.

`tests/notes.rs` drives this module through
`bookclerk_db_testkit::BindingDatabase` under the profile
`bookclerk-plugin test` names, so the same code is proven on the shipped
adapters, on D1 and on the smallest advertisement a host accepts.

See `docs/plugins/database-limits.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Note`](#note) | struct | One stored note. |
| [`NotesReader`](#notesreader) | struct | Reads the notes in id order without holding them all in memory. |
| [`migrations`](#migrations) | function | The `database_migrations` registration for `NOTES_BINDING`. |
| [`migrations_for`](#migrations_for) | function | The `database_migrations` answer for `binding`. |
| [`note_too_large`](#note_too_large) | function | A note refused before it is sent: storable, but never returnable. |
| [`note_from_row`](#note_from_row) | function | Reads one stored row back into a `Note`. |
| [`add_notes`](#add_notes) | function | Stores `bodies` as new notes. |
| [`count_notes`](#count_notes) | function | Counts the stored notes. |
| [`NOTES_BINDING`](#notes_binding) | constant | The `[[databases]]` binding name in `plugin.toml` that holds the notes. |

## Structs

### Note

```rust
pub struct Note {
    pub id: i64,
    pub body: String,
}
```

One stored note.

**Fields**

- `id`: Identity the database assigned, increasing in insertion order.
- `body`: The note text as written.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NotesReader

```rust
pub struct NotesReader {
    /* private fields */
}
```

Reads the notes in id order without holding them all in memory.

When the session streams (`bookclerk_plugin_sdk::DbCapabilities::streams_results`),
one `iterate()` reads the whole table as a single snapshot, a frame at a
time. When it does not, every read is held to the inline caps, so the
notes are read in pages keyed on `id` (never `OFFSET`), each sized from
`maxResultRows`, halved whenever a page's bytes pass `maxResultBytes`, and
doubled again (up to `maxResultRows`) after each full page, so one run of
wide notes does not slow the rest of the walk. With a limit, no read asks
for more rows than are still wanted.

Dropping the reader ends the read.

#### NotesReader::new

```rust
pub fn new(db: &DatabaseBinding, limit: Option<usize>) -> Self
```

A reader over `db`'s notes, oldest first.

##### Arguments

* `db` - The plugin's notes binding.
* `limit` - The most notes to hand out, or `None` for all of them.

#### NotesReader::next

```rust
pub async fn next(&mut self) -> Option<Result<Note, PluginError>>
```

The next note, `None` once the read is over.

##### Errors

Yields (once, then `None`) the binding's error: `limit_exceeded` when
even the streamed budget is too small or one note alone passes the
inline caps, or `invalid_params` for a row of the wrong shape.

Implements: `Debug`

## Functions

### migrations

```rust
pub fn migrations() -> Vec<PluginMigration>
```

The `database_migrations` registration for [`NOTES_BINDING`](#notes_binding).

Applied once, in order, before the binding is handed to the plugin.

### migrations_for

```rust
pub fn migrations_for(binding: &str) -> Vec<PluginMigration>
```

The `database_migrations` answer for `binding`.

#### Returns

[`migrations`](#migrations) for [`NOTES_BINDING`](#notes_binding), nothing for any other binding.

### note_too_large

```rust
pub fn note_too_large(used: u64, cap: u64) -> PluginError
```

A note refused before it is sent: storable, but never returnable.

A reply cell is held to `maxCellBytes`, so a longer note could be written
under a larger `maxPayloadBytes` and then never read back.

#### Arguments

* `used` - The note's size in UTF-8 bytes.
* `cap` - The session's `maxCellBytes`.

#### Returns

A typed `limit_exceeded` error, read with `PluginError::limit` like the
binding's own.

### note_from_row

```rust
pub fn note_from_row(row: &HashMap<String, DbValue>) -> Result<Note, PluginError>
```

Reads one stored row back into a [`Note`](#note).

#### Errors

Returns `invalid_params` when a column is missing or has an unexpected
type.

### add_notes

```rust
pub async fn add_notes(
    db: &DatabaseBinding,
    bodies: &[String],
    on_committed: impl FnMut(usize),
) -> Result<usize, PluginError>
```

Stores `bodies` as new notes.

Every body is first checked against the session's `maxCellBytes`: a note
the database would accept but could never return is refused up front.
Then `chunk()` splits the inserts into batches that each fit one request
under the session's `maxStatements` and `maxRequestBytes`. Each batch
commits atomically; the loop does not, so when a later batch fails the
notes of the earlier ones stay stored. `on_committed` hears the running
total after each batch, so a caller can say how far a failed write got.

#### Arguments

* `db` - The plugin's notes binding.
* `bodies` - Note texts, stored in order.
* `on_committed` - Called with the number of notes stored so far after
  each batch commits.

#### Returns

How many notes were stored.

#### Errors

Returns `limit_exceeded` before anything is written when a body passes
`maxCellBytes`, and the binding's error otherwise, `limit_exceeded`
included when one statement passes `maxPayloadBytes` or
`maxRequestBytes`.

### count_notes

```rust
pub async fn count_notes(db: &DatabaseBinding) -> Result<i64, PluginError>
```

Counts the stored notes.

#### Errors

Returns the binding's error, or `invalid_params` when the count does not
come back as one INTEGER.

## Constants

### NOTES_BINDING

```rust
pub const NOTES_BINDING: &str = "DB";
```

The `[[databases]]` binding name in `plugin.toml` that holds the notes.
