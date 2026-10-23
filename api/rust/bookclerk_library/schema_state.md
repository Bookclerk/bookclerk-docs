**bookclerk_library > schema_state**

# Module: schema_state

## Contents

**Enums**

- [`SchemaState`](#schemastate) - Durable Bookclerk library schema state.

**Constants**

- [`SCHEMA_STATE_FROZEN`](#schema_state_frozen) - On-disk `bookclerk_schema_migrations.state` for a frozen plan version.
- [`SCHEMA_STATE_UNRELEASED`](#schema_state_unreleased) - On-disk `bookclerk_schema_migrations.state` for an unreleased development pack.

---

## bookclerk_library::schema_state::SCHEMA_STATE_FROZEN

*Constant*: `&str`

On-disk `bookclerk_schema_migrations.state` for a frozen plan version.



## bookclerk_library::schema_state::SCHEMA_STATE_UNRELEASED

*Constant*: `&str`

On-disk `bookclerk_schema_migrations.state` for an unreleased development pack.



## bookclerk_library::schema_state::SchemaState

*Enum*

Durable Bookclerk library schema state.

**Variants:**
- `Uninitialized` - No Bookclerk bookkeeping and no host tables.
- `Unreleased{ base_version: i64, checksum: String }` - Development pack applied on top of a frozen base (or pre-v1).
- `Frozen{ version: i64, checksum: String }` - A frozen [`crate::migrations::HostMigrationStep`](migrations.md#bookclerk_librarymigrationshostmigrationstep) is applied.

**Methods:**

- `fn display(self: &Self) -> String` - CLI / JSON display: `uninitialized`, `unreleased@base<n>+<checksum>`,
- `fn frozen_version(self: &Self) -> Option<i64>` - Frozen revision when this state is `Self::Frozen`; otherwise `None`.
- `fn unreleased_base_version(self: &Self) -> Option<i64>` - Frozen base recorded on an unreleased database (`0` = pre-v1).
- `fn checksum(self: &Self) -> Option<&str>` - Checksum when the database has applied Bookclerk schema.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SchemaState) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> SchemaState`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`



