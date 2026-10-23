**bookclerk_plugin_host > host > plugin_backups**

# Module: host::plugin_backups

## Contents

**Functions**

- [`export_registered_plugin_units`](#export_registered_plugin_units) - Exports every registered plugin-owned database binding as SeaORM sessions.
- [`restore_plugin_backup_units`](#restore_plugin_backup_units) - Restores plugin binding units onto the **current** adapter.

---

## bookclerk_plugin_host::host::plugin_backups::export_registered_plugin_units

*Function*

Exports every registered plugin-owned database binding as SeaORM sessions.

A missing `plugin_databases` table (uninitialized library) is treated as an
empty registry. Fails closed if any registered binding cannot be opened.

# Errors

Returns when a binding is inaccessible.

```rust
fn export_registered_plugin_units(ext: &super::database::ExternalDatabase, config: &bookclerk_config::Config, library: &sea_orm::DatabaseConnection) -> crate::Result<Vec<bookclerk_library::PreparedPluginUnit>>
```



## bookclerk_plugin_host::host::plugin_backups::restore_plugin_backup_units

*Function*

Restores plugin binding units onto the **current** adapter.

Each unit is replaced independently. A later failure leaves earlier units
already replaced (no cross-database atomicity). Plugin migrations are not
run; version rows restore as captured data. Registry rows are rebound to
the target adapter's physical placement.

Each `open_binding_seaorm` session must itself advertise `atomicUnitRestore`.
Restore binds, payload, and request limits come from that session.

# Errors

Returns when provisioning or canonical restore fails. Partial completion is
reported in the error message.

```rust
fn restore_plugin_backup_units(ext: &super::database::ExternalDatabase, config: &bookclerk_config::Config, library: &sea_orm::DatabaseConnection, repo: &bookclerk_library::BackupRepository, units: &[bookclerk_library::BackupUnit]) -> crate::Result<()>
```



