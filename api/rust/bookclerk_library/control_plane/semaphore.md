**bookclerk_library > control_plane > semaphore**

# Module: control_plane::semaphore

## Contents

**Structs**

- [`SemaphorePermit`](#semaphorepermit) - One held permit: a slot lease of a semaphore.
- [`SemaphoreRequest`](#semaphorerequest) - One semaphore a caller wants a permit of.

**Enums**

- [`PermitAcquire`](#permitacquire) - Outcome of [`acquire_permits`](semaphore.md#bookclerk_librarycontrol_planesemaphoreacquire_permits).

**Functions**

- [`acquire_permit`](#acquire_permit) - Takes one permit of `name` for `holder`, valid for `ttl`.
- [`acquire_permits`](#acquire_permits) - Takes one permit of every semaphore in `requests`, or none.
- [`live_permits`](#live_permits) - Live permits of `name` among its first `max_slots` slots (expired slots
- [`permits_held_by`](#permits_held_by) - Permits of `name` among its first `max_slots` slots whose holder starts
- [`release_permits`](#release_permits) - Gives back every permit in `permits` so the slots are free at once.
- [`renew_fenced_lease`](#renew_fenced_lease) - Extends `lease` by `ttl` only while the row still carries its holder and
- [`renew_permits`](#renew_permits) - Extends every permit in `permits` by `ttl` while its holder and token
- [`semaphore_slot_lease_name`](#semaphore_slot_lease_name) - Lease name of slot `slot` of semaphore `name`.

**Constants**

- [`MAX_SEMAPHORE_LIMIT`](#max_semaphore_limit) - Largest limit a semaphore honours; larger limits are clamped.
- [`SEMAPHORE_LEASE_PREFIX`](#semaphore_lease_prefix) - Prefix of every semaphore slot lease name.

---

## bookclerk_library::control_plane::semaphore::MAX_SEMAPHORE_LIMIT

*Constant*: `u32`

Largest limit a semaphore honours; larger limits are clamped.



## bookclerk_library::control_plane::semaphore::PermitAcquire

*Enum*

Outcome of [`acquire_permits`](semaphore.md#bookclerk_librarycontrol_planesemaphoreacquire_permits).

**Variants:**
- `Granted(Vec<SemaphorePermit>)` - Every requested permit is held.
- `Blocked(String)` - The named semaphore had no free slot; nothing is held.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PermitAcquire`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PermitAcquire) -> bool`



## bookclerk_library::control_plane::semaphore::SEMAPHORE_LEASE_PREFIX

*Constant*: `&str`

Prefix of every semaphore slot lease name.



## bookclerk_library::control_plane::semaphore::SemaphorePermit

*Struct*

One held permit: a slot lease of a semaphore.

**Fields:**
- `semaphore: String` - Semaphore name.
- `slot: u32` - Slot index below the limit the permit was taken under.
- `lease: super::ClusterLease` - The slot lease (holder, fencing token, expiry).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SemaphorePermit`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SemaphorePermit) -> bool`



## bookclerk_library::control_plane::semaphore::SemaphoreRequest

*Struct*

One semaphore a caller wants a permit of.

**Fields:**
- `name: String` - Semaphore name (for example `jobs:network` or `account:<key>:<id>`).
- `limit: u32` - Permits the semaphore grants cluster-wide (at least 1).

**Methods:**

- `fn new<impl Into<String>>(name: impl Trait, limit: u32) -> Self` - A request for one permit of `name` under `limit`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SemaphoreRequest`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &SemaphoreRequest) -> $crate::option::Option<$crate::cmp::Ordering>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SemaphoreRequest) -> bool`
- **Ord**
  - `fn cmp(self: &Self, other: &SemaphoreRequest) -> $crate::cmp::Ordering`



## bookclerk_library::control_plane::semaphore::acquire_permit

*Function*

Takes one permit of `name` for `holder`, valid for `ttl`.

Tries the free or expired slots below `limit` in order; each attempt is
a fenced compare-and-swap, so a concurrent acquirer that wins a slot
makes this one move on. `holder` should be unique per piece of work
(it is also how a renewal proves ownership).

Live permits other holders keep on slots at or above `limit` (left over
from a higher limit) count against it, so after a limit drop no permit
is granted until the live count is under the new limit.

# Returns

The permit, or `None` when `limit` live permits are held by others.

# Errors

Returns an error when a read or write fails.

```rust
fn acquire_permit(store: &crate::store::LibraryStore, name: &str, limit: u32, holder: &str, ttl: std::time::Duration) -> crate::error::Result<Option<SemaphorePermit>>
```



## bookclerk_library::control_plane::semaphore::acquire_permits

*Function*

Takes one permit of every semaphore in `requests`, or none.

Requests are taken in name order (duplicates merge to the smallest
limit). When one semaphore is full, the permits already taken are
released and the blocked name is returned.

# Errors

Returns an error when a read or write fails; permits taken before the
failure are released on a best-effort basis.

```rust
fn acquire_permits(store: &crate::store::LibraryStore, requests: &[SemaphoreRequest], holder: &str, ttl: std::time::Duration) -> crate::error::Result<PermitAcquire>
```



## bookclerk_library::control_plane::semaphore::live_permits

*Function*

Live permits of `name` among its first `max_slots` slots (expired slots
are not counted).

# Errors

Returns an error when the read fails.

```rust
fn live_permits(store: &crate::store::LibraryStore, name: &str, max_slots: u32) -> crate::error::Result<Vec<SemaphorePermit>>
```



## bookclerk_library::control_plane::semaphore::permits_held_by

*Function*

Permits of `name` among its first `max_slots` slots whose holder starts
with `holder_prefix`, expired ones included.

A worker that holds permits only by name (a host that reconnected to
another owner node) finds them again with this. An expired slot nobody
took over is still returned, since [`renew_fenced_lease`](semaphore.md#bookclerk_librarycontrol_planesemaphorerenew_fenced_lease) still renews
it; a slot another holder took is not.

# Errors

Returns an error when the read fails.

```rust
fn permits_held_by(store: &crate::store::LibraryStore, name: &str, max_slots: u32, holder_prefix: &str) -> crate::error::Result<Vec<SemaphorePermit>>
```



## bookclerk_library::control_plane::semaphore::release_permits

*Function*

Gives back every permit in `permits` so the slots are free at once.

A permit whose fence was lost is left alone (the release matches no row).

# Errors

Returns an error when a write fails.

```rust
fn release_permits(store: &crate::store::LibraryStore, permits: &[SemaphorePermit]) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::semaphore::renew_fenced_lease

*Function*

Extends `lease` by `ttl` only while the row still carries its holder and
token. Unlike [`acquire_lease`](leases.md#bookclerk_librarycontrol_planeleasesacquire_lease), it never takes a lease over.

A lease that has expired but that nobody has taken is still renewed: the
unchanged token proves no other holder ran in between.

# Returns

`false` when another holder took the lease (the fence is lost).

# Errors

Returns an error when the write fails.

```rust
fn renew_fenced_lease(store: &crate::store::LibraryStore, lease: &super::ClusterLease, ttl: std::time::Duration) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::semaphore::renew_permits

*Function*

Extends every permit in `permits` by `ttl` while its holder and token
still own the slot.

# Returns

`false` when any permit was taken over (its fence is lost); the
remaining permits are still renewed.

# Errors

Returns an error when a write fails.

```rust
fn renew_permits(store: &crate::store::LibraryStore, permits: &[SemaphorePermit], ttl: std::time::Duration) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::semaphore::semaphore_slot_lease_name

*Function*

Lease name of slot `slot` of semaphore `name`.

```rust
fn semaphore_slot_lease_name(name: &str, slot: u32) -> String
```



