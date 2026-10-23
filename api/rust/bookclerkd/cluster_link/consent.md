**bookclerkd > cluster_link > consent**

# Module: cluster_link::consent

## Contents

**Structs**

- [`LinkSupervisor`](#linksupervisor) - Keeps one link's catalog and consented plugin set current.

**Functions**

- [`notify_consent_changed`](#notify_consent_changed) - Tells every link supervisor in this process to re-read consent now.
- [`platform_payloads`](#platform_payloads) - Payload digests of this host's platform-bundled plugins, by key.

**Constants**

- [`CONSENT_POLL`](#consent_poll) - How often consent rows are re-read without a notification.
- [`RETRY_MAX`](#retry_max) - Longest delay between retries of an unfinished refresh.
- [`RETRY_MIN`](#retry_min) - First delay before an unfinished refresh is retried.

---

## bookclerkd::cluster_link::consent::CONSENT_POLL

*Constant*: `std::time::Duration`

How often consent rows are re-read without a notification.



## bookclerkd::cluster_link::consent::LinkSupervisor

*Struct*

Keeps one link's catalog and consented plugin set current.

**Fields:**
- `library: bookclerk_library::LibraryStore` - This host's own database.
- `files_dir: std::path::PathBuf` - This host's files dir (the version cache lives under it).
- `handle: bookclerk_node_client::HostClientHandle` - The link's client.
- `catalog: std::sync::Arc<super::remote_bindings::RemoteCatalog>` - The link's catalog, shared with the job runner.
- `platform_payloads: std::collections::BTreeMap<String, String>` - Payload digests of this host's own platform-bundled plugins, by

**Methods:**

- `fn run(self: Self, shutdown: watch::Receiver<bool>)` - Runs until `shutdown` turns true.
- `fn cluster_id(self: &Self) -> &str` - The link's owner cluster id.
- `fn publish(self: &Self)` - Publishes the consented set to the client.
- `fn reevaluate(self: &Self) -> Option<Refreshed>` - Re-reads the consent rows and applies them; an instance that gains
- `fn refresh(self: &Self) -> Refreshed` - Re-reads what the owner deployed here and rebuilds the catalog.
- `fn reconcile(self: &Self, entries: & mut BTreeMap<String, RemoteInstanceEntry>) -> bool` - Brings each of `entries` up to the consent rows as they stand now
- `fn evaluate(self: &Self, item: &RemoteDeployment) -> (RemoteInstanceEntry, Refreshed)` - One deployment's catalog entry: its consent, and when consented its
- `fn evaluate_entry(self: &Self, item: &RemoteDeployment) -> (RemoteInstanceEntry, bool)` - `Self::evaluate` without the outcome; the flag marks a failure to
- `fn auto_approve_platform(self: &Self, row: RemotePluginConsent, version: &PluginVersion) -> RemotePluginConsent` - Approves a platform plugin that is unapproved, or whose automatic
- `fn materialize(self: &Self, version: &PluginVersion, grant: &PluginGrant, row: &RemotePluginConsent) -> Result<PathBuf, String>` - Materializes `version` into this host's version cache, but only the
- `fn instance_config(self: &Self, instance_id: &str, open_secrets: bool) -> Result<RemoteInstanceConfig, String>` - Reads an instance's `CONFIG` and `SECRETS` from the owner. The
- `fn observe(self: &Self, item: &RemoteDeployment, entry: &RemoteInstanceEntry) -> bool` - Reports one deployment's state to the owner; answers whether the



## bookclerkd::cluster_link::consent::RETRY_MAX

*Constant*: `std::time::Duration`

Longest delay between retries of an unfinished refresh.



## bookclerkd::cluster_link::consent::RETRY_MIN

*Constant*: `std::time::Duration`

First delay before an unfinished refresh is retried.



## bookclerkd::cluster_link::consent::notify_consent_changed

*Function*

Tells every link supervisor in this process to re-read consent now.

```rust
fn notify_consent_changed()
```



## bookclerkd::cluster_link::consent::platform_payloads

*Function*

Payload digests of this host's platform-bundled plugins, by key.

```rust
fn platform_payloads(config: &bookclerk_config::Config) -> std::collections::BTreeMap<String, String>
```



