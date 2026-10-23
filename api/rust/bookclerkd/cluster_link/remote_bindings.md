**bookclerkd > cluster_link > remote_bindings**

# Module: cluster_link::remote_bindings

## Contents

**Structs**

- [`HostJobBindings`](#hostjobbindings) - What one handed job's guest is opened with.
- [`RemoteCatalog`](#remotecatalog) - The instances one owner cluster deployed to this host, by instance id.
- [`RemoteInstanceConfig`](#remoteinstanceconfig) - An instance's `CONFIG` and `SECRETS` as the owner served them.
- [`RemoteInstanceEntry`](#remoteinstanceentry) - One instance an owner deployed to this host.

**Functions**

- [`host_job_bindings`](#host_job_bindings) - Builds the bindings a handed job's guest is opened with, failing closed

**Constants**

- [`INSTANCE_UNAVAILABLE`](#instance_unavailable) - Error kind for a handed job whose instance this host cannot open.

---

## bookclerkd::cluster_link::remote_bindings::HostJobBindings

*Struct*

What one handed job's guest is opened with.

**Fields:**
- `instance_id: String` - Owner plugin instance id.
- `plugin_key: String` - The instance's plugin key.
- `grant: bookclerk_plugin_host::PluginGrant` - The consented grant (the owner's grant narrowed by this host); the
- `root: std::path::PathBuf` - Materialized plugin tree.
- `version: bookclerk_library::control_plane::PluginVersion` - The version materialized at `root`.
- `open: bookclerk_plugin_host::PreparedOpen` - `CONFIG` and `SECRETS` for `PluginWorker.open`.
- `credentials: Vec<bookclerk_plugin_abi::AccountCredential>` - Store credentials for the job's accounts, from the owner's handed

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerkd::cluster_link::remote_bindings::INSTANCE_UNAVAILABLE

*Constant*: `&str`

Error kind for a handed job whose instance this host cannot open.



## bookclerkd::cluster_link::remote_bindings::RemoteCatalog

*Struct*

The instances one owner cluster deployed to this host, by instance id.

**Fields:**
- `entries: std::sync::RwLock<std::collections::BTreeMap<String, RemoteInstanceEntry>>` - Entries by instance id.

**Methods:**

- `fn read(self: &Self) -> std::sync::RwLockReadGuard<BTreeMap<String, RemoteInstanceEntry>>` - Reads the entries (recovering from a poisoned lock: each write
- `fn write(self: &Self) -> std::sync::RwLockWriteGuard<BTreeMap<String, RemoteInstanceEntry>>` - Writes the entries.
- `fn replace(self: &Self, entries: BTreeMap<String, RemoteInstanceEntry>)` - Replaces every entry.
- `fn entries(self: &Self) -> Vec<RemoteInstanceEntry>` - Every entry.
- `fn get(self: &Self, instance_id: &str) -> Option<RemoteInstanceEntry>` - One entry.
- `fn set_consent(self: &Self, plugin_key: &str, consent: &RemoteConsentState) -> bool` - Sets the consent of every entry of `plugin_key`; an entry that loses
- `fn consented_keys(self: &Self) -> BTreeSet<String>` - Plugin keys whose every deployed instance here can run now: the

**Trait Implementations:**

- **Default**
  - `fn default() -> RemoteCatalog`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::remote_bindings::RemoteInstanceConfig

*Struct*

An instance's `CONFIG` and `SECRETS` as the owner served them.

**Fields:**
- `revision: i64` - Instance document revision.
- `config: serde_json::Value` - `CONFIG` (JSON object).
- `secrets: Option<std::sync::Arc<bookclerk_node_proto::SecretValue>>` - `SECRETS` (JSON object bytes), opened with this node's box key;
- `secrets_withheld: bool` - True when the owner withheld secrets this node is not trusted with.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> RemoteInstanceConfig`



## bookclerkd::cluster_link::remote_bindings::RemoteInstanceEntry

*Struct*

One instance an owner deployed to this host.

**Fields:**
- `instance_id: String` - Owner plugin instance id.
- `plugin_key: String` - The instance's plugin key.
- `default_instance: bool` - True for the plugin's default instance (unpinned jobs use it).
- `consent: bookclerk_plugin_host::RemoteConsentState` - This host's consent to run it.
- `root: Option<std::path::PathBuf>` - Materialized plugin tree, once consented and fetched.
- `version: Option<bookclerk_library::control_plane::PluginVersion>` - The owner's version as materialized here, its grant the consented
- `config: Option<RemoteInstanceConfig>` - The instance's bindings, once consented and fetched.
- `problem: Option<String>` - Why it cannot run, when it cannot (observation detail).

**Methods:**

- `fn is_ready(self: &Self) -> bool` - True when a job may run on this entry: consented, materialized and

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RemoteInstanceEntry`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::remote_bindings::host_job_bindings

*Function*

Builds the bindings a handed job's guest is opened with, failing closed
when its instance is not deployed here, not consented, or not fetched.

# Errors

Returns [`AWAITING_HOST_CONSENT`](../../bookclerk_node_client/client.md#bookclerk_node_clientclientawaiting_host_consent) when the instance is not consented and
[`INSTANCE_UNAVAILABLE`](remote_bindings.md#bookclerkdcluster_linkremote_bindingsinstance_unavailable) when it is unknown, unfetched, or ambiguous.

```rust
fn host_job_bindings(catalog: &RemoteCatalog, ctx: &bookclerk_node_client::HostJobCtx) -> Result<HostJobBindings, bookclerk_node_client::HostJobError>
```



