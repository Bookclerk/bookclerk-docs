**bookclerk_plugin_database_sqlite > plugin**

# Module: plugin

## Contents

**Structs**

- [`SqliteRoot`](#sqliteroot) - Root capability for the platform SQLite database guest.

---

## bookclerk_plugin_database_sqlite::plugin::SqliteRoot

*Struct*

Root capability for the platform SQLite database guest.

**Unit Struct**

**Trait Implementations:**

- **PluginWorker**
  - `fn describe(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn open(self: &'life0 Self, _invocation: Invocation, bindings: Bindings) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



