# bookclerk_sandbox

Portable, unprivileged process confinement.

## Audience

Host binaries that confine media workers and plugin guests (`bookclerk-jail`,
`bookclerk-media-worker`). Guest plugins never link this crate; the host
imposes the jail. See `docs/internals/guest-jail.md#the-guest-jail` and `docs/server/media.md`.

Bookclerk confines code by what it may touch, not by which uid it runs as.
Every backend here works without root, without setuid helpers, and without
user namespaces or bind mounts:

- **Linux** — Landlock filesystem allowlist plus a seccomp-bpf deny list.
- **macOS** — Seatbelt (`sandbox_init`) with a deny-default SBPL profile.
- **Windows** — AppContainer applied at `CreateProcess`; see
  [`spawn`](spawn.md). A process cannot confine *itself* on Windows, so
  [`Policy::confine_current_process`](#policyconfine_current_process) reports the filesystem layer as
  [`LayerStatus::Unsupported`](#layerstatus) there. Children are confined by
  [`spawn::run_appcontainer`](spawn.md#run_appcontainer) (used by `bookclerk-jail`).

Callers pick the failure mode with [`Enforcement`](#enforcement). `Required` turns a
backend that cannot enforce into an error, which is what production paths
use so a missing jail can never pass silently.

| Item | Kind | Summary |
| --- | --- | --- |
| [`CgroupSample`](#cgroupsample) | struct | Parsed cgroup v2 files for the process that is actually running. |
| [`SystemMemory`](#systemmemory) | struct | Whole-machine memory figures for heartbeats. |
| [`DuplexHalf`](#duplexhalf) | struct | One unidirectional pipe end (stdio). |
| [`DuplexLink`](#duplexlink) | struct | One end of a host-created duplex IPC link (socketpair / overlapped pipe). |
| [`JailHandoff`](#jailhandoff) | struct | One-shot Windows jail stdin line. Bounded JSON; never a general broker. |
| [`JailHandoffExtra`](#jailhandoffextra) | struct | One extra handle named in the child environment. |
| [`StdioEnds`](#stdioends) | struct | Host and guest ends of a pair of unidirectional stdio pipes (RPC). |
| [`Spec`](#spec) | struct | A serializable confinement request. |
| [`Policy`](#policy) | struct | A confinement request: an allowlist of paths plus a few coarse switches. |
| [`ResourceLimits`](#resourcelimits) | struct | Optional OS resource ceilings carried on a `Policy` / `Spec`. |
| [`Report`](#report) | struct | What actually engaged when a `Policy` was applied. |
| [`Capabilities`](#capabilities) | struct | What this host can enforce, without applying anything. |
| [`CpuTimes`](#cputimes) | enum | Cumulative CPU time counters. Two samples give utilization. |
| [`LinkSpec`](#linkspec) | enum | How a confined child names an inherited duplex or pipe end. |
| [`LinkSpecError`](#linkspecerror) | enum | Why a `LinkSpec` string could not be parsed. |
| [`SpawnPathError`](#spawnpatherror) | enum | Why a spawn path was rejected. |
| [`Enforcement`](#enforcement) | enum | What to do when a confinement layer cannot be enforced. |
| [`NetPolicy`](#netpolicy) | enum | Network reachability granted to the confined process. |
| [`CpuRate`](#cpurate) | enum | CPU hard-cap carried on a `Spec` before label defaults are applied. |
| [`LayerStatus`](#layerstatus) | enum | Outcome for one confinement layer. |
| [`SandboxError`](#sandboxerror) | enum | Confinement failure when applying or requiring a `Policy`. |
| [`cgroup_sample_is_low_resource`](#cgroup_sample_is_low_resource) | function | True when `memory.max` is at most 1 GiB or `cpu.max` is at most one core. |
| [`cpu_used_millicores`](#cpu_used_millicores) | function | CPU in use between two samples, in millicores (1000 = one core busy). |
| [`filesystem_free_bytes`](#filesystem_free_bytes) | function | Free bytes available to unprivileged callers. |
| [`logical_cpu_count`](#logical_cpu_count) | function | Logical CPUs from `std::thread::available_parallelism`, or empty on failure. |
| [`macos_cpu_ticks`](#macos_cpu_ticks) | function | macOS `HOST_CPU_LOAD_INFO` ticks (`user`, `system`, `idle`, `nice`, summed over all CPUs) as `CpuTimes::System`. |
| [`macos_vm_memory`](#macos_vm_memory) | function | macOS available memory from `HOST_VM_INFO64` page counts. |
| [`parse_cpu_stat_usage`](#parse_cpu_stat_usage) | function | `usage_usec` from cgroup v2 `cpu.stat` text. |
| [`parse_meminfo`](#parse_meminfo) | function | Parses `/proc/meminfo` text (values are in kB). |
| [`parse_proc_stat`](#parse_proc_stat) | function | Aggregate `cpu` line of `/proc/stat` text. |
| [`process_cgroup_dir`](#process_cgroup_dir) | function | Directory of this process's cgroup v2 node (`/proc/self/cgroup` `0::` path). |
| [`read_cgroup_sample`](#read_cgroup_sample) | function | Reads `cpu.max`, `memory.max`, `memory.current`, and `memory.stat` in `dir`. |
| [`read_cpu_times`](#read_cpu_times) | function | Reads CPU time counters for utilization: the cgroup's own usage when the process runs under a CPU-limited cgroup (a container quota is the real ceiling), else the whole machine (a desktop's other applications count against what Bookclerk can use). The whole-machine counters come from `/proc/stat` on Linux, `HOST_CPU_LOAD_INFO` on macOS, and `GetSystemTimes` on Windows. |
| [`read_system_memory`](#read_system_memory) | function | Reads total and available physical memory for this machine. |
| [`windows_system_times`](#windows_system_times) | function | Windows `GetSystemTimes` values (100 ns units) as `CpuTimes::System`. |
| [`create_guest_ipc_dir`](#create_guest_ipc_dir) | function | Create a mode `0700` directory under `/tmp` whose longest guest socket fits in a macOS `sockaddr_un`. |
| [`ensure_guest_ipc_fits`](#ensure_guest_ipc_fits) | function | Reject `dir` when `{dir}/.s.PGSQL.65535` does not fit in a macOS `sockaddr_un`. |
| [`with_fd_spawn_lock`](#with_fd_spawn_lock) | function | Serialize macOS descriptor allocation with in-process `Command` spawns. |
| [`inherit_fd_at`](#inherit_fd_at) | function | `dup2` `src` onto `dest` and clear `FD_CLOEXEC` on the destination. |
| [`create_session_cgroup`](#create_session_cgroup) | function | Create an exclusive session cgroup leaf and write `limits` without moving the caller. |
| [`destroy_session_cgroup`](#destroy_session_cgroup) | function | Kill every member of `dir` and remove the leaf. |
| [`recent_platform_spawn_diagnostics`](#recent_platform_spawn_diagnostics) | function | Startup lines recorded by AppContainer profile, DACL, and proxy stages. |
| [`join_capped_lines`](#join_capped_lines) | function | Join `ring` and cap the copy at `total_bytes`. |
| [`push_capped_line`](#push_capped_line) | function | Push `line` onto `ring`, dropping the oldest entries until both caps hold. |
| [`record_spawn_diagnostic`](#record_spawn_diagnostic) | function | Remember one already-formatted line and emit it. |
| [`redact_capped`](#redact_capped) | function | Redact secrets and keep at most `max_bytes` of the result. |
| [`redact_diagnostic_text`](#redact_diagnostic_text) | function | Replace a 64-hex session challenge and the challenge env name. |
| [`snapshot_spawn_diagnostics`](#snapshot_spawn_diagnostics) | function | Byte-capped snapshot of `record_spawn_diagnostic` lines. |
| [`spawn_diag_raw_stderr`](#spawn_diag_raw_stderr) | function | `true` when raw stage lines should be written to this process's stderr. |
| [`spawn_diag_stderr_enabled`](#spawn_diag_stderr_enabled) | function | `true` when `BOOKCLERK_SPAWN_DIAG` is `1`, `true`, or `stderr`. |
| [`truncate_utf8`](#truncate_utf8) | function | Copy `text` up to `max_bytes` without splitting a scalar value. |
| [`canonicalize`](#canonicalize) | function | `std::fs::canonicalize` that also works inside a Windows AppContainer. |
| [`create_process_path`](#create_process_path) | function | Path passed to `CreateProcess`, `Command`, or a child that parses Win32 paths. |
| [`require_absolute_or_name`](#require_absolute_or_name) | function | Absolute path with no NUL, or a single PATH lookup name (`cargo`, `python3`). |
| [`require_absolute_spawn_path`](#require_absolute_spawn_path) | function | Like `require_absolute_or_name`, but the path must be absolute (no PATH names). |
| [`require_existing_regular_file`](#require_existing_regular_file) | function | Existing regular file after resolving relative paths against the process cwd. |
| [`require_helper_beside_or_absolute`](#require_helper_beside_or_absolute) | function | Accepts `path` when it equals `beside`'s sibling named `helper_name`, or is an absolute existing file (env override). |
| [`require_spawn_executable`](#require_spawn_executable) | function | Absolute existing regular file (or a single PATH name), with no interior NUL. |
| [`require_under_root`](#require_under_root) | function | Canonicalizes `path` and requires it to stay under canonical `root`. |
| [`node_logical_cpus`](#node_logical_cpus) | function | Number of logical CPUs visible to this process (at least 1). |
| [`host_cpu_rate_max`](#host_cpu_rate_max) | function | Maximum Spec / grant CPU rate: 100% × logical CPUs (one-core units). |
| [`windows_job_cpu_rate`](#windows_job_cpu_rate) | function | Map a percent-of-one-CPU hard cap onto a Windows Job Object `CpuRate`. |
| [`label_resource_defaults`](#label_resource_defaults) | function | Label-based Job defaults used when a `Spec` leaves resource fields unset. |
| [`capabilities`](#capabilities-1) | function | Probe what this host supports. Applies nothing; safe to call at any time. |
| [`ensure_dir`](#ensure_dir) | function | Resolve a path for use in an allowlist, creating it if absent. |
| [`LOW_RESOURCE_MEMORY_MAX_BYTES`](#low_resource_memory_max_bytes) | constant | `memory.max` at or below this many bytes is the small-VPS ceiling. |
| [`MACOS_SUN_PATH_CAPACITY`](#macos_sun_path_capacity) | constant | macOS `sun_path` capacity. Rust rejects a path whose length is greater than or equal to this value. |
| [`GATEWAY_GUEST_RPC_ENV`](#gateway_guest_rpc_env) | constant | Env var naming the guest Cap'n Proto link for `bookclerk-workerd`. |
| [`GATEWAY_GUEST_RPC_WRITE_ENV`](#gateway_guest_rpc_write_env) | constant | Windows-only env var naming the gateway's overlapped write half of guest RPC. |
| [`GATEWAY_PROXY_ENV`](#gateway_proxy_env) | constant | Env var naming the multiplexed CONNECT-proxy link for `bookclerk-workerd`. |
| [`GATEWAY_PROXY_FD`](#gateway_proxy_fd) | constant | Child fd the gateway uses for the proxy mux (`BOOKCLERK_GATEWAY_PROXY`). |
| [`GATEWAY_PROXY_WRITE_ENV`](#gateway_proxy_write_env) | constant | Windows-only env var naming the gateway's overlapped write half of the proxy. |
| [`GATEWAY_RPC_FD`](#gateway_rpc_fd) | constant | Child fd the gateway uses for the guest RPC socket (`BOOKCLERK_GATEWAY_GUEST_RPC`). |
| [`GUEST_PROXY_FD`](#guest_proxy_fd) | constant | Child fd the native guest uses for the proxy mux (`BOOKCLERK_SOCKET_PROXY`). |
| [`JAIL_HANDOFF_ENV`](#jail_handoff_env) | constant | When `1`, `bookclerk-jail` reads one `JailHandoff` JSON line from stdin. |
| [`SOCKET_PROXY_ENV`](#socket_proxy_env) | constant | Env var naming the socket-proxy link for a native guest (SDK `net::connect`). |
| [`SOCKET_PROXY_WRITE_ENV`](#socket_proxy_write_env) | constant | Windows-only env var naming the guest's overlapped write half of the proxy. |
| [`TEST_EXTRA_ENDPOINT_FD`](#test_extra_endpoint_fd) | constant | Test-only extra endpoint in the native guest. Production preserve lists omit it. |
| [`WORKERD_STATE_DIR_ENV`](#workerd_state_dir_env) | constant | Host-chosen workerd session directory (`0700`, under `$TMPDIR`). |
| [`BACKEND`](#backend) | constant | Backend name reported in diagnostics. |
| [`SPAWN_DIAG_MAX_LINES`](#spawn_diag_max_lines) | constant | Maximum lines in that ring. The byte cap is the budget; this only stops a flood of tiny lines. |
| [`SPAWN_DIAG_RECORD_BYTES`](#spawn_diag_record_bytes) | constant | Maximum bytes stored for one diagnostic line, cut on a UTF-8 boundary. |
| [`SPAWN_DIAG_TOTAL_BYTES`](#spawn_diag_total_bytes) | constant | Maximum bytes retained in the process-wide diagnostic ring. |
| [`PLUGIN_FD_CHANNEL`](#plugin_fd_channel) | constant | Reserved descriptor number for a native SCM_RIGHTS side channel (fd 3). |
| [`PLUGIN_FD_CHANNEL_ENV`](#plugin_fd_channel_env) | constant | Environment variable naming `PLUGIN_FD_CHANNEL`. |
| [`SPEC_ENV`](#spec_env) | constant | Environment variable carrying the JSON `Spec` to `bookclerk-jail`. |
| [`GATEWAY_WORKER_THREADS`](#gateway_worker_threads) | constant | Tokio worker threads for `bookclerk-workerd`. |
| [`GATEWAY_MAX_BLOCKING_THREADS`](#gateway_max_blocking_threads) | constant | Blocking-pool ceiling for `bookclerk-workerd`. |
| [`INFRASTRUCTURE_THREAD_BUDGET`](#infrastructure_thread_budget) | constant | Threads reserved for one native-behind-workerd session before `extraProcesses`. |

## Modules

- [`spawn`](spawn.md): Windows AppContainer spawn API (`plan_appcontainer`, `run_appcontainer`).

## Structs

### CgroupSample

```rust
pub struct CgroupSample {
    pub cpu_max_quota_us: Option<u64>,
    pub cpu_max_period_us: Option<u64>,
    pub memory_max_bytes: Option<u64>,
    pub memory_current_bytes: Option<u64>,
    pub memory_anon_bytes: Option<u64>,
}
```

Parsed cgroup v2 files for the process that is actually running.

Unlimited (`max`) and missing files are `None`.

**Fields**

- `cpu_max_quota_us`: `cpu.max` quota in microseconds. Empty when the file is missing or `max`.
- `cpu_max_period_us`: `cpu.max` period in microseconds. Empty when the file is missing or `max`.
- `memory_max_bytes`: `memory.max` in bytes. Empty when the file is missing or `max`.
- `memory_current_bytes`: `memory.current` in bytes.
- `memory_anon_bytes`: `memory.stat` field `anon`.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### SystemMemory

```rust
pub struct SystemMemory {
    pub total_bytes: Option<u64>,
    pub available_bytes: Option<u64>,
}
```

Whole-machine memory figures for heartbeats.

Linux reads `/proc/meminfo`, macOS `hw.memsize` and the Mach VM
statistics, Windows `GlobalMemoryStatusEx`. Other platforms report nothing
and the scheduler falls back to the detected CPU capacity alone.

**Fields**

- `total_bytes`: Physical memory in bytes (`MemTotal`, `hw.memsize`, `ullTotalPhys`).
- `available_bytes`: Bytes new work can use without swapping (`MemAvailable`; free plus
  inactive pages on macOS; `ullAvailPhys` on Windows).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### DuplexHalf

```rust
pub struct DuplexHalf {
    /* private fields */
}
```

One unidirectional pipe end (stdio).

#### DuplexHalf::as_raw_fd

```rust
pub fn as_raw_fd(&self) -> RawFd
```

Unix raw fd.

#### DuplexHalf::into_owned_fd

```rust
pub fn into_owned_fd(self) -> OwnedFd
```

Consume as [`std::os::fd::OwnedFd`](https://doc.rust-lang.org/stable/std/os/fd/owned/struct.OwnedFd.html).

Implements: `Debug`

### DuplexLink

```rust
pub struct DuplexLink {
    /* private fields */
}
```

One end of a host-created duplex IPC link (socketpair / overlapped pipe).

#### DuplexLink::pair

```rust
pub fn pair() -> Result<(Self, Self)>
```

Create a connected duplex pair. Both ends stay non-inheritable / `CLOEXEC`.

##### Errors

Returns an I/O error when the kernel cannot allocate the pair.

#### DuplexLink::try_clone

```rust
pub fn try_clone(&self) -> Result<Self>
```

Duplicate this end. Both copies stay `CLOEXEC` / non-inheritable.

Used so guest stdin and stdout can share one duplex without racing
`dup` in the caller.

##### Errors

Returns an I/O error when the kernel cannot duplicate the descriptor.

#### DuplexLink::as_raw_fd

```rust
pub fn as_raw_fd(&self) -> RawFd
```

Unix raw fd (still owned by `self`).

#### DuplexLink::into_owned_fd

```rust
pub fn into_owned_fd(self) -> OwnedFd
```

Consume this end as an [`std::os::fd::OwnedFd`](https://doc.rust-lang.org/stable/std/os/fd/owned/struct.OwnedFd.html).

Implements: `Debug`

### JailHandoff

```rust
pub struct JailHandoff {
    pub v: u32,
    pub stdin: Option<u64>,
    pub stdout: Option<u64>,
    pub extra: Vec<JailHandoffExtra>,
}
```

One-shot Windows jail stdin line. Bounded JSON; never a general broker.

The host derives every handle from values it just duplicated with
`duplicate_handle_into` (Windows). The jail is the sole spawner and marks
the inherited entries inheritable immediately before `CreateProcess`.
Entries with `inherit: false` are named in the child environment and are
not placed on the inherit list.

**Fields**

- `v`: Wire version. Only `1` is accepted.
- `stdin`: Guest stdin handle (RPC read end), when the jail should not create a pipe.
- `stdout`: Guest stdout handle (RPC write end), when the jail should not create a pipe.
- `extra`: Extra handles exported as `env=handle:<value>`.
  
  [`JailHandoffExtra::inherit`](#jailhandoffextra) decides which of these are placed on
  `PROC_THREAD_ATTRIBUTE_HANDLE_LIST`. The others are only named in the
  child environment.

#### JailHandoff::inherited_extra_handles

```rust
pub fn inherited_extra_handles(&self) -> Vec<u64>
```

Handle values `CreateProcess` should list.

`inherit: false` entries stay in this process. They are still exported
as environment variables by the jail.

#### JailHandoff::VERSION

```rust
pub const VERSION: u32 = 1
```

Current wire version.

#### JailHandoff::MAX_LINE_BYTES

```rust
pub const MAX_LINE_BYTES: usize = _
```

Maximum accepted JSON line length (8 KiB). Larger input is a protocol error.

#### JailHandoff::to_line

```rust
pub fn to_line(&self) -> Result<String>
```

Encode as a single JSON line (no trailing newline; the caller writes `\n`).

##### Errors

Returns [`std::io::Error`](https://doc.rust-lang.org/stable/std/io/error/struct.Error.html) when serde cannot encode the handoff.

#### JailHandoff::from_line

```rust
pub fn from_line(line: &str) -> Result<Self>
```

Parse one JSON line. Rejects `v != 1` and oversize input.

##### Errors

Returns [`std::io::Error`](https://doc.rust-lang.org/stable/std/io/error/struct.Error.html) when the line is too long, not JSON, or
the version is not [`Self::VERSION`](#jailhandoffversion).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JailHandoffExtra

```rust
pub struct JailHandoffExtra {
    pub env: String,
    pub handle: u64,
    pub inherit: bool,
}
```

One extra handle named in the child environment.

**Fields**

- `env`: Environment variable the child should see (`BOOKCLERK_SOCKET_PROXY`, …).
- `handle`: Handle value in the jail process (already duplicated).
- `inherit`: When false, the child learns `handle:<value>` and the handle stays off
  `PROC_THREAD_ATTRIBUTE_HANDLE_LIST`.

#### JailHandoffExtra::inherited

```rust
pub fn inherited(env: impl Into<String>, handle: u64) -> Self
```

Inherited handle. The child receives the object and `env=handle:<value>`.

#### JailHandoffExtra::named_only

```rust
pub fn named_only(env: impl Into<String>, handle: u64) -> Self
```

Name the handle in the child environment without inheriting the object.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### StdioEnds

```rust
pub struct StdioEnds {
    pub host_stdin: DuplexHalf,
    pub host_stdout: DuplexHalf,
    pub guest_stdin: DuplexHalf,
    pub guest_stdout: DuplexHalf,
}
```

Host and guest ends of a pair of unidirectional stdio pipes (RPC).

**Fields**

- `host_stdin`: Parent writes this into the child's stdin.
- `host_stdout`: Parent reads this from the child's stdout.
- `guest_stdin`: Child stdin read end.
- `guest_stdout`: Child stdout write end.

#### StdioEnds::pair

```rust
pub fn pair() -> Result<Self>
```

Two unidirectional pipes: host stdin-write / stdout-read, guest the opposite.

Windows: host ends are overlapped; guest ends are synchronous (Rust std
`Command` convention). All DACLs are owner-only.

##### Errors

Returns an I/O error when a pipe cannot be created.

Implements: `Debug`

### Spec

```rust
pub struct Spec {
    pub label: String,
    pub reads: Vec<PathBuf>,
    pub writes: Vec<PathBuf>,
    pub net: NetPolicy,
    pub allow_exec: bool,
    pub system_paths: bool,
    pub enforcement: Enforcement,
    pub preserve_fds: Vec<i32>,
    pub windows_profile_name: Option<String>,
    pub memory_bytes: Option<u64>,
    pub active_processes: Option<u32>,
    pub cpu_rate_percent: CpuRate,
    pub inherit_handles: Vec<u64>,
    pub cgroup_dir: Option<PathBuf>,
    pub unix_socket_dirs: Option<Vec<PathBuf>>,
}
```

A serializable confinement request.

Mirrors [`Policy`](#policy), which is a builder and deliberately does not expose its
unresolved fields. Paths are resolved by the launcher, in the launcher's own
view of the filesystem, rather than being canonicalized here — the host and
the jailed process must agree on physical paths, and only the process that
applies the policy can confirm they do.

**Fields**

- `label`: Diagnostics label, echoed in the launcher's report.
- `reads`: Paths the confined process may read.
- `writes`: Paths the confined process may read and write.
- `net`: Network reachability granted to the confined process.
- `allow_exec`: Whether `execve` is permitted.
  
  A launcher that has to `exec` the real program cannot deny this to
  itself. What it can rely on is that the restrictions are inherited and
  irreversible, so whatever the guest goes on to exec is confined the same
  way, and `no_new_privs` has already neutralized setuid.
- `system_paths`: Whether to include the platform's read-only system set.
- `enforcement`: What to do when a layer does not engage.
- `preserve_fds`: Descriptors the host wired up deliberately and the launcher must not
  close before the hand-off — typically a Unix socket for passing a fetch
  directory one RPC at a time.
- `windows_profile_name`: Pre-created AppContainer profile moniker (Windows).
  
  When set, `bookclerk-jail` attaches to this host-owned profile and does
  not delete it. When absent, the jail creates a unique per-launch profile
  and deletes it after the guest (and its Job Object) exits.
- `memory_bytes`: Soft memory ceiling in bytes (Job Object / cgroup v2 `memory.max`).
  
  `None` keeps platform defaults (Windows label heuristics; Linux applies
  no cgroup memory limit).
- `active_processes`: Cap on concurrent processes in the jail (Job active-process / `pids.max`).
- `cpu_rate_percent`: CPU hard-cap as a percent of **one logical CPU** (`1..=` host cores×100).
  
  Linux: cgroup v2 `cpu.max` quota for a 100 ms period (`quota = period ×
  percent / 100`; values above 100 request multiple cores). Windows: Job
  Object hard cap, scaled by logical CPU count so the same percent means
  one-core bandwidth (see [`crate::windows_job_cpu_rate`](#windows_job_cpu_rate)). macOS Seatbelt
  cannot enforce this (see docs).
  
  Omitted and `null` are unspecified (the label default). A number is a
  percent. `"off"` disables the rate so a nested Job is not filled with
  the plugin default.
- `inherit_handles`: Windows handles the jail must put on the child's inherit list.
  
  The host never marks these inheritable itself. It duplicates them into
  `bookclerk-jail` with `duplicate_handle_into` and names them here / in
  the stdin handoff line so the jail can add them to
  `PROC_THREAD_ATTRIBUTE_HANDLE_LIST`.
- `cgroup_dir`: Linux cgroup v2 leaf both sibling jails join (session aggregate limits).
  
  When set, the jail moves into this directory instead of creating
  `bookclerk-<pid>`. The host writes the ceilings; the jail must not
  overwrite them with a per-process budget.
- `unix_socket_dirs`: macOS Seatbelt pathname Unix-socket directories.
  
  `None` (default, omitted on the wire) keeps the historical rule: bind
  and connect under every writable path. `Some(dirs)` allows UDS only in
  those directories (`Some([])` grants none). The native-behind-workerd
  gateway sets the host-chosen session directory. `build_spec_with_grant`
  starts the Deny guest at `Some([])`; the host then replaces that with
  the guest's private IPC directory.

#### Spec::new

```rust
pub fn new(label: impl Into<String>) -> Self
```

Start a spec with nothing granted.

#### Spec::policy

```rust
pub fn policy(&self) -> Policy
```

Build the [`Policy`](#policy) this spec describes.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Policy

```rust
pub struct Policy {
    /* private fields */
}
```

A confinement request: an allowlist of paths plus a few coarse switches.

Paths that do not exist when the policy is applied are skipped, because a
Landlock rule on a missing path is an error rather than a wider grant.

#### Policy::new

```rust
pub fn new(label: impl Into<String>) -> Self
```

Start a deny-default policy. `label` appears in diagnostics only.

#### Policy::read

```rust
pub fn read(self, path: impl Into<PathBuf>) -> Self
```

Grant read access to `path` and everything beneath it.

#### Policy::write

```rust
pub fn write(self, path: impl Into<PathBuf>) -> Self
```

Grant read and write access to `path` and everything beneath it.

#### Policy::reads

```rust
pub fn reads<I, P>(self, paths: I) -> Self
where
    I: IntoIterator<Item = P>,
    P: Into<PathBuf>,
```

Grant read access to each of `paths`.

#### Policy::writes

```rust
pub fn writes<I, P>(self, paths: I) -> Self
where
    I: IntoIterator<Item = P>,
    P: Into<PathBuf>,
```

Grant read and write access to each of `paths`.

#### Policy::net

```rust
pub fn net(self, net: NetPolicy) -> Self
```

Set network reachability. Defaults to [`NetPolicy::Deny`](#netpolicy).

#### Policy::allow_exec

```rust
pub fn allow_exec(self, allow: bool) -> Self
```

Allow `execve` of binaries inside the read allowlist. Defaults to false.

#### Policy::system_paths

```rust
pub fn system_paths(self, include: bool) -> Self
```

Include the platform's read-only system set (shared libraries, the CA
bundle, resolver configuration). Defaults to true; a worker that needs
nothing but its own scratch directory can turn it off.

#### Policy::enforcement

```rust
pub fn enforcement(self, enforcement: Enforcement) -> Self
```

Set the failure mode. Defaults to [`Enforcement::Required`](#enforcement).

#### Policy::memory_bytes

```rust
pub fn memory_bytes(self, bytes: Option<u64>) -> Self
```

Soft memory ceiling in bytes (`None` = platform default / unset).

#### Policy::active_processes

```rust
pub fn active_processes(self, n: Option<u32>) -> Self
```

Cap on concurrent processes (`None` = platform default / unset).

#### Policy::cpu_rate

```rust
pub fn cpu_rate(self, rate: CpuRate) -> Self
```

CPU hard-cap. [`CpuRate::Unspecified`](#cpurate) takes the label default;
[`CpuRate::Disabled`](#cpurate) stays off. A percent is clamped to
`1..=`[`host_cpu_rate_max`](#host_cpu_rate_max).

#### Policy::cpu_rate_percent

```rust
pub fn cpu_rate_percent(self, percent: Option<u32>) -> Self
```

CPU hard-cap percent of one logical CPU.

`None` is unspecified (the label default), not disabled. Use
[`Self::cpu_rate`](#policycpu_rate)`(`[`CpuRate::Disabled`](#cpurate)`)` to keep a nested Job from
inheriting that default. A percent is clamped to `1..=`[`host_cpu_rate_max`](#host_cpu_rate_max).

#### Policy::unix_socket_dirs

```rust
pub fn unix_socket_dirs(self, dirs: Option<Vec<PathBuf>>) -> Self
```

Restrict pathname Unix sockets to `dirs` (`Some([])` denies them).

`None` (the default) keeps the historical Seatbelt rule: UDS under every
writable path. Hosts that want an explicit list — the native-behind-workerd
gateway session directory, or no UDS for a Deny guest — set this.

#### Policy::cgroup_dir

```rust
pub fn cgroup_dir(self, dir: Option<PathBuf>) -> Self
```

Join this cgroup v2 leaf instead of creating `bookclerk-<pid>`.

#### Policy::unix_socket_dirs_opt

```rust
pub fn unix_socket_dirs_opt(&self) -> Option<&[PathBuf]>
```

Explicit Unix-socket directories, or `None` for the writable-path default.

#### Policy::cgroup_dir_opt

```rust
pub fn cgroup_dir_opt(&self) -> Option<&Path>
```

Optional session cgroup to join (Linux).

#### Policy::label

```rust
pub fn label(&self) -> &str
```

Diagnostics label supplied to [`Self::new`](#policynew) (logs / doctor output only).

#### Policy::resource_limits

```rust
pub fn resource_limits(&self) -> ResourceLimits
```

Spec-provided resource ceilings (before label heuristics).

#### Policy::has_resource_limits

```rust
pub fn has_resource_limits(&self) -> bool
```

Whether the Spec/Policy asked for any OS resource ceiling.

#### Policy::resolved_job_limits

```rust
pub fn resolved_job_limits(&self) -> ResourceLimits
```

Job Object limits: Spec fields override [`label_resource_defaults`](#label_resource_defaults).

Always returns concrete ceilings suitable for Windows Jobs. Linux only
applies cgroup limits when [`Self::has_resource_limits`](#policyhas_resource_limits) is true.

#### Policy::resolved_reads

```rust
pub fn resolved_reads(&self) -> Vec<PathBuf>
```

Read allowlist, including the platform system set when enabled.

Missing paths are filtered out and the rest are resolved to their
physical (canonical) location.

#### Policy::resolved_writes

```rust
pub fn resolved_writes(&self) -> Vec<PathBuf>
```

Write allowlist, including the few system paths that have to be writable
when the system set is enabled.

Missing paths are filtered out and the rest are resolved to their
physical (canonical) location.

#### Policy::net_policy

```rust
pub fn net_policy(&self) -> NetPolicy
```

Network reachability granted by this policy.

#### Policy::exec_allowed

```rust
pub fn exec_allowed(&self) -> bool
```

Whether `execve` is permitted.

#### Policy::enforcement_mode

```rust
pub fn enforcement_mode(&self) -> Enforcement
```

What to do when a requested layer cannot engage.

#### Policy::confine_current_process

```rust
pub fn confine_current_process(&self) -> Result<Report, SandboxError>
```

Confine the calling process. Restrictions are irreversible and are
inherited by children, which can only narrow them further.

Call this at the top of `main`, before spawning threads or a runtime.
It allocates, so it is *not* async-signal-safe and must not be used from
`Command::pre_exec` in a threaded parent. Child processes we control
confine themselves at startup instead, which closes the window between
`exec` and the jail taking effect.

##### Errors

Returns [`SandboxError::NotEnforced`](#sandboxerror) when [`Enforcement::Required`](#enforcement) was
requested and a layer did not engage, and [`SandboxError::Backend`](#sandboxerror) when
a backend call fails outright.

Implements: `Clone`, `Debug`

### ResourceLimits

```rust
pub struct ResourceLimits {
    pub memory_bytes: Option<u64>,
    pub cpu_rate_percent: Option<u32>,
    pub active_processes: Option<u32>,
}
```

Optional OS resource ceilings carried on a [`Policy`](#policy) / [`Spec`](#spec).

Windows applies these (merged with label heuristics) via a Job Object.
Linux applies them best-effort via cgroup v2 when at least one field is set.
macOS Seatbelt has no equivalent — see [`Policy::has_resource_limits`](#policyhas_resource_limits).

**Fields**

- `memory_bytes`: Process memory ceiling in bytes.
- `cpu_rate_percent`: CPU hard-cap as a percent of **one logical CPU** (1..=host cores×100).
  
  Values above 100 request multi-core bandwidth (200 ≈ two cores). Linux
  writes cgroup v2 `cpu.max` as `period * percent / 100`. Windows Job
  `CpuRate` is a share of **all** processors, so [`windows_job_cpu_rate`](#windows_job_cpu_rate)
  scales by logical CPU count. Threads share one quota pool.
- `active_processes`: Maximum concurrent processes in the jail.

#### ResourceLimits::is_empty

```rust
pub fn is_empty(&self) -> bool
```

Whether any limit was requested.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### Report

```rust
pub struct Report {
    pub label: String,
    pub backend: &'static str,
    pub filesystem: LayerStatus,
    pub syscall: LayerStatus,
    pub network: LayerStatus,
    pub resources: LayerStatus,
}
```

What actually engaged when a [`Policy`](#policy) was applied.

**Fields**

- `label`: The policy's diagnostics label.
- `backend`: Backend name, e.g. `landlock+seccomp`.
- `filesystem`: Filesystem allowlist (Landlock / Seatbelt / AppContainer) status.
- `syscall`: Syscall restriction (seccomp / Seatbelt operation classes) status.
- `network`: Network restriction status for the requested [`NetPolicy`](#netpolicy).
- `resources`: Memory / CPU / process ceilings (Job Object / cgroup v2).

#### Report::is_confined

```rust
pub fn is_confined(&self) -> bool
```

Whether the filesystem allowlist — the layer that protects `master.key`
and `library.db` — is active.

#### Report::summary

```rust
pub fn summary(&self) -> String
```

One-line summary for startup logs and `bookclerk doctor`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Capabilities

```rust
pub struct Capabilities {
    pub backend: &'static str,
    pub filesystem: bool,
    pub spawn_filesystem: bool,
    pub syscall: bool,
    pub network: bool,
    pub detail: String,
}
```

What this host can enforce, without applying anything.

**Fields**

- `backend`: Backend name for this platform (e.g. `landlock+seccomp`).
- `filesystem`: Whether a process can confine *itself* (Landlock / Seatbelt).
  
  False on Windows: isolation is granted only at `CreateProcess`. Media
  workers that self-confine check this field.
- `spawn_filesystem`: Whether a child can be confined at spawn (AppContainer).
  
  True on Windows once AppContainer launch is wired. Plugin jails that
  start guests through `bookclerk-jail` check this (or [`Self::filesystem`](#capabilities)).
- `syscall`: Whether syscall filtering (seccomp / equivalent) is available.
- `network`: Whether network restriction can be applied on this host.
- `detail`: Human-readable detail, e.g. the Landlock ABI level found.

#### Capabilities::can_confine_guest

```rust
pub fn can_confine_guest(&self) -> bool
```

Whether this host can confine a guest somehow (self-confine or spawn-time).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### CpuTimes

```rust
pub enum CpuTimes {
    System { busy: u64, total: u64 },
    Cgroup { usage_usec: u64 },
}
```

Cumulative CPU time counters. Two samples give utilization.

[`CpuTimes::System`](#cputimes) units differ by platform (jiffies on Linux, Mach
ticks on macOS, 100 ns on Windows); only the busy/total ratio between two
samples of the same host is used, so the unit cancels out.

**Variants**

- `System`: Whole-machine busy and total time across all CPUs: the aggregate
  `/proc/stat` line on Linux, `HOST_CPU_LOAD_INFO` on macOS,
  `GetSystemTimes` on Windows.
  - `busy`: Time spent not idle (Linux: `user + nice + system + irq + softirq + steal`).
  - `total`: All time, idle (and Linux iowait) included.
- `Cgroup`: cgroup v2 `cpu.stat` `usage_usec` of a CPU-limited cgroup.
  - `usage_usec`: Microseconds of CPU the cgroup used since it was created.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### LinkSpec

```rust
pub enum LinkSpec {
    Fd(i32),
    Handle(u64),
}
```

How a confined child names an inherited duplex or pipe end.

**Variants**

- `Fd`: Unix inherited file descriptor (`fd:3`).
- `Handle`: Windows inherited handle value (`handle:18446744073709551615`).

#### LinkSpec::parse

```rust
pub fn parse(spec: &str) -> Result<Self, LinkSpecError>
```

Parse `fd:<n>` or `handle:<n>`.

##### Errors

Returns [`LinkSpecError`](#linkspecerror) when the prefix is missing or the number is
not a decimal `i32` / `u64`.

#### LinkSpec::fd

```rust
pub fn fd(n: i32) -> Self
```

`fd:<n>` form used on Unix.

#### LinkSpec::handle

```rust
pub fn handle(n: u64) -> Self
```

`handle:<n>` form used on Windows.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `FromStr`, `PartialEq`

### LinkSpecError

```rust
pub enum LinkSpecError {
    Unrecognized(String),
    BadNumber(String),
}
```

Why a [`LinkSpec`](#linkspec) string could not be parsed.

**Variants**

- `Unrecognized`: Missing `fd:` / `handle:` prefix or empty remainder.
- `BadNumber`: Prefix present but the number did not parse.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### SpawnPathError

```rust
pub enum SpawnPathError {
    Empty,
    Nul,
    NotAbsolute(PathBuf),
    NotFile(PathBuf),
    OutsideRoot { path: PathBuf, root: PathBuf },
    Canonicalize { path: PathBuf, source: Error },
}
```

Why a spawn path was rejected.

**Variants**

- `Empty`: Empty `OsStr` / path.
- `Nul`: Interior NUL byte (or wide NUL on Windows).
- `NotAbsolute`: Relative path with separators (not a single PATH lookup name).
- `NotFile`: Absolute path that is not a regular file when one is required.
- `OutsideRoot`: Canonical path escapes the trusted root.
  - `path`: Candidate path after normalization.
  - `root`: Trusted directory the path must stay under.
- `Canonicalize`: `canonicalize` failed for an existing path that must resolve.
  - `path`: Path that failed to resolve.
  - `source`: Underlying I/O error.

Implements: `Debug`, `Display`, `Error`

### Enforcement

```rust
pub enum Enforcement {
    Required,
    BestEffort,
    Disabled,
}
```

What to do when a confinement layer cannot be enforced.

**Variants**

- `Required`: Fail the call when any requested layer does not engage.
- `BestEffort`: Apply what the host supports and report the rest as unsupported.
- `Disabled`: Apply nothing. Used only by the documented opt-out.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NetPolicy

```rust
pub enum NetPolicy {
    Deny,
    Outbound,
    OutboundListen,
    Full,
}
```

Network reachability granted to the confined process.

**Variants**

- `Deny`: No IP sockets at all. Media workers only touch local files.
- `Outbound`: Outbound connections allowed, inbound listeners refused. Enough for a
  storefront plugin that only fetches over HTTPS.
- `OutboundListen`: Outbound connections plus a listener on a kernel-assigned port.
  
  This exists for one flow: an OAuth login that sends the authorization
  code back to a short-lived local callback server. The grant is narrower
  than [`Full`](#netpolicy) in that no fixed port can be claimed, so a
  confined process cannot stand up a service on a port anything else would
  know to connect to.
  
  What "kernel-assigned" buys differs by backend: Landlock rules are
  per-port, so Linux allows binds within `ip_local_port_range` and refuses
  every fixed port, while Seatbelt filters by address and restricts the
  listener to loopback. Neither confines it to both at once.
- `Full`: Unrestricted. The daemon binds its own control-plane listener.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CpuRate

```rust
pub enum CpuRate {
    Unspecified,
    Disabled,
    Percent(u32),
}
```

CPU hard-cap carried on a [`Spec`](#spec) before label defaults are applied.

Omitted and JSON `null` are [`Self::Unspecified`](#cpurate) (the label default). A
JSON number is [`Self::Percent`](#cpurate). [`Self::Disabled`](#cpurate) is the string `"off"`
and must not be filled from the label: sibling inner Jobs use it so they
do not pick up the plugin default of 80 under an outer cap.

**Variants**

- `Unspecified`: Use [`label_resource_defaults`](#label_resource_defaults) when resolving a Job.
- `Disabled`: Do not apply a CPU rate, and do not substitute the label default.
- `Percent`: Percent of one logical CPU, clamped to `1..=`[`host_cpu_rate_max`](#host_cpu_rate_max) when
  the policy is built.

#### CpuRate::percent

```rust
pub fn percent(self) -> Option<u32>
```

Percent when this is [`Self::Percent`](#cpurate). Unspecified and disabled are
both absent until [`Self::resolve`](#cpurateresolve).

#### CpuRate::resolve

```rust
pub fn resolve(self, label_default: Option<u32>) -> Option<u32>
```

Label default for unspecified, nothing for disabled, the percent otherwise.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LayerStatus

```rust
pub enum LayerStatus {
    Enforced,
    Partial(String),
    NotApplicable(String),
    Unsupported(String),
    NotRequested,
}
```

Outcome for one confinement layer.

**Variants**

- `Enforced`: The layer engaged fully.
- `Partial`: The layer engaged, but the host does not support every requested
  restriction. Carries what was lost.
- `NotApplicable`: This platform draws its boundaries differently and has no separate
  mechanism here, but the restrictions this layer stands for are carried
  by another layer in the same report. Nothing was lost.
  
  macOS is the case that motivates this: Seatbelt gates operation classes
  rather than syscall numbers, so there is no seccomp equivalent to
  report — while `(deny default)` in the profile already refuses `exec`
  and the network. Treating that as a failure would make
  [`Enforcement::Required`](#enforcement) impossible to satisfy on macOS.
  
  A backend must not use this to paper over a restriction that genuinely
  is not in effect; that is [`Unsupported`](#layerstatus).
- `Unsupported`: The host has no mechanism for this layer and the restriction is *not*
  in effect. Fails [`Enforcement::Required`](#enforcement).
- `NotRequested`: The policy did not ask for this layer.

#### LayerStatus::is_active

```rust
pub fn is_active(&self) -> bool
```

Whether this layer is itself providing protection.

False for [`NotApplicable`](#layerstatus): the protection exists,
but a different layer in the report is the one supplying it.

#### LayerStatus::is_gap

```rust
pub fn is_gap(&self) -> bool
```

Whether this status means a requested restriction is missing.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

### SandboxError

```rust
pub enum SandboxError {
    NotEnforced { label: String, layer: &'static str, detail: String },
    Backend { label: String, backend: &'static str, detail: String },
}
```

Confinement failure when applying or requiring a [`Policy`](#policy).

**Variants**

- `NotEnforced`: A required layer did not engage under [`Enforcement::Required`](#enforcement).
  - `label`: Diagnostics label from the policy that failed.
  - `layer`: Layer name (`filesystem`, `syscall`, or `network`).
  - `detail`: Why the layer did not engage.
- `Backend`: A backend call failed outright (kernel / API error).
  - `label`: Diagnostics label from the policy that failed.
  - `backend`: Backend name that returned the error.
  - `detail`: Underlying backend error text.

Implements: `Debug`, `Display`, `Error`

## Functions

### cgroup_sample_is_low_resource

```rust
pub fn cgroup_sample_is_low_resource(sample: &CgroupSample) -> bool
```

True when `memory.max` is at most 1 GiB or `cpu.max` is at most one core.

Missing files and the unlimited token `max` are not a low-resource signal.

### cpu_used_millicores

```rust
pub fn cpu_used_millicores(
    before: CpuTimes,
    after: CpuTimes,
    cores_millis: u64,
    elapsed: Duration,
) -> Option<u64>
```

CPU in use between two samples, in millicores (1000 = one core busy).

`cores_millis` scales the machine-wide busy fraction; `elapsed` is the
wall time between a cgroup's two samples. Returns `None` when the
samples are of different kinds or did not advance.

### filesystem_free_bytes

```rust
pub fn filesystem_free_bytes(path: &Path) -> Option<u64>
```

Free bytes available to unprivileged callers.

`statvfs` `f_bavail` on Linux and macOS; `GetDiskFreeSpaceExW` (free bytes
available to the caller, so quotas count) on Windows. Returns `None`
when the path cannot be stated or on other platforms.

### logical_cpu_count

```rust
pub fn logical_cpu_count() -> Option<i64>
```

Logical CPUs from [`std::thread::available_parallelism`](https://doc.rust-lang.org/stable/std/thread/fn.available_parallelism.html), or empty on failure.

### macos_cpu_ticks

```rust
pub fn macos_cpu_ticks(user: u32, system: u32, idle: u32, nice: u32) -> CpuTimes
```

macOS `HOST_CPU_LOAD_INFO` ticks (`user`, `system`, `idle`, `nice`,
summed over all CPUs) as [`CpuTimes::System`](#cputimes).

The kernel counters are 32-bit and wrap; a wrapped pair makes
[`cpu_used_millicores`](#cpu_used_millicores) skip that one sample rather than report nonsense.

### macos_vm_memory

```rust
pub fn macos_vm_memory(
    total_bytes: Option<u64>,
    free_pages: u64,
    inactive_pages: u64,
    page_size: u64,
) -> SystemMemory
```

macOS available memory from `HOST_VM_INFO64` page counts.

Available is `free + inactive` pages (the figure `psutil` reports): pages
the kernel hands to new work without paging anything out first. The
result is clamped to `total_bytes`. A zero page size leaves it empty.

### parse_cpu_stat_usage

```rust
pub fn parse_cpu_stat_usage(text: &str) -> Option<u64>
```

`usage_usec` from cgroup v2 `cpu.stat` text.

### parse_meminfo

```rust
pub fn parse_meminfo(text: &str) -> SystemMemory
```

Parses `/proc/meminfo` text (values are in kB).

### parse_proc_stat

```rust
pub fn parse_proc_stat(text: &str) -> Option<CpuTimes>
```

Aggregate `cpu` line of `/proc/stat` text.

### process_cgroup_dir

```rust
pub fn process_cgroup_dir() -> Option<PathBuf>
```

Directory of this process's cgroup v2 node (`/proc/self/cgroup` `0::` path).

Returns `None` when the file is missing, has no v2 entry, or the
corresponding `/sys/fs/cgroup` directory does not exist.

### read_cgroup_sample

```rust
pub fn read_cgroup_sample(dir: &Path) -> CgroupSample
```

Reads `cpu.max`, `memory.max`, `memory.current`, and `memory.stat` in `dir`.

Effective `memory.max` is the minimum ceiling among `dir` and ancestors
that still publish `memory.max` or `cpu.max`. Effective `cpu.max` is the
tighter quota/period ratio on that same walk. `memory.current` and `anon`
stay the leaf values. The walk stops when a parent has neither controller
file, so a temporary test directory does not escape into `/tmp`.

Does not create `dir` or any controller file.

### read_cpu_times

```rust
pub fn read_cpu_times(
    cgroup_dir: Option<&Path>,
    cgroup: &CgroupSample,
) -> Option<CpuTimes>
```

Reads CPU time counters for utilization: the cgroup's own usage when the
process runs under a CPU-limited cgroup (a container quota is the real
ceiling), else the whole machine (a desktop's other applications count
against what Bookclerk can use). The whole-machine counters come from
`/proc/stat` on Linux, `HOST_CPU_LOAD_INFO` on macOS, and
`GetSystemTimes` on Windows.

### read_system_memory

```rust
pub fn read_system_memory() -> SystemMemory
```

Reads total and available physical memory for this machine.

Linux: `MemTotal` / `MemAvailable` from `/proc/meminfo`. macOS:
`hw.memsize` and `HOST_VM_INFO64` (free plus inactive pages, see
[`macos_vm_memory`](#macos_vm_memory)). Windows: `GlobalMemoryStatusEx`. Fields stay empty
when the platform call fails or on other platforms.

### windows_system_times

```rust
pub fn windows_system_times(idle: u64, kernel: u64, user: u64) -> CpuTimes
```

Windows `GetSystemTimes` values (100 ns units) as [`CpuTimes::System`](#cputimes).

`kernel` already includes `idle`, so busy is `kernel + user - idle` and
total is `kernel + user`.

### create_guest_ipc_dir

```rust
pub fn create_guest_ipc_dir() -> Result<PathBuf, String>
```

Create a mode `0700` directory under `/tmp` whose longest guest socket fits
in a macOS `sockaddr_un`.

The returned path is canonical, so a `/tmp` symlink (`/private/tmp` on
macOS) is what the jail profile and the guest both see.

#### Errors

Returns when `/tmp` cannot hold the directory, or when the canonical path
plus `.s.PGSQL.65535` does not fit. The error includes the path length and
the directory that was too long.

### ensure_guest_ipc_fits

```rust
pub fn ensure_guest_ipc_fits(dir: &Path) -> Result<(), String>
```

Reject `dir` when `{dir}/.s.PGSQL.65535` does not fit in a macOS `sockaddr_un`.

#### Errors

The message includes the socket path length and `dir`.

### with_fd_spawn_lock

```rust
pub fn with_fd_spawn_lock<R>(f: impl FnOnce() -> R) -> R
```

Serialize macOS descriptor allocation with in-process `Command` spawns.

macOS has no `SOCK_CLOEXEC` or `pipe2`. Socket and pipe creation in this
crate holds the lock from allocation until `FD_CLOEXEC` is set. Every
`Command::spawn` in that process must hold it for the `spawn` call, or a
concurrent `fork` inherits the new descriptor. Linux `SOCK_CLOEXEC` /
`pipe2` are atomic, so this is a no-op there. Do not create a link while
already inside the lock: the mutex is not reentrant.

### inherit_fd_at

```rust
pub fn inherit_fd_at(src: RawFd, dest: RawFd) -> Result<()>
```

`dup2` `src` onto `dest` and clear `FD_CLOEXEC` on the destination.

Call from `Command::pre_exec` only. `src` stays `CLOEXEC` in the parent.
The child must close `src` when it differs from `dest` so the pre-dup
descriptor does not survive `exec`. After `dup2` the child sees `dest`
without `CLOEXEC`; the guest sets `FD_CLOEXEC` again when it adopts that
descriptor.

#### Errors

Returns an I/O error when `dup2` or `fcntl` fails.

#### Safety

Must run in the child after `fork` and before `exec`. `src` must be open.

### create_session_cgroup

```rust
pub fn create_session_cgroup(
    limits: &ResourceLimits,
    suffix: &str,
) -> Result<PathBuf, String>
```

Create an exclusive session cgroup leaf and write `limits` without moving the caller.

The host assigns both sibling jails via [`crate::Spec::cgroup_dir`](#spec). The leaf
name is the suffix the caller supplies; `create_dir` fails when that name
already exists so two sessions never share a leaf. `pids.max` is the thread
budget in `limits.active_processes` (not the Windows process baseline).

The leaf stays inside this process's current cgroup. Bookclerk does not
walk to an ancestor or enable controllers there. A populated cgroup gets a
`bookclerk-host` child for this process and its descendants; the session
leaf is created beside that child. A process that is not a descendant
blocks delegation. When this process is already in `bookclerk-host` and the
immediate parent delegates `pids`, the leaf is a sibling.

Failure is best-effort (same posture as `try_apply_cgroup_v2`): callers treat
`Err` as not-applicable and fall back to process-group kill, which does not
cover a descendant that calls `setsid`. That is not fake enforcement.

#### Errors

Returns a string when the hierarchy is missing, the leaf already exists, a
child cannot be created, `pids` cannot be delegated, or a present limit file
cannot be written. A partial directory is removed when writing limits fails.

### destroy_session_cgroup

```rust
pub fn destroy_session_cgroup(dir: &Path) -> Result<(), String>
```

Kill every member of `dir` and remove the leaf.

Writes `cgroup.kill`, which the kernel applies to the whole leaf including
tasks that fork while the write runs, then waits until `cgroup.events`
reports `populated 0`. This is ownership of the leaf, not a snapshot of
numeric pids. A descendant that called `setsid` is still inside the leaf
and dies with it. Without a delegated cgroup, process-group kill is a
different path and does not own that descendant.

When `cgroup.kill` is absent, equivalent teardown is unsupported. Bookclerk
does not signal pids read from `cgroup.procs`.

A missing directory is success, so teardown is idempotent.

#### Errors

Returns a string when `cgroup.kill` is missing or cannot be written, the
leaf stays populated, or the empty directory cannot be removed. A missing
`cgroup.kill` says equivalent teardown is unsupported.

### recent_platform_spawn_diagnostics

```rust
pub fn recent_platform_spawn_diagnostics() -> String
```

Startup lines recorded by AppContainer profile, DACL, and proxy stages.

Byte-capped. Raw copies are not written to the daemon log unless
`BOOKCLERK_SPAWN_DIAG` is set. A jailed child's own copy stays on its
stderr pipe so the host forwarder can re-emit it as tracing.

### join_capped_lines

```rust
pub fn join_capped_lines(ring: &VecDeque<String>, total_bytes: usize) -> String
```

Join `ring` and cap the copy at `total_bytes`.

### push_capped_line

```rust
pub fn push_capped_line(
    ring: &mut VecDeque<String>,
    line: &str,
    record_bytes: usize,
    total_bytes: usize,
    max_lines: usize,
)
```

Push `line` onto `ring`, dropping the oldest entries until both caps hold.

### record_spawn_diagnostic

```rust
pub fn record_spawn_diagnostic(message: &str)
```

Remember one already-formatted line and emit it.

The ring and the tracing event are capped. Raw stderr follows
[`spawn_diag_raw_stderr`](#spawn_diag_raw_stderr).

### redact_capped

```rust
pub fn redact_capped(message: &str, max_bytes: usize) -> String
```

Redact secrets and keep at most `max_bytes` of the result.

A 64-hex challenge or `BOOKCLERK_SESSION_CHALLENGE` that starts inside the cap and
continues past it is classified with a bounded lookahead, then replaced.
No prefix of that token is emitted. Bytes after the cap are not copied.
The cut stays on a UTF-8 boundary.

### redact_diagnostic_text

```rust
pub fn redact_diagnostic_text(message: &str) -> String
```

Replace a 64-hex session challenge and the challenge env name.

Non-hex bytes are copied as Unicode scalar values. A multibyte character
is never widened by casting each byte to `char`. Diagnostic retention uses
[`redact_capped`](#redact_capped) so a token that crosses the byte cap is recognized before
any of its prefix is stored.

### snapshot_spawn_diagnostics

```rust
pub fn snapshot_spawn_diagnostics() -> String
```

Byte-capped snapshot of [`record_spawn_diagnostic`](#record_spawn_diagnostic) lines.

### spawn_diag_raw_stderr

```rust
pub fn spawn_diag_raw_stderr() -> bool
```

`true` when raw stage lines should be written to this process's stderr.

Diagnostic mode is explicit. A jailed child also prints, because its
stderr is the host's capture pipe rather than the daemon log. The host
forwarder re-emits those lines as tracing and does not treat them as a
second daemon sink.

### spawn_diag_stderr_enabled

```rust
pub fn spawn_diag_stderr_enabled() -> bool
```

`true` when `BOOKCLERK_SPAWN_DIAG` is `1`, `true`, or `stderr`.

### truncate_utf8

```rust
pub fn truncate_utf8(text: &str, max_bytes: usize) -> &str
```

Copy `text` up to `max_bytes` without splitting a scalar value.

A 4-byte cap of `café` is `caf`. The truncated text never reinterprets the
leftover bytes of `é` as Latin-1.

### canonicalize

```rust
pub fn canonicalize(path: &Path) -> Result<PathBuf>
```

[`std::fs::canonicalize`](https://doc.rust-lang.org/stable/std/fs/fn.canonicalize.html) that also works inside a Windows AppContainer.

On Windows, `canonicalize` asks the mount manager for the DOS drive letter
(`VOLUME_NAME_DOS`), which an AppContainer token may not query, so every
path fails with "Access is denied" even when the file is readable. The
fallback resolves the volume-relative final path from the open handle,
re-attaches the caller's drive, and accepts it only when it opens as the
very same file (volume serial + file index); otherwise the original error
stands. `GetFinalPathNameByHandleW` follows reparse points; a junction to
another volume yields a different identity and falls back to the original
error. Callers that later reopen the returned path by name (rather than by
the handle used here) still see the usual TOCTOU gap — this helper proves
identity at the moment of canonicalize, not at a later open.

#### Errors

Returns the [`std::fs::canonicalize`](https://doc.rust-lang.org/stable/std/fs/fn.canonicalize.html) error when the path cannot be
resolved (or, on Windows, cannot be proven identical by handle).

### create_process_path

```rust
pub fn create_process_path(path: &Path) -> PathBuf
```

Path passed to `CreateProcess`, `Command`, or a child that parses Win32 paths.

Windows canonical paths use the `\\?\` prefix. An AppContainer can open
that form for read and still be denied process creation
(`ERROR_ACCESS_DENIED`). Pinned `workerd` rejects the same prefix: its
filesystem parser treats `?` as the first component and aborts. The prefix
is removed only after callers have already canonicalized, so the Win32 path
names the same resolved file. Other platforms return `path` unchanged.

### require_absolute_or_name

```rust
pub fn require_absolute_or_name(path: &Path) -> Result<PathBuf, SpawnPathError>
```

Absolute path with no NUL, or a single PATH lookup name (`cargo`, `python3`).

Does not require the path to exist (useful for generated argv file operands).
Lexical `..` in an absolute spelling is allowed; callers that need a real
file should use [`require_spawn_executable`](#require_spawn_executable) (canonicalize).

Prefer not using this for operator-selected tools like `$CARGO` — those may
legitimately be relative (`./custom-cargo`). Use it when the product
contract is “absolute or PATH name” for a helper beside the host.

#### Errors

Returns [`SpawnPathError`](#spawnpatherror) when the path is empty, contains NUL, or is a
multi-component relative path.

### require_absolute_spawn_path

```rust
pub fn require_absolute_spawn_path(path: &Path) -> Result<PathBuf, SpawnPathError>
```

Like [`require_absolute_or_name`](#require_absolute_or_name), but the path must be absolute (no PATH names).

#### Errors

Returns [`SpawnPathError`](#spawnpatherror) when the path is empty, contains NUL, or is not absolute.

### require_existing_regular_file

```rust
pub fn require_existing_regular_file(path: &Path) -> Result<PathBuf, SpawnPathError>
```

Existing regular file after resolving relative paths against the process cwd.

Absolute and multi-component relative paths (`./worker`, `bin/tool`) are
accepted when they canonicalize to a regular file. A bare name is accepted
only when that name exists as a file under the current directory — never as
an ambient `PATH` lookup. Use this for media worker / jail path knobs that
historically required `path.is_file()`.

#### Errors

Returns [`SpawnPathError`](#spawnpatherror) when the path is empty/NUL, cannot be resolved, or
is not a regular file.

### require_helper_beside_or_absolute

```rust
pub fn require_helper_beside_or_absolute(
    path: &Path,
    helper_name: &str,
    beside: Option<&Path>,
) -> Result<PathBuf, SpawnPathError>
```

Accepts `path` when it equals `beside`'s sibling named `helper_name`, or is an
absolute existing file (env override).

Equality with the beside-host helper is the preferred product path. Absolute
env overrides still require [`require_spawn_executable`](#require_spawn_executable); that is an
explicit operator trust decision, not proof the binary is safe.

Bare PATH names (single component, non-absolute) are rejected for this API
unless they match the beside-host helper. Callers that intentionally want
PATH lookup must use [`require_absolute_or_name`](#require_absolute_or_name) / [`require_spawn_executable`](#require_spawn_executable)
directly.

#### Errors

Returns [`SpawnPathError`](#spawnpatherror) when validation fails.

### require_spawn_executable

```rust
pub fn require_spawn_executable(path: &Path) -> Result<PathBuf, SpawnPathError>
```

Absolute existing regular file (or a single PATH name), with no interior NUL.

Absolute paths are canonicalized so symlink targets and lexical `..` resolve
to the file that will actually be executed. PATH lookup names are returned
as-is for the OS to resolve at spawn time.

Prefer [`require_existing_regular_file`](#require_existing_regular_file) for configured/env worker paths that
must identify a real file (including cwd-relative `./bin`) and must not
authorize ambient PATH lookup for a bare name.

#### Errors

Returns [`SpawnPathError`](#spawnpatherror) when validation fails, canonicalize fails, or an
absolute path is not a regular file.

### require_under_root

```rust
pub fn require_under_root(path: &Path, root: &Path) -> Result<PathBuf, SpawnPathError>
```

Canonicalizes `path` and requires it to stay under canonical `root`.

When `path` does not exist yet, canonicalizes its parent and rejoins the
final component so generated files under a trusted session directory work.

#### Errors

Returns [`SpawnPathError`](#spawnpatherror) when the path is unsafe or escapes `root`.

### node_logical_cpus

```rust
pub fn node_logical_cpus() -> u32
```

Number of logical CPUs visible to this process (at least 1).

### host_cpu_rate_max

```rust
pub fn host_cpu_rate_max() -> u32
```

Maximum Spec / grant CPU rate: 100% × logical CPUs (one-core units).

Example: 8 logical CPUs → `800` (eight full cores of CFS/Job bandwidth).

### windows_job_cpu_rate

```rust
pub fn windows_job_cpu_rate(percent_of_one_cpu: u32, logical_cpus: u32) -> u32
```

Map a percent-of-one-CPU hard cap onto a Windows Job Object `CpuRate`.

Job `CpuRate` is “cycles per 10 000” of **machine** capacity. Bookclerk’s
[`ResourceLimits::cpu_rate_percent`](#resourcelimits) is percent of **one** logical CPU, so
this divides by `logical_cpus` (at least 1). Result is clamped to `1..=10000`
because `CpuRate = 0` is rejected by the API.

#### Examples

- 80% of one core on an 8-CPU host → 10% of the machine → `1000`
- 400% (four cores) on an 8-CPU host → 50% of the machine → `5000`

### label_resource_defaults

```rust
pub fn label_resource_defaults(label: &str) -> ResourceLimits
```

Label-based Job defaults used when a [`Spec`](#spec) leaves resource fields unset.

Plugin guests are capped tightly; labels containing `"media"` get more
headroom (matching the historical Windows Job heuristics).

### capabilities

```rust
pub fn capabilities() -> Capabilities
```

Probe what this host supports. Applies nothing; safe to call at any time.

### ensure_dir

```rust
pub fn ensure_dir(path: &Path) -> Result<PathBuf>
```

Resolve a path for use in an allowlist, creating it if absent.

Landlock and Seatbelt both need the path to exist when the rule is added, so
callers that will write into a scratch directory should create it first.

#### Errors

Propagates directory-creation failures.

## Constants

### LOW_RESOURCE_MEMORY_MAX_BYTES

```rust
pub const LOW_RESOURCE_MEMORY_MAX_BYTES: u64 = _;
```

`memory.max` at or below this many bytes is the small-VPS ceiling.

### MACOS_SUN_PATH_CAPACITY

```rust
pub const MACOS_SUN_PATH_CAPACITY: usize = 104;
```

macOS `sun_path` capacity. Rust rejects a path whose length is greater than
or equal to this value.

### GATEWAY_GUEST_RPC_ENV

```rust
pub const GATEWAY_GUEST_RPC_ENV: &str = "BOOKCLERK_GATEWAY_GUEST_RPC";
```

Env var naming the guest Cap'n Proto link for `bookclerk-workerd`.

Unix: one duplex socket, `fd:<n>`. Windows: the gateway's overlapped read
half (guest stdout → gateway), `handle:<n>`. The write half is
[`GATEWAY_GUEST_RPC_WRITE_ENV`](#gateway_guest_rpc_write_env).

### GATEWAY_GUEST_RPC_WRITE_ENV

```rust
pub const GATEWAY_GUEST_RPC_WRITE_ENV: &str = "BOOKCLERK_GATEWAY_GUEST_RPC_WRITE";
```

Windows-only env var naming the gateway's overlapped write half of guest RPC.

`handle:<n>` writes into the guest's synchronous stdin. Absent on Unix,
where [`GATEWAY_GUEST_RPC_ENV`](#gateway_guest_rpc_env) is a full-duplex socket.

### GATEWAY_PROXY_ENV

```rust
pub const GATEWAY_PROXY_ENV: &str = "BOOKCLERK_GATEWAY_PROXY";
```

Env var naming the multiplexed CONNECT-proxy link for `bookclerk-workerd`.

Unix: one duplex socket, `fd:<n>`. Windows: the gateway's overlapped read
half, `handle:<n>`. The write half is [`GATEWAY_PROXY_WRITE_ENV`](#gateway_proxy_write_env).

### GATEWAY_PROXY_FD

```rust
pub const GATEWAY_PROXY_FD: i32 = 4;
```

Child fd the gateway uses for the proxy mux (`BOOKCLERK_GATEWAY_PROXY`).

### GATEWAY_PROXY_WRITE_ENV

```rust
pub const GATEWAY_PROXY_WRITE_ENV: &str = "BOOKCLERK_GATEWAY_PROXY_WRITE";
```

Windows-only env var naming the gateway's overlapped write half of the proxy.

Absent on Unix, where [`GATEWAY_PROXY_ENV`](#gateway_proxy_env) is a full-duplex socket.

### GATEWAY_RPC_FD

```rust
pub const GATEWAY_RPC_FD: i32 = 3;
```

Child fd the gateway uses for the guest RPC socket (`BOOKCLERK_GATEWAY_GUEST_RPC`).

### GUEST_PROXY_FD

```rust
pub const GUEST_PROXY_FD: i32 = 3;
```

Child fd the native guest uses for the proxy mux (`BOOKCLERK_SOCKET_PROXY`).

### JAIL_HANDOFF_ENV

```rust
pub const JAIL_HANDOFF_ENV: &str = "BOOKCLERK_JAIL_HANDOFF";
```

When `1`, `bookclerk-jail` reads one [`JailHandoff`](#jailhandoff) JSON line from stdin.

### SOCKET_PROXY_ENV

```rust
pub const SOCKET_PROXY_ENV: &str = "BOOKCLERK_SOCKET_PROXY";
```

Env var naming the socket-proxy link for a native guest (SDK `net::connect`).

Unix: one duplex socket, `fd:<n>`. Windows product spawns name the guest's
overlapped read half here; the write half is [`SOCKET_PROXY_WRITE_ENV`](#socket_proxy_write_env).

### SOCKET_PROXY_WRITE_ENV

```rust
pub const SOCKET_PROXY_WRITE_ENV: &str = "BOOKCLERK_SOCKET_PROXY_WRITE";
```

Windows-only env var naming the guest's overlapped write half of the proxy.

The SDK still accepts a single `handle:<n>` duplex when this is unset
(unit tests). Product spawns set both halves so a pending read cannot lock
a write on the same pipe.

### TEST_EXTRA_ENDPOINT_FD

```rust
pub const TEST_EXTRA_ENDPOINT_FD: i32 = 128;
```

Test-only extra endpoint in the native guest. Production preserve lists omit it.

The slot sits above the descriptors a dynamic loader opens after the jail
closes every fd it does not preserve. A low number would be reused by that
loader; an open non-socket there is a numeric collision, not proof the
endpoint is absent. Production guests still preserve only [`GUEST_PROXY_FD`](#guest_proxy_fd).

### WORKERD_STATE_DIR_ENV

```rust
pub const WORKERD_STATE_DIR_ENV: &str = "BOOKCLERK_WORKERD_STATE_DIR";
```

Host-chosen workerd session directory (`0700`, under `$TMPDIR`).

### BACKEND

```rust
pub const BACKEND: &str = "landlock+seccomp";
```

Backend name reported in diagnostics.

### SPAWN_DIAG_MAX_LINES

```rust
pub const SPAWN_DIAG_MAX_LINES: usize = 80;
```

Maximum lines in that ring. The byte cap is the budget; this only stops a
flood of tiny lines.

### SPAWN_DIAG_RECORD_BYTES

```rust
pub const SPAWN_DIAG_RECORD_BYTES: usize = 512;
```

Maximum bytes stored for one diagnostic line, cut on a UTF-8 boundary.

### SPAWN_DIAG_TOTAL_BYTES

```rust
pub const SPAWN_DIAG_TOTAL_BYTES: usize = _;
```

Maximum bytes retained in the process-wide diagnostic ring.

### PLUGIN_FD_CHANNEL

```rust
pub const PLUGIN_FD_CHANNEL: i32 = 3;
```

Reserved descriptor number for a native SCM_RIGHTS side channel (fd 3).

Plugin hosts do **not** leave this open or set [`PLUGIN_FD_CHANNEL_ENV`](#plugin_fd_channel_env).
Kept so a future host-selected native shortcut can reuse the same number.

### PLUGIN_FD_CHANNEL_ENV

```rust
pub const PLUGIN_FD_CHANNEL_ENV: &str = "BOOKCLERK_PLUGIN_FD_CHANNEL";
```

Environment variable naming [`PLUGIN_FD_CHANNEL`](#plugin_fd_channel).

Hosts do not set this; guests must not wait on `recvmsg` just because
they see fd 3 inherited from a parent.

### SPEC_ENV

```rust
pub const SPEC_ENV: &str = "BOOKCLERK_JAIL_SPEC";
```

Environment variable carrying the JSON [`Spec`](#spec) to `bookclerk-jail`.

The launcher drops it before handing off, so a guest never sees the shape of
its own jail and cannot pass it on to something it spawns.

### GATEWAY_WORKER_THREADS

```rust
pub const GATEWAY_WORKER_THREADS: usize = 2;
```

Tokio worker threads for `bookclerk-workerd`.

Fixed so a session `pids.max` does not grow with [`node_logical_cpus`](#node_logical_cpus).

### GATEWAY_MAX_BLOCKING_THREADS

```rust
pub const GATEWAY_MAX_BLOCKING_THREADS: usize = 4;
```

Blocking-pool ceiling for `bookclerk-workerd`.

### INFRASTRUCTURE_THREAD_BUDGET

```rust
pub const INFRASTRUCTURE_THREAD_BUDGET: u32 = _;
```

Threads reserved for one native-behind-workerd session before `extraProcesses`.

Covers the pinned gateway runtime ([`GATEWAY_WORKER_THREADS`](#gateway_worker_threads) plus
[`GATEWAY_MAX_BLOCKING_THREADS`](#gateway_max_blocking_threads)), the pinned Cloudflare `workerd` process,
the native guest's current-thread runtime, and the two jail supervisors.
This is not the Windows outer process baseline (5 + extra) and not the
payload process count (3 + extra). `pids.max` counts threads; guest
`extraProcesses` are added on top of this budget.
