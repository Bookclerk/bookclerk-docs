**bookclerk_library > migrations > engine**

# Module: migrations::engine

## Contents

**Enums**

- [`ApplyDirection`](#applydirection) - Whether [`apply_migration_plan`](engine.md#bookclerk_librarymigrationsengineapply_migration_plan) / [`downgrade_migration_plan`](engine.md#bookclerk_librarymigrationsenginedowngrade_migration_plan) is walking up or down.

**Functions**

- [`apply_migration_plan`](#apply_migration_plan) - Applies remaining frozen steps in `plan` for `plan.namespace`.
- [`downgrade_migration_plan`](#downgrade_migration_plan) - Explicit downgrade of `plan.namespace` toward `to_version` (0 = fully reverse).
- [`remaining_upgrade_batches`](#remaining_upgrade_batches) - Remaining forward apply batches for `plan` given durable `state`.
- [`schema_session_matches`](#schema_session_matches) - True when a binding session's expected host schema still matches durable state.

---

## bookclerk_library::migrations::engine::ApplyDirection

*Enum*

Whether [`apply_migration_plan`](engine.md#bookclerk_librarymigrationsengineapply_migration_plan) / [`downgrade_migration_plan`](engine.md#bookclerk_librarymigrationsenginedowngrade_migration_plan) is walking up or down.

**Variants:**
- `Upgrade` - Forward upgrade (installed plan newer than stored state).
- `Downgrade` - Explicit downgrade (every traversed step must have `down`).

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ApplyDirection`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ApplyDirection) -> bool`



## bookclerk_library::migrations::engine::apply_migration_plan

*Function*

Applies remaining frozen steps in `plan` for `plan.namespace`.

Takes the portable `schema:{namespace}` serialization slot, re-reads
durable state, and applies each missing step plus its marker as one
atomic [`ExecuteRequest`](../../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest). Matching version+checksum after an ambiguous
completion is success. Absent marker retries the same step.
Contradictory checksum/version fails closed. Constraint/type failures
are not retries.

Stored frozen state newer than this plan, or a checksum that is not in
the installed plan, fails closed. Empty plan: uninitialized is a no-op;
any stored frozen/unreleased rows fail closed.

Does **not** run as part of destructive restore. Restore writes captured
ledger rows; a later ordinary open walks forward.

# Errors

Returns `LibraryError::Schema` when the ledger is contradictory, the
installed plan cannot understand stored state, or apply exhausts retries.

```rust
fn apply_migration_plan(db: &sea_orm::DatabaseConnection, plan: &super::plan::MigrationPlan) -> crate::error::Result<crate::schema_state::SchemaState>
```



## bookclerk_library::migrations::engine::downgrade_migration_plan

*Function*

Explicit downgrade of `plan.namespace` toward `to_version` (0 = fully reverse).

Never runs on ordinary open. Every traversed step must have a proven `down`;
otherwise this fails closed and the operator must restore a recovery point.
An older plugin binary must not call this implicitly.

# Errors

Returns when stored state is not frozen, a step is irreversible, or apply fails.

```rust
fn downgrade_migration_plan(db: &sea_orm::DatabaseConnection, plan: &super::plan::MigrationPlan, to_version: i64) -> crate::error::Result<crate::schema_state::SchemaState>
```



## bookclerk_library::migrations::engine::remaining_upgrade_batches

*Function*

Remaining forward apply batches for `plan` given durable `state`.

Each inner `Vec` is one atomic unit (slot lock, ops, marker). Used by the
RPC binding path that cannot run SeaORM [`apply_migration_plan`](engine.md#bookclerk_librarymigrationsengineapply_migration_plan).

# Errors

Returns when stored state cannot be understood by `plan`.

```rust
fn remaining_upgrade_batches(plan: &super::plan::MigrationPlan, state: &crate::schema_state::SchemaState) -> crate::error::Result<Vec<Vec<String>>>
```



## bookclerk_library::migrations::engine::schema_session_matches

*Function*

True when a binding session's expected host schema still matches durable state.

# Errors

Returns when `observed` is a newer frozen revision, a different checksum,
or a different discriminant than `expected`.

```rust
fn schema_session_matches(expected: &crate::schema_state::SchemaState, observed: &crate::schema_state::SchemaState) -> crate::error::Result<()>
```



