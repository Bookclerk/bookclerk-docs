# bookclerk_node_client

The host side of the node API (H-4 design, sections 2 to 4; step 7).

A host node is a separate Bookclerk install that enrolled into an owner
cluster with a single-use link ([`join_cluster`](#join_cluster)) or a sign-in at the
owner ([`inspect_library`](#inspect_library), [`authorize_branch`](#authorize_branch),
[`redeem_branch_code`](#redeem_branch_code)); it tells the owner when it leaves
([`announce_leaving`](#announce_leaving)). An empty install may instead join as a clerk, an
owner node that shares the library's database and DEK
([`redeem_clerk_code`](#redeem_clerk_code), [`apply_clerk_bootstrap`](#apply_clerk_bootstrap)). Its node keys are
sealed in its own database
(`bookclerk_library::control_plane::record_cluster_link`). For every
active link, [`HostClient`](#hostclient) keeps one socket to the owner's
`/api/node/v1/connect`:

- TLS everywhere except loopback, verified against the link's SPKI pin
  or the webpki roots ([`OwnerEndpoint`](#ownerendpoint));
- the Ed25519 challenge signed with the enrolled key;
- heartbeats into the owner's `node_status`;
- claimed jobs run through a [`HostJobRunner`](#hostjobrunner), their secrets opened
  with the enrolled X25519 key, renewed and reported under the fence;
- refreshed store tokens written back sealed to the connection's owner
  box key ([`HostJobCtx::write_back`](#hostjobctxwrite_back));
- wakes acked after local subscribers have them;
- reconnect with backoff, resuming the fences still held.

- claims narrowed to the job kinds this host runs and the plugins its
  operator consented to ([`HostClientHandle::set_consented_plugins`](#hostclienthandleset_consented_plugins));
  a job whose consent is revoked is cancelled and given back unspent;
- after a wake, the host requests its deployment list and instance
  configuration from the owner and sends its observations back
  ([`HostClientHandle::request`](#hostclienthandlerequest) for `deployments.list`,
  `instance.config` and `observation`), and fetches a stored archive
  with [`HostClientHandle::fetch_artifact`](#hostclienthandlefetch_artifact), signed with the node key.

- a scan's rows returned in fenced batches
  ([`HostJobCtx::send_scan_results`](#hostjobctxsend_scan_results)); an acquire's titles asked for
  ([`HostJobCtx::acquire_plan`](#hostjobctxacquire_plan)) and its files streamed to the owner
  under a sealed upload token ([`HostJobCtx::grant_upload`](#hostjobctxgrant_upload),
  [`HostJobCtx::upload_file`](#hostjobctxupload_file), [`HostJobCtx::commit_upload`](#hostjobctxcommit_upload)); the job
  completes with its closing upload's commit.

The consent decisions themselves (which plugins, which grant) and how a
job runs its guest are the daemon's.

| Item | Kind | Summary |
| --- | --- | --- |
| [`HeartbeatSource`](#heartbeatsource) | trait | Measurements for `heartbeat` frames. |
| [`OwnerIo`](#ownerio) | trait | A byte stream to the owner, plain (loopback) or TLS. |
| [`HostJobRunner`](#hostjobrunner) | trait | Runs claimed jobs on this host. |
| [`BranchAuthorization`](#branchauthorization) | struct | One sign-in about to start. `Debug` prints no secret. |
| [`BranchRedeem`](#branchredeem) | struct | What `redeem_branch_code` needs besides the keys. |
| [`InspectedLibrary`](#inspectedlibrary) | struct | A library as `inspect_library` found it. |
| [`AppliedClerk`](#appliedclerk) | struct | What `apply_clerk_bootstrap` wrote. |
| [`ClerkJoined`](#clerkjoined) | struct | A redeemed clerk sign-in. `Debug` prints no secret. |
| [`PendingClerkJoin`](#pendingclerkjoin) | struct | A clerk join whose files are all in place but whose first start has not opened the library's database yet, or whose undoing was cut short. The daemon `confirm`s it once the database opens, or `revert`s it when it cannot, so a join to an unreachable database does not leave the server unable to start. |
| [`RevertFailed`](#revertfailed) | struct | Why `PendingClerkJoin::revert` did not finish. |
| [`CatalogBatchAccepted`](#catalogbatchaccepted) | struct | A recorded catalog batch (`HostClientHandle::catalog_sync`). |
| [`CatalogSession`](#catalogsession) | struct | An opened catalog session (`HostClientHandle::catalog_begin`). |
| [`ClientTiming`](#clienttiming) | struct | Timers of the client. The defaults follow the protocol constants. |
| [`HostClient`](#hostclient) | struct | The host side of the node API for one cluster link. |
| [`HostClientConfig`](#hostclientconfig) | struct | What one client connects with. |
| [`HostClientHandle`](#hostclienthandle) | struct | Watches a running client. |
| [`OwnerEndpoint`](#ownerendpoint) | struct | Where and how to reach one owner. |
| [`TlsProbe`](#tlsprobe) | struct | What a first TLS handshake showed of an owner's certificate, before any pin is known (`OwnerEndpoint::probe_json`). |
| [`HandedSecrets`](#handedsecrets) | struct | The job's secrets, opened with this node's box key. Values are zeroized on drop and `Debug` shows only which rows they are. |
| [`HostJob`](#hostjob) | struct | A job the owner handed this host, under its fence. |
| [`HostJobCtx`](#hostjobctx) | struct | Everything a runner gets for one job. |
| [`HostJobError`](#hostjoberror) | struct | How a job failed: a stable kind and an operator-facing message. Neither may contain a secret. |
| [`HostUpload`](#hostupload) | struct | An open upload of one title: its id and bearer token. The token is zeroized on drop and never shown by `Debug`. |
| [`ClientExit`](#clientexit) | enum | Why `HostClient::run` returned. |
| [`DialError`](#dialerror) | enum | Why the owner could not be reached. |
| [`JoinError`](#joinerror) | enum | Why joining failed. |
| [`OwnerCallError`](#ownercallerror) | enum | Why a call the job makes to the owner (results, plan, upload) failed. |
| [`WriteBackError`](#writebackerror) | enum | Why a token write-back was not stored. |
| [`announce_leaving`](#announce_leaving) | function | Tells the library of `link` that this install is leaving: connects with the link's keys (claiming nothing; `sampler` answers the connection's first heartbeat), sends `node.leaving`, and waits for the answer for up to `timeout`. Answers true when the library accepted it. Call it before the keys are deleted. |
| [`authorize_branch`](#authorize_branch) | function | Starts a sign-in at `library` for this install as `branch_name`, coming back to `redirect_uri` (a loopback or https URL). |
| [`authorize_join`](#authorize_join) | function | `authorize_branch`, optionally offering to join as a clerk (`offer_clerk`, `CLERK_OFFER_PARAM`): the consent page then lets an Owner of the library choose a clerk instead of a branch. Offer it only for an install that holds no library data of its own. |
| [`inspect_library`](#inspect_library) | function | Reads what the library at `address` says about itself, under the transport policy (plain HTTP only on loopback). |
| [`library_origin`](#library_origin) | function | Normalizes what an operator typed (`owner.example:8443`, `https://owner.example/`) to a canonical origin. A bare `host[:port]` means `https`. |
| [`redeem_branch_code`](#redeem_branch_code) | function | Redeems an approved sign-in: the library runs the same redeem as for a link (one node, a receipt, idempotent retries), then this install seals `keys` and records the link. The code and verifier go once in the request body and are never logged. |
| [`apply_clerk_bootstrap`](#apply_clerk_bootstrap) | function | Writes `bootstrap` as this install's own bootstrap under `files_dir`: |
| [`bootstrap_database_kind`](#bootstrap_database_kind) | function | The `[database].plugin` a clerk's bootstrap configures. |
| [`checked_d1_api_base`](#checked_d1_api_base) | function | Checks the D1 API origin a library's bootstrap names, and returns it normalized. Every D1 query carries the joining server's own API token to that origin, so a clerk may only be pointed at Cloudflare's API over HTTPS. The library checks its own origin the same way before it offers a clerk join, so a library on another D1 endpoint cannot take clerks. |
| [`database_guest_installed`](#database_guest_installed) | function | True when the database guest for `kind` is installed for `config`, so the daemon can open the library's database once the bootstrap is applied. A plugin directory that cannot be read counts as not installed. |
| [`install_problem`](#install_problem) | function | Why this install may not join as a clerk now, or `None`. |
| [`interrupted_clerk_join`](#interrupted_clerk_join) | function | Names the copies an interrupted clerk join left beside this install's bootstrap files, or `None` when there are none. The daemon refuses to start while any is there: a process that died while replacing the files leaves some new and some old, and starting on that mix would pair a database with the wrong node identity or key. The message says how to finish or undo the join by hand (see `apply_clerk_bootstrap`). |
| [`pending_clerk_join`](#pending_clerk_join) | function | The clerk join this install applied but has not yet started on (or began undoing), if any. A record that cannot be read, or that names any file other than this install's own bootstrap files and their copies, is not used: copies left beside those files then stop the start (see `interrupted_clerk_join`). |
| [`redeem_clerk_code`](#redeem_clerk_code) | function | Redeems an approved clerk sign-in and opens the bootstrap with the node's box secret. The code and verifier go once in the request body, and are registered for redaction (the caller unregisters them). |
| [`check_join_ancestry`](#check_join_ancestry) | function | Reads the library's `info` over `endpoint` and refuses a library whose ancestry names this install's own cluster id (A ← B ← A). Answers its ancestry, to record on the link. |
| [`join_cluster`](#join_cluster) | function | Enrolls this install into the owner cluster named by `link`. |
| [`refuse_cycle`](#refuse_cycle) | function | Refuses `ancestry` when it names this install's own cluster id, is malformed, or would take this library's ancestry past what one can name (`join_overflows_ancestry`). |
| [`OwnerSocket`](#ownersocket) | type alias | The owner socket. |
| [`CLERK_POSTGRES_URL_FILE`](#clerk_postgres_url_file) | constant | File under the files directory that holds a clerk's Postgres URL, mode `0600`. `[database.postgres].url_file` names it by its absolute path (a relative `url_file` would be read from the daemon's working directory). |
| [`DATABASE_ENV_OVERRIDES`](#database_env_overrides) | constant | Environment variables that would outrank a clerk's `[database]` bootstrap; a clerk join is refused while one is set. |
| [`AWAITING_HOST_CONSENT`](#awaiting_host_consent) | constant | Release reason for a job whose plugins this host's operator has not (or no longer) approved. |
| [`KIND_NOT_RUN_HERE`](#kind_not_run_here) | constant | Release reason for a job of a kind this host does not run. |
| [`MAX_ARTIFACT_BYTES`](#max_artifact_bytes) | constant | Largest artifact fetched from the owner (the library blob cap). |
| [`MAX_LEASE`](#max_lease) | constant | Longest lease this host counts on from one `work` or `renewed`; a longer grant is renewed as if it were this long. |
| [`SECRETS_UNOPENABLE`](#secrets_unopenable) | constant | Error kind reported when a job's secrets do not open with this node's box key. |
| [`CONNECT_PATH`](#connect_path) | constant | Path of the node socket. |
| [`ENROLL_PATH`](#enroll_path) | constant | Path of the enroll endpoint. |

## Re-exports

- [`endpoint::ENROLL_OAUTH_PATH`](../bookclerk_node_proto/index.md#enroll_oauth_path)
- [`endpoint::INFO_PATH`](../bookclerk_node_proto/index.md#info_path)

## Traits

### HeartbeatSource

```rust
pub trait HeartbeatSource: Send + 'static {
    fn sample(&mut self) -> NodeHeartbeat;
}
```

Measurements for `heartbeat` frames.

#### HeartbeatSource::sample

```rust
fn sample(&mut self) -> NodeHeartbeat
```

Samples this host now. May block briefly (it runs on a blocking
thread).

Implemented by: `NodeSampler`

### OwnerIo

```rust
pub trait OwnerIo: AsyncRead + AsyncWrite + Unpin + Send {}
```

A byte stream to the owner, plain (loopback) or TLS.

Implemented by: `T`

### HostJobRunner

```rust
pub trait HostJobRunner: Send + Sync + 'static {
    fn run(
        &self,
        ctx: HostJobCtx,
    ) -> impl Future<Output = Result<String, HostJobError>> + Send;
}
```

Runs claimed jobs on this host.

The runner must stop soon after [`HostJobCtx::is_cancelled`](#hostjobctxis_cancelled) turns true;
its result is then discarded. Error kinds and messages are recorded on
the owner and must never contain secret material.

#### HostJobRunner::run

```rust
fn run(
    &self,
    ctx: HostJobCtx,
) -> impl Future<Output = Result<String, HostJobError>> + Send
```

Runs one job; `Ok` carries the outcome text recorded as progress.

## Structs

### BranchAuthorization

```rust
pub struct BranchAuthorization {
    pub authorize_url: String,
    pub state: Zeroizing<String>,
    pub verifier: Zeroizing<String>,
    pub keys: ClusterLinkKeys,
    pub redirect_uri: String,
}
```

One sign-in about to start. `Debug` prints no secret.

**Fields**

- `authorize_url`: Where to send the browser.
- `state`: The OAuth `state` (also in the URL).
- `verifier`: The PKCE verifier; never leaves this install.
- `keys`: The node keys the branch will enroll.
- `redirect_uri`: The redirect URI in the URL.

Implements: `Debug`

### BranchRedeem

```rust
pub struct BranchRedeem<'a> {
    pub origin: &'a str,
    pub spki_pin: Option<&'a str>,
    pub cluster_id: &'a str,
    pub code: &'a str,
    pub verifier: &'a str,
    pub redirect_uri: &'a str,
    pub branch_name: &'a str,
    pub software_version: &'a str,
    pub enrolled_node_id: Option<&'a str>,
}
```

What [`redeem_branch_code`](#redeem_branch_code) needs besides the keys.

**Fields**

- `origin`: The library's canonical origin.
- `spki_pin`: The pin confirmed for a self-signed library.
- `cluster_id`: The cluster id the library reported before sign-in.
- `code`: The code from the callback.
- `verifier`: The PKCE verifier.
- `redirect_uri`: The redirect URI of the authorization.
- `branch_name`: The branch name.
- `software_version`: This install's version.
- `enrolled_node_id`: The node the library enrolled on an earlier attempt of this same
  exchange that could not be saved here ([`JoinError::SaveFailed`](#joinerror)),
  when this is a retry of it. The code is already spent then, so an
  ancestry check that now fails for good answers
  [`JoinError::Unsaved`](#joinerror) naming that node, for its owner to revoke.

### InspectedLibrary

```rust
pub struct InspectedLibrary {
    pub info: OwnerInfo,
    pub origin: String,
    pub spki_pin: Option<String>,
}
```

A library as [`inspect_library`](#inspect_library) found it.

**Fields**

- `info`: What the library said about itself.
- `origin`: Its canonical origin.
- `spki_pin`: The SPKI pin to trust from now on: set for a self-signed library
  (the operator must confirm it), `None` when the certificate chains
  to the public roots or the library is on loopback.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AppliedClerk

```rust
pub struct AppliedClerk {
    pub database: DatabasePluginKind,
    pub wrapped: bool,
    pub config_file: PathBuf,
}
```

What [`apply_clerk_bootstrap`](#apply_clerk_bootstrap) wrote.

**Fields**

- `database`: The `[database].plugin` now configured.
- `wrapped`: True when `master.key` is wrapped with this install's password.
- `config_file`: The config file written.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ClerkJoined

```rust
pub struct ClerkJoined {
    pub cluster_id: String,
    pub node_id: String,
    pub bootstrap: ClerkBootstrap,
}
```

A redeemed clerk sign-in. `Debug` prints no secret.

**Fields**

- `cluster_id`: The library's cluster id.
- `node_id`: The node id the library recorded (`kind = owner`).
- `bootstrap`: The opened bootstrap.

Implements: `Debug`

### PendingClerkJoin

```rust
pub struct PendingClerkJoin {
    /* private fields */
}
```

A clerk join whose files are all in place but whose first start has not
opened the library's database yet, or whose undoing was cut short. The
daemon [`confirm`](#pendingclerkjoinconfirm)s it once the database opens, or
[`revert`](#pendingclerkjoinrevert)s it when it cannot, so a join to an unreachable
database does not leave the server unable to start.

#### PendingClerkJoin::reverting

```rust
pub fn reverting(&self) -> bool
```

Whether an earlier start began putting the originals back and did not
finish: the daemon then finishes the [`revert`](#pendingclerkjoinrevert) before
anything else, and does not try the library's database again.

#### PendingClerkJoin::confirm

```rust
pub fn confirm(self)
```

The library's database opened: drops the record, then the copies of
the originals. The record goes first, so a crash part way leaves
copies with no record, which the next start refuses on with
[`interrupted_clerk_join`](#interrupted_clerk_join)'s instructions (its `config.toml` is the
library's, so the copies are to be deleted). A copy that cannot be
removed is logged, since it stops the next start.

#### PendingClerkJoin::revert

```rust
pub fn revert(self) -> Result<(), RevertFailed>
```

The library's database did not open: puts every original back,
removes the files the join created, and drops the record, so the
next start runs on this install's own bootstrap again. The record is
first marked [`reverting`](#pendingclerkjoinreverting), and every step can run
again (a copy already moved back is gone, a created file already
removed is absent), so a revert cut short by a crash or a failed
rename is finished by the next start instead of leaving a mix.

Before the revert starts, every recorded copy must still exist: a
copy missing at that point was removed by hand, not moved back, so
undoing the rest would leave a mix of this install's files and the
library's.

##### Errors

Returns [`RevertFailed`](#revertfailed) with [`started`](#revertfailed)
false when nothing was changed (a copy is missing, or the record could
not be marked), so the record still says the join is applied. With
`started` true it names the files that could not be put back; the
record stays marked reverting, so the next start finishes the undo
before anything else.

Implements: `Debug`

### RevertFailed

```rust
pub struct RevertFailed {
    pub started: bool,
    pub detail: String,
}
```

Why [`PendingClerkJoin::revert`](#pendingclerkjoinrevert) did not finish.

**Fields**

- `started`: Whether the undo had begun: when true the record is marked
  reverting and the next start finishes it; when false nothing was
  changed and the record still says the join is applied.
- `detail`: What went wrong, naming the files involved.

Implements: `Debug`, `Display`

### CatalogBatchAccepted

```rust
pub struct CatalogBatchAccepted {
    pub upserted: u64,
    pub skipped: u64,
    pub wanted: Vec<CatalogRef>,
}
```

A recorded catalog batch ([`HostClientHandle::catalog_sync`](#hostclienthandlecatalog_sync)).

**Fields**

- `upserted`: Titles recorded.
- `skipped`: Titles the library already holds (not echoed back).
- `wanted`: Titles whose files the library wants.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CatalogSession

```rust
pub struct CatalogSession {
    pub fence: i64,
    pub cursor: String,
    pub ancestry: Vec<String>,
    pub destinations: Vec<String>,
    pub wanted: Vec<CatalogRef>,
}
```

An opened catalog session ([`HostClientHandle::catalog_begin`](#hostclienthandlecatalog_begin)).

**Fields**

- `fence`: The catalog fence every later call names.
- `cursor`: Resume after this cursor (empty: from the start).
- `ancestry`: The library's ancestry, its own cluster id first.
- `destinations`: The library node's destination ids, the primary first.
- `wanted`: Titles whose files the library still wants.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ClientTiming

```rust
pub struct ClientTiming {
    pub heartbeat_interval: Duration,
    pub ping_interval: Duration,
    pub idle_timeout: Duration,
    pub backoff_min: Duration,
    pub backoff_max: Duration,
    pub request_timeout: Duration,
}
```

Timers of the client. The defaults follow the protocol constants.

**Fields**

- `heartbeat_interval`: Between heartbeats.
- `ping_interval`: Between pings.
- `idle_timeout`: Without any inbound frame before the client closes.
- `backoff_min`: First reconnect delay.
- `backoff_max`: Largest reconnect delay.
- `request_timeout`: How long a request waits for room in the send queue and then for
  its answer.

Implements: `Clone`, `Copy`, `Debug`, `Default`

### HostClient

```rust
pub struct HostClient<R: HostJobRunner> {
    /* private fields */
}
```

The host side of the node API for one cluster link.

#### HostClient::new

```rust
pub fn new(
    config: HostClientConfig,
    runner: R,
    sampler: Box<dyn HeartbeatSource>,
) -> Result<(Self, HostClientHandle), DialError>
```

A client for `config`, running jobs through `runner`.

##### Errors

Returns [`DialError`](#dialerror) when the link's origin or pin fails the
transport policy.

#### HostClient::run

```rust
pub async fn run(self, shutdown: Receiver<bool>) -> ClientExit
```

Connects, serves, and reconnects until `shutdown` turns true (or its
sender is dropped) or the owner revokes this node.

Every held job is cancelled as lost when this returns, whatever the
reason, and also when the future is dropped or its task aborted
first, so no job task keeps running for a client that is gone.

### HostClientConfig

```rust
pub struct HostClientConfig {
    pub link: ClusterLink,
    pub keys: ClusterLinkKeys,
    pub software_version: String,
    pub incarnation: String,
    pub max_jobs: usize,
    pub claim_kinds: Vec<String>,
    pub claims_after_refresh: bool,
    pub timing: ClientTiming,
}
```

What one client connects with.

**Fields**

- `link`: The active cluster link.
- `keys`: Its node keys.
- `software_version`: This host's Bookclerk version.
- `incarnation`: This process's incarnation (a plain id).
- `max_jobs`: Jobs run at once. Zero claims nothing.
- `claim_kinds`: Job kinds this host runs (`scan`, ...). Empty claims nothing.
- `claims_after_refresh`: When set, a connection claims nothing until
  [`HostClientHandle::deployments_refreshed`](#hostclienthandledeployments_refreshed) names it, so the caller
  can re-read what the owner deployed (which may have changed while
  this host was away) before work runs against it.
- `timing`: Timers.

### HostClientHandle

```rust
pub struct HostClientHandle {
    /* private fields */
}
```

Watches a running client.

#### HostClientHandle::subscribe_wakes

```rust
pub fn subscribe_wakes(&self) -> Receiver<Value>
```

Wakes the owner forwarded (`{"type": "deployment.changed", ...}`).

A wake is acked once it is handed to at least one subscriber; while
there is none it stays unacked and the owner redelivers it later.
A subscriber that falls more than 64 wakes behind sees
`broadcast::error::RecvError::Lagged` for wakes already acked, and
must then resync the state they announce from the owner.

#### HostClientHandle::connections

```rust
pub fn connections(&self) -> Receiver<u64>
```

Count of welcomed connections so far.

#### HostClientHandle::deployments_refreshed

```rust
pub fn deployments_refreshed(&self, connection: u64)
```

Records that the deployments were re-read on the connection counted
`connection` (a [`Self::connections`](#hostclienthandleconnections) value read before the
request), which opens its claims under
[`HostClientConfig::claims_after_refresh`](#hostclientconfig). A read made on an
earlier connection opens nothing on a later one.

#### HostClientHandle::is_connected

```rust
pub fn is_connected(&self) -> bool
```

True while a welcomed connection is live.

#### HostClientHandle::held_fences

```rust
pub fn held_fences(&self) -> Vec<HeldFence>
```

Fences this host holds and has not lost.

#### HostClientHandle::node_id

```rust
pub fn node_id(&self) -> &str
```

The node id this host has in the owner cluster.

#### HostClientHandle::cluster_id

```rust
pub fn cluster_id(&self) -> &str
```

The owner cluster id.

#### HostClientHandle::set_consented_plugins

```rust
pub fn set_consented_plugins(&self, plugins: BTreeSet<String>)
```

Sets the plugin keys this host's operator consents to. Claims carry
them as their filter from the next claim on; a held job that runs a
plugin outside them is cancelled and given back unrun.

The new set is published before the held jobs are scanned: a job
that is being started concurrently is either already held (and
caught by the scan) or checks consent after it is held and so reads
the new set. Scanning first would leave a gap where a job inserted
after the scan still read the old set and ran.

#### HostClientHandle::consented_plugins

```rust
pub fn consented_plugins(&self) -> BTreeSet<String>
```

The plugin keys consented to now.

#### HostClientHandle::request

```rust
pub async fn request(&self, body: Body) -> Option<Frame>
```

Sends `body` on the live connection (waiting up to the request
timeout for one) and answers the owner's frame, or `None` when no
connection answered.

#### HostClientHandle::open_instance_secrets

```rust
pub fn open_instance_secrets(
    &self,
    instance_id: &str,
    revision: i64,
    sealed: &str,
) -> Result<SecretValue, SealError>
```

Opens an instance's `SECRETS` sealed to this node's box key
(`instance_config.sealed_secrets`), registering the values with the
log and diagnostics scrubber.

##### Errors

Returns [`SealError`](../bookclerk_node_proto/index.md#sealerror) when it was not sealed to this node for this
instance and revision.

#### HostClientHandle::fetch_artifact

```rust
pub async fn fetch_artifact(&self, archive_sha256: &str) -> Result<Vec<u8>, DialError>
```

Downloads one archive the owner stores (`blob:sha256:<hex>`) from its
artifact endpoint, signing the request with this node's key. The
caller checks the digest.

##### Errors

Returns [`DialError`](#dialerror) when the request fails or the owner does not
answer 200 with at most [`MAX_ARTIFACT_BYTES`](#max_artifact_bytes).

#### HostClientHandle::catalog_begin

```rust
pub async fn catalog_begin(
    &self,
    share_books: bool,
    include_branches: bool,
) -> Result<CatalogSession, OwnerCallError>
```

`catalog.begin`: opens this branch's catalog sync with the link's
two settings and answers the session (a new fence each time).

##### Errors

Returns [`OwnerCallError`](#ownercallerror).

#### HostClientHandle::catalog_sync

```rust
pub async fn catalog_sync(
    &self,
    fence: i64,
    batch: u64,
    cursor: &str,
    done: bool,
    titles: Vec<CatalogTitle>,
) -> Result<CatalogBatchAccepted, OwnerCallError>
```

`catalog.sync`: one batch of shared titles under `fence`, naming
the cursor after it.

##### Errors

Returns [`OwnerCallError`](#ownercallerror); `Lost` when the fence moved on.

#### HostClientHandle::catalog_upload

```rust
pub async fn catalog_upload(
    &self,
    fence: i64,
    title: &CatalogRef,
    files: Vec<UploadFile>,
) -> Result<HostUpload, OwnerCallError>
```

`catalog.upload`: opens an upload of one shared title's `files` and
opens its sealed token (bound to this node, [`CATALOG_ITEM`](../bookclerk_node_proto/index.md#catalog_item), the
fence and the upload).

##### Errors

Returns [`OwnerCallError`](#ownercallerror).

#### HostClientHandle::upload_catalog_file

```rust
pub async fn upload_catalog_file<R: AsyncRead + Unpin + Send>(
    &self,
    upload: &HostUpload,
    index: usize,
    length: u64,
    body: R,
) -> Result<(), OwnerCallError>
```

Streams declared file `index` of a catalog `upload` (`length` bytes
of `body`) to the library's upload endpoint.

##### Errors

Returns [`OwnerCallError`](#ownercallerror).

#### HostClientHandle::commit_catalog_upload

```rust
pub async fn commit_catalog_upload(
    &self,
    upload: &HostUpload,
) -> Result<(), OwnerCallError>
```

Commits a catalog `upload`.

##### Errors

Returns [`OwnerCallError`](#ownercallerror).

Implements: `Clone`

### OwnerEndpoint

```rust
pub struct OwnerEndpoint {
    /* private fields */
}
```

Where and how to reach one owner.

#### OwnerEndpoint::new

```rust
pub fn new(origin: &str, spki_pin: Option<&str>) -> Result<Self, DialError>
```

Checks `origin` (and the link's `spki_pin`) against the transport
policy.

##### Errors

Returns [`DialError::Invalid`](#dialerror) for a malformed origin or pin and
[`DialError::Insecure`](#dialerror) for plain HTTP to a non-loopback host or a
pinned plain-HTTP origin.

#### OwnerEndpoint::origin

```rust
pub fn origin(&self) -> &str
```

The canonical origin the handshake signs.

#### OwnerEndpoint::is_pinned

```rust
pub fn is_pinned(&self) -> bool
```

True when the link pins the owner's certificate.

#### OwnerEndpoint::connect

```rust
pub async fn connect(&self) -> Result<Box<dyn OwnerIo>, DialError>
```

Opens a byte stream to the owner under the policy.

##### Errors

Returns [`DialError`](#dialerror) when the owner is unreachable or TLS fails.

#### OwnerEndpoint::connect_socket

```rust
pub async fn connect_socket(&self) -> Result<OwnerSocket, DialError>
```

Dials the node socket, offering [`SUBPROTOCOL`](../bookclerk_node_proto/index.md#subprotocol) and requiring the
owner to select it.

##### Errors

Returns [`DialError`](#dialerror) when the connection or the upgrade fails.

#### OwnerEndpoint::post_json

```rust
pub async fn post_json(
    &self,
    path: &str,
    body: &[u8],
) -> Result<(u16, Vec<u8>), DialError>
```

`POST path` with a JSON body; answers the status and body. The body
is written straight to the stream and never logged.

##### Errors

Returns [`DialError`](#dialerror) when the request cannot be made or the answer
is malformed.

#### OwnerEndpoint::get_json

```rust
pub async fn get_json(
    &self,
    path: &str,
    max_body: usize,
) -> Result<(u16, Vec<u8>), DialError>
```

`GET path` of a small JSON answer (at most `max_body` bytes), over the
verified transport (the link's pin or the webpki roots). Nothing
secret is sent.

##### Errors

Returns [`DialError`](#dialerror) when the request cannot be made, the answer is
malformed, or it is larger than `max_body`.

#### OwnerEndpoint::get_authorized

```rust
pub async fn get_authorized(
    &self,
    path: &str,
    authorization: &str,
    max_body: usize,
) -> Result<(u16, Vec<u8>), DialError>
```

`GET path` with an `Authorization` header; answers the status and a
body of at most `max_body` bytes. The header is written straight to
the stream and never logged.

##### Errors

Returns [`DialError`](#dialerror) when the request cannot be made, the answer is
malformed, or it is larger than `max_body`.

#### OwnerEndpoint::put_stream

```rust
pub async fn put_stream<R: AsyncRead + Unpin + Send>(
    &self,
    path: &str,
    authorization: &str,
    length: u64,
    body: R,
) -> Result<(u16, Vec<u8>), DialError>
```

`PUT path` streaming exactly `length` bytes from `body`, with an
`Authorization` header; answers the status and the (small) body.
The header and the bytes are written straight to the stream and
never logged. Nothing is buffered beyond one copy chunk.

An owner may answer before it has read the whole body (a lost
fence); the answer is still read after a failed write, for at most
the idle timeout. Every read of `body` and every write to the owner
must make progress within `UPLOAD_IDLE_TIMEOUT_SECS`.

##### Errors

Returns [`DialError`](#dialerror) when the request cannot be made, `body` ends
early, a read or write stalls ([`DialError::Timeout`](#dialerror)), or the
answer is malformed or does not come within
`UPLOAD_ANSWER_TIMEOUT_SECS` of the last byte.

#### OwnerEndpoint::post_json_authorized

```rust
pub async fn post_json_authorized(
    &self,
    path: &str,
    authorization: &str,
    body: &[u8],
) -> Result<(u16, Vec<u8>), DialError>
```

`POST path` with a JSON body and an `Authorization` header; answers
the status and body. Neither is logged.

##### Errors

Returns [`DialError`](#dialerror) when the request cannot be made or the answer
is malformed.

#### OwnerEndpoint::probe_json

```rust
pub async fn probe_json(
    &self,
    path: &str,
) -> Result<(u16, Vec<u8>, Option<TlsProbe>), DialError>
```

`GET path` on an owner whose certificate is not known yet: the TLS
handshake accepts any certificate whose key signs the handshake,
records its SPKI pin and whether it would pass the webpki roots, and
answers both with the status and body. Plain-HTTP (loopback) origins
answer no probe.

Only for reading what a library says about itself before its
operator confirms the pin (trust on first use); nothing secret is
sent. A pinned endpoint is refused.

##### Errors

Returns [`DialError`](#dialerror) when the request cannot be made or the answer
is malformed.

Implements: `Clone`, `Debug`

### TlsProbe

```rust
pub struct TlsProbe {
    pub spki_pin: String,
    pub webpki_trusted: bool,
}
```

What a first TLS handshake showed of an owner's certificate, before any
pin is known ([`OwnerEndpoint::probe_json`](#ownerendpointprobe_json)).

**Fields**

- `spki_pin`: base64url SHA-256 of the presented certificate's SubjectPublicKeyInfo.
- `webpki_trusted`: True when the certificate chains to the webpki roots and names the
  host (what an unpinned connection requires).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### HandedSecrets

```rust
pub struct HandedSecrets(/* private */);
```

The job's secrets, opened with this node's box key. Values are
zeroized on drop and `Debug` shows only which rows they are.

#### HandedSecrets::refs

```rust
pub fn refs(&self) -> impl Iterator<Item = &SecretRef>
```

The rows handed with the job.

#### HandedSecrets::get

```rust
pub fn get(&self, secret_ref: &SecretRef) -> Option<&SecretValue>
```

The value of one handed row.

#### HandedSecrets::len

```rust
pub fn len(&self) -> usize
```

Number of handed rows.

#### HandedSecrets::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when the job needs no secrets.

Implements: `Debug`

### HostJob

```rust
pub struct HostJob {
    pub node_id: String,
    pub job: WorkJob,
    pub fence: i64,
    pub lease_secs: u64,
    pub plugin_keys: Vec<String>,
    pub source_plugin_key: Option<String>,
    pub lease_deadline: SystemTime,
    pub resumed: bool,
}
```

A job the owner handed this host, under its fence.

**Fields**

- `node_id`: This host's node id in the owner cluster (the lease owner is
  `host:<node_id>`).
- `job`: The job as the owner stores it.
- `fence`: Lease generation; every frame for the job names it.
- `lease_secs`: Lease length the owner granted.
- `plugin_keys`: Plugin keys the job runs (each consented when it was claimed).
- `source_plugin_key`: The one plugin of `plugin_keys` whose source guest runs the job,
  when the owner named it.
- `lease_deadline`: When the granted lease runs out, counted as the client counts it:
  from before the claim was sent, capped at [`crate::MAX_LEASE`](#max_lease). Work
  handed a deadline should stop by then, since the fence may be lost
  once it passes without a renewal.
- `resumed`: The claim resumed a suspended job: no attempt was spent, and the
  job's checkpoint applies.

Implements: `Clone`, `Debug`

### HostJobCtx

```rust
pub struct HostJobCtx {
    pub job: HostJob,
    pub secrets: Arc<HandedSecrets>,
    /* private fields */
}
```

Everything a runner gets for one job.

**Fields**

- `job`: The job and its fence.
- `secrets`: Its opened secrets.

#### HostJobCtx::is_cancelled

```rust
pub fn is_cancelled(&self) -> bool
```

True when the runner should stop: the fence was lost or the
operator cancelled the job.

#### HostJobCtx::is_lost

```rust
pub fn is_lost(&self) -> bool
```

True once the job's fence is lost: the owner reported it, or the
lease ran out without a confirmed renewal. The owner may already
have handed the job to another run, which then owns the job's
titles, so this run must not undo anything it wrote under them.

#### HostJobCtx::cancel_flag

```rust
pub fn cancel_flag(&self) -> Arc<AtomicBool>
```

The cancel flag, for runners that poll it elsewhere.

#### HostJobCtx::write_back

```rust
pub async fn write_back(
    &self,
    secret_ref: &SecretRef,
    value: &[u8],
) -> Result<(), WriteBackError>
```

Writes a refreshed store token back to the owner (decision 3).

Only a row handed with this job may be written, and only under its
live fence. The value is sealed to the current connection's owner
box key, bound to this node, job, fence and ref; it never travels or
is logged as text.

##### Errors

Returns [`WriteBackError`](#writebackerror) when the ref was not handed, the owner
refuses, the fence is lost, or no owner connection answers.

#### HostJobCtx::send_scan_results

```rust
pub async fn send_scan_results(
    &self,
    plugin_key: &str,
    books: Vec<Value>,
) -> Result<u64, OwnerCallError>
```

Sends one batch of a scan's rows (the guest's `ScanBook` objects) to
the owner, which validates and writes them (decision 6); answers the
rows written. At most
[`bookclerk_node_proto::MAX_SCAN_BATCH_ROWS`](../bookclerk_node_proto/index.md#max_scan_batch_rows) rows, each at most
[`bookclerk_node_proto::MAX_SCAN_ROW_BYTES`](../bookclerk_node_proto/index.md#max_scan_row_bytes) serialized.

##### Errors

Returns [`OwnerCallError`](#ownercallerror); `Lost` also cancels the job.

#### HostJobCtx::acquire_plan

```rust
pub async fn acquire_plan(
    &self,
) -> Result<(Vec<AcquireTarget>, Vec<PlanDestination>), OwnerCallError>
```

Asks the owner which titles this acquire should fetch, and with
which destinations (the owner node's, the primary first, each with
its own `DownloadOptions`).

##### Errors

Returns [`OwnerCallError`](#ownercallerror); `Lost` also cancels the job.

#### HostJobCtx::grant_upload

```rust
pub async fn grant_upload(
    &self,
    book_uuid: &str,
    primary_key: &str,
    pdf_key: Option<&str>,
    files: Vec<UploadFile>,
) -> Result<HostUpload, OwnerCallError>
```

Opens an upload of one title's `files` (an empty `book_uuid` and no
files for the job's closing upload).

##### Errors

Returns [`OwnerCallError`](#ownercallerror); `Lost` also cancels the job.

#### HostJobCtx::upload_file

```rust
pub async fn upload_file<R: AsyncRead + Unpin + Send>(
    &self,
    upload: &HostUpload,
    index: usize,
    length: u64,
    body: R,
) -> Result<(), OwnerCallError>
```

Streams declared file `index` of `upload` (`length` bytes of
`body`) to the owner, which writes it to its destinations as it
arrives.

##### Errors

Returns [`OwnerCallError`](#ownercallerror); `Lost` (the owner aborted the upload)
also cancels the job.

#### HostJobCtx::commit_upload

```rust
pub async fn commit_upload(
    &self,
    upload: &HostUpload,
    finish: Option<&str>,
) -> Result<bool, OwnerCallError>
```

Commits `upload`. With `finish`, the owner also completes the job
(recording the text as its progress) and the client sends no
`complete` afterwards. Answers whether the job is finished.

##### Errors

Returns [`OwnerCallError`](#ownercallerror); `Lost` also cancels the job.

Implements: `Debug`

### HostJobError

```rust
pub struct HostJobError {
    pub kind: String,
    pub message: String,
}
```

How a job failed: a stable kind and an operator-facing message. Neither
may contain a secret.

**Fields**

- `kind`: Stable error kind recorded on the owner's job row.
- `message`: Operator-facing message.

#### HostJobError::new

```rust
pub fn new(kind: impl Into<String>, message: impl Into<String>) -> Self
```

A failure of `kind`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### HostUpload

```rust
pub struct HostUpload {
    pub upload_id: String,
    /* private fields */
}
```

An open upload of one title: its id and bearer token. The token is
zeroized on drop and never shown by `Debug`.

**Fields**

- `upload_id`: Upload id.

Implements: `Debug`, `Drop`

## Enums

### ClientExit

```rust
pub enum ClientExit {
    Shutdown,
    Revoked,
}
```

Why [`HostClient::run`](#hostclientrun) returned.

**Variants**

- `Shutdown`: The shutdown signal fired.
- `Revoked`: The owner closed with [`CLOSE_REVOKED`]: this node's credential is
  revoked. The caller should mark the link revoked.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DialError

```rust
pub enum DialError {
    Insecure(&'static str),
    Invalid(String),
    Io(Error),
    Tls(String),
    Refused(u16),
    Handshake(String),
    Http,
    Timeout,
}
```

Why the owner could not be reached.

**Variants**

- `Insecure`: The transport policy refuses this origin (plain HTTP to a
  non-loopback host, or a pin on a plain-HTTP origin).
- `Invalid`: The origin or pin is malformed.
- `Io`: TCP connect or I/O failed.
- `Tls`: The TLS handshake failed: a wrong pin, an untrusted certificate, or
  a protocol error.
- `Refused`: The owner answered the upgrade with an HTTP status.
- `Handshake`: The WebSocket handshake failed otherwise.
- `Http`: The owner's HTTP answer was malformed or too large.
- `Timeout`: A step did not finish in time.

Implements: `Debug`, `Display`, `Error`, `From<DialError>`, `From<Error>`

### JoinError

```rust
pub enum JoinError {
    Link(String),
    Dial(DialError),
    Refused,
    OriginMismatch { origin: String },
    Http { status: u16, error: String },
    Record(String),
    Cycle,
    AncestryTooDeep,
    Unsaved { node_id: Option<String>, reason: String },
    SaveFailed { node_id: String, reason: String },
}
```

Why joining failed.

**Variants**

- `Link`: The text is not a whole `https://<owner>/enroll#bcn1...` link.
- `Dial`: The owner could not be reached under the transport policy.
- `Refused`: The owner refused the link (spent, expired, revoked, or unknown), or
  the sign-in's code.
- `OriginMismatch`: The library names another origin as its own; join at that one.
  - `origin`: The origin the library names.
- `Http`: The owner answered with another status.
  - `status`: HTTP status.
  - `error`: The answer's `error` code.
- `Record`: The enroll request could not be built, or the library's `info`
  (its ancestry) could not be read.
- `Cycle`: The library is this one, or one of its own branches (its ancestry
  names this library): joining it would make a cycle.
- `AncestryTooDeep`: Joining the library would give this one more ancestors than an
  ancestry can name, so a later join under it could not be checked
  for a loop.
- `Unsaved`: The library accepted the join (an invite link or a sign-in code),
  but its answer did not parse or the local write failed. The one-use
  credential is spent and the node keys are gone, so this install
  cannot use the node the library minted: its owner removes that node
  (named when the answer said which) and this install joins again.
  - `node_id`: The node id the owner minted, when its answer named one.
  - `reason`: What failed.
- `SaveFailed`: A sign-in's code was redeemed and the library enrolled this host,
  but the local write failed. The library answers a repeat of the
  same exchange (code, verifier, keys) with the same node, so a retry
  with the sign-in still pending can finish the join.
  - `node_id`: The node id the library minted.
  - `reason`: What failed.

#### JoinError::is_retryable

```rust
pub fn is_retryable(&self) -> bool
```

True for a sign-in redeem failure that a repeat of the same exchange
(code, verifier, keys) can fix: the library unreachable (its answer
may have been lost after it enrolled the node, which a repeat gets
again), busy, or the local write failed.

Implements: `Debug`, `Display`, `Error`, `From<DialError>`

### OwnerCallError

```rust
pub enum OwnerCallError {
    Lost,
    Refused(String),
    Unavailable,
}
```

Why a call the job makes to the owner (results, plan, upload) failed.

**Variants**

- `Lost`: The fence is lost; the job must stop (it is already cancelled).
- `Refused`: The owner refused it; the reason is stable.
- `Unavailable`: No connection to the owner, or no usable answer.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### WriteBackError

```rust
pub enum WriteBackError {
    NotHanded,
    Size,
    Lost,
    Refused(String),
    Unavailable,
}
```

Why a token write-back was not stored.

**Variants**

- `NotHanded`: The ref was not handed with this job; nothing was sent.
- `Size`: The value is empty or larger than the owner accepts; nothing was
  sent.
- `Lost`: The fence is lost; the job must stop.
- `Refused`: The owner refused it (`fence_not_live`, `not_trusted`, ...).
- `Unavailable`: No connection to the owner, or it did not answer in time.

Implements: `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### announce_leaving

```rust
pub async fn announce_leaving(
    link: ClusterLink,
    keys: ClusterLinkKeys,
    software_version: &str,
    sampler: Box<dyn HeartbeatSource>,
    timeout: Duration,
) -> bool
```

Tells the library of `link` that this install is leaving: connects with
the link's keys (claiming nothing; `sampler` answers the connection's
first heartbeat), sends `node.leaving`, and waits for the answer for up
to `timeout`. Answers true when the library accepted it. Call it before
the keys are deleted.

A running client for the same link is superseded by this connection;
the library then revokes the node, so that client stops too.

### authorize_branch

```rust
pub fn authorize_branch(
    library: &InspectedLibrary,
    redirect_uri: &str,
    branch_name: &str,
) -> Result<BranchAuthorization, JoinError>
```

Starts a sign-in at `library` for this install as `branch_name`, coming
back to `redirect_uri` (a loopback or https URL).

#### Errors

Returns [`JoinError::Link`](#joinerror) for a redirect URI the library would refuse
or a malformed branch name.

### authorize_join

```rust
pub fn authorize_join(
    library: &InspectedLibrary,
    redirect_uri: &str,
    branch_name: &str,
    offer_clerk: bool,
) -> Result<BranchAuthorization, JoinError>
```

[`authorize_branch`](#authorize_branch), optionally offering to join as a clerk
(`offer_clerk`, [`CLERK_OFFER_PARAM`](../bookclerk_node_proto/index.md#clerk_offer_param)): the consent page then lets an
Owner of the library choose a clerk instead of a branch. Offer it only
for an install that holds no library data of its own.

#### Errors

As [`authorize_branch`](#authorize_branch).

### inspect_library

```rust
pub async fn inspect_library(address: &str) -> Result<InspectedLibrary, JoinError>
```

Reads what the library at `address` says about itself, under the
transport policy (plain HTTP only on loopback).

- A `self_signed` library must present the certificate whose pin it
  reports; the pin is answered for the operator to confirm.
- Any other TLS library must present a certificate that chains to the
  public roots.
- The library must name the origin it was reached at as its own (the
  origin its node handshake signs), so the link is recorded with an
  origin that works.

#### Errors

Returns [`JoinError`](#joinerror) when the address is malformed, the library is
unreachable or untrusted, or its answer is malformed or names another
origin.

### library_origin

```rust
pub fn library_origin(address: &str) -> Result<String, JoinError>
```

Normalizes what an operator typed (`owner.example:8443`,
`https://owner.example/`) to a canonical origin. A bare `host[:port]`
means `https`.

#### Errors

Returns [`JoinError::Link`](#joinerror) when it is not an http(s) origin.

### redeem_branch_code

```rust
pub async fn redeem_branch_code(
    store: &LibraryStore,
    redeem: &BranchRedeem<'_>,
    keys: &ClusterLinkKeys,
) -> Result<ClusterLink, JoinError>
```

Redeems an approved sign-in: the library runs the same redeem as for a
link (one node, a receipt, idempotent retries), then this install seals
`keys` and records the link. The code and verifier go once in the
request body and are never logged.

#### Errors

Returns [`JoinError`](#joinerror) when the library is unreachable or refuses; once
the library has enrolled a node, a malformed answer or one for another
cluster gives [`JoinError::Unsaved`](#joinerror), which names that node so it can be
revoked there, as does a link this install refuses to record. A failed
database or disk write gives [`JoinError::SaveFailed`](#joinerror) (redeeming the
same code again with the same verifier and keys gets the same node, so
that one can be retried, naming that node in
[`BranchRedeem::enrolled_node_id`](#branchredeem)).

### apply_clerk_bootstrap

```rust
pub fn apply_clerk_bootstrap(
    files_dir: &Path,
    config_file: &Path,
    bootstrap: &ClerkBootstrap,
    password: Option<&str>,
) -> Result<AppliedClerk, JoinError>
```

Writes `bootstrap` as this install's own bootstrap under `files_dir`:

- `database-postgres.url` (Postgres only): the URL, mode `0600`;
- `node-identity.json`: the node id the library recorded, bound to its
  cluster;
- `master.key`: the library's DEK, wrapped with `password` (this
  install's `BOOKCLERK_AUTH_PASSWORD` or `[auth].password`) when set;
- `[database]` in `config_file` (other settings kept): `postgres` with
  `url_file` naming the URL file (never the URL itself in the TOML), or
  `d1` with its ids (the D1 API token stays this node's
  environment-only bootstrap).

The files go in as one recoverable step. All the new contents are first
written and synced beside their targets (hidden `.<name>.clerk-<hex>`
files), and each existing target is copied to `<name>.pre-clerk`; a
failure up to here changes no target. The targets are then replaced
atomically one by one, in the order above. If one fails, the ones
already replaced are put back from their copies (a target that did not
exist before is removed), so the install keeps its own bootstrap; the
error names any file that could not be put back. On success the copies
stay, and `clerk-join.json` in `files_dir` records the join: the next start drops
them once the library's database opens ([`PendingClerkJoin::confirm`](#pendingclerkjoinconfirm))
and puts the originals back if it does not ([`PendingClerkJoin::revert`](#pendingclerkjoinrevert)).
The caller restarts the daemon afterwards.

A process that dies while replacing leaves the files before that point
in the list new and the rest old, with every original still beside its
file as `<name>.pre-clerk` and no record; a later apply refuses while a
copy is there, and so does the daemon ([`interrupted_clerk_join`](#interrupted_clerk_join)), with
instructions to finish the join by hand (when `config_file` is already
new) or undo it. Staged `.<name>.clerk-<hex>` files left by such a crash
are unused and can be deleted.

#### Errors

Returns [`JoinError::Record`](#joinerror) when a file cannot be written or the
existing `config.toml` does not parse.

### bootstrap_database_kind

```rust
pub fn bootstrap_database_kind(database: &ClerkDatabase) -> DatabasePluginKind
```

The `[database].plugin` a clerk's bootstrap configures.

### checked_d1_api_base

```rust
pub fn checked_d1_api_base(api_base: &str) -> Result<String, JoinError>
```

Checks the D1 API origin a library's bootstrap names, and returns it
normalized. Every D1 query carries the joining server's own API token
to that origin, so a clerk may only be pointed at Cloudflare's API over
HTTPS. The library checks its own origin the same way before it offers
a clerk join, so a library on another D1 endpoint cannot take clerks.

#### Errors

Returns [`JoinError::Record`](#joinerror) when `api_base` is not an `https` URL on
`api.cloudflare.com` (no other port, user name or password).

### database_guest_installed

```rust
pub fn database_guest_installed(config: &Config, kind: DatabasePluginKind) -> bool
```

True when the database guest for `kind` is installed for `config`, so
the daemon can open the library's database once the bootstrap is
applied. A plugin directory that cannot be read counts as not
installed.

### install_problem

```rust
pub async fn install_problem(store: &LibraryStore) -> Result<Option<String>, JoinError>
```

Why this install may not join as a clerk now, or `None`.

#### Errors

Returns the read error.

### interrupted_clerk_join

```rust
pub fn interrupted_clerk_join(files_dir: &Path, config_file: &Path) -> Option<String>
```

Names the copies an interrupted clerk join left beside this install's
bootstrap files, or `None` when there are none. The daemon refuses to
start while any is there: a process that died while replacing the files
leaves some new and some old, and starting on that mix would pair a
database with the wrong node identity or key. The message says how to
finish or undo the join by hand (see [`apply_clerk_bootstrap`](#apply_clerk_bootstrap)).

### pending_clerk_join

```rust
pub fn pending_clerk_join(
    files_dir: &Path,
    config_file: &Path,
) -> Option<PendingClerkJoin>
```

The clerk join this install applied but has not yet started on (or
began undoing), if any. A record that cannot be read, or that names any
file other than this install's own bootstrap files and their copies, is
not used: copies left beside those files then stop the start (see
[`interrupted_clerk_join`](#interrupted_clerk_join)).

### redeem_clerk_code

```rust
pub async fn redeem_clerk_code(
    redeem: &BranchRedeem<'_>,
    keys: &ClusterLinkKeys,
) -> Result<ClerkJoined, JoinError>
```

Redeems an approved clerk sign-in and opens the bootstrap with the
node's box secret. The code and verifier go once in the request body,
and are registered for redaction (the caller unregisters them).

#### Errors

Returns [`JoinError`](#joinerror) when the library is unreachable or refuses (the
code was approved for a branch, the user is no longer an Owner, or its
database cannot be shared). Once the library has enrolled the node, an
answer for another cluster or type, without a bootstrap, or with one
that does not open for this node gives [`JoinError::Unsaved`](#joinerror) naming
it. [`JoinError::is_retryable`](#joinerroris_retryable) tells which failures a repeat of the
same exchange can fix (the library answers it with the same node and a
freshly sealed bootstrap).

### check_join_ancestry

```rust
pub async fn check_join_ancestry(
    store: &LibraryStore,
    endpoint: &OwnerEndpoint,
) -> Result<Vec<String>, JoinError>
```

Reads the library's `info` over `endpoint` and refuses a library whose
ancestry names this install's own cluster id (A ← B ← A). Answers its
ancestry, to record on the link.

#### Errors

Returns [`JoinError::Cycle`](#joinerror) for a cycle, and [`JoinError`](#joinerror) when the
library cannot be read (fail closed: no join without the check).

### join_cluster

```rust
pub async fn join_cluster(
    store: &LibraryStore,
    link: &str,
    software_version: &str,
) -> Result<ClusterLink, JoinError>
```

Enrolls this install into the owner cluster named by `link`.

Generates fresh node keys, redeems the link at the owner (under the
transport policy and the link's SPKI pin), then seals the keys into this
install's `encrypted_secrets` and records the `cluster_links` row. The
link secret is sent once in the request body and never logged.

`link` is the whole link as minted (`https://owner/enroll#bcn1...`):
the origin it names is the one this host dials and signs from then on.

#### Errors

Returns [`JoinError`](#joinerror) when the link is malformed, the owner is
unreachable or refuses, or the local write fails.

### refuse_cycle

```rust
pub async fn refuse_cycle(
    store: &LibraryStore,
    ancestry: &[String],
) -> Result<(), JoinError>
```

Refuses `ancestry` when it names this install's own cluster id, is
malformed, or would take this library's ancestry past what one can name
([`join_overflows_ancestry`](../bookclerk_library/control_plane.md#join_overflows_ancestry)).

#### Errors

Returns [`JoinError::Cycle`](#joinerror), [`JoinError::AncestryTooDeep`](#joinerror) or
[`JoinError::Record`](#joinerror).

## Type aliases

### OwnerSocket

```rust
pub type OwnerSocket = WebSocketStream<Box<dyn OwnerIo>>;
```

The owner socket.

## Constants

### CLERK_POSTGRES_URL_FILE

```rust
pub const CLERK_POSTGRES_URL_FILE: &str = "database-postgres.url";
```

File under the files directory that holds a clerk's Postgres URL, mode
`0600`. `[database.postgres].url_file` names it by its absolute path
(a relative `url_file` would be read from the daemon's working
directory).

### DATABASE_ENV_OVERRIDES

```rust
pub const DATABASE_ENV_OVERRIDES: &[&str] = _;
```

Environment variables that would outrank a clerk's `[database]`
bootstrap; a clerk join is refused while one is set.

### AWAITING_HOST_CONSENT

```rust
pub const AWAITING_HOST_CONSENT: &str = "awaiting_host_consent";
```

Release reason for a job whose plugins this host's operator has not
(or no longer) approved.

### KIND_NOT_RUN_HERE

```rust
pub const KIND_NOT_RUN_HERE: &str = "kind_not_run_here";
```

Release reason for a job of a kind this host does not run.

### MAX_ARTIFACT_BYTES

```rust
pub const MAX_ARTIFACT_BYTES: usize = bookclerk_library::control_plane::MAX_ARTIFACT_BLOB_BYTES;
```

Largest artifact fetched from the owner (the library blob cap).

### MAX_LEASE

```rust
pub const MAX_LEASE: Duration = _;
```

Longest lease this host counts on from one `work` or `renewed`; a
longer grant is renewed as if it were this long.

### SECRETS_UNOPENABLE

```rust
pub const SECRETS_UNOPENABLE: &str = "secrets_unopenable";
```

Error kind reported when a job's secrets do not open with this node's
box key.

### CONNECT_PATH

```rust
pub const CONNECT_PATH: &str = "/api/node/v1/connect";
```

Path of the node socket.

### ENROLL_PATH

```rust
pub const ENROLL_PATH: &str = "/api/node/v1/enroll";
```

Path of the enroll endpoint.
