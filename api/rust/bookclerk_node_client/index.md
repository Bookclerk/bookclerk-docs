# bookclerk_node_client

The host side of the node API (H-4 design, sections 2 to 4; step 7).

A host node is a separate Bookclerk install that enrolled into an owner
cluster with a single-use link ([`join_cluster`](enroll.md#bookclerk_node_clientenrolljoin_cluster)) or a sign-in at the
owner ([`inspect_library`](branch.md#bookclerk_node_clientbranchinspect_library), [`authorize_branch`](branch.md#bookclerk_node_clientbranchauthorize_branch),
[`redeem_branch_code`](branch.md#bookclerk_node_clientbranchredeem_branch_code)); it tells the owner when it leaves
([`announce_leaving`](branch.md#bookclerk_node_clientbranchannounce_leaving)). An empty install may instead join as a clerk, an
owner node that shares the library's database and DEK
([`redeem_clerk_code`](clerk.md#bookclerk_node_clientclerkredeem_clerk_code), [`apply_clerk_bootstrap`](clerk.md#bookclerk_node_clientclerkapply_clerk_bootstrap)). Its node keys are
sealed in its own database
(`bookclerk_library::control_plane::record_cluster_link`). For every
active link, [`HostClient`](client.md#bookclerk_node_clientclienthostclient) keeps one socket to the owner's
`/api/node/v1/connect`:

- TLS everywhere except loopback, verified against the link's SPKI pin
  or the webpki roots ([`OwnerEndpoint`](endpoint.md#bookclerk_node_clientendpointownerendpoint));
- the Ed25519 challenge signed with the enrolled key;
- heartbeats into the owner's `node_status`;
- claimed jobs run through a [`HostJobRunner`](jobs.md#bookclerk_node_clientjobshostjobrunner), their secrets opened
  with the enrolled X25519 key, renewed and reported under the fence;
- refreshed store tokens written back sealed to the connection's owner
  box key (`HostJobCtx::write_back`);
- wakes acked after local subscribers have them;
- reconnect with backoff, resuming the fences still held.

- claims narrowed to the job kinds this host runs and the plugins its
  operator consented to (`HostClientHandle::set_consented_plugins`);
  a job whose consent is revoked is cancelled and given back unspent;
- after a wake, the host requests its deployment list and instance
  configuration from the owner and sends its observations back
  (`HostClientHandle::request` for `deployments.list`,
  `instance.config` and `observation`), and fetches a stored archive
  with `HostClientHandle::fetch_artifact`, signed with the node key.

- a scan's rows returned in fenced batches
  (`HostJobCtx::send_scan_results`); an acquire's titles asked for
  (`HostJobCtx::acquire_plan`) and its files streamed to the owner
  under a sealed upload token (`HostJobCtx::grant_upload`,
  `HostJobCtx::upload_file`, `HostJobCtx::commit_upload`); the job
  completes with its closing upload's commit.

The consent decisions themselves (which plugins, which grant) and how a
job runs its guest are the daemon's.

## Modules

### [`branch`](branch.md)

*3 structs, 6 functions*

### [`clerk`](clerk.md)

*2 constants, 4 structs, 8 functions*

### [`client`](client.md)

*1 enum, 1 trait, 5 constants, 6 structs*

### [`endpoint`](endpoint.md)

*1 enum, 1 trait, 1 type alias, 2 constants, 2 structs*

### [`enroll`](enroll.md)

*1 enum, 3 functions*

### [`jobs`](jobs.md)

*1 trait, 2 enums, 5 structs*

