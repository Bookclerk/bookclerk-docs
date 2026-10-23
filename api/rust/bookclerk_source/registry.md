**bookclerk_source > registry**

# Module: registry

## Contents

**Structs**

- [`SourceRegistry`](#sourceregistry) - Maps a registry address to an installed [`ContentSource`](traits.md#bookclerk_sourcetraitscontentsource).

---

## bookclerk_source::registry::SourceRegistry

*Struct*

Maps a registry address to an installed [`ContentSource`](traits.md#bookclerk_sourcetraitscontentsource).

A deployed source is addressed by `ContentSource::plugin_instance_id`.
A source with no instance id is addressed by `ContentSource::plugin_key`.
A plugin key or display alias that several instances share routes to the
plugin's default instance (see `Self::set_default_instance`); a job may
pin another instance through `Self::route`. Nothing fails closed on a
shared key: accounts belong to the plugin, so any instance may serve them.

**Methods:**

- `fn new() -> Self` - Empty registry with no sources registered.
- `fn register(self: & mut Self, source: Arc<dyn ContentSource>)` - Register (or replace) a source at its instance id, or its PluginKey.
- `fn remove_instance(self: & mut Self, plugin_instance_id: &str) -> bool` - Drops the source registered under `plugin_instance_id`.
- `fn set_default_instance(self: & mut Self, plugin_key: &str, plugin_instance_id: &str)` - Names the instance a bare plugin key or alias routes to when several
- `fn default_instance(self: &Self, plugin_key: &str) -> Option<&str>` - Default instance id recorded for `plugin_key`, if any.
- `fn get(self: &Self, id_or_alias: &str) -> Option<Arc<dyn ContentSource>>` - Look up a source by plugin instance id, PluginKey, or display alias.
- `fn route(self: &Self, id_or_alias: &str, preferred_instance: Option<&str>) -> Option<Arc<dyn ContentSource>>` - `Self::get` with an optional pinned instance.
- `fn require(self: &Self, id_or_alias: &str) -> Result<Arc<dyn ContentSource>>` - Look up a source or return `crate::SourceError::Api` when missing.
- `fn require_routed(self: &Self, id_or_alias: &str, preferred_instance: Option<&str>) -> Result<Arc<dyn ContentSource>>` - `Self::require` with an optional pinned instance.
- `fn resolve_id(self: &Self, id_or_alias: &str) -> Option<String>` - Resolve a needle to the canonical plugin id when registered.
- `fn all(self: &Self) -> Vec<Arc<dyn ContentSource>>` - All registered sources in stable plugin order.
- `fn routed(self: &Self) -> Vec<Arc<dyn ContentSource>>` - One routed source per registered plugin key, in stable plugin order.
- `fn scan_all(self: &Self, library: &LibraryStore, opts: ScanOptions) -> Result<ScanSummary>` - Scan every registered plugin once (honoring per-source account filters).
- `fn scan_one(self: &Self, library: &LibraryStore, source: Arc<dyn ContentSource>, opts: ScanOptions) -> Result<ScanSummary>` - Scan one registered source (a job pinned to a plugin instance).

**Trait Implementations:**

- **Default**
  - `fn default() -> SourceRegistry`
- **Clone**
  - `fn clone(self: &Self) -> SourceRegistry`



