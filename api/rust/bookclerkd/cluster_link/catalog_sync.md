**bookclerkd > cluster_link > catalog_sync**

# Module: cluster_link::catalog_sync

## Contents

**Structs**

- [`CatalogSync`](#catalogsync) - The catalog sync of one link.
- [`DaemonShelf`](#daemonshelf) - The daemon's [`BranchShelf`]: its deployed destinations.
- [`SyncReport`](#syncreport) - What one pass did.

**Functions**

- [`notify_sharing_changed`](#notify_sharing_changed) - Tells every running catalog sync to start a pass now.

**Traits**

- [`BranchShelf`](#branchshelf) - Where this install's own files are.

**Constants**

- [`SYNC_INTERVAL`](#sync_interval) - Between catalog passes while connected.

---

## bookclerkd::cluster_link::catalog_sync::BranchShelf

*Trait*

Where this install's own files are.

**Methods:**

- `primary`: This install's primary output destination, when one is deployed.



## bookclerkd::cluster_link::catalog_sync::CatalogSync

*Struct*

The catalog sync of one link.

**Generic Parameters:**
- S

**Fields:**
- `library: bookclerk_library::LibraryStore` - This install's library.
- `cluster_id: String` - The library (owner cluster) this install is a branch of.
- `handle: bookclerk_node_client::HostClientHandle` - The link's client.
- `shelf: S` - This install's files.

**Methods:**

- `fn run(self: Self, shutdown: watch::Receiver<bool>)` - Runs passes until `shutdown` turns true: on each welcomed
- `fn entries(self: &Self, include_branches: bool) -> anyhow::Result<Vec<Entry>>` - Every title this install may share now, in sync order, one per
- `fn title(self: &Self, shelf: &dyn StorageBackend, entry: &Entry) -> anyhow::Result<CatalogTitle>` - The [`CatalogTitle`](../../bookclerk_node_proto/catalog.md#bookclerk_node_protocatalogcatalogtitle) of `entry`, its files listed on `shelf`.
- `fn sync_once(self: &Self) -> anyhow::Result<SyncReport>` - One pass: open the session, send the titles after its cursor, and
- `fn deliver(self: &Self, shelf: &dyn StorageBackend, session: &CatalogSession, title: &CatalogTitle) -> anyhow::Result<()>` - Streams one title's files to the library: hashes each from the



## bookclerkd::cluster_link::catalog_sync::DaemonShelf

*Struct*

The daemon's [`BranchShelf`]: its deployed destinations.

**Fields:**
- `state: std::sync::Arc<crate::api::AppState>` - Daemon state.

**Trait Implementations:**

- **BranchShelf**
  - `fn primary(self: &Self) -> Option<Box<dyn StorageBackend>>`



## bookclerkd::cluster_link::catalog_sync::SYNC_INTERVAL

*Constant*: `std::time::Duration`

Between catalog passes while connected.



## bookclerkd::cluster_link::catalog_sync::SyncReport

*Struct*

What one pass did.

**Fields:**
- `shared: bool` - Sharing was on.
- `sent: u64` - Titles sent.
- `upserted: u64` - Titles the library recorded.
- `skipped: u64` - Titles the library already held.
- `delivered: u64` - Titles whose files were delivered.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> SyncReport`
- **PartialEq**
  - `fn eq(self: &Self, other: &SyncReport) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> SyncReport`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::catalog_sync::notify_sharing_changed

*Function*

Tells every running catalog sync to start a pass now.

```rust
fn notify_sharing_changed()
```



