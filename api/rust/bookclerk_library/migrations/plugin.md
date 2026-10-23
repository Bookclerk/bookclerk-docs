**bookclerk_library > migrations > plugin**

# Module: migrations::plugin

## Contents

**Structs**

- [`PluginJournalEntry`](#pluginjournalentry) - One durable host-private journal row.
- [`PluginMigrationHistory`](#pluginmigrationhistory) - Ordered durable plugin migration history.
- [`PluginMigrationSequence`](#pluginmigrationsequence) - Proven, ordered plugin migration sequence.
- [`ProvenPluginMigration`](#provenpluginmigration) - One proven registered migration.

**Functions**

- [`apply_plugin_migrations`](#apply_plugin_migrations) - Applies the pending registered suffix. Idempotent when history already matches.
- [`history_from_execute_reply`](#history_from_execute_reply) - Parses journal rows from a typed select reply.
- [`load_plugin_migration_history`](#load_plugin_migration_history) - Reads the host-private plugin migration journal.
- [`load_plugin_migration_history_on`](#load_plugin_migration_history_on) - Reads the journal on an already-open connection (including a backup capture
- [`next_pending_plugin_migration`](#next_pending_plugin_migration) - Next registered migration after a verified durable prefix, if any.
- [`pending_plugin_suffix`](#pending_plugin_suffix) - Pending registered migrations after the durable prefix.
- [`plugin_apply_statements`](#plugin_apply_statements) - One atomic apply unit: serialization slot, registered ops, host journal append.
- [`plugin_binding_type_env`](#plugin_binding_type_env) - Binding catalog after host bootstrap plus schema ops from the first `applied`
- [`plugin_history_digest`](#plugin_history_digest) - SHA-256 of ordered `(migration_id, checksum)` pairs.
- [`plugin_history_session_matches`](#plugin_history_session_matches) - True when a binding session's captured history digest still matches durable history.
- [`plugin_journal_has_entry`](#plugin_journal_has_entry) - True when `history` already contains this `(ordinal, id, checksum)` row.
- [`plugin_journal_select_request`](#plugin_journal_select_request) - Typed SELECT used by RPC hosts to read the plugin migration journal.
- [`plugin_migration_checksum`](#plugin_migration_checksum) - Checksum of one registered migration (id + classified SQL).
- [`prove_plugin_migration_sequence`](#prove_plugin_migration_sequence) - Proves a complete registration with one evolving [`SqlTypeEnv`](../../bookclerk_plugin_abi/sql_types.md#bookclerk_plugin_abisql_typessqltypeenv).
- [`remaining_plugin_suffix_batches`](#remaining_plugin_suffix_batches) - Remaining atomic apply batches for RPC hosts that cannot run SeaORM apply.
- [`require_history_prefix`](#require_history_prefix) - Fail closed unless durable history is an exact prefix of `registered`.

**Constants**

- [`MAX_PLUGIN_MIGRATION_APPLY_ATTEMPTS`](#max_plugin_migration_apply_attempts) - Bounded retries for **one** expected plugin migration apply unit.
- [`PLUGIN_MIGRATION_SLOT_KEY`](#plugin_migration_slot_key) - Per-binding serialization slot for plugin migration apply.

---

## bookclerk_library::migrations::plugin::MAX_PLUGIN_MIGRATION_APPLY_ATTEMPTS

*Constant*: `usize`

Bounded retries for **one** expected plugin migration apply unit.

Independent of how many migrations are registered. A fresh binding may
apply the complete registration; this only caps retries of a single
expected `(ordinal, id, checksum)` while durable history is unchanged.



## bookclerk_library::migrations::plugin::PLUGIN_MIGRATION_SLOT_KEY

*Constant*: `&str`

Per-binding serialization slot for plugin migration apply.



## bookclerk_library::migrations::plugin::PluginJournalEntry

*Struct*

One durable host-private journal row.

**Fields:**
- `ordinal: i64` - Host-private storage position (`0..n-1`). Not a plugin version.
- `migration_id: String` - Opaque plugin-chosen identity.
- `checksum: String` - Checksum recorded when this row was applied.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginJournalEntry) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PluginJournalEntry`



## bookclerk_library::migrations::plugin::PluginMigrationHistory

*Struct*

Ordered durable plugin migration history.

**Fields:**
- `entries: Vec<PluginJournalEntry>` - Journal rows in ordinal order.

**Methods:**

- `fn digest(self: &Self) -> String` - SHA-256 digest of the complete ordered `(migration_id, checksum)` list.
- `fn len(self: &Self) -> usize` - Number of applied journal rows.
- `fn is_empty(self: &Self) -> bool` - True when no plugin migrations have been applied.
- `fn display(self: &Self) -> String` - Operator-facing history label (`history@{n}+{digest}`).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginMigrationHistory`
- **Default**
  - `fn default() -> PluginMigrationHistory`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginMigrationHistory) -> bool`



## bookclerk_library::migrations::plugin::PluginMigrationSequence

*Struct*

Proven, ordered plugin migration sequence.

**Fields:**
- `migrations: Vec<ProvenPluginMigration>` - Registration-order migrations with checksums.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginMigrationSequence) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PluginMigrationSequence`



## bookclerk_library::migrations::plugin::ProvenPluginMigration

*Struct*

One proven registered migration.

**Fields:**
- `id: String` - Opaque plugin-chosen identity.
- `operations: Vec<bookclerk_plugin_abi::PluginMigrationOp>` - Ordered operations.
- `checksum: String` - Deterministic checksum of `(id, classified ops)`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ProvenPluginMigration) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ProvenPluginMigration`



## bookclerk_library::migrations::plugin::apply_plugin_migrations

*Function*

Applies the pending registered suffix. Idempotent when history already matches.

Outer loop walks durable history (exact-prefix). Each pending migration is
one short atomic unit: serialization-slot mutation + plugin operations +
journal append, with a bounded inner retry loop that does **not** consume
later migration capacity. There is no lock held across those independent
transactions. Concurrent hosts re-read durable history before applying;
uniqueness / conflict / an ambiguous reply causes another journal read.
Matching `(id, checksum)` at the expected ordinal is success.
Contradictory history fails closed.

# Errors

Returns when prefix verification fails, apply exhausts retries, or the
journal is contradictory.

```rust
fn apply_plugin_migrations(db: &sea_orm::DatabaseConnection, registered: &PluginMigrationSequence) -> crate::error::Result<PluginMigrationHistory>
```



## bookclerk_library::migrations::plugin::history_from_execute_reply

*Function*

Parses journal rows from a typed select reply.

# Errors

Returns when row shape or ordinals are invalid.

```rust
fn history_from_execute_reply(reply: &bookclerk_plugin_abi::ExecuteReply) -> crate::error::Result<PluginMigrationHistory>
```



## bookclerk_library::migrations::plugin::load_plugin_migration_history

*Function*

Reads the host-private plugin migration journal.

# Errors

Returns when the journal cannot be read or ordinals are not contiguous.

```rust
fn load_plugin_migration_history(db: &sea_orm::DatabaseConnection) -> crate::error::Result<PluginMigrationHistory>
```



## bookclerk_library::migrations::plugin::load_plugin_migration_history_on

*Function*

Reads the journal on an already-open connection (including a backup capture
transaction).

# Errors

Returns when the journal cannot be read or ordinals are not contiguous.

```rust
fn load_plugin_migration_history_on<C>(conn: &C) -> crate::error::Result<PluginMigrationHistory>
```



## bookclerk_library::migrations::plugin::next_pending_plugin_migration

*Function*

Next registered migration after a verified durable prefix, if any.

Uses [`require_history_prefix`](plugin.md#bookclerk_librarymigrationspluginrequire_history_prefix) then indexes `registered[history.len()]`
rather than rebuilding the pending suffix vector.

# Errors

Returns [`require_history_prefix`](plugin.md#bookclerk_librarymigrationspluginrequire_history_prefix) errors.

```rust
fn next_pending_plugin_migration<'a>(history: &PluginMigrationHistory, registered: &'a PluginMigrationSequence) -> crate::error::Result<Option<(usize, &'a ProvenPluginMigration)>>
```



## bookclerk_library::migrations::plugin::pending_plugin_suffix

*Function*

Pending registered migrations after the durable prefix.

# Errors

Returns [`require_history_prefix`](plugin.md#bookclerk_librarymigrationspluginrequire_history_prefix) errors.

```rust
fn pending_plugin_suffix<'a>(history: &PluginMigrationHistory, registered: &'a PluginMigrationSequence) -> crate::error::Result<&'a [ProvenPluginMigration]>
```



## bookclerk_library::migrations::plugin::plugin_apply_statements

*Function*

One atomic apply unit: serialization slot, registered ops, host journal append.

```rust
fn plugin_apply_statements(ordinal: i64, migration: &ProvenPluginMigration) -> Vec<String>
```



## bookclerk_library::migrations::plugin::plugin_binding_type_env

*Function*

Binding catalog after host bootstrap plus schema ops from the first `applied`
registered migrations.

Plugin apply stamps journal DML and suffix data ops against this evolving
environment. `applied` is the durable prefix length (host-private ordinal
count), not a plugin version.

```rust
fn plugin_binding_type_env(registered: &PluginMigrationSequence, applied: usize) -> bookclerk_plugin_abi::SqlTypeEnv
```



## bookclerk_library::migrations::plugin::plugin_history_digest

*Function*

SHA-256 of ordered `(migration_id, checksum)` pairs.

```rust
fn plugin_history_digest(entries: &[PluginJournalEntry]) -> String
```



## bookclerk_library::migrations::plugin::plugin_history_session_matches

*Function*

True when a binding session's captured history digest still matches durable history.

# Errors

Returns when the digest differs (peer advanced or rewritten history).

```rust
fn plugin_history_session_matches(expected_digest: &str, observed: &PluginMigrationHistory) -> crate::error::Result<()>
```



## bookclerk_library::migrations::plugin::plugin_journal_has_entry

*Function*

True when `history` already contains this `(ordinal, id, checksum)` row.

```rust
fn plugin_journal_has_entry(history: &PluginMigrationHistory, ordinal: i64, id: &str, checksum: &str) -> bool
```



## bookclerk_library::migrations::plugin::plugin_journal_select_request

*Function*

Typed SELECT used by RPC hosts to read the plugin migration journal.

```rust
fn plugin_journal_select_request<impl Into<String>>(operation_id: impl Trait, deadline_unix_ms: u64) -> bookclerk_plugin_abi::ExecuteRequest
```



## bookclerk_library::migrations::plugin::plugin_migration_checksum

*Function*

Checksum of one registered migration (id + classified SQL).

```rust
fn plugin_migration_checksum(id: &str, operations: &[bookclerk_plugin_abi::PluginMigrationOp]) -> String
```



## bookclerk_library::migrations::plugin::prove_plugin_migration_sequence

*Function*

Proves a complete registration with one evolving [`SqlTypeEnv`](../../bookclerk_plugin_abi/sql_types.md#bookclerk_plugin_abisql_typessqltypeenv).

Schema operations update the environment. Data operations typecheck against
the current environment. The environment is not reset between migrations.

# Errors

Returns when resource limits are exceeded, an id is invalid, ids repeat,
an operation is empty/malformed, or BookclerkSQL proof fails.
Transport/resource validation is
[`bookclerk_plugin_abi::require_plugin_migration_registration`](../../bookclerk_plugin_abi/plugin_migration.md#bookclerk_plugin_abiplugin_migrationrequire_plugin_migration_registration);
this function is the authoritative semantic proof afterward.

```rust
fn prove_plugin_migration_sequence(migrations: Vec<bookclerk_plugin_abi::PluginMigration>) -> crate::error::Result<PluginMigrationSequence>
```



## bookclerk_library::migrations::plugin::remaining_plugin_suffix_batches

*Function*

Remaining atomic apply batches for RPC hosts that cannot run SeaORM apply.

# Errors

Returns [`require_history_prefix`](plugin.md#bookclerk_librarymigrationspluginrequire_history_prefix) errors.

```rust
fn remaining_plugin_suffix_batches(history: &PluginMigrationHistory, registered: &PluginMigrationSequence) -> crate::error::Result<Vec<Vec<String>>>
```



## bookclerk_library::migrations::plugin::require_history_prefix

*Function*

Fail closed unless durable history is an exact prefix of `registered`.

# Errors

Returns when an applied id/checksum/order does not match, or durable history
is longer than the current registration (older/incompatible plugin).

```rust
fn require_history_prefix(history: &PluginMigrationHistory, registered: &PluginMigrationSequence) -> crate::error::Result<()>
```



