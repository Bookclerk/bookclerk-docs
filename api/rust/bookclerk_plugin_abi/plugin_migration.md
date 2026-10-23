**bookclerk_plugin_abi > plugin_migration**

# Module: plugin_migration

## Contents

**Structs**

- [`PluginMigration`](#pluginmigration) - One plugin-owned migration application in registration order.

**Enums**

- [`PluginMigrationOp`](#pluginmigrationop) - One already-separated BookclerkSQL operation in a registered migration.

**Functions**

- [`capnp_u32_len`](#capnp_u32_len) - Converts a Rust list length into a Cap'n Proto `UInt32` list size.
- [`plugin_migration_registration_bytes`](#plugin_migration_registration_bytes) - Aggregate UTF-8 bytes of ids plus SQL across a complete registration.
- [`require_plugin_migration_json_lists`](#require_plugin_migration_json_lists) - Rejects JSON `databaseMigrations` list sizes before deserializing operations.
- [`require_plugin_migration_registration`](#require_plugin_migration_registration) - Rejects a plugin migration registration that exceeds RPC resource limits.

**Constants**

- [`MAX_PLUGIN_MIGRATION_ID_BYTES`](#max_plugin_migration_id_bytes) - Maximum UTF-8 bytes of one plugin-chosen migration id.
- [`PLUGIN_MIGRATIONS_TABLE`](#plugin_migrations_table) - Host-private plugin-binding migration journal (not a plugin-owned table).

---

## bookclerk_plugin_abi::plugin_migration::MAX_PLUGIN_MIGRATION_ID_BYTES

*Constant*: `usize`

Maximum UTF-8 bytes of one plugin-chosen migration id.



## bookclerk_plugin_abi::plugin_migration::PLUGIN_MIGRATIONS_TABLE

*Constant*: `&str`

Host-private plugin-binding migration journal (not a plugin-owned table).



## bookclerk_plugin_abi::plugin_migration::PluginMigration

*Struct*

One plugin-owned migration application in registration order.

**Fields:**
- `id: String` - Opaque plugin-chosen stable identity.
- `operations: Vec<PluginMigrationOp>` - Ordered already-separated BookclerkSQL operations.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginMigration`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginMigration) -> bool`



## bookclerk_plugin_abi::plugin_migration::PluginMigrationOp

*Enum*

One already-separated BookclerkSQL operation in a registered migration.

**Variants:**
- `Schema(String)` - Admitted schema DDL (`CREATE` / `DROP` / index).
- `Data(String)` - Admitted data DML (`INSERT` / `UPDATE` / `DELETE`).

**Methods:**

- `fn sql(self: &Self) -> &str` - Canonical BookclerkSQL for this operation.
- `fn is_schema(self: &Self) -> bool` - True when this operation is classified as schema DDL.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginMigrationOp) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginMigrationOp`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::plugin_migration::capnp_u32_len

*Function*

Converts a Rust list length into a Cap'n Proto `UInt32` list size.

# Errors

Returns `PluginError::payload_too_large` when `len` cannot fit in `u32`.

```rust
fn capnp_u32_len(len: usize, what: &str) -> crate::Result<u32>
```



## bookclerk_plugin_abi::plugin_migration::plugin_migration_registration_bytes

*Function*

Aggregate UTF-8 bytes of ids plus SQL across a complete registration.

```rust
fn plugin_migration_registration_bytes(migrations: &[PluginMigration]) -> usize
```



## bookclerk_plugin_abi::plugin_migration::require_plugin_migration_json_lists

*Function*

Rejects JSON `databaseMigrations` list sizes before deserializing operations.

Walks nested `operations` array lengths only; does not construct
[`PluginMigration`](plugin_migration.md#bookclerk_plugin_abiplugin_migrationpluginmigration) values. Call this before `serde_json::from_value`.

# Errors

Returns `PluginError::payload_too_large` when migration count, operations
per migration, or total operations exceed ABI limits.

```rust
fn require_plugin_migration_json_lists(value: &serde_json::Value) -> crate::Result<()>
```



## bookclerk_plugin_abi::plugin_migration::require_plugin_migration_registration

*Function*

Rejects a plugin migration registration that exceeds RPC resource limits.

Call this before encoding a Cap'n Proto / JSON reply and while decoding,
before semantic BookclerkSQL proof. Limits:

- migration count: [`MAX_LIST_PAGE`](limits.md#bookclerk_plugin_abilimitsmax_list_page)
- operations per migration: [`MAX_PLUGIN_MIGRATION_OPS`](limits.md#bookclerk_plugin_abilimitsmax_plugin_migration_ops)
- total operations: [`MAX_PLUGIN_MIGRATION_TOTAL_OPS`](limits.md#bookclerk_plugin_abilimitsmax_plugin_migration_total_ops)
- each SQL text: [`MAX_SCALAR_BYTES`](limits.md#bookclerk_plugin_abilimitsmax_scalar_bytes)
- aggregate id + SQL bytes: [`MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](limits.md#bookclerk_plugin_abilimitsmax_plugin_migration_registration_bytes)

Total operations are checked before walking SQL text so a registration of
many tiny operations is rejected without treating the 256 KiB text budget
as a heap bound.

# Errors

Returns `PluginError::payload_too_large` when any limit is exceeded.

```rust
fn require_plugin_migration_registration(migrations: &[PluginMigration]) -> crate::Result<()>
```



