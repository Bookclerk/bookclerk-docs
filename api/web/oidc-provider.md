# OIDC provider

Bookclerk as an OpenID Connect provider for other apps.

Back to the [web API reference](README.md).

## `GET /.well-known/openid-configuration`

Serves the OIDC discovery document for registered clients.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The OIDC discovery document. | [`DiscoveryDocument`](schemas.md#discoverydocument) (`application/json`) |

## `GET /oidc/authorize`

Starts the authorization-code flow; unauthenticated users are sent to SPA login.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `client_id` | query | string | yes | Registered OIDC client id (must exist in the library store). |
| `redirect_uri` | query | string | yes | Redirect URI; must match a URI registered for the client. |
| `response_type` | query | string | yes | OAuth `response_type`; only `code` is accepted. |
| `scope` | query | string | no | Space-separated scopes; defaults to `openid profile` when omitted. |
| `state` | query | string | no | Opaque CSRF value echoed back on the redirect. |
| `code_challenge` | query | string | yes | PKCE S256 challenge (base64url SHA-256 of the verifier). |
| `code_challenge_method` | query | string | no | PKCE method; omitted is treated as `plain` and then rejected. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 302 | Redirects to sign-in, the consent page, or the client. |  |
| 400 | The request is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |

## `POST /oidc/authorize`

Completes consent: deny redirects with `access_denied`, grant issues a code.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/x-www-form-urlencoded`: [`ConsentBody`](schemas.md#consentbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 302 | Redirects to the client with a code or `access_denied`. |  |
| 400 | The request is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | An operator token was sent, or the user is disabled. | [`ApiError`](schemas.md#apierror) (`application/json`) |

## `POST /oidc/token`

Dispatches token grants; unknown `grant_type` values return 400.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/x-www-form-urlencoded`: [`TokenForm`](schemas.md#tokenform)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Tokens. | [`TokenResponse`](schemas.md#tokenresponse) (`application/json`) |
| 400 | The grant is invalid or unsupported. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The client failed to authenticate. | [`ApiError`](schemas.md#apierror) (`application/json`) |

## `GET /oidc/userinfo`

Returns `sub` / `name` / `preferred_username` for a valid access JWT.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `oidcAccessToken`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Claims for the token's user. | any JSON (`application/json`) |
| 401 | The access token is missing or invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The token's user is disabled. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The user could not be read. | [`ApiError`](schemas.md#apierror) (`application/json`) |

## `POST /oidc/revoke`

Best-effort refresh-token revoke; always returns 200 per RFC 7009.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/x-www-form-urlencoded`: [`RevokeForm`](schemas.md#revokeform)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Always 200 (RFC 7009). |  |

## `GET /api/auth/oidc/clients`

Owner/operator list of Bookclerk-as-IdP clients (secrets never included).

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Registered clients (no secrets). | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/oidc/clients`

Create a client; confidential clients return the generated secret once.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`OidcClientWrite`](schemas.md#oidcclientwrite)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The client; a confidential client's secret appears once. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | A client with this id already exists. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `PUT /api/auth/oidc/clients/{client_id}`

Update redirects, name, scopes, and public/confidential mode (no secret echo unless rotating).

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `client_id` | path | string | yes | Client id. |

### Request body

`application/json`: [`OidcClientWrite`](schemas.md#oidcclientwrite)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The client after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such client. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/auth/oidc/clients/{client_id}`

Delete a registered client.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `client_id` | path | string | yes | Client id. |

### Request body

Optional; only `current_password` is read, to confirm an Owner outside the recent sign-in window. Optional.

`application/json`: [`OidcClientWrite`](schemas.md#oidcclientwrite) or null

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 204 | The client was deleted. |  |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such client. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The client is provided by a plugin; disable it instead. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/auth/oidc/clients/{client_id}/rotate-secret`

Mint a new secret for a confidential client; plaintext is returned once.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `client_id` | path | string | yes | Client id. |

### Request body

Optional.

`application/json`: [`RotateSecretBody`](schemas.md#rotatesecretbody) or null

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The new secret (shown once). | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such client. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |
