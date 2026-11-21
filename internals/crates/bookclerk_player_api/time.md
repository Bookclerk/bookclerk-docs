# bookclerk_player_api::time

Lengths of time on the wire and in the playback core.

The apps speak seconds as JSON numbers (fractional, `currentTime:
754.2`); the playback core keeps whole milliseconds, at most sixty days
(`bookclerk_library::MAX_PLAYBACK_MS`). Conversions here are exact in
both directions for every value in range.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ms_to_secs`](#ms_to_secs) | function | Whole milliseconds as wire seconds, exactly. |
| [`secs_to_ms`](#secs_to_ms) | function | Wire seconds as whole milliseconds, rounded to the nearest. |
| [`epoch_ms`](#epoch_ms) | function | Milliseconds since the Unix epoch, as the apps send times of day (`updatedAt`, `startedAt`, `lastUpdate`). |
| [`MAX_MS`](#max_ms) | constant | The longest length of time the API accepts, in milliseconds: sixty days, the playback core's own bound. |
| [`MAX_SECS`](#max_secs) | constant | The longest length of time the API accepts, in seconds. |

## Functions

### ms_to_secs

```rust
pub fn ms_to_secs(ms: u64) -> f64
```

Whole milliseconds as wire seconds, exactly.

#### Arguments

* `ms` - Milliseconds; values past [`MAX_MS`](#max_ms) are clamped to it.

#### Returns

The same length in seconds.

### secs_to_ms

```rust
pub fn secs_to_ms(secs: f64) -> Option<u64>
```

Wire seconds as whole milliseconds, rounded to the nearest.

#### Arguments

* `secs` - A number an app sent.

#### Returns

`None` for a value that is not finite, is negative or is past
[`MAX_SECS`](#max_secs).

### epoch_ms

```rust
pub fn epoch_ms(value: f64) -> Option<i64>
```

Milliseconds since the Unix epoch, as the apps send times of day
(`updatedAt`, `startedAt`, `lastUpdate`).

#### Arguments

* `value` - A number an app sent.

#### Returns

`None` unless the value is a whole, non-negative number of milliseconds
before the year 10000.

## Constants

### MAX_MS

```rust
pub const MAX_MS: u64 = _;
```

The longest length of time the API accepts, in milliseconds: sixty
days, the playback core's own bound.

### MAX_SECS

```rust
pub const MAX_SECS: f64 = 5_184_000.0;
```

The longest length of time the API accepts, in seconds.
