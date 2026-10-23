**bookclerk_sandbox > spec**

# Module: spec

## Contents

**Structs**

- [`Spec`](#spec) - A serializable confinement request.

**Constants**

- [`PLUGIN_FD_CHANNEL`](#plugin_fd_channel) - Reserved descriptor number for a native SCM_RIGHTS side channel (fd 3).
- [`PLUGIN_FD_CHANNEL_ENV`](#plugin_fd_channel_env) - Environment variable naming [`PLUGIN_FD_CHANNEL`](spec.md#bookclerk_sandboxspecplugin_fd_channel).
- [`SPEC_ENV`](#spec_env) - Environment variable carrying the JSON [`Spec`](spec.md#bookclerk_sandboxspecspec) to `bookclerk-jail`.

---

## bookclerk_sandbox::spec::PLUGIN_FD_CHANNEL

*Constant*: `i32`

Reserved descriptor number for a native SCM_RIGHTS side channel (fd 3).

Plugin hosts do **not** leave this open or set [`PLUGIN_FD_CHANNEL_ENV`](spec.md#bookclerk_sandboxspecplugin_fd_channel_env).
Kept so a future host-selected native shortcut can reuse the same number.



## bookclerk_sandbox::spec::PLUGIN_FD_CHANNEL_ENV

*Constant*: `&str`

Environment variable naming [`PLUGIN_FD_CHANNEL`](spec.md#bookclerk_sandboxspecplugin_fd_channel).

Hosts do not set this; guests must not wait on `recvmsg` just because
they see fd 3 inherited from a parent.



## bookclerk_sandbox::spec::SPEC_ENV

*Constant*: `&str`

Environment variable carrying the JSON [`Spec`](spec.md#bookclerk_sandboxspecspec) to `bookclerk-jail`.

The launcher drops it before handing off, so a guest never sees the shape of
its own jail and cannot pass it on to something it spawns.



## bookclerk_sandbox::spec::Spec

*Struct*

A serializable confinement request.

Mirrors [`Policy`](bookclerk_sandbox.md#bookclerk_sandboxpolicy), which is a builder and deliberately does not expose its
unresolved fields. Paths are resolved by the launcher, in the launcher's own
view of the filesystem, rather than being canonicalized here — the host and
the jailed process must agree on physical paths, and only the process that
applies the policy can confirm they do.

**Fields:**
- `label: String` - Diagnostics label, echoed in the launcher's report.
- `reads: Vec<std::path::PathBuf>` - Paths the confined process may read.
- `writes: Vec<std::path::PathBuf>` - Paths the confined process may read and write.
- `net: crate::NetPolicy` - Network reachability granted to the confined process.
- `allow_exec: bool` - Whether `execve` is permitted.
- `system_paths: bool` - Whether to include the platform's read-only system set.
- `enforcement: crate::Enforcement` - What to do when a layer does not engage.
- `preserve_fds: Vec<i32>` - Descriptors the host wired up deliberately and the launcher must not
- `windows_profile_name: Option<String>` - Pre-created AppContainer profile moniker (Windows).
- `memory_bytes: Option<u64>` - Soft memory ceiling in bytes (Job Object / cgroup v2 `memory.max`).
- `active_processes: Option<u32>` - Cap on concurrent processes in the jail (Job active-process / `pids.max`).
- `cpu_rate_percent: crate::CpuRate` - CPU hard-cap as a percent of **one logical CPU** (`1..=` host cores×100).
- `inherit_handles: Vec<u64>` - Windows handles the jail must put on the child's inherit list.
- `cgroup_dir: Option<std::path::PathBuf>` - Linux cgroup v2 leaf both sibling jails join (session aggregate limits).
- `unix_socket_dirs: Option<Vec<std::path::PathBuf>>` - macOS Seatbelt pathname Unix-socket directories.

**Methods:**

- `fn new<impl Into<String>>(label: impl Trait) -> Self` - Start a spec with nothing granted.
- `fn policy(self: &Self) -> Policy` - Build the [`Policy`](bookclerk_sandbox.md#bookclerk_sandboxpolicy) this spec describes.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Spec`
- **PartialEq**
  - `fn eq(self: &Self, other: &Spec) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



