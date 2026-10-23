**bookclerk_library > schema_walk**

# Module: schema_walk

## Contents

**Structs**

- [`SchemaWalk`](#schemawalk) - Result of walking a host migration plan from one version to another.

**Functions**

- [`plan_schema_walk`](#plan_schema_walk) - Plans ups or downs from `from` toward `to` using `plan`.
- [`plan_schema_walk_from_state`](#plan_schema_walk_from_state) - Plans a frozen walk from an explicit [`SchemaState`](schema_state.md#bookclerk_libraryschema_stateschemastate).

---

## bookclerk_library::schema_walk::SchemaWalk

*Struct*

Result of walking a host migration plan from one version to another.

**Fields:**
- `from: i64` - Database version before the walk.
- `requested_to: i64` - Version the caller asked to reach.
- `stopped_at: i64` - Version the walk can actually reach (may be higher than `requested_to`
- `ups: Vec<crate::migrations::HostMigrationStep>` - Remaining `up` steps in order (empty when rolling back).
- `downs: Vec<crate::migrations::HostMigrationStep>` - `down` steps newest-first (empty when upgrading).
- `blocked: bool` - True when a missing `down` stopped a rollback short of `requested_to`.

**Methods:**

- `fn is_noop(self: &Self) -> bool` - True when no DDL remains to apply.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SchemaWalk`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SchemaWalk) -> bool`



## bookclerk_library::schema_walk::plan_schema_walk

*Function*

Plans ups or downs from `from` toward `to` using `plan`.

Downgrades walk newest-first and stop at the last reversible version when
the next step has `down: None`. Versions newer than the compiled plan
cannot be reversed by this binary. Versions older than the first frozen
step in `plan` (except empty `from == 0`) cannot be upgraded by this
binary. An empty plan means no frozen schema versions exist.

# Errors

Returns `LibraryError::Schema` when `to` is below zero, `from` is ahead
of every compiled step (unknown newer schema), or `from` is below the
plan's first retained frozen version.

```rust
fn plan_schema_walk(plan: &[crate::migrations::HostMigrationStep], from: i64, to: i64) -> crate::error::Result<SchemaWalk>
```



## bookclerk_library::schema_walk::plan_schema_walk_from_state

*Function*

Plans a frozen walk from an explicit [`SchemaState`](schema_state.md#bookclerk_libraryschema_stateschemastate).

`SchemaState::Unreleased` is never treated as empty `from == 0`. A future
freeze must not apply version-1 ups on top of an unreleased development
database — recreate (`cargo reset --yes`) unless a later explicit promote
path is added.

# Errors

Returns `LibraryError::Schema` when an unreleased database is asked to
become a frozen revision, or when [`plan_schema_walk`](schema_walk.md#bookclerk_libraryschema_walkplan_schema_walk) fails.

```rust
fn plan_schema_walk_from_state(plan: &[crate::migrations::HostMigrationStep], state: &crate::schema_state::SchemaState, to: i64) -> crate::error::Result<SchemaWalk>
```



