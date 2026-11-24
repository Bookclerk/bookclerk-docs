# Web API

The HTTP API of `bookclerkd`, the Bookclerk daemon: the web interface, the CLI and other nodes use it. Request bodies are JSON (`Content-Type: application/json`; send `{}` for defaults) unless a route says otherwise. Error responses are JSON objects with an `error` slug (`ApiError`); bare statuses get `{"error", "message", "status"}` (`ErrorJson`).

The daemon serves this reference as an OpenAPI 3.1 document, *Bookclerk daemon API*, at `GET /api/openapi.json` (API version `0.1.0`). Load it into an OpenAPI viewer such as Scalar or Swagger UI, or a client generator. Tools that only read OpenAPI 3.0, such as Cloudflare API Shield's schema validation, can use the same API restated as OpenAPI 3.0.3 at `GET /api/openapi-3.0.json`.

## Quick start

The daemon listens on `http://127.0.0.1:8787` unless `daemon.listen` says otherwise. This request calls `GET /api/status` with the `operatorToken` credential; [Authentication](#authentication) says where to get it.

```console
$ curl 'http://127.0.0.1:8787/api/status' \
    -H 'Authorization: Bearer <operatorToken>'
```

[`GET /api/status`](status.md#get-apistatus) describes the response.

## Authentication

Each route lists the credentials it accepts and who may call it. The credentials are:

| Name | Kind | Description |
| --- | --- | --- |
| `clerkToken` | header `Authorization` | A library card (`POST /api/clerk/v1/token`), as `Authorization: Bookclerk-Clerk <token>`. Accepted only on routes any library session may use, over TLS except on loopback. |
| `copyToken` | header `Authorization` | The token of a copy grant, as `Authorization: Bookclerk-Copy <token>`. |
| `nodeSignature` | header `Authorization` | A node's signed request, as `Authorization: Bookclerk-Node <node_id> <issued_at> <signature>` with the node's identity key. |
| `oidcAccessToken` | HTTP bearer | An access token from this daemon's OIDC token endpoint (`POST /oidc/token`), as `Authorization: Bearer <token>`. |
| `operatorSession` | cookie `bookclerk_operator_session` | The operator's browser session, set by `POST /api/auth/login`. |
| `operatorToken` | HTTP bearer | The operator token (`bookclerk daemon token`), as `Authorization: Bearer <token>`. |
| `playerToken` | HTTP bearer | A player card's access token (`POST /player/login`), as `Authorization: Bearer <token>` or, in audio and cover URLs, the `token` query parameter. Accepted only under `/player`. |
| `portalSession` | cookie `bookclerk_portal_session` | A library user's browser session, set by a sign-in route. Cookie-authenticated writes must send an `Origin` (or `Referer`) matching the daemon's public origin or its host. |
| `uploadToken` | header `Authorization` | The token of an upload grant, as `Authorization: Bookclerk-Upload <token>`. |

## Routes

### [Health](health.md)

Liveness.

| Route | Summary |
| --- | --- |
| [`GET /health`](health.md#get-health) | Serves `GET /health` with a static `ok` payload for liveness probes. |

### [Webhooks](webhooks.md)

Inbound webhooks for plugins, outside operator auth: the URL token is the credential.

| Route | Summary |
| --- | --- |
| [`POST /hooks/{token}/{name}`](webhooks.md#post-hookstokenname) | Stores one inbound webhook request for a plugin instance. |

### [Spec](spec.md)

This document.

| Route | Summary |
| --- | --- |
| [`GET /api/openapi.json`](spec.md#get-apiopenapijson) | Serves the OpenAPI document. |
| [`GET /api/openapi-3.0.json`](spec.md#get-apiopenapi-30json) | Serves the OpenAPI document restated as OpenAPI 3.0.3. |

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
| [`POST /api/auth/verify-email`](auth.md#post-apiauthverify-email) | Uses a verification link. The link is the proof, so no sign-in is needed; it works once, for 24 hours, and only while the patron still has the address it was sent to. |
| [`POST /api/auth/impersonate`](auth.md#post-apiauthimpersonate) | Operator (or elevated) session starts impersonating a User. |
| [`DELETE /api/auth/impersonate`](auth.md#delete-apiauthimpersonate) | Clear impersonation on the current operator session. |
| [`POST /api/auth/profile/email/verification`](auth.md#post-apiauthprofileemailverification) | Sends the caller a new link to verify their email address, voiding the last one. A patron may ask once every five minutes. |
| [`GET /api/auth/branch-consent`](auth.md#get-apiauthbranch-consent) | `GET /api/auth/branch-consent?<authorize params>`: what the consent page shows. 400 `invalid_request` for a malformed request (the page then shows the problem and never redirects). |
| [`POST /api/auth/branch-consent`](auth.md#post-apiauthbranch-consent) | `POST /api/auth/branch-consent`: approve or deny. Answers `{redirect}`, the URL to send the browser to: the branch's callback with a code (or `error=access_denied`). Approving needs an active signed-in user, host enrollment on, and a connection the transport policy allows. |
| [`POST /api/auth/branch-consent/step-up`](auth.md#post-apiauthbranch-consentstep-up) | `POST /api/auth/branch-consent/step-up`: starts a passkey step-up for a shared branch consent (an Owner with a passkey). Answers the WebAuthn request options with a `challenge_id` to send back in the consent's `step_up`. |
| [`GET /api/auth/clerk-consent`](auth.md#get-apiauthclerk-consent) | `GET /api/auth/clerk-consent?<authorize params>`: what the consent page shows. 400 `invalid_request` for a malformed request (the page then shows the problem and never redirects). |
| [`POST /api/auth/clerk-consent`](auth.md#post-apiauthclerk-consent) | `POST /api/auth/clerk-consent`: approve or deny. Answers `{redirect}`, the URL to send the browser to: the clerk's callback with a code (or `error=access_denied`). |
| [`POST /api/auth/logout`](auth.md#post-apiauthlogout) | Revokes hashed operator and portal sessions and clears both cookies. |
| [`GET /api/auth/sessions`](auth.md#get-apiauthsessions) | List sessions for the current principal only (not every operator session). |
| [`DELETE /api/auth/sessions/{id}`](auth.md#delete-apiauthsessionsid) | Revoke a session by id. Operators may revoke any operator session; portal users may only revoke their own. |
| [`PATCH /api/auth/profile`](auth.md#patch-apiauthprofile) | Updates the caller's display name and/or email. |
| [`GET /api/auth/profile/catalog-listing`](auth.md#get-apiauthprofilecatalog-listing) | Reads whether the caller's books are listed to the libraries this install is a branch of. |
| [`PUT /api/auth/profile/catalog-listing`](auth.md#put-apiauthprofilecatalog-listing) | Sets whether the caller's books are listed to the libraries this install is a branch of. |
| [`PUT /api/auth/profile/avatar`](auth.md#put-apiauthprofileavatar) | Accepts a JPEG/PNG/WebP upload and stores it under `files_dir/avatars`. |
| [`DELETE /api/auth/profile/avatar`](auth.md#delete-apiauthprofileavatar) | Removes the caller's stored avatar, if any. |
| [`GET /api/auth/oidc/providers`](auth.md#get-apiauthoidcproviders) | Lists enabled IdPs for the login picker (id and display name only). |
| [`GET /api/auth/oidc/identities`](auth.md#get-apiauthoidcidentities) | Lists OIDC portal identities for the cookie-authenticated caller. |
| [`GET /api/auth/oidc/login`](auth.md#get-apiauthoidclogin) | Starts an OIDC login authorize redirect, honoring login throttle. |
| [`GET /api/auth/oidc/elevate`](auth.md#get-apiauthoidcelevate) | Starts an Owner re-auth elevate flow against a linked IdP (`prompt=login`). |
| [`GET /api/auth/oidc/callback`](auth.md#get-apiauthoidccallback) | GET callback (most IdPs); finishes the code exchange. |
| [`POST /api/auth/oidc/callback`](auth.md#post-apiauthoidccallback) | POST callback (Apple `form_post`); finishes the code exchange. |
| [`GET /api/auth/oidc/config`](auth.md#get-apiauthoidcconfig) | Operator GET: broker settings plus secret-presence flags (never plaintext). |
| [`PUT /api/auth/oidc/config`](auth.md#put-apiauthoidcconfig) | Operator-only PUT: writes next-generation sealed secrets, then publishes config.toml. |
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
| [`GET /api/auth/player-passwords`](auth.md#get-apiauthplayer-passwords) | Lists the caller's player passwords. |
| [`POST /api/auth/player-passwords`](auth.md#post-apiauthplayer-passwords) | Makes a player password for the caller and shows it once. |
| [`DELETE /api/auth/player-passwords/{id}`](auth.md#delete-apiauthplayer-passwordsid) | Removes one of the caller's player passwords and signs out every app it signed in. |
| [`GET /api/auth/player-consent`](auth.md#get-apiauthplayer-consent) | What the consent page shows. |
| [`POST /api/auth/player-consent`](auth.md#post-apiauthplayer-consent) | Approves or declines an app's browser sign-in. |

### [Users](users.md)

Library users and their claim tickets.

| Route | Summary |
| --- | --- |
| [`GET /api/users`](users.md#get-apiusers) | List first-party users (operator or administrator provisioner). |
| [`POST /api/users`](users.md#post-apiusers) | Owner/admin or operator creates a user and optional invite magic link. |
| [`DELETE /api/users/{id}`](users.md#delete-apiusersid) | Delete a user and their personal data (wishlist / links / sessions); keep acquired books. |
| [`PATCH /api/users/{id}`](users.md#patch-apiusersid) | Patch role/status/display/login/email for a first-party user (provisioner). |
| [`GET /api/users/default-scope`](users.md#get-apiusersdefault-scope) | Reads the library's default scope, which patrons without their own follow. |
| [`PUT /api/users/default-scope`](users.md#put-apiusersdefault-scope) | Replaces the library's default scope, if it is still at the `expected_version` the caller read; otherwise nothing is written and the answer is 409. Every patron without their own scope follows the change at once. |
| [`GET /api/users/{id}/scope`](users.md#get-apiusersidscope) | Reads which part of the library a patron sees. |
| [`PUT /api/users/{id}/scope`](users.md#put-apiusersidscope) | Gives a patron their own scope, or returns them to the library's default with `"scope": null`, if their scope is still at the `expected_version` the caller read; otherwise nothing is written and the answer is 409, so two Administrators editing one patron never undo each other unseen. |
| [`GET /api/users/default-controls`](users.md#get-apiusersdefault-controls) | Reads the library's default member controls, which every Member follows unless a control is set for them alone. |
| [`PATCH /api/users/default-controls`](users.md#patch-apiusersdefault-controls) | Changes the library's default for the controls named in `changes`; controls left out keep their stored value. Every Member follows the change at once, except in the controls set for them alone; Administrators do not follow the default. |
| [`GET /api/users/{id}/controls`](users.md#get-apiusersidcontrols) | Reads what a patron may do: the library's default, their own overrides, and what applies. |
| [`PATCH /api/users/{id}/controls`](users.md#patch-apiusersidcontrols) | Changes the controls named in `changes` for one patron: `on` or `off` for them alone, or `default` to follow the library's default again (for an Administrator: allowed). Controls left out keep what they had, so two Administrators changing different controls keep both changes. |
| [`GET /api/users/default-admin-controls`](users.md#get-apiusersdefault-admin-controls) | Reads the library's default Administrator switches. |
| [`PATCH /api/users/default-admin-controls`](users.md#patch-apiusersdefault-admin-controls) | Changes the named default Administrator switches; the others keep their stored value. Every Administrator follows the change at once, except in the switches set for them alone. Only an Owner or the operator may. |
| [`GET /api/users/{id}/admin-controls`](users.md#get-apiusersidadmin-controls) | Reads what an Administrator may do: the library's default, their own overrides, and what applies. An Administrator may read their own. |
| [`PATCH /api/users/{id}/admin-controls`](users.md#patch-apiusersidadmin-controls) | Changes the named switches for one Administrator; every switch it does not name keeps what it had. Only an Owner or the operator may, and switches stored for someone who is not an Administrator apply only if their role changes to Administrator. |
| [`POST /api/users/{id}/claim-ticket`](users.md#post-apiusersidclaim-ticket) | Mint a fresh local claim ticket for an active first-party user. |
| [`POST /api/users/{id}/invite-email`](users.md#post-apiusersidinvite-email) | Mails a user a new invite through account mail. The invite's ticket is minted when the mail is sent and voids the user's other unredeemed tickets, so only the newest invite works. |
| [`POST /api/users/{id}/reset-password`](users.md#post-apiusersidreset-password) | Invalidate a user's password, revoke sessions, and mint a claim ticket for reset. |
| [`GET /api/avatars/gravatar/{hash}`](users.md#get-apiavatarsgravatarhash) | Serves a user's Gravatar as a 128 px JPEG through the cover service. |
| [`GET /api/users/{id}/avatar`](users.md#get-apiusersidavatar) | Serves a stored avatar JPEG for any authenticated household member. |

### [Library](library.md)

Books in the library, scans and acquires.

| Route | Summary |
| --- | --- |
| [`POST /api/library/convert`](library.md#post-apilibraryconvert) | Queues a convert of stored books into another copy; returns the accepted job ids. |
| [`DELETE /api/config/nodes/{node_id}`](library.md#delete-apiconfignodesnode_id) | `DELETE /api/config/nodes/{node_id}`: remove a shared node (a shared branch, or any node sharing this database) that is no longer running from the library's node list. Only the operator (or an elevated Owner) may: it deletes the node and its rows, which is hard to undo. A running node, this node, or a branch is refused (a branch is revoked instead). |
| [`PUT /api/config/nodes/{node_id}/labels`](library.md#put-apiconfignodesnode_idlabels) | `PUT /api/config/nodes/{node_id}/labels`: replace a node's placement labels. The scheduler re-plans on the wakeup this publishes. |
| [`PUT /api/config/nodes/{node_id}/owner`](library.md#put-apiconfignodesnode_idowner) | `PUT /api/config/nodes/{node_id}/owner`: assign or clear a node's owner. The owner may then edit that node's runtime settings without operator authority. |
| [`GET /api/config/library-consents`](library.md#get-apiconfiglibrary-consents) | `GET /api/config/library-consents`. |
| [`POST /api/config/libraries/{library_id}/scan`](library.md#post-apiconfiglibrarieslibrary_idscan) | `POST /api/config/libraries/{library_id}/scan`: asks the library to queue a scan for this node's owner (H-5). The owner's planner decides what that user may scan; the answer lists the queued job ids. |
| [`POST /api/config/libraries/{library_id}/jobs`](library.md#post-apiconfiglibrarieslibrary_idjobs) | `POST /api/config/libraries/{library_id}/jobs`: the owner's view of jobs this node may see (those it would be trusted to claim). |
| [`POST /api/config/library-consents/approve`](library.md#post-apiconfiglibrary-consentsapprove) | `POST /api/config/library-consents/approve`. |
| [`POST /api/config/library-consents/revoke`](library.md#post-apiconfiglibrary-consentsrevoke) | `POST /api/config/library-consents/revoke`. |
| [`GET /api/config/join`](library.md#get-apiconfigjoin) | `GET /api/config/join`. |
| [`POST /api/config/join/inspect`](library.md#post-apiconfigjoininspect) | `POST /api/config/join/inspect`. |
| [`POST /api/config/join/start`](library.md#post-apiconfigjoinstart) | `POST /api/config/join/start`. |
| [`POST /api/config/join/callback`](library.md#post-apiconfigjoincallback) | `POST /api/config/join/callback`: redeems the library's code with the sign-in's sealed verifier and keys. The pending sign-in is kept when the redeem fails for a reason a retry can fix (answered as 502 `join_retry`; the page offers Try again with the same code), and ended on success, refusal, or anything else. |
| [`POST /api/config/join/invite`](library.md#post-apiconfigjoininvite) | `POST /api/config/join/invite`: join with a pasted branch invite. The link is redeemed once and dropped; it is never logged or echoed. A library slower than [`REDEEM_TIMEOUT`] is answered 504 `join_pending` while the join carries on and starts its client when it lands. |
| [`POST /api/config/join/leave`](library.md#post-apiconfigjoinleave) | `POST /api/config/join/leave`: Leave library. The leave runs on its own task, so a request cut short (the client gone, or the API deadline) still records it once the library has been told. |
| [`GET /api/config/nodes`](library.md#get-apiconfignodes) | `GET /api/config/nodes`: every node for an operator; the caller's own nodes for a node owner. |
| [`GET /api/config/shared-branches`](library.md#get-apiconfigshared-branches) | `GET /api/config/shared-branches`: the shared nodes (this one and its shared branches) for Settings → Shared branches, shaped like `GET /api/config/nodes`. The operator or an Owner sees every one (only the operator may remove one, with `DELETE /api/config/nodes/{node_id}`); another user only the ones they own, which the nodes listing already shows them. Branches are not listed. |
| [`PUT /api/config/nodes/{node_id}/runtime`](library.md#put-apiconfignodesnode_idruntime) | `PUT /api/config/nodes/{node_id}/runtime`: edit a node's label, budgets, or drain switch. Applies at that node's next heartbeat. A node owner may edit only the nodes it owns. |
| [`POST /api/library/scan`](library.md#post-apilibraryscan) | Plans and enqueues a library scan; returns the accepted job ids. |
| [`POST /api/library/acquire`](library.md#post-apilibraryacquire) | Plans and enqueues acquire jobs; returns the accepted job ids. |
| [`PUT /api/library/listings/{id}/pin`](library.md#put-apilibrarylistingsidpin) | Pins a store listing to the edition it sells. |
| [`DELETE /api/library/listings/{id}/pin`](library.md#delete-apilibrarylistingsidpin) | Removes a listing's pin. |
| [`GET /api/library/conflicts`](library.md#get-apilibraryconflicts) | Pages the conflicts the resolver could not decide, for an Administrator to settle with a pin. |
| [`GET /api/library/editions/{id}/chapters/claims`](library.md#get-apilibraryeditionsidchaptersclaims) | Lists every source's chapter claim for an edition, with its chapters. |
| [`POST /api/library/editions/{id}/chapters/realign`](library.md#post-apilibraryeditionsidchaptersrealign) | Queues the alignment of a metadata provider's chapters to the edition's stored copy. |
| [`GET /api/library/editions/{id}/chapters`](library.md#get-apilibraryeditionsidchapters) | Reads the chapters an edition plays. |
| [`PUT /api/library/editions/{id}/chapters`](library.md#put-apilibraryeditionsidchapters) | Sets an edition's chapters by hand. |
| [`DELETE /api/library/editions/{id}/chapters`](library.md#delete-apilibraryeditionsidchapters) | Removes an Administrator's chapter edit, so the edition plays the best remaining claim again. |
| [`PUT /api/library/editions/{id}/chapters/pin`](library.md#put-apilibraryeditionsidchapterspin) | Pins one claim, so the edition plays it whatever the others say. |
| [`DELETE /api/library/editions/{id}/chapters/pin`](library.md#delete-apilibraryeditionsidchapterspin) | Removes the pin on an edition's chapters; the resolver chooses again. |
| [`POST /api/library/works/{id}/split`](library.md#post-apilibraryworksidsplit) | Splits a work an automatic merge joined wrongly. |
| [`GET /api/library/works/{id}/tags`](library.md#get-apilibraryworksidtags) | Reads a work's library tags. |
| [`PUT /api/library/works/{id}/tags`](library.md#put-apilibraryworksidtags) | Replaces the tags an Administrator set on a work. |
| [`GET /api/library/collections`](library.md#get-apilibrarycollections) | Reads the library's collections. |
| [`PUT /api/library/collections`](library.md#put-apilibrarycollections) | Replaces the library's collections. |
| [`GET /api/config/node-invites`](library.md#get-apiconfignode-invites) | `GET /api/config/node-invites`: every link for the operator or an Owner; the caller's own links for anyone else, and with `?mine=true`. |
| [`POST /api/config/node-invites`](library.md#post-apiconfignode-invites) | `POST /api/config/node-invites`: mint an enrollment link. Any user may mint one for themselves while branch enrollment is on; the operator or an Owner may mint for any active user. |
| [`DELETE /api/config/node-invites/{id}`](library.md#delete-apiconfignode-invitesid) | `DELETE /api/config/node-invites/{id}`: revoke an unredeemed link. |
| [`GET /api/config/branches`](library.md#get-apiconfigbranches) | `GET /api/config/branches`: this library's join settings: whether new branches may join, the transport mode, and the SPKI fingerprint a joining operator confirms in `self_signed` mode. |
| [`PUT /api/config/branches`](library.md#put-apiconfigbranches) | `PUT /api/config/branches`: turn "Let new branches join" on or off. |
| [`POST /api/config/branches/titles/remove`](library.md#post-apiconfigbranchestitlesremove) | `POST /api/config/branches/titles/remove`: removes the entries of titles from a branch that stopped sharing or left. Delivered files stay. |
| [`PUT /api/config/branches/members-books`](library.md#put-apiconfigbranchesmembers-books) | `PUT /api/config/branches/members-books`: turn this library's "Acquire all of this branch's members' books" on or off for one branch. |
| [`GET /api/config/join/sharing`](library.md#get-apiconfigjoinsharing) | `GET /api/config/join/sharing`: Settings → Library for the operator or an Owner: every library this install is a branch of, with its two sharing settings (`{libraries: [...]}`). |
| [`POST /api/config/join/sharing`](library.md#post-apiconfigjoinsharing) | `POST /api/config/join/sharing`: Settings → Library's sharing and work settings for one library this install is a branch of (the two book-sharing settings, the two work settings, and optionally "List my members' books"). The operator or an Owner (verified from the stored role); anyone else gets 403. |
| [`GET /api/config/join/window`](library.md#get-apiconfigjoinwindow) | `GET /api/config/join/window`: "Books in other libraries" for the operator or an Owner: every book of each library this install is a branch of, or holds a library card for, merged by work, each edition marked when it is this install's person's (`{libraries, cards, books}`). |
| [`POST /api/config/join/connections`](library.md#post-apiconfigjoinconnections) | `POST /api/config/join/connections`: "Add my connections to *library*". |
| [`POST /api/config/join/connections/disconnect`](library.md#post-apiconfigjoinconnectionsdisconnect) | `POST /api/config/join/connections/disconnect`: disconnects one of the person's store accounts here, and with `everywhere` in every linked library it was shared with. |
| [`POST /api/config/join/capacity`](library.md#post-apiconfigjoincapacity) | `POST /api/config/join/capacity`: Settings → Library's job cap for one library this install is a branch of. The operator or an Owner (verified from the stored role); anyone else gets 403. The running link's share changes at once: its claims stop at the new share and its next heartbeat reports the library the new part of the machine. Jobs it already runs finish. |
| [`DELETE /api/config/node-credentials/{node_id}`](library.md#delete-apiconfignode-credentialsnode_id) | `DELETE /api/config/node-credentials/{node_id}`: revoke an enrolled host's credential so it can no longer connect. |
| [`GET /api/library/works`](library.md#get-apilibraryworks) | Lists the library's works, one page at a time. |
| [`GET /api/library/works/meaning`](library.md#get-apilibraryworksmeaning) | Says whether search by meaning and similar works are ready. |
| [`GET /api/library/works/{id}/similar`](library.md#get-apilibraryworksidsimilar) | Lists the held works most like one work. |
| [`GET /api/library/works/{id}`](library.md#get-apilibraryworksid) | Reads one work with its editions, listings, holdings and the caller's own state. |
| [`PUT /api/library/works/{id}/state`](library.md#put-apilibraryworksidstate) | Changes the caller's own state for a work: finished, ratings and tags. |
| [`GET /api/library/works/facets/{facet}`](library.md#get-apilibraryworksfacetsfacet) | Lists the values a facet has across the library's held works, with how many works have each, for the library page's filter menus. `tag` lists the caller's own tags (none for an operator token). |
| [`GET /api/library/editions/{id}/cover`](library.md#get-apilibraryeditionsidcover) | Serves the JPEG cover written beside an edition's acquired file on this node's local destination. |
| [`GET /api/library/editions/{id}/playback`](library.md#get-apilibraryeditionsidplayback) | Reads whether and what the caller may play of an edition, with their own progress and bookmarks. |
| [`GET /api/library/editions/{id}/bookmarks`](library.md#get-apilibraryeditionsidbookmarks) | Lists the caller's bookmarks in an edition. |
| [`POST /api/library/editions/{id}/bookmarks`](library.md#post-apilibraryeditionsidbookmarks) | Adds a bookmark of the caller's in an edition. |
| [`GET /api/library/editions/{id}/sample`](library.md#get-apilibraryeditionsidsample) | Plays a few minutes of an edition this node holds. |
| [`GET /api/uploads`](library.md#get-apiuploads) | Lists the caller's uploads, newest first. |
| [`POST /api/uploads`](library.md#post-apiuploads) | Starts an upload from a manifest of the files to come. |
| [`GET /api/uploads/review`](library.md#get-apiuploadsreview) | Lists uploaded books stored under a new local edition because nothing strong identified them, for an Administrator to check and, when they are a known edition, pin. |
| [`GET /api/uploads/{id}`](library.md#get-apiuploadsid) | Reads one of the caller's uploads with every file: how many bytes each has received (where to resume) and, once ingested, what became of it. |
| [`DELETE /api/uploads/{id}`](library.md#delete-apiuploadsid) | Cancels an upload that has not started ingesting and drops its bytes. |
| [`PATCH /api/uploads/{id}/files/{member_no}`](library.md#patch-apiuploadsidfilesmember_no) | Sends bytes of one file, from the offset in the `Upload-Offset` header. |
| [`POST /api/uploads/{id}/commit`](library.md#post-apiuploadsidcommit) | Commits an upload whose files have all arrived: queues its ingest job. |

### [Playback](playback.md)

Play sessions, audio streams, listening progress and bookmarks.

| Route | Summary |
| --- | --- |
| [`POST /api/playback/sessions`](playback.md#post-apiplaybacksessions) | Opens a play session for the caller. |
| [`POST /api/playback/sessions/{id}/sync`](playback.md#post-apiplaybacksessionsidsync) | Reports where the caller is in a playing session. |
| [`POST /api/playback/sessions/{id}/close`](playback.md#post-apiplaybacksessionsidclose) | Sends a session's last report and ends it. |
| [`GET /api/playback/sessions/{id}/tracks/{index}`](playback.md#get-apiplaybacksessionsidtracksindex) | Streams one track of a play session. |
| [`GET /api/playback/progress`](playback.md#get-apiplaybackprogress) | Lists the caller's progress, most recently played first. |
| [`PUT /api/playback/progress/{edition_id}`](playback.md#put-apiplaybackprogressedition_id) | Sets the caller's progress in an edition outright: Mark finished, Start over, or a place chosen with no session open. |
| [`DELETE /api/playback/progress/{edition_id}`](playback.md#delete-apiplaybackprogressedition_id) | Forgets the caller's progress in an edition. The work's finished flag is left as it is. |
| [`PUT /api/playback/bookmarks/{id}`](playback.md#put-apiplaybackbookmarksid) | Changes one of the caller's bookmarks. |
| [`DELETE /api/playback/bookmarks/{id}`](playback.md#delete-apiplaybackbookmarksid) | Removes one of the caller's bookmarks. |

### [Discover](discover.md)

Recommendations, catalog search and title metadata.

| Route | Summary |
| --- | --- |
| [`POST /api/discover/sync-listening`](discover.md#post-apidiscoversync-listening) | Enqueues a listening-progress sync; 400 when no capable integrations are loaded. |
| [`GET /api/covers/{token}/{size}`](discover.md#get-apicoverstokensize) | Serves one size of a cover the API linked to. |
| [`GET /api/images/{token}`](discover.md#get-apiimagestoken) | Serves a signed image that is not a cover as a 128 px JPEG. |
| [`GET /api/stores/{id}/icon`](discover.md#get-apistoresidicon) | Serves a storefront's or integration's icon as a 128 px JPEG through the cover service. |
| [`GET /api/samples/{token}`](discover.md#get-apisamplestoken) | Plays a storefront's sample of a title. |
| [`POST /api/discover/title-sample`](discover.md#post-apidiscovertitle-sample) | Finds the sample a title's details can play: one cut from the held edition when this node can, else a storefront's. |
| [`GET /api/discover/status`](discover.md#get-apidiscoverstatus) | Reports whether the caller's Discover shelves are stored yet, so provisioning can wait for a new visitor's shelves before sending them in. |
| [`POST /api/discover/refresh`](discover.md#post-apidiscoverrefresh) | Queues a storefront fetch and ranking for the caller's shelves now, as the Discover page's Refresh button asks. Unchanged store calls answer from the day's cache, so a refresh is cheap; a fetch already waiting absorbs the request. |
| [`GET /api/discover/recommendations`](discover.md#get-apidiscoverrecommendations) | Returns the caller's stored Discover shelves; never builds them on the request. |
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
| [`GET /api/request-queue`](wishlist.md#get-apirequest-queue) | Reads the shared open-request queue from its stored ranking. |
| [`POST /api/branch-requests`](wishlist.md#post-apibranch-requests) | Asks a branch for a book it listed. |
| [`POST /api/download-requests`](wishlist.md#post-apidownload-requests) | Asks this library to download a book it holds but has no copy of. |

### [Jobs](jobs.md)

Queued and running control-plane jobs.

| Route | Summary |
| --- | --- |
| [`GET /api/jobs`](jobs.md#get-apijobs) | Returns durable scan/acquire/listen_sync jobs, newest first. |
| [`POST /api/jobs/{id}/cancel`](jobs.md#post-apijobsidcancel) | Cancel a pending job or request cooperative stop of a running job. |

### [Notifications](notifications.md)

Channels, routing rules, patron offers, the delivery log, and each user's own notification settings and inbox.

| Route | Summary |
| --- | --- |
| [`GET /api/notifications/catalog`](notifications.md#get-apinotificationscatalog) | Lists the notification types and the notifier instances the routing screens pick from. |
| [`GET /api/notifications/channels`](notifications.md#get-apinotificationschannels) | Lists the library's channels. |
| [`POST /api/notifications/channels`](notifications.md#post-apinotificationschannels) | Makes a library channel on a notifier instance. |
| [`DELETE /api/notifications/channels/{id}`](notifications.md#delete-apinotificationschannelsid) | Deletes a channel with its routes and delivery-log rows. |
| [`PATCH /api/notifications/channels/{id}`](notifications.md#patch-apinotificationschannelsid) | Renames a channel. |
| [`POST /api/notifications/channels/{id}/resume`](notifications.md#post-apinotificationschannelsidresume) | Resumes a channel repeated failures paused, and clears its failure count. |
| [`POST /api/notifications/channels/{id}/test`](notifications.md#post-apinotificationschannelsidtest) | Sends a test notification to one channel, whatever the routes say. |
| [`POST /api/notifications/notifiers/{id}/deliverability`](notifications.md#post-apinotificationsnotifiersiddeliverability) | Runs a notifier instance's deliverability check. |
| [`GET /api/notifications/routes`](notifications.md#get-apinotificationsroutes) | Lists every routing rule. |
| [`PUT /api/notifications/routes/{type}`](notifications.md#put-apinotificationsroutestype) | Replaces the routes of one operational type. |
| [`GET /api/notifications/offers`](notifications.md#get-apinotificationsoffers) | Lists which notifiers patrons may pick for each patron type. |
| [`PUT /api/notifications/offers/{type}`](notifications.md#put-apinotificationsofferstype) | Replaces which notifiers patrons may pick for one patron type. |
| [`GET /api/notifications/deliveries`](notifications.md#get-apinotificationsdeliveries) | Pages the delivery log, newest first. |
| [`GET /api/notifications/account-mail`](notifications.md#get-apinotificationsaccount-mail) | Reads account mail's settings and whether it can be sent. |
| [`PUT /api/notifications/account-mail`](notifications.md#put-apinotificationsaccount-mail) | Chooses the notifier instance that sends account mail, or turns it off. Choosing another instance clears the last test. |
| [`POST /api/notifications/account-mail/test`](notifications.md#post-apinotificationsaccount-mailtest) | Sends a test message to the caller's own email address through the account-mail notifier. Its delivery marks the notifier tested, which self-service password reset requires. |
| [`GET /api/me/notifications`](notifications.md#get-apimenotifications) | Reads the caller's notification settings: the types they can receive beyond the inbox, what they chose, and their destinations. |
| [`PUT /api/me/notifications/choices/{type}`](notifications.md#put-apimenotificationschoicestype) | Sets what the caller gets for one patron type, replacing the defaults. |
| [`DELETE /api/me/notifications/choices/{type}`](notifications.md#delete-apimenotificationschoicestype) | Drops the caller's choice for one type, so the defaults apply again. |
| [`POST /api/me/notifications/destinations`](notifications.md#post-apimenotificationsdestinations) | Adds a personal destination for the caller. |
| [`DELETE /api/me/notifications/destinations/{id}`](notifications.md#delete-apimenotificationsdestinationsid) | Deletes one of the caller's destinations. |
| [`POST /api/me/notifications/destinations/{id}/resume`](notifications.md#post-apimenotificationsdestinationsidresume) | Resumes one of the caller's destinations that repeated failures paused, and clears its failure count. |
| [`GET /api/me/notifications/inbox`](notifications.md#get-apimenotificationsinbox) | Pages the caller's inbox, newest first. |
| [`POST /api/me/notifications/inbox/read`](notifications.md#post-apimenotificationsinboxread) | Marks one inbox entry, or all of them, read. |

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
| [`GET /api/status/media`](status.md#get-apistatusmedia) | Returns the audio codecs, file types and output encodings this build's media worker handles. |
| [`GET /api/library-status`](status.md#get-apilibrary-status) | `GET /api/library-status`: library-wide health every node administrator may see (node counts, committed capacity, active jobs, delivery backlog), plus detail for the nodes the caller may see. |

### [Settings](settings.md)

Daemon settings and user preferences.

| Route | Summary |
| --- | --- |
| [`GET /api/settings`](settings.md#get-apisettings) | Returns file-backed and runtime-effective Settings plus per-plugin groups. |
| [`PATCH /api/settings`](settings.md#patch-apisettings) | Validates, writes, and reloads allowlisted Settings keys under `reload_lock`. |
| [`GET /api/diagnostics/reports`](settings.md#get-apidiagnosticsreports) | Lists the diagnostics reports this install sent, newest first, with the IDs the collector gave them. |
| [`GET /api/diagnostics/reports/{id}`](settings.md#get-apidiagnosticsreportsid) | Reads a sent report back from the collector with its access code: the stored report (redacted, as sent), the GitHub issues tracking it and the fix, if one is recorded. |
| [`DELETE /api/diagnostics/reports/{id}`](settings.md#delete-apidiagnosticsreportsid) | Deletes the collector's stored copy of a sent report. The collector keeps its one-line index entry and issue links, so the report still counts; the install's history notes the deletion. |
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
| [`POST /api/config/plugin-instances/{id}/provision`](plugins.md#post-apiconfigplugin-instancesidprovision) | `POST /api/config/plugin-instances/{id}/provision`: runs the plugin's guided setup with the operator's token and stores what it returns. |
| [`POST /api/config/plugin-instances/{id}/deprovision`](plugins.md#post-apiconfigplugin-instancesiddeprovision) | `POST /api/config/plugin-instances/{id}/deprovision`: asks the plugin to remove what its guided setup created, before the instance is deleted. |
| [`GET /api/config/plugin-instances/{id}/provisioning`](plugins.md#get-apiconfigplugin-instancesidprovisioning) | `GET /api/config/plugin-instances/{id}/provisioning`: whether the instance's plugin offers a guided setup here, and what it asks for. |
| [`GET /api/config/plugin-instances/{id}/filter`](plugins.md#get-apiconfigplugin-instancesidfilter) | `GET /api/config/plugin-instances/{id}/filter`: which books the storage destination takes. |
| [`PUT /api/config/plugin-instances/{id}/filter`](plugins.md#put-apiconfigplugin-instancesidfilter) | `PUT /api/config/plugin-instances/{id}/filter`: replaces which books the storage destination takes. Acquires that start afterwards use it; files already stored stay where they are. |
| [`GET /api/config/plugin-instances/{id}/intake`](plugins.md#get-apiconfigplugin-instancesidintake) | `GET /api/config/plugin-instances/{id}/intake`: whether the storage instance is a drop folder, and its settings. |
| [`PUT /api/config/plugin-instances/{id}/intake`](plugins.md#put-apiconfigplugin-instancesidintake) | `PUT /api/config/plugin-instances/{id}/intake`: makes the storage instance a drop folder with these settings, or changes a drop folder's settings. Turning a destination into a drop folder is refused while the instance is deployed on any node. |
| [`DELETE /api/config/plugin-instances/{id}/intake`](plugins.md#delete-apiconfigplugin-instancesidintake) | `DELETE /api/config/plugin-instances/{id}/intake`: makes a drop folder a destination again. Refused while the instance is deployed on any node. |
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
| [`GET /api/config/ingress`](configuration.md#get-apiconfigingress) | `GET /api/config/ingress`: the library's public address settings. |
| [`POST /api/config/ingress`](configuration.md#post-apiconfigingress) | `POST /api/config/ingress`: turns ingress on or off, sets what it exposes, or accepts a reported origin. The deployment reconciler then starts or stops the instance. |
| [`GET /api/config/clerk`](configuration.md#get-apiconfigclerk) | `GET /api/config/clerk`. |
| [`POST /api/config/clerk/inspect`](configuration.md#post-apiconfigclerkinspect) | `POST /api/config/clerk/inspect`. |
| [`POST /api/config/clerk/start`](configuration.md#post-apiconfigclerkstart) | `POST /api/config/clerk/start`. |
| [`POST /api/config/clerk/callback`](configuration.md#post-apiconfigclerkcallback) | `POST /api/config/clerk/callback`: redeems the library's code with the sign-in's verifier, seals the card and starts reading the person's books. The pending sign-in is ended whatever the outcome; a failed one is started again. |
| [`POST /api/config/clerk/sign-out`](configuration.md#post-apiconfigclerksign-out) | `POST /api/config/clerk/sign-out`: ends a library card at the library when it can be reached, then drops the card, the books read with it, and the library from the list here whatever the library answered. |
| [`POST /api/config/clerk/request`](configuration.md#post-apiconfigclerkrequest) | `POST /api/config/clerk/request`: asks a library this install holds a card for, with the card, to get a copy of a book: from one of its branches (`from` `branch`) or by downloading it on one of its own store accounts (`from` `library`). |
| [`GET /api/config/branch-enrollment`](configuration.md#get-apiconfigbranch-enrollment) | `GET /api/config/branch-enrollment`: whether branch nodes may enroll. |
| [`PUT /api/config/branch-enrollment`](configuration.md#put-apiconfigbranch-enrollment) | `PUT /api/config/branch-enrollment`: turn branch enrollment on or off. The operator may always; a user only while the stored role is an active Owner. |
| [`GET /api/config/master-key`](configuration.md#get-apiconfigmaster-key) | `GET /api/config/master-key`. |
| [`POST /api/config/master-key/recovery-copy`](configuration.md#post-apiconfigmaster-keyrecovery-copy) | `POST /api/config/master-key/recovery-copy`. |
| [`PUT /api/config/master-key/recovery-copy/state`](configuration.md#put-apiconfigmaster-keyrecovery-copystate) | `PUT /api/config/master-key/recovery-copy/state`. |

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
| [`POST /api/portal/sources/{source}/connections/{account_id}/revoke`](accounts.md#post-apiportalsourcessourceconnectionsaccount_idrevoke) | Marks a store account revoked, signs it out, deletes its stored credentials, then unlinks it. |
| [`POST /api/portal/sources/{source}/connections/{account_id}/pace`](accounts.md#post-apiportalsourcessourceconnectionsaccount_idpace) | Changes how fast Bookclerk downloads from one of the caller's store accounts when it starts the downloads itself, and returns the result. |

### [Clerk sign-in](clerk-sign-in.md)

A clerk's sign-in as a person: redeem an approved code for a session token.

| Route | Summary |
| --- | --- |
| [`POST /api/clerk/v1/token`](clerk-sign-in.md#post-apiclerkv1token) | `POST /api/clerk/v1/token`: redeem an approved clerk sign-in for a session token. The code is spent by any attempt, matching or not. |
| [`GET /api/clerk/v1/connect`](clerk-sign-in.md#get-apiclerkv1connect) | `GET /api/clerk/v1/connect`: upgrade to the clerk socket after the transport check and the library card. |

### [Node protocol](node-protocol.md)

Node-to-node enrollment, socket, artifacts and uploads; each route checks its own credential.

| Route | Summary |
| --- | --- |
| [`POST /api/node/v1/enroll`](node-protocol.md#post-apinodev1enroll) | `POST /api/node/v1/enroll`: redeem an enrollment link. Answers `{library_id, node_id}`; 403 `transport_insecure` when the connection fails the transport policy, 403 `enrollment_refused` for any link that does not redeem. |
| [`GET /api/node/v1/info`](node-protocol.md#get-apinodev1info) | `GET /api/node/v1/info`. |
| [`POST /api/node/v1/enroll/oauth`](node-protocol.md#post-apinodev1enrolloauth) | `POST /api/node/v1/enroll/oauth`: redeem an approved branch sign-in. Answers `{library_id, node_id}` for a branch, plus `connection` and the sealed `bootstrap` for a shared branch; 403 `transport_insecure` for a refused connection, 403 `enrollment_refused` for a code that does not redeem (unknown, used, expired, another client's, a wrong verifier or redirect URI, enrollment off, or the user no longer active). |
| [`GET /api/node/v1/connect`](node-protocol.md#get-apinodev1connect) | `GET /api/node/v1/connect`: upgrade to the branch-node socket after the transport check. 403 `transport_insecure` for a refused connection, 400 `subprotocol_required` without [`SUBPROTOCOL`]. |
| [`GET /api/node/v1/artifacts/{sha256}`](node-protocol.md#get-apinodev1artifactssha256) | `GET /api/node/v1/artifacts/{sha256}`. |
| [`PUT /api/node/v1/uploads/{upload_id}/files/{index}`](node-protocol.md#put-apinodev1uploadsupload_idfilesindex) | `PUT /api/node/v1/uploads/{upload_id}/files/{index}`. |
| [`POST /api/node/v1/uploads/{upload_id}/commit`](node-protocol.md#post-apinodev1uploadsupload_idcommit) | `POST /api/node/v1/uploads/{upload_id}/commit`. |

### [Library copies](library-copies.md)

Downloads of an edition's files the library already holds, by a branch or clerk it granted the copy to; the grant's token is the credential.

| Route | Summary |
| --- | --- |
| [`GET /api/copies/v1/{copy_id}/files/{index}`](library-copies.md#get-apicopiesv1copy_idfilesindex) | Streams one file of a granted copy. |

### [OIDC provider](oidc-provider.md)

Bookclerk as an OpenID Connect provider for other apps.

| Route | Summary |
| --- | --- |
| [`GET /.well-known/openid-configuration`](oidc-provider.md#get-well-knownopenid-configuration) | Serves the OIDC discovery document for registered clients. |
| [`GET /oidc/authorize`](oidc-provider.md#get-oidcauthorize) | Starts the authorization-code flow; unauthenticated users are sent to SPA login. |
| [`POST /oidc/authorize`](oidc-provider.md#post-oidcauthorize) | Completes consent: deny redirects with `access_denied`, grant issues a code. |
| [`POST /oidc/token`](oidc-provider.md#post-oidctoken) | Dispatches token grants. Every refusal is an RFC 6749 §5.2 JSON body ([`TokenError`]), never the branded error page, so a client library can tell `invalid_grant` from `invalid_client`. |
| [`GET /oidc/userinfo`](oidc-provider.md#get-oidcuserinfo) | Returns `sub` / `name` / `preferred_username` for a valid access JWT. |
| [`POST /oidc/revoke`](oidc-provider.md#post-oidcrevoke) | Best-effort refresh-token revoke; always returns 200 per RFC 7009. |
| [`GET /api/auth/oidc/clients`](oidc-provider.md#get-apiauthoidcclients) | Owner/operator list of Bookclerk-as-IdP clients (secrets never included). |
| [`POST /api/auth/oidc/clients`](oidc-provider.md#post-apiauthoidcclients) | Create a client; confidential clients return the generated secret once. |
| [`PUT /api/auth/oidc/clients/{client_id}`](oidc-provider.md#put-apiauthoidcclientsclient_id) | Update redirects, name, scopes, and public/confidential mode; switching a public client to confidential mints its secret, shown once. |
| [`DELETE /api/auth/oidc/clients/{client_id}`](oidc-provider.md#delete-apiauthoidcclientsclient_id) | Delete a registered client. |
| [`POST /api/auth/oidc/clients/{client_id}/rotate-secret`](oidc-provider.md#post-apiauthoidcclientsclient_idrotate-secret) | Mint a new secret for a confidential client; plaintext is returned once. |

### [Player](player.md)

The player API for audiobook apps written for an Audiobookshelf server (`docs/server/player-apps.md`).

| Route | Summary |
| --- | --- |
| [`GET /player/status`](player.md#get-playerstatus) | Tells an app what the server is before it offers a sign-in. |
| [`GET /player/ping`](player.md#get-playerping) | Answers that the player API is up, without a card. |
| [`GET /player/healthcheck`](player.md#get-playerhealthcheck) | Answers `OK` in plain text, without a card, for apps that check health. |
| [`POST /player/login`](player.md#post-playerlogin) | Signs an app in with a username and a player password. |
| [`POST /player/auth/refresh`](player.md#post-playerauthrefresh) | Trades a refresh token for a new access and refresh token. |
| [`POST /player/logout`](player.md#post-playerlogout) | Ends the card a refresh token (or the bearer access token) belongs to. |
| [`GET /player/auth/openid`](player.md#get-playerauthopenid) | Starts the browser sign-in: answers a 302 to the library's consent page, which the app opens in a browser. |
| [`GET /player/auth/openid/mobile-redirect`](player.md#get-playerauthopenidmobile-redirect) | Sends the browser back to the app with the code (or the refusal). |
| [`GET /player/auth/openid/callback`](player.md#get-playerauthopenidcallback) | Trades a browser sign-in's code and PKCE verifier for a card. |
| [`GET /player/socket.io/`](player.md#get-playersocketio) | Opens the live socket. |
| [`GET /player/socket.io`](player.md#get-playersocketio-1) | Opens the live socket for a client that drops the trailing slash; the same socket as [`get_socket`]. |
| [`POST /player/api/authorize`](player.md#post-playerapiauthorize) | The signed-in patron, as at sign-in, without new tokens. |
| [`GET /player/api/me`](player.md#get-playerapime) | The signed-in patron. |
| [`GET /player/api/libraries`](player.md#get-playerapilibraries) | Lists the libraries the patron may open: the one library. |
| [`GET /player/api/libraries/{id}`](player.md#get-playerapilibrariesid) | Reads the library. |
| [`GET /player/api/libraries/{id}/items`](player.md#get-playerapilibrariesiditems) | Lists the library's items, one page at a time. |
| [`GET /player/api/libraries/{id}/series`](player.md#get-playerapilibrariesidseries) | Lists the library's series, one page at a time. |
| [`GET /player/api/libraries/{id}/authors`](player.md#get-playerapilibrariesidauthors) | Lists the library's authors. |
| [`GET /player/api/libraries/{id}/narrators`](player.md#get-playerapilibrariesidnarrators) | Lists the library's narrators. |
| [`GET /player/api/libraries/{id}/collections`](player.md#get-playerapilibrariesidcollections) | Lists the library's collections: none yet. |
| [`GET /player/api/libraries/{id}/playlists`](player.md#get-playerapilibrariesidplaylists) | Lists the patron's playlists in the library: none yet. |
| [`GET /player/api/libraries/{id}/personalized`](player.md#get-playerapilibrariesidpersonalized) | Reads the patron's home shelves: continue listening, recently added and listen again. |
| [`GET /player/api/libraries/{id}/search`](player.md#get-playerapilibrariesidsearch) | Searches the library's items, series, authors and narrators. |
| [`GET /player/api/libraries/{id}/stats`](player.md#get-playerapilibrariesidstats) | Adds up the library's items and their runtimes. |
| [`GET /player/api/libraries/{id}/filterdata`](player.md#get-playerapilibrariesidfilterdata) | Reads the library's filter values. |
| [`GET /player/api/series/{id}`](player.md#get-playerapiseriesid) | Reads one series with its items and the patron's progress through it. |
| [`GET /player/api/authors/{id}`](player.md#get-playerapiauthorsid) | Reads one author, with their items and series when asked for. |
| [`GET /player/api/items/{id}`](player.md#get-playerapiitemsid) | Reads one item; expanded, with its tracks and files. |
| [`POST /player/api/items/batch/get`](player.md#post-playerapiitemsbatchget) | Reads several items at once. |
| [`GET /player/api/items/{id}/cover`](player.md#get-playerapiitemsidcover) | Serves an item's cover. |
| [`POST /player/api/items/{id}/play`](player.md#post-playerapiitemsidplay) | Opens a play session on an item. |
| [`GET /player/api/items/{id}/file/{ino}`](player.md#get-playerapiitemsidfileino) | Streams one audio file of an item. |
| [`GET /player/api/items/{id}/file/{ino}/download`](player.md#get-playerapiitemsidfileinodownload) | Downloads one audio file of an item for offline listening. |
| [`POST /player/api/session/{id}/sync`](player.md#post-playerapisessionidsync) | Reports where the patron is in a playing session. |
| [`POST /player/api/session/{id}/close`](player.md#post-playerapisessionidclose) | Sends a session's last report, if any, and ends it. |
| [`POST /player/api/session/local`](player.md#post-playerapisessionlocal) | Uploads one session an app played offline. |
| [`POST /player/api/session/local-all`](player.md#post-playerapisessionlocal-all) | Uploads every session an app played offline. |
| [`GET /player/api/me/progress/{id}`](player.md#get-playerapimeprogressid) | Reads the patron's progress on an item. |
| [`DELETE /player/api/me/progress/{id}`](player.md#delete-playerapimeprogressid) | Clears the patron's progress on an item. |
| [`PATCH /player/api/me/progress/{id}`](player.md#patch-playerapimeprogressid) | Sets the patron's progress on an item: a seek, Mark finished or Start over. |
| [`PATCH /player/api/me/progress/batch/update`](player.md#patch-playerapimeprogressbatchupdate) | Sets the patron's progress on several items. |
| [`GET /player/api/me/items-in-progress`](player.md#get-playerapimeitems-in-progress) | Lists the items the patron is part way through, most recent first. |
| [`GET /player/api/me/listening-sessions`](player.md#get-playerapimelistening-sessions) | Lists the patron's play sessions, newest first. |
| [`GET /player/api/me/listening-stats`](player.md#get-playerapimelistening-stats) | Sums the patron's listening, by item and by day. |
| [`POST /player/api/me/item/{id}/bookmark`](player.md#post-playerapimeitemidbookmark) | Adds a bookmark to an item. |
| [`PATCH /player/api/me/item/{id}/bookmark`](player.md#patch-playerapimeitemidbookmark) | Renames the bookmark at a time in an item. |
| [`DELETE /player/api/me/item/{id}/bookmark/{time}`](player.md#delete-playerapimeitemidbookmarktime) | Removes the bookmark at a time in an item. |

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
