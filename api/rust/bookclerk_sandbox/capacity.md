**bookclerk_sandbox > capacity**

# Module: capacity

## Contents

**Structs**

- [`CgroupSample`](#cgroupsample) - Parsed cgroup v2 files for the process that is actually running.
- [`SystemMemory`](#systemmemory) - Whole-machine memory figures for heartbeats.

**Enums**

- [`CpuTimes`](#cputimes) - Cumulative CPU time counters. Two samples give utilization.

**Functions**

- [`cgroup_sample_is_low_resource`](#cgroup_sample_is_low_resource) - True when `memory.max` is at most 1 GiB or `cpu.max` is at most one core.
- [`cpu_used_millicores`](#cpu_used_millicores) - CPU in use between two samples, in millicores (1000 = one core busy).
- [`filesystem_free_bytes`](#filesystem_free_bytes) - Free bytes available to unprivileged callers.
- [`logical_cpu_count`](#logical_cpu_count) - Logical CPUs from [`std::thread::available_parallelism`](https://doc.rust-lang.org/stable/std/thread/fn.available_parallelism.html), or empty on failure.
- [`macos_cpu_ticks`](#macos_cpu_ticks) - macOS `HOST_CPU_LOAD_INFO` ticks (`user`, `system`, `idle`, `nice`,
- [`macos_vm_memory`](#macos_vm_memory) - macOS available memory from `HOST_VM_INFO64` page counts.
- [`parse_cpu_stat_usage`](#parse_cpu_stat_usage) - `usage_usec` from cgroup v2 `cpu.stat` text.
- [`parse_meminfo`](#parse_meminfo) - Parses `/proc/meminfo` text (values are in kB).
- [`parse_proc_stat`](#parse_proc_stat) - Aggregate `cpu` line of `/proc/stat` text.
- [`process_cgroup_dir`](#process_cgroup_dir) - Directory of this process's cgroup v2 node (`/proc/self/cgroup` `0::` path).
- [`read_cgroup_sample`](#read_cgroup_sample) - Reads `cpu.max`, `memory.max`, `memory.current`, and `memory.stat` in `dir`.
- [`read_cpu_times`](#read_cpu_times) - Reads CPU time counters for utilization: the cgroup's own usage when the
- [`read_system_memory`](#read_system_memory) - Reads total and available physical memory for this machine.
- [`windows_system_times`](#windows_system_times) - Windows `GetSystemTimes` values (100 ns units) as `CpuTimes::System`.

**Constants**

- [`LOW_RESOURCE_MEMORY_MAX_BYTES`](#low_resource_memory_max_bytes) - `memory.max` at or below this many bytes is the small-VPS ceiling.

---

## bookclerk_sandbox::capacity::CgroupSample

*Struct*

Parsed cgroup v2 files for the process that is actually running.

Unlimited (`max`) and missing files are `None`.

**Fields:**
- `cpu_max_quota_us: Option<u64>` - `cpu.max` quota in microseconds. Empty when the file is missing or `max`.
- `cpu_max_period_us: Option<u64>` - `cpu.max` period in microseconds. Empty when the file is missing or `max`.
- `memory_max_bytes: Option<u64>` - `memory.max` in bytes. Empty when the file is missing or `max`.
- `memory_current_bytes: Option<u64>` - `memory.current` in bytes.
- `memory_anon_bytes: Option<u64>` - `memory.stat` field `anon`.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &CgroupSample) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CgroupSample`
- **Default**
  - `fn default() -> CgroupSample`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_sandbox::capacity::CpuTimes

*Enum*

Cumulative CPU time counters. Two samples give utilization.

`CpuTimes::System` units differ by platform (jiffies on Linux, Mach
ticks on macOS, 100 ns on Windows); only the busy/total ratio between two
samples of the same host is used, so the unit cancels out.

**Variants:**
- `System{ busy: u64, total: u64 }` - Whole-machine busy and total time across all CPUs: the aggregate
- `Cgroup{ usage_usec: u64 }` - cgroup v2 `cpu.stat` `usage_usec` of a CPU-limited cgroup.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &CpuTimes) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CpuTimes`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_sandbox::capacity::LOW_RESOURCE_MEMORY_MAX_BYTES

*Constant*: `u64`

`memory.max` at or below this many bytes is the small-VPS ceiling.



## bookclerk_sandbox::capacity::SystemMemory

*Struct*

Whole-machine memory figures for heartbeats.

Linux reads `/proc/meminfo`, macOS `hw.memsize` and the Mach VM
statistics, Windows `GlobalMemoryStatusEx`. Other platforms report nothing
and the scheduler falls back to the detected CPU capacity alone.

**Fields:**
- `total_bytes: Option<u64>` - Physical memory in bytes (`MemTotal`, `hw.memsize`, `ullTotalPhys`).
- `available_bytes: Option<u64>` - Bytes new work can use without swapping (`MemAvailable`; free plus

**Traits:** Eq, Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> SystemMemory`
- **Clone**
  - `fn clone(self: &Self) -> SystemMemory`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SystemMemory) -> bool`



## bookclerk_sandbox::capacity::cgroup_sample_is_low_resource

*Function*

True when `memory.max` is at most 1 GiB or `cpu.max` is at most one core.

Missing files and the unlimited token `max` are not a low-resource signal.

```rust
fn cgroup_sample_is_low_resource(sample: &CgroupSample) -> bool
```



## bookclerk_sandbox::capacity::cpu_used_millicores

*Function*

CPU in use between two samples, in millicores (1000 = one core busy).

`cores_millis` scales the machine-wide busy fraction; `elapsed` is the
wall time between a cgroup's two samples. Returns `None` when the
samples are of different kinds or did not advance.

```rust
fn cpu_used_millicores(before: CpuTimes, after: CpuTimes, cores_millis: u64, elapsed: std::time::Duration) -> Option<u64>
```



## bookclerk_sandbox::capacity::filesystem_free_bytes

*Function*

Free bytes available to unprivileged callers.

`statvfs` `f_bavail` on Linux and macOS; `GetDiskFreeSpaceExW` (free bytes
available to the caller, so quotas count) on Windows. Returns `None`
when the path cannot be stated or on other platforms.

```rust
fn filesystem_free_bytes(path: &std::path::Path) -> Option<u64>
```



## bookclerk_sandbox::capacity::logical_cpu_count

*Function*

Logical CPUs from [`std::thread::available_parallelism`](https://doc.rust-lang.org/stable/std/thread/fn.available_parallelism.html), or empty on failure.

```rust
fn logical_cpu_count() -> Option<i64>
```



## bookclerk_sandbox::capacity::macos_cpu_ticks

*Function*

macOS `HOST_CPU_LOAD_INFO` ticks (`user`, `system`, `idle`, `nice`,
summed over all CPUs) as `CpuTimes::System`.

The kernel counters are 32-bit and wrap; a wrapped pair makes
[`cpu_used_millicores`](capacity.md#bookclerk_sandboxcapacitycpu_used_millicores) skip that one sample rather than report nonsense.

```rust
fn macos_cpu_ticks(user: u32, system: u32, idle: u32, nice: u32) -> CpuTimes
```



## bookclerk_sandbox::capacity::macos_vm_memory

*Function*

macOS available memory from `HOST_VM_INFO64` page counts.

Available is `free + inactive` pages (the figure `psutil` reports): pages
the kernel hands to new work without paging anything out first. The
result is clamped to `total_bytes`. A zero page size leaves it empty.

```rust
fn macos_vm_memory(total_bytes: Option<u64>, free_pages: u64, inactive_pages: u64, page_size: u64) -> SystemMemory
```



## bookclerk_sandbox::capacity::parse_cpu_stat_usage

*Function*

`usage_usec` from cgroup v2 `cpu.stat` text.

```rust
fn parse_cpu_stat_usage(text: &str) -> Option<u64>
```



## bookclerk_sandbox::capacity::parse_meminfo

*Function*

Parses `/proc/meminfo` text (values are in kB).

```rust
fn parse_meminfo(text: &str) -> SystemMemory
```



## bookclerk_sandbox::capacity::parse_proc_stat

*Function*

Aggregate `cpu` line of `/proc/stat` text.

```rust
fn parse_proc_stat(text: &str) -> Option<CpuTimes>
```



## bookclerk_sandbox::capacity::process_cgroup_dir

*Function*

Directory of this process's cgroup v2 node (`/proc/self/cgroup` `0::` path).

Returns `None` when the file is missing, has no v2 entry, or the
corresponding `/sys/fs/cgroup` directory does not exist.

```rust
fn process_cgroup_dir() -> Option<std::path::PathBuf>
```



## bookclerk_sandbox::capacity::read_cgroup_sample

*Function*

Reads `cpu.max`, `memory.max`, `memory.current`, and `memory.stat` in `dir`.

Effective `memory.max` is the minimum ceiling among `dir` and ancestors
that still publish `memory.max` or `cpu.max`. Effective `cpu.max` is the
tighter quota/period ratio on that same walk. `memory.current` and `anon`
stay the leaf values. The walk stops when a parent has neither controller
file, so a temporary test directory does not escape into `/tmp`.

Does not create `dir` or any controller file.

```rust
fn read_cgroup_sample(dir: &std::path::Path) -> CgroupSample
```



## bookclerk_sandbox::capacity::read_cpu_times

*Function*

Reads CPU time counters for utilization: the cgroup's own usage when the
process runs under a CPU-limited cgroup (a container quota is the real
ceiling), else the whole machine (a desktop's other applications count
against what Bookclerk can use). The whole-machine counters come from
`/proc/stat` on Linux, `HOST_CPU_LOAD_INFO` on macOS, and
`GetSystemTimes` on Windows.

```rust
fn read_cpu_times(cgroup_dir: Option<&std::path::Path>, cgroup: &CgroupSample) -> Option<CpuTimes>
```



## bookclerk_sandbox::capacity::read_system_memory

*Function*

Reads total and available physical memory for this machine.

Linux: `MemTotal` / `MemAvailable` from `/proc/meminfo`. macOS:
`hw.memsize` and `HOST_VM_INFO64` (free plus inactive pages, see
[`macos_vm_memory`](capacity.md#bookclerk_sandboxcapacitymacos_vm_memory)). Windows: `GlobalMemoryStatusEx`. Fields stay empty
when the platform call fails or on other platforms.

```rust
fn read_system_memory() -> SystemMemory
```



## bookclerk_sandbox::capacity::windows_system_times

*Function*

Windows `GetSystemTimes` values (100 ns units) as `CpuTimes::System`.

`kernel` already includes `idle`, so busy is `kernel + user - idle` and
total is `kernel + user`.

```rust
fn windows_system_times(idle: u64, kernel: u64, user: u64) -> CpuTimes
```



