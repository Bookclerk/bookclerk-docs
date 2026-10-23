**bookclerk_library > control_plane > schemas**

# Module: control_plane::schemas

## Contents

**Structs**

- [`PinnedPluginSchema`](#pinnedpluginschema) - One `plugin_schemas` row.
- [`SchemaPin`](#schemapin) - Inputs for [`pin_plugin_schema`](schemas.md#bookclerk_librarycontrol_planeschemaspin_plugin_schema).
- [`SchemaPinOutcome`](#schemapinoutcome) - Outcome of [`pin_plugin_schema`]: the row that is now pinned and how it

**Enums**

- [`SchemaPinChange`](#schemapinchange) - How [`pin_plugin_schema`](schemas.md#bookclerk_librarycontrol_planeschemaspin_plugin_schema) changed the row.

**Functions**

- [`list_plugin_schemas`](#list_plugin_schemas) - Every pinned schema, ordered by plugin key.
- [`load_plugin_schema`](#load_plugin_schema) - The pinned schema for `plugin_key`, if any node has pinned one.
- [`pin_plugin_schema`](#pin_plugin_schema) - Pins `pin` according to the version rule in the module docs.
- [`pin_version_schema`](#pin_version_schema) - Pins the schema of the plugin's active release.
- [`require_plugin_schema`](#require_plugin_schema) - Loads the pinned schema or fails closed with an operator-facing message.
- [`schema_fingerprint`](#schema_fingerprint) - Hex SHA-256 of the schema's canonical JSON. Two manifests that declare
- [`stale_against_pin`](#stale_against_pin) - Whether the pinned row for `plugin_key` is newer than `local_version`.

---

## bookclerk_library::control_plane::schemas::PinnedPluginSchema

*Struct*

One `plugin_schemas` row.

**Fields:**
- `plugin_key: String` - Canonical plugin key.
- `manifest_id: String` - Manifest id (display alias) at pin time.
- `schema_sha256: String` - Fingerprint of the normalized schema ([`schema_fingerprint`](schemas.md#bookclerk_librarycontrol_planeschemasschema_fingerprint)).
- `package_version: String` - Semver package version that produced the pinned schema.
- `schema: bookclerk_plugin_abi::SettingsSchema` - Declared settings.
- `recorded_at: String` - RFC 3339 time of the last pin.
- `recorded_by_node: String` - Node that recorded the last pin.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PinnedPluginSchema) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PinnedPluginSchema`



## bookclerk_library::control_plane::schemas::SchemaPin

*Struct*

Inputs for [`pin_plugin_schema`](schemas.md#bookclerk_librarycontrol_planeschemaspin_plugin_schema).

**Generic Parameters:**
- 'a

**Fields:**
- `plugin_key: &'a str` - Canonical plugin key.
- `manifest_id: &'a str` - Manifest id (display alias).
- `package_version: &'a str` - Semver package version of the local manifest.
- `schema: &'a bookclerk_plugin_abi::SettingsSchema` - Declared settings.
- `node_id: &'a str` - Node recording the pin.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> SchemaPin<'a>`



## bookclerk_library::control_plane::schemas::SchemaPinChange

*Enum*

How [`pin_plugin_schema`](schemas.md#bookclerk_librarycontrol_planeschemaspin_plugin_schema) changed the row.

**Variants:**
- `Unchanged` - The row already matched the local manifest and version.
- `Pinned` - No node had pinned this plugin before.
- `Upgraded{ previous_version: String }` - The local package is newer than the pin; the row now carries it.
- `Repinned` - Same version, different manifest (an unreleased edit); re-pinned in
- `Stale{ local_version: String }` - The local package is older than the pin. The row was not touched and
- `RolledBack{ previous_version: String }` - The active release moved to an older version (a rollback); the row

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SchemaPinChange`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SchemaPinChange) -> bool`



## bookclerk_library::control_plane::schemas::SchemaPinOutcome

*Struct*

Outcome of [`pin_plugin_schema`]: the row that is now pinned and how it
relates to the local manifest.

**Fields:**
- `schema: PinnedPluginSchema` - The pinned row. For `SchemaPinChange::Stale` this is the newer pin
- `change: SchemaPinChange` - What happened to the row.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SchemaPinOutcome`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SchemaPinOutcome) -> bool`



## bookclerk_library::control_plane::schemas::list_plugin_schemas

*Function*

Every pinned schema, ordered by plugin key.

# Errors

Returns an error when the read fails or a row cannot be decoded.

```rust
fn list_plugin_schemas(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<PinnedPluginSchema>>
```



## bookclerk_library::control_plane::schemas::load_plugin_schema

*Function*

The pinned schema for `plugin_key`, if any node has pinned one.

# Errors

Returns an error when the read fails or the stored JSON is not a schema
this binary understands.

```rust
fn load_plugin_schema(store: &crate::store::LibraryStore, plugin_key: &str) -> crate::error::Result<Option<PinnedPluginSchema>>
```



## bookclerk_library::control_plane::schemas::pin_plugin_schema

*Function*

Pins `pin` according to the version rule in the module docs.

An unchanged row is left alone, including `recorded_at`. The insert and
the update are both conditional on the row read just before, and a lost
race re-reads and decides again, so a concurrent newer pin is never
overwritten by an older one.

# Errors

Returns an error when `package_version` is not semver, the schema fails
to serialize, the write fails, or the row keeps changing underneath.

```rust
fn pin_plugin_schema(store: &crate::store::LibraryStore, pin: &SchemaPin) -> crate::error::Result<SchemaPinOutcome>
```



## bookclerk_library::control_plane::schemas::pin_version_schema

*Function*

Pins the schema of the plugin's active release.

The pinned schema of a released plugin is the active release's schema,
so an older version is pinned too (`SchemaPinChange::RolledBack`)
instead of being refused as stale.

# Errors

As [`pin_plugin_schema`](schemas.md#bookclerk_librarycontrol_planeschemaspin_plugin_schema).

```rust
fn pin_version_schema(store: &crate::store::LibraryStore, pin: &SchemaPin) -> crate::error::Result<SchemaPinOutcome>
```



## bookclerk_library::control_plane::schemas::require_plugin_schema

*Function*

Loads the pinned schema or fails closed with an operator-facing message.

# Errors

Returns `LibraryError::NotFound` when no node has pinned the plugin yet.

```rust
fn require_plugin_schema(store: &crate::store::LibraryStore, plugin_key: &str) -> crate::error::Result<PinnedPluginSchema>
```



## bookclerk_library::control_plane::schemas::schema_fingerprint

*Function*

Hex SHA-256 of the schema's canonical JSON. Two manifests that declare
the same settings fingerprint the same, whatever their bytes.

# Errors

Returns an error when the schema fails to serialize.

```rust
fn schema_fingerprint(schema: &bookclerk_plugin_abi::SettingsSchema) -> crate::error::Result<String>
```



## bookclerk_library::control_plane::schemas::stale_against_pin

*Function*

Whether the pinned row for `plugin_key` is newer than `local_version`.

`None` when nothing is pinned yet. Used by the reconciler before a spawn
so a node that was not the one to upgrade the pin still refuses to run an
older package against documents written for the newer schema.

# Errors

Returns an error when the read fails or `local_version` is not semver.

```rust
fn stale_against_pin(store: &crate::store::LibraryStore, plugin_key: &str, local_version: &str) -> crate::error::Result<Option<PinnedPluginSchema>>
```



