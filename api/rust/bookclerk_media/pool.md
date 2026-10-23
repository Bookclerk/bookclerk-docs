**bookclerk_media > pool**

# Module: pool

## Contents

**Structs**

- [`MediaPool`](#mediapool) - Bounded pool of confined media workers.
- [`MediaPoolConfig`](#mediapoolconfig) - Pool sizing and isolation settings.

**Enums**

- [`Confinement`](#confinement) - How strictly a worker must be confined before it will process a job.

**Functions**

- [`init_pool`](#init_pool) - Install the process-wide pool if none is installed yet.
- [`init_pool_from_config`](#init_pool_from_config) - Build the pool from `[media]` config and install it, logging what it will do.
- [`pool`](#pool) - The process-wide pool, creating a default one if startup never installed it.
- [`replace_pool`](#replace_pool) - Swap in `pool`, returning the one it replaced.

**Constants**

- [`JAIL_BIN_ENV`](#jail_bin_env) - Environment variable naming the jail launcher (shared with plugin hosts).
- [`JAIL_BIN_NAME`](#jail_bin_name) - File name of the jail launcher when it sits beside the host / worker.
- [`WORKER_BIN_ENV`](#worker_bin_env) - Environment variable naming the worker binary explicitly.
- [`WORKER_BIN_NAME`](#worker_bin_name) - File name of the worker binary when it sits beside the host executable.
- [`WORKER_ENFORCEMENT_ENV`](#worker_enforcement_env) - Environment variable the pool sets on each worker to select its failure mode.

---

## bookclerk_media::pool::Confinement

*Enum*

How strictly a worker must be confined before it will process a job.

**Variants:**
- `Required` - Refuse to run the job unless the worker confined itself. Production
- `BestEffort` - Confine where the platform allows and log what did not engage.
- `Off` - Run jobs on a blocking thread in this process. Intended for development

**Methods:**

- `fn from_env_value(value: &str) -> Option<Self>` - Parse the value carried in [`WORKER_ENFORCEMENT_ENV`](pool.md#bookclerk_mediapoolworker_enforcement_env).
- `fn as_env_value(self: Self) -> &'static str` - The value to pass through [`WORKER_ENFORCEMENT_ENV`](pool.md#bookclerk_mediapoolworker_enforcement_env).

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &Confinement) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> Confinement`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Confinement`



## bookclerk_media::pool::JAIL_BIN_ENV

*Constant*: `&str`

Environment variable naming the jail launcher (shared with plugin hosts).



## bookclerk_media::pool::JAIL_BIN_NAME

*Constant*: `&str`

File name of the jail launcher when it sits beside the host / worker.



## bookclerk_media::pool::MediaPool

*Struct*

Bounded pool of confined media workers.

**Methods:**

- `fn new(config: MediaPoolConfig) -> Self` - Build a pool from `config`, resolving the worker binary once.
- `fn in_process() -> Self` - A pool that runs every job on a blocking thread in this process.
- `fn capacity(self: &Self) -> usize` - Maximum jobs this pool will run at once.
- `fn is_isolated(self: &Self) -> bool` - Whether jobs run in a separate, confined process.
- `fn is_refusing(self: &Self) -> bool` - Whether this pool will refuse every job because isolation is required
- `fn worker_bin(self: &Self) -> Option<&PathBuf>` - Resolved worker binary, when one was found.
- `fn summary(self: &Self) -> String` - One-line summary for startup logs and `bookclerk doctor`.
- `fn run(self: &Self, job: MediaJob) -> Result<MediaJobOutput>` - Run `job`, waiting for a free slot first.

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)` - A pool is dropped when the last handle to it goes away, which for a pool
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Self`



## bookclerk_media::pool::MediaPoolConfig

*Struct*

Pool sizing and isolation settings.

**Fields:**
- `workers: usize` - Maximum jobs running at once. Zero means "derive from available
- `confinement: Confinement` - How strictly workers must be confined.
- `worker_bin: Option<std::path::PathBuf>` - Explicit worker binary path. When absent the pool looks at

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> MediaPoolConfig`
- **Clone**
  - `fn clone(self: &Self) -> MediaPoolConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(config: &bookclerk_config::MediaConfig) -> Self`
- **PartialEq**
  - `fn eq(self: &Self, other: &MediaPoolConfig) -> bool`



## bookclerk_media::pool::WORKER_BIN_ENV

*Constant*: `&str`

Environment variable naming the worker binary explicitly.



## bookclerk_media::pool::WORKER_BIN_NAME

*Constant*: `&str`

File name of the worker binary when it sits beside the host executable.



## bookclerk_media::pool::WORKER_ENFORCEMENT_ENV

*Constant*: `&str`

Environment variable the pool sets on each worker to select its failure mode.



## bookclerk_media::pool::init_pool

*Function*

Install the process-wide pool if none is installed yet.

# Errors

Returns the rejected pool when one was already installed. Use
[`replace_pool`](pool.md#bookclerk_mediapoolreplace_pool) to swap a pool that is already running.

```rust
fn init_pool(pool: MediaPool) -> std::result::Result<(), MediaPool>
```



## bookclerk_media::pool::init_pool_from_config

*Function*

Build the pool from `[media]` config and install it, logging what it will do.

Safe to call more than once, which is what a config reload does. An
unchanged `[media]` leaves the running pool alone; a changed one swaps in a
new pool for subsequent jobs and lets the old one drain. See
[`replace_pool`](pool.md#bookclerk_mediapoolreplace_pool).

```rust
fn init_pool_from_config(config: &bookclerk_config::MediaConfig)
```



## bookclerk_media::pool::pool

*Function*

The process-wide pool, creating a default one if startup never installed it.

Returns an owned handle on purpose. Holding it for the length of a job is
what lets a reload swap the pool without disturbing work already running.

```rust
fn pool() -> std::sync::Arc<MediaPool>
```



## bookclerk_media::pool::replace_pool

*Function*

Swap in `pool`, returning the one it replaced.

Safe to call with jobs in flight, because [`pool`](pool.md#bookclerk_mediapoolpool) hands out an `Arc` and a
job holds it for its whole duration. The moment this returns, the outgoing
pool is unreachable, so it can only drain: its permits are released as its
jobs finish and it is dropped after the last one. Nothing is interrupted and
nothing waits.

In-flight jobs keep the isolation they started with, which is the only
possible answer — a worker's confinement is applied inside the child process
at spawn and cannot be changed afterwards.

While the outgoing pool drains, total codec concurrency can briefly reach the
old pool's in-flight count plus the new pool's limit. That overshoot only
shrinks, since the old pool never admits another job.

```rust
fn replace_pool(pool: MediaPool) -> Option<std::sync::Arc<MediaPool>>
```



