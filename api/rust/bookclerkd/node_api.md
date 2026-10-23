**bookclerkd > node_api**

# Module: node_api

## Contents

**Modules**

- [`branch`](#branch) - Join a library with a sign-in: the owner side of the OAuth code + PKCE
- [`tls`](#tls) - The built-in TLS listener for the node API (`[daemon.node_api] mode =
- [`transport`](#transport) - Transport policy for the node API (H-4 design section 4, as amended by
- [`ws`](#ws) - The host-node socket: `GET /api/node/v1/connect` (H-4 design, section 3).

**Structs**

- [`NodeInvitesQuery`](#nodeinvitesquery) - `GET /api/config/node-invites` query.

**Functions**

- [`delete_node_credential`](#delete_node_credential) - `DELETE /api/config/node-credentials/{node_id}`: revoke an enrolled
- [`delete_node_invite`](#delete_node_invite) - `DELETE /api/config/node-invites/{id}`: revoke an unredeemed link.
- [`get_host_enrollment`](#get_host_enrollment) - `GET /api/config/host-enrollment`: whether host nodes may enroll.
- [`get_node_invites`](#get_node_invites) - `GET /api/config/node-invites`: every link for the operator or an
- [`node_router`](#node_router) - The node API routes, as every listener mounts them: outside the cookie
- [`post_enroll`](#post_enroll) - `POST /api/node/v1/enroll`: redeem an enrollment link. Answers
- [`post_node_invite`](#post_node_invite) - `POST /api/config/node-invites`: mint an enrollment link. Any user may
- [`put_host_enrollment`](#put_host_enrollment) - `PUT /api/config/host-enrollment`: turn host enrollment on or off. The

---

## bookclerkd::node_api::NodeInvitesQuery

*Struct*

`GET /api/config/node-invites` query.

**Fields:**
- `mine: bool` - Only the links for the calling user's own branches (an Owner's

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> NodeInvitesQuery`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## Module: branch

Join a library with a sign-in: the owner side of the OAuth code + PKCE
flow of the built-in public client [`BRANCH_CLIENT_ID`](../bookclerk_node_proto/join.md#bookclerk_node_protojoinbranch_client_id).

- `GET /api/node/v1/info` ([`get_info`](node_api/branch.md#bookclerkdnode_apibranchget_info)): the cluster id, a library
  name, the canonical origin, the node API transport mode, whether new
  branches may join, and in `self_signed` mode the SPKI fingerprint the
  joining operator confirms. Outside the cookie and auth layers, after
  the transport policy.
- `GET|POST /api/auth/branch-consent` ([`get_branch_consent`](node_api/branch.md#bookclerkdnode_apibranchget_branch_consent),
  [`post_branch_consent`](node_api/branch.md#bookclerkdnode_apibranchpost_branch_consent)): the consent page's API. `/oidc/authorize`
  sends a [`BRANCH_CLIENT_ID`](../bookclerk_node_proto/join.md#bookclerk_node_protojoinbranch_client_id) request to the SPA page `/join/approve`,
  which signs the user in and asks "Let *branch* join *library* as
  yours?". Approving checks the signed-in user is active, host
  enrollment is on and the connection passes the transport policy, then
  issues a single-use code that lives [`BRANCH_CODE_TTL_SECS`](../bookclerk_node_proto/join.md#bookclerk_node_protojoinbranch_code_ttl_secs).
- `POST /api/node/v1/enroll/oauth` ([`post_enroll_oauth`](node_api/branch.md#bookclerkdnode_apibranchpost_enroll_oauth)): the joining
  daemon redeems the code with its PKCE verifier and the node's public
  keys. The code is read and checked (client, redirect URI, scope and
  the consented name and type it holds, S256 verifier), its link
  recorded under the consented name, and only then consumed. The
  redeem is the same canonical batch a minted link takes
  ([`redeem_enrollment_link_as`](../bookclerk_library/control_plane/enrollment.md#bookclerk_librarycontrol_planeenrollmentredeem_enrollment_link_as)), through a link whose id derives from
  the code and whose secret from the code, verifier and node kind
  ([`authorized_enrollment_link`](../bookclerk_library/control_plane/enrollment.md#bookclerk_librarycontrol_planeenrollmentauthorized_enrollment_link)):
  one node row, `node_credentials`,
  a receipt, idempotent retries, and one node under concurrency.
- `GET|PUT /api/config/branches` ([`get_branches`](node_api/branch.md#bookclerkdnode_apibranchget_branches), [`put_branches`](node_api/branch.md#bookclerkdnode_apibranchput_branches)):
  the owner's Settings → Branches (operator or Owner): the fingerprint,
  "Let new branches join", and each branch with its two sharing
  settings as it last reported them (read-only here; the branch's
  operator sets them) and counts of the titles it shared.
- `POST /api/config/branches/titles/remove` `{node_id?}`
  ([`post_remove_branch_titles`](node_api/branch.md#bookclerkdnode_apibranchpost_remove_branch_titles)): removes, in bulk, the entries of
  titles from a branch that stopped sharing or left (of one branch, or
  all). Files already delivered stay in this library's outputs.

**Clerks.** When the joining install offers it ([`bookclerk_node_proto::CLERK_OFFER_PARAM`]:
it is empty) and the signed-in user is an Owner (the stored role, read
at consent and again at redeem), the consent page also offers "Join as
a clerk of this library": an owner node (`kind = owner`) that shares
this library's database and DEK. That needs a database more than one
server can open (PostgreSQL or D1, never the SQLite file:
`shared_database`) and a fresh second factor on the consent page
(TOTP or a user-verified passkey, [`post_branch_consent_step_up`](node_api/branch.md#bookclerkdnode_apibranchpost_branch_consent_step_up)); an
Owner with neither is refused. The code is bound to the type chosen (the
granted scope, `ConnectionType::granted_scope`, stored with the
consented branch name), so a branch code is never redeemed as a clerk. A clerk's redeem answer carries the
`[database]` bootstrap and the DEK sealed to the joining node's box key
([`seal_clerk_bootstrap`](../bookclerk_node_proto/clerk.md#bookclerk_node_protoclerkseal_clerk_bootstrap)); never the `master.key` file, never through
the browser, never logged. Both the consent and the redeem run under the
node API transport policy: TLS, or loopback when both ends are on this
machine.



## bookclerkd::node_api::delete_node_credential

*Function*

`DELETE /api/config/node-credentials/{node_id}`: revoke an enrolled
host's credential so it can no longer connect.

# Arguments

* `caller` - Who is asking; the operator or an Owner may revoke any host,
  any other active user only the hosts they own.
* `node_id` - The host node to revoke.

# Returns

`{credential}` with `revoked_at` set; revoking twice keeps the first
time.

# Errors

Returns 404 when the node is unknown or not the caller's, 400 when the
node has no host credential, 403 for a disabled user, and an error
response when the write fails.

```rust
fn delete_node_credential(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, __arg2: axum::extract::Path<String>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::node_api::delete_node_invite

*Function*

`DELETE /api/config/node-invites/{id}`: revoke an unredeemed link.

```rust
fn delete_node_invite(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, __arg2: axum::extract::Path<String>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::node_api::get_host_enrollment

*Function*

`GET /api/config/host-enrollment`: whether host nodes may enroll.

# Returns

`{enabled, revision}`; `revision` is `null` while the default (enabled)
applies.

# Errors

Returns an error response when the switch cannot be read.

```rust
fn get_host_enrollment(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::node_api::get_node_invites

*Function*

`GET /api/config/node-invites`: every link for the operator or an
Owner; the caller's own links for anyone else, and with `?mine=true`.

```rust
fn get_node_invites(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, __arg2: axum::extract::Query<NodeInvitesQuery>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::node_api::node_router

*Function*

The node API routes, as every listener mounts them: outside the cookie
and operator auth layers.

```rust
fn node_router<S>(state: std::sync::Arc<crate::api::AppState>) -> axum::Router<S>
```



## bookclerkd::node_api::post_enroll

*Function*

`POST /api/node/v1/enroll`: redeem an enrollment link. Answers
`{cluster_id, node_id}`; 403 `transport_insecure` when the connection
fails the transport policy, 403 `enrollment_refused` for any link that
does not redeem.

```rust
fn post_enroll(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, request: axum::extract::Request) -> axum::response::Response
```



## bookclerkd::node_api::post_node_invite

*Function*

`POST /api/config/node-invites`: mint an enrollment link. Any user may
mint one for themselves while host enrollment is on; the operator or an
Owner may mint for any active user.

# Arguments

* `caller` - Who is asking; decides whose links they may mint.
* `body` - Owner, lifetime, and label of the new link.

# Returns

`{link, url}`; the secret appears once, in `url`.

# Errors

Returns 503 `node_api_pin_unavailable` in `self_signed` mode while the
certificate's SPKI pin cannot be read (nothing is minted), and an error
response when the library refuses the mint.

```rust
fn post_node_invite(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, __arg2: axum::Json<PostNodeInvite>) -> Result<axum::response::Response, axum::response::Response>
```



## bookclerkd::node_api::put_host_enrollment

*Function*

`PUT /api/config/host-enrollment`: turn host enrollment on or off. The
operator may always; a user only while the stored role is an active
Owner.

# Arguments

* `caller` - Who is asking; a user's role is read from the database.
* `body` - The new switch value.

# Returns

`{enabled, revision}` as stored.

# Errors

Returns 403 when the caller is neither the operator nor an active Owner,
and an error response when the switch cannot be written.

```rust
fn put_host_enrollment(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, __arg2: axum::Json<PutHostEnrollment>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## Module: tls

The built-in TLS listener for the node API (`[daemon.node_api] mode =
"acme" | "self_signed"`, H-4 design section 4 and decision 1).

- `acme`: certificates for `integrations.public_origin`'s host come from
  an ACME CA (Let's Encrypt by default) over TLS-ALPN-01, answered on
  this same listener, and are cached under `$BOOKCLERK_FILES_DIR/acme/`
  (mode 0700). Renewal is automatic.
- `self_signed`: a P-256 certificate generated once and kept under
  `$BOOKCLERK_FILES_DIR/node-api/self-signed/` (mode 0700). Its SPKI pin
  (base64url SHA-256 of the SubjectPublicKeyInfo) is written beside it
  and travels in enrollment links.

The listener serves HTTP/1.1 (so WebSocket upgrades work) and tags each
request `ListenerKind::AcmeTls` or `ListenerKind::SelfSignedTls` for
[`super::transport::classify_transport`](node_api/transport.md#bookclerkdnode_apitransportclassify_transport). Given no router,
[`start_with`](node_api/tls.md#bookclerkdnode_apitlsstart_with) serves only the node API routes; the daemon passes its
whole web app,
because `integrations.public_origin` names this listener and a joining
branch's owner signs in there (Join a library: `/oidc/authorize`, the
consent page, sign-in), as user invites and SSO callbacks already
assume. The web app keeps its own authentication on every route.

[`NodeApiGate`](node_api/tls.md#bookclerkdnode_apitlsnodeapigate) keeps the node API unmounted, on every listener, until
the TLS listener is serving a certificate. If ACME fails, or the listener
cannot bind, every node API route answers 404 `node_api_unmounted`;
there is no plain-HTTP fallback. `[daemon.node_api]` is bootstrap
configuration: changing it needs a daemon restart.



## Module: transport

Transport policy for the node API (H-4 design section 4, as amended by
decision 1: TLS everywhere except loopback).

[`classify_transport`](node_api/transport.md#bookclerkdnode_apitransportclassify_transport) is a pure function of the connection, the request
headers, and the bootstrap `[daemon.node_api]` settings. It runs before
every enroll (and, later, every node socket upgrade and upload). A
connection is allowed in exactly one mode:

- `acme`: it arrived on the built-in rustls listener with an ACME
  certificate (`[daemon.node_api] mode = "acme"`).
- `self_signed`: it arrived on the built-in rustls listener with a pinned
  self-signed certificate (`mode = "self_signed"`).
- `proxy`: plain HTTP from a peer in `daemon.trusted_proxies`, carrying
  exactly one `X-Forwarded-Proto: https`, with an https
  `integrations.public_origin` (`mode = "proxy"`).
- `loopback`: both ends are loopback and no forwarding header is present.
  Allowed in every mode: the traffic never leaves the machine. In proxy
  mode a trusted proxy's connection is always checked as proxied, so a
  same-host proxy cannot relay a request as loopback.

Everything else is refused with 403 `transport_insecure`. Forwarding
headers from a peer that is not a trusted proxy are never believed, and
plain HTTP from any non-loopback peer (a private RFC 1918 address
included) is refused.



## Module: ws

The host-node socket: `GET /api/node/v1/connect` (H-4 design, section 3).

The route sits outside the cookie and operator auth layers and is never
disabled by `auth.enabled = false`: the node's enrolled Ed25519 key is
the credential. Order of checks:

1. [`super::transport::classify_transport`](node_api/transport.md#bookclerkdnode_apitransportclassify_transport), before the upgrade.
2. The client must offer the [`SUBPROTOCOL`](../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protosubprotocol) subprotocol.
3. hello → challenge → auth. The owner always sends a challenge, so an
   unknown node id and a bad signature look the same from outside; every
   authentication failure closes with [`CLOSE_AUTH_FAILED`](../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protoclose_auth_failed) and the one
   reason `AUTH_FAILED_REASON`. A valid signature from a revoked
   credential closes with [`CLOSE_REVOKED`](../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protoclose_revoked). Each failure writes a log
   line with the detailed reason and a `security_audit_events` row, at
   most `AUDIT_ROWS_PER_KEY` per peer, claimed node and reason, and
   `AUDIT_ROWS_PER_WINDOW` in all, per `AUDIT_WINDOW`, so
   unauthenticated peers cannot grow the table without bound.
   The connection is registered for `node.revoked` wakes before its
   credential is stamped, so a revoke during the handshake still closes
   it.
4. At most `MAX_PENDING_HANDSHAKES` upgraded connections may be in the
   handshake at once; each holds a permit until `welcome` or refusal,
   and an upgrade beyond that answers 503 `handshakes_busy`.

After `welcome` (and `resumed`, when the hello listed fences the host
still holds; see `super::work::resume`), the session pings every
[`PING_INTERVAL_SECS`](../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protoping_interval_secs), closes after [`IDLE_TIMEOUT_SECS`](../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protoidle_timeout_secs) without an
inbound frame, re-reads the credential on every ping tick, and closes
with [`CLOSE_REVOKED`](../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protoclose_revoked) as soon as a `node.revoked` wake arrives
([`close_node_sessions`](node_api/ws.md#bookclerkdnode_apiwsclose_node_sessions)). Binary frames and malformed frames close with
[`CLOSE_PROTOCOL`](../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protoclose_protocol).

Request frames are served by `super::work` (heartbeat, job claim,
renew, complete, fail, token write-back) and the host's `node.<id>`
queue is forwarded as `wake` frames by `super::wakes`. Everything the
owner sends goes through one bounded queue of [`SEND_QUEUE_FRAMES`](../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protosend_queue_frames)
frames drained by a writer task: wakes are only claimed when the queue
has room, and an answer that cannot be queued within
[`IDLE_TIMEOUT_SECS`](../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protoidle_timeout_secs) (the host stopped reading) ends the connection.
Nothing is released when a connection ends: job leases expire on their
own unless the host resumes them. The one exception is a `work` answer
produced after the connection was told to close (a newer connection
superseded it, or the node was revoked) or that cannot be queued: the
host never sees that job, so its claim is given back unspent at once.

A newer connection for the same host shares the owner node's wake
catalog row with the one it supersedes. Promoting a welcomed connection
to live with enabling the row, and unregistering one with disabling it,
run under one lock, and only the connection still live disables the row,
so a superseded session ending late never switches off the live
session's wakes.



