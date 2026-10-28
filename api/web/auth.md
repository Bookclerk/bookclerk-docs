# Auth

Sign-in, sessions, passkeys, TOTP and the operator's elevation.

Back to the [web API reference](README.md).

## `POST /api/auth/login`

Validates the operator token (throttled, constant-time) and issues a hashed session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`LoginRequest`](schemas.md#loginrequest)

- `token` *string*, **required** — Operator token from the client; compared in constant time and never persisted.

### Responses

- `200` — Signed in; the response sets the operator session cookie. Body: any JSON (`application/json`)
- `401` — The token is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — An Owner exists, so operator sign-in is off (`operator_login_disabled`); sign in as a user. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `429` — Too many attempts; retry after `Retry-After` seconds. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/login' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"token": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/login' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"token": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/login",
    json={"token": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/login", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"token": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/login")
    .json(&serde_json::json!({"token": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in; the response sets the operator session cookie. The body is JSON the schemas do not describe yet.

Status `401`: The token is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: An Owner exists, so operator sign-in is off (`operator_login_disabled`); sign in as a user.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `429`: Too many attempts; retry after `Retry-After` seconds.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/signin`

Public sign-in picker: operator-token availability, SSO IdPs, integration logins.

### Errors

Returns 500 when the owner-count query fails.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — The sign-in methods this library offers. Body: any JSON (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/signin'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/signin'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/api/auth/signin")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/signin");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/api/auth/signin").await?;
println!("{} {}", response.status(), response.text().await?);
```

## `POST /api/auth/tray-handoff/prepare`

Mint a single-use tray Open Bookclerk ticket (Bearer, direct loopback only).

Returns a cryptographically random code and its lifetime. The durable
operator token stays in `Authorization`. Each call replaces any unused
ticket. The code is registered for log redaction (including percent-encoded
form). Lifetime comes from live `[daemon.auth] tray_handoff_ttl_secs`
(default 180, clamped to 30..=900).

### Errors

Returns 403 when the request is not a direct loopback call, 401 when Bearer
does not match, or 429 when this client is locked out.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken`

### Responses

- `200` — A single-use tray ticket. Body: any JSON (`application/json`)
- `204` — Operator authentication is off, so no ticket is needed. No body.
- `401` — The Bearer token is missing or wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The request did not come straight from loopback. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `429` — Too many failed attempts from this client; retry after `Retry-After` seconds. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/tray-handoff/prepare' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/tray-handoff/prepare' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/tray-handoff/prepare",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/tray-handoff/prepare", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/tray-handoff/prepare")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A single-use tray ticket. The body is JSON the schemas do not describe yet.

Status `204`: Operator authentication is off, so no ticket is needed. No body.

Status `401`: The Bearer token is missing or wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The request did not come straight from loopback.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `429`: Too many failed attempts from this client; retry after `Retry-After` seconds.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/tray-handoff`

Consume the pending tray ticket and set a localhost operator session cookie.

Direct loopback only (TCP peer, localhost `Host`, no forwarded headers).
Requires the one-time `code` from prepare; `?token=` is ignored and cannot
mint a session. Wrong or missing codes do not consume the slot.

### Errors

Returns 403 when the request is not a direct loopback call or the code does
not match a still-valid ticket.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Parameters

- `code` (query) *string* — Presented handoff code; missing or empty fails closed without consuming the slot.

### Responses

- `307` — Redirects into the app with a localhost operator session. No body.
- `403` — The ticket is missing, used or not from loopback. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/tray-handoff'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/tray-handoff'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/api/auth/tray-handoff")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/tray-handoff");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/api/auth/tray-handoff").await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `307`: Redirects into the app with a localhost operator session. No body.

Status `403`: The ticket is missing, used or not from loopback.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/me`

Resolves the caller and returns `/me` JSON; unauthenticated callers get 401.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Who the caller is (operator, portal user, or impersonation). Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/me' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/me' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/auth/me",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/me", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/auth/me")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Who the caller is (operator, portal user, or impersonation). The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/elevate`

Owner portal session + password re-auth → short-lived elevated operator cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`ElevateRequest`](schemas.md#elevaterequest)

- `password` *string*, **required** — Owner account password (re-authentication).

### Responses

- `200` — Elevated; the response sets a short-lived operator cookie. Body: any JSON (`application/json`)
- `400` — Operator auth is off, or the password is empty. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `409` — The caller is already an operator. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/elevate' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"password": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/elevate' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"password": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/elevate",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"password": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/elevate", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"password": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/elevate")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"password": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Elevated; the response sets a short-lived operator cookie. The body is JSON the schemas do not describe yet.

Status `400`: Operator auth is off, or the password is empty.

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

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `409`: The caller is already an operator.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `DELETE /api/auth/elevate`

End elevation by revoking the elevated operator session cookie.

**Access:** The operator (token or session) only.

**Credentials:** `operatorSession`

### Responses

- `200` — Elevation ended. Body: any JSON (`application/json`)
- `400` — The session is not an elevated one. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/elevate' \
    -X DELETE \
    -b 'bookclerk_operator_session=<operatorSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_operator_session', '<operatorSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/elevate' `
    -Method Delete `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/auth/elevate",
    headers={"Cookie": "bookclerk_operator_session=<operatorSession>", "Origin": "http://127.0.0.1:8787"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/elevate", {
  method: "DELETE",
  headers: { Cookie: "bookclerk_operator_session=<operatorSession>", Origin: "http://127.0.0.1:8787" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/auth/elevate")
    .header("Cookie", "bookclerk_operator_session=<operatorSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Elevation ended. The body is JSON the schemas do not describe yet.

Status `400`: The session is not an elevated one.

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

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/bootstrap`

Operator-only bootstrap of the first Owner when none exist.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`BootstrapRequest`](schemas.md#bootstraprequest)

- `display_name` *string or null* — Optional display name; falls back to login or email.
- `login_name` *string or null* — Optional local login name.
- `email` *string or null* — Optional email.
- `password` *string or null* — Optional password; hashed before persist and never stored in plaintext.

### Responses

- `200` — The first Owner was created. Body: any JSON (`application/json`)
- `400` — The email is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `409` — An Owner already exists. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/bootstrap' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/bootstrap' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/bootstrap",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/bootstrap", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/bootstrap")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The first Owner was created. The body is JSON the schemas do not describe yet.

Status `400`: The email is invalid.

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

Status `403`: The caller may not do this.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `409`: An Owner already exists.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PUT /api/auth/password`

Set password for self (portal) or target user (operator/admin); revokes sessions.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`SetPasswordRequest`](schemas.md#setpasswordrequest)

- `password` *string*, **required** — New password; hashed before persist. Empty is rejected.
- `current_password` *string or null* — Required when the caller already has a password (not first-time invite setup).
- `user_id` *integer (int64) or null* — Target `users.id` when a provisioner sets another user's password.

### Responses

- `200` — The password was set and other sessions revoked. Body: any JSON (`application/json`)
- `400` — The password is too weak or the request is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — No such user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/password' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"password": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/password' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"password": "string"}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/auth/password",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"password": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/password", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"password": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/auth/password")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"password": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The password was set and other sessions revoked. The body is JSON the schemas do not describe yet.

Status `400`: The password is too weak or the request is invalid.

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

Status `403`: The caller may not do this.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: No such user.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/password`

Local password login → portal session cookie, or a TOTP challenge when enabled.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`PasswordLoginRequest`](schemas.md#passwordloginrequest)

- `login` *string*, **required** — Login name or email.
- `password` *string*, **required** — Password verified against the stored hash; mismatch fails closed and counts toward lockout.

### Responses

- `200` — Signed in (portal session cookie), or a TOTP challenge. Body: any JSON (`application/json`)
- `401` — The login or password is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The account is disabled (`account_disabled`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `429` — Too many attempts; retry after `Retry-After` seconds. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/password' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"login": "string", "password": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/password' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"login": "string", "password": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/password",
    json={"login": "string", "password": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/password", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"login": "string", "password": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/password")
    .json(&serde_json::json!({"login": "string", "password": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in (portal session cookie), or a TOTP challenge. The body is JSON the schemas do not describe yet.

Status `401`: The login or password is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The account is disabled (`account_disabled`).

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `429`: Too many attempts; retry after `Retry-After` seconds.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/impersonate`

Operator (or elevated) session starts impersonating a User.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`ImpersonateRequest`](schemas.md#impersonaterequest)

- `user_id` *integer (int64)*, **required** — Target `users.id` to impersonate.

### Responses

- `200` — The session now acts as the user. Body: any JSON (`application/json`)
- `400` — Operator auth is off. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — No such user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/impersonate' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"user_id": 0}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/impersonate' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"user_id": 0}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/impersonate",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"user_id": 0},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/impersonate", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"user_id": 0}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/impersonate")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"user_id": 0}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The session now acts as the user. The body is JSON the schemas do not describe yet.

Status `400`: Operator auth is off.

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

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: No such user.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `DELETE /api/auth/impersonate`

Clear impersonation on the current operator session.

**Access:** The operator (token or session) only.

**Credentials:** `operatorSession`

### Responses

- `200` — Impersonation ended. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/impersonate' \
    -X DELETE \
    -b 'bookclerk_operator_session=<operatorSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_operator_session', '<operatorSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/impersonate' `
    -Method Delete `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/auth/impersonate",
    headers={"Cookie": "bookclerk_operator_session=<operatorSession>", "Origin": "http://127.0.0.1:8787"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/impersonate", {
  method: "DELETE",
  headers: { Cookie: "bookclerk_operator_session=<operatorSession>", Origin: "http://127.0.0.1:8787" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/auth/impersonate")
    .header("Cookie", "bookclerk_operator_session=<operatorSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Impersonation ended. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/branch-consent`

`GET /api/auth/branch-consent?<authorize params>`: what the consent page
shows. 400 `invalid_request` for a malformed request (the page then
shows the problem and never redirects).

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Parameters

- `client_id` (query) *string*, **required** — Must be [`BRANCH_CLIENT_ID`].
- `redirect_uri` (query) *string*, **required** — Where the code goes: loopback or https.
- `response_type` (query) *string*, **required** — Must be `code`.
- `scope` (query) *string* — Must be [`BRANCH_SCOPE`].
- `state` (query) *string* — Echoed on the redirect.
- `code_challenge` (query) *string*, **required** — PKCE S256 challenge.
- `code_challenge_method` (query) *string* — Must be `S256`.
- `branch_name` (query) *string* — The name the branch asks to join as.
- `clerk_offer` (query) *string* — `1` when the joining install offers to join as a clerk ([`bookclerk_node_proto::CLERK_OFFER_PARAM`]).

### Responses

- `200` — What the consent page shows. Body: any JSON (`application/json`)
- `400` — The authorize parameters are invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller is not a library user (`user_required`; an operator session counts only while impersonating) or the account is inactive. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `503` — The library could not be read; try again. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/branch-consent?client_id=string&redirect_uri=string&response_type=string&code_challenge=string' \
    -b 'bookclerk_portal_session=<portalSession>'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/branch-consent?client_id=string&redirect_uri=string&response_type=string&code_challenge=string' `
    -WebSession $session
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/auth/branch-consent?client_id=string&redirect_uri=string&response_type=string&code_challenge=string",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/branch-consent?client_id=string&redirect_uri=string&response_type=string&code_challenge=string", {
  headers: { Cookie: "bookclerk_portal_session=<portalSession>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/auth/branch-consent?client_id=string&redirect_uri=string&response_type=string&code_challenge=string")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: What the consent page shows. The body is JSON the schemas do not describe yet.

Status `400`: The authorize parameters are invalid.

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

Status `403`: The caller is not a library user (`user_required`; an operator session counts only while impersonating) or the account is inactive.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `503`: The library could not be read; try again.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/branch-consent`

`POST /api/auth/branch-consent`: approve or deny. Answers `{redirect}`,
the URL to send the browser to: the branch's callback with a code (or
`error=access_denied`). Approving needs an active signed-in user, host
enrollment on, and a connection the transport policy allows.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Request body

`application/json`: [`BranchConsentBody`](schemas.md#branchconsentbody)

- `approve` *boolean*, **required** — True to let the branch join; false to deny.
- `connection` *string or null* — `branch` (default) or `clerk`.
- `step_up` *[`StepUp`](schemas.md#stepup) or null* — The fresh second factor a clerk needs.

### Responses

- `200` — `{redirect}` back to the branch. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller is not an active library user, or a clerk join lacks the Owner role or a passed step-up (`step_up_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `409` — This library's database cannot be shared with a clerk (`database_not_shareable`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `429` — Too many wrong step-up codes; wait a few minutes (`step_up_locked`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `503` — The library could not be read; try again. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/branch-consent' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"client_id": "string", "redirect_uri": "string", "response_type": "string", "code_challenge": "string", "approve": true}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/branch-consent' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"client_id": "string", "redirect_uri": "string", "response_type": "string", "code_challenge": "string", "approve": true}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/branch-consent",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"client_id": "string", "redirect_uri": "string", "response_type": "string", "code_challenge": "string", "approve": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/branch-consent", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"client_id": "string", "redirect_uri": "string", "response_type": "string", "code_challenge": "string", "approve": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/branch-consent")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"client_id": "string", "redirect_uri": "string", "response_type": "string", "code_challenge": "string", "approve": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: `{redirect}` back to the branch. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller is not an active library user, or a clerk join lacks the Owner role or a passed step-up (`step_up_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `409`: This library's database cannot be shared with a clerk (`database_not_shareable`).

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `429`: Too many wrong step-up codes; wait a few minutes (`step_up_locked`).

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `503`: The library could not be read; try again.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/branch-consent/step-up`

`POST /api/auth/branch-consent/step-up`: starts a passkey step-up for a
clerk consent (an Owner with a passkey). Answers the WebAuthn request
options with a `challenge_id` to send back in the consent's `step_up`.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Responses

- `200` — Passkey request options. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller is not an active library user, or not an Owner (`owner_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — The account has no passkey (`no_passkey`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `503` — The library could not be read, or the passkey check could not start. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/branch-consent/step-up' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/branch-consent/step-up' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/branch-consent/step-up",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/branch-consent/step-up", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/branch-consent/step-up")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Passkey request options. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller is not an active library user, or not an Owner (`owner_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: The account has no passkey (`no_passkey`).

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `503`: The library could not be read, or the passkey check could not start.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/logout`

Revokes hashed operator and portal sessions and clears both cookies.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Signed out; both session cookies are cleared. Body: any JSON (`application/json`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/logout' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/logout' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/logout",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/logout", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/logout")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed out; both session cookies are cleared. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/sessions`

List sessions for the current principal only (not every operator session).

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The caller's sessions. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — The user has no sign-in identity. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/sessions' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/sessions' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/auth/sessions",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/sessions", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/auth/sessions")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's sessions. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: The user has no sign-in identity.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `DELETE /api/auth/sessions/{id}`

Revoke a session by id. Operators may revoke any operator session;
portal users may only revoke their own.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — Session id.

### Responses

- `200` — The session was revoked. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — No such session. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/sessions/<id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/sessions/<id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/auth/sessions/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/sessions/<id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/auth/sessions/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The session was revoked. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: No such session.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PATCH /api/auth/profile`

Updates the caller's display name and/or email.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PatchProfileRequest`](schemas.md#patchprofilerequest)

- `display_name` *string or null* — Replacement display name; empty or whitespace clears the field.
- `email` *string or null* — Replacement email; empty or whitespace clears the field.
- `avatar_source` *string or null* — Picture choice (`auto`, `monogram`, `gravatar`, `upload`, or `sso:{id}`).

### Responses

- `200` — The profile after the change. Body: any JSON (`application/json`)
- `400` — A value is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — The caller's user no longer exists. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `409` — The email is used by another account (`email_in_use`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/profile' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/profile' `
    -Method Patch `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.patch(
    "http://127.0.0.1:8787/api/auth/profile",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/profile", {
  method: "PATCH",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .patch("http://127.0.0.1:8787/api/auth/profile")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The profile after the change. The body is JSON the schemas do not describe yet.

Status `400`: A value is invalid.

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

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: The caller's user no longer exists.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `409`: The email is used by another account (`email_in_use`).

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PUT /api/auth/profile/avatar`

Accepts a JPEG/PNG/WebP upload and stores it under `files_dir/avatars`.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`image/*`: [`BinaryBody`](schemas.md#binarybody)

### Responses

- `200` — The avatar was stored. Body: any JSON (`application/json`)
- `400` — The image is not JPEG, PNG or WebP. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `413` — The image is too large. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/profile/avatar' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: image/*' \
    --data-binary @file
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/profile/avatar' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'image/*' `
    -InFile 'file'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/auth/profile/avatar",
    headers={"Authorization": "Bearer <operatorToken>", "Content-Type": "image/*"},
    data=open("file", "rb"),
)
print(response.status_code, response.text)
```

```javascript
import { readFile } from "node:fs/promises";

const response = await fetch("http://127.0.0.1:8787/api/auth/profile/avatar", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "image/*" },
  body: await readFile("file"),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/auth/profile/avatar")
    .header("Authorization", "Bearer <operatorToken>")
    .header("Content-Type", "image/*")
    .body(std::fs::read("file")?)
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The avatar was stored. The body is JSON the schemas do not describe yet.

Status `400`: The image is not JPEG, PNG or WebP.

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

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `413`: The image is too large.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `DELETE /api/auth/profile/avatar`

Removes the caller's stored avatar, if any.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The avatar was removed. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/profile/avatar' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/profile/avatar' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/auth/profile/avatar",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/profile/avatar", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/auth/profile/avatar")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The avatar was removed. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/oidc/providers`

Lists enabled IdPs for the login picker (id and display name only).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — Enabled identity providers (id and name). Body: any JSON (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/providers'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/providers'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/api/auth/oidc/providers")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/providers");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/api/auth/oidc/providers").await?;
println!("{} {}", response.status(), response.text().await?);
```

## `GET /api/auth/oidc/identities`

Lists OIDC portal identities for the cookie-authenticated caller.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — The caller's linked identities. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/identities' \
    -b 'bookclerk_portal_session=<portalSession>'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/identities' `
    -WebSession $session
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/auth/oidc/identities",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/identities", {
  headers: { Cookie: "bookclerk_portal_session=<portalSession>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/auth/oidc/identities")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's linked identities. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/oidc/login`

Starts an OIDC login authorize redirect, honoring login throttle.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Parameters

- `provider` (query) *string* — IdP id to start; omitted when the caller has only one enabled provider.

### Responses

- `302` — Redirects to the identity provider. No body.
- `400` — The provider is unknown or disabled. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `429` — Too many attempts; retry after `Retry-After` seconds. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/login'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/login'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/api/auth/oidc/login")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/login");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/api/auth/oidc/login").await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `302`: Redirects to the identity provider. No body.

Status `400`: The provider is unknown or disabled.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `429`: Too many attempts; retry after `Retry-After` seconds.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/oidc/elevate`

Starts an Owner re-auth elevate flow against a linked IdP (`prompt=login`).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

- `provider` (query) *string* — IdP id to start; omitted when the caller has only one enabled provider.

### Responses

- `302` — Redirects to the identity provider for re-authentication. No body.
- `400` — The provider is unknown or disabled. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `429` — Too many attempts; retry after `Retry-After` seconds. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/elevate' \
    -b 'bookclerk_portal_session=<portalSession>'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/elevate' `
    -WebSession $session
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/auth/oidc/elevate",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/elevate", {
  headers: { Cookie: "bookclerk_portal_session=<portalSession>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/auth/oidc/elevate")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `302`: Redirects to the identity provider for re-authentication. No body.

Status `400`: The provider is unknown or disabled.

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

Status `403`: The caller may not do this.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `429`: Too many attempts; retry after `Retry-After` seconds.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/oidc/callback`

GET callback (most IdPs); finishes the code exchange.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Parameters

- `code` (query) *string* — Authorization code to exchange; required unless `error` is set.
- `state` (query) *string* — CSRF `state` that must match the `bookclerk_oidc_tx` cookie.
- `error` (query) *string* — IdP-reported deny/error; redirects the browser to `/?sso_error=denied`.
- `user` (query) *string* — Apple first-login name/email JSON (form_post only).

### Responses

- `302` — Redirects into the app, signed in. No body.
- `400` — The state or code is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/callback'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/callback'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/api/auth/oidc/callback")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/callback");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/api/auth/oidc/callback").await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `302`: Redirects into the app, signed in. No body.

Status `400`: The state or code is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/oidc/callback`

POST callback (Apple `form_post`); finishes the code exchange.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/x-www-form-urlencoded`: [`CallbackParams`](schemas.md#callbackparams)

- `code` *string or null* — Authorization code to exchange; required unless `error` is set.
- `state` *string or null* — CSRF `state` that must match the `bookclerk_oidc_tx` cookie.
- `error` *string or null* — IdP-reported deny/error; redirects the browser to `/?sso_error=denied`.
- `user` *string or null* — Apple first-login name/email JSON (form_post only).

### Responses

- `302` — Redirects into the app, signed in. No body.
- `400` — The state or code is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/callback' \
    -X POST
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/callback' `
    -Method Post
```

```python
import requests

response = requests.post("http://127.0.0.1:8787/api/auth/oidc/callback")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/callback", {
  method: "POST",
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/oidc/callback")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `302`: Redirects into the app, signed in. No body.

Status `400`: The state or code is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/oidc/config`

Operator GET: broker settings plus secret-presence flags (never plaintext).

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Broker settings, with secrets as presence flags. Body: [`OidcConfigResponse`](schemas.md#oidcconfigresponse) (`application/json`)
  - `enabled` *boolean*, **required** — Whether the broker accepts new authorize starts.
  - `allowed_email_domains` *array of string*, **required** — Broker-wide email domain allowlist applied before per-provider rules.
  - `public_origin` *string or null* — Configured `integrations.public_origin` (issuer / callback / invite base).
  - `issuer_url` *string*, **required** — Effective issuer origin (`public_origin`, else this request, else localhost).
  - `detected_origin` *string*, **required** — Origin detected from this request (loopback IPs rewritten to `localhost`).
  - `discovery_url` *string*, **required** — Absolute OpenID discovery URL (`/.well-known/openid-configuration`).
  - `authorization_endpoint` *string*, **required** — Absolute authorization endpoint (`/oidc/authorize`) for Bookclerk-as-IdP.
  - `token_endpoint` *string*, **required** — Absolute token endpoint (`/oidc/token`) for Bookclerk-as-IdP.
  - `userinfo_endpoint` *string*, **required** — Absolute userinfo endpoint (`/oidc/userinfo`) for Bookclerk-as-IdP.
  - `revocation_endpoint` *string*, **required** — Absolute revocation endpoint (`/oidc/revoke`) for Bookclerk-as-IdP.
  - `callback_url` *string*, **required** — Absolute `/api/auth/oidc/callback` derived from the effective issuer origin.
  - `providers` *array of [`OidcProviderView`](schemas.md#oidcproviderview)*, **required** — Configured IdPs with presence flags instead of secret material.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/config' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/config' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/auth/oidc/config",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/config", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/auth/oidc/config")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Broker settings, with secrets as presence flags.

```json
{
  "enabled": true,
  "allowed_email_domains": [
    "string"
  ],
  "public_origin": "string",
  "issuer_url": "string",
  "detected_origin": "string",
  "discovery_url": "string",
  "authorization_endpoint": "string",
  "token_endpoint": "string",
  "userinfo_endpoint": "string",
  "revocation_endpoint": "string",
  "callback_url": "string",
  "providers": [
    {
      "id": "string",
      "name": "string",
      "preset": "string",
      "issuer": "string",
      "client_id": "string",
      "scopes": [
        "string"
      ],
      "provision": "string",
      "default_role": "string",
      "role_claim": "string",
      "role_map": {
        "key": "string"
      },
      "link_by_email": true,
      "allowed_email_domains": [
        "string"
      ],
      "allowed_emails": [
        "string"
      ],
      "allowed_subjects": [
        "string"
      ],
      "apple_team_id": "string",
      "apple_key_id": "string",
      "has_client_secret": true,
      "has_apple_private_key": true,
      "secret_source": "string"
    }
  ]
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PUT /api/auth/oidc/config`

Operator PUT: writes next-generation sealed secrets, then publishes config.toml.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`OidcConfigPut`](schemas.md#oidcconfigput)

- `enabled` *boolean*, **required** — Whether the broker should accept authorize starts after this write.
- `allowed_email_domains` *array of string* — Replacement broker-wide email domain allowlist.
- `providers` *array of [`OidcProviderPut`](schemas.md#oidcproviderput)* — Full replacement list of IdPs (omitted secrets keep the previous generation).
- `public_origin` *string or null* — `integrations.public_origin`. Omit to leave unchanged; empty string clears.
- `current_password` *string or null* — Required for a non-elevated Owner whose portal session is older than 15 minutes.

### Responses

- `200` — The settings after the change. Body: [`OidcConfigResponse`](schemas.md#oidcconfigresponse) (`application/json`)
  - `enabled` *boolean*, **required** — Whether the broker accepts new authorize starts.
  - `allowed_email_domains` *array of string*, **required** — Broker-wide email domain allowlist applied before per-provider rules.
  - `public_origin` *string or null* — Configured `integrations.public_origin` (issuer / callback / invite base).
  - `issuer_url` *string*, **required** — Effective issuer origin (`public_origin`, else this request, else localhost).
  - `detected_origin` *string*, **required** — Origin detected from this request (loopback IPs rewritten to `localhost`).
  - `discovery_url` *string*, **required** — Absolute OpenID discovery URL (`/.well-known/openid-configuration`).
  - `authorization_endpoint` *string*, **required** — Absolute authorization endpoint (`/oidc/authorize`) for Bookclerk-as-IdP.
  - `token_endpoint` *string*, **required** — Absolute token endpoint (`/oidc/token`) for Bookclerk-as-IdP.
  - `userinfo_endpoint` *string*, **required** — Absolute userinfo endpoint (`/oidc/userinfo`) for Bookclerk-as-IdP.
  - `revocation_endpoint` *string*, **required** — Absolute revocation endpoint (`/oidc/revoke`) for Bookclerk-as-IdP.
  - `callback_url` *string*, **required** — Absolute `/api/auth/oidc/callback` derived from the effective issuer origin.
  - `providers` *array of [`OidcProviderView`](schemas.md#oidcproviderview)*, **required** — Configured IdPs with presence flags instead of secret material.
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/oidc/config' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"enabled": true}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/oidc/config' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"enabled": true}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/auth/oidc/config",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"enabled": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/oidc/config", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"enabled": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/auth/oidc/config")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"enabled": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings after the change.

```json
{
  "enabled": true,
  "allowed_email_domains": [
    "string"
  ],
  "public_origin": "string",
  "issuer_url": "string",
  "detected_origin": "string",
  "discovery_url": "string",
  "authorization_endpoint": "string",
  "token_endpoint": "string",
  "userinfo_endpoint": "string",
  "revocation_endpoint": "string",
  "callback_url": "string",
  "providers": [
    {
      "id": "string",
      "name": "string",
      "preset": "string",
      "issuer": "string",
      "client_id": "string",
      "scopes": [
        "string"
      ],
      "provision": "string",
      "default_role": "string",
      "role_claim": "string",
      "role_map": {
        "key": "string"
      },
      "link_by_email": true,
      "allowed_email_domains": [
        "string"
      ],
      "allowed_emails": [
        "string"
      ],
      "allowed_subjects": [
        "string"
      ],
      "apple_team_id": "string",
      "apple_key_id": "string",
      "has_client_secret": true,
      "has_apple_private_key": true,
      "secret_source": "string"
    }
  ]
}
```

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

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/passkeys`

Lists this user's stored passkeys as `{ id, credential_id }` rows.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — The caller's passkeys. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys' \
    -b 'bookclerk_portal_session=<portalSession>'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys' `
    -WebSession $session
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/auth/passkeys",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys", {
  headers: { Cookie: "bookclerk_portal_session=<portalSession>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/auth/passkeys")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's passkeys. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/passkeys/register/begin`

Starts passkey registration (skips reauth only for first local-only setup).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`PasskeyReauthBody`](schemas.md#passkeyreauthbody)

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

### Responses

- `200` — WebAuthn creation options. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/register/begin' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/register/begin' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/passkeys/register/begin",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/register/begin", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/passkeys/register/begin")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: WebAuthn creation options. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/passkeys/register/finish`

Completes registration, stores the passkey, and revokes elevated operator sessions.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`CeremonyFinish`](schemas.md#ceremonyfinish)

- `challenge_id` *string*, **required** — Server-issued challenge id (consumed once; 5-minute TTL).
- `credential` *any JSON*, **required** — Browser `PublicKeyCredential` JSON for `finish_*`.
- `name` *string or null* — Label stored with the credential (`Passkey` when empty).

### Responses

- `200` — The passkey was stored. Body: any JSON (`application/json`)
- `400` — The attestation is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `409` — The passkey is already registered. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/register/finish' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"challenge_id": "string"}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/register/finish' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"challenge_id": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/passkeys/register/finish",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"challenge_id": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/register/finish", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"challenge_id": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/passkeys/register/finish")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"challenge_id": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The passkey was stored. The body is JSON the schemas do not describe yet.

Status `400`: The attestation is invalid.

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

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `409`: The passkey is already registered.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `DELETE /api/auth/passkeys/{id}`

Deletes one passkey after recent reauth; 404 when the id is not this user's.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — Passkey id.

### Request body

`application/json`: [`PasskeyReauthBody`](schemas.md#passkeyreauthbody)

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

### Responses

- `200` — The passkey was deleted. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — No such passkey for this user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/<id>' \
    -X DELETE \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/<id>' `
    -Method Delete `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/auth/passkeys/<id>",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/<id>", {
  method: "DELETE",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/auth/passkeys/<id>")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The passkey was deleted. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: No such passkey for this user.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/passkeys/login/begin`

Rate-limits and starts passkey login; records a failure on any error.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`LoginBegin`](schemas.md#loginbegin)

- `login` *string*, **required** — Login name or email looked up before issuing an authentication challenge.

### Responses

- `200` — WebAuthn request options. Body: any JSON (`application/json`)
- `400` — The request is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — No user has this login name or email. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The user is disabled. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — The user has no usable passkeys. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `429` — Too many attempts; try later. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/login/begin' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"login": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/login/begin' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"login": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/passkeys/login/begin",
    json={"login": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/login/begin", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"login": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/passkeys/login/begin")
    .json(&serde_json::json!({"login": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: WebAuthn request options. The body is JSON the schemas do not describe yet.

Status `400`: The request is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: No user has this login name or email.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The user is disabled.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: The user has no usable passkeys.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `429`: Too many attempts; try later.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/passkeys/login/finish`

Verifies the assertion, updates the credential counter, and issues a portal session.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`CeremonyFinish`](schemas.md#ceremonyfinish)

- `challenge_id` *string*, **required** — Server-issued challenge id (consumed once; 5-minute TTL).
- `credential` *any JSON*, **required** — Browser `PublicKeyCredential` JSON for `finish_*`.
- `name` *string or null* — Label stored with the credential (`Passkey` when empty).

### Responses

- `200` — Signed in; the response sets the portal session cookie. Body: any JSON (`application/json`)
- `400` — The challenge is unknown or expired, or the credential is malformed. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The assertion is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The user is disabled. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/login/finish' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"challenge_id": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/login/finish' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"challenge_id": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/passkeys/login/finish",
    json={"challenge_id": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/login/finish", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"challenge_id": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/passkeys/login/finish")
    .json(&serde_json::json!({"challenge_id": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in; the response sets the portal session cookie. The body is JSON the schemas do not describe yet.

Status `400`: The challenge is unknown or expired, or the credential is malformed.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: The assertion is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The user is disabled.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/passkeys/elevate/begin`

Starts Owner elevation; 403 unless the signed-in user is an Owner with passkeys.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — WebAuthn request options. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — The caller has no passkeys. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/elevate/begin' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/elevate/begin' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/passkeys/elevate/begin",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/elevate/begin", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/passkeys/elevate/begin")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: WebAuthn request options. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: The caller has no passkeys.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/passkeys/elevate/finish`

Completes Owner elevation only when the assertion is user-verified.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`CeremonyFinish`](schemas.md#ceremonyfinish)

- `challenge_id` *string*, **required** — Server-issued challenge id (consumed once; 5-minute TTL).
- `credential` *any JSON*, **required** — Browser `PublicKeyCredential` JSON for `finish_*`.
- `name` *string or null* — Label stored with the credential (`Passkey` when empty).

### Responses

- `200` — Elevated; the response sets a short-lived operator cookie. Body: any JSON (`application/json`)
- `400` — The challenge is unknown or expired, or the credential is malformed. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/elevate/finish' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"challenge_id": "string"}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/elevate/finish' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"challenge_id": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/passkeys/elevate/finish",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"challenge_id": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/elevate/finish", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"challenge_id": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/passkeys/elevate/finish")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"challenge_id": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Elevated; the response sets a short-lived operator cookie. The body is JSON the schemas do not describe yet.

Status `400`: The challenge is unknown or expired, or the credential is malformed.

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

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/totp/login`

Completes password+TOTP login and issues a portal session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`LoginVerify`](schemas.md#loginverify)

- `challenge_id` *string*, **required** — Server-issued challenge id (5-minute TTL, single use).
- `code` *string*, **required** — Six-digit authenticator code.

### Responses

- `200` — Signed in; the response sets the portal session cookie. Body: any JSON (`application/json`)
- `401` — The challenge or code is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `429` — Too many attempts; try later. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/totp/login' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"challenge_id": "string", "code": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/totp/login' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"challenge_id": "string", "code": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/totp/login",
    json={"challenge_id": "string", "code": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/totp/login", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"challenge_id": "string", "code": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/totp/login")
    .json(&serde_json::json!({"challenge_id": "string", "code": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in; the response sets the portal session cookie. The body is JSON the schemas do not describe yet.

Status `401`: The challenge or code is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `429`: Too many attempts; try later.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/totp/enroll/begin`

Starts TOTP enrollment: new pending secret, otpauth URL, and QR SVG.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`TotpReauthBody`](schemas.md#totpreauthbody)

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

### Responses

- `200` — A pending secret, its otpauth URL and a QR code SVG. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `409` — TOTP is already on. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/totp/enroll/begin' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/totp/enroll/begin' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/totp/enroll/begin",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/totp/enroll/begin", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/totp/enroll/begin")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A pending secret, its otpauth URL and a QR code SVG. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `409`: TOTP is already on.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/auth/totp/enroll/finish`

Confirms the pending secret with a valid code and sets `users.totp_enabled`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`EnrollFinish`](schemas.md#enrollfinish)

- `code` *string*, **required** — Six-digit code from the authenticator app.

### Responses

- `200` — TOTP is on. Body: any JSON (`application/json`)
- `400` — No enrollment is pending; start one first. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in, or the code is wrong (`invalid_code`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `409` — TOTP is already on. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/totp/enroll/finish' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"code": "string"}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/totp/enroll/finish' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"code": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/totp/enroll/finish",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"code": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/totp/enroll/finish", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"code": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/totp/enroll/finish")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"code": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: TOTP is on. The body is JSON the schemas do not describe yet.

Status `400`: No enrollment is pending; start one first.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: The caller is not signed in, or the code is wrong (`invalid_code`).

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `409`: TOTP is already on.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/totp`

GET `/api/auth/totp` — whether TOTP is confirmed for this user.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — Whether TOTP is on for the caller. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/totp' \
    -b 'bookclerk_portal_session=<portalSession>'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/totp' `
    -WebSession $session
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/auth/totp",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/totp", {
  headers: { Cookie: "bookclerk_portal_session=<portalSession>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/auth/totp")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Whether TOTP is on for the caller. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `DELETE /api/auth/totp`

Disables TOTP after recent reauth and deletes sealed secrets.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`TotpReauthBody`](schemas.md#totpreauthbody)

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

### Responses

- `200` — TOTP is off. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/totp' \
    -X DELETE \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/totp' `
    -Method Delete `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/auth/totp",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/totp", {
  method: "DELETE",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/auth/totp")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: TOTP is off. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/auth/mfa-policy`

Current MFA policy (`require_second_factor`).

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Whether a second factor is required. Body: any JSON (`application/json`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/mfa-policy' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/mfa-policy' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/auth/mfa-policy",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/mfa-policy", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/auth/mfa-policy")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Whether a second factor is required. The body is JSON the schemas do not describe yet.

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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PUT /api/auth/mfa-policy`

Persists `daemon.auth.require_second_factor` to `config.toml`.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`MfaPolicyPut`](schemas.md#mfapolicyput)

- `require_second_factor` *boolean*, **required** — When true, password login requires TOTP or the user must use a passkey.
- `current_password` *string or null* — Current password for Owner step-up (operators omit this).

### Responses

- `200` — The policy after the change. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/mfa-policy' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"require_second_factor": true}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/mfa-policy' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"require_second_factor": true}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/auth/mfa-policy",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"require_second_factor": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/mfa-policy", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"require_second_factor": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/auth/mfa-policy")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"require_second_factor": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The policy after the change. The body is JSON the schemas do not describe yet.

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

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
