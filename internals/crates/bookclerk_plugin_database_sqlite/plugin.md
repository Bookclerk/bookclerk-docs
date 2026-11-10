# bookclerk_plugin_database_sqlite::plugin

Database factory for the platform SQLite guest.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SqliteRoot`](#sqliteroot) | struct | Root capability for the platform SQLite database guest. |
| [`binding_database`](#binding_database) | function | The adapter a plugin database binding gets, on its own connection. |

## Structs

### SqliteRoot

```rust
pub struct SqliteRoot;
```

Root capability for the platform SQLite database guest.

Implements: `PluginWorker`

## Functions

### binding_database

```rust
pub fn binding_database(conn: DatabaseConnection) -> Box<dyn Database>
```

The adapter a plugin database binding gets, on its own connection.

This is the `databaseAdapter` entrypoint the guest serves for a binding
open, without the RPC transport, so a test can run the adapter
conformance suite (`bookclerk-db-testkit`) on it in process.
