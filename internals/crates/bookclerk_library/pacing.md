# bookclerk_library::pacing

Download pacing per store account: the limits, the operator's override,
and the decision whether a download may start now.

A storefront declares a pace in `plugin.toml` `[acquire]` (a daily
budget over any rolling 24 hours, a minimum gap between downloads, and
the cool-down to apply when the store answers `rateLimited` without a
retry time). The operator may override it per account. Downloads the
host starts itself wait for the pace ([`decide`](#decide)); downloads a person
starts by hand are counted but never held. This module is pure: the
ledger and the override live in the library database
(`LibraryStore::account_pace_state`), and `bookclerkd` holds a paced
acquire job between titles when [`decide`](#decide) says to wait.

See `docs/internals/jobs.md#download-pacing`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AcquirePace`](#acquirepace) | struct | A storefront's declared pace (`[acquire]`), or the limits in force after an override. |
| [`PaceRequest`](#pacerequest) | struct | A change to one account's pace: the body of `POST /api/portal/sources/{source}/connections/{account_id}/pace`. |
| [`DownloadWindow`](#downloadwindow) | struct | What the ledger says about one account's recent downloads. |
| [`PaceOverride`](#paceoverride) | enum | The operator's override of a storefront's pace for one account. |
| [`PaceOverrideError`](#paceoverrideerror) | enum | Why an override was refused. |
| [`WaitReason`](#waitreason) | enum | Why a paced download has to wait. |
| [`PaceVerdict`](#paceverdict) | enum | Whether a paced download may start. |
| [`parse_pace_request`](#parse_pace_request) | function | Parses and checks a pace change request (sans-IO, so the fuzz target and the route run the same code). |
| [`decide`](#decide) | function | Decides whether a paced download on one account may start at `now`. |
| [`cooldown_end`](#cooldown_end) | function | The cool-down end to record after the store answered `rateLimited`. |
| [`BUDGET_WINDOW`](#budget_window) | constant | The window the daily budget counts downloads over. |
| [`LEDGER_RETENTION`](#ledger_retention) | constant | How long a download stays in the ledger; older rows are pruned. Longer than `BUDGET_WINDOW` so a clock step never empties the window early. |
| [`DEFAULT_COOLDOWN`](#default_cooldown) | constant | Cool-down when the store says to slow down without a retry time and the storefront declares no `cooldown_seconds`. |
| [`MAX_COOLDOWN`](#max_cooldown) | constant | Longest cool-down the host records, whatever the store asked for: a store answer that names a later time is capped here, so a bad `retryAfterMs` cannot stop an account for good. |
| [`MAX_DOWNLOADS_PER_DAY`](#max_downloads_per_day) | constant | Most downloads a day an override may set (the manifest's cap too). |
| [`MAX_MIN_GAP_SECONDS`](#max_min_gap_seconds) | constant | Longest gap an override may set, in seconds (one day). |

## Structs

### AcquirePace

```rust
pub struct AcquirePace {
    pub downloads_per_day: Option<NonZeroU32>,
    pub min_gap: Duration,
    pub cooldown: Duration,
}
```

A storefront's declared pace (`[acquire]`), or the limits in force after
an override.

**Fields**

- `downloads_per_day`: Downloads allowed in any rolling [`BUDGET_WINDOW`](#budget_window); `None` for no
  daily budget.
- `min_gap`: Least time between the start of two downloads on one account.
- `cooldown`: Cool-down when the store says to slow down without a retry time.

#### AcquirePace::from_declared

```rust
pub fn from_declared(
    downloads_per_day: Option<i64>,
    min_gap_seconds: Option<i64>,
    cooldown_seconds: Option<i64>,
) -> Self
```

Builds a pace from a manifest's `[acquire]` values.

Values the manifest validation already bounds are taken as they are;
a value out of range here (a manifest built without validation) is
clamped rather than refused, and a non-positive budget means none.

##### Arguments

* `downloads_per_day` - `[acquire].downloads_per_day`.
* `min_gap_seconds` - `[acquire].min_gap_seconds`.
* `cooldown_seconds` - `[acquire].cooldown_seconds`.

#### AcquirePace::with_override

```rust
pub fn with_override(self, override_: &PaceOverride) -> Self
```

The limits in force once `override_` applies to this declared pace.

[`PaceOverride::Unlimited`](#paceoverride) lifts the budget and the gap but keeps
the cool-down: a store that asked to slow down is still obeyed by
automatic downloads until the operator clears the cool-down.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### PaceRequest

```rust
pub struct PaceRequest {
    pub override_: Option<PaceOverride>,
    pub clear_cooldown: bool,
}
```

A change to one account's pace: the body of
`POST /api/portal/sources/{source}/connections/{account_id}/pace`.

**Fields**

- `override_`: The override to store, or `None` for a request that only clears
  the cool-down and leaves the stored pace as it is.
- `clear_cooldown`: End the store's cool-down too.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### DownloadWindow

```rust
pub struct DownloadWindow {
    pub starts: Vec<DateTime<Utc>>,
}
```

What the ledger says about one account's recent downloads.

**Fields**

- `starts`: Start times of the downloads in the last [`BUDGET_WINDOW`](#budget_window), oldest
  first.

#### DownloadWindow::last

```rust
pub fn last(&self) -> Option<DateTime<Utc>>
```

The latest download start, if any.

#### DownloadWindow::count

```rust
pub fn count(&self) -> usize
```

Downloads in the window.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

## Enums

### PaceOverride

```rust
pub enum PaceOverride {
    Default,
    Custom { downloads_per_day: Option<NonZeroU32>, min_gap_seconds: Option<u32> },
    Unlimited,
}
```

The operator's override of a storefront's pace for one account.

**Variants**

- `Default`: The storefront's declared pace.
- `Custom`: Custom values; a field left `None` keeps the storefront's value.
  - `downloads_per_day`: Downloads allowed in any rolling 24 hours.
  - `min_gap_seconds`: Least seconds between two downloads, at most
    [`MAX_MIN_GAP_SECONDS`](#max_min_gap_seconds).
- `Unlimited`: No budget and no gap (the cool-down still applies).

#### PaceOverride::mode_str

```rust
pub fn mode_str(&self) -> &'static str
```

The stored `pace_mode` spelling (`default`, `custom`, `unlimited`).

#### PaceOverride::MODES

```rust
pub const MODES: [&'static str; 3] = _
```

The modes [`Self::mode_str`](#paceoverridemode_str) returns, for the schema's CHECK list.

#### PaceOverride::validate

```rust
pub fn validate(&self) -> Result<(), PaceOverrideError>
```

Checks an override's values against their ranges.

##### Errors

Returns a [`PaceOverrideError`](#paceoverrideerror) naming the first value out of range,
or [`PaceOverrideError::EmptyCustom`](#paceoverrideerror) for a custom override that sets
nothing.

#### PaceOverride::from_row

```rust
pub fn from_row(
    mode: &str,
    downloads_per_day: Option<i64>,
    min_gap_seconds: Option<i64>,
) -> Self
```

Rebuilds an override from its stored row.

An unknown mode or a custom row with any value out of range (a row
written by something other than this crate) falls back to
[`Self::Default`](#paceoverride), so a bad row never lifts the storefront's pace;
a value that is absent leaves that limit unset.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PaceOverrideError

```rust
pub enum PaceOverrideError {
    DownloadsPerDay,
    MinGap,
    EmptyCustom,
}
```

Why an override was refused.

**Variants**

- `DownloadsPerDay`: The daily budget is above [`MAX_DOWNLOADS_PER_DAY`](#max_downloads_per_day).
- `MinGap`: The gap is above [`MAX_MIN_GAP_SECONDS`](#max_min_gap_seconds).
- `EmptyCustom`: A custom override with neither field set.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### WaitReason

```rust
pub enum WaitReason {
    Cooldown,
    DailyBudget,
    Gap,
}
```

Why a paced download has to wait.

**Variants**

- `Cooldown`: The store asked to slow down.
- `DailyBudget`: The rolling 24-hour budget is used up.
- `Gap`: The previous download started less than the gap ago.

#### WaitReason::phrase

```rust
pub fn phrase(self) -> &'static str
```

A short phrase for job progress and logs ("the daily download
budget").

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PaceVerdict

```rust
pub enum PaceVerdict {
    Go,
    Wait { until: DateTime<Utc>, reason: WaitReason },
}
```

Whether a paced download may start.

**Variants**

- `Go`: Start it now.
- `Wait`: Wait until `until`.
  - `until`: When the download may start.
  - `reason`: What it is waiting for (the limit that ends last).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### parse_pace_request

```rust
pub fn parse_pace_request(body: &[u8]) -> Result<PaceRequest>
```

Parses and checks a pace change request (sans-IO, so the fuzz target and
the route run the same code).

The body is one JSON object with a `mode` of `default`, `custom` or
`unlimited`, optional `downloads_per_day` (1 to
[`MAX_DOWNLOADS_PER_DAY`](#max_downloads_per_day)) and `min_gap_seconds` (0 to
[`MAX_MIN_GAP_SECONDS`](#max_min_gap_seconds)) that only `custom` may carry and that it needs
at least one of, and an optional boolean `clear_cooldown`. A body
without `mode` only clears the cool-down: it carries no number and
`clear_cooldown` must be `true`, and the stored pace is left alone. No
other key and no repeated key is accepted; `null` counts as absent.

#### Errors

Returns [`crate::LibraryError::Invalid`](README.md#libraryerror) describing the first rule the
body breaks.

### decide

```rust
pub fn decide(
    pace: &AcquirePace,
    window: &DownloadWindow,
    cooldown_until: Option<DateTime<Utc>>,
    now: DateTime<Utc>,
) -> PaceVerdict
```

Decides whether a paced download on one account may start at `now`.

Every limit that holds is checked and the one that ends last wins, so a
job parked until `until` finds every limit clear when it wakes (unless
another download started meanwhile).

#### Arguments

* `pace` - The limits in force ([`AcquirePace::with_override`](#acquirepacewith_override)).
* `window` - The account's downloads in the last [`BUDGET_WINDOW`](#budget_window).
* `cooldown_until` - The end of the store's cool-down, if one is set.
* `now` - The current time.

### cooldown_end

```rust
pub fn cooldown_end(
    pace: &AcquirePace,
    retry_after: Option<Duration>,
    now: DateTime<Utc>,
) -> DateTime<Utc>
```

The cool-down end to record after the store answered `rateLimited`.

The store's `retry_after` wins when it gave one; otherwise the
storefront's declared cool-down. Either is capped at [`MAX_COOLDOWN`](#max_cooldown).

#### Arguments

* `pace` - The account's declared pace (for its cool-down).
* `retry_after` - How long the store asked to be left alone, if it said.
* `now` - The current time.

## Constants

### BUDGET_WINDOW

```rust
pub const BUDGET_WINDOW: Duration = _;
```

The window the daily budget counts downloads over.

### LEDGER_RETENTION

```rust
pub const LEDGER_RETENTION: Duration = _;
```

How long a download stays in the ledger; older rows are pruned. Longer
than [`BUDGET_WINDOW`](#budget_window) so a clock step never empties the window early.

### DEFAULT_COOLDOWN

```rust
pub const DEFAULT_COOLDOWN: Duration = _;
```

Cool-down when the store says to slow down without a retry time and the
storefront declares no `cooldown_seconds`.

### MAX_COOLDOWN

```rust
pub const MAX_COOLDOWN: Duration = _;
```

Longest cool-down the host records, whatever the store asked for: a
store answer that names a later time is capped here, so a bad
`retryAfterMs` cannot stop an account for good.

### MAX_DOWNLOADS_PER_DAY

```rust
pub const MAX_DOWNLOADS_PER_DAY: u32 = 10_000;
```

Most downloads a day an override may set (the manifest's cap too).

### MAX_MIN_GAP_SECONDS

```rust
pub const MAX_MIN_GAP_SECONDS: u32 = 86_400;
```

Longest gap an override may set, in seconds (one day).
