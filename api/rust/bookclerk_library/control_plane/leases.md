**bookclerk_library > control_plane > leases**

# Module: control_plane::leases

## Contents

**Structs**

- [`ClusterLease`](#clusterlease) - A held cluster lease.
- [`LeaseFence`](#leasefence) - Proof, checked against the database, that this holder still holds a
- [`SingletonDuty`](#singletonduty) - One periodic duty that exactly one live node runs per interval.

**Functions**

- [`acquire_lease`](#acquire_lease) - Acquires or renews `name` for `holder` for `ttl`.
- [`acquire_lease_on`](#acquire_lease_on) - [`acquire_lease`](leases.md#bookclerk_librarycontrol_planeleasesacquire_lease) on a bare connection (schema migration runs before a
- [`list_leases`](#list_leases) - Every lease row, held or expired, by name.
- [`load_lease`](#load_lease) - The current row of lease `name`, held or expired.
- [`load_lease_on`](#load_lease_on) - [`load_lease`](leases.md#bookclerk_librarycontrol_planeleasesload_lease) on a bare connection.
- [`release_lease`](#release_lease) - Gives up a held lease so another holder may take it at once.
- [`release_lease_on`](#release_lease_on) - [`release_lease`](leases.md#bookclerk_librarycontrol_planeleasesrelease_lease) on a bare connection.
- [`renew_lease`](#renew_lease) - Fencing check: extends a lease this holder still holds.
- [`renew_lease_on`](#renew_lease_on) - [`renew_lease`](leases.md#bookclerk_librarycontrol_planeleasesrenew_lease) on a bare connection.

**Constants**

- [`EVENT_RETENTION_LEASE`](#event_retention_lease) - Lease of the event retention duty (terminal deliveries, empty events,
- [`JOB_RETENTION_LEASE`](#job_retention_lease) - Lease of the job retention duty (terminal `jobs` rows).
- [`KV_EXPIRY_LEASE`](#kv_expiry_lease) - Lease of the plugin KV expiry duty.
- [`MIGRATION_LEASE`](#migration_lease) - Lease a schema migration runs under (see
- [`SESSION_EXPIRY_LEASE`](#session_expiry_lease) - Lease of the session duty (expired operator sessions, OIDC RP states,

---

## bookclerk_library::control_plane::leases::ClusterLease

*Struct*

A held cluster lease.

**Fields:**
- `name: String` - Lease name.
- `holder: String` - Holder (a node id).
- `token: i64` - Fencing token; a new holder always gets a larger one.
- `expires_at_ms: i64` - Expiry in Unix milliseconds.

**Methods:**

- `fn is_held_at(self: &Self, now_ms: i64) -> bool` - True when the lease has not expired at `now_ms`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ClusterLease`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ClusterLease) -> bool`



## bookclerk_library::control_plane::leases::EVENT_RETENTION_LEASE

*Constant*: `&str`

Lease of the event retention duty (terminal deliveries, empty events,
stale subscriber catalog rows).



## bookclerk_library::control_plane::leases::JOB_RETENTION_LEASE

*Constant*: `&str`

Lease of the job retention duty (terminal `jobs` rows).



## bookclerk_library::control_plane::leases::KV_EXPIRY_LEASE

*Constant*: `&str`

Lease of the plugin KV expiry duty.



## bookclerk_library::control_plane::leases::LeaseFence

*Struct*

Proof, checked against the database, that this holder still holds a
lease. Call `LeaseFence::check` before each destructive step.

**Generic Parameters:**
- 'a

**Methods:**

- `fn new(store: &'a LibraryStore, lease: ClusterLease, ttl: Duration) -> Self` - A fence for a lease acquired with `ttl`.
- `fn lease(self: &Self) -> &ClusterLease` - The lease as of the last successful check.
- `fn check(self: & mut Self) -> Result<()>` - Renews the lease, or fails when this holder no longer holds it.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::leases::MIGRATION_LEASE

*Constant*: `&str`

Lease a schema migration runs under (see
[`crate::apply_host_schema_with_batch_opts`](../host_schema.md#bookclerk_libraryhost_schemaapply_host_schema_with_batch_opts)).



## bookclerk_library::control_plane::leases::SESSION_EXPIRY_LEASE

*Constant*: `&str`

Lease of the session duty (expired operator sessions, OIDC RP states,
WebAuthn challenges).



## bookclerk_library::control_plane::leases::SingletonDuty

*Struct*

One periodic duty that exactly one live node runs per interval.

Every node keeps one per duty and calls `SingletonDuty::poll` more
often than the lease TTL. `poll` acquires or renews the lease and hands
back a [`LeaseFence`](leases.md#bookclerk_librarycontrol_planeleasesleasefence) only when this node holds it and the duty is due:
at once after taking the lease, then every `interval`.

**Methods:**

- `fn new<impl Into<String>, impl Into<String>>(lease_name: impl Trait, holder: impl Trait, ttl: Duration, interval: Duration) -> Self` - A duty under lease `lease_name` for `holder`.
- `fn lease_name(self: &Self) -> &str` - The lease name.
- `fn poll<'a>(self: & mut Self, store: &'a LibraryStore) -> Result<Option<LeaseFence<'a>>>` - Takes or renews the lease; returns a fence when the duty should run
- `fn mark_ran(self: & mut Self)` - Records that the duty ran (successfully or not) so the next run waits

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SingletonDuty`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::leases::acquire_lease

*Function*

Acquires or renews `name` for `holder` for `ttl`.

Returns `None` when another holder's lease has not expired, or when a
concurrent acquire won. Renewing an unexpired lease keeps the token;
taking an expired (or released) lease bumps it, even when `holder` is
the one that held it.

# Errors

Returns an error when the read or write fails.

```rust
fn acquire_lease(store: &crate::store::LibraryStore, name: &str, holder: &str, ttl: std::time::Duration) -> crate::error::Result<Option<ClusterLease>>
```



## bookclerk_library::control_plane::leases::acquire_lease_on

*Function*

[`acquire_lease`](leases.md#bookclerk_librarycontrol_planeleasesacquire_lease) on a bare connection (schema migration runs before a
[`LibraryStore`](../store.md#bookclerk_librarystorelibrarystore) exists).

# Errors

Returns an error when the read or write fails.

```rust
fn acquire_lease_on<C>(db: &C, name: &str, holder: &str, ttl: std::time::Duration) -> crate::error::Result<Option<ClusterLease>>
```



## bookclerk_library::control_plane::leases::list_leases

*Function*

Every lease row, held or expired, by name.

# Errors

Returns an error when the read fails.

```rust
fn list_leases(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<ClusterLease>>
```



## bookclerk_library::control_plane::leases::load_lease

*Function*

The current row of lease `name`, held or expired.

# Errors

Returns an error when the read fails.

```rust
fn load_lease(store: &crate::store::LibraryStore, name: &str) -> crate::error::Result<Option<ClusterLease>>
```



## bookclerk_library::control_plane::leases::load_lease_on

*Function*

[`load_lease`](leases.md#bookclerk_librarycontrol_planeleasesload_lease) on a bare connection.

# Errors

Returns an error when the read fails.

```rust
fn load_lease_on<C>(db: &C, name: &str) -> crate::error::Result<Option<ClusterLease>>
```



## bookclerk_library::control_plane::leases::release_lease

*Function*

Gives up a held lease so another holder may take it at once.

The row stays (holder and token remain visible to status reads) with an
expiry of zero.

# Errors

Returns an error when the write fails.

```rust
fn release_lease(store: &crate::store::LibraryStore, lease: &ClusterLease) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::leases::release_lease_on

*Function*

[`release_lease`](leases.md#bookclerk_librarycontrol_planeleasesrelease_lease) on a bare connection.

# Errors

Returns an error when the write fails.

```rust
fn release_lease_on<C>(db: &C, lease: &ClusterLease) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::leases::renew_lease

*Function*

Fencing check: extends a lease this holder still holds.

Succeeds only while the row names `lease.holder` and `lease.token` and
has not expired. Returns the extended lease, or `None` when the lease
expired or another holder took it; the caller must then stop before any
further destructive work.

# Errors

Returns an error when the write fails.

```rust
fn renew_lease(store: &crate::store::LibraryStore, lease: &ClusterLease, ttl: std::time::Duration) -> crate::error::Result<Option<ClusterLease>>
```



## bookclerk_library::control_plane::leases::renew_lease_on

*Function*

[`renew_lease`](leases.md#bookclerk_librarycontrol_planeleasesrenew_lease) on a bare connection.

# Errors

Returns an error when the write fails.

```rust
fn renew_lease_on<C>(db: &C, lease: &ClusterLease, ttl: std::time::Duration) -> crate::error::Result<Option<ClusterLease>>
```



