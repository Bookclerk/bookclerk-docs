**bookclerkd > jobs**

# Module: jobs

## Contents

**Structs**

- [`AcquireAsk`](#acquireask) - What one acquire request asks for.
- [`AcquireMode`](#acquiremode) - The flags of one acquire job.
- [`JobUnroutable`](#jobunroutable) - Admission refused a job that no node could ever claim.

**Enums**

- [`AdmitJob`](#admitjob) - Result of admitting work into the durable queue.

**Functions**

- [`enqueue_acquire`](#enqueue_acquire) - Admit acquire jobs through the acquire planner.
- [`enqueue_integration_scan`](#enqueue_integration_scan) - Admit a remote integration library scan.
- [`enqueue_listen_sync`](#enqueue_listen_sync) - Admit a listening-progress sync.
- [`enqueue_plugin_copy`](#enqueue_plugin_copy) - Admit an ABI `JobHandler` stream-copy invocation.
- [`enqueue_scan`](#enqueue_scan) - Admit a library scan through the scan planner. Does not spawn a
- [`note_job_failure`](#note_job_failure) - Log a handler failure and request a diagnostics upload.
- [`run_acquire`](#run_acquire) - Acquire pending titles synchronously.
- [`run_integration_scan`](#run_integration_scan) - Run a remote integration library scan synchronously.
- [`run_listen_sync`](#run_listen_sync) - Sync listening progress from capable integrations.
- [`run_plugin_copy`](#run_plugin_copy) - Runs `plugin.worker().handle(stream_copy)` on a loaded destination guest.
- [`run_scan`](#run_scan) - Run a scan synchronously (worker / tests).

**Constants**

- [`JOB_SUSPENDED_DETAIL_PREFIX`](#job_suspended_detail_prefix) - Progress/detail prefix after a fenced `bookclerk_plugin_host::JobOutcome::Suspended` commit.

---

## bookclerkd::jobs::AcquireAsk

*Struct*

What one acquire request asks for.

**Fields:**
- `titles: Vec<String>` - Title ids (uuid, ASIN, ISBN, product id); each becomes its own job.
- `account: Option<String>` - Limit to one store account.
- `plugin_instance_id: Option<String>` - Pin every fetch to one deployed instance of the book's plugin;
- `force: bool` - Re-download titles already acquired.
- `pdf_only: bool` - Fetch only each title's companion PDF.

**Trait Implementations:**

- **Default**
  - `fn default() -> AcquireAsk`
- **Clone**
  - `fn clone(self: &Self) -> AcquireAsk`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::jobs::AcquireMode

*Struct*

The flags of one acquire job.

**Fields:**
- `force: bool` - Re-download titles already acquired.
- `pdf_only: bool` - Fetch only each title's companion PDF.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> AcquireMode`
- **Default**
  - `fn default() -> AcquireMode`
- **PartialEq**
  - `fn eq(self: &Self, other: &AcquireMode) -> bool`



## bookclerkd::jobs::AdmitJob

*Enum*

Result of admitting work into the durable queue.

**Variants:**
- `Created(String)` - A new row was inserted.
- `Duplicate(String)` - An equivalent pending/running job already exists.
- `QueueFull` - The pending+running cap was reached.

**Traits:** Eq

**Trait Implementations:**

- **From**
  - `fn from(value: EnqueueOutcome) -> Self`
- **Clone**
  - `fn clone(self: &Self) -> AdmitJob`
- **PartialEq**
  - `fn eq(self: &Self, other: &AdmitJob) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::jobs::JOB_SUSPENDED_DETAIL_PREFIX

*Constant*: `&str`

Progress/detail prefix after a fenced `bookclerk_plugin_host::JobOutcome::Suspended` commit.

[`crate::job_worker`](job_worker.md) must not call `complete_job` for this outcome: suspend already moved
the row to `pending`.



## bookclerkd::jobs::JobUnroutable

*Struct*

Admission refused a job that no node could ever claim.

The job's required plugin instances share no node where all of them are
deployed to run, so the row would wait forever. Callers report it as a
conflict the operator fixes by deploying the instances to a common node,
not as a server failure.

**Tuple Struct**: `(String)`

**Traits:** Eq, Error

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> JobUnroutable`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobUnroutable) -> bool`
- **Display**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerkd::jobs::enqueue_acquire

*Function*

Admit acquire jobs through the acquire planner.

`control_plane::plan_acquire` decides what `requester` may acquire and
splits the request: one job per title id, and for a user who is not an
Owner or Administrator one job per account they own.

# Returns

One admission outcome per planned job, in plan order. Empty when the
requester owns nothing to acquire.

# Errors

Fails with a wrapped `bookclerk_library::LibraryError::Forbidden` when
the requester may not acquire what was asked, and with the read or write
error otherwise. Jobs admitted before a failure stay admitted.

```rust
fn enqueue_acquire(state: std::sync::Arc<crate::api::AppState>, requester: bookclerk_library::control_plane::ScanRequester, ask: AcquireAsk, trigger: bookclerk_library::JobTrigger) -> anyhow::Result<Vec<AdmitJob>>
```



## bookclerkd::jobs::enqueue_integration_scan

*Function*

Admit a remote integration library scan.

```rust
fn enqueue_integration_scan(state: std::sync::Arc<crate::api::AppState>, integration_id: String, force: bool, trigger: bookclerk_library::JobTrigger) -> anyhow::Result<AdmitJob>
```



## bookclerkd::jobs::enqueue_listen_sync

*Function*

Admit a listening-progress sync.

```rust
fn enqueue_listen_sync(state: std::sync::Arc<crate::api::AppState>, trigger: bookclerk_library::JobTrigger) -> anyhow::Result<AdmitJob>
```



## bookclerkd::jobs::enqueue_plugin_copy

*Function*

Admit an ABI `JobHandler` stream-copy invocation.

```rust
fn enqueue_plugin_copy(state: std::sync::Arc<crate::api::AppState>, plugin_id: String, source_key: String, dest_key: String, trigger: bookclerk_library::JobTrigger) -> anyhow::Result<AdmitJob>
```



## bookclerkd::jobs::enqueue_scan

*Function*

Admit a library scan through the scan planner. Does not spawn a
per-request task.

`control_plane::plan_scan` decides what `requester` may scan: operator
authority, an Owner or an Administrator scanning every account gets one
job per scan-enabled account (so each can go to any node trusted with
it); any other user only scans accounts they own. `plugin_instance_id`
pins every planned job to one deployed instance.

# Returns

One admission outcome per planned job, in plan order. Empty when the
requester owns nothing to scan.

# Errors

Fails with a wrapped `bookclerk_library::LibraryError::Forbidden` when
the requester may not scan what was asked, and with the read or write
error otherwise. Jobs admitted before a failure stay admitted.

```rust
fn enqueue_scan(state: std::sync::Arc<crate::api::AppState>, requester: bookclerk_library::control_plane::ScanRequester, account: Option<String>, plugin_instance_id: Option<String>, trigger: bookclerk_library::JobTrigger) -> anyhow::Result<Vec<AdmitJob>>
```



## bookclerkd::jobs::note_job_failure

*Function*

Log a handler failure and request a diagnostics upload.

```rust
fn note_job_failure(job_id: &str, err: &anyhow::Error)
```



## bookclerkd::jobs::run_acquire

*Function*

Acquire pending titles synchronously.

When `ctx` is set, progress is fenced and cancel is checked between titles.

`plugin_instance_id` pins every fetch to that deployed instance of the
book's plugin; otherwise the plugin's default instance is used. `mode`
carries the job's `force` and `pdf_only` flags.

```rust
fn run_acquire(state: &crate::api::AppState, title: Option<&str>, account: Option<&str>, plugin_instance_id: Option<&str>, mode: AcquireMode, ctx: Option<&crate::job_handler::JobExecCtx>) -> anyhow::Result<String>
```



## bookclerkd::jobs::run_integration_scan

*Function*

Run a remote integration library scan synchronously.

The remote `scan_library` RPC is not interruptible. Cancel is checked
before the call; a repeat scan is idempotent (remote upsert / rescan).

```rust
fn run_integration_scan(state: &crate::api::AppState, integration_id: &str, force: bool, ctx: Option<&crate::job_handler::JobExecCtx>) -> anyhow::Result<String>
```



## bookclerkd::jobs::run_listen_sync

*Function*

Sync listening progress from capable integrations.

Cancel is checked between providers. An in-flight provider RPC finishes.

```rust
fn run_listen_sync(state: &crate::api::AppState, ctx: Option<&crate::job_handler::JobExecCtx>) -> anyhow::Result<String>
```



## bookclerkd::jobs::run_plugin_copy

*Function*

Runs `plugin.worker().handle(stream_copy)` on a loaded destination guest.

```rust
fn run_plugin_copy(state: &crate::api::AppState, plugin_id: Option<&str>, source_key: Option<&str>, dest_key: Option<&str>, ctx: Option<&crate::job_handler::JobExecCtx>) -> anyhow::Result<String>
```



## bookclerkd::jobs::run_scan

*Function*

Run a scan synchronously (worker / tests).

Cancel is checked between sources. An in-flight source page fetch finishes
before the next source starts.

`plugin_instance_id` pins the scan to one deployed instance; otherwise
every plugin is scanned once through its default instance.

```rust
fn run_scan(state: &crate::api::AppState, account: Option<&str>, plugin_instance_id: Option<&str>, ctx: Option<&crate::job_handler::JobExecCtx>) -> anyhow::Result<String>
```



