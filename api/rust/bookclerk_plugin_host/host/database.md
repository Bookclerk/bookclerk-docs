**bookclerk_plugin_host > host > database**

# Module: host::database

## Contents

**Structs**

- [`DatabaseRegistry`](#databaseregistry) - Long-lived external database plugin for the active `[database].plugin`.
- [`ExternalDatabase`](#externaldatabase) - External database backend spawned for `[database].plugin`.

**Functions**

- [`backup_adapter_id`](#backup_adapter_id) - Adapter id recorded on a backup (diagnostic; restore is capability-driven).
- [`database_connect_bindings`](#database_connect_bindings) - Builds the [`bookclerk_plugin_sdk::BindingValues`](../../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolesbindingvalues) for the adapter's
- [`load_external_database`](#load_external_database) - Discover and spawn the external database plugin matching `[database].plugin`.
- [`migrate_database_plugin`](#migrate_database_plugin) - Copy library data from one database plugin backend to another.
- [`migrate_library_schema`](#migrate_library_schema) - Opens the active database guest without auto-apply and migrates to `target`.
- [`open_library_store`](#open_library_store) - Open [`bookclerk_library::LibraryStore`](../../bookclerk_library/store.md#bookclerk_librarystorelibrarystore) via the external database guest (required).
- [`open_library_store_for_plugin`](#open_library_store_for_plugin) - Open the library for a specific `[database].plugin` id (ignoring the active config value).

---

## bookclerk_plugin_host::host::database::DatabaseRegistry

*Struct*

Long-lived external database plugin for the active `[database].plugin`.

**Methods:**

- `fn active(self: &Self) -> Option<Arc<ExternalDatabase>>` - Currently active external database guest for `[database].plugin`.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DatabaseRegistry`
- **Default**
  - `fn default() -> DatabaseRegistry`



## bookclerk_plugin_host::host::database::ExternalDatabase

*Struct*

External database backend spawned for `[database].plugin`.

**Methods:**

- `fn spawn(plugin: &DiscoveredPlugin, config: &Config) -> PluginResult<Self>` - Spawn and describe a database plugin (connection happens later).
- `fn spawn_with(plugin: &DiscoveredPlugin, config: &Config, services: SessionServices) -> PluginResult<Self>` - `Self::spawn` with explicit session services (spawn transport, …).
- `fn connect(self: &Self, config: &Config) -> Result<(DatabaseConnection, DbCapabilities), DbErr>` - Open the library connection through the guest (`database.openSession`).
- `fn connect_without_migrate(self: &Self, config: &Config) -> Result<(DatabaseConnection, DbCapabilities), DbErr>` - Open the library connection without applying host schema (CLI migrate / version).
- `fn library_backup_ops(self: &Self) -> bookclerk_plugin_abi::SharedAdapterBackupOps` - Adapter snapshot/identity/restore hooks for the library session.
- `fn binding_backup_ops<impl Into<String>>(self: &Self, key: impl Trait) -> bookclerk_plugin_abi::SharedAdapterBackupOps` - Adapter snapshot/identity/restore hooks for one named binding session.
- `fn migrate_to(self: &Self, db: &DatabaseConnection, caps: &DbCapabilities, target: i64, opts: bookclerk_library::SchemaApplyOptions) -> Result<bookclerk_library::SchemaWalk, DbErr>` - Migrates toward `target` using guest `executeAtomic` batches (CLI migrate / downgrade).
- `fn open_binding_databases(self: &Self, config: &Config, store: &bookclerk_library::LibraryStore, owner_plugin_id: &str, bindings: &[String], owner: &PluginSession) -> PluginResult<Vec<(String, crate::rpc_session::GuestDatabaseFactory)>>` - Opens (provisioning on first use) isolated sessions for the named
- `fn open_binding_seaorm(self: &Self, config: &Config, owner_plugin_id: &str, binding: &str, unit_ref: &str, provision: bool) -> PluginResult<(DatabaseConnection, DbCapabilities)>` - Opens one plugin binding as a SeaORM connection through the guest session.
- `fn drop_provisioned_unit(config: &Config, adapter_plugin_key: &str, unit_ref: &str) -> PluginResult<()>` - Physically deletes a provisioned binding unit. The registry row is the
- `fn adapter_plugin_key(self: &Self) -> &str` - Canonical PluginKey of this adapter guest.
- `fn first_party_kind(self: &Self) -> Option<DatabasePluginKind>` - Verified first-party adapter kind, if any.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ExternalDatabase`



## bookclerk_plugin_host::host::database::backup_adapter_id

*Function*

Adapter id recorded on a backup (diagnostic; restore is capability-driven).

```rust
fn backup_adapter_id(plugin_id: &str) -> String
```



## bookclerk_plugin_host::host::database::database_connect_bindings

*Function*

Builds the [`bookclerk_plugin_sdk::BindingValues`](../../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolesbindingvalues) for the adapter's
`PluginWorker.open` (then `databaseAdapter.openSession`).

Used by the CLI diagnose probe and mirrors `ExternalDatabase::connect`.

# Errors

Returns an error when plugin data paths, secrets, or context encoding fail.

```rust
fn database_connect_bindings(config: &bookclerk_config::Config, plugin: &crate::discover::DiscoveredPlugin, session: &crate::rpc_session::PluginSession) -> crate::Result<bookclerk_plugin_sdk::BindingValues>
```



## bookclerk_plugin_host::host::database::load_external_database

*Function*

Discover and spawn the external database plugin matching `[database].plugin`.

Guests are required: when the matching database plugin is missing or fails to
start, [`open_library_store`](database.md#bookclerk_plugin_hosthostdatabaseopen_library_store) returns an error (no in-process engine).

# Errors

Returns an error when the operation fails.

```rust
fn load_external_database(config: &bookclerk_config::Config) -> crate::Result<DatabaseRegistry>
```



## bookclerk_plugin_host::host::database::migrate_database_plugin

*Function*

Copy library data from one database plugin backend to another.

# Errors

Returns an error when the operation fails.

```rust
fn migrate_database_plugin(config: &bookclerk_config::Config, from_plugin: &str, to_plugin: &str, opts: &bookclerk_library::BackendMigrateOptions) -> bookclerk_library::Result<bookclerk_library::BackendMigrateSummary>
```



## bookclerk_plugin_host::host::database::migrate_library_schema

*Function*

Opens the active database guest without auto-apply and migrates to `target`.

Uses guest `executeAtomic` batches (the same path as daemon connect).

# Errors

Returns when the guest cannot start, connect, or apply schema.

```rust
fn migrate_library_schema(config: &bookclerk_config::Config, target: i64, opts: bookclerk_library::SchemaApplyOptions) -> crate::Result<bookclerk_library::SchemaWalk>
```



## bookclerk_plugin_host::host::database::open_library_store

*Function*

Open [`bookclerk_library::LibraryStore`](../../bookclerk_library/store.md#bookclerk_librarystorelibrarystore) via the external database guest (required).

# Errors

Returns an error when the operation fails.

```rust
fn open_library_store(config: &bookclerk_config::Config, registry: &DatabaseRegistry) -> bookclerk_library::Result<bookclerk_library::LibraryStore>
```



## bookclerk_plugin_host::host::database::open_library_store_for_plugin

*Function*

Open the library for a specific `[database].plugin` id (ignoring the active config value).

# Errors

Returns an error when the operation fails.

```rust
fn open_library_store_for_plugin(config: &bookclerk_config::Config, plugin_id: &str) -> bookclerk_library::Result<bookclerk_library::LibraryStore>
```



