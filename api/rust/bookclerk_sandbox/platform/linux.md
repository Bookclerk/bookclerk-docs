**bookclerk_sandbox > platform > linux**

# Module: platform::linux

## Contents

**Functions**

- [`create_session_cgroup`](#create_session_cgroup) - Create an exclusive session cgroup leaf and write `limits` without moving the caller.
- [`destroy_session_cgroup`](#destroy_session_cgroup) - Kill every member of `dir` and remove the leaf.

**Constants**

- [`BACKEND`](#backend) - Backend name reported in diagnostics.

---

## bookclerk_sandbox::platform::linux::BACKEND

*Constant*: `&str`

Backend name reported in diagnostics.



## bookclerk_sandbox::platform::linux::create_session_cgroup

*Function*

Create an exclusive session cgroup leaf and write `limits` without moving the caller.

The host assigns both sibling jails via `crate::Spec::cgroup_dir`. The leaf
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

# Errors

Returns a string when the hierarchy is missing, the leaf already exists, a
child cannot be created, `pids` cannot be delegated, or a present limit file
cannot be written. A partial directory is removed when writing limits fails.

```rust
fn create_session_cgroup(limits: &crate::ResourceLimits, suffix: &str) -> Result<std::path::PathBuf, String>
```



## bookclerk_sandbox::platform::linux::destroy_session_cgroup

*Function*

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

# Errors

Returns a string when `cgroup.kill` is missing or cannot be written, the
leaf stays populated, or the empty directory cannot be removed. A missing
`cgroup.kill` says equivalent teardown is unsupported.

```rust
fn destroy_session_cgroup(dir: &std::path::Path) -> Result<(), String>
```



