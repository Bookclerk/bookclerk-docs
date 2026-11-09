# bookclerk_db_testkit::fixtures

Realistic row contents sized against a capability advertisement.

Short ASCII test rows never reach a cap. These helpers build text the
way real catalogs do (multibyte descriptions, wide JSON documents) and
size it from the advertisement, so a test that passes under
`crate::Profile::Minimum` has exercised the limits.

| Item | Kind | Summary |
| --- | --- | --- |
| [`multibyte_text`](#multibyte_text) | function | Multibyte text of at most `max_bytes` UTF-8 bytes, cut on a character boundary. |
| [`description_near_cell_cap`](#description_near_cell_cap) | function | A description that fits one cell with `headroom` bytes to spare under `caps.max_cell_bytes`. |
| [`description_over_cell_cap`](#description_over_cell_cap) | function | A description that is one byte over `caps.max_cell_bytes`. |
| [`wide_json`](#wide_json) | function | A JSON object of about `bytes` bytes keyed by `key`: a title, a multibyte description and ASCII padding to the size. |
| [`rows_over_result_cap`](#rows_over_result_cap) | function | `count` `(key, json)` rows whose JSON together exceeds `caps.max_result_bytes`, so a listing of them all does not fit one inline result, while each row's JSON fits one cell under `caps.max_cell_bytes`. |

## Functions

### multibyte_text

```rust
pub fn multibyte_text(max_bytes: usize) -> String
```

Multibyte text of at most `max_bytes` UTF-8 bytes, cut on a character
boundary.

Repeats a mixed-script sentence so byte and character counts differ:
a cap counted in characters instead of bytes lets this through.

### description_near_cell_cap

```rust
pub fn description_near_cell_cap(caps: &DbCapabilities, headroom: usize) -> String
```

A description that fits one cell with `headroom` bytes to spare under
`caps.max_cell_bytes`.

### description_over_cell_cap

```rust
pub fn description_over_cell_cap(caps: &DbCapabilities) -> String
```

A description that is one byte over `caps.max_cell_bytes`.

Padded with ASCII, so the length is exact.

### wide_json

```rust
pub fn wide_json(key: &str, bytes: usize) -> String
```

A JSON object of about `bytes` bytes keyed by `key`: a title, a
multibyte description and ASCII padding to the size.

`key` and the title are serialized as JSON strings, so a key holding a
quote or a backslash still produces a valid document with that key.

### rows_over_result_cap

```rust
pub fn rows_over_result_cap(
    caps: &DbCapabilities,
    prefix: &str,
    count: usize,
) -> Vec<(String, String)>
```

`count` `(key, json)` rows whose JSON together exceeds
`caps.max_result_bytes`, so a listing of them all does not fit one
inline result, while each row's JSON fits one cell under
`caps.max_cell_bytes`.

Keys are `prefix` followed by a zero-padded index, so they sort in
insertion order.

#### Panics

Panics when `count` rows of at most one cell each cannot exceed the
result cap, so a test asking for too few rows fails at the fixture
instead of passing on a listing that never reached the cap, and when a
generated row does not fit a cell after all.
