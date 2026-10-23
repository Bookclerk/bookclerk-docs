**bookclerk_node_proto > frames**

# Module: frames

## Contents

**Structs**

- [`AcquireTarget`](#acquiretarget) - One title an acquire job should fetch on a host
- [`ClaimFilter`](#claimfilter) - What a host is willing to claim (`claim.filter`).
- [`FileCopy`](#filecopy) - Where one uploaded file is written: a destination and its key there.
- [`Frame`](#frame) - One frame: the envelope and its body.
- [`HeldFence`](#heldfence) - A lease the host still holds when it reconnects (`hello.resume`).
- [`JobStatusEntry`](#jobstatusentry) - One job's state in `Body::JobStatuses`.
- [`PlanDestination`](#plandestination) - One output destination of the owner node that answered an acquire plan
- [`RemoteArtifact`](#remoteartifact) - One downloadable artifact of a [`RemoteVersion`](frames.md#bookclerk_node_protoframesremoteversion).
- [`RemoteDeployment`](#remotedeployment) - One deployment that targets a host (`deployments.items`).
- [`RemoteVersion`](#remoteversion) - The active version of a plugin a host is asked to run.
- [`ScanImport`](#scanimport) - The owner's library import settings a host scan runs with.
- [`UploadAnswer`](#uploadanswer) - Answer of the upload endpoint (every status carries one).
- [`UploadCommitRequest`](#uploadcommitrequest) - Body of `POST <UPLOADS_PATH><upload_id>/commit`.
- [`UploadFile`](#uploadfile) - One file of an upload (`Body::UploadGrant`). Its content crosses the
- [`WorkJob`](#workjob) - A job handed to a host.

**Enums**

- [`Body`](#body) - What a frame says, tagged by `type`.
- [`FrameError`](#frameerror) - Why a frame was not accepted.

**Functions**

- [`is_destination_id`](#is_destination_id) - True when `id` is a destination id: a short lowercase token.
- [`is_storage_key`](#is_storage_key) - True when `key` is a relative storage key: no empty, `.` or `..`

**Constants**

- [`ITEM_JOB`](#item_job) - Work item kind for jobs.
- [`MAX_DESTINATION_ID_BYTES`](#max_destination_id_bytes) - Longest destination id.
- [`MAX_SCAN_BATCH_ROWS`](#max_scan_batch_rows) - Most rows one `Body::ScanResults` batch may carry.
- [`MAX_SCAN_ROW_BYTES`](#max_scan_row_bytes) - Largest serialized row in a `Body::ScanResults` batch.
- [`MAX_STATUS_JOBS`](#max_status_jobs) - Most job ids one `Body::JobStatus` may ask about.
- [`MAX_UPLOAD_COPIES`](#max_upload_copies) - Most destinations one uploaded file may be written to.
- [`MAX_UPLOAD_FILES`](#max_upload_files) - Most files one upload may declare.
- [`MAX_UPLOAD_FILE_BYTES`](#max_upload_file_bytes) - Largest file one upload may declare (8 GiB).
- [`MAX_UPLOAD_KEY_BYTES`](#max_upload_key_bytes) - Longest storage key an upload may declare.

---

## bookclerk_node_proto::frames::AcquireTarget

*Struct*

One title an acquire job should fetch on a host
(`Body::AcquireTargets`).

**Fields:**
- `book_uuid: String` - Library id of the title.
- `plugin_key: String` - Plugin key of the store that sells it.
- `account_id: String` - Store account that owns it.
- `book: serde_json::Value` - The owner's library row (JSON), for naming and metadata.
- `parent: Option<serde_json::Value>` - The owner's row for the title's podcast parent (JSON), when the

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AcquireTarget`
- **PartialEq**
  - `fn eq(self: &Self, other: &AcquireTarget) -> bool`



## bookclerk_node_proto::frames::Body

*Enum*

What a frame says, tagged by `type`.

**Variants:**
- `Hello{ node_id: String, incarnation: String, software_version: String, resume: Vec<HeldFence> }` - Host → owner: first frame.
- `Challenge{ nonce: String, cluster_id: String }` - Owner → host: the nonce to sign.
- `Auth{ sig: String }` - Host → owner: proof of key possession.
- `Welcome{ node_id: String, cluster_id: String, box_key: String }` - Owner → host: the handshake succeeded.
- `Resumed{ renewed: Vec<HeldFence>, lost: Vec<HeldFence>, unchecked: Vec<HeldFence>, lease_secs: Option<u64> }` - Owner → host, right after `Body::Welcome` when the hello listed
- `Error{ code: String, message: String }` - Either side: a request was not served. The connection stays open.
- `Heartbeat{ started_at: String, interval_secs: u64, cpu_millicores: i64, memory_bytes: Option<i64>, cpu_used_millicores: Option<i64>, memory_available_bytes: Option<i64>, files_dir_free_bytes: Option<i64> }` - Host → owner: the host's own measurements, recorded in
- `Claim{ item: String, class: Option<String>, filter: Option<ClaimFilter> }` - Host → owner: pull one work item. Answered with `Body::Work` or
- `Work{ item: String, job: WorkJob, fence: i64, lease_secs: u64, sealed_secrets: Option<String>, plugin_keys: Vec<String>, source_plugin_key: Option<String>, resumed: bool }` - Owner → host: a claimed job, its lease, and its sealed secrets.
- `NoWork{ retry_after_secs: u64 }` - Owner → host: nothing to claim now.
- `Renew{ item: String, job_id: String, fence: i64, progress: Option<String> }` - Host → owner: extend a lease. Answered with `Body::Renewed` or
- `Renewed{ lease_secs: u64, cancel: bool }` - Owner → host: the lease was extended.
- `Lost{ item: String, job_id: String, fence: i64 }` - Owner → host: the fence no longer owns the item; abort it and drop
- `Complete{ job_id: String, fence: i64, detail: Option<String> }` - Host → owner: the job finished. Answered with `Body::Accepted` or
- `Fail{ job_id: String, fence: i64, error_kind: String, message: String }` - Host → owner: the job failed. Answered with `Body::Accepted` or
- `Release{ job_id: String, fence: i64, reason: String, resumed: bool }` - Host → owner: give a job back unrun (its operator no longer
- `DeploymentsList{  }` - Host → owner: the deployments that target this host and should run.
- `Deployments{ items: Vec<RemoteDeployment> }` - Owner → host: the answer to `Body::DeploymentsList`.
- `InstanceConfigGet{ instance_id: String }` - Host → owner: `CONFIG` and `SECRETS` of an instance deployed to this
- `InstanceConfig{ instance_id: String, revision: i64, config: serde_json::Value, sealed_secrets: Option<String>, secrets_withheld: bool }` - Owner → host: the answer to `Body::InstanceConfigGet`.
- `Observation{ deployment_id: String, status: String, detail: String, applied_config_revision: Option<i64> }` - Host → owner: the host's observation of one of its deployments
- `SecretUpdate{ job_id: String, fence: i64, secret_ref: crate::sealed::SecretRef, sealed_value: String }` - Host → owner: write back a refreshed store token. Accepted only for a
- `ScanResults{ job_id: String, fence: i64, plugin_key: String, batch: u64, books: Vec<serde_json::Value> }` - Host → owner: one batch of a scan's book rows (decision 6). The
- `ScanAccepted{ batch: u64, upserted: u64 }` - Owner → host: a results batch was written.
- `AcquirePlan{ job_id: String, fence: i64 }` - Host → owner: what an acquire job should fetch here. Answered with
- `AcquireTargets{ targets: Vec<AcquireTarget>, destinations: Vec<PlanDestination> }` - Owner → host: the titles to acquire and how to package them.
- `UploadGrant{ job_id: String, fence: i64, book_uuid: String, primary_key: String, pdf_key: Option<String>, files: Vec<UploadFile> }` - Host → owner: ask to upload one title's files. Answered with
- `UploadGranted{ upload_id: String, sealed_token: String, expires_in_secs: u64 }` - Owner → host: the upload is open.
- `ScanRequest{ account: Option<String>, plugin_instance_id: Option<String> }` - Host → owner: queue a library scan on the owner cluster for the
- `ScanPlanned{ created: Vec<String>, duplicates: Vec<String>, queue_full: u64 }` - Owner → host: what a `Body::ScanRequest` queued.
- `JobStatus{ job_ids: Vec<String> }` - Host → owner: the state of jobs this node may see (H-5). A job the
- `JobStatuses{ jobs: Vec<JobStatusEntry> }` - Owner → host: the jobs of a `Body::JobStatus`.
- `Wake{ message: serde_json::Value }` - Owner → host: a message from the host's `node.<id>` queue. The host
- `Ack{  }` - Host → owner: the wake with this frame id was handled.
- `NodeLeaving{  }` - Host → owner: this host is leaving the cluster (its operator chose
- `CatalogBegin{ share_books: bool, include_branches: bool }` - Branch → library: open (or reopen) this branch's catalog sync with
- `CatalogSession{ fence: i64, cursor: String, ancestry: Vec<String>, destinations: Vec<String>, wanted: Vec<crate::catalog::CatalogRef> }` - Library → branch: the answer to `Body::CatalogBegin`.
- `CatalogSync{ fence: i64, batch: u64, cursor: String, done: bool, titles: Vec<crate::catalog::CatalogTitle> }` - Branch → library: one batch of shared titles (no account or user
- `CatalogAccepted{ batch: u64, upserted: u64, skipped: u64, wanted: Vec<crate::catalog::CatalogRef> }` - Library → branch: a catalog batch was recorded.
- `CatalogUpload{ fence: i64, store: String, product_id: String, files: Vec<UploadFile> }` - Branch → library: open an upload of one recorded title's files.
- `Accepted{  }` - Owner → host: the request with this frame id was applied.
- `Unknown` - A frame type this side does not know.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Body`
- **PartialEq**
  - `fn eq(self: &Self, other: &Body) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::frames::ClaimFilter

*Struct*

What a host is willing to claim (`claim.filter`).

**Fields:**
- `kinds: Vec<String>` - Job kinds the host runs.
- `plugins: Vec<String>` - Plugin keys the host's operator consented to.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ClaimFilter) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ClaimFilter`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> ClaimFilter`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::frames::FileCopy

*Struct*

Where one uploaded file is written: a destination and its key there.

**Fields:**
- `destination: String` - A `PlanDestination::id`.
- `key: String` - Storage key on that destination.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &FileCopy) -> $crate::option::Option<$crate::cmp::Ordering>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Ord**
  - `fn cmp(self: &Self, other: &FileCopy) -> $crate::cmp::Ordering`
- **Clone**
  - `fn clone(self: &Self) -> FileCopy`
- **PartialEq**
  - `fn eq(self: &Self, other: &FileCopy) -> bool`



## bookclerk_node_proto::frames::Frame

*Struct*

One frame: the envelope and its body.

**Fields:**
- `v: u32` - Envelope version; always [`FRAME_VERSION`](bookclerk_node_proto.md#bookclerk_node_protoframe_version).
- `id: String` - Request id, echoed by the answer.
- `body: Body` - The body, tagged by `type`.

**Methods:**

- `fn new<impl Into<String>>(id: impl Trait, body: Body) -> Self` - A version-1 frame.
- `fn parse(text: &str) -> Result<Self, FrameError>` - Parses a text frame, checking size and version.
- `fn to_text(self: &Self) -> String` - Serializes the frame as JSON text.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Frame`
- **PartialEq**
  - `fn eq(self: &Self, other: &Frame) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::frames::FrameError

*Enum*

Why a frame was not accepted.

**Variants:**
- `TooLarge` - More than [`MAX_FRAME_BYTES`](bookclerk_node_proto.md#bookclerk_node_protomax_frame_bytes).
- `Malformed` - Not a JSON frame of the expected shape.
- `Version(u32)` - `v` is not [`FRAME_VERSION`](bookclerk_node_proto.md#bookclerk_node_protoframe_version).

**Traits:** Error, Eq

**Trait Implementations:**

- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &FrameError) -> bool`



## bookclerk_node_proto::frames::HeldFence

*Struct*

A lease the host still holds when it reconnects (`hello.resume`).

**Fields:**
- `kind: String` - `job` or `delivery`.
- `id: String` - Job id or delivery id.
- `fence: i64` - Lease generation the host holds.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &HeldFence) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> HeldFence`



## bookclerk_node_proto::frames::ITEM_JOB

*Constant*: `&str`

Work item kind for jobs.



## bookclerk_node_proto::frames::JobStatusEntry

*Struct*

One job's state in `Body::JobStatuses`.

**Fields:**
- `id: String` - Job id.
- `kind: String` - Job kind (`scan`, `acquire`, ...).
- `state: String` - `pending`, `running`, `succeeded`, `failed`, or `cancelled`.
- `progress: Option<String>` - Progress or result text.
- `error_kind: Option<String>` - Structured error kind when it failed or was cancelled.
- `error_message: Option<String>` - Error text when it failed or was cancelled.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> JobStatusEntry`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobStatusEntry) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::frames::MAX_DESTINATION_ID_BYTES

*Constant*: `usize`

Longest destination id.



## bookclerk_node_proto::frames::MAX_SCAN_BATCH_ROWS

*Constant*: `usize`

Most rows one `Body::ScanResults` batch may carry.



## bookclerk_node_proto::frames::MAX_SCAN_ROW_BYTES

*Constant*: `usize`

Largest serialized row in a `Body::ScanResults` batch.



## bookclerk_node_proto::frames::MAX_STATUS_JOBS

*Constant*: `usize`

Most job ids one `Body::JobStatus` may ask about.



## bookclerk_node_proto::frames::MAX_UPLOAD_COPIES

*Constant*: `usize`

Most destinations one uploaded file may be written to.



## bookclerk_node_proto::frames::MAX_UPLOAD_FILES

*Constant*: `usize`

Most files one upload may declare.



## bookclerk_node_proto::frames::MAX_UPLOAD_FILE_BYTES

*Constant*: `u64`

Largest file one upload may declare (8 GiB).



## bookclerk_node_proto::frames::MAX_UPLOAD_KEY_BYTES

*Constant*: `usize`

Longest storage key an upload may declare.



## bookclerk_node_proto::frames::PlanDestination

*Struct*

One output destination of the owner node that answered an acquire plan
(`Body::AcquireTargets`).

**Fields:**
- `id: String` - Destination id on that owner node (its output kind, such as `local`
- `options: serde_json::Value` - That destination's `DownloadOptions` (JSON): its naming templates,

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PlanDestination`
- **PartialEq**
  - `fn eq(self: &Self, other: &PlanDestination) -> bool`



## bookclerk_node_proto::frames::RemoteArtifact

*Struct*

One downloadable artifact of a [`RemoteVersion`](frames.md#bookclerk_node_protoframesremoteversion).

**Fields:**
- `target: String` - Bookclerk target (`linux-x64-gnu`, `any`, ...).
- `url: String` - `https:` URL, or `blob:sha256:<hex>` for an archive the owner stores
- `format: String` - Archive format.
- `archive_root: String` - Path inside the archive to the plugin root.
- `archive_sha256: String` - SHA-256 of the archive.
- `payload_root_sha256: String` - Payload-root digest of the extracted tree.
- `executable: String` - Executable inside the plugin root.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteArtifact) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> RemoteArtifact`



## bookclerk_node_proto::frames::RemoteDeployment

*Struct*

One deployment that targets a host (`deployments.items`).

**Fields:**
- `deployment_id: String` - Deployment id.
- `instance_id: String` - Plugin instance the deployment runs.
- `plugin_key: String` - The instance's plugin key.
- `desired: String` - Desired state (`present`, `placed`).
- `revision: i64` - Desired-row revision.
- `default_instance: bool` - True for the plugin's default instance (unpinned jobs use it).
- `version: Option<RemoteVersion>` - The plugin's active version; absent when none is installed.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> RemoteDeployment`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteDeployment) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::frames::RemoteVersion

*Struct*

The active version of a plugin a host is asked to run.

**Fields:**
- `version: String` - Semver version.
- `manifest_id: String` - Manifest id (display alias).
- `manifest_toml: String` - `plugin.toml` text.
- `settings: serde_json::Value` - Settings schema the version declares (JSON).
- `artifacts: Vec<RemoteArtifact>` - Per-target artifacts.
- `grant_json: String` - The grant the cluster's operator approved (JSON); empty while the
- `publisher_verified: bool` - True when the publisher was verified at install.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> RemoteVersion`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteVersion) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_node_proto::frames::ScanImport

*Struct*

The owner's library import settings a host scan runs with.

**Fields:**
- `import_episodes: bool` - Import podcast episodes.
- `import_plus_titles: bool` - Import subscription (Plus) titles.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScanImport) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ScanImport`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_node_proto::frames::UploadAnswer

*Struct*

Answer of the upload endpoint (every status carries one).

**Fields:**
- `status: String` - Stable outcome: `received`, `committed`, `lost`, `refused`,
- `reason: Option<String>` - Why, for `refused`.
- `job_completed: bool` - For `committed`: the job is finished.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> UploadAnswer`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &UploadAnswer) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> UploadAnswer`



## bookclerk_node_proto::frames::UploadCommitRequest

*Struct*

Body of `POST <UPLOADS_PATH><upload_id>/commit`.

**Fields:**
- `finish: bool` - Complete the job with this commit (the job's closing upload).
- `detail: Option<String>` - Outcome text recorded as the job's progress when `finish` is set.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UploadCommitRequest`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> UploadCommitRequest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &UploadCommitRequest) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_node_proto::frames::UploadFile

*Struct*

One file of an upload (`Body::UploadGrant`). Its content crosses the
network once and is written to every copy, each destination under the
key that destination's own naming options give it.

**Fields:**
- `copies: Vec<FileCopy>` - Where the file is written; at most one copy per destination.
- `size: u64` - Exact size in bytes.
- `sha256: String` - Lowercase hex SHA-256 of the content.
- `content_type: Option<String>` - MIME type to store with it.

**Methods:**

- `fn is_well_formed(self: &Self) -> bool` - True when the file has between one and [`MAX_UPLOAD_COPIES`](frames.md#bookclerk_node_protoframesmax_upload_copies)
- `fn key_on(self: &Self, destination: &str) -> Option<&str>` - This file's key on `destination`, if it is written there.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> UploadFile`
- **PartialEq**
  - `fn eq(self: &Self, other: &UploadFile) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::frames::WorkJob

*Struct*

A job handed to a host.

**Fields:**
- `id: String` - Job id.
- `kind: String` - Job kind (`scan`, `acquire`, ...).
- `payload: serde_json::Value` - Kind-specific payload, as the owner stores it.
- `attempt: i64` - Claims of this job so far, this one included.
- `scan_import: Option<ScanImport>` - For a scan: which titles the owner's library imports. The owner

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &WorkJob) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> WorkJob`



## bookclerk_node_proto::frames::is_destination_id

*Function*

True when `id` is a destination id: a short lowercase token.

```rust
fn is_destination_id(id: &str) -> bool
```



## bookclerk_node_proto::frames::is_storage_key

*Function*

True when `key` is a relative storage key: no empty, `.` or `..`
segment, no backslash or control character, not absolute, at most
[`MAX_UPLOAD_KEY_BYTES`](frames.md#bookclerk_node_protoframesmax_upload_key_bytes), and never a Windows path that leaves the
destination's root (`colon_is_safe`).

```rust
fn is_storage_key(key: &str) -> bool
```



