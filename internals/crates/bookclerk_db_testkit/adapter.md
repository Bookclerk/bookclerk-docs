# bookclerk_db_testkit::adapter

Conformance suite for database adapter plugins.

The rest of this kit serves code that *uses* a database. This module
checks the adapter itself: it drives one empty logical unit through the
public [`Database`](../bookclerk_plugin_abi/README.md#database) / [`AdapterDatabaseSession`](../bookclerk_plugin_abi/README.md#adapterdatabasesession) surface every adapter
exports, the way the host drives a plugin's `[[databases]]` binding, and
reports which parts of the SQL-v1 contract the adapter keeps.

The same suite runs against an adapter value in process and against the
RPC client of a spawned guest ([`bookclerk_plugin_abi::DatabaseClient`](../bookclerk_plugin_abi/README.md#databaseclient)
implements [`Database`](../bookclerk_plugin_abi/README.md#database) too), so a first-party adapter and an outside one
meet one bar. It checks:

- **capabilities**: the advertisement meets the host minimums, declares
  schema migrations, and is the same on every call and every session.
- **bootstrap**: the host's binding bootstrap and a plugin migration apply
  as typed atomic batches.
- **values**: every [`DbValue`](../bookclerk_plugin_abi/db_value.md#dbvalue) variant round-trips through declared
  columns as the same variant (`Int64` extremes, `Float64`, multibyte and
  `b64:`-prefixed text, bytes, `Boolean`, typed NULL).
- **atomic_batch**: a batch whose last statement fails leaves nothing of
  the earlier statements behind.
- **max_statements** / **max_binds**: a batch of exactly `maxStatements`
  statements and a statement with exactly `maxBinds` binds are accepted,
  so the advertised numbers are real. Where `maxBinds` binds cannot fit
  `maxPayloadBytes` / `maxRequestBytes` as host admission measures them,
  `max_binds` sends the most binds that do: the largest statement a
  guest can send.
- **result_rows**: `maxResultRows` rows read inline, one more is refused
  by the adapter with a typed `limit_exceeded` error instead of being
  truncated or returned.
- **cell_bytes** / **result_bytes**: a cell as wide as one bind can
  carry under the byte caps (at most 32 KiB) round-trips, a cell built
  past `maxCellBytes` in SQL and a result past the inline byte caps are
  refused with a typed `limit_exceeded` error, and the refusal comes
  from the adapter, not only from the host's own reply check.
- **streamed**: an adapter that advertises a streamed budget answers
  `executeStreamed` with results past the inline caps; one that does not
  answers `unsupported`.
- **receipt_replay** / **receipt_conflict**: a retried batch with the
  same operation id replays its first reply without applying again (the
  retry after a lost reply), and the same id with a different request is
  refused with `conflict`.
- **close**: the session closes cleanly.

Open the adapter on an empty unit and call [`assert_adapter_conforms`](#assert_adapter_conforms)
from a test, or [`run_adapter_suite`](#run_adapter_suite) to inspect the [`AdapterReport`](#adapterreport).

| Item | Kind | Summary |
| --- | --- | --- |
| [`AdapterReport`](#adapterreport) | struct | The result of one suite run. |
| [`NativeAdapter`](#nativeadapter) | struct | A native adapter guest binary spawned directly and reached over the Cap'n Proto boundary the host speaks, for running the suite against the shipped artifact. |
| [`Case`](#case) | enum | One check of the suite, in the order the suite runs them. |
| [`Outcome`](#outcome) | enum | How one case ended. |
| [`assert_adapter_conforms`](#assert_adapter_conforms) | function | Runs the suite and panics with the report when a case fails. |
| [`run_adapter_suite`](#run_adapter_suite) | function | Runs every `Case` against `database` and reports each outcome. |

## Structs

### AdapterReport

```rust
pub struct AdapterReport {
    pub capabilities: Option<DbCapabilities>,
    pub outcomes: Vec<(Case, Outcome)>,
}
```

The result of one suite run.

**Fields**

- `capabilities`: The advertisement the first session answered, when it answered one.
- `outcomes`: One entry per [`Case`](#case), in run order.

#### AdapterReport::passed

```rust
pub fn passed(&self) -> bool
```

True when no case failed. Skipped cases do not fail the report.

#### AdapterReport::failures

```rust
pub fn failures(&self) -> impl Iterator<Item = (Case, &str)>
```

The failed cases and their reasons.

#### AdapterReport::outcome

```rust
pub fn outcome(&self, case: Case) -> Option<&Outcome>
```

The outcome of `case`.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

### NativeAdapter

```rust
pub struct NativeAdapter {
    /* private fields */
}
```

A native adapter guest binary spawned directly and reached over the
Cap'n Proto boundary the host speaks, for running the suite against the
shipped artifact.

The guest runs unconfined and without the `bookclerk-workerd` front door
(the direct diagnostic transport), so this is for tests only. Create and
use it inside a [`tokio::task::LocalSet`]: the RPC system runs as a local
task.

#### NativeAdapter::spawn

```rust
pub async fn spawn(
    binary: &Path,
    envs: &[(&str, &OsStr)],
    values: BindingValues,
) -> Result<Self, String>
```

Spawns `binary` with `envs`, then `describe`s it and `open`s its
`databaseAdapter` entrypoint with `values` (the host-private connect
params, for example from
[`bookclerk_plugin_abi::db::binding_values_from_params`](../bookclerk_plugin_abi/db.md#binding_values_from_params)).

##### Errors

Returns the spawn, describe or open failure, or a message when the
guest exports no `databaseAdapter`.

#### NativeAdapter::database

```rust
pub fn database(&self) -> &dyn Database
```

The guest's adapter, as the host holds it.

#### NativeAdapter::shutdown

```rust
pub async fn shutdown(self)
```

Kills the guest.

## Enums

### Case

```rust
pub enum Case {
    Capabilities,
    Bootstrap,
    Values,
    AtomicBatch,
    MaxStatements,
    MaxBinds,
    ResultRows,
    CellBytes,
    ResultBytes,
    Streamed,
    ReceiptReplay,
    ReceiptConflict,
    Close,
}
```

One check of the suite, in the order the suite runs them.

**Variants**

- `Capabilities`: The advertisement meets the host minimums and is stable.
- `Bootstrap`: The binding bootstrap and a plugin migration apply.
- `Values`: Every value variant round-trips as the same variant.
- `AtomicBatch`: A failed batch rolls back the statements before the failure.
- `MaxStatements`: A batch of exactly `maxStatements` statements is accepted.
- `MaxBinds`: The largest statement the bind and byte caps admit is accepted.
- `ResultRows`: `maxResultRows` rows read; one more is a typed refusal.
- `CellBytes`: A wide written cell round-trips; one past `maxCellBytes` is refused.
- `ResultBytes`: A result past the inline byte caps is a typed refusal.
- `Streamed`: `executeStreamed` matches the advertised streamed budget.
- `ReceiptReplay`: A retry with the same operation id replays without reapplying.
- `ReceiptConflict`: The same operation id with another request is a conflict.
- `Close`: The session closes cleanly.

#### Case::ALL

```rust
pub const ALL: [Self; 13] = _
```

Every case, in run order.

#### Case::name

```rust
pub fn name(self) -> &'static str
```

The case's stable snake_case name, as reports print it.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `PartialEq`

### Outcome

```rust
pub enum Outcome {
    Passed,
    Failed(String),
    Skipped(String),
}
```

How one case ended.

**Variants**

- `Passed`: The adapter kept this part of the contract.
- `Failed`: The adapter broke it; the text says how.
- `Skipped`: The case could not run: an earlier case it depends on failed, or the
  advertisement makes it unreachable (for example a cell cap so small
  that no row-capped result can pass the byte cap). The text says why.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### assert_adapter_conforms

```rust
pub async fn assert_adapter_conforms(database: &dyn Database) -> AdapterReport
```

Runs the suite and panics with the report when a case fails.

`database` must be open on an empty unit: the suite creates its own
tables there.

#### Panics

Panics when [`AdapterReport::passed`](#adapterreportpassed) is false, printing every case.

### run_adapter_suite

```rust
pub async fn run_adapter_suite(database: &dyn Database) -> AdapterReport
```

Runs every [`Case`](#case) against `database` and reports each outcome.

`database` must be open on an empty unit: the suite applies the host's
binding bootstrap and its own plugin migration there, then writes and
reads only its own `conformance_*` tables. Cases never panic; a broken
adapter shows up as [`Outcome::Failed`](#outcome).
