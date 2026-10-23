# Auth

Sign-in, sessions, passkeys, TOTP and the operator's elevation.

Back to the [web API reference](README.md).

## `POST /api/auth/login`

Validates the operator token (throttled, constant-time) and issues a hashed session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`LoginRequest`](schemas.md#loginrequest)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Signed in; the response sets the operator session cookie. | any JSON (`application/json`) |
| 401 | The token is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | An Owner exists, so operator sign-in is off (`operator_login_disabled`); sign in as a user. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 429 | Too many attempts; retry after `Retry-After` seconds. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/signin`

Public sign-in picker: operator-token availability, SSO IdPs, integration logins.

# Errors

Returns 500 when the owner-count query fails.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The sign-in methods this library offers. | any JSON (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/tray-handoff/prepare`

Mint a single-use tray Open Bookclerk ticket (Bearer, direct loopback only).

Returns a cryptographically random code and its lifetime. The durable
operator token stays in `Authorization`. Each call replaces any unused
ticket. The code is registered for log redaction (including percent-encoded
form). Lifetime comes from live `[daemon.auth] tray_handoff_ttl_secs`
(default 180, clamped to 30..=900).

# Errors

Returns 403 when the request is not a direct loopback call, 401 when Bearer
does not match, or 429 when this client is locked out.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | A single-use tray ticket. | any JSON (`application/json`) |
| 204 | Operator authentication is off, so no ticket is needed. |  |
| 401 | The Bearer token is missing or wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The request did not come straight from loopback. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 429 | Too many failed attempts from this client; retry after `Retry-After` seconds. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/tray-handoff`

Consume the pending tray ticket and set a localhost operator session cookie.

Direct loopback only (TCP peer, localhost `Host`, no forwarded headers).
Requires the one-time `code` from prepare; `?token=` is ignored and cannot
mint a session. Wrong or missing codes do not consume the slot.

# Errors

Returns 403 when the request is not a direct loopback call or the code does
not match a still-valid ticket.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `code` | query | string | no | Presented handoff code; missing or empty fails closed without consuming the slot. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 307 | Redirects into the app with a localhost operator session. |  |
| 403 | The ticket is missing, used or not from loopback. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/me`

Resolves the caller and returns `/me` JSON; unauthenticated callers get 401.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Who the caller is (operator, portal user, or impersonation). | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/elevate`

Owner portal session + password re-auth → short-lived elevated operator cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`ElevateRequest`](schemas.md#elevaterequest)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Elevated; the response sets a short-lived operator cookie. | any JSON (`application/json`) |
| 400 | Operator auth is off, or the password is empty. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The caller is already an operator. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/auth/elevate`

End elevation by revoking the elevated operator session cookie.

**Access:** The operator (token or session) only.

**Credentials:** `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Elevation ended. | any JSON (`application/json`) |
| 400 | The session is not an elevated one. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/bootstrap`

Operator-only bootstrap of the first Owner when none exist.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`BootstrapRequest`](schemas.md#bootstraprequest)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The first Owner was created. | any JSON (`application/json`) |
| 400 | The email is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | An Owner already exists. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `PUT /api/auth/password`

Set password for self (portal) or target user (operator/admin); revokes sessions.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`SetPasswordRequest`](schemas.md#setpasswordrequest)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The password was set and other sessions revoked. | any JSON (`application/json`) |
| 400 | The password is too weak or the request is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such user. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/password`

Local password login → portal session cookie, or a TOTP challenge when enabled.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`PasswordLoginRequest`](schemas.md#passwordloginrequest)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Signed in (portal session cookie), or a TOTP challenge. | any JSON (`application/json`) |
| 401 | The login or password is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The account is disabled (`account_disabled`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 429 | Too many attempts; retry after `Retry-After` seconds. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/impersonate`

Operator (or elevated) session starts impersonating a User.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`ImpersonateRequest`](schemas.md#impersonaterequest)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The session now acts as the user. | any JSON (`application/json`) |
| 400 | Operator auth is off. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such user. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/auth/impersonate`

Clear impersonation on the current operator session.

**Access:** The operator (token or session) only.

**Credentials:** `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Impersonation ended. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/branch-consent`

`GET /api/auth/branch-consent?<authorize params>`: what the consent page
shows. 400 `invalid_request` for a malformed request (the page then
shows the problem and never redirects).

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `client_id` | query | string | yes | Must be [`BRANCH_CLIENT_ID`]. |
| `redirect_uri` | query | string | yes | Where the code goes: loopback or https. |
| `response_type` | query | string | yes | Must be `code`. |
| `scope` | query | string | no | Must be [`BRANCH_SCOPE`]. |
| `state` | query | string | no | Echoed on the redirect. |
| `code_challenge` | query | string | yes | PKCE S256 challenge. |
| `code_challenge_method` | query | string | no | Must be `S256`. |
| `branch_name` | query | string | no | The name the branch asks to join as. |
| `clerk_offer` | query | string | no | `1` when the joining install offers to join as a clerk ([`bookclerk_node_proto::CLERK_OFFER_PARAM`]). |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | What the consent page shows. | any JSON (`application/json`) |
| 400 | The authorize parameters are invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller is not a library user (`user_required`; an operator session counts only while impersonating) or the account is inactive. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 503 | The library could not be read; try again. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/branch-consent`

`POST /api/auth/branch-consent`: approve or deny. Answers `{redirect}`,
the URL to send the browser to: the branch's callback with a code (or
`error=access_denied`). Approving needs an active signed-in user, host
enrollment on, and a connection the transport policy allows.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Request body

`application/json`: [`BranchConsentBody`](schemas.md#branchconsentbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | `{redirect}` back to the branch. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller is not an active library user, or a clerk join lacks the Owner role or a passed step-up (`step_up_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | This library's database cannot be shared with a clerk (`database_not_shareable`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 429 | Too many wrong step-up codes; wait a few minutes (`step_up_locked`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 503 | The library could not be read; try again. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/branch-consent/step-up`

`POST /api/auth/branch-consent/step-up`: starts a passkey step-up for a
clerk consent (an Owner with a passkey). Answers the WebAuthn request
options with a `challenge_id` to send back in the consent's `step_up`.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Passkey request options. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller is not an active library user, or not an Owner (`owner_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The account has no passkey (`no_passkey`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 503 | The library could not be read, or the passkey check could not start. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/logout`

Revokes hashed operator and portal sessions and clears both cookies.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Signed out; both session cookies are cleared. | any JSON (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/sessions`

List sessions for the current principal only (not every operator session).

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The caller's sessions. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The user has no sign-in identity. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/auth/sessions/{id}`

Revoke a session by id. Operators may revoke any operator session;
portal users may only revoke their own.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | integer (int64) | yes | Session id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The session was revoked. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such session. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `PATCH /api/auth/profile`

Updates the caller's display name and/or email.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PatchProfileRequest`](schemas.md#patchprofilerequest)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The profile after the change. | any JSON (`application/json`) |
| 400 | A value is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The caller's user no longer exists. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The email is used by another account (`email_in_use`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `PUT /api/auth/profile/avatar`

Accepts a JPEG/PNG/WebP upload and stores it under `files_dir/avatars`.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`image/*`: [`BinaryBody`](schemas.md#binarybody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The avatar was stored. | any JSON (`application/json`) |
| 400 | The image is not JPEG, PNG or WebP. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 413 | The image is too large. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/auth/profile/avatar`

Removes the caller's stored avatar, if any.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The avatar was removed. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/oidc/providers`

Lists enabled IdPs for the login picker (id and display name only).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Enabled identity providers (id and name). | any JSON (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/oidc/identities`

Lists OIDC portal identities for the cookie-authenticated caller.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The caller's linked identities. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/oidc/login`

Starts an OIDC login authorize redirect, honoring login throttle.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `provider` | query | string | no | IdP id to start; omitted when the caller has only one enabled provider. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 302 | Redirects to the identity provider. |  |
| 400 | The provider is unknown or disabled. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 429 | Too many attempts; retry after `Retry-After` seconds. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/oidc/elevate`

Starts an Owner re-auth elevate flow against a linked IdP (`prompt=login`).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `provider` | query | string | no | IdP id to start; omitted when the caller has only one enabled provider. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 302 | Redirects to the identity provider for re-authentication. |  |
| 400 | The provider is unknown or disabled. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 429 | Too many attempts; retry after `Retry-After` seconds. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/oidc/callback`

GET callback (most IdPs); finishes the code exchange.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `code` | query | string | no | Authorization code to exchange; required unless `error` is set. |
| `state` | query | string | no | CSRF `state` that must match the `bookclerk_oidc_tx` cookie. |
| `error` | query | string | no | IdP-reported deny/error; redirects the browser to `/?sso_error=denied`. |
| `user` | query | string | no | Apple first-login name/email JSON (form_post only). |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 302 | Redirects into the app, signed in. |  |
| 400 | The state or code is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/oidc/callback`

POST callback (Apple `form_post`); finishes the code exchange.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/x-www-form-urlencoded`: [`CallbackParams`](schemas.md#callbackparams)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 302 | Redirects into the app, signed in. |  |
| 400 | The state or code is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/oidc/config`

Operator GET: broker settings plus secret-presence flags (never plaintext).

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Broker settings, with secrets as presence flags. | [`OidcConfigResponse`](schemas.md#oidcconfigresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `PUT /api/auth/oidc/config`

Operator PUT: writes next-generation sealed secrets, then publishes config.toml.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`OidcConfigPut`](schemas.md#oidcconfigput)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The settings after the change. | [`OidcConfigResponse`](schemas.md#oidcconfigresponse) (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/passkeys`

Lists this user's stored passkeys as `{ id, credential_id }` rows.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The caller's passkeys. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/passkeys/register/begin`

Starts passkey registration (skips reauth only for first local-only setup).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`PasskeyReauthBody`](schemas.md#passkeyreauthbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | WebAuthn creation options. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/passkeys/register/finish`

Completes registration, stores the passkey, and revokes elevated operator sessions.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`CeremonyFinish`](schemas.md#ceremonyfinish)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The passkey was stored. | any JSON (`application/json`) |
| 400 | The attestation is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The passkey is already registered. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/auth/passkeys/{id}`

Deletes one passkey after recent reauth; 404 when the id is not this user's.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | integer (int64) | yes | Passkey id. |

### Request body

`application/json`: [`PasskeyReauthBody`](schemas.md#passkeyreauthbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The passkey was deleted. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such passkey for this user. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/passkeys/login/begin`

Rate-limits and starts passkey login; records a failure on any error.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`LoginBegin`](schemas.md#loginbegin)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | WebAuthn request options. | any JSON (`application/json`) |
| 400 | The request is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | No user has this login name or email. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The user is disabled. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The user has no usable passkeys. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 429 | Too many attempts; try later. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/passkeys/login/finish`

Verifies the assertion, updates the credential counter, and issues a portal session.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`CeremonyFinish`](schemas.md#ceremonyfinish)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Signed in; the response sets the portal session cookie. | any JSON (`application/json`) |
| 400 | The challenge is unknown or expired, or the credential is malformed. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The assertion is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The user is disabled. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/passkeys/elevate/begin`

Starts Owner elevation; 403 unless the signed-in user is an Owner with passkeys.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | WebAuthn request options. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The caller has no passkeys. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/passkeys/elevate/finish`

Completes Owner elevation only when the assertion is user-verified.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`CeremonyFinish`](schemas.md#ceremonyfinish)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Elevated; the response sets a short-lived operator cookie. | any JSON (`application/json`) |
| 400 | The challenge is unknown or expired, or the credential is malformed. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/totp/login`

Completes password+TOTP login and issues a portal session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`LoginVerify`](schemas.md#loginverify)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Signed in; the response sets the portal session cookie. | any JSON (`application/json`) |
| 401 | The challenge or code is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 429 | Too many attempts; try later. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/totp/enroll/begin`

Starts TOTP enrollment: new pending secret, otpauth URL, and QR SVG.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`TotpReauthBody`](schemas.md#totpreauthbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | A pending secret, its otpauth URL and a QR code SVG. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | TOTP is already on. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/totp/enroll/finish`

Confirms the pending secret with a valid code and sets `users.totp_enabled`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`EnrollFinish`](schemas.md#enrollfinish)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | TOTP is on. | any JSON (`application/json`) |
| 400 | No enrollment is pending; start one first. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the code is wrong (`invalid_code`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | TOTP is already on. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/totp`

GET `/api/auth/totp` — whether TOTP is confirmed for this user.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Whether TOTP is on for the caller. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/auth/totp`

Disables TOTP after recent reauth and deletes sealed secrets.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`TotpReauthBody`](schemas.md#totpreauthbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | TOTP is off. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/auth/mfa-policy`

Current MFA policy (`require_second_factor`).

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Whether a second factor is required. | any JSON (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `PUT /api/auth/mfa-policy`

Persists `daemon.auth.require_second_factor` to `config.toml`.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`MfaPolicyPut`](schemas.md#mfapolicyput)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The policy after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |
