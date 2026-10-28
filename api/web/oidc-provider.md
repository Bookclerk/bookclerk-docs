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

### Example request

```console
$ curl 'http://127.0.0.1:8787/.well-known/openid-configuration'
```

### Example response

Status `200`: The OIDC discovery document.

```json
{
  "issuer": "string",
  "authorization_endpoint": "string",
  "token_endpoint": "string",
  "userinfo_endpoint": "string",
  "revocation_endpoint": "string",
  "response_types_supported": [
    "string"
  ],
  "grant_types_supported": [
    "string"
  ],
  "code_challenge_methods_supported": [
    "string"
  ],
  "scopes_supported": [
    "string"
  ],
  "subject_types_supported": [
    "string"
  ],
  "id_token_signing_alg_values_supported": [
    "string"
  ],
  "token_endpoint_auth_methods_supported": [
    "string"
  ]
}
```

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

### Example request

```console
$ curl 'http://127.0.0.1:8787/oidc/authorize?client_id=string&redirect_uri=string&response_type=string&code_challenge=string'
```

### Example response

Status `302`: Redirects to sign-in, the consent page, or the client. No body.

Status `400`: The request is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

## `POST /oidc/authorize`

Completes consent: deny redirects with `access_denied`, grant issues a code.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/x-www-form-urlencoded`: [`ConsentBody`](schemas.md#consentbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `client_id` | string | yes | Registered OIDC client id (must exist in the library store). |
| `redirect_uri` | string | yes | Redirect URI; must match a URI registered for the client. |
| `response_type` | string | yes | OAuth `response_type`; only `code` is accepted. |
| `scope` | string or null | no | Space-separated scopes; defaults to `openid profile` when omitted. |
| `state` | string or null | no | Opaque CSRF value echoed back on the redirect. |
| `code_challenge` | string | yes | PKCE S256 challenge (base64url SHA-256 of the verifier). |
| `code_challenge_method` | string or null | no | PKCE method; omitted is treated as `plain` and then rejected. |
| `consent` | string or null | no | `deny` aborts with `access_denied`; any other value proceeds to issue a code. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 302 | Redirects to the client with a code or `access_denied`. |  |
| 400 | The request is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | An operator token was sent, or the user is disabled. | [`ApiError`](schemas.md#apierror) (`application/json`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/oidc/authorize' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -d 'client_id=string&redirect_uri=string&response_type=string&code_challenge=string'
```

### Example response

Status `302`: Redirects to the client with a code or `access_denied`. No body.

Status `400`: The request is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: An operator token was sent, or the user is disabled.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

## `POST /oidc/token`

Dispatches token grants; unknown `grant_type` values return 400.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/x-www-form-urlencoded`: [`TokenForm`](schemas.md#tokenform)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `grant_type` | string | yes | Grant: `authorization_code` or `refresh_token`; anything else is 400. |
| `code` | string or null | no | Authorization code from the redirect (`authorization_code` grant). |
| `redirect_uri` | string or null | no | Must match the URI bound to the consumed authorization code. |
| `client_id` | string or null | no | Client id; required for code exchange, optional (but checked) on refresh. |
| `client_secret` | string or null | no | Client secret for confidential clients (`client_secret_post`). |
| `code_verifier` | string or null | no | PKCE verifier; SHA-256 must match the stored challenge. |
| `refresh_token` | string or null | no | Refresh token to rotate (`refresh_token` grant). |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Tokens. | [`TokenResponse`](schemas.md#tokenresponse) (`application/json`) |
| 400 | The grant is invalid or unsupported. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The client failed to authenticate. | [`ApiError`](schemas.md#apierror) (`application/json`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/oidc/token' \
    -X POST \
    -d 'grant_type=string'
```

### Example response

Status `200`: Tokens.

```json
{
  "access_token": "string",
  "token_type": "string",
  "expires_in": 0,
  "refresh_token": "string",
  "id_token": "string",
  "scope": "string"
}
```

Status `400`: The grant is invalid or unsupported.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: The client failed to authenticate.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

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

### Example request

```console
$ curl 'http://127.0.0.1:8787/oidc/userinfo' \
    -H 'Authorization: Bearer <oidcAccessToken>'
```

### Example response

Status `200`: Claims for the token's user. The body is JSON the schemas do not describe yet.

Status `401`: The access token is missing or invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The token's user is disabled.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The user could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

## `POST /oidc/revoke`

Best-effort refresh-token revoke; always returns 200 per RFC 7009.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/x-www-form-urlencoded`: [`RevokeForm`](schemas.md#revokeform)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `token` | string | yes | Token to revoke; treated as a refresh token (hashed before lookup). |
| `token_type_hint` | string or null | no | Optional hint ignored; this server only stores refresh tokens. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Always 200 (RFC 7009). |  |

### Example request

```console
$ curl 'http://127.0.0.1:8787/oidc/revoke' \
    -X POST \
    -d 'token=string'
```

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

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/clients' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: Registered clients (no secrets). The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller's role does not allow this route.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/oidc/clients`

Create a client; confidential clients return the generated secret once.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`OidcClientWrite`](schemas.md#oidcclientwrite)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `client_id` | string or null | no | Public client_id (create only; path wins on update). |
| `name` | string or null | no | Operator-facing display name. |
| `redirect_uris` | array of string | no | Allowed OAuth redirect URIs. |
| `confidential` | boolean | no | When true, mint a client secret (confidential). When false, public PKCE. |
| `issue_refresh_token` | boolean | no | When true, token responses include a refresh token. |
| `allowed_scopes` | array of string | no | Scopes this client may request (`openid`, `profile`, `email`). |
| `enabled` | boolean | no | When true, authorize and token endpoints accept this client. |
| `current_password` | string or null | no | Required for a non-elevated Owner whose portal session is older than 15 minutes. |

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

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/clients' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`: The client; a confidential client's secret appears once. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `409`: A client with this id already exists.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `client_id` | string or null | no | Public client_id (create only; path wins on update). |
| `name` | string or null | no | Operator-facing display name. |
| `redirect_uris` | array of string | no | Allowed OAuth redirect URIs. |
| `confidential` | boolean | no | When true, mint a client secret (confidential). When false, public PKCE. |
| `issue_refresh_token` | boolean | no | When true, token responses include a refresh token. |
| `allowed_scopes` | array of string | no | Scopes this client may request (`openid`, `profile`, `email`). |
| `enabled` | boolean | no | When true, authorize and token endpoints accept this client. |
| `current_password` | string or null | no | Required for a non-elevated Owner whose portal session is older than 15 minutes. |

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

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`: The client after the change. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: No such client.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `client_id` | string or null | no | Public client_id (create only; path wins on update). |
| `name` | string or null | no | Operator-facing display name. |
| `redirect_uris` | array of string | no | Allowed OAuth redirect URIs. |
| `confidential` | boolean | no | When true, mint a client secret (confidential). When false, public PKCE. |
| `issue_refresh_token` | boolean | no | When true, token responses include a refresh token. |
| `allowed_scopes` | array of string | no | Scopes this client may request (`openid`, `profile`, `email`). |
| `enabled` | boolean | no | When true, authorize and token endpoints accept this client. |
| `current_password` | string or null | no | Required for a non-elevated Owner whose portal session is older than 15 minutes. |

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

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `204`: The client was deleted. No body.

Status `400`: The body is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: No such client.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `409`: The client is provided by a plugin; disable it instead.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `current_password` | string or null | no | Required for a non-elevated Owner whose portal session is older than 15 minutes. |

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

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>/rotate-secret' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`: The new secret (shown once). The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: No such client.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
