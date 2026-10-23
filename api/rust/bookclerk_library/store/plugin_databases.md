**bookclerk_library > store > plugin_databases**

# Module: store::plugin_databases

## Contents

**Structs**

- [`PluginDatabaseRecord`](#plugindatabaserecord) - One provisioned plugin database binding registry row.

---

## bookclerk_library::store::plugin_databases::PluginDatabaseRecord

*Struct*

One provisioned plugin database binding registry row.

**Fields:**
- `plugin_id: String` - Owning plugin PluginKey (canonical text).
- `binding: String` - Binding name from `plugin.toml` `capabilities.bindings.databases`.
- `adapter_plugin_key: String` - Adapter PluginKey that provisioned the physical unit.
- `backend_kind: String` - Diagnostic adapter family (`sqlite`, `postgres`, `d1`, or custom).
- `unit_ref: String` - Backend-native unit: file path, schema name, or D1 database name.
- `created_at: String` - RFC 3339 provisioning time.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginDatabaseRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginDatabaseRecord) -> bool`



