**bookclerkd > cluster_link > host_jobs**

# Module: cluster_link::host_jobs

## Contents

**Structs**

- [`AcquiredTitle`](#acquiredtitle) - What acquiring one title left in its outbox.
- [`GuestHost`](#guesthost) - The daemon's [`HostGuests`]: spawns the owner's plugin version
- [`HostDestination`](#hostdestination) - One output destination of the owner node an acquire was planned with.
- [`HostJobTransport`](#hostjobtransport) - The [`JobTransport`](../job_handler.md#bookclerkdjob_handlerjobtransport) for jobs an owner cluster handed this host.
- [`HostScan`](#hostscan) - What a host scan covers.
- [`HostTitle`](#hosttitle) - One title an acquire fetches here.

**Functions**

- [`acquire_request`](#acquire_request) - The acquire request for `book` named with `options`: what a host's
- [`copy_to_local_destinations`](#copy_to_local_destinations) - Copies one title's files (the primary destination's keys) to every one
- [`destination_outbox`](#destination_outbox) - Where destination `index`'s files go inside a title's outbox.
- [`is_work_id`](#is_work_id) - True when `id` (a job or title id the owner sent) is safe as one path
- [`job_work_path`](#job_work_path) - The scratch dir of one fence generation of job `job_id` under `dir`:
- [`outbox_keys`](#outbox_keys) - Every finished file under `dir`, as storage keys relative to it
- [`remove_local_copies`](#remove_local_copies) - Deletes `copies` (the files one run wrote for a title) from this host's
- [`title_cache_dir`](#title_cache_dir) - Where the acquire pipeline downloads and packages a title fetched into
- [`until_cancelled`](#until_cancelled) - Runs `work` until it ends or `cancel` turns true. On cancel the work is
- [`upload_manifest`](#upload_manifest) - The upload manifest of an acquired title: one entry per distinct file
- [`work_path`](#work_path) - `dir`'s child named by the owner-supplied `id`, refused unless `id` is

**Traits**

- [`HostGuests`](#hostguests) - The guest work a host job does; the daemon's is [`GuestHost`](host_jobs.md#bookclerkdcluster_linkhost_jobsguesthost).

**Constants**

- [`CANCEL_POLL`](#cancel_poll) - How often a running guest call or upload checks the job's cancel flag.
- [`MAX_BATCH_BYTES`](#max_batch_bytes) - Largest serialized `scan.results` batch (well under a frame).
- [`MAX_WORK_ID_BYTES`](#max_work_id_bytes) - Longest owner-supplied id used as a path segment under the work dir.

**Type Aliases**

- [`LocalCopy`](#localcopy) - One file this run wrote to one of this host's own destinations: the

---

## bookclerkd::cluster_link::host_jobs::AcquiredTitle

*Struct*

What acquiring one title left in its outbox.

**Fields:**
- `primary_key: String` - Storage key of the primary audio file on the primary destination.
- `pdf_key: Option<String>` - Storage key of the companion PDF on the primary destination, when
- `keys: Vec<Vec<String>>` - Every file per destination (in `HostTitle::destinations` order),

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AcquiredTitle`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::host_jobs::CANCEL_POLL

*Constant*: `std::time::Duration`

How often a running guest call or upload checks the job's cancel flag.



## bookclerkd::cluster_link::host_jobs::GuestHost

*Struct*

The daemon's [`HostGuests`]: spawns the owner's plugin version
materialized here as a jailed guest.

**Fields:**
- `state: std::sync::Arc<crate::api::AppState>` - Daemon state (config, this host's destinations).
- `files_dir: std::path::PathBuf` - This host's files dir.

**Methods:**

- `fn new(state: Arc<AppState>, files_dir: PathBuf) -> Self` - Guests for the daemon at `state`, whose files dir is `files_dir`.
- `fn open_source(self: &Self, bindings: &HostJobBindings) -> anyhow::Result<bookclerk_plugin_host::ExternalSource>` - Spawns the job's guest under the consented grant, with only the

**Trait Implementations:**

- **HostGuests**
  - `fn scan(self: &Self, bindings: &HostJobBindings, scan: HostScan, cancel: Arc<AtomicBool>) -> anyhow::Result<Vec<ScanBook>>`
  - `fn acquire(self: &Self, bindings: &HostJobBindings, title: &HostTitle, outbox: &Path, _cancel: Arc<AtomicBool>) -> anyhow::Result<AcquiredTitle>`
  - `fn local_destinations(self: &Self) -> anyhow::Result<Vec<Box<dyn StorageBackend>>>`
  - `fn work_dir(self: &Self) -> PathBuf`



## bookclerkd::cluster_link::host_jobs::HostDestination

*Struct*

One output destination of the owner node an acquire was planned with.

**Fields:**
- `id: String` - Its id on the owner node (`local`, `s3`).
- `kind: bookclerk_config::OutputBackendKind` - Its kind (one destination per kind).
- `options: bookclerk_source::DownloadOptions` - Its naming and packaging options.

**Methods:**

- `fn from_plan(plan: PlanDestination) -> anyhow::Result<Self>` - Parses a plan's destination; refuses an unknown kind or options

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> HostDestination`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::host_jobs::HostGuests

*Trait*

The guest work a host job does; the daemon's is [`GuestHost`](host_jobs.md#bookclerkdcluster_linkhost_jobsguesthost).

The transport drops a call once the job's cancel flag turns true (see
[`until_cancelled`](host_jobs.md#bookclerkdcluster_linkhost_jobsuntil_cancelled)); `cancel` is the same flag, for guests that can
also stop cooperatively.

**Methods:**

- `scan`: Runs the job's guest scan with the handed credentials and answers
- `acquire`: Acquires `title` with the job's guest into `outbox`: destination
- `local_destinations`: This host's own deployed output destinations (H-2: a host's local
- `work_dir`: Where per-job scratch lives (under this host's files dir).



## bookclerkd::cluster_link::host_jobs::HostJobTransport

*Struct*

The [`JobTransport`](../job_handler.md#bookclerkdjob_handlerjobtransport) for jobs an owner cluster handed this host.

**Generic Parameters:**
- G

**Fields:**
- `guests: G` - Guest work.

**Methods:**

- `fn new(guests: G) -> Self` - A transport over `guests`.

**Trait Implementations:**

- **JobTransport**
  - `fn execute(self: &Self, cmd: JobCommand, ctx: JobExecCtx) -> anyhow::Result<String>`



## bookclerkd::cluster_link::host_jobs::HostScan

*Struct*

What a host scan covers.

**Fields:**
- `accounts: Vec<String>` - Account ids; empty means every handed account.
- `import_episodes: bool` - Import podcast episodes (the owner's setting).
- `import_plus_titles: bool` - Import subscription (Plus) titles (the owner's setting).

**Trait Implementations:**

- **Default**
  - `fn default() -> HostScan`
- **Clone**
  - `fn clone(self: &Self) -> HostScan`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::host_jobs::HostTitle

*Struct*

One title an acquire fetches here.

**Fields:**
- `book: bookclerk_library::BookRecord` - The owner's library row.
- `parent: Option<bookclerk_library::BookRecord>` - The owner's row for the title's podcast parent, when it sent one;
- `destinations: Vec<HostDestination>` - The owner node's destinations, the primary first; never empty.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> HostTitle`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::host_jobs::LocalCopy

*Type Alias*: `(std::sync::Arc<dyn StorageBackend>, String)`

One file this run wrote to one of this host's own destinations: the
destination handle it was written through and the key. The handle is
kept rather than re-read from `HostGuests::local_destinations`, so a
destination deployed or retired before cleanup cannot redirect the
delete to a destination this run never wrote.



## bookclerkd::cluster_link::host_jobs::MAX_BATCH_BYTES

*Constant*: `usize`

Largest serialized `scan.results` batch (well under a frame).



## bookclerkd::cluster_link::host_jobs::MAX_WORK_ID_BYTES

*Constant*: `usize`

Longest owner-supplied id used as a path segment under the work dir.



## bookclerkd::cluster_link::host_jobs::acquire_request

*Function*

The acquire request for `book` named with `options`: what a host's
acquire runs, and what the owner computes the title's key scope from
(so both name the title the same way).

```rust
fn acquire_request(cfg: &bookclerk_config::Config, book: &bookclerk_library::BookRecord, options: &bookclerk_source::DownloadOptions) -> bookclerk_acquire::AcquireRequest
```



## bookclerkd::cluster_link::host_jobs::copy_to_local_destinations

*Function*

Copies one title's files (the primary destination's keys) to every one
of this host's own destinations. Any failed copy fails the title. Each
copy this run created (its key was not there before) is put in `copied`
as soon as it is written, so the caller removes exactly what this run
created ([`remove_local_copies`](host_jobs.md#bookclerkdcluster_linkhost_jobsremove_local_copies)) whether this fails or is dropped
mid-copy; a key that was already there is never deleted by it.

# Errors

Returns an error when the destinations cannot be used or a copy fails.

```rust
fn copy_to_local_destinations<G>(guests: &G, book_uuid: &str, outbox: &std::path::Path, acquired: &AcquiredTitle, copied: &std::sync::Mutex<Vec<(std::sync::Arc<dyn StorageBackend>, String)>>) -> anyhow::Result<()>
```



## bookclerkd::cluster_link::host_jobs::destination_outbox

*Function*

Where destination `index`'s files go inside a title's outbox.

```rust
fn destination_outbox(outbox: &std::path::Path, index: usize) -> std::path::PathBuf
```



## bookclerkd::cluster_link::host_jobs::is_work_id

*Function*

True when `id` (a job or title id the owner sent) is safe as one path
segment: non-empty, at most [`MAX_WORK_ID_BYTES`](host_jobs.md#bookclerkdcluster_linkhost_jobsmax_work_id_bytes), and only ASCII
letters, digits, `-` and `_`, so no separator, `.` / `..`, drive or
root prefix, or other special name can come through.

```rust
fn is_work_id(id: &str) -> bool
```



## bookclerkd::cluster_link::host_jobs::job_work_path

*Function*

The scratch dir of one fence generation of job `job_id` under `dir`:
`<job_id>-g<fence>`, one plain segment. A stale run still unwinding
after its lease expired and the replacement generation this host
claimed never share it, so the stale run's cleanup removes only its own
scratch. The name is unambiguous because `fence` is all digits: it is
whatever follows the last `-g`.

# Errors

Returns an error when `job_id` is not a usable segment ([`is_work_id`](host_jobs.md#bookclerkdcluster_linkhost_jobsis_work_id)).

```rust
fn job_work_path(dir: &std::path::Path, job_id: &str, fence: i64) -> anyhow::Result<std::path::PathBuf>
```



## bookclerkd::cluster_link::host_jobs::outbox_keys

*Function*

Every finished file under `dir`, as storage keys relative to it
(sorted; in-flight temp files are skipped).

# Errors

Returns an error when the directory cannot be read.

```rust
fn outbox_keys(dir: &std::path::Path) -> anyhow::Result<Vec<String>>
```



## bookclerkd::cluster_link::host_jobs::remove_local_copies

*Function*

Deletes `copies` (the files one run wrote for a title) from this host's
own destinations, best effort: a title the owner did not commit keeps
no copy here. Skipped when `fence_lost`: the owner may have handed the
job to a replacement run, here or elsewhere, that writes the same keys,
and a host-side copy carries no writer identity to tell the two apart,
so a stale run's cleanup could delete the replacement's file. The
stale copy is left for the replacement to overwrite.

```rust
fn remove_local_copies(book_uuid: &str, copies: &[(std::sync::Arc<dyn StorageBackend>, String)], fence_lost: bool)
```



## bookclerkd::cluster_link::host_jobs::title_cache_dir

*Function*

Where the acquire pipeline downloads and packages a title fetched into
`outbox`: beside its scratch library, inside the job dir, so a dropped
acquire (a cancel, a lost fence) leaves nothing once the job dir is
removed, and no other run (this host's own jobs, or a retry of the
title) shares it. Only this daemon touches it; the guest fetches into
its own plugin `tmp`.

```rust
fn title_cache_dir(outbox: &std::path::Path) -> std::path::PathBuf
```



## bookclerkd::cluster_link::host_jobs::until_cancelled

*Function*

Runs `work` until it ends or `cancel` turns true. On cancel the work is
dropped (for the daemon's guests that shuts the guest down) and the
answer is a `cancelled` error. Work that ends while `cancel` is set
(a guest that stopped cooperatively, returning what it had so far) is
answered `cancelled` too, never with its result.

# Errors

Returns `work`'s error, or `cancelled`.

```rust
fn until_cancelled<T, impl Future<Output = anyhow::Result<T>>>(cancel: &std::sync::atomic::AtomicBool, work: impl Trait) -> anyhow::Result<T>
```



## bookclerkd::cluster_link::host_jobs::upload_manifest

*Function*

The upload manifest of an acquired title: one entry per distinct file
content (size and SHA-256), listing its key on every destination that
has it, with the local path its bytes are read from.

# Errors

Returns an error when a file cannot be read, a key is unusable, a
destination has no files, or there are too many files.

```rust
fn upload_manifest(destinations: &[HostDestination], outbox: &std::path::Path, acquired: &AcquiredTitle) -> anyhow::Result<Vec<(bookclerk_node_proto::UploadFile, std::path::PathBuf)>>
```



## bookclerkd::cluster_link::host_jobs::work_path

*Function*

`dir`'s child named by the owner-supplied `id`, refused unless `id` is
one plain segment ([`is_work_id`](host_jobs.md#bookclerkdcluster_linkhost_jobsis_work_id)), so the host's job scratch (created,
written and removed recursively) never leaves `dir`.

# Errors

Returns an error when `id` is not a usable segment.

```rust
fn work_path(dir: &std::path::Path, id: &str) -> anyhow::Result<std::path::PathBuf>
```



