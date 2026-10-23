**bookclerkd > cluster_link**

# Module: cluster_link

## Contents

**Modules**

- [`api`](#api) - Operator API for host consent (H-4 design, step 8):
- [`catalog_sync`](#catalog_sync) - On a branch: sharing this library's books with the library it joined
- [`consent`](#consent) - Host consent for owner plugins, per cluster link (H-4 design, section 5;
- [`host_jobs`](#host_jobs) - How a host runs a job an owner cluster handed it (H-4 step 9).
- [`join`](#join) - Operator API for joining a library from the web UI, and for leaving one
- [`remote_bindings`](#remote_bindings) - What a host node knows of the instances an owner cluster deployed to

**Structs**

- [`TransportRunner`](#transportrunner) - Runs jobs an owner cluster handed this host through a [`JobTransport`](job_handler.md#bookclerkdjob_handlerjobtransport),

**Functions**

- [`leave_link`](#leave_link) - Leave library: tells the owner (`node.leaving`, through the running
- [`spawn_cluster_links`](#spawn_cluster_links) - Starts the cluster-link supervisor: one host client per active link of
- [`start_link_now`](#start_link_now) - Starts a client for `cluster_id` now (a join through this daemon), with

**Constants**

- [`HOST_MAX_JOBS`](#host_max_jobs) - Jobs a host client holds at once.
- [`HOST_RUNNABLE_KINDS`](#host_runnable_kinds) - Job kinds a host claims: a scan returns its rows in fenced results
- [`INVALID_HANDED_JOB`](#invalid_handed_job) - Error kind for a handed job this daemon cannot decode.
- [`LEAVE_NOTICE_TIMEOUT`](#leave_notice_timeout) - How long Leave library waits for the owner to answer `node.leaving`.

---

## bookclerkd::cluster_link::HOST_MAX_JOBS

*Constant*: `usize`

Jobs a host client holds at once.



## bookclerkd::cluster_link::HOST_RUNNABLE_KINDS

*Constant*: `&[&str]`

Job kinds a host claims: a scan returns its rows in fenced results
batches, an acquire streams its files to the owner.



## bookclerkd::cluster_link::INVALID_HANDED_JOB

*Constant*: `&str`

Error kind for a handed job this daemon cannot decode.



## bookclerkd::cluster_link::LEAVE_NOTICE_TIMEOUT

*Constant*: `std::time::Duration`

How long Leave library waits for the owner to answer `node.leaving`.



## bookclerkd::cluster_link::TransportRunner

*Struct*

Runs jobs an owner cluster handed this host through a [`JobTransport`](job_handler.md#bookclerkdjob_handlerjobtransport),
the same path the daemon's own workers take.

**Generic Parameters:**
- T

**Fields:**
- `transport: T` - The job transport.
- `catalog: std::sync::Arc<self::remote_bindings::RemoteCatalog>` - The link's catalog of consented, fetched instances.

**Methods:**

- `fn new(transport: T, catalog: Arc<RemoteCatalog>) -> Self` - A runner over `transport` for the instances in `catalog`.

**Trait Implementations:**

- **HostJobRunner**
  - `fn run(self: &Self, ctx: HostJobCtx) -> Result<String, HostJobError>`



## Module: api

Operator API for host consent (H-4 design, step 8):

- `GET /api/config/cluster-consents`: the owner clusters this install
  is enrolled in (no key material: links carry ids, origin and state
  only), what each asks to run here (version, grant, and the archive URL
  and digest this host would fetch), and how this host stands on it;
- `POST /api/config/cluster-consents/approve`: approve one request by
  the `request_digest` the listing shows (refused with `409` when the
  recorded request is no longer that one, `404` when none is recorded,
  `400` for a narrowing it does not ask for, `503` on a store fault),
  optionally narrowed (denied domains, bindings, or all network);
- `POST /api/config/cluster-consents/revoke`: withdraw an approval (a
  revoke holds: a platform plugin is not approved automatically again).

- `POST /api/config/clusters/{cluster_id}/scan`: ask the owner cluster
  to queue a scan for this node's owner (H-5);
- `POST /api/config/clusters/{cluster_id}/jobs`: the owner's state of
  jobs this node may see.

All of them are operator-only: owner plugins run under this install's
jail with this install's network, so only its operator consents. A
change wakes every link supervisor in this process
([`super::consent::notify_consent_changed`](cluster_link/consent.md#bookclerkdcluster_linkconsentnotify_consent_changed)).



## Module: catalog_sync

On a branch: sharing this library's books with the library it joined
(branches of branches).

One [`CatalogSync`](cluster_link/catalog_sync.md#bookclerkdcluster_linkcatalog_synccatalogsync) runs beside each link's client. On every welcomed
connection, every [`SYNC_INTERVAL`](cluster_link/catalog_sync.md#bookclerkdcluster_linkcatalog_syncsync_interval), and when Settings → Library changes
a setting ([`notify_sharing_changed`](cluster_link/catalog_sync.md#bookclerkdcluster_linkcatalog_syncnotify_sharing_changed)), it:

1. sends `catalog.begin` with the link's two settings ("Share this
   branch's books with *library*", "Include books from my own
   branches"); the library answers a fresh fence, where to resume, its
   ancestry (recorded on the link, so `info` and the join cycle check
   stay current), its destination ids, and titles whose files it still
   wants;
2. with sharing on, sends the titles after the cursor in fenced
   `catalog.sync` batches, in a stable order (last change, then id): the
   books acquired on this install itself and, with the second setting
   on, the titles this install's own branches shared with it and whose
   files arrived here. A title carries store, product id, metadata and
   its file manifest only ([`bookclerk_node_proto::CatalogTitle`](../bookclerk_node_proto/catalog.md#bookclerk_node_protocatalogcatalogtitle)): no
   store account, user, or downstream branch is named;
3. streams the files of every wanted title from this install's primary
   destination to the library's upload endpoint (`catalog.upload`, one
   `PUT` per file, `commit`), which writes them straight to the
   library's destinations under the same keys.

The library keeps the cursor and the undelivered titles, so a sync cut
by a disconnect resumes where it stopped. Nothing of the library's work
is sent onward: this module only reads this install's own library.



## Module: consent

Host consent for owner plugins, per cluster link (H-4 design, section 5;
step 8).

[`LinkSupervisor`](cluster_link/consent.md#bookclerkdcluster_linkconsentlinksupervisor) keeps one link's [`RemoteCatalog`](cluster_link/remote_bindings.md#bookclerkdcluster_linkremote_bindingsremotecatalog) current and tells
the host client which plugins it may claim jobs for:

- on every connection and on each `deployment.changed`,
  `install.changed`, `placement.changed` and `configuration.changed`
  wake it re-reads `deployments.list`, records what the owner asks to
  run (`remote_plugin_consents`, the owner's approved grant and the
  archive URL and digest for this host as the request), auto-approves a Bookclerk platform plugin only when this
  host's own provenance check says the owner's artifact is the bundled
  one, and evaluates consent;
- for each consented instance it materializes the version (an owner
  `blob:` archive comes over the signed artifact endpoint and is checked
  against its digest before it is stored; an `https:` archive is fetched
  by the materializer itself, only when its URL and digest are the ones
  this host's operator approved and its host (and every redirect
  hop's) is not local or private; `file:` URLs are refused) and reads the
  instance's `CONFIG` and `SECRETS` with `instance.config`;
- it reports each deployment as `materialized`, or `error` with the
  consent state (`awaiting_host_consent`, `awaiting_host_reapproval`,
  `invalid_grant`) or the failure as the detail;
- every [`CONSENT_POLL`](cluster_link/consent.md#bookclerkdcluster_linkconsentconsent_poll) and whenever [`notify_consent_changed`](cluster_link/consent.md#bookclerkdcluster_linkconsentnotify_consent_changed) is
  called (the API's approve and revoke) it re-reads the consent rows, so
  an approval or a revoke from the CLI takes effect without a wake.
  Losing consent cancels the plugin's held jobs and gives them back. A
  change notified while a refresh runs is not lost (it is a generation
  counter, not a one-shot wake-up), and a refresh re-reads the consent
  rows after its slow steps (fetch, `instance.config`) and before it
  publishes, so a revoke that lands mid-refresh is never overwritten by
  the state the refresh read when it started;
- a refresh that could not finish (the owner unreachable, or a consented
  instance whose archive or `instance.config` failed) is retried with
  backoff ([`RETRY_MIN`](cluster_link/consent.md#bookclerkdcluster_linkconsentretry_min) doubling to [`RETRY_MAX`](cluster_link/consent.md#bookclerkdcluster_linkconsentretry_max)) until it does, so a
  transient failure never leaves a consented plugin unclaimable;
- an operator's revoke is recorded and holds: a platform plugin is not
  approved automatically again until the operator approves it.



## Module: host_jobs

How a host runs a job an owner cluster handed it (H-4 step 9).

[`HostJobTransport`](cluster_link/host_jobs.md#bookclerkdcluster_linkhost_jobshostjobtransport) is the [`JobTransport`](job_handler.md#bookclerkdjob_handlerjobtransport) behind every host client.
It never runs a job in process against this install's library: the
job's guest is the owner's plugin version materialized here, spawned
jailed under the grant this host's operator consented to (through the
mandatory `bookclerk-workerd` front door), opened with the bindings
[`super::remote_bindings::host_job_bindings`](cluster_link/remote_bindings.md#bookclerkdcluster_linkremote_bindingshost_job_bindings) built, and given only the
store credentials the owner handed with the job. Any failure to open it
fails the job; there is no fallback.

- **Scan.** The guest's rows come back unwritten
  (`bookclerk_plugin_host::ExternalSource::scan_handed`) and go to the
  owner in fenced `scan.results` batches of at most
  [`MAX_SCAN_BATCH_ROWS`](../bookclerk_node_proto/frames.md#bookclerk_node_protoframesmax_scan_batch_rows) rows and [`MAX_BATCH_BYTES`](cluster_link/host_jobs.md#bookclerkdcluster_linkhost_jobsmax_batch_bytes) (a row larger
  than [`MAX_SCAN_ROW_BYTES`](../bookclerk_node_proto/frames.md#bookclerk_node_protoframesmax_scan_row_bytes) is skipped). The owner validates and
  writes them (decision 6); the scan completes with `complete`.
- **Acquire.** The host asks the owner for the job's titles
  (`acquire.plan`, again until no new title comes back), acquires each
  into a per-title outbox under this host's files dir with one outbox
  destination per owner destination, each with that destination's own
  naming options, so every destination's keys come out as a local
  acquire on the owner would name them (the acquire pipeline runs
  against a scratch library
  opened through this host's database guest, seeded with the owner's
  row; media work stays in `bookclerk-media-worker`), copies the files
  to this host's own deployed destinations (H-2), and streams them to
  the owner (`upload.grant`, one `PUT` per distinct file content, each
  listing its key on every destination, then `commit`), which writes
  them straight to its destinations. A title is uploaded only once every
  one of this host's own destinations has it; a failed copy fails the
  title, and so does any failure of its upload after the copies were
  written (the grant, a file, the commit, or a cancel), which removes
  those copies again. The job completes with its closing upload's commit, never with
  `complete`, and the owner accepts that only once every title of the
  job is acquired. A host may stage; the owner never does.

**Cancellation.** The job's cancel flag (its fence lost, the operator's
cancel, or this host's consent revoked) stops every guest call within
[`CANCEL_POLL`]: the call is dropped, which shuts its guest down and
fences its host-side authority, so the guest stops using the handed
credentials. A title's upload is dropped the same way, and the owner
discards what that upload had written.

The guest work itself sits behind [`HostGuests`](cluster_link/host_jobs.md#bookclerkdcluster_linkhost_jobshostguests), so tests can stand in
a guest without spawning one; [`GuestHost`](cluster_link/host_jobs.md#bookclerkdcluster_linkhost_jobsguesthost) is the daemon's.



## Module: join

Operator API for joining a library from the web UI, and for leaving one
(the welcome screen, the Join wizard and its "I have an invite" tab, and
Settings → Library → Leave library).

- `GET /api/config/join`: whether this is a fresh install (no Owner,
  no Administrator, no cluster link), and the links it has.
- `POST /api/config/join/inspect` `{address}`: reads the library's
  `GET /api/node/v1/info` from this daemon (never from the browser)
  under the transport policy; a self-signed library's fingerprint comes
  back for the operator to confirm.
- `POST /api/config/join/start` `{address, fingerprint?, branch_name?,
  clerk?}`: reads the library again, checks the confirmed fingerprint,
  makes the PKCE verifier, `state` and node keys, keeps them sealed in a
  pending join (10 minutes), and answers the authorize URL to send the
  browser to. The redirect URI is this install's own `/join/callback`.
  With `clerk: true` the sign-in also offers to join as a clerk; only an
  empty install may ([`bookclerk_node_client::install_problem`](../bookclerk_node_client/clerk.md#bookclerk_node_clientclerkinstall_problem)).
- `POST /api/config/join/callback` `{code, state, connection?}`: the
  browser came back. The daemon matches `state` to its pending join and
  redeems the code with the verifier only it holds. As a branch it
  records the link and starts its client at once. As a clerk (only for
  a pending join that offered it, and only while the install is still
  empty) it opens the sealed bootstrap with the node key, checks the
  library's database guest is installed here, writes the bootstrap as
  this install's own (`[database]`, `master.key` wrapped with this
  install's password, `node-identity.json`), answers, and restarts on it
  ([`crate::restart`](restart.md)): this install is then an owner node of the
  library.
- `POST /api/config/join/invite` `{link}`: joins with a pasted branch
  invite; the link is used once and not kept.
- `POST /api/config/join/leave` `{cluster_id}`: Leave library.
- `GET /api/config/join/sharing`: the libraries this install is a
  branch of, with their two sharing settings.
- `POST /api/config/join/sharing` `{cluster_id, share_books,
  include_branches}`: Settings → Library's "Share this branch's books
  with *library*" and "Include books from my own branches"; the
  catalog sync reports them to the library at once.

Inspect and start refuse a library whose ancestry names this library's
own cluster id (409 `join_cycle`), so a tree of libraries never loops;
a branch's redeem checks it again before the code is spent.

The sharing routes are for the operator or an Owner (the stored role is
read on every call; anyone else gets 403); all the others are
operator-only. Answers carrying a URL with `state` are
`Cache-Control: no-store`; nothing secret is logged.



## bookclerkd::cluster_link::leave_link

*Function*

Leave library: tells the owner (`node.leaving`, through the running
client of the node the link names or, when none runs here as that node,
a short connection with the link's keys), then marks that node's link
`left`, deletes its node keys, and stops the client. A link a join
enrolled as another node meanwhile is not left (an error).

# Returns

The link as it now stands and whether the owner accepted the notice;
`None` when there is no such link.

# Errors

Returns the read or write error of the local leave.

```rust
fn leave_link(state: &std::sync::Arc<crate::api::AppState>, cluster_id: &str) -> bookclerk_library::Result<Option<(bookclerk_library::control_plane::ClusterLink, bool)>>
```



## Module: remote_bindings

What a host node knows of the instances an owner cluster deployed to
it, and the bindings a handed job runs with (H-4 design, step 8).

[`RemoteCatalog`](cluster_link/remote_bindings.md#bookclerkdcluster_linkremote_bindingsremotecatalog) is per link and in memory only: [`super::consent`](cluster_link/consent.md)
fills it from `deployments.list` and `instance.config` and re-evaluates
each entry's host consent. [`host_job_bindings`](cluster_link/remote_bindings.md#bookclerkdcluster_linkremote_bindingshost_job_bindings) turns one catalog
entry plus the secrets the owner handed with a job into what the guest
is opened with: `CONFIG`, `SECRETS` (only when the consented grant
carries the `secrets` binding) and the job's store credentials. Nothing
here is written to disk, and every type holding secret bytes redacts
its `Debug` and zeroizes on drop.



## bookclerkd::cluster_link::spawn_cluster_links

*Function*

Starts the cluster-link supervisor: one host client per active link of
the live library, restarted on the new store after every database swap
(`AppState::library_swaps`).

```rust
fn spawn_cluster_links(state: std::sync::Arc<crate::api::AppState>)
```



## bookclerkd::cluster_link::start_link_now

*Function*

Starts a client for `cluster_id` now (a join through this daemon), with
no restart: the supervisor started by [`spawn_cluster_links`](cluster_link.md#bookclerkdcluster_linkspawn_cluster_links) is woken
to start the link, and this waits up to `START_NOW_TIMEOUT` for a
client running as the link's current node. A client of an enrollment
the join replaced is stopped first (within [`consent::CONSENT_POLL`](cluster_link/consent.md#bookclerkdcluster_linkconsentconsent_poll)),
so a join over a still-active link can take that long.

# Returns

Whether a client for the link's current enrollment runs.

```rust
fn start_link_now(state: &std::sync::Arc<crate::api::AppState>, cluster_id: &str) -> bool
```



