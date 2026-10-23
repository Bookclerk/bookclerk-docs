**bookclerk_library > host_schema**

# Module: host_schema

## Contents

**Structs**

- [`SchemaApplyOptions`](#schemaapplyoptions) - Options for [`apply_host_schema_with_options`](host_schema.md#bookclerk_libraryhost_schemaapply_host_schema_with_options).
- [`SchemaBatch`](#schemabatch) - Canonical schema apply batch: already-separate host statements plus marker.

**Functions**

- [`apply_host_schema`](#apply_host_schema) - Applies pending host-authored DDL from the canonical migration plan.
- [`apply_host_schema_with_batch`](#apply_host_schema_with_batch) - Applies schema using `run_batch` (typed `executeAtomic`) for each version.
- [`apply_host_schema_with_batch_opts`](#apply_host_schema_with_batch_opts) - Applies schema with backups using `run_batch` for each version.
- [`apply_host_schema_with_options`](#apply_host_schema_with_options) - Applies pending host schema with optional in-place backups.
- [`current_schema_state`](#current_schema_state) - Reads explicit [`SchemaState`](schema_state.md#bookclerk_libraryschema_stateschemastate) in the Bookclerk host/bootstrap namespace.
- [`current_schema_state_in`](#current_schema_state_in) - Reads explicit [`SchemaState`](schema_state.md#bookclerk_libraryschema_stateschemastate) for one ledger `namespace`.
- [`current_schema_version`](#current_schema_version) - Frozen revision when the database is `SchemaState::Frozen`; otherwise `0`.
- [`ensure_restore_target_is_replaceable`](#ensure_restore_target_is_replaceable) - Fails closed when this binary cannot interpret the target database's
- [`migrate_host_schema_to`](#migrate_host_schema_to) - Migrates toward `target`, including reversible downs for CLI rollback.
- [`migrate_host_schema_to_with_batch`](#migrate_host_schema_to_with_batch) - Migrates toward `target` using `run_batch` (typed `executeAtomic`) per version.
- [`require_schema_migrations`](#require_schema_migrations) - Requires the `schemaMigrations` versioning contract from typed
- [`schema_state_compatible`](#schema_state_compatible) - True when `state` is exactly the schema this binary applies: a start

**Constants**

- [`MIGRATION_LEASE_TTL`](#migration_lease_ttl) - How long a schema migration lease lasts without renewal. The migrator
- [`MIGRATION_LEASE_WAIT`](#migration_lease_wait) - How long a starter waits for another starter's migration before it fails

---

## bookclerk_library::host_schema::MIGRATION_LEASE_TTL

*Constant*: `std::time::Duration`

How long a schema migration lease lasts without renewal. The migrator
renews it every third of this while it works.



## bookclerk_library::host_schema::MIGRATION_LEASE_WAIT

*Constant*: `std::time::Duration`

How long a starter waits for another starter's migration before it fails
closed. Longer than [`MIGRATION_LEASE_TTL`](host_schema.md#bookclerk_libraryhost_schemamigration_lease_ttl), so a migrator that died is
replaced by a waiter rather than failing every waiter.



## bookclerk_library::host_schema::SchemaApplyOptions

*Struct*

Options for [`apply_host_schema_with_options`](host_schema.md#bookclerk_libraryhost_schemaapply_host_schema_with_options).

**Fields:**
- `backup: Option<crate::backup::SchemaBackupOpts>` - When set, write a recovery point before applying DDL to a non-empty database.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SchemaApplyOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> SchemaApplyOptions`



## bookclerk_library::host_schema::SchemaBatch

*Struct*

Canonical schema apply batch: already-separate host statements plus marker.

Adapters lower each statement at execution; they must not re-split SQL.

**Fields:**
- `statements: Vec<String>` - Ordered SQL strings; the last statement is the version/state marker.

**Methods:**

- `fn from_statements_and_marker<impl Into<String>, impl IntoIterator<Item = impl Into<String>>, impl Into<String>>(stmts: impl Trait, marker: impl Trait) -> Self` - Builds a batch from an ordered statement list plus a trailing marker.
- `fn unreleased<impl Into<String>, impl IntoIterator<Item = impl Into<String>>>(stmts: impl Trait, checksum: &str) -> Self` - Unreleased development pack plus `bookclerk_schema_migrations` unreleased row.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SchemaBatch) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> SchemaBatch`



## bookclerk_library::host_schema::apply_host_schema

*Function*

Applies pending host-authored DDL from the canonical migration plan.

In-process adapters (sqlite tests, native postgres tests) infer the engine
from the opened connection. Production plugin-host uses
[`apply_host_schema_with_batch_opts`](host_schema.md#bookclerk_libraryhost_schemaapply_host_schema_with_batch_opts) so adapters lower canonical DDL after
RPC.

# Errors

Returns [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror) when a version read or DDL statement fails, the
database is newer than this binary, or a frozen checksum does not match.

```rust
fn apply_host_schema(db: &sea_orm::DatabaseConnection) -> crate::error::Result<()>
```



## bookclerk_library::host_schema::apply_host_schema_with_batch

*Function*

Applies schema using `run_batch` (typed `executeAtomic`) for each version.

# Errors

Returns [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror) when a version read, DDL statement, or batch fails.

```rust
fn apply_host_schema_with_batch<F, Fut>(db: &sea_orm::DatabaseConnection, run_batch: F) -> crate::error::Result<()>
```



## bookclerk_library::host_schema::apply_host_schema_with_batch_opts

*Function*

Applies schema with backups using `run_batch` for each version.

# Errors

Returns [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror) when a version read, backup, DDL, or batch fails.

```rust
fn apply_host_schema_with_batch_opts<F, Fut>(db: &sea_orm::DatabaseConnection, opts: SchemaApplyOptions, run_batch: F) -> crate::error::Result<()>
```



## bookclerk_library::host_schema::apply_host_schema_with_options

*Function*

Applies pending host schema with optional in-place backups.

# Errors

Returns [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror) when a version read, backup, or DDL statement fails.

```rust
fn apply_host_schema_with_options(db: &sea_orm::DatabaseConnection, opts: SchemaApplyOptions) -> crate::error::Result<()>
```



## bookclerk_library::host_schema::current_schema_state

*Function*

Reads explicit [`SchemaState`](schema_state.md#bookclerk_libraryschema_stateschemastate) in the Bookclerk host/bootstrap namespace.

An empty `bookclerk_schema_migrations` table is uninitialized; leftover SQLite
`PRAGMA user_version` is ignored.

# Errors

Returns `LibraryError::Schema` on malformed, partial, or contradictory markers.

```rust
fn current_schema_state(db: &sea_orm::DatabaseConnection) -> crate::error::Result<crate::schema_state::SchemaState>
```



## bookclerk_library::host_schema::current_schema_state_in

*Function*

Reads explicit [`SchemaState`](schema_state.md#bookclerk_libraryschema_stateschemastate) for one ledger `namespace`.

Host library and binding bootstrap use [`BOOKCLERK_SCHEMA_NAMESPACE`](migrations/plan.md#bookclerk_librarymigrationsplanbookclerk_schema_namespace).
Frozen checksums are verified against [`host_migration_plan`](migrations.md#bookclerk_librarymigrationshost_migration_plan) only for
the Bookclerk namespace. Plugin-owned history uses `bookclerk_plugin_migrations`,
not this ledger.

# Errors

Returns `LibraryError::Schema` on malformed, partial, or contradictory markers.

```rust
fn current_schema_state_in(db: &sea_orm::DatabaseConnection, namespace: &str) -> crate::error::Result<crate::schema_state::SchemaState>
```



## bookclerk_library::host_schema::current_schema_version

*Function*

Frozen revision when the database is `SchemaState::Frozen`; otherwise `0`.

This is **not** a discriminator for uninitialized vs unreleased. Prefer
[`current_schema_state`](host_schema.md#bookclerk_libraryhost_schemacurrent_schema_state). `0` means “no frozen revision,” never “applied
development schema.”

# Errors

Returns when the state query fails or the marker is malformed.

```rust
fn current_schema_version(db: &sea_orm::DatabaseConnection) -> crate::error::Result<i64>
```



## bookclerk_library::host_schema::ensure_restore_target_is_replaceable

*Function*

Fails closed when this binary cannot interpret the target database's
schema history well enough to drop every unknown table before restore.

Uninitialized targets are replaceable. Frozen history newer than this
binary, unreleased checksums this binary does not know, and unreadable
markers are not.

# Errors

Returns `LibraryError::Schema` when the target is newer or uninterpretable.

```rust
fn ensure_restore_target_is_replaceable(db: &sea_orm::DatabaseConnection) -> crate::error::Result<()>
```



## bookclerk_library::host_schema::migrate_host_schema_to

*Function*

Migrates toward `target`, including reversible downs for CLI rollback.

# Errors

Returns [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror) when the walk cannot start, a backup fails, or DDL fails.
A blocked downgrade still applies every reversible step and returns the walk
(the caller treats `blocked` / `stopped_at != target` as failure).

```rust
fn migrate_host_schema_to(db: &sea_orm::DatabaseConnection, target: i64, opts: SchemaApplyOptions) -> crate::error::Result<crate::schema_walk::SchemaWalk>
```



## bookclerk_library::host_schema::migrate_host_schema_to_with_batch

*Function*

Migrates toward `target` using `run_batch` (typed `executeAtomic`) per version.

A walk that writes anything (backup or DDL) runs under the
[`MIGRATION_LEASE`](control_plane/leases.md#bookclerk_librarycontrol_planeleasesmigration_lease) cluster lease, like a start that migrates: it waits
up to [`MIGRATION_LEASE_WAIT`](host_schema.md#bookclerk_libraryhost_schemamigration_lease_wait) for another migrator, renews the lease
while it works, and checks it before every DDL batch. A no-op walk only
checks the database and takes no lease.

# Errors

Returns [`LibraryError`](error.md#bookclerk_libraryerrorlibraryerror) when the walk cannot start, a backup fails, a
batch fails, another migrator holds the lease past the wait, or the lease
is lost mid-walk.

```rust
fn migrate_host_schema_to_with_batch<F, Fut>(db: &sea_orm::DatabaseConnection, target: i64, opts: SchemaApplyOptions, run_batch: F) -> crate::error::Result<crate::schema_walk::SchemaWalk>
```



## bookclerk_library::host_schema::require_schema_migrations

*Function*

Requires the `schemaMigrations` versioning contract from typed
[`bookclerk_plugin_abi::DbCapabilities`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executedbcapabilities).

The host always records versions in canonical `bookclerk_schema_migrations`
rows; canonical Bookclerk SQL is [`crate::migrations::current_canonical_schema`](migrations.md#bookclerk_librarymigrationscurrent_canonical_schema)
and adapters lower it for the live connection backend at execution. Plugin
identity and diagnostic engine names are not consulted.

# Errors

Returns `LibraryError::Other` when `schemaMigrations` is missing.

```rust
fn require_schema_migrations(caps: &bookclerk_plugin_abi::DbCapabilities) -> crate::error::Result<()>
```



## bookclerk_library::host_schema::schema_state_compatible

*Function*

True when `state` is exactly the schema this binary applies: a start
against it neither migrates nor fails closed.

```rust
fn schema_state_compatible(state: &crate::schema_state::SchemaState) -> bool
```



