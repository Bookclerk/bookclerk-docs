# bookclerk_plugin_manifest::cron

Five-field cron schedules for `[triggers] crons` (Workers `scheduled`).

A [`CronSchedule`](#cronschedule) is parsed from an untrusted `plugin.toml` string, so
parsing is sans-IO and bounded: at most [`MAX_CRON_SOURCE_BYTES`](#max_cron_source_bytes) bytes,
exactly five whitespace-separated fields, and every value range-checked.
Times are UTC, as in Cloudflare Workers.

| Field | Values | Names |
| --- | --- | --- |
| minute | `0-59` | |
| hour | `0-23` | |
| day of month | `1-31` | |
| month | `1-12` | `JAN`-`DEC` |
| day of week | `0-7` (`0` and `7` are Sunday) | `SUN`-`SAT` |

Each field is a comma list of items. An item is `*`, a value `n`, or a
range `a-b`, optionally followed by `/step`; `n/step` means `n` to the
field maximum. Names are case-insensitive. Quartz extensions (`L`, `W`,
`#`, `?`) and `@daily`-style macros are rejected.

When both day of month and day of week are restricted (neither field
starts with `*`), a day matches if **either** matches, as in Vixie cron
and Workers. A schedule that can never fire (`0 0 30 2 *`) is rejected.

| Item | Kind | Summary |
| --- | --- | --- |
| [`UnixMinute`](#unixminute) | struct | One minute on the UTC timeline, counted from the Unix epoch. |
| [`CronSchedule`](#cronschedule) | struct | A validated five-field cron schedule. |
| [`CronError`](#cronerror) | struct | Why a cron string was rejected. |
| [`MAX_CRON_SOURCE_BYTES`](#max_cron_source_bytes) | constant | Longest accepted cron source text, in bytes. |

## Structs

### UnixMinute

```rust
pub struct UnixMinute(pub i64);
```

One minute on the UTC timeline, counted from the Unix epoch.

Cron schedules have minute resolution, so every fire time is a whole
minute. Negative values are minutes before 1970-01-01T00:00Z.

#### UnixMinute::from_unix_ms

```rust
pub fn from_unix_ms(unix_ms: i64) -> Self
```

The minute containing `unix_ms` (floored, so it never rounds up).

##### Arguments

* `unix_ms` - Milliseconds since the Unix epoch.

#### UnixMinute::to_unix_ms

```rust
pub fn to_unix_ms(self) -> Option<i64>
```

Milliseconds since the Unix epoch at the start of this minute.

##### Returns

`None` when the result does not fit an `i64`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### CronSchedule

```rust
pub struct CronSchedule {
    /* private fields */
}
```

A validated five-field cron schedule.

Equality compares the normalized source (fields joined by one space),
which fully determines the parsed fields.

#### CronSchedule::parse

```rust
pub fn parse(text: &str) -> Result<Self, CronError>
```

Parses and validates a five-field cron expression.

##### Arguments

* `text` - Cron source from `plugin.toml` (`*/30 * * * *`).

##### Returns

The schedule, with its source normalized to single spaces.

##### Errors

Returns [`CronError`](#cronerror) when the text is too long, not ASCII, does not
have exactly five fields, has a value, range or step out of bounds,
or describes a schedule that can never fire.

#### CronSchedule::as_str

```rust
pub fn as_str(&self) -> &str
```

Normalized source text: the five fields joined by one space.

This is the `cron` string a `scheduled` handler receives.

#### CronSchedule::matches

```rust
pub fn matches(&self, minute: UnixMinute) -> bool
```

True when the schedule fires at `minute`.

##### Arguments

* `minute` - A minute on the UTC timeline.

#### CronSchedule::next_after

```rust
pub fn next_after(&self, after: UnixMinute) -> Option<UnixMinute>
```

The first fire time strictly after `after`.

##### Arguments

* `after` - Exclusive lower bound.

##### Returns

`None` when the schedule does not fire in the next nine years (only
possible near the end of the representable timeline, since
[`Self::parse`](#cronscheduleparse) rejects schedules that never fire).

#### CronSchedule::latest_at_or_before

```rust
pub fn latest_at_or_before(&self, at: UnixMinute) -> Option<UnixMinute>
```

The latest fire time at or before `at`.

The scheduler uses this to coalesce missed fires: after downtime a
schedule fires once, for the newest time it missed.

##### Arguments

* `at` - Inclusive upper bound.

##### Returns

`None` when the schedule did not fire in the nine years before `at`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `From<CronSchedule>`, `FromStr`, `Hash`, `PartialEq`, `Serialize`, `TryFrom<String>`

### CronError

```rust
pub struct CronError(/* private */);
```

Why a cron string was rejected.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Constants

### MAX_CRON_SOURCE_BYTES

```rust
pub const MAX_CRON_SOURCE_BYTES: usize = _;
```

Longest accepted cron source text, in bytes.
