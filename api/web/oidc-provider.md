# OIDC provider

Bookclerk as an OpenID Connect provider for other apps.

Back to the [web API reference](README.md).

## `GET /.well-known/openid-configuration`

Serves the OIDC discovery document for registered clients.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — The OIDC discovery document. Body: [`DiscoveryDocument`](schemas.md#discoverydocument) (`application/json`)
  - `issuer` *string*, **required** — Issuer origin used as `iss` in minted JWTs (`public_origin` or loopback).
  - `authorization_endpoint` *string*, **required** — Absolute URL of the authorization endpoint (`/oidc/authorize`).
  - `token_endpoint` *string*, **required** — Absolute URL of the token endpoint (`/oidc/token`).
  - `userinfo_endpoint` *string*, **required** — Absolute URL of the userinfo endpoint (`/oidc/userinfo`).
  - `revocation_endpoint` *string*, **required** — Absolute URL of the refresh-token revocation endpoint (`/oidc/revoke`).
  - `response_types_supported` *array of string*, **required** — Supported `response_type` values; this server only issues `code`.
  - `grant_types_supported` *array of string*, **required** — Supported token grants: `authorization_code` and `refresh_token`.
  - `code_challenge_methods_supported` *array of string*, **required** — PKCE methods; only S256 is accepted (`plain` is rejected).
  - `scopes_supported` *array of string*, **required** — Scopes advertised to clients (`openid`, `profile`, and `email`).
  - `subject_types_supported` *array of string*, **required** — Subject identifier types; only public (stable user id) is used.
  - `id_token_signing_alg_values_supported` *array of string*, **required** — ID-token signing algorithms; tokens are HS256 with a derived key.
  - `token_endpoint_auth_methods_supported` *array of string*, **required** — Client auth methods: public PKCE (`none`) or `client_secret_post`.

### Example request

```console
$ curl 'http://127.0.0.1:8787/.well-known/openid-configuration'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/.well-known/openid-configuration'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/.well-known/openid-configuration")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/.well-known/openid-configuration");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/.well-known/openid-configuration").await?;
println!("{} {}", response.status(), response.text().await?);
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

Schema:

```json
{
  "type": "object",
  "description": "OIDC discovery document served at `/.well-known/openid-configuration`.",
  "required": [
    "issuer",
    "authorization_endpoint",
    "token_endpoint",
    "userinfo_endpoint",
    "revocation_endpoint",
    "response_types_supported",
    "grant_types_supported",
    "code_challenge_methods_supported",
    "scopes_supported",
    "subject_types_supported",
    "id_token_signing_alg_values_supported",
    "token_endpoint_auth_methods_supported"
  ],
  "properties": {
    "issuer": {
      "type": "string",
      "description": "Issuer origin used as `iss` in minted JWTs (`public_origin` or loopback)."
    },
    "authorization_endpoint": {
      "type": "string",
      "description": "Absolute URL of the authorization endpoint (`/oidc/authorize`)."
    },
    "token_endpoint": {
      "type": "string",
      "description": "Absolute URL of the token endpoint (`/oidc/token`)."
    },
    "userinfo_endpoint": {
      "type": "string",
      "description": "Absolute URL of the userinfo endpoint (`/oidc/userinfo`)."
    },
    "revocation_endpoint": {
      "type": "string",
      "description": "Absolute URL of the refresh-token revocation endpoint (`/oidc/revoke`)."
    },
    "response_types_supported": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Supported `response_type` values; this server only issues `code`."
    },
    "grant_types_supported": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Supported token grants: `authorization_code` and `refresh_token`."
    },
    "code_challenge_methods_supported": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "PKCE methods; only S256 is accepted (`plain` is rejected)."
    },
    "scopes_supported": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Scopes advertised to clients (`openid`, `profile`, and `email`)."
    },
    "subject_types_supported": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Subject identifier types; only public (stable user id) is used."
    },
    "id_token_signing_alg_values_supported": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "ID-token signing algorithms; tokens are HS256 with a derived key."
    },
    "token_endpoint_auth_methods_supported": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Client auth methods: public PKCE (`none`) or `client_secret_post`."
    }
  }
}
```

## `GET /oidc/authorize`

Starts the authorization-code flow; unauthenticated users are sent to SPA login.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Parameters

- `client_id` (query) *string*, **required** — Registered OIDC client id (must exist in the library store).
- `redirect_uri` (query) *string*, **required** — Redirect URI; must match a URI registered for the client.
- `response_type` (query) *string*, **required** — OAuth `response_type`; only `code` is accepted.
- `scope` (query) *string* — Space-separated scopes; defaults to `openid profile` when omitted.
- `state` (query) *string* — Opaque CSRF value echoed back on the redirect.
- `code_challenge` (query) *string*, **required** — PKCE S256 challenge (base64url SHA-256 of the verifier).
- `code_challenge_method` (query) *string* — PKCE method; omitted is treated as `plain` and then rejected.

### Responses

- `302` — Redirects to sign-in, the consent page, or the client. No body.
- `400` — The request is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.

### Example request

```console
$ curl 'http://127.0.0.1:8787/oidc/authorize?client_id=string&redirect_uri=string&response_type=string&code_challenge=string'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/oidc/authorize?client_id=string&redirect_uri=string&response_type=string&code_challenge=string'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/oidc/authorize?client_id=string&redirect_uri=string&response_type=string&code_challenge=string")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/oidc/authorize?client_id=string&redirect_uri=string&response_type=string&code_challenge=string");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/oidc/authorize?client_id=string&redirect_uri=string&response_type=string&code_challenge=string").await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `302`: Redirects to sign-in, the consent page, or the client. No body.

Status `400`: The request is invalid.

```json
{
  "error": "bad_request",
  "message": "The request could not be understood.",
  "status": 400
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

## `POST /oidc/authorize`

Completes consent: deny redirects with `access_denied`, grant issues a code.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/x-www-form-urlencoded`: [`ConsentBody`](schemas.md#consentbody)

- `client_id` *string*, **required** — Registered OIDC client id (must exist in the library store).
- `redirect_uri` *string*, **required** — Redirect URI; must match a URI registered for the client.
- `response_type` *string*, **required** — OAuth `response_type`; only `code` is accepted.
- `scope` *string or null* — Space-separated scopes; defaults to `openid profile` when omitted.
- `state` *string or null* — Opaque CSRF value echoed back on the redirect.
- `code_challenge` *string*, **required** — PKCE S256 challenge (base64url SHA-256 of the verifier).
- `code_challenge_method` *string or null* — PKCE method; omitted is treated as `plain` and then rejected.
- `consent` *string or null* — `deny` aborts with `access_denied`; any other value proceeds to issue a code.

### Responses

- `302` — Redirects to the client with a code or `access_denied`. No body.
- `400` — The request is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — An operator token was sent, or the user is disabled. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.

### Example request

```console
$ curl 'http://127.0.0.1:8787/oidc/authorize' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -d 'client_id=string&redirect_uri=string&response_type=string&code_challenge=string'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/oidc/authorize' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/x-www-form-urlencoded' `
    -Body 'client_id=string&redirect_uri=string&response_type=string&code_challenge=string'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/oidc/authorize",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    data={"client_id": "string", "redirect_uri": "string", "response_type": "string", "code_challenge": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/oidc/authorize", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787" },
  body: new URLSearchParams("client_id=string&redirect_uri=string&response_type=string&code_challenge=string"),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/oidc/authorize")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .form(&[("client_id", "string"), ("redirect_uri", "string"), ("response_type", "string"), ("code_challenge", "string")])
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `302`: Redirects to the client with a code or `access_denied`. No body.

Status `400`: The request is invalid.

```json
{
  "error": "bad_request",
  "message": "The request could not be understood.",
  "status": 400
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `403`: An operator token was sent, or the user is disabled.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

## `POST /oidc/token`

Dispatches token grants; unknown `grant_type` values return 400.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/x-www-form-urlencoded`: [`TokenForm`](schemas.md#tokenform)

- `grant_type` *string*, **required** — Grant: `authorization_code` or `refresh_token`; anything else is 400.
- `code` *string or null* — Authorization code from the redirect (`authorization_code` grant).
- `redirect_uri` *string or null* — Must match the URI bound to the consumed authorization code.
- `client_id` *string or null* — Client id; required for code exchange, optional (but checked) on refresh.
- `client_secret` *string or null* — Client secret for confidential clients (`client_secret_post`).
- `code_verifier` *string or null* — PKCE verifier; SHA-256 must match the stored challenge.
- `refresh_token` *string or null* — Refresh token to rotate (`refresh_token` grant).

### Responses

- `200` — Tokens. Body: [`TokenResponse`](schemas.md#tokenresponse) (`application/json`)
  - `access_token` *string*, **required** — HS256 JWT access token bound to the User (`token_use=access`).
  - `token_type` *string*, **required** — Always `Bearer` for this server.
  - `expires_in` *integer (int64)*, **required** — Access-token lifetime in seconds (same as [`ACCESS_TTL_SECS`]).
  - `refresh_token` *string or null* — Opaque refresh token when the client is allowed to receive one.
  - `id_token` *string*, **required** — HS256 ID token; profile/email claims follow the granted scope.
  - `scope` *string*, **required** — Space-separated scopes granted to this token pair.
- `400` — The grant is invalid or unsupported. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The client failed to authenticate. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.

### Example request

```console
$ curl 'http://127.0.0.1:8787/oidc/token' \
    -X POST \
    -d 'grant_type=string'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/oidc/token' `
    -Method Post `
    -ContentType 'application/x-www-form-urlencoded' `
    -Body 'grant_type=string'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/oidc/token",
    data={"grant_type": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/oidc/token", {
  method: "POST",
  body: new URLSearchParams("grant_type=string"),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/oidc/token")
    .form(&[("grant_type", "string")])
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
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

Schema:

```json
{
  "type": "object",
  "description": "Successful token-endpoint JSON (access, optional refresh, and ID tokens).",
  "required": [
    "access_token",
    "token_type",
    "expires_in",
    "id_token",
    "scope"
  ],
  "properties": {
    "access_token": {
      "type": "string",
      "description": "HS256 JWT access token bound to the User (`token_use=access`)."
    },
    "token_type": {
      "type": "string",
      "description": "Always `Bearer` for this server."
    },
    "expires_in": {
      "type": "integer",
      "format": "int64",
      "description": "Access-token lifetime in seconds (same as [`ACCESS_TTL_SECS`])."
    },
    "refresh_token": {
      "type": [
        "string",
        "null"
      ],
      "description": "Opaque refresh token when the client is allowed to receive one."
    },
    "id_token": {
      "type": "string",
      "description": "HS256 ID token; profile/email claims follow the granted scope."
    },
    "scope": {
      "type": "string",
      "description": "Space-separated scopes granted to this token pair."
    }
  }
}
```

Status `400`: The grant is invalid or unsupported.

```json
{
  "error": "bad_request",
  "message": "The request could not be understood.",
  "status": 400
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `401`: The client failed to authenticate.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

## `GET /oidc/userinfo`

Returns `sub` / `name` / `preferred_username` for a valid access JWT.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `oidcAccessToken`

### Responses

- `200` — Claims for the token's user. Body: [`UserinfoResponse`](schemas.md#userinforesponse) (`application/json`)
  - `sub` *string*, **required** — Subject: the Bookclerk `users.id` as a string.
  - `name` *string or null* — Display name (`profile` scope), when the user has one.
  - `preferred_username` *string or null* — Login name (`profile` scope), when the user has one.
  - `email` *string or null* — Email (`email` scope), when the user has one.
  - `email_verified` *boolean or null* — Always `true` when `email` is present: Bookclerk trusts the address the Owner recorded.
- `401` — The access token is missing or invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The token's user is disabled. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The user could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.

### Example request

```console
$ curl 'http://127.0.0.1:8787/oidc/userinfo' \
    -H 'Authorization: Bearer <oidcAccessToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/oidc/userinfo' `
    -Headers @{ Authorization = 'Bearer <oidcAccessToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/oidc/userinfo",
    headers={"Authorization": "Bearer <oidcAccessToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/oidc/userinfo", {
  headers: { Authorization: "Bearer <oidcAccessToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/oidc/userinfo")
    .header("Authorization", "Bearer <oidcAccessToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Claims for the token's user.

```json
{
  "sub": "7",
  "name": "Sam",
  "preferred_username": "sam",
  "email": "sam@example.com",
  "email_verified": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Standard OIDC claims for `GET /oidc/userinfo`, limited to the token's scopes.",
  "required": [
    "sub"
  ],
  "properties": {
    "sub": {
      "type": "string",
      "description": "Subject: the Bookclerk `users.id` as a string.",
      "example": "7"
    },
    "name": {
      "type": [
        "string",
        "null"
      ],
      "description": "Display name (`profile` scope), when the user has one.",
      "example": "Sam"
    },
    "preferred_username": {
      "type": [
        "string",
        "null"
      ],
      "description": "Login name (`profile` scope), when the user has one.",
      "example": "sam"
    },
    "email": {
      "type": [
        "string",
        "null"
      ],
      "description": "Email (`email` scope), when the user has one.",
      "example": "sam@example.com"
    },
    "email_verified": {
      "type": [
        "boolean",
        "null"
      ],
      "description": "Always `true` when `email` is present: Bookclerk trusts the address the Owner recorded.",
      "example": true
    }
  }
}
```

Status `401`: The access token is missing or invalid.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `403`: The token's user is disabled.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `500`: The user could not be read.

```json
{
  "error": "internal_error",
  "message": "Something went wrong on the Bookclerk daemon.",
  "status": 500
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

## `POST /oidc/revoke`

Best-effort refresh-token revoke; always returns 200 per RFC 7009.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/x-www-form-urlencoded`: [`RevokeForm`](schemas.md#revokeform)

- `token` *string*, **required** — Token to revoke; treated as a refresh token (hashed before lookup).
- `token_type_hint` *string or null* — Optional hint ignored; this server only stores refresh tokens.

### Responses

- `200` — Always 200 (RFC 7009). No body.

### Example request

```console
$ curl 'http://127.0.0.1:8787/oidc/revoke' \
    -X POST \
    -d 'token=string'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/oidc/revoke' `
    -Method Post `
    -ContentType 'application/x-www-form-urlencoded' `
    -Body 'token=string'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/oidc/revoke",
    data={"token": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/oidc/revoke", {
  method: "POST",
  body: new URLSearchParams("token=string"),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/oidc/revoke")
    .form(&[("token", "string")])
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

## `GET /api/auth/oidc/clients`

Owner/operator list of Bookclerk-as-IdP clients (secrets never included).

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Registered clients (no secrets). Body: [`OidcClientsResponse`](schemas.md#oidcclientsresponse) (`application/json`)
  - `clients` *array of [`OidcClientView`](schemas.md#oidcclientview)*, **required** — Every registered client, without secrets.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is neither the operator nor an Owner (`forbidden`), or must enroll a second factor first (`mfa_enrollment_required`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The clients could not be read (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/clients' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/clients' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/auth/oidc/clients",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/clients", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/auth/oidc/clients")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Registered clients (no secrets).

```json
{
  "clients": [
    {
      "allowed_scopes": [
        "openid",
        "profile",
        "email"
      ],
      "client_id": "audiobookshelf",
      "confidential": true,
      "enabled": true,
      "has_secret": true,
      "issue_refresh_token": true,
      "name": "Audiobookshelf",
      "plugin_id": "audiobookshelf",
      "redirect_uris": [
        "https://abs.example.com/auth/openid/callback"
      ]
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/auth/oidc/clients`.",
  "required": [
    "clients"
  ],
  "properties": {
    "clients": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A Bookclerk-as-IdP client as operators see it; the secret appears only when freshly minted.",
        "required": [
          "client_id",
          "redirect_uris",
          "confidential",
          "issue_refresh_token",
          "allowed_scopes",
          "has_secret",
          "enabled"
        ],
        "properties": {
          "client_id": {
            "type": "string",
            "description": "Public client id used at the authorize and token endpoints.",
            "example": "audiobookshelf"
          },
          "name": {
            "type": [
              "string",
              "null"
            ],
            "description": "Operator-facing display name.",
            "example": "Audiobookshelf"
          },
          "redirect_uris": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Allowed OAuth redirect URIs.",
            "example": [
              "https://abs.example.com/auth/openid/callback"
            ]
          },
          "confidential": {
            "type": "boolean",
            "description": "Whether the client authenticates with a secret (otherwise it is a public PKCE client).",
            "example": true
          },
          "issue_refresh_token": {
            "type": "boolean",
            "description": "Whether token responses include a refresh token.",
            "example": true
          },
          "allowed_scopes": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Scopes the client may be granted.",
            "example": [
              "openid",
              "profile",
              "email"
            ]
          },
          "has_secret": {
            "type": "boolean",
            "description": "Same as `confidential`: a secret hash is stored.",
            "example": true
          },
          "enabled": {
            "type": "boolean",
            "description": "Whether the authorize and token endpoints accept this client.",
            "example": true
          },
          "plugin_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Plugin that provides this client, or `null` for an operator-created one.",
            "example": "audiobookshelf"
          },
          "client_secret": {
            "type": [
              "string",
              "null"
            ],
            "description": "The new secret, shown once after create, rotate-secret, or a PUT that makes a public client confidential; never stored in clear.",
            "example": "Qm9va2NsZXJrLXNlY3JldC1leGFtcGxlLXZhbHVl"
          }
        }
      },
      "description": "Every registered client, without secrets."
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `403`: The caller is neither the operator nor an Owner (`forbidden`), or must enroll a second factor first (`mfa_enrollment_required`).

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `500`: The clients could not be read (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection",
  "status": 500
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/auth/oidc/clients`

Create a client; confidential clients return the generated secret once.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`OidcClientWrite`](schemas.md#oidcclientwrite)

- `client_id` *string or null* — Public client_id (create only; path wins on update).
- `name` *string or null* — Operator-facing display name.
- `redirect_uris` *array of string* — Allowed OAuth redirect URIs.
- `confidential` *boolean* — When true, mint a client secret (confidential). When false, public PKCE.
- `issue_refresh_token` *boolean* — When true, token responses include a refresh token.
- `allowed_scopes` *array of string* — Scopes this client may request (`openid`, `profile`, `email`).
- `enabled` *boolean* — When true, authorize and token endpoints accept this client.
- `current_password` *string or null* — Required for a non-elevated Owner whose portal session is older than 15 minutes.

### Responses

- `200` — The client; a confidential client's secret appears once. Body: [`OidcClientView`](schemas.md#oidcclientview) (`application/json`)
  - `client_id` *string*, **required** — Public client id used at the authorize and token endpoints.
  - `name` *string or null* — Operator-facing display name.
  - `redirect_uris` *array of string*, **required** — Allowed OAuth redirect URIs.
  - `confidential` *boolean*, **required** — Whether the client authenticates with a secret (otherwise it is a public PKCE client).
  - `issue_refresh_token` *boolean*, **required** — Whether token responses include a refresh token.
  - `allowed_scopes` *array of string*, **required** — Scopes the client may be granted.
  - `has_secret` *boolean*, **required** — Same as `confidential`: a secret hash is stored.
  - `enabled` *boolean*, **required** — Whether the authorize and token endpoints accept this client.
  - `plugin_id` *string or null* — Plugin that provides this client, or `null` for an operator-created one.
  - `client_secret` *string or null* — The new secret, shown once after create, rotate-secret, or a PUT that makes a public client confidential; never stored in clear.
- `400` — The client id or a redirect URI is invalid (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `401` — The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — The caller is neither the operator nor an Owner (the standard error body before the handler, `oidc_client` from it), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `409` — A client with this id already exists (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The client could not be read or stored, or an Owner's current password could not be read or checked (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/clients' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/clients' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/oidc/clients",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/clients", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/oidc/clients")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The client; a confidential client's secret appears once.

```json
{
  "client_id": "audiobookshelf",
  "name": "Audiobookshelf",
  "redirect_uris": [
    "https://abs.example.com/auth/openid/callback"
  ],
  "confidential": true,
  "issue_refresh_token": true,
  "allowed_scopes": [
    "openid",
    "profile",
    "email"
  ],
  "has_secret": true,
  "enabled": true,
  "plugin_id": "audiobookshelf",
  "client_secret": "Qm9va2NsZXJrLXNlY3JldC1leGFtcGxlLXZhbHVl"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A Bookclerk-as-IdP client as operators see it; the secret appears only when freshly minted.",
  "required": [
    "client_id",
    "redirect_uris",
    "confidential",
    "issue_refresh_token",
    "allowed_scopes",
    "has_secret",
    "enabled"
  ],
  "properties": {
    "client_id": {
      "type": "string",
      "description": "Public client id used at the authorize and token endpoints.",
      "example": "audiobookshelf"
    },
    "name": {
      "type": [
        "string",
        "null"
      ],
      "description": "Operator-facing display name.",
      "example": "Audiobookshelf"
    },
    "redirect_uris": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Allowed OAuth redirect URIs.",
      "example": [
        "https://abs.example.com/auth/openid/callback"
      ]
    },
    "confidential": {
      "type": "boolean",
      "description": "Whether the client authenticates with a secret (otherwise it is a public PKCE client).",
      "example": true
    },
    "issue_refresh_token": {
      "type": "boolean",
      "description": "Whether token responses include a refresh token.",
      "example": true
    },
    "allowed_scopes": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Scopes the client may be granted.",
      "example": [
        "openid",
        "profile",
        "email"
      ]
    },
    "has_secret": {
      "type": "boolean",
      "description": "Same as `confidential`: a secret hash is stored.",
      "example": true
    },
    "enabled": {
      "type": "boolean",
      "description": "Whether the authorize and token endpoints accept this client.",
      "example": true
    },
    "plugin_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Plugin that provides this client, or `null` for an operator-created one.",
      "example": "audiobookshelf"
    },
    "client_secret": {
      "type": [
        "string",
        "null"
      ],
      "description": "The new secret, shown once after create, rotate-secret, or a PUT that makes a public client confidential; never stored in clear.",
      "example": "Qm9va2NsZXJrLXNlY3JldC1leGFtcGxlLXZhbHVl"
    }
  }
}
```

Status `400`: The client id or a redirect URI is invalid (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "at least one redirect URI is required",
  "status": 400
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `401`: The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`oidc_client`).

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `403`: The caller is neither the operator nor an Owner (the standard error body before the handler, `oidc_client` from it), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "oidc_client",
  "message": "forbidden",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `409`: A client with this id already exists (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "a client with this id already exists",
  "status": 409
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `500`: The client could not be read or stored, or an Owner's current password could not be read or checked (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "the current password could not be checked",
  "status": 500
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/auth/oidc/clients/{client_id}`

Update redirects, name, scopes, and public/confidential mode; switching a public client to confidential mints its secret, shown once.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `client_id` (path) *string*, **required** — Client id.

### Request body

`application/json`: [`OidcClientWrite`](schemas.md#oidcclientwrite)

- `client_id` *string or null* — Public client_id (create only; path wins on update).
- `name` *string or null* — Operator-facing display name.
- `redirect_uris` *array of string* — Allowed OAuth redirect URIs.
- `confidential` *boolean* — When true, mint a client secret (confidential). When false, public PKCE.
- `issue_refresh_token` *boolean* — When true, token responses include a refresh token.
- `allowed_scopes` *array of string* — Scopes this client may request (`openid`, `profile`, `email`).
- `enabled` *boolean* — When true, authorize and token endpoints accept this client.
- `current_password` *string or null* — Required for a non-elevated Owner whose portal session is older than 15 minutes.

### Responses

- `200` — The client after the change; when the update made a public client confidential, its new secret appears once. Body: [`OidcClientView`](schemas.md#oidcclientview) (`application/json`)
  - `client_id` *string*, **required** — Public client id used at the authorize and token endpoints.
  - `name` *string or null* — Operator-facing display name.
  - `redirect_uris` *array of string*, **required** — Allowed OAuth redirect URIs.
  - `confidential` *boolean*, **required** — Whether the client authenticates with a secret (otherwise it is a public PKCE client).
  - `issue_refresh_token` *boolean*, **required** — Whether token responses include a refresh token.
  - `allowed_scopes` *array of string*, **required** — Scopes the client may be granted.
  - `has_secret` *boolean*, **required** — Same as `confidential`: a secret hash is stored.
  - `enabled` *boolean*, **required** — Whether the authorize and token endpoints accept this client.
  - `plugin_id` *string or null* — Plugin that provides this client, or `null` for an operator-created one.
  - `client_secret` *string or null* — The new secret, shown once after create, rotate-secret, or a PUT that makes a public client confidential; never stored in clear.
- `400` — The client id or a redirect URI is invalid (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `401` — The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — The caller is neither the operator nor an Owner (the standard error body before the handler, `oidc_client` from it), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such client (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The client could not be read or stored, or an Owner's current password could not be read or checked (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The client after the change; when the update made a public client confidential, its new secret appears once.

```json
{
  "client_id": "audiobookshelf",
  "name": "Audiobookshelf",
  "redirect_uris": [
    "https://abs.example.com/auth/openid/callback"
  ],
  "confidential": true,
  "issue_refresh_token": true,
  "allowed_scopes": [
    "openid",
    "profile",
    "email"
  ],
  "has_secret": true,
  "enabled": true,
  "plugin_id": "audiobookshelf",
  "client_secret": "Qm9va2NsZXJrLXNlY3JldC1leGFtcGxlLXZhbHVl"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A Bookclerk-as-IdP client as operators see it; the secret appears only when freshly minted.",
  "required": [
    "client_id",
    "redirect_uris",
    "confidential",
    "issue_refresh_token",
    "allowed_scopes",
    "has_secret",
    "enabled"
  ],
  "properties": {
    "client_id": {
      "type": "string",
      "description": "Public client id used at the authorize and token endpoints.",
      "example": "audiobookshelf"
    },
    "name": {
      "type": [
        "string",
        "null"
      ],
      "description": "Operator-facing display name.",
      "example": "Audiobookshelf"
    },
    "redirect_uris": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Allowed OAuth redirect URIs.",
      "example": [
        "https://abs.example.com/auth/openid/callback"
      ]
    },
    "confidential": {
      "type": "boolean",
      "description": "Whether the client authenticates with a secret (otherwise it is a public PKCE client).",
      "example": true
    },
    "issue_refresh_token": {
      "type": "boolean",
      "description": "Whether token responses include a refresh token.",
      "example": true
    },
    "allowed_scopes": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Scopes the client may be granted.",
      "example": [
        "openid",
        "profile",
        "email"
      ]
    },
    "has_secret": {
      "type": "boolean",
      "description": "Same as `confidential`: a secret hash is stored.",
      "example": true
    },
    "enabled": {
      "type": "boolean",
      "description": "Whether the authorize and token endpoints accept this client.",
      "example": true
    },
    "plugin_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Plugin that provides this client, or `null` for an operator-created one.",
      "example": "audiobookshelf"
    },
    "client_secret": {
      "type": [
        "string",
        "null"
      ],
      "description": "The new secret, shown once after create, rotate-secret, or a PUT that makes a public client confidential; never stored in clear.",
      "example": "Qm9va2NsZXJrLXNlY3JldC1leGFtcGxlLXZhbHVl"
    }
  }
}
```

Status `400`: The client id or a redirect URI is invalid (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "client id may only contain letters, digits, '.', '_', '-', and '~'",
  "status": 400
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `401`: The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`oidc_client`).

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `403`: The caller is neither the operator nor an Owner (the standard error body before the handler, `oidc_client` from it), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "oidc_client",
  "message": "forbidden",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `404`: No such client (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "unknown client",
  "status": 404
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `500`: The client could not be read or stored, or an Owner's current password could not be read or checked (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "the current password could not be checked",
  "status": 500
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `DELETE /api/auth/oidc/clients/{client_id}`

Delete a registered client.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `client_id` (path) *string*, **required** — Client id.

### Request body

Optional; only `current_password` is read, to confirm an Owner outside the recent sign-in window. Optional.

`application/json`: [`OidcClientWrite`](schemas.md#oidcclientwrite) or null

- `client_id` *string or null* — Public client_id (create only; path wins on update).
- `name` *string or null* — Operator-facing display name.
- `redirect_uris` *array of string* — Allowed OAuth redirect URIs.
- `confidential` *boolean* — When true, mint a client secret (confidential). When false, public PKCE.
- `issue_refresh_token` *boolean* — When true, token responses include a refresh token.
- `allowed_scopes` *array of string* — Scopes this client may request (`openid`, `profile`, `email`).
- `enabled` *boolean* — When true, authorize and token endpoints accept this client.
- `current_password` *string or null* — Required for a non-elevated Owner whose portal session is older than 15 minutes.

### Responses

- `204` — The client was deleted. No body.
- `401` — The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — The caller is neither the operator nor an Owner (the standard error body before the handler, `oidc_client` from it), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such client (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `409` — The client is provided by a plugin; disable it instead (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The client could not be read or removed, or an Owner's current password could not be read or checked (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The client was deleted. No body.

Status `401`: The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`oidc_client`).

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `403`: The caller is neither the operator nor an Owner (the standard error body before the handler, `oidc_client` from it), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "oidc_client",
  "message": "forbidden",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `404`: No such client (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "unknown client",
  "status": 404
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `409`: The client is provided by a plugin; disable it instead (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "plugin-provided clients cannot be deleted; disable them instead",
  "status": 409
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `500`: The client could not be read or removed, or an Owner's current password could not be read or checked (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "the current password could not be checked",
  "status": 500
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/auth/oidc/clients/{client_id}/rotate-secret`

Mint a new secret for a confidential client; plaintext is returned once.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `client_id` (path) *string*, **required** — Client id.

### Request body

Optional.

`application/json`: [`RotateSecretBody`](schemas.md#rotatesecretbody) or null

- `current_password` *string or null* — Required for a non-elevated Owner whose portal session is older than 15 minutes.

### Responses

- `200` — The client with its new secret (shown once). Body: [`OidcClientView`](schemas.md#oidcclientview) (`application/json`)
  - `client_id` *string*, **required** — Public client id used at the authorize and token endpoints.
  - `name` *string or null* — Operator-facing display name.
  - `redirect_uris` *array of string*, **required** — Allowed OAuth redirect URIs.
  - `confidential` *boolean*, **required** — Whether the client authenticates with a secret (otherwise it is a public PKCE client).
  - `issue_refresh_token` *boolean*, **required** — Whether token responses include a refresh token.
  - `allowed_scopes` *array of string*, **required** — Scopes the client may be granted.
  - `has_secret` *boolean*, **required** — Same as `confidential`: a secret hash is stored.
  - `enabled` *boolean*, **required** — Whether the authorize and token endpoints accept this client.
  - `plugin_id` *string or null* — Plugin that provides this client, or `null` for an operator-created one.
  - `client_secret` *string or null* — The new secret, shown once after create, rotate-secret, or a PUT that makes a public client confidential; never stored in clear.
- `401` — The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — The caller is neither the operator nor an Owner (the standard error body before the handler, `oidc_client` from it), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such client (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The secret could not be stored, or an Owner's current password could not be read or checked (`oidc_client`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>/rotate-secret' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>/rotate-secret' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>/rotate-secret",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>/rotate-secret", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/oidc/clients/<client_id>/rotate-secret")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The client with its new secret (shown once).

```json
{
  "client_id": "audiobookshelf",
  "name": "Audiobookshelf",
  "redirect_uris": [
    "https://abs.example.com/auth/openid/callback"
  ],
  "confidential": true,
  "issue_refresh_token": true,
  "allowed_scopes": [
    "openid",
    "profile",
    "email"
  ],
  "has_secret": true,
  "enabled": true,
  "plugin_id": "audiobookshelf",
  "client_secret": "Qm9va2NsZXJrLXNlY3JldC1leGFtcGxlLXZhbHVl"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A Bookclerk-as-IdP client as operators see it; the secret appears only when freshly minted.",
  "required": [
    "client_id",
    "redirect_uris",
    "confidential",
    "issue_refresh_token",
    "allowed_scopes",
    "has_secret",
    "enabled"
  ],
  "properties": {
    "client_id": {
      "type": "string",
      "description": "Public client id used at the authorize and token endpoints.",
      "example": "audiobookshelf"
    },
    "name": {
      "type": [
        "string",
        "null"
      ],
      "description": "Operator-facing display name.",
      "example": "Audiobookshelf"
    },
    "redirect_uris": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Allowed OAuth redirect URIs.",
      "example": [
        "https://abs.example.com/auth/openid/callback"
      ]
    },
    "confidential": {
      "type": "boolean",
      "description": "Whether the client authenticates with a secret (otherwise it is a public PKCE client).",
      "example": true
    },
    "issue_refresh_token": {
      "type": "boolean",
      "description": "Whether token responses include a refresh token.",
      "example": true
    },
    "allowed_scopes": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Scopes the client may be granted.",
      "example": [
        "openid",
        "profile",
        "email"
      ]
    },
    "has_secret": {
      "type": "boolean",
      "description": "Same as `confidential`: a secret hash is stored.",
      "example": true
    },
    "enabled": {
      "type": "boolean",
      "description": "Whether the authorize and token endpoints accept this client.",
      "example": true
    },
    "plugin_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Plugin that provides this client, or `null` for an operator-created one.",
      "example": "audiobookshelf"
    },
    "client_secret": {
      "type": [
        "string",
        "null"
      ],
      "description": "The new secret, shown once after create, rotate-secret, or a PUT that makes a public client confidential; never stored in clear.",
      "example": "Qm9va2NsZXJrLXNlY3JldC1leGFtcGxlLXZhbHVl"
    }
  }
}
```

Status `401`: The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`oidc_client`).

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `403`: The caller is neither the operator nor an Owner (the standard error body before the handler, `oidc_client` from it), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "oidc_client",
  "message": "forbidden",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `404`: No such client (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "unknown client",
  "status": 404
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `500`: The secret could not be stored, or an Owner's current password could not be read or checked (`oidc_client`).

```json
{
  "error": "oidc_client",
  "message": "the current password could not be checked",
  "status": 500
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.
