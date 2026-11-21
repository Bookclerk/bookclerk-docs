# bookclerk_player_api::tracks

How long each audio file of an item plays, for the track list apps
seek across.

The playback core knows each file's size and, once probed, its average
bitrate; the catalog knows the whole runtime to the minute. Apps need
each track's start and duration in seconds, so a file's duration comes
from its bitrate when every file has one, and otherwise from its share
of the bytes of the whole runtime.

| Item | Kind | Summary |
| --- | --- | --- |
| [`TrackSize`](#tracksize) | struct | One file to time: its size and its probed average bitrate. |
| [`track_durations_ms`](#track_durations_ms) | function | Each track's duration in milliseconds, in order. |
| [`track_starts_ms`](#track_starts_ms) | function | Each track's start in milliseconds: the sum of the durations before it. |

## Structs

### TrackSize

```rust
pub struct TrackSize {
    pub bytes: u64,
    pub bitrate_kbps: Option<u32>,
}
```

One file to time: its size and its probed average bitrate.

**Fields**

- `bytes`: Size in bytes.
- `bitrate_kbps`: Average bitrate in kilobits per second, when probed (`0` counts as
  not probed).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### track_durations_ms

```rust
pub fn track_durations_ms(tracks: &[TrackSize], runtime_ms: Option<u64>) -> Vec<u64>
```

Each track's duration in milliseconds, in order.

#### Arguments

* `tracks` - The files in play order.
* `runtime_ms` - The whole runtime the catalog states, if known.

#### Returns

From the bitrates when every file has one; else each file's share of
the bytes of `runtime_ms`; else zeros. Each value is at most
[`MAX_MS`](time.md#max_ms), and the shares add up to the runtime (the last track takes
the remainder).

### track_starts_ms

```rust
pub fn track_starts_ms(durations_ms: &[u64]) -> Vec<u64>
```

Each track's start in milliseconds: the sum of the durations before it.

#### Arguments

* `durations_ms` - The durations, in order.
