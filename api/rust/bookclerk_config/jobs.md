**bookclerk_config > jobs**

# Module: jobs

## Contents

**Structs**

- [`JobsConcurrencyConfig`](#jobsconcurrencyconfig) - Per-resource-class worker counts and cluster-wide job semaphores.
- [`JobsConfig`](#jobsconfig) - `[jobs]` — durable queue admission, leases, and resource-class concurrency.

---

## bookclerk_config::jobs::JobsConcurrencyConfig

*Struct*

Per-resource-class worker counts and cluster-wide job semaphores.

**Fields:**
- `network: u32` - Scan / acquire / listening-sync workers on this node (default 1),
- `plugin_instance: u32` - Cluster-wide running jobs that require one plugin instance (the

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobsConcurrencyConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobsConcurrencyConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_config::jobs::JobsConfig

*Struct*

`[jobs]` — durable queue admission, leases, and resource-class concurrency.

**Fields:**
- `max_pending: u32` - Maximum pending+running jobs. Further admits return queue-full.
- `lease_seconds: u64` - Lease length in seconds; expired running jobs are reclaimed.
- `max_attempts: u32` - Maximum claims before a failure is terminal.
- `retention_days: u64` - Days to retain succeeded/failed/cancelled rows.
- `temp_quota_bytes: u64` - Refuse new acquire scratch dirs when registered temps exceed this.
- `concurrency: JobsConcurrencyConfig` - Per-class concurrency (`[jobs.concurrency]`).

**Methods:**

- `fn apply_env_overrides(self: & mut Self)` - Apply `BOOKCLERK_JOBS_*` environment overrides.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobsConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> JobsConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



