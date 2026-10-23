# bookclerk_library::migrations

Host library schema: frozen plan steps plus the unreleased development pack.

Bookclerk has **not** frozen a production v1 schema. [`host_migration_plan`](#host_migration_plan)
is empty until a release cut. Live schema lives in [`unreleased_ops`](#unreleased_ops) (already
separated [`MigrationOp`](#migrationop)s). Fresh databases apply those ops (frozen ups +
unreleased) and persist [`crate::SchemaState::Unreleased`](index.md#schemastate) with `base_version`
equal to [`SCHEMA_VERSION`](#schema_version) (`0` today: no frozen revisions). Fresh init
records each frozen plan step's checksum before the unreleased marker; when
the unreleased bucket is empty the database ends
[`crate::SchemaState::Frozen`](index.md#schemastate). Joined SQL ([`unreleased_sql`](#unreleased_sql),
[`current_canonical_schema`](#current_canonical_schema)) is derived for diagnostics/export/tests.
Adapters lower canonical DDL at the execution edge
([`bookclerk_db_exec::expand_host_schema_batch`](../bookclerk_db_exec/index.md#expand_host_schema_batch)).

| Item | Kind | Summary |
| --- | --- | --- |
| [`MigrationPlan`](#migrationplan) | struct | Ordered frozen plan for one ledger namespace. |
| [`MigrationStep`](#migrationstep) | struct | One ordered schema revision: checksumed `up` ops and optional `down` ops. |
| [`PluginJournalEntry`](#pluginjournalentry) | struct | One durable host-private journal row. |
| [`HostMigrationStep`](#hostmigrationstep) | struct | One host-owned schema version in the canonical Bookclerk migration plan. |
| [`MigrationOp`](#migrationop) | enum | One admitted BookclerkSQL statement in a host migration apply unit. |
| [`ApplyDirection`](#applydirection) | enum | Whether `apply_migration_plan` / `downgrade_migration_plan` is walking up or down. |
| [`PlanOp`](#planop) | enum | One admitted BookclerkSQL statement owned by a `MigrationStep`. |
| [`latest_schema_sqlite`](#latest_schema_sqlite) | function | Final SQLite DDL for a fresh Bookclerk library database. |
| [`unreleased_sql`](#unreleased_sql) | function | Live development DDL derived from `unreleased_ops` (joined with `;\n`). |
| [`apply_migration_plan`](#apply_migration_plan) | function | Applies remaining frozen steps in `plan` for `plan.namespace`. |
| [`downgrade_migration_plan`](#downgrade_migration_plan) | function | Explicit downgrade of `plan.namespace` toward `to_version` (0 = fully reverse). |
| [`remaining_upgrade_batches`](#remaining_upgrade_batches) | function | Remaining forward apply batches for `plan` given durable `state`. |
| [`schema_session_matches`](#schema_session_matches) | function | True when a binding session's expected host schema still matches durable state. |
| [`frozen_marker_delete_sql`](#frozen_marker_delete_sql) | function | `DELETE` of one frozen step's namespaced marker. |
| [`frozen_marker_sql`](#frozen_marker_sql) | function | `INSERT` for a frozen namespaced `bookclerk_schema_migrations` row. |
| [`schema_slot_key`](#schema_slot_key) | function | Slot key that serializes schema apply for one namespace. |
| [`unreleased_marker_sql_in`](#unreleased_marker_sql_in) | function | `INSERT` for an unreleased namespaced `bookclerk_schema_migrations` row. |
| [`validate_schema_namespace`](#validate_schema_namespace) | function | True when `namespace` is `bookclerk` or a plugin-id grammar string. |
| [`load_plugin_migration_history_on`](#load_plugin_migration_history_on) | function | Reads the journal on an already-open connection (including a backup capture transaction). |
| [`pending_plugin_suffix`](#pending_plugin_suffix) | function | Pending registered migrations after the durable prefix. |
| [`plugin_history_digest`](#plugin_history_digest) | function | SHA-256 of ordered `(migration_id, checksum)` pairs. |
| [`plugin_migration_checksum`](#plugin_migration_checksum) | function | Checksum of one registered migration (id + classified SQL). |
| [`remaining_plugin_suffix_batches`](#remaining_plugin_suffix_batches) | function | Remaining atomic apply batches for RPC hosts that cannot run SeaORM apply. |
| [`min_supported_schema_version`](#min_supported_schema_version) | function | Oldest frozen schema version this binary can run. |
| [`min_supported_schema_version_in`](#min_supported_schema_version_in) | function | Oldest frozen version in `plan` (`None` when `plan` is empty). |
| [`migration_ops_checksum`](#migration_ops_checksum) | function | SHA-256 of length-prefixed migration ops (not a joined script). |
| [`migration_sql_checksum`](#migration_sql_checksum) | function | SHA-256 of canonical up statements, plus down statements when reversible. |
| [`pack_migration_sql`](#pack_migration_sql) | function | Packs `sql` with the SQL-v1 lexer. Empty input yields no statements. |
| [`join_op_sql`](#join_op_sql) | function | Joins already-separated ops with `;\n` for diagnostics/export. |
| [`binding_bootstrap_ops`](#binding_bootstrap_ops) | function | Host-owned bootstrap ops applied inside every isolated plugin binding. |
| [`binding_bootstrap_sql`](#binding_bootstrap_sql) | function | Derived diagnostic SQL for `binding_bootstrap_ops`. |
| [`binding_bootstrap_statements`](#binding_bootstrap_statements) | function | Ordered canonical statements for `binding_bootstrap_ops`. |
| [`binding_bootstrap_type_env`](#binding_bootstrap_type_env) | function | Type environment implied by `binding_bootstrap_ops` (journal, slots, catalog). |
| [`binding_unreleased_checksum`](#binding_unreleased_checksum) | function | SHA-256 of `binding_bootstrap_ops` (length-prefixed statement list). |
| [`current_canonical_schema`](#current_canonical_schema) | function | Frozen ups concatenated with `unreleased_ops` (derived diagnostic SQL). |
| [`current_canonical_statements`](#current_canonical_statements) | function | Ordered canonical statements for `current_canonical_schema`. |
| [`unreleased_ops`](#unreleased_ops) | function | Live unreleased host schema ops (source of truth). |
| [`unreleased_statements`](#unreleased_statements) | function | Ordered statements in `unreleased_ops`. |
| [`unreleased_checksum`](#unreleased_checksum) | function | SHA-256 of `unreleased_ops` (empty string when empty). |
| [`unreleased_state_marker_sql`](#unreleased_state_marker_sql) | function | `INSERT` for an unreleased `bookclerk_schema_migrations` row in the Bookclerk namespace. |
| [`prove_migration_ops`](#prove_migration_ops) | function | Proves each op is exactly one BookclerkSQL statement of the declared kind. |
| [`require_single_migration_statement`](#require_single_migration_statement) | function | Packs `sql` and requires exactly one statement. |
| [`current_canonical_table_names`](#current_canonical_table_names) | function | Host table names declared by `current_canonical_schema`, plus `bookclerk_schema_migrations`. |
| [`host_sql_type_env`](#host_sql_type_env) | function | Column types implied by `current_canonical_schema`, plus the `bookclerk_schema_migrations` ledger created before any plan step. |
| [`host_migration_plan`](#host_migration_plan) | function | Frozen host migration steps. Empty until a release cut copies `unreleased_ops` into version 1. |
| [`SCHEMA_VERSION`](#schema_version) | constant | Highest **frozen** schema version this binary knows (`0` while the plan is empty). |
| [`SCHEMA_MIGRATIONS_DDL`](#schema_migrations_ddl) | constant | Host bookkeeping table created before applying plan versions. |
| [`CLUSTER_LEASES_DDL`](#cluster_leases_ddl) | constant | `cluster_leases`: named singleton leases (scheduler, maintenance duties, schema migration). |
| [`PLUGIN_MIGRATION_SLOT_KEY`](#plugin_migration_slot_key) | constant | Per-binding serialization slot for plugin migration apply. |
| [`BINDING_SCHEMA_VERSION`](#binding_schema_version) | constant | Highest frozen binding-bootstrap version (`0` until a binding freeze). |

## Structs

### MigrationPlan

```rust
pub struct MigrationPlan {
    pub namespace: String,
    pub steps: Vec<MigrationStep>,
}
```

Ordered frozen plan for one ledger namespace.

**Fields**

- `namespace`: `bookclerk_schema_migrations.namespace` (`bookclerk` for host-owned plans).
- `steps`: Frozen steps in increasing version order.

#### MigrationPlan::try_new

```rust
pub fn try_new(namespace: impl Into<String>, steps: Vec<MigrationStep>) -> Result<Self>
```

Builds a plan after proving every op and checking version order.

##### Errors

Returns when `namespace` is not a ledger namespace, versions are not
strictly increasing positive integers, or an op fails BookclerkSQL proof.

#### MigrationPlan::min_version

```rust
pub fn min_version(&self) -> Option<i64>
```

Oldest frozen version (`None` when this plan has no frozen steps).

#### MigrationPlan::max_version

```rust
pub fn max_version(&self) -> i64
```

Highest frozen version in this plan (`0` when empty).

#### MigrationPlan::step

```rust
pub fn step(&self, version: i64) -> Option<&MigrationStep>
```

Step with `version`, if present.

#### MigrationPlan::checksums_through

```rust
pub fn checksums_through(&self, through: i64) -> HashMap<i64, String>
```

Expected checksum map for frozen rows through `through`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### MigrationStep

```rust
pub struct MigrationStep {
    pub version: i64,
    pub up: Vec<PlanOp>,
    pub down: Option<Vec<PlanOp>>,
    pub introduced_in: String,
}
```

One ordered schema revision: checksumed `up` ops and optional `down` ops.

**Fields**

- `version`: Monotonic schema revision recorded on the namespaced ledger.
- `up`: Ordered schema and data ops for this version.
- `down`: Reverse ops when this step is reversible; `None` means restore a backup.
- `introduced_in`: First package/semver that shipped this step.

#### MigrationStep::checksum

```rust
pub fn checksum(&self) -> String
```

SHA-256 hex digest of length-prefixed `up` (and `down` when present).

#### MigrationStep::reversible

```rust
pub fn reversible(&self) -> bool
```

True when [`Self::down`](#migrationstep) is present so an explicit downgrade can apply it.

#### MigrationStep::from_host

```rust
pub fn from_host(step: &HostMigrationStep) -> Self
```

Converts a static host step into an owned engine step.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PluginJournalEntry

```rust
pub struct PluginJournalEntry {
    pub ordinal: i64,
    pub migration_id: String,
    pub checksum: String,
}
```

One durable host-private journal row.

**Fields**

- `ordinal`: Host-private storage position (`0..n-1`). Not a plugin version.
- `migration_id`: Opaque plugin-chosen identity.
- `checksum`: Checksum recorded when this row was applied.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### HostMigrationStep

```rust
pub struct HostMigrationStep {
    pub version: i64,
    pub steps: &'static [MigrationOp],
    pub down: Option<&'static [MigrationOp]>,
    pub introduced_in: &'static str,
}
```

One host-owned schema version in the canonical Bookclerk migration plan.

Each version is recorded as a `bookclerk_schema_migrations` row (adapters
must advertise `schemaMigrations`). The live connection backend lowers
each [`MigrationOp`](#migrationop) at the adapter boundary (Postgres) or applies it
verbatim (SQLite / D1).

**Fields**

- `version`: `bookclerk_schema_migrations.version` for this step.
- `steps`: Ordered schema and data ops for this version (the `up`).
- `down`: Reverse ops when this step is reversible; `None` means restore a backup.
- `introduced_in`: First Bookclerk semver that shipped this step.

#### HostMigrationStep::checksum

```rust
pub fn checksum(&self) -> String
```

SHA-256 hex digest of length-prefixed `up` (and `down` when present).

#### HostMigrationStep::reversible

```rust
pub fn reversible(&self) -> bool
```

True when [`Self::down`](#hostmigrationstep) is present so CLI rollback can apply this step.

#### HostMigrationStep::up_sql

```rust
pub fn up_sql(&self) -> String
```

Concatenated up SQL (tests / derived views). Boundaries are the op list.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Enums

### MigrationOp

```rust
pub enum MigrationOp {
    Schema(&'static str),
    Data(&'static str),
}
```

One admitted BookclerkSQL statement in a host migration apply unit.

**Variants**

- `Schema`: Schema mutation (`CREATE` / `DROP` / `ALTER` / index).
- `Data`: Data backfill (`INSERT` / `UPDATE` / `DELETE`).

#### MigrationOp::sql

```rust
pub const fn sql(self) -> &'static str
```

Canonical BookclerkSQL for this op.

#### MigrationOp::is_schema

```rust
pub const fn is_schema(self) -> bool
```

True when this op is admitted schema DDL.

#### MigrationOp::is_data

```rust
pub const fn is_data(self) -> bool
```

True when this op is admitted DML / backfill.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ApplyDirection

```rust
pub enum ApplyDirection {
    Upgrade,
    Downgrade,
}
```

Whether [`apply_migration_plan`](#apply_migration_plan) / [`downgrade_migration_plan`](#downgrade_migration_plan) is walking up or down.

**Variants**

- `Upgrade`: Forward upgrade (installed plan newer than stored state).
- `Downgrade`: Explicit downgrade (every traversed step must have `down`).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### PlanOp

```rust
pub enum PlanOp {
    Schema(String),
    Data(String),
}
```

One admitted BookclerkSQL statement owned by a [`MigrationStep`](#migrationstep).

**Variants**

- `Schema`: Schema mutation (`CREATE` / `DROP` / `ALTER` / index).
- `Data`: Data backfill (`INSERT` / `UPDATE` / `DELETE`).

#### PlanOp::schema_static

```rust
pub fn schema_static(sql: &'static str) -> Self
```

Schema op from static host SQL.

#### PlanOp::sql

```rust
pub fn sql(&self) -> &str
```

Canonical BookclerkSQL for this op.

#### PlanOp::is_schema

```rust
pub fn is_schema(&self) -> bool
```

True when this op is admitted schema DDL.

#### PlanOp::from_static

```rust
pub fn from_static(op: MigrationOp) -> Self
```

Converts a host static op into an owned plan op.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### latest_schema_sqlite

```rust
pub fn latest_schema_sqlite() -> &'static str
```

Final SQLite DDL for a fresh Bookclerk library database.

SeaORM entities in [`crate::entities`](entities.md) mirror these columns exactly. All
integer columns map to `i64`, reals to `f64`, blobs to `Vec<u8>`, and text
(including RFC 3339 timestamps) to `String`.

### unreleased_sql

```rust
pub fn unreleased_sql() -> &'static str
```

Live development DDL derived from [`unreleased_ops`](#unreleased_ops) (joined with `;\n`).

Apply, checksum, and backup use the op list. This script is diagnostics,
export, and `execute_batch` tests only.

### apply_migration_plan

```rust
pub async fn apply_migration_plan(
    db: &DatabaseConnection,
    plan: &MigrationPlan,
) -> Result<SchemaState>
```

Applies remaining frozen steps in `plan` for `plan.namespace`.

Takes the portable `schema:{namespace}` serialization slot, re-reads
durable state, and applies each missing step plus its marker as one
atomic [`ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest). Matching version+checksum after an ambiguous
completion is success. Absent marker retries the same step.
Contradictory checksum/version fails closed. Constraint/type failures
are not retries.

Stored frozen state newer than this plan, or a checksum that is not in
the installed plan, fails closed. Empty plan: uninitialized is a no-op;
any stored frozen/unreleased rows fail closed.

Does **not** run as part of destructive restore. Restore writes captured
ledger rows; a later ordinary open walks forward.

#### Errors

Returns [`LibraryError::Schema`](index.md#libraryerror) when the ledger is contradictory, the
installed plan cannot understand stored state, or apply exhausts retries.

### downgrade_migration_plan

```rust
pub async fn downgrade_migration_plan(
    db: &DatabaseConnection,
    plan: &MigrationPlan,
    to_version: i64,
) -> Result<SchemaState>
```

Explicit downgrade of `plan.namespace` toward `to_version` (0 = fully reverse).

Never runs on ordinary open. Every traversed step must have a proven `down`;
otherwise this fails closed and the operator must restore a recovery point.
An older plugin binary must not call this implicitly.

#### Errors

Returns when stored state is not frozen, a step is irreversible, or apply fails.

### remaining_upgrade_batches

```rust
pub fn remaining_upgrade_batches(
    plan: &MigrationPlan,
    state: &SchemaState,
) -> Result<Vec<Vec<String>>>
```

Remaining forward apply batches for `plan` given durable `state`.

Each inner `Vec` is one atomic unit (slot lock, ops, marker). Used by the
RPC binding path that cannot run SeaORM [`apply_migration_plan`](#apply_migration_plan).

#### Errors

Returns when stored state cannot be understood by `plan`.

### schema_session_matches

```rust
pub fn schema_session_matches(
    expected: &SchemaState,
    observed: &SchemaState,
) -> Result<()>
```

True when a binding session's expected host schema still matches durable state.

#### Errors

Returns when `observed` is a newer frozen revision, a different checksum,
or a different discriminant than `expected`.

### frozen_marker_delete_sql

```rust
pub fn frozen_marker_delete_sql(namespace: &str, version: i64) -> String
```

`DELETE` of one frozen step's namespaced marker.

### frozen_marker_sql

```rust
pub fn frozen_marker_sql(namespace: &str, version: i64, checksum: &str) -> String
```

`INSERT` for a frozen namespaced `bookclerk_schema_migrations` row.

### schema_slot_key

```rust
pub fn schema_slot_key(namespace: &str) -> String
```

Slot key that serializes schema apply for one namespace.

### unreleased_marker_sql_in

```rust
pub fn unreleased_marker_sql_in(
    namespace: &str,
    checksum: &str,
    base_version: i64,
) -> String
```

`INSERT` for an unreleased namespaced `bookclerk_schema_migrations` row.

### validate_schema_namespace

```rust
pub fn validate_schema_namespace(namespace: &str) -> Result<()>
```

True when `namespace` is `bookclerk` or a plugin-id grammar string.

#### Errors

Returns when the namespace is empty, too long, or contains characters other
than lowercase ascii, digits, and single `_`.

### load_plugin_migration_history_on

```rust
pub async fn load_plugin_migration_history_on<C>(
    conn: &C,
) -> Result<PluginMigrationHistory>
where
    C: ConnectionTrait + StreamTrait,
```

Reads the journal on an already-open connection (including a backup capture
transaction).

#### Errors

Returns when the journal cannot be read or ordinals are not contiguous.

### pending_plugin_suffix

```rust
pub fn pending_plugin_suffix<'a>(
    history: &PluginMigrationHistory,
    registered: &'a PluginMigrationSequence,
) -> Result<&'a [ProvenPluginMigration]>
```

Pending registered migrations after the durable prefix.

#### Errors

Returns [`require_history_prefix`](index.md#require_history_prefix) errors.

### plugin_history_digest

```rust
pub fn plugin_history_digest(entries: &[PluginJournalEntry]) -> String
```

SHA-256 of ordered `(migration_id, checksum)` pairs.

### plugin_migration_checksum

```rust
pub fn plugin_migration_checksum(id: &str, operations: &[PluginMigrationOp]) -> String
```

Checksum of one registered migration (id + classified SQL).

### remaining_plugin_suffix_batches

```rust
pub fn remaining_plugin_suffix_batches(
    history: &PluginMigrationHistory,
    registered: &PluginMigrationSequence,
) -> Result<Vec<Vec<String>>>
```

Remaining atomic apply batches for RPC hosts that cannot run SeaORM apply.

#### Errors

Returns [`require_history_prefix`](index.md#require_history_prefix) errors.

### min_supported_schema_version

```rust
pub fn min_supported_schema_version() -> Option<i64>
```

Oldest frozen schema version this binary can run.

Derived from [`host_migration_plan`]: `None` while the plan is empty
(no frozen schema versions exist). Once frozen steps exist this is the
first retained step, not a separately synchronized constant.

### min_supported_schema_version_in

```rust
pub fn min_supported_schema_version_in(plan: &[HostMigrationStep]) -> Option<i64>
```

Oldest frozen version in `plan` (`None` when `plan` is empty).

### migration_ops_checksum

```rust
pub fn migration_ops_checksum(ups: &[MigrationOp], down: Option<&[MigrationOp]>) -> String
```

SHA-256 of length-prefixed migration ops (not a joined script).

### migration_sql_checksum

```rust
pub fn migration_sql_checksum(canonical: &str, down: Option<&str>) -> Result<String>
```

SHA-256 of canonical up statements, plus down statements when reversible.

#### Errors

Returns when `canonical` or `down` is not a BookclerkSQL statement list.
Parser failure is never treated as a single opaque statement.

### pack_migration_sql

```rust
pub fn pack_migration_sql(sql: &str) -> Result<Vec<String>>
```

Packs `sql` with the SQL-v1 lexer. Empty input yields no statements.

#### Errors

Returns when `sql` is not a BookclerkSQL statement list.

### join_op_sql

```rust
pub fn join_op_sql(ops: &[MigrationOp]) -> String
```

Joins already-separated ops with `;\n` for diagnostics/export.

### binding_bootstrap_ops

```rust
pub fn binding_bootstrap_ops() -> &'static [MigrationOp]
```

Host-owned bootstrap ops applied inside every isolated plugin binding.

#### Panics

Panics when a bootstrap op is not one admitted BookclerkSQL statement of
the declared [`MigrationOp`](#migrationop) kind.

### binding_bootstrap_sql

```rust
pub fn binding_bootstrap_sql() -> &'static str
```

Derived diagnostic SQL for [`binding_bootstrap_ops`](#binding_bootstrap_ops).

### binding_bootstrap_statements

```rust
pub fn binding_bootstrap_statements() -> &'static [String]
```

Ordered canonical statements for [`binding_bootstrap_ops`](#binding_bootstrap_ops).

### binding_bootstrap_type_env

```rust
pub fn binding_bootstrap_type_env() -> SqlTypeEnv
```

Type environment implied by [`binding_bootstrap_ops`](#binding_bootstrap_ops) (journal, slots, catalog).

Host-private binding bookkeeping lives here so plugin apply can typecheck
journal DML without using the host library catalog.

### binding_unreleased_checksum

```rust
pub fn binding_unreleased_checksum() -> String
```

SHA-256 of [`binding_bootstrap_ops`](#binding_bootstrap_ops) (length-prefixed statement list).

### current_canonical_schema

```rust
pub fn current_canonical_schema() -> &'static str
```

Frozen ups concatenated with [`unreleased_ops`](#unreleased_ops) (derived diagnostic SQL).

While the production plan is empty this is exactly [`unreleased_sql`](#unreleased_sql). After
a release cut it is frozen step SQL plus whatever is again unreleased — do
not assume it equals [`unreleased_sql`](#unreleased_sql) forever. Test plan overrides must
not feed this `OnceLock` (`production_host_migration_plan` only).

### current_canonical_statements

```rust
pub fn current_canonical_statements() -> &'static [String]
```

Ordered canonical statements for [`current_canonical_schema`](#current_canonical_schema).

### unreleased_ops

```rust
pub fn unreleased_ops() -> &'static [MigrationOp]
```

Live unreleased host schema ops (source of truth).

#### Panics

Panics when an unreleased op is not one admitted BookclerkSQL statement of
the declared [`MigrationOp`](#migrationop) kind.

### unreleased_statements

```rust
pub fn unreleased_statements() -> &'static [String]
```

Ordered statements in [`unreleased_ops`](#unreleased_ops).

### unreleased_checksum

```rust
pub fn unreleased_checksum() -> String
```

SHA-256 of [`unreleased_ops`](#unreleased_ops) (empty string when empty).

### unreleased_state_marker_sql

```rust
pub fn unreleased_state_marker_sql(checksum: &str, base_version: i64) -> String
```

`INSERT` for an unreleased `bookclerk_schema_migrations` row in the Bookclerk namespace.

### prove_migration_ops

```rust
pub fn prove_migration_ops(ops: &[MigrationOp]) -> Result<()>
```

Proves each op is exactly one BookclerkSQL statement of the declared kind.

Schema ops must be admitted DDL (`CREATE` / `ALTER` / `DROP`). Data ops must
be admitted DML. Both go through the SQL-v1 typechecker; Schema ops also
update `env` so later Data/index ops see prior `CREATE TABLE`.

#### Errors

Returns when packing, kind, or typechecking fails.

### require_single_migration_statement

```rust
pub fn require_single_migration_statement(sql: &str) -> Result<String>
```

Packs `sql` and requires exactly one statement.

#### Errors

Returns when packing fails or `sql` is not a single statement.

### current_canonical_table_names

```rust
pub fn current_canonical_table_names() -> Vec<String>
```

Host table names declared by [`current_canonical_schema`](#current_canonical_schema), plus
`bookclerk_schema_migrations`.

### host_sql_type_env

```rust
pub fn host_sql_type_env() -> SqlTypeEnv
```

Column types implied by [`current_canonical_schema`](#current_canonical_schema), plus the
`bookclerk_schema_migrations` ledger created before any plan step.

### host_migration_plan

```rust
pub fn host_migration_plan() -> Vec<HostMigrationStep>
```

Frozen host migration steps. Empty until a release cut copies
[`unreleased_ops`](#unreleased_ops) into version 1.

## Constants

### SCHEMA_VERSION

```rust
pub const SCHEMA_VERSION: i64 = 0;
```

Highest **frozen** schema version this binary knows (`0` while the plan is empty).

This is not a discriminator for uninitialized vs unreleased. Use
[`crate::SchemaState`](index.md#schemastate).

### SCHEMA_MIGRATIONS_DDL

```rust
pub const SCHEMA_MIGRATIONS_DDL: &str = "CREATE TABLE IF NOT EXISTS bookclerk_schema_migrations (
        namespace TEXT NOT NULL,
        version INTEGER NOT NULL,
        state TEXT NOT NULL,
        checksum TEXT NOT NULL,
        app_version TEXT NOT NULL,
        applied_at TEXT NOT NULL,
        PRIMARY KEY (namespace, state, version)
    )";
```

Host bookkeeping table created before applying plan versions.

`namespace` separates Bookclerk-owned ledger rows (`bookclerk`) from any
leftover rows. Plugin-owned history uses the separate `bookclerk_plugin_migrations`
journal, not this table.

### CLUSTER_LEASES_DDL

```rust
pub const CLUSTER_LEASES_DDL: &str = r"CREATE TABLE IF NOT EXISTS cluster_leases (
        name TEXT PRIMARY KEY NOT NULL,
        holder TEXT NOT NULL,
        token INTEGER NOT NULL,
        expires_at_ms INTEGER NOT NULL,
        renewed_at TEXT NOT NULL
    )";
```

`cluster_leases`: named singleton leases (scheduler, maintenance duties,
schema migration).

Part of the unreleased pack, and also created on its own before a
schema migration, because the migration lease must exist before the pack
that defines it is applied. Both paths run the same `IF NOT EXISTS`
statement, so whichever runs second is a no-op.

### PLUGIN_MIGRATION_SLOT_KEY

```rust
pub const PLUGIN_MIGRATION_SLOT_KEY: &str = "bookclerk_plugin_migrations";
```

Per-binding serialization slot for plugin migration apply.

### BINDING_SCHEMA_VERSION

```rust
pub const BINDING_SCHEMA_VERSION: i64 = 0;
```

Highest frozen binding-bootstrap version (`0` until a binding freeze).
