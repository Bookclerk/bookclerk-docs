**bookclerk_library > control_plane > job_routing**

# Module: control_plane::job_routing

## Contents

**Structs**

- [`JobClaimFilter`](#jobclaimfilter) - What a Host is willing to claim: the job kinds it runs and the plugin
- [`JobClaimLimits`](#jobclaimlimits) - Cluster-wide permit counts a routed claim enforces.
- [`JobClaimRequest`](#jobclaimrequest) - One routed claim attempt by one worker.
- [`RoutedJobClaim`](#routedjobclaim) - A claimed job and the semaphore permits it holds.

**Functions**

- [`account_semaphore`](#account_semaphore) - Name of the cluster semaphore for one store account.
- [`claim_routed_job`](#claim_routed_job) - Claims the best ready job this node can run, under the cluster semaphores.
- [`class_semaphore`](#class_semaphore) - Name of the cluster semaphore for one resource class.
- [`common_deployment_nodes`](#common_deployment_nodes) - Nodes where every instance in `requires` has a deployment that should run.
- [`instance_semaphore`](#instance_semaphore) - Name of the cluster semaphore for one plugin instance.
- [`job_permit_holder_prefix`](#job_permit_holder_prefix) - The holder prefix of the semaphore permits a routed claim of `job_id`
- [`job_plugin_keys`](#job_plugin_keys) - The plugin keys `job` runs, or `None` when the job alone does not say.
- [`job_semaphores`](#job_semaphores) - The cluster semaphores `job` counts against under `limits`.
- [`job_source_plugin_key`](#job_source_plugin_key) - The one plugin whose source guest runs `job`, or `None` when the job
- [`node_can_run_job`](#node_can_run_job) - True when `node_id` can run a job that requires `requires`.
- [`resolve_job_requirements`](#resolve_job_requirements) - The instances a new job must find on its claiming node.

**Constants**

- [`ACCOUNT_SEMAPHORE_LIMIT`](#account_semaphore_limit) - Permits an account semaphore grants: one job per store account at a time.
- [`ROUTED_CLAIM_CANDIDATES`](#routed_claim_candidates) - Pending candidates read per page of a routed claim attempt (the claim

---

## bookclerk_library::control_plane::job_routing::ACCOUNT_SEMAPHORE_LIMIT

*Constant*: `u32`

Permits an account semaphore grants: one job per store account at a time.



## bookclerk_library::control_plane::job_routing::JobClaimFilter

*Struct*

What a Host is willing to claim: the job kinds it runs and the plugin
keys its operator consented to. Both must admit a job.

**Fields:**
- `kinds: std::collections::BTreeSet<String>` - Job kinds (`scan`, `acquire`, ...).
- `plugin_keys: std::collections::BTreeSet<String>` - Canonical plugin keys.

**Methods:**

- `fn is_empty(self: &Self) -> bool` - True when the filter admits nothing.
- `fn admits(self: &Self, store: &LibraryStore, job: &JobRecord) -> Result<bool>` - True when `job` is of an admitted kind, names the account it runs

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> JobClaimFilter`
- **Clone**
  - `fn clone(self: &Self) -> JobClaimFilter`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobClaimFilter) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::job_routing::JobClaimLimits

*Struct*

Cluster-wide permit counts a routed claim enforces.

**Fields:**
- `class: u32` - Running jobs of the claimed resource class, cluster-wide.
- `plugin_instance: u32` - Running jobs that require one plugin instance, cluster-wide.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobClaimLimits`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &JobClaimLimits) -> bool`



## bookclerk_library::control_plane::job_routing::JobClaimRequest

*Struct*

One routed claim attempt by one worker.

**Generic Parameters:**
- 'a

**Fields:**
- `resource_class: crate::models::JobResourceClass` - Resource class the worker serves.
- `node_id: &'a str` - The claiming node (`node-identity.json`). Empty when unknown, in
- `claimant: super::trust::Claimant` - How the node reaches the queue; decides how an unknown node id is
- `owner: &'a str` - Worker lease owner written on the job row.
- `lease_secs: u64` - Lease length; permits are taken for the same time.
- `limits: JobClaimLimits` - Semaphore limits.
- `filter: Option<&'a JobClaimFilter>` - The claimer's own narrowing; `None` claims every job it is trusted

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JobClaimRequest<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::job_routing::ROUTED_CLAIM_CANDIDATES

*Constant*: `u64`

Pending candidates read per page of a routed claim attempt (the claim
pages on until it claims a job or the ready rows run out).



## bookclerk_library::control_plane::job_routing::RoutedJobClaim

*Struct*

A claimed job and the semaphore permits it holds.

**Fields:**
- `job: crate::models::JobRecord` - The running job (its fence is on the record).
- `permits: Vec<super::semaphore::SemaphorePermit>` - Permits to renew with the job heartbeat and release at the end.
- `resumed: bool` - True when this claim resumed a suspended job, so it spent no

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RoutedJobClaim`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::job_routing::account_semaphore

*Function*

Name of the cluster semaphore for one store account.

```rust
fn account_semaphore(plugin_key: &str, account_id: &str) -> String
```



## bookclerk_library::control_plane::job_routing::claim_routed_job

*Function*

Claims the best ready job this node can run, under the cluster semaphores.

Pages through the ready pending jobs of the class, best first
([`ROUTED_CLAIM_CANDIDATES`](job_routing.md#bookclerk_librarycontrol_planejob_routingrouted_claim_candidates) per page); skips those the node is not
trusted with ([`super::node_trust`](trust.md#bookclerk_librarycontrol_planetrustnode_trust)), those whose requirements this node
does not meet, and those whose account or instance semaphore is full;
takes the permits of the first one left and claims it with
`LibraryStore::claim_job_by_id` at the lease generation of the record
it evaluated, so a job whose requirements were rewritten after this node
checked them is not claimed on the stale check. The permit holder is
[`job_permit_holder_prefix`](job_routing.md#bookclerk_librarycontrol_planejob_routingjob_permit_holder_prefix) for the generation the claim runs at plus a
random suffix, so permits left over from an earlier claim of the same
job by the same owner never pass for this claim's. A lost claim gives
the permits back and moves on. It stops at the first claim, when the
rows run out, or when the class semaphore itself is full (no further
candidate could run).

# Returns

The claimed job and its permits, or `None` when nothing could be claimed.

# Errors

Returns an error when a read or write fails; permits taken for the
failing attempt are released on a best-effort basis.

```rust
fn claim_routed_job(store: &crate::store::LibraryStore, request: &JobClaimRequest) -> crate::error::Result<Option<RoutedJobClaim>>
```



## bookclerk_library::control_plane::job_routing::class_semaphore

*Function*

Name of the cluster semaphore for one resource class.

```rust
fn class_semaphore(class: crate::models::JobResourceClass) -> String
```



## bookclerk_library::control_plane::job_routing::common_deployment_nodes

*Function*

Nodes where every instance in `requires` has a deployment that should run.

# Errors

Returns an error when a read fails.

```rust
fn common_deployment_nodes(store: &crate::store::LibraryStore, requires: &[String]) -> crate::error::Result<std::collections::BTreeSet<String>>
```



## bookclerk_library::control_plane::job_routing::instance_semaphore

*Function*

Name of the cluster semaphore for one plugin instance.

```rust
fn instance_semaphore(instance_id: &str) -> String
```



## bookclerk_library::control_plane::job_routing::job_permit_holder_prefix

*Function*

The holder prefix of the semaphore permits a routed claim of `job_id`
by `owner` takes when the claim runs at lease `generation`:
`<owner>/<job_id>/g<generation>/`.

Every permit of that claim's holder starts with it (a random suffix
follows), and the trailing `/` keeps one generation's prefix from
matching another's (`g1/` is not a prefix of `g10/`). A worker that has
to find the claim's permits again by name, such as a Host renewing
through another owner node, matches on this prefix, so a still-live
permit left over from an earlier claim of the same job is never taken
for the current one.

```rust
fn job_permit_holder_prefix(owner: &str, job_id: &str, generation: i64) -> String
```



## bookclerk_library::control_plane::job_routing::job_plugin_keys

*Function*

The plugin keys `job` runs, or `None` when the job alone does not say.

Only a scan or acquire says: the plugin of the instance it is pinned to
and of each instance it requires, plus every plugin holding an
`accounts` row with the account it names (within the pinned plugin when
pinned). An unpinned acquire that names no account runs the plugins of
the library rows it matches. An unpinned scan of every account, an
acquire matching no row, any other kind, and a job naming an unknown
instance or account answer `None`.

# Errors

Returns an error when a read fails.

```rust
fn job_plugin_keys(store: &crate::store::LibraryStore, job: &crate::models::JobRecord) -> crate::error::Result<Option<std::collections::BTreeSet<String>>>
```



## bookclerk_library::control_plane::job_routing::job_semaphores

*Function*

The cluster semaphores `job` counts against under `limits`.

Every job takes its class semaphore and one per required instance. Scan
and acquire also take one per store account they touch: the payload's
account when it names one (limited to the pinned instance's plugin when
pinned), otherwise every account of the pinned plugin, or every account.

# Errors

Returns an error when a read fails.

```rust
fn job_semaphores(store: &crate::store::LibraryStore, job: &crate::models::JobRecord, limits: JobClaimLimits) -> crate::error::Result<Vec<super::semaphore::SemaphoreRequest>>
```



## bookclerk_library::control_plane::job_routing::job_source_plugin_key

*Function*

The one plugin whose source guest runs `job`, or `None` when the job
alone does not name exactly one.

Only a scan or acquire has one: the plugin of the instance it is pinned
to, or else the one plugin holding an `accounts` row the account it
names resolves to (by id or label, ignoring ASCII case). Required
instances (`crate::JobPayload::requires`, the destinations an acquire
writes) never count. An unpinned job of every account, or one whose
account name matches rows under several plugins, answers
`None`, as do other kinds and unknown instances or accounts.

# Errors

Returns an error when a read fails.

```rust
fn job_source_plugin_key(store: &crate::store::LibraryStore, job: &crate::models::JobRecord) -> crate::error::Result<Option<String>>
```



## bookclerk_library::control_plane::job_routing::node_can_run_job

*Function*

True when `node_id` can run a job that requires `requires`.

Each instance needs a deployment on the node that should run and an
observation from the node with the guest up. An empty list is always
satisfied; an unknown node or an unparsable id never is.

# Errors

Returns an error when a read fails.

```rust
fn node_can_run_job(store: &crate::store::LibraryStore, node_id: &str, requires: &[String]) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::job_routing::resolve_job_requirements

*Function*

The instances a new job must find on its claiming node.

`pinned` is the instance the job was pinned to; it is kept only when it
is a valid instance id with at least one deployment that should run, so a
job pinned to an undeployed instance is claimed anywhere and fails there
as before rather than waiting forever. For `JobKind::Acquire`, every
cluster instance of a plugin key in `node_bound_plugin_keys` that is
deployed to run somewhere is added: the acquire writes to that output, so
it must run where the output's disk is. A node-local instance (H-2,
`owner_node_id` set) is not: it adds to the outputs of its own node only,
which copies what it acquires to it, and requiring it would leave every
acquire claimable on that one node alone, or nowhere.

The required instances must share at least one node where all of them
are deployed to run; otherwise no node could ever claim the job, so it
is refused here instead of being left pending. Required instances are
never dropped to make a node fit: an acquire must write to every
deployed node-bound output.

# Errors

Returns `LibraryError::Conflict` when the required instances have no
common deployment node, or an error when a read fails.

```rust
fn resolve_job_requirements(store: &crate::store::LibraryStore, kind: crate::models::JobKind, payload: &crate::models::JobPayload, node_bound_plugin_keys: &std::collections::BTreeSet<String>) -> crate::error::Result<Vec<String>>
```



