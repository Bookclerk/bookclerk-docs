**bookclerk_node_client > jobs**

# Module: jobs

## Contents

**Structs**

- [`HandedSecrets`](#handedsecrets) - The job's secrets, opened with this node's box key. Values are
- [`HostJob`](#hostjob) - A job the owner handed this host, under its fence.
- [`HostJobCtx`](#hostjobctx) - Everything a runner gets for one job.
- [`HostJobError`](#hostjoberror) - How a job failed: a stable kind and an operator-facing message. Neither
- [`HostUpload`](#hostupload) - An open upload of one title: its id and bearer token. The token is

**Enums**

- [`OwnerCallError`](#ownercallerror) - Why a call the job makes to the owner (results, plan, upload) failed.
- [`WriteBackError`](#writebackerror) - Why a token write-back was not stored.

**Traits**

- [`HostJobRunner`](#hostjobrunner) - Runs claimed jobs on this host.

---

## bookclerk_node_client::jobs::HandedSecrets

*Struct*

The job's secrets, opened with this node's box key. Values are
zeroized on drop and `Debug` shows only which rows they are.

**Tuple Struct**: `()`

**Methods:**

- `fn refs(self: &Self) -> impl Trait` - The rows handed with the job.
- `fn get(self: &Self, secret_ref: &SecretRef) -> Option<&SecretValue>` - The value of one handed row.
- `fn len(self: &Self) -> usize` - Number of handed rows.
- `fn is_empty(self: &Self) -> bool` - True when the job needs no secrets.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_node_client::jobs::HostJob

*Struct*

A job the owner handed this host, under its fence.

**Fields:**
- `node_id: String` - This host's node id in the owner cluster (the lease owner is
- `job: bookclerk_node_proto::WorkJob` - The job as the owner stores it.
- `fence: i64` - Lease generation; every frame for the job names it.
- `lease_secs: u64` - Lease length the owner granted.
- `plugin_keys: Vec<String>` - Plugin keys the job runs (each consented when it was claimed).
- `source_plugin_key: Option<String>` - The one plugin of `plugin_keys` whose source guest runs the job,
- `lease_deadline: std::time::SystemTime` - When the granted lease runs out, counted as the client counts it:
- `resumed: bool` - The claim resumed a suspended job: no attempt was spent, and the

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> HostJob`



## bookclerk_node_client::jobs::HostJobCtx

*Struct*

Everything a runner gets for one job.

**Fields:**
- `job: HostJob` - The job and its fence.
- `secrets: std::sync::Arc<HandedSecrets>` - Its opened secrets.

**Methods:**

- `fn send_scan_results(self: &Self, plugin_key: &str, books: Vec<serde_json::Value>) -> Result<u64, OwnerCallError>` - Sends one batch of a scan's rows (the guest's `ScanBook` objects) to
- `fn acquire_plan(self: &Self) -> Result<(Vec<AcquireTarget>, Vec<PlanDestination>), OwnerCallError>` - Asks the owner which titles this acquire should fetch, and with
- `fn grant_upload(self: &Self, book_uuid: &str, primary_key: &str, pdf_key: Option<&str>, files: Vec<UploadFile>) -> Result<HostUpload, OwnerCallError>` - Opens an upload of one title's `files` (an empty `book_uuid` and no
- `fn upload_file<R>(self: &Self, upload: &HostUpload, index: usize, length: u64, body: R) -> Result<(), OwnerCallError>` - Streams declared file `index` of `upload` (`length` bytes of
- `fn commit_upload(self: &Self, upload: &HostUpload, finish: Option<&str>) -> Result<bool, OwnerCallError>` - Commits `upload`. With `finish`, the owner also completes the job
- `fn is_cancelled(self: &Self) -> bool` - True when the runner should stop: the fence was lost or the
- `fn is_lost(self: &Self) -> bool` - True once the job's fence is lost: the owner reported it, or the
- `fn cancel_flag(self: &Self) -> Arc<AtomicBool>` - The cancel flag, for runners that poll it elsewhere.
- `fn write_back(self: &Self, secret_ref: &SecretRef, value: &[u8]) -> Result<(), WriteBackError>` - Writes a refreshed store token back to the owner (decision 3).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_node_client::jobs::HostJobError

*Struct*

How a job failed: a stable kind and an operator-facing message. Neither
may contain a secret.

**Fields:**
- `kind: String` - Stable error kind recorded on the owner's job row.
- `message: String` - Operator-facing message.

**Methods:**

- `fn new<impl Into<String>, impl Into<String>>(kind: impl Trait, message: impl Trait) -> Self` - A failure of `kind`.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &HostJobError) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> HostJobError`



## bookclerk_node_client::jobs::HostJobRunner

*Trait*

Runs claimed jobs on this host.

The runner must stop soon after `HostJobCtx::is_cancelled` turns true;
its result is then discarded. Error kinds and messages are recorded on
the owner and must never contain secret material.

**Methods:**

- `run`: Runs one job; `Ok` carries the outcome text recorded as progress.



## bookclerk_node_client::jobs::HostUpload

*Struct*

An open upload of one title: its id and bearer token. The token is
zeroized on drop and never shown by `Debug`.

**Fields:**
- `upload_id: String` - Upload id.

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)` - The token leaves the redaction registry (it was registered when
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_node_client::jobs::OwnerCallError

*Enum*

Why a call the job makes to the owner (results, plan, upload) failed.

**Variants:**
- `Lost` - The fence is lost; the job must stop (it is already cancelled).
- `Refused(String)` - The owner refused it; the reason is stable.
- `Unavailable` - No connection to the owner, or no usable answer.

**Traits:** Error, Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &OwnerCallError) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> OwnerCallError`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`



## bookclerk_node_client::jobs::WriteBackError

*Enum*

Why a token write-back was not stored.

**Variants:**
- `NotHanded` - The ref was not handed with this job; nothing was sent.
- `Size` - The value is empty or larger than the owner accepts; nothing was
- `Lost` - The fence is lost; the job must stop.
- `Refused(String)` - The owner refused it (`fence_not_live`, `not_trusted`, ...).
- `Unavailable` - No connection to the owner, or it did not answer in time.

**Traits:** Error, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &WriteBackError) -> bool`



