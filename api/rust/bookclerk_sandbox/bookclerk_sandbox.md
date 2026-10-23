**bookclerk_sandbox**

# Module: bookclerk_sandbox

## Contents

**Modules**

- [`spawn`](#spawn) - Windows AppContainer spawn API ([`plan_appcontainer`](platform/windows_spawn.md#bookclerk_sandboxplatformwindows_spawnplan_appcontainer),

**Structs**

- [`Capabilities`](#capabilities) - What this host can enforce, without applying anything.
- [`Policy`](#policy) - A confinement request: an allowlist of paths plus a few coarse switches.
- [`Report`](#report) - What actually engaged when a [`Policy`](bookclerk_sandbox.md#bookclerk_sandboxpolicy) was applied.
- [`ResourceLimits`](#resourcelimits) - Optional OS resource ceilings carried on a [`Policy`](bookclerk_sandbox.md#bookclerk_sandboxpolicy) / [`Spec`](spec.md#bookclerk_sandboxspecspec).

**Enums**

- [`CpuRate`](#cpurate) - CPU hard-cap carried on a [`Spec`](spec.md#bookclerk_sandboxspecspec) before label defaults are applied.
- [`Enforcement`](#enforcement) - What to do when a confinement layer cannot be enforced.
- [`LayerStatus`](#layerstatus) - Outcome for one confinement layer.
- [`NetPolicy`](#netpolicy) - Network reachability granted to the confined process.
- [`SandboxError`](#sandboxerror) - Confinement failure when applying or requiring a [`Policy`](bookclerk_sandbox.md#bookclerk_sandboxpolicy).

**Functions**

- [`capabilities`](#capabilities) - Probe what this host supports. Applies nothing; safe to call at any time.
- [`ensure_dir`](#ensure_dir) - Resolve a path for use in an allowlist, creating it if absent.
- [`host_cpu_rate_max`](#host_cpu_rate_max) - Maximum Spec / grant CPU rate: 100% × logical CPUs (one-core units).
- [`label_resource_defaults`](#label_resource_defaults) - Label-based Job defaults used when a [`Spec`](spec.md#bookclerk_sandboxspecspec) leaves resource fields unset.
- [`node_logical_cpus`](#node_logical_cpus) - Number of logical CPUs visible to this process (at least 1).
- [`windows_job_cpu_rate`](#windows_job_cpu_rate) - Map a percent-of-one-CPU hard cap onto a Windows Job Object `CpuRate`.

**Constants**

- [`GATEWAY_MAX_BLOCKING_THREADS`](#gateway_max_blocking_threads) - Blocking-pool ceiling for `bookclerk-workerd`.
- [`GATEWAY_WORKER_THREADS`](#gateway_worker_threads) - Tokio worker threads for `bookclerk-workerd`.
- [`INFRASTRUCTURE_THREAD_BUDGET`](#infrastructure_thread_budget) - Threads reserved for one native-behind-workerd session before `extraProcesses`.

---

## bookclerk_sandbox::Capabilities

*Struct*

What this host can enforce, without applying anything.

**Fields:**
- `backend: &'static str` - Backend name for this platform (e.g. `landlock+seccomp`).
- `filesystem: bool` - Whether a process can confine *itself* (Landlock / Seatbelt).
- `spawn_filesystem: bool` - Whether a child can be confined at spawn (AppContainer).
- `syscall: bool` - Whether syscall filtering (seccomp / equivalent) is available.
- `network: bool` - Whether network restriction can be applied on this host.
- `detail: String` - Human-readable detail, e.g. the Landlock ABI level found.

**Methods:**

- `fn can_confine_guest(self: &Self) -> bool` - Whether this host can confine a guest somehow (self-confine or spawn-time).

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &Capabilities) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> Capabilities`



## bookclerk_sandbox::CpuRate

*Enum*

CPU hard-cap carried on a [`Spec`](spec.md#bookclerk_sandboxspecspec) before label defaults are applied.

Omitted and JSON `null` are `Self::Unspecified` (the label default). A
JSON number is `Self::Percent`. `Self::Disabled` is the string `"off"`
and must not be filled from the label: sibling inner Jobs use it so they
do not pick up the plugin default of 80 under an outer cap.

**Variants:**
- `Unspecified` - Use [`label_resource_defaults`](bookclerk_sandbox.md#bookclerk_sandboxlabel_resource_defaults) when resolving a Job.
- `Disabled` - Do not apply a CPU rate, and do not substitute the label default.
- `Percent(u32)` - Percent of one logical CPU, clamped to `1..=`[`host_cpu_rate_max`](bookclerk_sandbox.md#bookclerk_sandboxhost_cpu_rate_max) when

**Methods:**

- `fn percent(self: Self) -> Option<u32>` - Percent when this is `Self::Percent`. Unspecified and disabled are
- `fn resolve(self: Self, label_default: Option<u32>) -> Option<u32>` - Label default for unspecified, nothing for disabled, the percent otherwise.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Serialize**
  - `fn serialize<S>(self: &Self, serializer: S) -> Result<<S as >::Ok, <S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CpuRate) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CpuRate`
- **Deserialize**
  - `fn deserialize<D>(deserializer: D) -> Result<Self, <D as >::Error>`
- **Default**
  - `fn default() -> CpuRate`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_sandbox::Enforcement

*Enum*

What to do when a confinement layer cannot be enforced.

**Variants:**
- `Required` - Fail the call when any requested layer does not engage.
- `BestEffort` - Apply what the host supports and report the rest as unsupported.
- `Disabled` - Apply nothing. Used only by the documented opt-out.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &Enforcement) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> Enforcement`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> Enforcement`



## bookclerk_sandbox::GATEWAY_MAX_BLOCKING_THREADS

*Constant*: `usize`

Blocking-pool ceiling for `bookclerk-workerd`.



## bookclerk_sandbox::GATEWAY_WORKER_THREADS

*Constant*: `usize`

Tokio worker threads for `bookclerk-workerd`.

Fixed so a session `pids.max` does not grow with [`node_logical_cpus`](bookclerk_sandbox.md#bookclerk_sandboxnode_logical_cpus).



## bookclerk_sandbox::INFRASTRUCTURE_THREAD_BUDGET

*Constant*: `u32`

Threads reserved for one native-behind-workerd session before `extraProcesses`.

Covers the pinned gateway runtime ([`GATEWAY_WORKER_THREADS`](bookclerk_sandbox.md#bookclerk_sandboxgateway_worker_threads) plus
[`GATEWAY_MAX_BLOCKING_THREADS`](bookclerk_sandbox.md#bookclerk_sandboxgateway_max_blocking_threads)), the pinned Cloudflare `workerd` process,
the native guest's current-thread runtime, and the two jail supervisors.
This is not the Windows outer process baseline (5 + extra) and not the
payload process count (3 + extra). `pids.max` counts threads; guest
`extraProcesses` are added on top of this budget.



## bookclerk_sandbox::LayerStatus

*Enum*

Outcome for one confinement layer.

**Variants:**
- `Enforced` - The layer engaged fully.
- `Partial(String)` - The layer engaged, but the host does not support every requested
- `NotApplicable(String)` - This platform draws its boundaries differently and has no separate
- `Unsupported(String)` - The host has no mechanism for this layer and the restriction is *not*
- `NotRequested` - The policy did not ask for this layer.

**Methods:**

- `fn is_active(self: &Self) -> bool` - Whether this layer is itself providing protection.
- `fn is_gap(self: &Self) -> bool` - Whether this status means a requested restriction is missing.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LayerStatus`
- **PartialEq**
  - `fn eq(self: &Self, other: &LayerStatus) -> bool`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_sandbox::NetPolicy

*Enum*

Network reachability granted to the confined process.

**Variants:**
- `Deny` - No IP sockets at all. Media workers only touch local files.
- `Outbound` - Outbound connections allowed, inbound listeners refused. Enough for a
- `OutboundListen` - Outbound connections plus a listener on a kernel-assigned port.
- `Full` - Unrestricted. The daemon binds its own control-plane listener.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &NetPolicy) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NetPolicy`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> NetPolicy`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_sandbox::Policy

*Struct*

A confinement request: an allowlist of paths plus a few coarse switches.

Paths that do not exist when the policy is applied are skipped, because a
Landlock rule on a missing path is an error rather than a wider grant.

**Methods:**

- `fn new<impl Into<String>>(label: impl Trait) -> Self` - Start a deny-default policy. `label` appears in diagnostics only.
- `fn read<impl Into<PathBuf>>(self: Self, path: impl Trait) -> Self` - Grant read access to `path` and everything beneath it.
- `fn write<impl Into<PathBuf>>(self: Self, path: impl Trait) -> Self` - Grant read and write access to `path` and everything beneath it.
- `fn reads<I, P>(self: Self, paths: I) -> Self` - Grant read access to each of `paths`.
- `fn writes<I, P>(self: Self, paths: I) -> Self` - Grant read and write access to each of `paths`.
- `fn net(self: Self, net: NetPolicy) -> Self` - Set network reachability. Defaults to `NetPolicy::Deny`.
- `fn allow_exec(self: Self, allow: bool) -> Self` - Allow `execve` of binaries inside the read allowlist. Defaults to false.
- `fn system_paths(self: Self, include: bool) -> Self` - Include the platform's read-only system set (shared libraries, the CA
- `fn enforcement(self: Self, enforcement: Enforcement) -> Self` - Set the failure mode. Defaults to `Enforcement::Required`.
- `fn memory_bytes(self: Self, bytes: Option<u64>) -> Self` - Soft memory ceiling in bytes (`None` = platform default / unset).
- `fn active_processes(self: Self, n: Option<u32>) -> Self` - Cap on concurrent processes (`None` = platform default / unset).
- `fn cpu_rate(self: Self, rate: CpuRate) -> Self` - CPU hard-cap. `CpuRate::Unspecified` takes the label default;
- `fn cpu_rate_percent(self: Self, percent: Option<u32>) -> Self` - CPU hard-cap percent of one logical CPU.
- `fn unix_socket_dirs(self: Self, dirs: Option<Vec<PathBuf>>) -> Self` - Restrict pathname Unix sockets to `dirs` (`Some([])` denies them).
- `fn cgroup_dir(self: Self, dir: Option<PathBuf>) -> Self` - Join this cgroup v2 leaf instead of creating `bookclerk-<pid>`.
- `fn unix_socket_dirs_opt(self: &Self) -> Option<&[PathBuf]>` - Explicit Unix-socket directories, or `None` for the writable-path default.
- `fn cgroup_dir_opt(self: &Self) -> Option<&std::path::Path>` - Optional session cgroup to join (Linux).
- `fn label(self: &Self) -> &str` - Diagnostics label supplied to `Self::new` (logs / doctor output only).
- `fn resource_limits(self: &Self) -> ResourceLimits` - Spec-provided resource ceilings (before label heuristics).
- `fn has_resource_limits(self: &Self) -> bool` - Whether the Spec/Policy asked for any OS resource ceiling.
- `fn resolved_job_limits(self: &Self) -> ResourceLimits` - Job Object limits: Spec fields override [`label_resource_defaults`](bookclerk_sandbox.md#bookclerk_sandboxlabel_resource_defaults).
- `fn resolved_reads(self: &Self) -> Vec<PathBuf>` - Read allowlist, including the platform system set when enabled.
- `fn resolved_writes(self: &Self) -> Vec<PathBuf>` - Write allowlist, including the few system paths that have to be writable
- `fn net_policy(self: &Self) -> NetPolicy` - Network reachability granted by this policy.
- `fn exec_allowed(self: &Self) -> bool` - Whether `execve` is permitted.
- `fn enforcement_mode(self: &Self) -> Enforcement` - What to do when a requested layer cannot engage.
- `fn confine_current_process(self: &Self) -> Result<Report, SandboxError>` - Confine the calling process. Restrictions are irreversible and are

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> Policy`



## bookclerk_sandbox::Report

*Struct*

What actually engaged when a [`Policy`](bookclerk_sandbox.md#bookclerk_sandboxpolicy) was applied.

**Fields:**
- `label: String` - The policy's diagnostics label.
- `backend: &'static str` - Backend name, e.g. `landlock+seccomp`.
- `filesystem: LayerStatus` - Filesystem allowlist (Landlock / Seatbelt / AppContainer) status.
- `syscall: LayerStatus` - Syscall restriction (seccomp / Seatbelt operation classes) status.
- `network: LayerStatus` - Network restriction status for the requested [`NetPolicy`](bookclerk_sandbox.md#bookclerk_sandboxnetpolicy).
- `resources: LayerStatus` - Memory / CPU / process ceilings (Job Object / cgroup v2).

**Methods:**

- `fn is_confined(self: &Self) -> bool` - Whether the filesystem allowlist — the layer that protects `master.key`
- `fn summary(self: &Self) -> String` - One-line summary for startup logs and `bookclerk doctor`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> Report`
- **PartialEq**
  - `fn eq(self: &Self, other: &Report) -> bool`



## bookclerk_sandbox::ResourceLimits

*Struct*

Optional OS resource ceilings carried on a [`Policy`](bookclerk_sandbox.md#bookclerk_sandboxpolicy) / [`Spec`](spec.md#bookclerk_sandboxspecspec).

Windows applies these (merged with label heuristics) via a Job Object.
Linux applies them best-effort via cgroup v2 when at least one field is set.
macOS Seatbelt has no equivalent — see `Policy::has_resource_limits`.

**Fields:**
- `memory_bytes: Option<u64>` - Process memory ceiling in bytes.
- `cpu_rate_percent: Option<u32>` - CPU hard-cap as a percent of **one logical CPU** (1..=host cores×100).
- `active_processes: Option<u32>` - Maximum concurrent processes in the jail.

**Methods:**

- `fn is_empty(self: &Self) -> bool` - Whether any limit was requested.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ResourceLimits) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ResourceLimits`
- **Default**
  - `fn default() -> ResourceLimits`



## bookclerk_sandbox::SandboxError

*Enum*

Confinement failure when applying or requiring a [`Policy`](bookclerk_sandbox.md#bookclerk_sandboxpolicy).

**Variants:**
- `NotEnforced{ label: String, layer: &'static str, detail: String }` - A required layer did not engage under `Enforcement::Required`.
- `Backend{ label: String, backend: &'static str, detail: String }` - A backend call failed outright (kernel / API error).

**Traits:** Error

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`



## bookclerk_sandbox::capabilities

*Function*

Probe what this host supports. Applies nothing; safe to call at any time.

```rust
fn capabilities() -> Capabilities
```



## bookclerk_sandbox::ensure_dir

*Function*

Resolve a path for use in an allowlist, creating it if absent.

Landlock and Seatbelt both need the path to exist when the rule is added, so
callers that will write into a scratch directory should create it first.

# Errors

Propagates directory-creation failures.

```rust
fn ensure_dir(path: &std::path::Path) -> std::io::Result<std::path::PathBuf>
```



## bookclerk_sandbox::host_cpu_rate_max

*Function*

Maximum Spec / grant CPU rate: 100% × logical CPUs (one-core units).

Example: 8 logical CPUs → `800` (eight full cores of CFS/Job bandwidth).

```rust
fn host_cpu_rate_max() -> u32
```



## bookclerk_sandbox::label_resource_defaults

*Function*

Label-based Job defaults used when a [`Spec`](spec.md#bookclerk_sandboxspecspec) leaves resource fields unset.

Plugin guests are capped tightly; labels containing `"media"` get more
headroom (matching the historical Windows Job heuristics).

```rust
fn label_resource_defaults(label: &str) -> ResourceLimits
```



## bookclerk_sandbox::node_logical_cpus

*Function*

Number of logical CPUs visible to this process (at least 1).

```rust
fn node_logical_cpus() -> u32
```



## Module: spawn

Windows AppContainer spawn API ([`plan_appcontainer`](platform/windows_spawn.md#bookclerk_sandboxplatformwindows_spawnplan_appcontainer),
[`run_appcontainer`](platform/windows_spawn.md#bookclerk_sandboxplatformwindows_spawnrun_appcontainer)).

Windows AppContainer planning / launch helpers re-exported on every OS so
callers can depend on a stable `bookclerk_sandbox::spawn` path. Non-Windows
builds keep the planning helpers and return a clear error from launch/ACL
entry points.



## bookclerk_sandbox::windows_job_cpu_rate

*Function*

Map a percent-of-one-CPU hard cap onto a Windows Job Object `CpuRate`.

Job `CpuRate` is “cycles per 10 000” of **machine** capacity. Bookclerk’s
`ResourceLimits::cpu_rate_percent` is percent of **one** logical CPU, so
this divides by `logical_cpus` (at least 1). Result is clamped to `1..=10000`
because `CpuRate = 0` is rejected by the API.

# Examples

- 80% of one core on an 8-CPU host → 10% of the machine → `1000`
- 400% (four cores) on an 8-CPU host → 50% of the machine → `5000`

```rust
fn windows_job_cpu_rate(percent_of_one_cpu: u32, logical_cpus: u32) -> u32
```



