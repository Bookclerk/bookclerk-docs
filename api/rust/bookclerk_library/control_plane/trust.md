**bookclerk_library > control_plane > trust**

# Module: control_plane::trust

## Contents

**Enums**

- [`Claimant`](#claimant) - How the claiming node reaches the job queue.
- [`NodeTrust`](#nodetrust) - What a node is trusted with, resolved from the database.
- [`ScanRequester`](#scanrequester) - Who asked for a scan.

**Functions**

- [`accounts_owned_by`](#accounts_owned_by) - Store accounts linked to `user_id` through any of the user's identities.
- [`book_matches_title`](#book_matches_title) - True when `id` names `book` (uuid, product id, ISBN or ASIN, any case),
- [`claimant_may_run`](#claimant_may_run) - True when a `claimant` may run `job` at all, whatever it is trusted
- [`handoff_routed_job`](#handoff_routed_job) - Re-checks a claimed job just before its secrets are handed to the
- [`job_item_accounts`](#job_item_accounts) - The store accounts a scan or acquire touches: its items.
- [`job_trusted`](#job_trusted) - True when a node with `trust` may receive `job`.
- [`node_may_claim_job`](#node_may_claim_job) - True when `node_id` may claim `job` right now.
- [`node_trust`](#node_trust) - Resolves what `node_id` is trusted with right now.
- [`plan_acquire`](#plan_acquire) - Plans the acquire jobs one request admits.
- [`plan_scan`](#plan_scan) - Plans the scan jobs one request admits.
- [`remote_node_may_view_job`](#remote_node_may_view_job) - True when a remote node may read `job`'s state.
- [`remote_node_requester`](#remote_node_requester) - Whom library work a remote node asks for runs as (a `scan.request` over

**Type Aliases**

- [`AccountKey`](#accountkey) - A store account, keyed as in `accounts`: `(plugin_key, account_id)`.

---

## bookclerk_library::control_plane::trust::AccountKey

*Type Alias*: `(String, String)`

A store account, keyed as in `accounts`: `(plugin_key, account_id)`.



## bookclerk_library::control_plane::trust::Claimant

*Enum*

How the claiming node reaches the job queue.

**Variants:**
- `Local` - A worker in this process, which holds the library database itself.
- `Remote` - A node that authenticated over the node API. It must have a `nodes`

**Traits:** Copy, Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> Claimant`
- **Clone**
  - `fn clone(self: &Self) -> Claimant`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &Claimant) -> bool`



## bookclerk_library::control_plane::trust::NodeTrust

*Enum*

What a node is trusted with, resolved from the database.

**Variants:**
- `All` - Every job: an owner node, or a Host owned by an active Owner or
- `OwnedBy(i64)` - Only jobs whose secrets all belong to this user.
- `Nothing` - No job at all.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeTrust) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NodeTrust`



## bookclerk_library::control_plane::trust::ScanRequester

*Enum*

Who asked for a scan.

**Variants:**
- `Operator` - Operator authority: the operator token or session, the daemon's own
- `User(i64)` - A signed-in user; their role is read from the database.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScanRequester) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ScanRequester`



## bookclerk_library::control_plane::trust::accounts_owned_by

*Function*

Store accounts linked to `user_id` through any of the user's identities.

# Errors

Returns an error when a read fails.

```rust
fn accounts_owned_by(store: &crate::store::LibraryStore, user_id: i64) -> crate::error::Result<std::collections::BTreeSet<AccountKey>>
```



## bookclerk_library::control_plane::trust::book_matches_title

*Function*

True when `id` names `book` (uuid, product id, ISBN or ASIN, any case),
as an acquire's title filter matches.

```rust
fn book_matches_title(book: &crate::models::BookRecord, id: &str) -> bool
```



## bookclerk_library::control_plane::trust::claimant_may_run

*Function*

True when a `claimant` may run `job` at all, whatever it is trusted
with: a forced or PDF-only acquire runs only on a node that holds the
library database, since the host acquire path does neither.

```rust
fn claimant_may_run(claimant: Claimant, job: &crate::models::JobRecord) -> bool
```



## bookclerk_library::control_plane::trust::handoff_routed_job

*Function*

Re-checks a claimed job just before its secrets are handed to the
claiming node.

The claim is returned unchanged only when both hold:

- the claim still owns the job: the row is `running` under the claim's
  fence (same lease owner and generation), that owner is
  `request.owner`, and the lease has not expired. A claim that was
  reclaimed, finished, released, or left to expire is not handed over;
- the node is still trusted with the job ([`node_may_claim_job`](trust.md#bookclerk_librarycontrol_planetrustnode_may_claim_job)).

Ownership is checked again after the trust reads, so a lease lost while
they ran is refused too.

Otherwise `None` is returned, so nothing is unsealed. When the claim was
taken by `request.owner`, the job goes back to `pending` without spending
an attempt if the claim's fence still owns the row (a row another fence
owns is left alone), and the claim's permits are released (a permit taken
over since holds a new token and is left alone too). A claim with another
lease owner is refused without touching the row or its permits.

# Errors

Returns an error when a read or write fails. A failure while releasing
leaves the lease to expire, after which the job is redelivered.

```rust
fn handoff_routed_job(store: &crate::store::LibraryStore, request: &super::job_routing::JobClaimRequest, claim: super::job_routing::RoutedJobClaim) -> crate::error::Result<Option<super::job_routing::RoutedJobClaim>>
```



## bookclerk_library::control_plane::trust::job_item_accounts

*Function*

The store accounts a scan or acquire touches: its items.

- A scan that names an account touches every `accounts` row with that
  id (within the pinned instance's plugin). A scan of every account is
  `None`: what it touches is not known until it runs.
- An acquire touches the named account's rows, plus the account of every
  library row its filters match (title id, account, pinned plugin),
  whatever the row's acquire status. That is a superset of the titles it
  will fetch, so a node must be trusted with at least everything it
  fetches.
- Anything else, and a job pinned to an unknown instance, is `None`.

# Errors

Returns an error when a read fails.

```rust
fn job_item_accounts(store: &crate::store::LibraryStore, job: &crate::models::JobRecord) -> crate::error::Result<Option<std::collections::BTreeSet<AccountKey>>>
```



## bookclerk_library::control_plane::trust::job_trusted

*Function*

True when a node with `trust` may receive `job`.

A node trusted only with one user's work receives a scan or acquire
that names an account (`owned_job_account`) only when every store
account the name reaches is linked to that user.

# Errors

Returns an error when a read fails.

```rust
fn job_trusted(store: &crate::store::LibraryStore, trust: &NodeTrust, job: &crate::models::JobRecord) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::trust::node_may_claim_job

*Function*

True when `node_id` may claim `job` right now.

# Errors

Returns an error when a read fails.

```rust
fn node_may_claim_job(store: &crate::store::LibraryStore, node_id: &str, claimant: Claimant, job: &crate::models::JobRecord) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::trust::node_trust

*Function*

Resolves what `node_id` is trusted with right now.

# Errors

Returns an error when a read fails.

```rust
fn node_trust(store: &crate::store::LibraryStore, node_id: &str, claimant: Claimant) -> crate::error::Result<NodeTrust>
```



## bookclerk_library::control_plane::trust::plan_acquire

*Function*

Plans the acquire jobs one request admits.

`titles` are title ids (uuid, ASIN, ISBN, product id); each becomes its
own job, so each title can go to any node trusted with it. `account`
narrows to one store account and `plugin_instance_id` pins every job.

- Operator authority, an Owner or an Administrator: one job per title,
  or one job for everything pending (it touches many accounts, and only
  nodes trusted with every one of them claim it; splitting it per
  account would repeat the storage listing for every account).
- Any other active user: a named account must be theirs, and so must
  every library row a named title matches. A title named without an
  account becomes one job per account id it is in, since a node trusted
  only with its owner's work takes only jobs that name an account. With
  neither, the request becomes one job per account they wholly own, so
  each can go to a node trusted with it; the plan is empty when they own
  none.

# Errors

Returns `LibraryError::Forbidden` when the user is unknown or disabled,
or names an account or title that is not theirs (unknown ones read the
same), `LibraryError::Invalid` for a malformed instance id, and the
read error when a read fails.

```rust
fn plan_acquire(store: &crate::store::LibraryStore, requester: ScanRequester, account: Option<&str>, titles: &[String], plugin_instance_id: Option<&str>) -> crate::error::Result<Vec<crate::models::JobPayload>>
```



## bookclerk_library::control_plane::trust::plan_scan

*Function*

Plans the scan jobs one request admits.

`account` names one store account; `plugin_instance_id` pins the scan to
one instance (and so to its plugin's accounts).

- Operator authority, an Owner or an Administrator: a named account is
  one job as asked. A scan of every account becomes one job per distinct
  scan-enabled account id (within the pinned plugin), so the planner can
  route each to any node trusted with that account. When the id also
  names a scan-disabled account on another plugin, its job is pinned to
  the default instance of each plugin where it is scan-enabled, so the
  disabled account is never touched. With no scan-enabled
  account the request stays one unsplit job, which only fully trusted
  nodes may claim and which reports that nothing was scanned.
- Any other active user: a named account must be theirs, and a scan of
  every account covers only the scan-enabled accounts they own. An
  account id that another user also holds on a different plugin is left
  out, since its job would touch both. The plan is empty when the user
  owns nothing to scan.

# Errors

Returns `LibraryError::Forbidden` when the user is unknown or disabled,
or names an account that is not theirs (an unknown account reads the
same), `LibraryError::Invalid` for a malformed instance id, and the
read error when a read fails.

```rust
fn plan_scan(store: &crate::store::LibraryStore, requester: ScanRequester, account: Option<&str>, plugin_instance_id: Option<&str>) -> crate::error::Result<Vec<crate::models::JobPayload>>
```



## bookclerk_library::control_plane::trust::remote_node_may_view_job

*Function*

True when a remote node may read `job`'s state.

A node trusted with everything reads every job. A node trusted only
with its owner's work reads any scan or acquire that names an account
and touches only its owner's accounts ([`job_item_accounts`](trust.md#bookclerk_librarycontrol_planetrustjob_item_accounts), within a
pinned job's plugin), including a pinned one it could not
claim itself, since its owner may queue those through it
([`plan_scan`](trust.md#bookclerk_librarycontrol_planetrustplan_scan)). A node trusted with nothing reads none.

# Errors

Returns an error when a read fails.

```rust
fn remote_node_may_view_job(store: &crate::store::LibraryStore, node_id: &str, job: &crate::models::JobRecord) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::trust::remote_node_requester

*Function*

Whom library work a remote node asks for runs as (a `scan.request` over
the node API): the user who owns the node, while the node is trusted
with anything. `None` for a node that is trusted with nothing (revoked,
ownerless, a disabled owner) or unknown, which may ask for nothing.

# Errors

Returns an error when a read fails.

```rust
fn remote_node_requester(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<Option<ScanRequester>>
```



