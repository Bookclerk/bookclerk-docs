# bookclerk_plugin_abi::backup_ops

Host-facing backup/restore primitives (Send + Sync).

[`crate::AdapterDatabaseSession`](index.md#adapterdatabasesession) methods are `?Send` Cap'n stubs. Library
backup orchestration stores a [`AdapterBackupOps`](#adapterbackupops) handle so capture/restore
can call identity and catalog primitives without emitting dialect SQL.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AdapterBackupOps`](#adapterbackupops) | trait | Snapshot / identity / restore hooks implemented by first-party adapters. |
| [`SharedAdapterBackupOps`](#sharedadapterbackupops) | type alias | Shared handle stored on backup/restore options. |

## Traits

### AdapterBackupOps

```rust
pub trait AdapterBackupOps: Send + Sync {
    fn export_identity<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<DbIdentityHighWater>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn import_identity<'life0, 'life1, 'async_trait>(
        &'life0 self,
        rows: &'life1 [DbIdentityHighWater],
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn list_user_relations<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn prepare_unit_restore<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn drop_user_relations<'life0, 'life1, 'async_trait>(
        &'life0 self,
        names: &'life1 [String],
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn assert_restore_constraints<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Snapshot / identity / restore hooks implemented by first-party adapters.

#### AdapterBackupOps::export_identity

```rust
fn export_identity<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<DbIdentityHighWater>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Identity high-water from adapter catalogs (`sqlite_sequence`, `bookclerk_identity`).

#### AdapterBackupOps::import_identity

```rust
fn import_identity<'life0, 'life1, 'async_trait>(
    &'life0 self,
    rows: &'life1 [DbIdentityHighWater],
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Restore identity high-water into adapter catalogs.

#### AdapterBackupOps::list_user_relations

```rust
fn list_user_relations<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

User-visible relation names (excludes engine catalogs such as `sqlite_*`).

#### AdapterBackupOps::prepare_unit_restore

```rust
fn prepare_unit_restore<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Prepare an open restore transaction (SQLite deferred FK checks).

#### AdapterBackupOps::drop_user_relations

```rust
fn drop_user_relations<'life0, 'life1, 'async_trait>(
    &'life0 self,
    names: &'life1 [String],
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Drop named user relations (Postgres `CASCADE` + identity companions).

#### AdapterBackupOps::assert_restore_constraints

```rust
fn assert_restore_constraints<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Fail closed when the restore transaction still has FK violations.

## Type aliases

### SharedAdapterBackupOps

```rust
pub type SharedAdapterBackupOps = Arc<dyn AdapterBackupOps>;
```

Shared handle stored on backup/restore options.
