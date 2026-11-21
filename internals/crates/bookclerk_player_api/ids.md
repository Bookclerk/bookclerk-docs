# bookclerk_player_api::ids

The ids the `/player` API hands out and reads back.

- A **library** is the whole library the patron may see
  ([`WHOLE_LIBRARY_ID`](#whole_library_id)); collections join as libraries of their own
  once the library has them.
- An **item** is an edition with a stored copy: its id is the edition
  id.
- A **file** of an item is one of its audio tracks: its `ino` is the
  track's place in play order, in decimal.
- An **author**, **series** or **narrator** is a value of the library's
  page index ([`facet_id`](#facet_id)): its key in URL-safe base64, so the same
  name is the same id wherever it appears, and a filter by it reads the
  index directly.

| Item | Kind | Summary |
| --- | --- | --- |
| [`is_entity_id`](#is_entity_id) | function | True when `id` has the shape of an id this library hands out: 1 to `MAX_ID_BYTES` bytes of ASCII letters, digits, `-`, `_`, `.` and `:`. Anything else is answered as unknown without a lookup. |
| [`facet_id`](#facet_id) | function | The id of a page index value (an author, series or narrator): its key (lowercase, trimmed) in URL-safe base64 without padding. |
| [`parse_facet_id`](#parse_facet_id) | function | The page index key a `facet_id` names. |
| [`parse_file_ino`](#parse_file_ino) | function | The track a file `ino` names: its index in play order. |
| [`file_ino`](#file_ino) | function | The `ino` of the track at `index`. |
| [`WHOLE_LIBRARY_ID`](#whole_library_id) | constant | The library id of the whole library the patron may see. |
| [`MAX_ID_BYTES`](#max_id_bytes) | constant | The longest id the API reads back (an edition, author, series or bookmark id), in bytes: room for a `facet_id` of the longest key. |
| [`MAX_FACET_KEY_BYTES`](#max_facet_key_bytes) | constant | The longest page index key a `facet_id` carries, in bytes (the page index's own bound on a value). |
| [`MAX_TRACKS`](#max_tracks) | constant | Most tracks an item lists; the playback core's own bound. |

## Functions

### is_entity_id

```rust
pub fn is_entity_id(id: &str) -> bool
```

True when `id` has the shape of an id this library hands out: 1 to
[`MAX_ID_BYTES`](#max_id_bytes) bytes of ASCII letters, digits, `-`, `_`, `.` and
`:`. Anything else is answered as unknown without a lookup.

### facet_id

```rust
pub fn facet_id(key: &str) -> String
```

The id of a page index value (an author, series or narrator): its key
(lowercase, trimmed) in URL-safe base64 without padding.

#### Arguments

* `key` - The value's key, at most [`MAX_FACET_KEY_BYTES`](#max_facet_key_bytes) bytes.

### parse_facet_id

```rust
pub fn parse_facet_id(id: &str) -> Option<String>
```

The page index key a [`facet_id`](#facet_id) names.

#### Returns

`None` unless `id` is an entity id that decodes to 1 to
[`MAX_FACET_KEY_BYTES`](#max_facet_key_bytes) bytes of UTF-8 with no control character.

### parse_file_ino

```rust
pub fn parse_file_ino(ino: &str) -> Option<usize>
```

The track a file `ino` names: its index in play order.

#### Arguments

* `ino` - The path segment an app sent.

#### Returns

`None` unless `ino` is a canonical decimal (no sign, no leading zero)
below [`MAX_TRACKS`](#max_tracks).

### file_ino

```rust
pub fn file_ino(index: usize) -> String
```

The `ino` of the track at `index`.

## Constants

### WHOLE_LIBRARY_ID

```rust
pub const WHOLE_LIBRARY_ID: &str = "bookclerk-library";
```

The library id of the whole library the patron may see.

### MAX_ID_BYTES

```rust
pub const MAX_ID_BYTES: usize = 344;
```

The longest id the API reads back (an edition, author, series or
bookmark id), in bytes: room for a [`facet_id`](#facet_id) of the longest key.

### MAX_FACET_KEY_BYTES

```rust
pub const MAX_FACET_KEY_BYTES: usize = 256;
```

The longest page index key a [`facet_id`](#facet_id) carries, in bytes (the page
index's own bound on a value).

### MAX_TRACKS

```rust
pub const MAX_TRACKS: usize = 512;
```

Most tracks an item lists; the playback core's own bound.
