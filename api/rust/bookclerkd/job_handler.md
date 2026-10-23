**bookclerkd > job_handler**

# Module: job_handler

## Contents

**Structs**

- [`InProcessJobTransport`](#inprocessjobtransport) - In-process adapter used by `bookclerkd` today.
- [`JobCommand`](#jobcommand) - Versioned command decoded from a durable job row.
- [`JobExecCtx`](#jobexecctx) - Fence + cooperative cancel token handed to a handler.

**Traits**

- [`JobTransport`](#jobtransport) - Executes a validated [`JobCommand`](job_handler.md#bookclerkdjob_handlerjobcommand).

**Constants**

- [`JOB_COMMAND_VERSION`](#job_command_version) - Current command envelope version accepted by [`InProcessJobTransport`](job_handler.md#bookclerkdjob_handlerinprocessjobtransport).

---

## bookclerkd::job_handler::InProcessJobTransport

*Struct*

In-process adapter used by `bookclerkd` today.

**Fields:**
- `state: std::sync::Arc<crate::api::AppState>` - Shared daemon state (config, library, integrations).

**Methods:**

- `fn new(state: Arc<AppState>) -> Self` - Builds an adapter over `state`.

**Trait Implementations:**

- **JobTransport**
  - `fn execute(self: &Self, cmd: JobCommand, ctx: JobExecCtx) -> anyhow::Result<String>`



## bookclerkd::job_handler::JOB_COMMAND_VERSION

*Constant*: `u32`

Current command envelope version accepted by [`InProcessJobTransport`](job_handler.md#bookclerkdjob_handlerinprocessjobtransport).



## bookclerkd::job_handler::JobCommand

*Struct*

Versioned command decoded from a durable job row.

**Fields:**
- `version: u32` - Envelope version (`JobPayload.v`).
- `kind: bookclerk_library::JobKind` - Command kind.
- `payload: bookclerk_library::JobPayload` - Kind-specific filters.

**Methods:**

- `fn from_record(job: &JobRecord) -> anyhow::Result<Self>` - Builds a command from a claimed row, rejecting unknown/invalid kinds.
- `fn from_parts(kind: JobKind, payload: JobPayload) -> anyhow::Result<Self>` - Builds a command from a kind and payload (a job handed by an owner



## bookclerkd::job_handler::JobExecCtx

*Struct*

Fence + cooperative cancel token handed to a handler.

**Fields:**
- `fence: bookclerk_library::JobFence` - Lease identity the handler must use for progress and finalization.
- `cancel: std::sync::Arc<std::sync::atomic::AtomicBool>` - Set when the heartbeat loses the fence or the operator cancels.
- `attempt_count: i64` - 1-based attempt counter from the claim that produced this fence.
- `dedup_key: String` - Durable dedup key; stable across reclaim of the same command.
- `lease_expires_at: Option<chrono::DateTime<chrono::Utc>>` - Lease expiry; used as the guest `deadlineUnixMs` hint.
- `checkpoint: Option<bookclerk_plugin_host::JobCheckpoint>` - Checkpoint restored after a prior suspension, when present.
- `invocation_sequence: u32` - Resume ordinal from the durable payload; distinct from `Self::attempt_count`.
- `host: Option<std::sync::Arc<bookclerk_node_client::HostJobCtx>>` - Set when this daemon runs the job as a host node for an owner
- `host_bindings: Option<std::sync::Arc<crate::cluster_link::remote_bindings::HostJobBindings>>` - Set with [`Self::host`]: what the owner plugin's guest is opened

**Methods:**

- `fn is_cancelled(self: &Self) -> bool` - True when the handler should stop after the current step.
- `fn request_cancel(self: &Self)` - Marks the handler cancelled (heartbeat loss or operator request).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobExecCtx`



## bookclerkd::job_handler::JobTransport

*Trait*

Executes a validated [`JobCommand`](job_handler.md#bookclerkdjob_handlerjobcommand).

**Methods:**

- `execute`: Runs `cmd` until it finishes or `ctx` is cancelled.



