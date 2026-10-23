**bookclerk_library > migrations**

# Module: migrations

## Contents

**Structs**

- [`HostMigrationStep`](#hostmigrationstep) - One host-owned schema version in the canonical Bookclerk migration plan.

**Enums**

- [`MigrationOp`](#migrationop) - One admitted BookclerkSQL statement in a host migration apply unit.

**Functions**

- [`binding_bootstrap_ops`](#binding_bootstrap_ops) - Host-owned bootstrap ops applied inside every isolated plugin binding.
- [`binding_bootstrap_sql`](#binding_bootstrap_sql) - Derived diagnostic SQL for [`binding_bootstrap_ops`](migrations.md#bookclerk_librarymigrationsbinding_bootstrap_ops).
- [`binding_bootstrap_statements`](#binding_bootstrap_statements) - Ordered canonical statements for [`binding_bootstrap_ops`](migrations.md#bookclerk_librarymigrationsbinding_bootstrap_ops).
- [`binding_bootstrap_type_env`](#binding_bootstrap_type_env) - Type environment implied by [`binding_bootstrap_ops`](migrations.md#bookclerk_librarymigrationsbinding_bootstrap_ops) (journal, slots, catalog).
- [`binding_unreleased_checksum`](#binding_unreleased_checksum) - SHA-256 of [`binding_bootstrap_ops`](migrations.md#bookclerk_librarymigrationsbinding_bootstrap_ops) (length-prefixed statement list).
- [`current_canonical_schema`](#current_canonical_schema) - Frozen ups concatenated with [`unreleased_ops`](migrations.md#bookclerk_librarymigrationsunreleased_ops) (derived diagnostic SQL).
- [`current_canonical_statements`](#current_canonical_statements) - Ordered canonical statements for [`current_canonical_schema`](migrations.md#bookclerk_librarymigrationscurrent_canonical_schema).
- [`current_canonical_table_names`](#current_canonical_table_names) - Host table names declared by [`current_canonical_schema`](migrations.md#bookclerk_librarymigrationscurrent_canonical_schema), plus
- [`host_migration_plan`](#host_migration_plan) - Frozen host migration steps. Empty until a release cut copies
- [`host_sql_type_env`](#host_sql_type_env) - Column types implied by [`current_canonical_schema`](migrations.md#bookclerk_librarymigrationscurrent_canonical_schema), plus the
- [`join_op_sql`](#join_op_sql) - Joins already-separated ops with `;\n` for diagnostics/export.
- [`latest_schema_sqlite`](#latest_schema_sqlite) - Final SQLite DDL for a fresh Bookclerk library database.
- [`migration_ops_checksum`](#migration_ops_checksum) - SHA-256 of length-prefixed migration ops (not a joined script).
- [`migration_sql_checksum`](#migration_sql_checksum) - SHA-256 of canonical up statements, plus down statements when reversible.
- [`min_supported_schema_version`](#min_supported_schema_version) - Oldest frozen schema version this binary can run.
- [`min_supported_schema_version_in`](#min_supported_schema_version_in) - Oldest frozen version in `plan` (`None` when `plan` is empty).
- [`pack_migration_sql`](#pack_migration_sql) - Packs `sql` with the SQL-v1 lexer. Empty input yields no statements.
- [`prove_migration_ops`](#prove_migration_ops) - Proves each op is exactly one BookclerkSQL statement of the declared kind.
- [`require_single_migration_statement`](#require_single_migration_statement) - Packs `sql` and requires exactly one statement.
- [`unreleased_checksum`](#unreleased_checksum) - SHA-256 of [`unreleased_ops`](migrations.md#bookclerk_librarymigrationsunreleased_ops) (empty string when empty).
- [`unreleased_ops`](#unreleased_ops) - Live unreleased host schema ops (source of truth).
- [`unreleased_sql`](#unreleased_sql) - Live development DDL derived from [`unreleased_ops`](migrations.md#bookclerk_librarymigrationsunreleased_ops) (joined with `;\n`).
- [`unreleased_state_marker_sql`](#unreleased_state_marker_sql) - `INSERT` for an unreleased `bookclerk_schema_migrations` row in the Bookclerk namespace.
- [`unreleased_statements`](#unreleased_statements) - Ordered statements in [`unreleased_ops`](migrations.md#bookclerk_librarymigrationsunreleased_ops).

**Constants**

- [`BINDING_SCHEMA_VERSION`](#binding_schema_version) - Highest frozen binding-bootstrap version (`0` until a binding freeze).
- [`CLUSTER_LEASES_DDL`](#cluster_leases_ddl) - `cluster_leases`: named singleton leases (scheduler, maintenance duties,
- [`SCHEMA_MIGRATIONS_DDL`](#schema_migrations_ddl) - Host bookkeeping table created before applying plan versions.
- [`SCHEMA_VERSION`](#schema_version) - Highest **frozen** schema version this binary knows (`0` while the plan is empty).

---

## bookclerk_library::migrations::BINDING_SCHEMA_VERSION

*Constant*: `i64`

Highest frozen binding-bootstrap version (`0` until a binding freeze).



## bookclerk_library::migrations::CLUSTER_LEASES_DDL

*Constant*: `&str`

`cluster_leases`: named singleton leases (scheduler, maintenance duties,
schema migration).

Part of the unreleased pack, and also created on its own before a
schema migration, because the migration lease must exist before the pack
that defines it is applied. Both paths run the same `IF NOT EXISTS`
statement, so whichever runs second is a no-op.



## bookclerk_library::migrations::HostMigrationStep

*Struct*

One host-owned schema version in the canonical Bookclerk migration plan.

Each version is recorded as a `bookclerk_schema_migrations` row (adapters
must advertise `schemaMigrations`). The live connection backend lowers
each [`MigrationOp`](migrations.md#bookclerk_librarymigrationsmigrationop) at the adapter boundary (Postgres) or applies it
verbatim (SQLite / D1).

**Fields:**
- `version: i64` - `bookclerk_schema_migrations.version` for this step.
- `steps: &'static [MigrationOp]` - Ordered schema and data ops for this version (the `up`).
- `down: Option<&'static [MigrationOp]>` - Reverse ops when this step is reversible; `None` means restore a backup.
- `introduced_in: &'static str` - First Bookclerk semver that shipped this step.

**Methods:**

- `fn checksum(self: &Self) -> String` - SHA-256 hex digest of length-prefixed `up` (and `down` when present).
- `fn reversible(self: &Self) -> bool` - True when `Self::down` is present so CLI rollback can apply this step.
- `fn up_sql(self: &Self) -> String` - Concatenated up SQL (tests / derived views). Boundaries are the op list.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> HostMigrationStep`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &HostMigrationStep) -> bool`



## bookclerk_library::migrations::MigrationOp

*Enum*

One admitted BookclerkSQL statement in a host migration apply unit.

**Variants:**
- `Schema(&'static str)` - Schema mutation (`CREATE` / `DROP` / `ALTER` / index).
- `Data(&'static str)` - Data backfill (`INSERT` / `UPDATE` / `DELETE`).

**Methods:**

- `fn sql(self: Self) -> &'static str` - Canonical BookclerkSQL for this op.
- `fn is_schema(self: Self) -> bool` - True when this op is admitted schema DDL.
- `fn is_data(self: Self) -> bool` - True when this op is admitted DML / backfill.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MigrationOp`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &MigrationOp) -> bool`



## bookclerk_library::migrations::SCHEMA_MIGRATIONS_DDL

*Constant*: `&str`

Host bookkeeping table created before applying plan versions.

`namespace` separates Bookclerk-owned ledger rows (`bookclerk`) from any
leftover rows. Plugin-owned history uses the separate `bookclerk_plugin_migrations`
journal, not this table.



## bookclerk_library::migrations::SCHEMA_VERSION

*Constant*: `i64`

Highest **frozen** schema version this binary knows (`0` while the plan is empty).

This is not a discriminator for uninitialized vs unreleased. Use
[`crate::SchemaState`](schema_state.md#bookclerk_libraryschema_stateschemastate).



## bookclerk_library::migrations::binding_bootstrap_ops

*Function*

Host-owned bootstrap ops applied inside every isolated plugin binding.

# Panics

Panics when a bootstrap op is not one admitted BookclerkSQL statement of
the declared [`MigrationOp`](migrations.md#bookclerk_librarymigrationsmigrationop) kind.

```rust
fn binding_bootstrap_ops() -> &'static [MigrationOp]
```



## bookclerk_library::migrations::binding_bootstrap_sql

*Function*

Derived diagnostic SQL for [`binding_bootstrap_ops`](migrations.md#bookclerk_librarymigrationsbinding_bootstrap_ops).

```rust
fn binding_bootstrap_sql() -> &'static str
```



## bookclerk_library::migrations::binding_bootstrap_statements

*Function*

Ordered canonical statements for [`binding_bootstrap_ops`](migrations.md#bookclerk_librarymigrationsbinding_bootstrap_ops).

```rust
fn binding_bootstrap_statements() -> &'static [String]
```



## bookclerk_library::migrations::binding_bootstrap_type_env

*Function*

Type environment implied by [`binding_bootstrap_ops`](migrations.md#bookclerk_librarymigrationsbinding_bootstrap_ops) (journal, slots, catalog).

Host-private binding bookkeeping lives here so plugin apply can typecheck
journal DML without using the host library catalog.

```rust
fn binding_bootstrap_type_env() -> bookclerk_plugin_abi::SqlTypeEnv
```



## bookclerk_library::migrations::binding_unreleased_checksum

*Function*

SHA-256 of [`binding_bootstrap_ops`](migrations.md#bookclerk_librarymigrationsbinding_bootstrap_ops) (length-prefixed statement list).

```rust
fn binding_unreleased_checksum() -> String
```



## bookclerk_library::migrations::current_canonical_schema

*Function*

Frozen ups concatenated with [`unreleased_ops`](migrations.md#bookclerk_librarymigrationsunreleased_ops) (derived diagnostic SQL).

While the production plan is empty this is exactly [`unreleased_sql`](migrations.md#bookclerk_librarymigrationsunreleased_sql). After
a release cut it is frozen step SQL plus whatever is again unreleased — do
not assume it equals [`unreleased_sql`](migrations.md#bookclerk_librarymigrationsunreleased_sql) forever. Test plan overrides must
not feed this `OnceLock` (`production_host_migration_plan` only).

```rust
fn current_canonical_schema() -> &'static str
```



## bookclerk_library::migrations::current_canonical_statements

*Function*

Ordered canonical statements for [`current_canonical_schema`](migrations.md#bookclerk_librarymigrationscurrent_canonical_schema).

```rust
fn current_canonical_statements() -> &'static [String]
```



## bookclerk_library::migrations::current_canonical_table_names

*Function*

Host table names declared by [`current_canonical_schema`](migrations.md#bookclerk_librarymigrationscurrent_canonical_schema), plus
`bookclerk_schema_migrations`.

```rust
fn current_canonical_table_names() -> Vec<String>
```



## bookclerk_library::migrations::host_migration_plan

*Function*

Frozen host migration steps. Empty until a release cut copies
[`unreleased_ops`](migrations.md#bookclerk_librarymigrationsunreleased_ops) into version 1.

```rust
fn host_migration_plan() -> Vec<HostMigrationStep>
```



## bookclerk_library::migrations::host_sql_type_env

*Function*

Column types implied by [`current_canonical_schema`](migrations.md#bookclerk_librarymigrationscurrent_canonical_schema), plus the
`bookclerk_schema_migrations` ledger created before any plan step.

```rust
fn host_sql_type_env() -> bookclerk_plugin_abi::SqlTypeEnv
```



## bookclerk_library::migrations::join_op_sql

*Function*

Joins already-separated ops with `;\n` for diagnostics/export.

```rust
fn join_op_sql(ops: &[MigrationOp]) -> String
```



## bookclerk_library::migrations::latest_schema_sqlite

*Function*

Final SQLite DDL for a fresh Bookclerk library database.

SeaORM entities in [`crate::entities`](entities.md) mirror these columns exactly. All
integer columns map to `i64`, reals to `f64`, blobs to `Vec<u8>`, and text
(including RFC 3339 timestamps) to `String`.

```rust
fn latest_schema_sqlite() -> &'static str
```



## bookclerk_library::migrations::migration_ops_checksum

*Function*

SHA-256 of length-prefixed migration ops (not a joined script).

```rust
fn migration_ops_checksum(ups: &[MigrationOp], down: Option<&[MigrationOp]>) -> String
```



## bookclerk_library::migrations::migration_sql_checksum

*Function*

SHA-256 of canonical up statements, plus down statements when reversible.

# Errors

Returns when `canonical` or `down` is not a BookclerkSQL statement list.
Parser failure is never treated as a single opaque statement.

```rust
fn migration_sql_checksum(canonical: &str, down: Option<&str>) -> crate::error::Result<String>
```



## bookclerk_library::migrations::min_supported_schema_version

*Function*

Oldest frozen schema version this binary can run.

Derived from [`host_migration_plan`]: `None` while the plan is empty
(no frozen schema versions exist). Once frozen steps exist this is the
first retained step, not a separately synchronized constant.

```rust
fn min_supported_schema_version() -> Option<i64>
```



## bookclerk_library::migrations::min_supported_schema_version_in

*Function*

Oldest frozen version in `plan` (`None` when `plan` is empty).

```rust
fn min_supported_schema_version_in(plan: &[HostMigrationStep]) -> Option<i64>
```



## bookclerk_library::migrations::pack_migration_sql

*Function*

Packs `sql` with the SQL-v1 lexer. Empty input yields no statements.

# Errors

Returns when `sql` is not a BookclerkSQL statement list.

```rust
fn pack_migration_sql(sql: &str) -> crate::error::Result<Vec<String>>
```



## bookclerk_library::migrations::prove_migration_ops

*Function*

Proves each op is exactly one BookclerkSQL statement of the declared kind.

Schema ops must be admitted DDL (`CREATE` / `ALTER` / `DROP`). Data ops must
be admitted DML. Both go through the SQL-v1 typechecker; Schema ops also
update `env` so later Data/index ops see prior `CREATE TABLE`.

# Errors

Returns when packing, kind, or typechecking fails.

```rust
fn prove_migration_ops(ops: &[MigrationOp]) -> crate::error::Result<()>
```



## bookclerk_library::migrations::require_single_migration_statement

*Function*

Packs `sql` and requires exactly one statement.

# Errors

Returns when packing fails or `sql` is not a single statement.

```rust
fn require_single_migration_statement(sql: &str) -> crate::error::Result<String>
```



## bookclerk_library::migrations::unreleased_checksum

*Function*

SHA-256 of [`unreleased_ops`](migrations.md#bookclerk_librarymigrationsunreleased_ops) (empty string when empty).

```rust
fn unreleased_checksum() -> String
```



## bookclerk_library::migrations::unreleased_ops

*Function*

Live unreleased host schema ops (source of truth).

# Panics

Panics when an unreleased op is not one admitted BookclerkSQL statement of
the declared [`MigrationOp`](migrations.md#bookclerk_librarymigrationsmigrationop) kind.

```rust
fn unreleased_ops() -> &'static [MigrationOp]
```



## bookclerk_library::migrations::unreleased_sql

*Function*

Live development DDL derived from [`unreleased_ops`](migrations.md#bookclerk_librarymigrationsunreleased_ops) (joined with `;\n`).

Apply, checksum, and backup use the op list. This script is diagnostics,
export, and `execute_batch` tests only.

```rust
fn unreleased_sql() -> &'static str
```



## bookclerk_library::migrations::unreleased_state_marker_sql

*Function*

`INSERT` for an unreleased `bookclerk_schema_migrations` row in the Bookclerk namespace.

```rust
fn unreleased_state_marker_sql(checksum: &str, base_version: i64) -> String
```



## bookclerk_library::migrations::unreleased_statements

*Function*

Ordered statements in [`unreleased_ops`](migrations.md#bookclerk_librarymigrationsunreleased_ops).

```rust
fn unreleased_statements() -> &'static [String]
```



