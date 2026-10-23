**bookclerk_library > migrations > plan**

# Module: migrations::plan

## Contents

**Structs**

- [`MigrationPlan`](#migrationplan) - Ordered frozen plan for one ledger namespace.
- [`MigrationStep`](#migrationstep) - One ordered schema revision: checksumed `up` ops and optional `down` ops.

**Enums**

- [`PlanOp`](#planop) - One admitted BookclerkSQL statement owned by a [`MigrationStep`](plan.md#bookclerk_librarymigrationsplanmigrationstep).

**Functions**

- [`frozen_marker_delete_sql`](#frozen_marker_delete_sql) - `DELETE` of one frozen step's namespaced marker.
- [`frozen_marker_sql`](#frozen_marker_sql) - `INSERT` for a frozen namespaced `bookclerk_schema_migrations` row.
- [`schema_slot_key`](#schema_slot_key) - Slot key that serializes schema apply for one namespace.
- [`sql_string_literal`](#sql_string_literal) - SQL string literal with `'` doubled.
- [`unreleased_marker_sql_in`](#unreleased_marker_sql_in) - `INSERT` for an unreleased namespaced `bookclerk_schema_migrations` row.
- [`validate_schema_namespace`](#validate_schema_namespace) - True when `namespace` is `bookclerk` or a plugin-id grammar string.

**Constants**

- [`BOOKCLERK_SCHEMA_NAMESPACE`](#bookclerk_schema_namespace) - Ledger namespace for host library schema and Bookclerk-owned binding bootstrap.

---

## bookclerk_library::migrations::plan::BOOKCLERK_SCHEMA_NAMESPACE

*Constant*: `&str`

Ledger namespace for host library schema and Bookclerk-owned binding bootstrap.



## bookclerk_library::migrations::plan::MigrationPlan

*Struct*

Ordered frozen plan for one ledger namespace.

**Fields:**
- `namespace: String` - `bookclerk_schema_migrations.namespace` (`bookclerk` for host-owned plans).
- `steps: Vec<MigrationStep>` - Frozen steps in increasing version order.

**Methods:**

- `fn try_new<impl Into<String>>(namespace: impl Trait, steps: Vec<MigrationStep>) -> Result<Self>` - Builds a plan after proving every op and checking version order.
- `fn min_version(self: &Self) -> Option<i64>` - Oldest frozen version (`None` when this plan has no frozen steps).
- `fn max_version(self: &Self) -> i64` - Highest frozen version in this plan (`0` when empty).
- `fn step(self: &Self, version: i64) -> Option<&MigrationStep>` - Step with `version`, if present.
- `fn checksums_through(self: &Self, through: i64) -> std::collections::HashMap<i64, String>` - Expected checksum map for frozen rows through `through`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MigrationPlan`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &MigrationPlan) -> bool`



## bookclerk_library::migrations::plan::MigrationStep

*Struct*

One ordered schema revision: checksumed `up` ops and optional `down` ops.

**Fields:**
- `version: i64` - Monotonic schema revision recorded on the namespaced ledger.
- `up: Vec<PlanOp>` - Ordered schema and data ops for this version.
- `down: Option<Vec<PlanOp>>` - Reverse ops when this step is reversible; `None` means restore a backup.
- `introduced_in: String` - First package/semver that shipped this step.

**Methods:**

- `fn checksum(self: &Self) -> String` - SHA-256 hex digest of length-prefixed `up` (and `down` when present).
- `fn reversible(self: &Self) -> bool` - True when `Self::down` is present so an explicit downgrade can apply it.
- `fn from_host(step: &HostMigrationStep) -> Self` - Converts a static host step into an owned engine step.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MigrationStep`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &MigrationStep) -> bool`



## bookclerk_library::migrations::plan::PlanOp

*Enum*

One admitted BookclerkSQL statement owned by a [`MigrationStep`](plan.md#bookclerk_librarymigrationsplanmigrationstep).

**Variants:**
- `Schema(String)` - Schema mutation (`CREATE` / `DROP` / `ALTER` / index).
- `Data(String)` - Data backfill (`INSERT` / `UPDATE` / `DELETE`).

**Methods:**

- `fn schema_static(sql: &'static str) -> Self` - Schema op from static host SQL.
- `fn sql(self: &Self) -> &str` - Canonical BookclerkSQL for this op.
- `fn is_schema(self: &Self) -> bool` - True when this op is admitted schema DDL.
- `fn from_static(op: MigrationOp) -> Self` - Converts a host static op into an owned plan op.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PlanOp`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PlanOp) -> bool`



## bookclerk_library::migrations::plan::frozen_marker_delete_sql

*Function*

`DELETE` of one frozen step's namespaced marker.

```rust
fn frozen_marker_delete_sql(namespace: &str, version: i64) -> String
```



## bookclerk_library::migrations::plan::frozen_marker_sql

*Function*

`INSERT` for a frozen namespaced `bookclerk_schema_migrations` row.

```rust
fn frozen_marker_sql(namespace: &str, version: i64, checksum: &str) -> String
```



## bookclerk_library::migrations::plan::schema_slot_key

*Function*

Slot key that serializes schema apply for one namespace.

```rust
fn schema_slot_key(namespace: &str) -> String
```



## bookclerk_library::migrations::plan::sql_string_literal

*Function*

SQL string literal with `'` doubled.

```rust
fn sql_string_literal(value: &str) -> String
```



## bookclerk_library::migrations::plan::unreleased_marker_sql_in

*Function*

`INSERT` for an unreleased namespaced `bookclerk_schema_migrations` row.

```rust
fn unreleased_marker_sql_in(namespace: &str, checksum: &str, base_version: i64) -> String
```



## bookclerk_library::migrations::plan::validate_schema_namespace

*Function*

True when `namespace` is `bookclerk` or a plugin-id grammar string.

# Errors

Returns when the namespace is empty, too long, or contains characters other
than lowercase ascii, digits, and single `_`.

```rust
fn validate_schema_namespace(namespace: &str) -> crate::error::Result<()>
```



