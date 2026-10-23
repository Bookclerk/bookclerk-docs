**bookclerk_plugin_abi > backup_ops**

# Module: backup_ops

## Contents

**Traits**

- [`AdapterBackupOps`](#adapterbackupops) - Snapshot / identity / restore hooks implemented by first-party adapters.

**Type Aliases**

- [`SharedAdapterBackupOps`](#sharedadapterbackupops) - Shared handle stored on backup/restore options.

---

## bookclerk_plugin_abi::backup_ops::AdapterBackupOps

*Trait*

Snapshot / identity / restore hooks implemented by first-party adapters.

**Methods:**

- `export_identity`: Identity high-water from adapter catalogs (`sqlite_sequence`, `bookclerk_identity`).
- `import_identity`: Restore identity high-water into adapter catalogs.
- `list_user_relations`: User-visible relation names (excludes engine catalogs such as `sqlite_*`).
- `prepare_unit_restore`: Prepare an open restore transaction (SQLite deferred FK checks).
- `drop_user_relations`: Drop named user relations (Postgres `CASCADE` + identity companions).
- `assert_restore_constraints`: Fail closed when the restore transaction still has FK violations.



## bookclerk_plugin_abi::backup_ops::SharedAdapterBackupOps

*Type Alias*: `std::sync::Arc<dyn AdapterBackupOps>`

Shared handle stored on backup/restore options.



