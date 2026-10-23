**bookclerk_node_client > client**

# Module: client

## Contents

**Structs**

- [`CatalogBatchAccepted`](#catalogbatchaccepted) - A recorded catalog batch (`HostClientHandle::catalog_sync`).
- [`CatalogSession`](#catalogsession) - An opened catalog session (`HostClientHandle::catalog_begin`).
- [`ClientTiming`](#clienttiming) - Timers of the client. The defaults follow the protocol constants.
- [`HostClient`](#hostclient) - The host side of the node API for one cluster link.
- [`HostClientConfig`](#hostclientconfig) - What one client connects with.
- [`HostClientHandle`](#hostclienthandle) - Watches a running client.

**Enums**

- [`ClientExit`](#clientexit) - Why `HostClient::run` returned.

**Traits**

- [`HeartbeatSource`](#heartbeatsource) - Measurements for `heartbeat` frames.

**Constants**

- [`AWAITING_HOST_CONSENT`](#awaiting_host_consent) - Release reason for a job whose plugins this host's operator has not
- [`KIND_NOT_RUN_HERE`](#kind_not_run_here) - Release reason for a job of a kind this host does not run.
- [`MAX_ARTIFACT_BYTES`](#max_artifact_bytes) - Largest artifact fetched from the owner (the library blob cap).
- [`MAX_LEASE`](#max_lease) - Longest lease this host counts on from one `work` or `renewed`; a
- [`SECRETS_UNOPENABLE`](#secrets_unopenable) - Error kind reported when a job's secrets do not open with this node's

---

## bookclerk_node_client::client::AWAITING_HOST_CONSENT

*Constant*: `&str`

Release reason for a job whose plugins this host's operator has not
(or no longer) approved.



## bookclerk_node_client::client::CatalogBatchAccepted

*Struct*

A recorded catalog batch (`HostClientHandle::catalog_sync`).

**Fields:**
- `upserted: u64` - Titles recorded.
- `skipped: u64` - Titles the library already holds (not echoed back).
- `wanted: Vec<bookclerk_node_proto::CatalogRef>` - Titles whose files the library wants.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CatalogBatchAccepted`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogBatchAccepted) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_client::client::CatalogSession

*Struct*

An opened catalog session (`HostClientHandle::catalog_begin`).

**Fields:**
- `fence: i64` - The catalog fence every later call names.
- `cursor: String` - Resume after this cursor (empty: from the start).
- `ancestry: Vec<String>` - The library's ancestry, its own cluster id first.
- `destinations: Vec<String>` - The library node's destination ids, the primary first.
- `wanted: Vec<bookclerk_node_proto::CatalogRef>` - Titles whose files the library still wants.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSession`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogSession) -> bool`



## bookclerk_node_client::client::ClientExit

*Enum*

Why `HostClient::run` returned.

**Variants:**
- `Shutdown` - The shutdown signal fired.
- `Revoked` - The owner closed with [`CLOSE_REVOKED`]: this node's credential is

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ClientExit`
- **PartialEq**
  - `fn eq(self: &Self, other: &ClientExit) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_client::client::ClientTiming

*Struct*

Timers of the client. The defaults follow the protocol constants.

**Fields:**
- `heartbeat_interval: std::time::Duration` - Between heartbeats.
- `ping_interval: std::time::Duration` - Between pings.
- `idle_timeout: std::time::Duration` - Without any inbound frame before the client closes.
- `backoff_min: std::time::Duration` - First reconnect delay.
- `backoff_max: std::time::Duration` - Largest reconnect delay.
- `request_timeout: std::time::Duration` - How long a request waits for room in the send queue and then for

**Traits:** Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> ClientTiming`
- **Default**
  - `fn default() -> Self`



## bookclerk_node_client::client::HeartbeatSource

*Trait*

Measurements for `heartbeat` frames.

**Methods:**

- `sample`: Samples this host now. May block briefly (it runs on a blocking



## bookclerk_node_client::client::HostClient

*Struct*

The host side of the node API for one cluster link.

**Generic Parameters:**
- R

**Methods:**

- `fn new(config: HostClientConfig, runner: R, sampler: Box<dyn HeartbeatSource>) -> Result<(Self, HostClientHandle), DialError>` - A client for `config`, running jobs through `runner`.
- `fn run(self: Self, shutdown: watch::Receiver<bool>) -> ClientExit` - Connects, serves, and reconnects until `shutdown` turns true (or its



## bookclerk_node_client::client::HostClientConfig

*Struct*

What one client connects with.

**Fields:**
- `link: bookclerk_library::control_plane::ClusterLink` - The active cluster link.
- `keys: bookclerk_library::control_plane::ClusterLinkKeys` - Its node keys.
- `software_version: String` - This host's Bookclerk version.
- `incarnation: String` - This process's incarnation (a plain id).
- `max_jobs: usize` - Jobs run at once. Zero claims nothing.
- `claim_kinds: Vec<String>` - Job kinds this host runs (`scan`, ...). Empty claims nothing.
- `claims_after_refresh: bool` - When set, a connection claims nothing until
- `timing: ClientTiming` - Timers.



## bookclerk_node_client::client::HostClientHandle

*Struct*

Watches a running client.

**Methods:**

- `fn subscribe_wakes(self: &Self) -> broadcast::Receiver<serde_json::Value>` - Wakes the owner forwarded (`{"type": "deployment.changed", ...}`).
- `fn connections(self: &Self) -> watch::Receiver<u64>` - Count of welcomed connections so far.
- `fn deployments_refreshed(self: &Self, connection: u64)` - Records that the deployments were re-read on the connection counted
- `fn is_connected(self: &Self) -> bool` - True while a welcomed connection is live.
- `fn held_fences(self: &Self) -> Vec<HeldFence>` - Fences this host holds and has not lost.
- `fn node_id(self: &Self) -> &str` - The node id this host has in the owner cluster.
- `fn cluster_id(self: &Self) -> &str` - The owner cluster id.
- `fn set_consented_plugins(self: &Self, plugins: BTreeSet<String>)` - Sets the plugin keys this host's operator consents to. Claims carry
- `fn consented_plugins(self: &Self) -> BTreeSet<String>` - The plugin keys consented to now.
- `fn request(self: &Self, body: Body) -> Option<Frame>` - Sends `body` on the live connection (waiting up to the request
- `fn open_instance_secrets(self: &Self, instance_id: &str, revision: i64, sealed: &str) -> Result<SecretValue, SealError>` - Opens an instance's `SECRETS` sealed to this node's box key
- `fn fetch_artifact(self: &Self, archive_sha256: &str) -> Result<Vec<u8>, DialError>` - Downloads one archive the owner stores (`blob:sha256:<hex>`) from its
- `fn catalog_begin(self: &Self, share_books: bool, include_branches: bool) -> Result<CatalogSession, OwnerCallError>` - `catalog.begin`: opens this branch's catalog sync with the link's
- `fn catalog_sync(self: &Self, fence: i64, batch: u64, cursor: &str, done: bool, titles: Vec<CatalogTitle>) -> Result<CatalogBatchAccepted, OwnerCallError>` - `catalog.sync`: one batch of shared titles under `fence`, naming
- `fn catalog_upload(self: &Self, fence: i64, title: &CatalogRef, files: Vec<UploadFile>) -> Result<HostUpload, OwnerCallError>` - `catalog.upload`: opens an upload of one shared title's `files` and
- `fn upload_catalog_file<R>(self: &Self, upload: &HostUpload, index: usize, length: u64, body: R) -> Result<(), OwnerCallError>` - Streams declared file `index` of a catalog `upload` (`length` bytes
- `fn commit_catalog_upload(self: &Self, upload: &HostUpload) -> Result<(), OwnerCallError>` - Commits a catalog `upload`.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> HostClientHandle`



## bookclerk_node_client::client::KIND_NOT_RUN_HERE

*Constant*: `&str`

Release reason for a job of a kind this host does not run.



## bookclerk_node_client::client::MAX_ARTIFACT_BYTES

*Constant*: `usize`

Largest artifact fetched from the owner (the library blob cap).



## bookclerk_node_client::client::MAX_LEASE

*Constant*: `std::time::Duration`

Longest lease this host counts on from one `work` or `renewed`; a
longer grant is renewed as if it were this long.



## bookclerk_node_client::client::SECRETS_UNOPENABLE

*Constant*: `&str`

Error kind reported when a job's secrets do not open with this node's
box key.



