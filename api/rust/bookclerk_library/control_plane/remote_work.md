**bookclerk_library > control_plane > remote_work**

# Module: control_plane::remote_work

## Contents

**Structs**

- [`JobSecret`](#jobsecret) - One unsealed secret handed with a job. The value is zeroized on drop and
- [`JobSecretRef`](#jobsecretref) - Names one store-account secret a job is handed. Carries no secret
- [`RemoteClaimSettings`](#remoteclaimsettings) - What a Host's claim settles on: the routing and lease settings of the
- [`RemoteJobClaim`](#remotejobclaim) - A job claimed for a Host, with the secrets to seal for it.

**Enums**

- [`RemoteFinish`](#remotefinish) - Outcome of [`finish_remote_job`](remote_work.md#bookclerk_librarycontrol_planeremote_workfinish_remote_job).
- [`RemoteOutcome`](#remoteoutcome) - How a Host's job ended.
- [`RemoteRenew`](#remoterenew) - Outcome of [`renew_remote_job`](remote_work.md#bookclerk_librarycontrol_planeremote_workrenew_remote_job).
- [`WriteBack`](#writeback) - Outcome of [`write_back_job_secret`](remote_work.md#bookclerk_librarycontrol_planeremote_workwrite_back_job_secret).

**Functions**

- [`claim_remote_job`](#claim_remote_job) - Claims the best job `node_id` may run and unseals its secrets.
- [`finish_remote_job`](#finish_remote_job) - Completes or fails the job `node_id` holds at `generation`, then gives
- [`job_secret_refs`](#job_secret_refs) - The store-account secrets `job` needs on a Host.
- [`live_remote_job`](#live_remote_job) - The job `node_id` holds at `generation`, while that fence is the job's
- [`release_remote_job`](#release_remote_job) - Gives back the job `node_id` holds at `generation` without running it:
- [`remote_job_fence`](#remote_job_fence) - The fence a Host holds for `job_id` at lease generation `generation`.
- [`remote_lease_owner`](#remote_lease_owner) - Lease owner of every job `node_id` runs over the node API.
- [`renew_remote_job`](#renew_remote_job) - Extends the lease of the job `node_id` holds at `generation`, and its
- [`write_back_job_secret`](#write_back_job_secret) - Stores a refreshed store token a Host writes back.

**Constants**

- [`MAX_WRITE_BACK_BYTES`](#max_write_back_bytes) - Largest store token a Host may write back.
- [`REMOTE_LEASE_OWNER_PREFIX`](#remote_lease_owner_prefix) - Prefix of the lease owner of a job a Host runs.
- [`SECRETS_UNAVAILABLE`](#secrets_unavailable) - Error kind recorded when a job's secrets cannot be unsealed for a Host.

---

## bookclerk_library::control_plane::remote_work::JobSecret

*Struct*

One unsealed secret handed with a job. The value is zeroized on drop and
never shown by `Debug`.

**Fields:**
- `secret_ref: JobSecretRef` - Which row the value belongs to.
- `value: zeroize::Zeroizing<Vec<u8>>` - The unsealed bytes.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> JobSecret`



## bookclerk_library::control_plane::remote_work::JobSecretRef

*Struct*

Names one store-account secret a job is handed. Carries no secret
material.

**Fields:**
- `kind: String` - Secret kind (`source_auth`, `widevine`, ...).
- `provider: String` - Plugin key that owns the secret.
- `account_id: String` - Store account id.
- `name: String` - Logical secret name.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobSecretRef) -> bool`
- **Ord**
  - `fn cmp(self: &Self, other: &JobSecretRef) -> $crate::cmp::Ordering`
- **Clone**
  - `fn clone(self: &Self) -> JobSecretRef`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &JobSecretRef) -> $crate::option::Option<$crate::cmp::Ordering>`



## bookclerk_library::control_plane::remote_work::MAX_WRITE_BACK_BYTES

*Constant*: `usize`

Largest store token a Host may write back.



## bookclerk_library::control_plane::remote_work::REMOTE_LEASE_OWNER_PREFIX

*Constant*: `&str`

Prefix of the lease owner of a job a Host runs.



## bookclerk_library::control_plane::remote_work::RemoteClaimSettings

*Struct*

What a Host's claim settles on: the routing and lease settings of the
owner node serving it.

**Fields:**
- `lease_secs: u64` - Lease length; permits are taken for the same time.
- `limits: super::job_routing::JobClaimLimits` - Cluster semaphore limits.

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RemoteClaimSettings`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::remote_work::RemoteFinish

*Enum*

Outcome of [`finish_remote_job`](remote_work.md#bookclerk_librarycontrol_planeremote_workfinish_remote_job).

**Variants:**
- `Done` - The job row was finalized and its permits given back.
- `Lost` - The fence no longer owns the job.
- `Refused(&'static str)` - The request is not allowed for this job (the reason is stable).

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RemoteFinish`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteFinish) -> bool`



## bookclerk_library::control_plane::remote_work::RemoteJobClaim

*Struct*

A job claimed for a Host, with the secrets to seal for it.

**Fields:**
- `job: crate::models::JobRecord` - The running job (its fence is on the record).
- `secrets: Vec<JobSecret>` - The account secrets the job needs, unsealed.
- `plugin_keys: Vec<String>` - The plugin keys the job runs ([`job_plugin_keys`](job_routing.md#bookclerk_librarycontrol_planejob_routingjob_plugin_keys)); empty when the
- `source_plugin_key: Option<String>` - The one plugin whose source guest runs the job
- `resumed: bool` - The claim resumed a suspended job (`RoutedJobClaim::resumed`).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::remote_work::RemoteOutcome

*Enum*

How a Host's job ended.

**Generic Parameters:**
- 'a

**Variants:**
- `Complete(Option<&'a str>)` - The job succeeded; the text is recorded as its progress.
- `Fail{ kind: &'a str, message: &'a str }` - The job failed.

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RemoteOutcome<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::remote_work::RemoteRenew

*Enum*

Outcome of [`renew_remote_job`](remote_work.md#bookclerk_librarycontrol_planeremote_workrenew_remote_job).

**Variants:**
- `Renewed{ cancel: bool }` - The lease and its permits were extended.
- `Lost` - The fence no longer owns the job, a permit was taken over, or the

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RemoteRenew) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> RemoteRenew`



## bookclerk_library::control_plane::remote_work::SECRETS_UNAVAILABLE

*Constant*: `&str`

Error kind recorded when a job's secrets cannot be unsealed for a Host.



## bookclerk_library::control_plane::remote_work::WriteBack

*Enum*

Outcome of [`write_back_job_secret`](remote_work.md#bookclerk_librarycontrol_planeremote_workwrite_back_job_secret).

**Variants:**
- `Written` - The secret was re-sealed and stored.
- `Refused(&'static str)` - Refused; nothing was written (the reason is stable).

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> WriteBack`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &WriteBack) -> bool`



## bookclerk_library::control_plane::remote_work::claim_remote_job

*Function*

Claims the best job `node_id` may run and unseals its secrets.

The claim is [`claim_routed_job`](job_routing.md#bookclerk_librarycontrol_planejob_routingclaim_routed_job) as a `Claimant::Remote` with the
lease owner [`remote_lease_owner`](remote_work.md#bookclerk_librarycontrol_planeremote_workremote_lease_owner); [`handoff_routed_job`](trust.md#bookclerk_librarycontrol_planetrusthandoff_routed_job) then re-checks
trust, and only then are the job's account secrets
([`job_secret_refs`](remote_work.md#bookclerk_librarycontrol_planeremote_workjob_secret_refs)) unsealed. The refs handed are recorded under the
claim's lease generation before the secrets are returned, and are the
only ones [`write_back_job_secret`](remote_work.md#bookclerk_librarycontrol_planeremote_workwrite_back_job_secret) accepts under that fence. A job whose
secrets cannot be unsealed is failed with [`SECRETS_UNAVAILABLE`](remote_work.md#bookclerk_librarycontrol_planeremote_worksecrets_unavailable) (so it
retries with backoff and any node holding the database may take it) and
`None` is returned.

`filter` is the Host's own narrowing ([`JobClaimFilter`](job_routing.md#bookclerk_librarycontrol_planejob_routingjobclaimfilter)): only jobs of
its kinds whose every plugin it consented to, and whose one source
plugin can be told, are claimed.

# Errors

Returns an error when a read or write fails. Permits taken for a failed
attempt are released, and a claim whose plugin keys could not be read
or whose handoff could not be recorded is given back unspent (permits
included), on a best-effort basis.

```rust
fn claim_remote_job(store: &crate::store::LibraryStore, node_id: &str, resource_class: crate::models::JobResourceClass, settings: RemoteClaimSettings, filter: Option<&super::job_routing::JobClaimFilter>) -> crate::error::Result<Option<RemoteJobClaim>>
```



## bookclerk_library::control_plane::remote_work::finish_remote_job

*Function*

Completes or fails the job `node_id` holds at `generation`, then gives
its semaphore permits back.

A failure is always accepted under a live fence, so the job retries
promptly. A completion also needs the node to still be trusted with the
job, and is refused for an acquire, which completes only when its
upload is committed.

# Errors

Returns an error when a read or write fails.

```rust
fn finish_remote_job(store: &crate::store::LibraryStore, node_id: &str, job_id: &str, generation: i64, limits: super::job_routing::JobClaimLimits, outcome: RemoteOutcome) -> crate::error::Result<RemoteFinish>
```



## bookclerk_library::control_plane::remote_work::job_secret_refs

*Function*

The store-account secrets `job` needs on a Host.

For a scan or acquire that names an account: every `integration` row of
`encrypted_secrets` for each store account the name reaches, by id or
label and ignoring case, as the executor resolves it (the same accounts
the trust predicate checked), limited to the pinned instance's plugin
when the job is pinned to one. Every other job is handed no secrets.
Sorted, without duplicates.

# Errors

Returns an error when a read fails.

```rust
fn job_secret_refs(store: &crate::store::LibraryStore, job: &crate::models::JobRecord) -> crate::error::Result<Vec<JobSecretRef>>
```



## bookclerk_library::control_plane::remote_work::live_remote_job

*Function*

The job `node_id` holds at `generation`, while that fence is the job's
live lease and the node is still trusted with the job; `None` when it
is lost.

# Errors

Returns an error when a read fails.

```rust
fn live_remote_job(store: &crate::store::LibraryStore, node_id: &str, job_id: &str, generation: i64) -> crate::error::Result<Option<crate::models::JobRecord>>
```



## bookclerk_library::control_plane::remote_work::release_remote_job

*Function*

Gives back the job `node_id` holds at `generation` without running it:
the job returns to `pending`, and its semaphore permits and handoff rows
are released.

For a Host whose operator no longer consents to the job's plugin, and
for a `work` answer that cannot reach the Host (its connection was
superseded or is closing), so the job is claimable again at once instead
of only after its lease expires. `resumed` is the claim's
[`RemoteJobClaim::resumed`]: a resumed claim gets its resume flag back,
any other its attempt.

The attempt is not spent, unless the job carries a checkpoint (it may
have resumed, which spent none, or saved progress under this claim), in
which case it is kept as spent and the job resumes from the checkpoint.
Nothing is refunded twice: a release never lowers the count below what
the claim raised it by. The release is accepted from a node that is no
longer trusted, since it only gives work back.

# Returns

`RemoteFinish::Done` when the claim was given back;
`RemoteFinish::Lost` when the fence no longer owns the job.

# Errors

Returns an error when a read or write fails.

```rust
fn release_remote_job(store: &crate::store::LibraryStore, node_id: &str, job_id: &str, generation: i64, resumed: bool, limits: super::job_routing::JobClaimLimits) -> crate::error::Result<RemoteFinish>
```



## bookclerk_library::control_plane::remote_work::remote_job_fence

*Function*

The fence a Host holds for `job_id` at lease generation `generation`.

```rust
fn remote_job_fence(node_id: &str, job_id: &str, generation: i64) -> crate::models::JobFence
```



## bookclerk_library::control_plane::remote_work::remote_lease_owner

*Function*

Lease owner of every job `node_id` runs over the node API.

```rust
fn remote_lease_owner(node_id: &str) -> String
```



## bookclerk_library::control_plane::remote_work::renew_remote_job

*Function*

Extends the lease of the job `node_id` holds at `generation`, and its
semaphore permits, recording `progress` when given.

Trust and the node credential are re-read first: a node that is no
longer trusted with the job (revoked, owner demoted, account unlinked)
gets `RemoteRenew::Lost`, and the lease then expires as for a crashed
worker. A fence that expired and was reclaimed is lost as well, as is a
job whose permit another holder took over. Permits are checked before
the job lease is extended, so a renewal that answers `Lost` never pushes
the reclaim out; should a permit be taken over between that check and
its renewal, the just-extended lease is cut back to expire at once.

# Errors

Returns an error when a read or write fails.

```rust
fn renew_remote_job(store: &crate::store::LibraryStore, node_id: &str, job_id: &str, generation: i64, settings: RemoteClaimSettings, progress: Option<&str>) -> crate::error::Result<RemoteRenew>
```



## bookclerk_library::control_plane::remote_work::write_back_job_secret

*Function*

Stores a refreshed store token a Host writes back.

Accepted only when all of these hold, checked immediately before the
write: the node still has a live credential and is still trusted with
the job; `generation` is the job's live lease generation for this node;
`secret_ref` was recorded as handed with the job under that generation
(a secret created for the account after the handoff is refused); and it
is still one of the job's secrets ([`job_secret_refs`](remote_work.md#bookclerk_librarycontrol_planeremote_workjob_secret_refs)), so it still
exists and its account is still linked. The value is re-sealed with this
node's DEK under the same identity.

# Errors

Returns an error when a read, the seal, or the write fails.

```rust
fn write_back_job_secret(store: &crate::store::LibraryStore, node_id: &str, job_id: &str, generation: i64, secret_ref: &JobSecretRef, value: &[u8]) -> crate::error::Result<WriteBack>
```



