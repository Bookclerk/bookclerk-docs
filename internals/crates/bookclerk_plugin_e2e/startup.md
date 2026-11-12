# bookclerk_plugin_e2e::startup

Summaries for the plugin cold-start measurement (`tests/cold_start.rs`).

The measurement spawns each staged guest several times through the
production path and records how long each phase took. This module turns
those samples into per-plugin and per-group figures and renders them,
sans-IO, so the unit tests drive the same code the measurement does.

The figures feed the on-demand activation decision in
`docs/adr/event-delivery-model.md`: how long a family takes from "no
process" to "answered its first call" decides whether an idle instance
of that family may sleep.

| Item | Kind | Summary |
| --- | --- | --- |
| [`StartupSample`](#startupsample) | struct | One spawn of one guest, measured phase by phase. |
| [`StartupStats`](#startupstats) | struct | Figures for a set of samples, in milliseconds. |
| [`PluginStartup`](#pluginstartup) | struct | One measured guest and how it is grouped in the report. |
| [`GroupStats`](#groupstats) | struct | Figures for one group of guests (a family or a runtime). |
| [`median`](#median) | function | Median of `values` (the mean of the middle two for an even count). |
| [`summarize`](#summarize) | function | Summarizes `samples`, the first being the coldest. |
| [`group_by`](#group_by) | function | Groups `plugins` by `key` and takes the median of each member's median `ready` (and first call). |
| [`markdown_report`](#markdown_report) | function | Renders the report as Markdown: per plugin, per family, per runtime. |
| [`json_report`](#json_report) | function | Renders the report as JSON for later comparison. |

## Structs

### StartupSample

```rust
pub struct StartupSample {
    pub spawn: Duration,
    pub warm_describe: Duration,
    pub first_call: Option<Duration>,
}
```

One spawn of one guest, measured phase by phase.

**Fields**

- `spawn`: From the spawn call to a ready session: process start, jail, runtime
  boot, the Cap'n Proto handshake, and the guest's first `describe`
  with its negotiation, which the spawn waits for.
- `warm_describe`: One more `describe` on the started session: a warm round trip, for
  comparing against the cold one inside `spawn`.
- `first_call`: The first entrypoint call (`health`), when the guest has one the
  measurement can make without credentials; `None` otherwise.

#### StartupSample::ready

```rust
pub fn ready(&self) -> Duration
```

Time until the guest could serve a call.

##### Returns

`spawn`, which already includes the guest's first `describe`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### StartupStats

```rust
pub struct StartupStats {
    pub runs: usize,
    pub first_ready_ms: f64,
    pub median_ready_ms: f64,
    pub max_ready_ms: f64,
    pub median_warm_rpc_ms: f64,
    pub median_first_call_ms: Option<f64>,
}
```

Figures for a set of samples, in milliseconds.

**Fields**

- `runs`: Samples summarized.
- `first_ready_ms`: `ready` of the first sample: the coldest start (file cache and
  runtime caches not yet warm).
- `median_ready_ms`: Median `ready` across all samples.
- `max_ready_ms`: Largest `ready` across all samples.
- `median_warm_rpc_ms`: Median warm `describe` round trip across all samples.
- `median_first_call_ms`: Median first entrypoint call, over the samples that made one.

Implements: `Clone`, `Debug`, `PartialEq`

### PluginStartup

```rust
pub struct PluginStartup {
    pub id: String,
    pub runtime: String,
    pub families: Vec<String>,
    pub stats: StartupStats,
}
```

One measured guest and how it is grouped in the report.

**Fields**

- `id`: Plugin manifest id.
- `runtime`: How the guest runs: `workerd/js`, `workerd/python`, or `native` plus
  its launcher (`native/node`, `native/python`, `native/binary`).
- `families`: Plugin families from the manifest (`source`, `output`, `database`,
  `integration`).
- `stats`: The guest's figures.

Implements: `Clone`, `Debug`, `PartialEq`

### GroupStats

```rust
pub struct GroupStats {
    pub members: Vec<String>,
    pub median_ready_ms: f64,
    pub max_ready_ms: f64,
    pub median_first_call_ms: Option<f64>,
}
```

Figures for one group of guests (a family or a runtime).

**Fields**

- `members`: Plugin ids in the group.
- `median_ready_ms`: Median of the members' median `ready`, in milliseconds.
- `max_ready_ms`: Largest single `ready` any member took, in milliseconds.
- `median_first_call_ms`: Median of the members' median first call, when any made one.

Implements: `Clone`, `Debug`, `PartialEq`

## Functions

### median

```rust
pub fn median(values: &[f64]) -> Option<f64>
```

Median of `values` (the mean of the middle two for an even count).

#### Returns

`None` for an empty slice.

### summarize

```rust
pub fn summarize(samples: &[StartupSample]) -> Option<StartupStats>
```

Summarizes `samples`, the first being the coldest.

#### Returns

`None` when `samples` is empty.

### group_by

```rust
pub fn group_by<F>(plugins: &[PluginStartup], keys: F) -> BTreeMap<String, GroupStats>
where
    F: Fn(&PluginStartup) -> Vec<String>,
```

Groups `plugins` by `key` and takes the median of each member's median
`ready` (and first call).

#### Arguments

* `plugins` - Measured guests.
* `keys` - The group keys of one guest (a guest in several families
  counts in each).

#### Returns

Group key → (members, median ready ms, worst max ready ms, median first
call ms).

### markdown_report

```rust
pub fn markdown_report(plugins: &[PluginStartup], context: &str) -> String
```

Renders the report as Markdown: per plugin, per family, per runtime.

#### Arguments

* `plugins` - Measured guests, in report order.
* `context` - One line on where it ran (CPU limit, host), shown first.

#### Returns

The Markdown text.

### json_report

```rust
pub fn json_report(plugins: &[PluginStartup], context: &str) -> Value
```

Renders the report as JSON for later comparison.

#### Returns

A JSON value with one object per plugin.
