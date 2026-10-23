**bookclerk_library > backup**

# Module: backup

## Contents

**Structs**

- [`BackupListEntry`](#backuplistentry) - Catalog entry used by `bookclerk db backup list`.
- [`BackupManifest`](#backupmanifest) - On-disk recovery-point metadata (`manifests/<id>.json`).
- [`BackupOutcome`](#backupoutcome) - Result of publishing a recovery point.
- [`BackupRequest`](#backuprequest) - Inputs for an automatic or CLI backup.
- [`BackupTable`](#backuptable) - One table's chunk list inside a recovery-point unit.
- [`BackupUnit`](#backupunit) - One logical database in a recovery point (library or plugin binding).
- [`CanonicalDatabaseSchema`](#canonicaldatabaseschema) - Admitted canonical schema used to export or restore one database unit.
- [`CanonicalExportOpts`](#canonicalexportopts) - Options for a consistent canonical export.
- [`CanonicalRestoreOpts`](#canonicalrestoreopts) - Negotiated limits for parameterized canonical restore.
- [`CanonicalTableSchema`](#canonicaltableschema) - One admitted `CREATE TABLE` plus its parsed IR.
- [`IdentityHighWater`](#identityhighwater) - Identity high-water for one generated column.
- [`PreparedPluginUnit`](#preparedpluginunit) - Open plugin binding captured through the active adapter session.
- [`RestorePlan`](#restoreplan) - Library restore result plus plugin units for the host to apply.
- [`SchemaBackupOpts`](#schemabackupopts) - Optional in-place backup taken before applying ups or downs.
- [`ValidatedBackup`](#validatedbackup) - Recovery point whose objects, schema, and typed rows have been verified.

**Enums**

- [`BackupReason`](#backupreason) - Why a backup was written.
- [`BackupResolve`](#backupresolve) - How [`resolve_backup_spec`](backup.md#bookclerk_librarybackupresolve_backup_spec) located a recovery point.
- [`CanonicalRestoreKind`](#canonicalrestorekind) - How restore applies DDL companions.
- [`DatabaseUnitKind`](#databaseunitkind) - Logical kind of one database unit in a recovery point.

**Functions**

- [`archive_backup`](#archive_backup) - Packages one recovery point (manifest + referenced objects) as `.tar.gz`.
- [`backup_library`](#backup_library) - Writes a crash-safe recovery point under `files_dir/backups/`.
- [`extract_backup_archive`](#extract_backup_archive) - Extracts a `.tar.gz` recovery-point archive into `dest` (a backups root).
- [`list_backups`](#list_backups) - Lists published recovery points oldest-first.
- [`prune_automatic_backups`](#prune_automatic_backups) - Deletes oldest automatic pre-migrate recovery points. Never deletes `manual`.
- [`resolve_backup_spec`](#resolve_backup_spec) - Resolves a recovery-point id, timestamp prefix, or archive path.
- [`restore_backup`](#restore_backup) - Restores the library unit after verifying **all** objects and units.
- [`restore_backup_in_repo`](#restore_backup_in_repo) - Restores from an already-open repository (extracted archives).

**Constants**

- [`BACKUPS_DIR`](#backups_dir) - Subdirectory of `$BOOKCLERK_FILES_DIR` holding the backup repository.
- [`BACKUP_FORMAT_VERSION`](#backup_format_version) - Manifest format written by this binary.
- [`BACKUP_RETENTION`](#backup_retention) - Automatic `pre-migrate` recovery points retained after prune.
- [`LIBRARY_SKIP_TABLES`](#library_skip_tables) - Library registry rows are environment-local (`unit_ref` is not portable).

---

## bookclerk_library::backup::BACKUPS_DIR

*Constant*: `&str`

Subdirectory of `$BOOKCLERK_FILES_DIR` holding the backup repository.



## bookclerk_library::backup::BACKUP_FORMAT_VERSION

*Constant*: `u32`

Manifest format written by this binary.



## bookclerk_library::backup::BACKUP_RETENTION

*Constant*: `usize`

Automatic `pre-migrate` recovery points retained after prune.



## bookclerk_library::backup::BackupListEntry

*Struct*

Catalog entry used by `bookclerk db backup list`.

**Fields:**
- `manifest: BackupManifest` - Parsed manifest.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BackupListEntry`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::backup::BackupManifest

*Struct*

On-disk recovery-point metadata (`manifests/<id>.json`).

**Fields:**
- `format_version: u32` - Catalog format (`1` for content-addressed canonical objects).
- `id: String` - Stable recovery-point id (UUID).
- `created_at: String` - RFC 3339 UTC timestamp.
- `schema_state: String` - `SchemaState::display` at capture.
- `schema_checksum: Option<String>` - SHA-256 recorded on the library schema state marker.
- `frozen_version: Option<i64>` - Frozen revision when the source library was frozen.
- `sql_contract_version: u32` - Bookclerk SQL contract of the library unit.
- `app_version: String` - Bookclerk version that wrote the backup.
- `reason: BackupReason` - Operator vs automatic.
- `migrate_to: i64` - Target frozen version a following migrate step was heading toward.
- `include_plugin_databases: bool` - When true, every registered plugin binding was required at capture.
- `units: Vec<BackupUnit>` - Logical database units (library first).

**Methods:**

- `fn referenced_objects(self: &Self) -> BTreeSet<String>` - Union of every object digest referenced by this recovery point.
- `fn library_unit(self: &Self) -> Result<&BackupUnit>` - Library unit from this manifest.
- `fn plugin_units(self: &Self) -> Vec<&BackupUnit>` - Plugin binding units.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> BackupManifest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::backup::BackupOutcome

*Struct*

Result of publishing a recovery point.

**Fields:**
- `dir: std::path::PathBuf` - Repository root (`files_dir/backups`).
- `manifest: BackupManifest` - Parsed manifest.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BackupOutcome`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::backup::BackupReason

*Enum*

Why a backup was written.

**Variants:**
- `Manual` - Operator-created (`bookclerk db backup create`). Never pruned automatically.
- `PreMigrate` - Written immediately before a frozen schema walk.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> BackupReason`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &BackupReason) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> BackupReason`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::backup::BackupRequest

*Struct*

Inputs for an automatic or CLI backup.

**Fields:**
- `files_dir: std::path::PathBuf` - `$BOOKCLERK_FILES_DIR`.
- `schema_state: crate::schema_state::SchemaState` - Durable library schema state at capture. `SchemaState::Uninitialized` skips.
- `reason: BackupReason` - Operator vs pre-migrate.
- `to_version: i64` - Version the upcoming migrate step will target (`0` for manual).
- `include_plugin_databases: bool` - Include every registered plugin-owned database binding.
- `consistent_backup_read: bool` - Guest advertised consistent backup read.
- `backend_at_capture: String` - Adapter id at capture (diagnostic; not used to branch restore).
- `max_result_rows: u32` - Negotiated `maxResultRows` (clamped).
- `max_result_bytes: u32` - Negotiated `maxResultBytes` for one SELECT page.
- `max_atomic_result_bytes: u32` - Negotiated `maxAtomicResultBytes` (capture page budget is the min of both).
- `plugin_units: Vec<PreparedPluginUnit>` - Plugin sessions prepared by the host (must match the registry when inclusion is on).
- `adapter: Option<bookclerk_plugin_abi::SharedAdapterBackupOps>` - Library adapter hooks. `None` uses the in-process SDK (tests).
- `in_process: bool` - True when leftover SQL is physically lowered on the opened connection.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> BackupRequest`



## bookclerk_library::backup::BackupResolve

*Enum*

How [`resolve_backup_spec`](backup.md#bookclerk_librarybackupresolve_backup_spec) located a recovery point.

**Variants:**
- `Id(String)` - Catalog id under `backups/manifests/`.
- `Archive(std::path::PathBuf)` - `.tar.gz` archive of one recovery point.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> BackupResolve`



## bookclerk_library::backup::BackupTable

*Struct*

One table's chunk list inside a recovery-point unit.

**Fields:**
- `name: String` - Folded table name.
- `columns: Vec<String>` - Column names in CREATE declaration order.
- `chunks: Vec<String>` - SHA-256 hex of each table-data chunk (canonical order).



## bookclerk_library::backup::BackupUnit

*Struct*

One logical database in a recovery point (library or plugin binding).

**Fields:**
- `kind: DatabaseUnitKind` - Library vs plugin binding.
- `plugin_id: Option<String>` - Plugin id when `DatabaseUnitKind::PluginBinding`.
- `binding: Option<String>` - Binding name when `DatabaseUnitKind::PluginBinding`.
- `backend_at_capture: String` - Adapter that captured this unit (diagnostic only; restore is cross-adapter).
- `sql_contract_version: u32` - Bookclerk SQL grammar/ABI contract, not a library schema revision.
- `schema_object: String` - SHA-256 of the schema object.
- `identity_object: String` - SHA-256 of the identity object.
- `tables: Vec<BackupTable>` - User-visible tables (not catalog/identity companions).
- `plugin_schema_namespace: Option<String>` - Plugin id when `DatabaseUnitKind::PluginBinding` (owner identity).
- `plugin_schema_state: Option<String>` - `crate::PluginMigrationHistory::display` at capture (`history@{n}+{digest}`).
- `plugin_schema_checksum: Option<String>` - Digest of the complete ordered `(migration_id, checksum)` journal.

**Methods:**

- `fn restore_kind(self: &Self) -> CanonicalRestoreKind` - Restore companion kind for this unit.
- `fn referenced_objects(self: &Self) -> BTreeSet<String>` - Object digests this unit needs to restore.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> BackupUnit`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &BackupUnit) -> bool`



## bookclerk_library::backup::CanonicalDatabaseSchema

*Struct*

Admitted canonical schema used to export or restore one database unit.

**Fields:**
- `sql_contract_version: u32` - Bookclerk SQL grammar/ABI contract, not a library schema revision.
- `tables: Vec<CanonicalTableSchema>` - `CREATE TABLE` statements in FK-safe order after [`crate::sort_tables_by_foreign_keys`](backup/schema.md#bookclerk_librarybackupschemasort_tables_by_foreign_keys).
- `indexes: Vec<bookclerk_plugin_abi::CreateIndexSchema>` - `CREATE INDEX` statements (after tables).

**Methods:**

- `fn schema_sql(self: &Self) -> Vec<String>` - Ordered canonical DDL (tables then indexes).
- `fn table_names(self: &Self) -> Vec<String>` - Folded user table names in CREATE order.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &CanonicalDatabaseSchema) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CanonicalDatabaseSchema`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::backup::CanonicalExportOpts

*Struct*

Options for a consistent canonical export.

**Fields:**
- `consistent_backup_read: bool` - Guest advertised `DbCapabilities::supports_consistent_backup_read`.
- `skip_tables: std::collections::BTreeSet<String>` - Folded table names omitted from row capture (library `plugin_databases`).
- `max_result_rows: u32` - Page size for `SELECT … LIMIT` (honors adapter `maxResultRows`).
- `max_result_bytes: u32` - Encoded-byte budget for one SELECT page (`maxResultBytes`).
- `max_atomic_result_bytes: u32` - Encoded-byte budget for one atomic reply (`maxAtomicResultBytes`).
- `chunk_target_bytes: usize` - Uncompressed JSON target for one table-data chunk.
- `adapter: Option<bookclerk_plugin_abi::SharedAdapterBackupOps>` - Adapter snapshot/identity hooks. `None` uses the in-process SDK (tests).
- `in_process: bool` - True when leftover SQL is physically lowered on the opened connection.

**Methods:**

- `fn from_caps(caps: &DbCapabilities) -> Self` - Page-size and result-byte limits from negotiated adapter capabilities.
- `fn page_byte_budget(self: &Self) -> usize` - Effective encoded-byte budget for one capture SELECT page.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> CanonicalExportOpts`
- **Default**
  - `fn default() -> Self`



## bookclerk_library::backup::CanonicalRestoreKind

*Enum*

How restore applies DDL companions.

**Variants:**
- `Library` - Host library: Postgres identity companions only (no plugin catalog).
- `PluginBinding` - Plugin binding: catalog + Postgres identity companions.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CanonicalRestoreKind) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CanonicalRestoreKind`



## bookclerk_library::backup::CanonicalRestoreOpts

*Struct*

Negotiated limits for parameterized canonical restore.

**Fields:**
- `atomic_unit_restore: bool` - Guest advertised `atomicUnitRestore`.
- `max_binds: u32` - Maximum bound parameters per INSERT (`maxBinds`).
- `max_payload_bytes: u32` - Maximum UTF-8 bytes of SQL plus binds per statement (`maxPayloadBytes`).
- `max_request_bytes: u32` - Maximum encoded bytes of one [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest).
- `adapter: Option<bookclerk_plugin_abi::SharedAdapterBackupOps>` - Adapter restore hooks. `None` uses the in-process SDK (tests).
- `in_process: bool` - True when leftover SQL is physically lowered on the opened connection.

**Methods:**

- `fn from_caps(caps: &DbCapabilities) -> Result<Self>` - Restore limits copied from negotiated adapter capabilities.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CanonicalRestoreOpts`
- **Default**
  - `fn default() -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_library::backup::CanonicalTableSchema

*Struct*

One admitted `CREATE TABLE` plus its parsed IR.

**Fields:**
- `create_sql: String` - Original canonical SQL (no trailing semicolon).
- `parsed: bookclerk_plugin_abi::CreateTableSchema` - Parsed columns, constraints, and identity.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CanonicalTableSchema) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CanonicalTableSchema`



## bookclerk_library::backup::DatabaseUnitKind

*Enum*

Logical kind of one database unit in a recovery point.

**Variants:**
- `Library` - Host library database.
- `PluginBinding` - Plugin-owned binding (`plugin_id` + `binding`).

**Traits:** Copy, Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DatabaseUnitKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &DatabaseUnitKind) -> bool`



## bookclerk_library::backup::IdentityHighWater

*Struct*

Identity high-water for one generated column.

**Fields:**
- `column: String` - Identity column name.
- `last: i64` - Highest generated or stored value that must not be reused.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &IdentityHighWater) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> IdentityHighWater`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::backup::LIBRARY_SKIP_TABLES

*Constant*: `&[&str]`

Library registry rows are environment-local (`unit_ref` is not portable).



## bookclerk_library::backup::PreparedPluginUnit

*Struct*

Open plugin binding captured through the active adapter session.

**Fields:**
- `plugin_id: String` - Owning plugin id.
- `binding: String` - Binding name.
- `backend_at_capture: String` - Adapter that produced this unit (diagnostic).
- `db: sea_orm::DatabaseConnection` - SeaORM connection opened via the active adapter (not a native dump).
- `max_result_rows: u32` - Negotiated `maxResultRows` for this binding session.
- `max_result_bytes: u32` - Negotiated `maxResultBytes` for this binding session.
- `max_atomic_result_bytes: u32` - Negotiated `maxAtomicResultBytes` for this binding session.
- `adapter: Option<bookclerk_plugin_abi::SharedAdapterBackupOps>` - Binding-scoped adapter hooks. `None` uses the in-process SDK (tests).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> PreparedPluginUnit`



## bookclerk_library::backup::RestorePlan

*Struct*

Library restore result plus plugin units for the host to apply.

Plugin databases are restored separately after this returns. A failure
while restoring a later unit leaves earlier units replaced (no bundle-level
atomicity). Each unit still uses complete replacement semantics.

**Fields:**
- `manifest: BackupManifest` - Verified manifest.
- `plugin_units: Vec<BackupUnit>` - Plugin units to restore onto the target adapter.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RestorePlan`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::backup::SchemaBackupOpts

*Struct*

Optional in-place backup taken before applying ups or downs.

**Fields:**
- `files_dir: std::path::PathBuf` - `$BOOKCLERK_FILES_DIR` root for `backups/`.
- `include_plugin_databases: bool` - Include every registered plugin-owned database binding.
- `consistent_backup_read: bool` - Guest advertised consistent backup read.
- `backend_at_capture: String` - Adapter id at capture (diagnostic).
- `max_result_rows: u32` - Negotiated page size.
- `max_result_bytes: u32` - Negotiated encoded SELECT-page budget.
- `max_atomic_result_bytes: u32` - Negotiated encoded atomic-reply budget.
- `plugin_units: Vec<PreparedPluginUnit>` - Plugin sessions prepared by the host when inclusion is requested.
- `adapter: Option<bookclerk_plugin_abi::SharedAdapterBackupOps>` - Library adapter hooks. `None` uses the in-process SDK (tests).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SchemaBackupOpts`
- **Default**
  - `fn default() -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_library::backup::ValidatedBackup

*Struct*

Recovery point whose objects, schema, and typed rows have been verified.

**Fields:**
- `manifest: BackupManifest` - Parsed, supported manifest.
- `library: BackupUnit` - Library unit metadata (objects already verified in the repository).
- `plugin_units: Vec<BackupUnit>` - Plugin binding units (already verified).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ValidatedBackup`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::backup::archive_backup

*Function*

Packages one recovery point (manifest + referenced objects) as `.tar.gz`.

# Errors

Returns when the destination cannot be written or objects are missing.

```rust
fn archive_backup(files_dir: &std::path::Path, id: &str, dest: &std::path::Path) -> crate::error::Result<()>
```



## bookclerk_library::backup::backup_library

*Function*

Writes a crash-safe recovery point under `files_dir/backups/`.

Skips only `SchemaState::Uninitialized`. The manifest is published only
after every referenced object exists. Pre-migrate backups prune older
automatic recovery points and garbage-collect unreferenced objects.
Manual backups are never auto-pruned.

# Errors

Returns when the files dir cannot be written, capture fails, or a requested
plugin binding is missing from `BackupRequest::plugin_units`.

```rust
fn backup_library(db: &sea_orm::DatabaseConnection, req: &BackupRequest) -> crate::error::Result<Option<BackupOutcome>>
```



## bookclerk_library::backup::extract_backup_archive

*Function*

Extracts a `.tar.gz` recovery-point archive into `dest` (a backups root).

Expansion is bounded by entry count, per-entry size, total expanded bytes,
and gzip-decoded tar stream size so a small archive cannot fill the disk.

# Errors

Returns when the archive is missing, malformed, a path escapes `dest`, or
an extract budget would be exceeded.

```rust
fn extract_backup_archive(archive: &std::path::Path, dest: &std::path::Path) -> crate::error::Result<()>
```



## bookclerk_library::backup::list_backups

*Function*

Lists published recovery points oldest-first.

# Errors

Returns when the repository cannot be opened.

```rust
fn list_backups(files_dir: &std::path::Path) -> crate::error::Result<Vec<BackupListEntry>>
```



## bookclerk_library::backup::prune_automatic_backups

*Function*

Deletes oldest automatic pre-migrate recovery points. Never deletes `manual`.

# Errors

Returns when a manifest cannot be unlinked.

```rust
fn prune_automatic_backups(files_dir: &std::path::Path) -> crate::error::Result<usize>
```



## bookclerk_library::backup::resolve_backup_spec

*Function*

Resolves a recovery-point id, timestamp prefix, or archive path.

# Errors

Returns when no complete backup matches or the spec is ambiguous.

```rust
fn resolve_backup_spec(files_dir: &std::path::Path, spec: &str) -> crate::error::Result<BackupResolve>
```



## bookclerk_library::backup::restore_backup

*Function*

Restores the library unit after verifying **all** objects and units.

Plugin units are returned for the host/CLI to restore onto the target
adapter. Restore replaces Bookclerk-visible schema for the library unit
and does not merge or auto-migrate. Plugin-owned migrations are not run.

The library `plugin_databases` registry is preserved when the recovery
point carries no plugin units (including `--include-plugin-databases` with
an empty registry at capture). When plugin units are present, library
restore clears the registry so leftover bindings cannot outlive the
captured set; the host/CLI rebinds while restoring those units.

# Errors

Returns when the backup is unsupported, an object is missing/corrupt, or
library replace fails. Destructive SQL starts only after complete preflight.

```rust
fn restore_backup(db: &sea_orm::DatabaseConnection, files_dir: &std::path::Path, id: &str, opts: &CanonicalRestoreOpts) -> crate::error::Result<RestorePlan>
```



## bookclerk_library::backup::restore_backup_in_repo

*Function*

Restores from an already-open repository (extracted archives).

Target [`crate::SchemaState`](schema_state.md#bookclerk_libraryschema_stateschemastate) is read from the canonical
`bookclerk_schema_migrations` rows. This function never infers a
versioning contract from [`sea_orm::DbBackend`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/type.DbBackend.html).

# Errors

Same as [`restore_backup`](backup.md#bookclerk_librarybackuprestore_backup).

```rust
fn restore_backup_in_repo(db: &sea_orm::DatabaseConnection, repo: &self::repository::BackupRepository, id: &str, opts: &CanonicalRestoreOpts) -> crate::error::Result<RestorePlan>
```



