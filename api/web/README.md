# Bookclerk daemon API

The HTTP API of `bookclerkd`, the Bookclerk daemon: the web interface, the CLI and other nodes use it. Request bodies are JSON (`Content-Type: application/json`; send `{}` for defaults) unless a route says otherwise. Error responses are JSON objects with an `error` slug (`ApiError`); bare statuses get `{"error", "message", "status"}` (`ErrorJson`).

The daemon serves this reference as an OpenAPI document at `GET /api/openapi.json` (API version `0.1.0`).

## Authentication

Each route lists the credentials it accepts and who may call it. The credentials are:

| Name | Kind | Description |
| --- | --- | --- |
| `nodeSignature` | header `Authorization` | A node's signed request, as `Authorization: Bookclerk-Node <node_id> <issued_at> <signature>` with the node's identity key. |
| `oidcAccessToken` | HTTP bearer | An access token from this daemon's OIDC token endpoint (`POST /oidc/token`), as `Authorization: Bearer <token>`. |
| `operatorSession` | cookie `bookclerk_operator_session` | The operator's browser session, set by `POST /api/auth/login`. |
| `operatorToken` | HTTP bearer | The operator token (`bookclerk daemon token`), as `Authorization: Bearer <token>`. |
| `portalSession` | cookie `bookclerk_portal_session` | A library user's browser session, set by a sign-in route. Cookie-authenticated writes must send an `Origin` (or `Referer`) matching the daemon's public origin or its host. |
| `uploadToken` | header `Authorization` | The token of an upload grant, as `Authorization: Bookclerk-Upload <token>`. |

## Routes

### [Health](health.md)

Liveness.

| Route | Summary |
| --- | --- |
| [`GET /health`](health.md#get-health) | Serves `GET /health` with a static `ok` payload for liveness probes. |

### [Spec](spec.md)

This document.

| Route | Summary |
| --- | --- |
| [`GET /api/openapi.json`](spec.md#get-apiopenapijson) | Serves the OpenAPI document. |

### [Auth](auth.md)

Sign-in, sessions, passkeys, TOTP and the operator's elevation.

| Route | Summary |
| --- | --- |
| [`POST /api/auth/login`](auth.md#post-apiauthlogin) | Validates the operator token (throttled, constant-time) and issues a hashed session cookie. |
| [`GET /api/auth/signin`](auth.md#get-apiauthsignin) | Public sign-in picker: operator-token availability, SSO IdPs, integration logins. |
| [`POST /api/auth/tray-handoff/prepare`](auth.md#post-apiauthtray-handoffprepare) | Mint a single-use tray Open Bookclerk ticket (Bearer, direct loopback only). |
| [`GET /api/auth/tray-handoff`](auth.md#get-apiauthtray-handoff) | Consume the pending tray ticket and set a localhost operator session cookie. |
| [`GET /api/auth/me`](auth.md#get-apiauthme) | Resolves the caller and returns `/me` JSON; unauthenticated callers get 401. |
| [`POST /api/auth/elevate`](auth.md#post-apiauthelevate) | Owner portal session + password re-auth → short-lived elevated operator cookie. |
| [`DELETE /api/auth/elevate`](auth.md#delete-apiauthelevate) | End elevation by revoking the elevated operator session cookie. |
| [`POST /api/auth/bootstrap`](auth.md#post-apiauthbootstrap) | Operator-only bootstrap of the first Owner when none exist. |
| [`PUT /api/auth/password`](auth.md#put-apiauthpassword) | Set password for self (portal) or target user (operator/admin); revokes sessions. |
| [`POST /api/auth/password`](auth.md#post-apiauthpassword) | Local password login → portal session cookie, or a TOTP challenge when enabled. |
| [`POST /api/auth/impersonate`](auth.md#post-apiauthimpersonate) | Operator (or elevated) session starts impersonating a User. |
| [`DELETE /api/auth/impersonate`](auth.md#delete-apiauthimpersonate) | Clear impersonation on the current operator session. |
| [`GET /api/auth/branch-consent`](auth.md#get-apiauthbranch-consent) | `GET /api/auth/branch-consent?<authorize params>`: what the consent page shows. 400 `invalid_request` for a malformed request (the page then shows the problem and never redirects). |
| [`POST /api/auth/branch-consent`](auth.md#post-apiauthbranch-consent) | `POST /api/auth/branch-consent`: approve or deny. Answers `{redirect}`, the URL to send the browser to: the branch's callback with a code (or `error=access_denied`). Approving needs an active signed-in user, host enrollment on, and a connection the transport policy allows. |
| [`POST /api/auth/branch-consent/step-up`](auth.md#post-apiauthbranch-consentstep-up) | `POST /api/auth/branch-consent/step-up`: starts a passkey step-up for a clerk consent (an Owner with a passkey). Answers the WebAuthn request options with a `challenge_id` to send back in the consent's `step_up`. |
| [`POST /api/auth/logout`](auth.md#post-apiauthlogout) | Revokes hashed operator and portal sessions and clears both cookies. |
| [`GET /api/auth/sessions`](auth.md#get-apiauthsessions) | List sessions for the current principal only (not every operator session). |
| [`DELETE /api/auth/sessions/{id}`](auth.md#delete-apiauthsessionsid) | Revoke a session by id. Operators may revoke any operator session; portal users may only revoke their own. |
| [`PATCH /api/auth/profile`](auth.md#patch-apiauthprofile) | Updates the caller's display name and/or email. |
| [`PUT /api/auth/profile/avatar`](auth.md#put-apiauthprofileavatar) | Accepts a JPEG/PNG/WebP upload and stores it under `files_dir/avatars`. |
| [`DELETE /api/auth/profile/avatar`](auth.md#delete-apiauthprofileavatar) | Removes the caller's stored avatar, if any. |
| [`GET /api/auth/oidc/providers`](auth.md#get-apiauthoidcproviders) | Lists enabled IdPs for the login picker (id and display name only). |
| [`GET /api/auth/oidc/identities`](auth.md#get-apiauthoidcidentities) | Lists OIDC portal identities for the cookie-authenticated caller. |
| [`GET /api/auth/oidc/login`](auth.md#get-apiauthoidclogin) | Starts an OIDC login authorize redirect, honoring login throttle. |
| [`GET /api/auth/oidc/elevate`](auth.md#get-apiauthoidcelevate) | Starts an Owner re-auth elevate flow against a linked IdP (`prompt=login`). |
| [`GET /api/auth/oidc/callback`](auth.md#get-apiauthoidccallback) | GET callback (most IdPs); finishes the code exchange. |
| [`POST /api/auth/oidc/callback`](auth.md#post-apiauthoidccallback) | POST callback (Apple `form_post`); finishes the code exchange. |
| [`GET /api/auth/oidc/config`](auth.md#get-apiauthoidcconfig) | Operator GET: broker settings plus secret-presence flags (never plaintext). |
| [`PUT /api/auth/oidc/config`](auth.md#put-apiauthoidcconfig) | Operator PUT: writes next-generation sealed secrets, then publishes config.toml. |
| [`GET /api/auth/passkeys`](auth.md#get-apiauthpasskeys) | Lists this user's stored passkeys as `{ id, credential_id }` rows. |
| [`POST /api/auth/passkeys/register/begin`](auth.md#post-apiauthpasskeysregisterbegin) | Starts passkey registration (skips reauth only for first local-only setup). |
| [`POST /api/auth/passkeys/register/finish`](auth.md#post-apiauthpasskeysregisterfinish) | Completes registration, stores the passkey, and revokes elevated operator sessions. |
| [`DELETE /api/auth/passkeys/{id}`](auth.md#delete-apiauthpasskeysid) | Deletes one passkey after recent reauth; 404 when the id is not this user's. |
| [`POST /api/auth/passkeys/login/begin`](auth.md#post-apiauthpasskeysloginbegin) | Rate-limits and starts passkey login; records a failure on any error. |
| [`POST /api/auth/passkeys/login/finish`](auth.md#post-apiauthpasskeysloginfinish) | Verifies the assertion, updates the credential counter, and issues a portal session. |
| [`POST /api/auth/passkeys/elevate/begin`](auth.md#post-apiauthpasskeyselevatebegin) | Starts Owner elevation; 403 unless the signed-in user is an Owner with passkeys. |
| [`POST /api/auth/passkeys/elevate/finish`](auth.md#post-apiauthpasskeyselevatefinish) | Completes Owner elevation only when the assertion is user-verified. |
| [`POST /api/auth/totp/login`](auth.md#post-apiauthtotplogin) | Completes password+TOTP login and issues a portal session cookie. |
| [`POST /api/auth/totp/enroll/begin`](auth.md#post-apiauthtotpenrollbegin) | Starts TOTP enrollment: new pending secret, otpauth URL, and QR SVG. |
| [`POST /api/auth/totp/enroll/finish`](auth.md#post-apiauthtotpenrollfinish) | Confirms the pending secret with a valid code and sets `users.totp_enabled`. |
| [`GET /api/auth/totp`](auth.md#get-apiauthtotp) | GET `/api/auth/totp` — whether TOTP is confirmed for this user. |
| [`DELETE /api/auth/totp`](auth.md#delete-apiauthtotp) | Disables TOTP after recent reauth and deletes sealed secrets. |
| [`GET /api/auth/mfa-policy`](auth.md#get-apiauthmfa-policy) | Current MFA policy (`require_second_factor`). |
| [`PUT /api/auth/mfa-policy`](auth.md#put-apiauthmfa-policy) | Persists `daemon.auth.require_second_factor` to `config.toml`. |

### [Users](users.md)

Library users and their claim tickets.

| Route | Summary |
| --- | --- |
| [`GET /api/users`](users.md#get-apiusers) | List first-party users (operator or administrator provisioner). |
| [`POST /api/users`](users.md#post-apiusers) | Owner/admin or operator creates a user and optional invite magic link. |
| [`DELETE /api/users/{id}`](users.md#delete-apiusersid) | Delete a user and their personal data (wishlist / links / sessions); keep acquired books. |
| [`PATCH /api/users/{id}`](users.md#patch-apiusersid) | Patch role/status/display/login for a first-party user (provisioner). |
| [`POST /api/users/{id}/claim-ticket`](users.md#post-apiusersidclaim-ticket) | Mint a fresh local claim ticket for an active first-party user. |
| [`POST /api/users/{id}/reset-password`](users.md#post-apiusersidreset-password) | Invalidate a user's password, revoke sessions, and mint a claim ticket for reset. |
| [`GET /api/users/{id}/avatar`](users.md#get-apiusersidavatar) | Serves a stored avatar JPEG for any authenticated household member. |

### [Library](library.md)

Books in the library, scans and acquires.

| Route | Summary |
| --- | --- |
| [`POST /api/library/scan`](library.md#post-apilibraryscan) | Plans and enqueues a library scan; returns the accepted job ids. |
| [`POST /api/library/acquire`](library.md#post-apilibraryacquire) | Plans and enqueues acquire jobs; returns the accepted job ids. |
| [`GET /api/library/books`](library.md#get-apilibrarybooks) | Lists or searches library titles with optional account/status filters and pagination. |
| [`GET /api/library/books/{uuid}`](library.md#get-apilibrarybooksuuid) | Returns one title by UUID, or 404 when it is absent. |
| [`GET /api/library/books/{uuid}/cover`](library.md#get-apilibrarybooksuuidcover) | Serves the JPEG cover sidecar from the deployed local destination, or 404 when no local destination runs on this node. |

### [Discover](discover.md)

Recommendations, catalog search and title metadata.

| Route | Summary |
| --- | --- |
| [`POST /api/discover/sync-listening`](discover.md#post-apidiscoversync-listening) | Enqueues a listening-progress sync; 400 when no capable integrations are loaded. |
| [`GET /api/discover/recommendations`](discover.md#get-apidiscoverrecommendations) | Builds the Discover recommendation feed, skipping embed work on an empty library. |
| [`POST /api/discover/purchase-hints`](discover.md#post-apidiscoverpurchase-hints) | Resolves storefront price/availability for one title, forcing server-side preferred stores. |
| [`POST /api/discover/purchase-hints/batch`](discover.md#post-apidiscoverpurchase-hintsbatch) | Resolves up to 24 purchase-hint queries with caller-derived preferred stores. |
| [`GET /api/discover/search`](discover.md#get-apidiscoversearch) | Runs a multi-store catalog search with a 15s wall budget and empty-query short circuit. |
| [`POST /api/discover/title-meta`](discover.md#post-apidiscovertitle-meta) | Resolves Audnexus-style title metadata for one query. |
| [`POST /api/discover/title-meta/batch`](discover.md#post-apidiscovertitle-metabatch) | Resolves up to 24 title-metadata queries without failing the whole batch on one error. |
| [`POST /api/discover/title-reviews`](discover.md#post-apidiscovertitle-reviews) | Fetches Audnexus reviews for an ASIN; rejects an empty ASIN with 400. |

### [Wishlist](wishlist.md)

Titles users have asked for.

| Route | Summary |
| --- | --- |
| [`GET /api/wishlist`](wishlist.md#get-apiwishlist) | Lists open wishlist rows for the caller (portal identity or operator-owned). |
| [`POST /api/wishlist`](wishlist.md#post-apiwishlist) | Creates a wishlist / title-request row for the authenticated caller. |
| [`DELETE /api/wishlist/{uuid}`](wishlist.md#delete-apiwishlistuuid) | Cancels an open wishlist item the caller owns; others get 403. |
| [`GET /api/request-queue`](wishlist.md#get-apirequest-queue) | Ranks the shared open-request queue without warming ONNX embeddings. |

### [Jobs](jobs.md)

Queued and running control-plane jobs.

| Route | Summary |
| --- | --- |
| [`GET /api/jobs`](jobs.md#get-apijobs) | Returns durable scan/acquire/listen_sync jobs, newest first. |
| [`POST /api/jobs/{id}/cancel`](jobs.md#post-apijobsidcancel) | Cancel a pending job or request cooperative stop of a running job. |

### [Events](events.md)

Domain events and their deliveries.

| Route | Summary |
| --- | --- |
| [`GET /api/events`](events.md#get-apievents) | Returns recent domain-event outbox rows, newest first. |
| [`GET /api/events/deliveries`](events.md#get-apieventsdeliveries) | Returns event deliveries, optionally filtered by `state`. |
| [`GET /api/queues`](events.md#get-apiqueues) | Returns every queue the outbox or the live catalog knows about, with delivery counters and the consumers subscribed to it. |
| [`GET /api/queues/{name}`](events.md#get-apiqueuesname) | Returns one queue's counters, consumers, recent messages, and deliveries. |
| [`POST /api/events/deliveries/{id}/retry`](events.md#post-apieventsdeliveriesidretry) | Re-queue a dead-lettered delivery. |
| [`POST /api/events/deliveries/{id}/acknowledge`](events.md#post-apieventsdeliveriesidacknowledge) | Acknowledge a dead-lettered delivery (no further retry). |
| [`POST /api/events/deliveries/{id}/cancel`](events.md#post-apieventsdeliveriesidcancel) | Cancel a pending delivery or request cooperative stop of a running one. |
| [`POST /api/events/deliveries/{id}/resume`](events.md#post-apieventsdeliveriesidresume) | Resume a suspended delivery (wake now, next claim uses resume_pending). |

### [Live](live.md)

The web interface's live-updates socket.

| Route | Summary |
| --- | --- |
| [`GET /api/live`](live.md#get-apilive) | `GET /api/live`. |

### [Status](status.md)

Library and daemon snapshot.

| Route | Summary |
| --- | --- |
| [`GET /api/status`](status.md#get-apistatus) | Returns library counts, listen addresses, storage backends, and whether operator auth is on. |
| [`GET /api/cluster/status`](status.md#get-apiclusterstatus) | `GET /api/cluster/status`: cluster-wide health every node administrator may see (node counts, committed capacity, active jobs, delivery backlog), plus detail for the nodes the caller may see. |

### [Settings](settings.md)

Daemon settings and user preferences.

| Route | Summary |
| --- | --- |
| [`GET /api/settings`](settings.md#get-apisettings) | Returns file-backed and runtime-effective Settings plus per-plugin groups. |
| [`PATCH /api/settings`](settings.md#patch-apisettings) | Validates, writes, and reloads allowlisted Settings keys under `reload_lock`. |
| [`POST /api/database/migrate`](settings.md#post-apidatabasemigrate) | Copies library rows between database plugins and optionally switches `[database].plugin`. |
| [`GET /api/preferences`](settings.md#get-apipreferences) | Loads preferences for the caller with a 3s timeout, creating defaults when missing. |
| [`PATCH /api/preferences`](settings.md#patch-apipreferences) | Merges a preferences PATCH onto the caller’s stored row and writes it back. |

### [Plugins](plugins.md)

Plugin packages, instances, consent and deployment.

| Route | Summary |
| --- | --- |
| [`GET /api/config/plugin-instances`](plugins.md#get-apiconfigplugin-instances) | `GET /api/config/plugin-instances`. |
| [`POST /api/config/plugin-instances`](plugins.md#post-apiconfigplugin-instances) | `POST /api/config/plugin-instances`: a second (non-default) instance of a plugin, with an empty document so schema defaults apply. With `node_id`, the instance is local to that node. |
| [`DELETE /api/config/plugin-instances/{id}`](plugins.md#delete-apiconfigplugin-instancesid) | `DELETE /api/config/plugin-instances/{id}`: removes an instance that has no deployment left. |
| [`GET /api/config/plugin-instances/{id}/config`](plugins.md#get-apiconfigplugin-instancesidconfig) | `GET /api/config/plugin-instances/{id}/config`. |
| [`PUT /api/config/plugin-instances/{id}/config`](plugins.md#put-apiconfigplugin-instancesidconfig) | `PUT /api/config/plugin-instances/{id}/config`. |
| [`PUT /api/config/plugin-instances/{id}/deployments/{node_id}`](plugins.md#put-apiconfigplugin-instancesiddeploymentsnode_id) | `PUT /api/config/plugin-instances/{id}/deployments/{node_id}`: the Deployed switch. Creates the row when the node has none. |
| [`PUT /api/config/plugin-instances/{id}/placement`](plugins.md#put-apiconfigplugin-instancesidplacement) | `PUT /api/config/plugin-instances/{id}/placement`: hand the instance's deployments to the scheduler with a replica count and constraints. |
| [`DELETE /api/config/plugin-instances/{id}/placement`](plugins.md#delete-apiconfigplugin-instancesidplacement) | `DELETE /api/config/plugin-instances/{id}/placement`: return the instance to hand-managed deployments; its deployments stay as they are. |
| [`GET /api/config/plugins`](plugins.md#get-apiconfigplugins) | `GET /api/config/plugins`: every installed plugin. |
| [`POST /api/config/plugins`](plugins.md#post-apiconfigplugins) | `POST /api/config/plugins`: install (and maybe activate) a version. |
| [`GET /api/config/plugins/{key}`](plugins.md#get-apiconfigpluginskey) | `GET /api/config/plugins/{key}`: one installed plugin with its pinned settings schema. |
| [`DELETE /api/config/plugins/{key}`](plugins.md#delete-apiconfigpluginskey) | `DELETE /api/config/plugins/{key}`: uninstall the plugin (active version cleared, placements dropped, every deployment `absent`). |
| [`PUT /api/config/plugins/{key}/active`](plugins.md#put-apiconfigpluginskeyactive) | `PUT /api/config/plugins/{key}/active`: activate, update, or roll back. |
| [`POST /api/config/plugins/{key}/versions/{version}/approve`](plugins.md#post-apiconfigpluginskeyversionsversionapprove) | `POST /api/config/plugins/{key}/versions/{version}/approve`: record the version's requested consent as approved for every node. |
| [`GET /api/plugins/{id}/consent`](plugins.md#get-apipluginsidconsent) | Loads the consent dialog for one plugin, including coverage against the stored grant. |
| [`POST /api/plugins/{id}/consent`](plugins.md#post-apipluginsidconsent) | Persists an approved plugin grant after validating operator overrides. |
| [`GET /api/plugins/{kind}/{id}/logo`](plugins.md#get-apipluginskindidlogo) | Serve an embedded `plugin.toml` `logo` file from under the plugin install root. |

### [Configuration](configuration.md)

Live configuration domains and reloads.

| Route | Summary |
| --- | --- |
| [`POST /api/config/reload`](configuration.md#post-apiconfigreload) | Reloads `config.toml` and republishes daemon runtime without a process restart. |
| [`GET /api/config/domains/core.events`](configuration.md#get-apiconfigdomainscoreevents) | `GET /api/config/domains/core.events`. |
| [`PUT /api/config/domains/core.events`](configuration.md#put-apiconfigdomainscoreevents) | `PUT /api/config/domains/core.events`. |
| [`GET /api/config/domains/core.output`](configuration.md#get-apiconfigdomainscoreoutput) | `GET /api/config/domains/core.output`. |
| [`PUT /api/config/domains/core.output`](configuration.md#put-apiconfigdomainscoreoutput) | `PUT /api/config/domains/core.output`. |

### [Cluster](cluster.md)

Nodes, clerks, branches, joining a library and host enrollment.

| Route | Summary |
| --- | --- |
| [`PUT /api/config/nodes/{node_id}/labels`](cluster.md#put-apiconfignodesnode_idlabels) | `PUT /api/config/nodes/{node_id}/labels`: replace a node's placement labels. The scheduler re-plans on the wakeup this publishes. |
| [`PUT /api/config/nodes/{node_id}/owner`](cluster.md#put-apiconfignodesnode_idowner) | `PUT /api/config/nodes/{node_id}/owner`: assign or clear a node's owner. The owner may then edit that node's runtime settings without operator authority. |
| [`GET /api/config/cluster-consents`](cluster.md#get-apiconfigcluster-consents) | `GET /api/config/cluster-consents`. |
| [`POST /api/config/clusters/{cluster_id}/scan`](cluster.md#post-apiconfigclusterscluster_idscan) | `POST /api/config/clusters/{cluster_id}/scan`: asks the owner cluster to queue a scan for this node's owner (H-5). The owner's planner decides what that user may scan; the answer lists the queued job ids. |
| [`POST /api/config/clusters/{cluster_id}/jobs`](cluster.md#post-apiconfigclusterscluster_idjobs) | `POST /api/config/clusters/{cluster_id}/jobs`: the owner's view of jobs this node may see (those it would be trusted to claim). |
| [`POST /api/config/cluster-consents/approve`](cluster.md#post-apiconfigcluster-consentsapprove) | `POST /api/config/cluster-consents/approve`. |
| [`POST /api/config/cluster-consents/revoke`](cluster.md#post-apiconfigcluster-consentsrevoke) | `POST /api/config/cluster-consents/revoke`. |
| [`GET /api/config/join`](cluster.md#get-apiconfigjoin) | `GET /api/config/join`. |
| [`POST /api/config/join/inspect`](cluster.md#post-apiconfigjoininspect) | `POST /api/config/join/inspect`. |
| [`POST /api/config/join/start`](cluster.md#post-apiconfigjoinstart) | `POST /api/config/join/start`. |
| [`POST /api/config/join/callback`](cluster.md#post-apiconfigjoincallback) | `POST /api/config/join/callback`: redeems the library's code with the sign-in's sealed verifier and keys. The pending sign-in is kept when the redeem fails for a reason a retry can fix (answered as 502 `join_retry`; the page offers Try again with the same code), and ended on success, refusal, or anything else. |
| [`POST /api/config/join/invite`](cluster.md#post-apiconfigjoininvite) | `POST /api/config/join/invite`: join with a pasted branch invite. The link is redeemed once and dropped; it is never logged or echoed. A library slower than [`REDEEM_TIMEOUT`] is answered 504 `join_pending` while the join carries on and starts its client when it lands. |
| [`POST /api/config/join/leave`](cluster.md#post-apiconfigjoinleave) | `POST /api/config/join/leave`: Leave library. The leave runs on its own task, so a request cut short (the client gone, or the API deadline) still records it once the library has been told. |
| [`GET /api/config/nodes`](cluster.md#get-apiconfignodes) | `GET /api/config/nodes`: every node for an operator; the caller's own nodes for a node owner. |
| [`DELETE /api/config/nodes/{node_id}`](cluster.md#delete-apiconfignodesnode_id) | `DELETE /api/config/nodes/{node_id}`: remove an owner node (a clerk, or any node sharing this database) that is no longer running from the library's node list. The operator or an Owner may remove any; another user only one they own. A running node, this node, or a branch is refused (a branch is revoked instead). |
| [`GET /api/config/clerks`](cluster.md#get-apiconfigclerks) | `GET /api/config/clerks`: the owner nodes (this one and its clerks) for Settings → Clerks, shaped like `GET /api/config/nodes`. The operator or an Owner sees every one, as `DELETE /api/config/nodes/{node_id}` lets them remove any; another user only the ones they own, which the nodes listing already shows them. Branches are not listed. |
| [`PUT /api/config/nodes/{node_id}/runtime`](cluster.md#put-apiconfignodesnode_idruntime) | `PUT /api/config/nodes/{node_id}/runtime`: edit a node's label, budgets, or drain switch. Applies at that node's next heartbeat. A node owner may edit only the nodes it owns. |
| [`GET /api/config/node-invites`](cluster.md#get-apiconfignode-invites) | `GET /api/config/node-invites`: every link for the operator or an Owner; the caller's own links for anyone else, and with `?mine=true`. |
| [`POST /api/config/node-invites`](cluster.md#post-apiconfignode-invites) | `POST /api/config/node-invites`: mint an enrollment link. Any user may mint one for themselves while host enrollment is on; the operator or an Owner may mint for any active user. |
| [`DELETE /api/config/node-invites/{id}`](cluster.md#delete-apiconfignode-invitesid) | `DELETE /api/config/node-invites/{id}`: revoke an unredeemed link. |
| [`GET /api/config/branches`](cluster.md#get-apiconfigbranches) | `GET /api/config/branches`: this library's join settings: whether new branches may join, the transport mode, and the SPKI fingerprint a joining operator confirms in `self_signed` mode. |
| [`PUT /api/config/branches`](cluster.md#put-apiconfigbranches) | `PUT /api/config/branches`: turn "Let new branches join" on or off. |
| [`POST /api/config/branches/titles/remove`](cluster.md#post-apiconfigbranchestitlesremove) | `POST /api/config/branches/titles/remove`: removes the entries of titles from a branch that stopped sharing or left. Delivered files stay. |
| [`GET /api/config/join/sharing`](cluster.md#get-apiconfigjoinsharing) | `GET /api/config/join/sharing`: Settings → Library for the operator or an Owner: every library this install is a branch of, with its two sharing settings (`{clusters: [...]}`). |
| [`POST /api/config/join/sharing`](cluster.md#post-apiconfigjoinsharing) | `POST /api/config/join/sharing`: Settings → Library's two sharing settings for one library this install is a branch of. The operator or an Owner (verified from the stored role); anyone else gets 403. |
| [`DELETE /api/config/node-credentials/{node_id}`](cluster.md#delete-apiconfignode-credentialsnode_id) | `DELETE /api/config/node-credentials/{node_id}`: revoke an enrolled host's credential so it can no longer connect. |
| [`GET /api/config/host-enrollment`](cluster.md#get-apiconfighost-enrollment) | `GET /api/config/host-enrollment`: whether host nodes may enroll. |
| [`PUT /api/config/host-enrollment`](cluster.md#put-apiconfighost-enrollment) | `PUT /api/config/host-enrollment`: turn host enrollment on or off. The operator may always; a user only while the stored role is an active Owner. |

### [Accounts](accounts.md)

Store accounts a library user connects (the Accounts page).

| Route | Summary |
| --- | --- |
| [`POST /api/portal/redeem`](accounts.md#post-apiportalredeem) | Redeems a claim ticket, optionally setting a first password, and sets the session cookie. |
| [`POST /api/portal/login/integration`](accounts.md#post-apiportalloginintegration) | Authenticates against an enabled integration and mints a portal session. |
| [`POST /api/portal/logout`](accounts.md#post-apiportallogout) | Revokes the portal session row and clears the session cookie. |
| [`GET /api/portal/me`](accounts.md#get-apiportalme) | Returns the signed-in portal identity, or 401 when the cookie is missing or expired. |
| [`GET /api/portal/sources`](accounts.md#get-apiportalsources) | Lists the deployed content sources (one per plugin, through its default instance) with portal auth mode, config options, and brand colors. |
| [`POST /api/portal/sources/{id}/login`](accounts.md#post-apiportalsourcesidlogin) | Connects a password storefront, upserts the account, and links it to the signed-in identity. |
| [`POST /api/portal/sources/{id}/oauth/start`](accounts.md#post-apiportalsourcesidoauthstart) | Starts interactive OAuth for an OAuth storefront and returns the browser login URL. |
| [`POST /api/portal/libro/login`](accounts.md#post-apiportallibrologin) (deprecated) | Legacy `POST /libro/login` alias that forwards to the Libro password-login handler. |
| [`POST /api/portal/audible/start`](accounts.md#post-apiportalaudiblestart) (deprecated) | Legacy `POST /audible/start` alias that forwards to Audible OAuth start. |
| [`GET /api/portal/connections`](accounts.md#get-apiportalconnections) | Lists store accounts linked to the signed-in identity, including disabled sources. |
| [`POST /api/portal/sources/{source}/connections/{account_id}/revoke`](accounts.md#post-apiportalsourcessourceconnectionsaccount_idrevoke) | Unlinks a store account from this identity and deletes secrets when no other identity remains. |

### [Node protocol](node-protocol.md)

Node-to-node enrollment, socket, artifacts and uploads; each route checks its own credential.

| Route | Summary |
| --- | --- |
| [`POST /api/node/v1/enroll`](node-protocol.md#post-apinodev1enroll) | `POST /api/node/v1/enroll`: redeem an enrollment link. Answers `{cluster_id, node_id}`; 403 `transport_insecure` when the connection fails the transport policy, 403 `enrollment_refused` for any link that does not redeem. |
| [`GET /api/node/v1/info`](node-protocol.md#get-apinodev1info) | `GET /api/node/v1/info`. |
| [`POST /api/node/v1/enroll/oauth`](node-protocol.md#post-apinodev1enrolloauth) | `POST /api/node/v1/enroll/oauth`: redeem an approved branch sign-in. Answers `{cluster_id, node_id}`; 403 `transport_insecure` for a refused connection, 403 `enrollment_refused` for a code that does not redeem (unknown, used, expired, another client's, a wrong verifier or redirect URI, enrollment off, or the user no longer active). |
| [`GET /api/node/v1/connect`](node-protocol.md#get-apinodev1connect) | `GET /api/node/v1/connect`: upgrade to the host-node socket after the transport check. 403 `transport_insecure` for a refused connection, 400 `subprotocol_required` without [`SUBPROTOCOL`]. |
| [`GET /api/node/v1/artifacts/{sha256}`](node-protocol.md#get-apinodev1artifactssha256) | `GET /api/node/v1/artifacts/{sha256}`. |
| [`PUT /api/node/v1/uploads/{upload_id}/files/{index}`](node-protocol.md#put-apinodev1uploadsupload_idfilesindex) | `PUT /api/node/v1/uploads/{upload_id}/files/{index}`. |
| [`POST /api/node/v1/uploads/{upload_id}/commit`](node-protocol.md#post-apinodev1uploadsupload_idcommit) | `POST /api/node/v1/uploads/{upload_id}/commit`. |

### [OIDC provider](oidc-provider.md)

Bookclerk as an OpenID Connect provider for other apps.

| Route | Summary |
| --- | --- |
| [`GET /.well-known/openid-configuration`](oidc-provider.md#get-well-knownopenid-configuration) | Serves the OIDC discovery document for registered clients. |
| [`GET /oidc/authorize`](oidc-provider.md#get-oidcauthorize) | Starts the authorization-code flow; unauthenticated users are sent to SPA login. |
| [`POST /oidc/authorize`](oidc-provider.md#post-oidcauthorize) | Completes consent: deny redirects with `access_denied`, grant issues a code. |
| [`POST /oidc/token`](oidc-provider.md#post-oidctoken) | Dispatches token grants; unknown `grant_type` values return 400. |
| [`GET /oidc/userinfo`](oidc-provider.md#get-oidcuserinfo) | Returns `sub` / `name` / `preferred_username` for a valid access JWT. |
| [`POST /oidc/revoke`](oidc-provider.md#post-oidcrevoke) | Best-effort refresh-token revoke; always returns 200 per RFC 7009. |
| [`GET /api/auth/oidc/clients`](oidc-provider.md#get-apiauthoidcclients) | Owner/operator list of Bookclerk-as-IdP clients (secrets never included). |
| [`POST /api/auth/oidc/clients`](oidc-provider.md#post-apiauthoidcclients) | Create a client; confidential clients return the generated secret once. |
| [`PUT /api/auth/oidc/clients/{client_id}`](oidc-provider.md#put-apiauthoidcclientsclient_id) | Update redirects, name, scopes, and public/confidential mode (no secret echo unless rotating). |
| [`DELETE /api/auth/oidc/clients/{client_id}`](oidc-provider.md#delete-apiauthoidcclientsclient_id) | Delete a registered client. |
| [`POST /api/auth/oidc/clients/{client_id}/rotate-secret`](oidc-provider.md#post-apiauthoidcclientsclient_idrotate-secret) | Mint a new secret for a confidential client; plaintext is returned once. |

### [Legacy](legacy.md)

Old unprefixed paths; use the `/api/…` route named in each.

| Route | Summary |
| --- | --- |
| [`GET /status`](legacy.md#get-status) (deprecated) | Deprecated alias of `GET /api/status`. |
| [`POST /scan`](legacy.md#post-scan) (deprecated) | Deprecated alias of `POST /api/library/scan`. |
| [`POST /acquire`](legacy.md#post-acquire) (deprecated) | Deprecated alias of `POST /api/library/acquire`. |
| [`GET /jobs`](legacy.md#get-jobs) (deprecated) | Deprecated alias of `GET /api/jobs`. |
| [`POST /jobs/{id}/cancel`](legacy.md#post-jobsidcancel) (deprecated) | Deprecated alias of `POST /api/jobs/{id}/cancel`. |
| [`POST /integrations/{id}/scan`](legacy.md#post-integrationsidscan) | Enqueues a remote integration library scan; 404 if missing, 400 if unsupported. |

## Schemas

Request and response bodies are described in [Schemas](schemas.md).
