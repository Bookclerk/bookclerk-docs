**bookclerk_plugin_host > host > destination**

# Module: host::destination

## Contents

**Structs**

- [`DestinationRegistry`](#destinationregistry) - Long-lived external output plugins loaded at host startup.
- [`DestinationSettings`](#destinationsettings) - Key layout settings the host reads from a deployed destination's instance

---

## bookclerk_plugin_host::host::destination::DestinationRegistry

*Struct*

Long-lived external output plugins loaded at host startup.

**Methods:**

- `fn s3(self: &Self) -> Option<Arc<dyn StorageBackend>>` - External S3 output backend, when exactly one is loaded.
- `fn require_s3(self: &Self) -> std::result::Result<Option<Arc<dyn StorageBackend>>, String>` - The S3 backend, or an error when more than one deployed instance owns it.
- `fn local(self: &Self) -> Option<Arc<dyn StorageBackend>>` - External local-filesystem output backend, when exactly one is loaded.
- `fn require_local(self: &Self) -> std::result::Result<Option<Arc<dyn StorageBackend>>, String>` - The local backend, or an error when more than one deployed instance owns it.
- `fn settings_for(self: &Self, kind: OutputBackendKind) -> Option<DestinationSettings>` - Key layout settings of the deployed destination of `kind`, when exactly
- `fn plugin_session(self: &Self, plugin_id: &str, account_id: &str) -> Option<Arc<PluginSession>>` - Plugin session for `plugin_id` and `account_id`, when that guest was loaded.
- `fn require_plugin_session(self: &Self, plugin_id: &str, account_id: &str) -> std::result::Result<Arc<PluginSession>, String>` - `Self::plugin_session` that reports same-key ambiguity.
- `fn reattach_owned(self: &Self, candidate: & mut Self, deployed_instances: &std::collections::BTreeSet<String>)` - Copies deployed sessions from this registry onto `candidate`.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DestinationRegistry`
- **Default**
  - `fn default() -> DestinationRegistry`



## bookclerk_plugin_host::host::destination::DestinationSettings

*Struct*

Key layout settings the host reads from a deployed destination's instance
document: the `local` / `s3` plugins declare `prefix` and the naming
overrides as `[[settings]]`, and acquire needs them to plan storage keys.

**Fields:**
- `prefix: String` - Key prefix under the destination root or bucket.
- `naming: bookclerk_config::DestinationNaming` - Naming overrides; unset values inherit global `[output]`.



