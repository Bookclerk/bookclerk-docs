**bookclerkd > job_worker**

# Module: job_worker

## Contents

**Functions**

- [`jittered_delay`](#jittered_delay) - Sleep `interval` with ±10% jitter (never negative).
- [`reconcile_on_startup`](#reconcile_on_startup) - Reclaim expired leases, fail orphaned book rows, and delete unregistered scratch dirs.
- [`start_job_runtime`](#start_job_runtime) - Reclaim leases, reconcile orphan acquire rows, sweep scratch dirs, start workers.
- [`sweep_orphan_temp_dirs`](#sweep_orphan_temp_dirs) - Delete `{cache}/acquire*` directories that are not registered to an active job.

---

## bookclerkd::job_worker::jittered_delay

*Function*

Sleep `interval` with ±10% jitter (never negative).

```rust
fn jittered_delay(interval: std::time::Duration) -> std::time::Duration
```



## bookclerkd::job_worker::reconcile_on_startup

*Function*

Reclaim expired leases, fail orphaned book rows, and delete unregistered scratch dirs.

```rust
fn reconcile_on_startup(state: &crate::api::AppState) -> anyhow::Result<()>
```



## bookclerkd::job_worker::start_job_runtime

*Function*

Reclaim leases, reconcile orphan acquire rows, sweep scratch dirs, start workers.

```rust
fn start_job_runtime(state: std::sync::Arc<crate::api::AppState>)
```



## bookclerkd::job_worker::sweep_orphan_temp_dirs

*Function*

Delete `{cache}/acquire*` directories that are not registered to an active job.

```rust
fn sweep_orphan_temp_dirs(library: &bookclerk_library::LibraryStore, cache_dir: &std::path::Path) -> anyhow::Result<u32>
```



