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

- `200` — Signed in; the response sets the operator session cookie. Body: [`LoginResponse`](schemas.md#loginresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `role` *string*, **required** — Issued role (`operator` for token login).
  - `default_view` *string*, **required** — Post-login SPA landing view from operator preferences (`discover` on lookup failure).
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The token is wrong (`unauthorized`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — An Owner exists, so operator sign-in is off (`operator_login_disabled`); sign in as a user. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many attempts from this client; retry after `Retry-After` seconds. Body: [`LoginThrottled`](schemas.md#loginthrottled) (`application/json`)
  - `error` *string*, **required** — Always `too_many_requests`.
  - `message` *string*, **required** — Caller-facing explanation naming the wait.
  - `status` *integer (int32)*, **required** — Always 429 (duplicates the response status for clients). At least 0.
  - `retry_after_secs` *integer (int64)*, **required** — Seconds until this client may try again; the same value as `Retry-After`. At least 0.
- `500` — The library could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/login' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"token": "9b1e4f6a2c8d3e5f7a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/login' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"token": "9b1e4f6a2c8d3e5f7a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/login",
    json={"token": "9b1e4f6a2c8d3e5f7a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/login", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"token": "9b1e4f6a2c8d3e5f7a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/login")
    .json(&serde_json::json!({"token": "9b1e4f6a2c8d3e5f7a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in; the response sets the operator session cookie.

```json
{
  "ok": true,
  "role": "operator",
  "default_view": "discover"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after a successful operator token login.",
  "required": [
    "ok",
    "role",
    "default_view"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "role": {
      "type": "string",
      "description": "Issued role (`operator` for token login).",
      "example": "operator"
    },
    "default_view": {
      "type": "string",
      "description": "Post-login SPA landing view from operator preferences (`discover` on lookup failure).",
      "example": "discover"
    }
  }
}
```

Status `400`: The body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

Status `401`: The token is wrong (`unauthorized`).

```json
{
  "error": "unauthorized",
  "message": "Invalid operator token.",
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

Status `403`: An Owner exists, so operator sign-in is off (`operator_login_disabled`); sign in as a user.

```json
{
  "error": "operator_login_disabled",
  "message": "Operator token sign-in is unavailable after an Owner account exists. Open Bookclerk from the system tray, or elevate from an Owner session.",
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `token` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many attempts from this client; retry after `Retry-After` seconds.

```json
{
  "error": "too_many_requests",
  "message": "Too many failed login attempts. Try again in 45 seconds.",
  "retry_after_secs": 45,
  "status": 429
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a 429 after too many failed sign-in attempts from one client.",
  "required": [
    "error",
    "message",
    "status",
    "retry_after_secs"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Always `too_many_requests`.",
      "example": "too_many_requests"
    },
    "message": {
      "type": "string",
      "description": "Caller-facing explanation naming the wait.",
      "example": "Too many failed login attempts. Try again in 45 seconds."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Always 429 (duplicates the response status for clients).",
      "example": 429,
      "minimum": 0
    },
    "retry_after_secs": {
      "type": "integer",
      "format": "int64",
      "description": "Seconds until this client may try again; the same value as `Retry-After`.",
      "example": 45,
      "minimum": 0
    }
  }
}
```

Status `500`: The library could not be read.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `GET /api/auth/signin`

Public sign-in picker: operator-token availability, SSO IdPs, integration logins.

### Errors

Returns 500 when the owner-count query fails.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — The sign-in methods this library offers. Body: [`SigninMethods`](schemas.md#signinmethods) (`application/json`)
  - `operator_token` *boolean*, **required** — Whether the operator token may be entered in the SPA (only before an Owner exists).
  - `oidc` *array of [`SigninOidcProvider`](schemas.md#signinoidcprovider)*, **required** — Enabled OIDC identity providers, in configuration order.
  - `integrations` *array of [`SigninIntegration`](schemas.md#signinintegration)*, **required** — Integrations that accept their own credentials at sign-in.
  - `require_second_factor` *boolean*, **required** — Whether `daemon.auth.require_second_factor` is on, so clients can explain the enrollment step.
- `500` — The library could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

### Example response

Status `200`: The sign-in methods this library offers.

```json
{
  "operator_token": false,
  "oidc": [
    {
      "id": "google",
      "name": "Google",
      "preset": "google"
    }
  ],
  "integrations": [
    {
      "id": "audiobookshelf",
      "name": "Audiobookshelf"
    }
  ],
  "require_second_factor": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/auth/signin`: the ways this library lets people sign in.",
  "required": [
    "operator_token",
    "oidc",
    "integrations",
    "require_second_factor"
  ],
  "properties": {
    "operator_token": {
      "type": "boolean",
      "description": "Whether the operator token may be entered in the SPA (only before an Owner exists).",
      "example": false
    },
    "oidc": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One OIDC identity provider offered on the sign-in page.",
        "required": [
          "id",
          "name"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Provider id from `[auth.oidc.providers]`, used in `/api/auth/oidc/{id}/start`.",
            "example": "google"
          },
          "name": {
            "type": "string",
            "description": "Button label.",
            "example": "Google"
          },
          "preset": {
            "type": [
              "string",
              "null"
            ],
            "description": "Known social preset the client may style (`google`, `github`, …), when the provider matches one.",
            "example": "google"
          }
        }
      },
      "description": "Enabled OIDC identity providers, in configuration order."
    },
    "integrations": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One integration that signs people in with its own credentials.",
        "required": [
          "id",
          "name"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Integration id, used in `/api/integrations/{id}/login`.",
            "example": "audiobookshelf"
          },
          "name": {
            "type": "string",
            "description": "Button label.",
            "example": "Audiobookshelf"
          }
        }
      },
      "description": "Integrations that accept their own credentials at sign-in."
    },
    "require_second_factor": {
      "type": "boolean",
      "description": "Whether `daemon.auth.require_second_factor` is on, so clients can explain the enrollment step.",
      "example": false
    }
  }
}
```

Status `500`: The library could not be read.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

- `200` — A single-use tray ticket. Body: [`TrayHandoffPrepareResponse`](schemas.md#trayhandoffprepareresponse) (`application/json`)
  - `code` *string*, **required** — Single-use loopback handoff code for `GET /api/auth/tray-handoff?code=`.
  - `expires_in_secs` *integer (int64)*, **required** — Seconds until this ticket expires (matches the in-process deadline). At least 0.
- `204` — Operator authentication is off, so no ticket is needed. No body.
- `401` — The Bearer token is missing or wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The request did not come straight from loopback. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `429` — Too many failed attempts from this client; retry after `Retry-After` seconds. Body: [`LoginThrottled`](schemas.md#loginthrottled) (`application/json`)
  - `error` *string*, **required** — Always `too_many_requests`.
  - `message` *string*, **required** — Caller-facing explanation naming the wait.
  - `status` *integer (int32)*, **required** — Always 429 (duplicates the response status for clients). At least 0.
  - `retry_after_secs` *integer (int64)*, **required** — Seconds until this client may try again; the same value as `Retry-After`. At least 0.
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

Status `200`: A single-use tray ticket.

```json
{
  "code": "9b4f1c2e-3d5a-4b6c-8e7f-0a1b2c3d4e5f",
  "expires_in_secs": 180
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON returned to the tray after a successful prepare.",
  "required": [
    "code",
    "expires_in_secs"
  ],
  "properties": {
    "code": {
      "type": "string",
      "description": "Single-use loopback handoff code for `GET /api/auth/tray-handoff?code=`.",
      "example": "9b4f1c2e-3d5a-4b6c-8e7f-0a1b2c3d4e5f"
    },
    "expires_in_secs": {
      "type": "integer",
      "format": "int64",
      "description": "Seconds until this ticket expires (matches the in-process deadline).",
      "example": 180,
      "minimum": 0
    }
  }
}
```

Status `204`: Operator authentication is off, so no ticket is needed. No body.

Status `401`: The Bearer token is missing or wrong.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The request did not come straight from loopback.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `429`: Too many failed attempts from this client; retry after `Retry-After` seconds.

```json
{
  "error": "too_many_requests",
  "message": "Too many failed login attempts. Try again in 45 seconds.",
  "retry_after_secs": 45,
  "status": 429
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a 429 after too many failed sign-in attempts from one client.",
  "required": [
    "error",
    "message",
    "status",
    "retry_after_secs"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Always `too_many_requests`.",
      "example": "too_many_requests"
    },
    "message": {
      "type": "string",
      "description": "Caller-facing explanation naming the wait.",
      "example": "Too many failed login attempts. Try again in 45 seconds."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Always 429 (duplicates the response status for clients).",
      "example": 429,
      "minimum": 0
    },
    "retry_after_secs": {
      "type": "integer",
      "format": "int64",
      "description": "Seconds until this client may try again; the same value as `Retry-After`.",
      "example": 45,
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
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `GET /api/auth/me`

Resolves the caller and returns `/me` JSON; unauthenticated callers get 401.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Who the caller is (operator, portal user, or impersonation). Body: [`AuthMeResponse`](schemas.md#authmeresponse) (`application/json`)
  - `authenticated` *boolean*, **required** — Whether the request resolved to an operator, portal, or impersonated session.
  - `role` *string or null* — `operator`, `owner`, `administrator`, `host`, or `member` (legacy clients may still send `portal`).
  - `default_view` *string*, **required** — Configured post-auth landing view (`discover` / `library` / `accounts` / `wishlist`). Loaded from per-user DB preferences (not config.toml).
  - `can_acquire` *boolean*, **required** — Whether this session may acquire / scan / manage jobs. True for operator and administrator.
  - `elevated` *boolean*, **required** — True when this operator session was created via Owner elevate.
  - `impersonating` *[`AuthMeImpersonating`](schemas.md#authmeimpersonating) or null* — Present when the operator is impersonating a User.
  - `portal` *[`PortalMeInfo`](schemas.md#portalmeinfo) or null* — Linked portal identity when the caller is a portal (or impersonated) user.
  - `user` *[`AuthMeUser`](schemas.md#authmeuser) or null* — First-party user when the session is linked to a `users` row.
  - `second_factor` *[`AuthSecondFactor`](schemas.md#authsecondfactor) or null* — Password-login second-factor policy and this user's enrollment.
- `401` — Nobody is signed in: the same body with `authenticated` false, so the app can show the sign-in screen. Body: [`AuthMeResponse`](schemas.md#authmeresponse) (`application/json`)
  - `authenticated` *boolean*, **required** — Whether the request resolved to an operator, portal, or impersonated session.
  - `role` *string or null* — `operator`, `owner`, `administrator`, `host`, or `member` (legacy clients may still send `portal`).
  - `default_view` *string*, **required** — Configured post-auth landing view (`discover` / `library` / `accounts` / `wishlist`). Loaded from per-user DB preferences (not config.toml).
  - `can_acquire` *boolean*, **required** — Whether this session may acquire / scan / manage jobs. True for operator and administrator.
  - `elevated` *boolean*, **required** — True when this operator session was created via Owner elevate.
  - `impersonating` *[`AuthMeImpersonating`](schemas.md#authmeimpersonating) or null* — Present when the operator is impersonating a User.
  - `portal` *[`PortalMeInfo`](schemas.md#portalmeinfo) or null* — Linked portal identity when the caller is a portal (or impersonated) user.
  - `user` *[`AuthMeUser`](schemas.md#authmeuser) or null* — First-party user when the session is linked to a `users` row.
  - `second_factor` *[`AuthSecondFactor`](schemas.md#authsecondfactor) or null* — Password-login second-factor policy and this user's enrollment.
- `503` — The portal session could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than showing the sign-in screen. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `200`: Who the caller is (operator, portal user, or impersonation).

```json
{
  "authenticated": true,
  "role": "administrator",
  "default_view": "library",
  "can_acquire": true,
  "elevated": false,
  "impersonating": {
    "user_id": 7,
    "display_name": "Sam"
  },
  "portal": {
    "identity_id": 3,
    "provider": "local",
    "external_user_id": "7",
    "label": "Sam"
  },
  "user": {
    "id": 7,
    "role": "administrator",
    "display_name": "Sam",
    "email": "sam@example.com",
    "has_password": true,
    "has_avatar": false,
    "avatar_source": "auto",
    "gravatar_hash": "0bc83cb571cd1c50ba6f3e8a78ef1346b6b31bd4d35b8d3dc1f0c8a3b4a9f7e2",
    "sso_pictures": [
      {
        "identity_id": 3,
        "provider": "oidc:google",
        "picture_url": "https://lh3.googleusercontent.com/a/example=s96-c",
        "picture_path": "/api/images/aHR0cHM6Ly9saDMuZ29vZ2xldXNlcmNvbnRlbnQuY29tL2EvZXhhbXBsZT1zOTYtYw.N14rqh66Bok9-eCqJKBNoQ",
        "last_used_at": "2026-10-29T08:15:00Z"
      }
    ]
  },
  "second_factor": {
    "required": false,
    "totp": true,
    "passkey_count": 1,
    "enrolled": true
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/auth/me`.",
  "required": [
    "authenticated",
    "default_view",
    "can_acquire",
    "elevated"
  ],
  "properties": {
    "authenticated": {
      "type": "boolean",
      "description": "Whether the request resolved to an operator, portal, or impersonated session.",
      "example": true
    },
    "role": {
      "type": [
        "string",
        "null"
      ],
      "description": "`operator`, `owner`, `administrator`, `host`, or `member` (legacy clients may\nstill send `portal`).",
      "example": "administrator"
    },
    "default_view": {
      "type": "string",
      "description": "Configured post-auth landing view (`discover` / `library` / `accounts` /\n`wishlist`).\nLoaded from per-user DB preferences (not config.toml).",
      "example": "library"
    },
    "can_acquire": {
      "type": "boolean",
      "description": "Whether this session may acquire / scan / manage jobs.\nTrue for operator and administrator.",
      "example": true
    },
    "elevated": {
      "type": "boolean",
      "description": "True when this operator session was created via Owner elevate.",
      "example": false
    },
    "impersonating": {
      "oneOf": [
        {
          "type": "object",
          "description": "Present when the operator is impersonating a User.",
          "required": [
            "user_id"
          ],
          "properties": {
            "user_id": {
              "type": "integer",
              "format": "int64",
              "description": "First-party `users.id` being impersonated.",
              "example": 7
            },
            "display_name": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display name of the impersonated user when the row exists.",
              "example": "Sam"
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "portal": {
      "oneOf": [
        {
          "type": "object",
          "description": "Linked portal identity when the caller is a portal (or impersonated) user.",
          "required": [
            "identity_id",
            "provider",
            "external_user_id"
          ],
          "properties": {
            "identity_id": {
              "type": "integer",
              "format": "int64",
              "description": "`portal_identities.id` for this session.",
              "example": 3
            },
            "provider": {
              "type": "string",
              "description": "Identity-broker id (`local`, OIDC provider name, …).",
              "example": "local"
            },
            "external_user_id": {
              "type": "string",
              "description": "Provider-stable subject / external user id.",
              "example": "7"
            },
            "label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Optional operator-facing label for the identity.",
              "example": "Sam"
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "user": {
      "oneOf": [
        {
          "type": "object",
          "description": "First-party user when the session is linked to a `users` row.",
          "required": [
            "id",
            "role",
            "has_password",
            "has_avatar",
            "avatar_source"
          ],
          "properties": {
            "id": {
              "type": "integer",
              "format": "int64",
              "description": "First-party `users.id`.",
              "example": 7
            },
            "role": {
              "type": "string",
              "description": "Role string (`owner`, `administrator`, `member`).",
              "example": "administrator"
            },
            "display_name": {
              "type": [
                "string",
                "null"
              ],
              "description": "Optional human-readable name from the user row.",
              "example": "Sam"
            },
            "email": {
              "type": [
                "string",
                "null"
              ],
              "description": "Optional email; omitted from JSON when unset.",
              "example": "sam@example.com"
            },
            "has_password": {
              "type": "boolean",
              "description": "True when a local password hash is stored (invite users may be false).",
              "example": true
            },
            "has_avatar": {
              "type": "boolean",
              "description": "True when a JPEG avatar is stored under `files_dir/avatars/{id}.*`.",
              "example": false
            },
            "avatar_source": {
              "type": "string",
              "description": "Selected picture source (`auto`, `monogram`, `gravatar`, `upload`, `sso:{id}`).",
              "example": "auto"
            },
            "gravatar_hash": {
              "type": [
                "string",
                "null"
              ],
              "description": "SHA-256 hex of the contact email for Gravatar, when an email is set.",
              "example": "0bc83cb571cd1c50ba6f3e8a78ef1346b6b31bd4d35b8d3dc1f0c8a3b4a9f7e2"
            },
            "sso_pictures": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "HTTPS picture from a linked identity provider.",
                "required": [
                  "identity_id",
                  "provider",
                  "picture_url"
                ],
                "properties": {
                  "identity_id": {
                    "type": "integer",
                    "format": "int64",
                    "description": "`portal_identities.id`.",
                    "example": 3
                  },
                  "provider": {
                    "type": "string",
                    "description": "Identity-broker id (`oidc:google`, …).",
                    "example": "oidc:google"
                  },
                  "picture_url": {
                    "type": "string",
                    "description": "HTTPS URL to display.",
                    "example": "https://lh3.googleusercontent.com/a/example=s96-c"
                  },
                  "picture_path": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Same-origin 128 px copy of `picture_url` for the web app to show,\nwhen this node can sign it.",
                    "example": "/api/images/aHR0cHM6Ly9saDMuZ29vZ2xldXNlcmNvbnRlbnQuY29tL2EvZXhhbXBsZT1zOTYtYw.N14rqh66Bok9-eCqJKBNoQ"
                  },
                  "last_used_at": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "RFC 3339 last portal-session activity for this identity, when any.",
                    "example": "2026-10-29T08:15:00Z"
                  }
                }
              },
              "description": "IdP-supplied pictures the user can pick."
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "second_factor": {
      "oneOf": [
        {
          "type": "object",
          "description": "Password-login second-factor policy and this user's enrollment.",
          "required": [
            "required",
            "totp",
            "passkey_count",
            "enrolled"
          ],
          "properties": {
            "required": {
              "type": "boolean",
              "description": "`daemon.auth.require_second_factor`.",
              "example": false
            },
            "totp": {
              "type": "boolean",
              "description": "Confirmed authenticator-app TOTP.",
              "example": true
            },
            "passkey_count": {
              "type": "integer",
              "format": "int64",
              "description": "Number of registered passkeys.",
              "example": 1,
              "minimum": 0
            },
            "enrolled": {
              "type": "boolean",
              "description": "True when TOTP is enabled or at least one passkey exists.",
              "example": true
            }
          }
        },
        {
          "type": "null"
        }
      ]
    }
  }
}
```

Status `401`: Nobody is signed in: the same body with `authenticated` false, so the app can show the sign-in screen.

```json
{
  "authenticated": false,
  "can_acquire": false,
  "default_view": "discover",
  "elevated": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/auth/me`.",
  "required": [
    "authenticated",
    "default_view",
    "can_acquire",
    "elevated"
  ],
  "properties": {
    "authenticated": {
      "type": "boolean",
      "description": "Whether the request resolved to an operator, portal, or impersonated session.",
      "example": true
    },
    "role": {
      "type": [
        "string",
        "null"
      ],
      "description": "`operator`, `owner`, `administrator`, `host`, or `member` (legacy clients may\nstill send `portal`).",
      "example": "administrator"
    },
    "default_view": {
      "type": "string",
      "description": "Configured post-auth landing view (`discover` / `library` / `accounts` /\n`wishlist`).\nLoaded from per-user DB preferences (not config.toml).",
      "example": "library"
    },
    "can_acquire": {
      "type": "boolean",
      "description": "Whether this session may acquire / scan / manage jobs.\nTrue for operator and administrator.",
      "example": true
    },
    "elevated": {
      "type": "boolean",
      "description": "True when this operator session was created via Owner elevate.",
      "example": false
    },
    "impersonating": {
      "oneOf": [
        {
          "type": "object",
          "description": "Present when the operator is impersonating a User.",
          "required": [
            "user_id"
          ],
          "properties": {
            "user_id": {
              "type": "integer",
              "format": "int64",
              "description": "First-party `users.id` being impersonated.",
              "example": 7
            },
            "display_name": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display name of the impersonated user when the row exists.",
              "example": "Sam"
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "portal": {
      "oneOf": [
        {
          "type": "object",
          "description": "Linked portal identity when the caller is a portal (or impersonated) user.",
          "required": [
            "identity_id",
            "provider",
            "external_user_id"
          ],
          "properties": {
            "identity_id": {
              "type": "integer",
              "format": "int64",
              "description": "`portal_identities.id` for this session.",
              "example": 3
            },
            "provider": {
              "type": "string",
              "description": "Identity-broker id (`local`, OIDC provider name, …).",
              "example": "local"
            },
            "external_user_id": {
              "type": "string",
              "description": "Provider-stable subject / external user id.",
              "example": "7"
            },
            "label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Optional operator-facing label for the identity.",
              "example": "Sam"
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "user": {
      "oneOf": [
        {
          "type": "object",
          "description": "First-party user when the session is linked to a `users` row.",
          "required": [
            "id",
            "role",
            "has_password",
            "has_avatar",
            "avatar_source"
          ],
          "properties": {
            "id": {
              "type": "integer",
              "format": "int64",
              "description": "First-party `users.id`.",
              "example": 7
            },
            "role": {
              "type": "string",
              "description": "Role string (`owner`, `administrator`, `member`).",
              "example": "administrator"
            },
            "display_name": {
              "type": [
                "string",
                "null"
              ],
              "description": "Optional human-readable name from the user row.",
              "example": "Sam"
            },
            "email": {
              "type": [
                "string",
                "null"
              ],
              "description": "Optional email; omitted from JSON when unset.",
              "example": "sam@example.com"
            },
            "has_password": {
              "type": "boolean",
              "description": "True when a local password hash is stored (invite users may be false).",
              "example": true
            },
            "has_avatar": {
              "type": "boolean",
              "description": "True when a JPEG avatar is stored under `files_dir/avatars/{id}.*`.",
              "example": false
            },
            "avatar_source": {
              "type": "string",
              "description": "Selected picture source (`auto`, `monogram`, `gravatar`, `upload`, `sso:{id}`).",
              "example": "auto"
            },
            "gravatar_hash": {
              "type": [
                "string",
                "null"
              ],
              "description": "SHA-256 hex of the contact email for Gravatar, when an email is set.",
              "example": "0bc83cb571cd1c50ba6f3e8a78ef1346b6b31bd4d35b8d3dc1f0c8a3b4a9f7e2"
            },
            "sso_pictures": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "HTTPS picture from a linked identity provider.",
                "required": [
                  "identity_id",
                  "provider",
                  "picture_url"
                ],
                "properties": {
                  "identity_id": {
                    "type": "integer",
                    "format": "int64",
                    "description": "`portal_identities.id`.",
                    "example": 3
                  },
                  "provider": {
                    "type": "string",
                    "description": "Identity-broker id (`oidc:google`, …).",
                    "example": "oidc:google"
                  },
                  "picture_url": {
                    "type": "string",
                    "description": "HTTPS URL to display.",
                    "example": "https://lh3.googleusercontent.com/a/example=s96-c"
                  },
                  "picture_path": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Same-origin 128 px copy of `picture_url` for the web app to show,\nwhen this node can sign it.",
                    "example": "/api/images/aHR0cHM6Ly9saDMuZ29vZ2xldXNlcmNvbnRlbnQuY29tL2EvZXhhbXBsZT1zOTYtYw.N14rqh66Bok9-eCqJKBNoQ"
                  },
                  "last_used_at": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "RFC 3339 last portal-session activity for this identity, when any.",
                    "example": "2026-10-29T08:15:00Z"
                  }
                }
              },
              "description": "IdP-supplied pictures the user can pick."
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "second_factor": {
      "oneOf": [
        {
          "type": "object",
          "description": "Password-login second-factor policy and this user's enrollment.",
          "required": [
            "required",
            "totp",
            "passkey_count",
            "enrolled"
          ],
          "properties": {
            "required": {
              "type": "boolean",
              "description": "`daemon.auth.require_second_factor`.",
              "example": false
            },
            "totp": {
              "type": "boolean",
              "description": "Confirmed authenticator-app TOTP.",
              "example": true
            },
            "passkey_count": {
              "type": "integer",
              "format": "int64",
              "description": "Number of registered passkeys.",
              "example": 1,
              "minimum": 0
            },
            "enrolled": {
              "type": "boolean",
              "description": "True when TOTP is enabled or at least one passkey exists.",
              "example": true
            }
          }
        },
        {
          "type": "null"
        }
      ]
    }
  }
}
```

Status `503`: The portal session could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than showing the sign-in screen.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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

## `POST /api/auth/elevate`

Owner portal session + password re-auth → short-lived elevated operator cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`ElevateRequest`](schemas.md#elevaterequest)

- `password` *string*, **required** — Owner account password (re-authentication).

### Responses

- `200` — Elevated; the response sets a short-lived operator cookie. Body: [`Elevated`](schemas.md#elevated) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `elevated` *boolean*, **required** — Always `true`: the cookie the response sets is an elevated operator session.
  - `expires_in_secs` *integer (int64)*, **required** — Seconds until the elevated session expires. At least 0.
- `400` — Operator auth is off, or the password is empty. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — No portal session is signed in, or the password is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is not an Owner, or the Owner has no password to check. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The caller is already an operator. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many wrong passwords from this client or for this Owner; retry after `Retry-After` seconds. Body: [`LoginThrottled`](schemas.md#loginthrottled) (`application/json`)
  - `error` *string*, **required** — Always `too_many_requests`.
  - `message` *string*, **required** — Caller-facing explanation naming the wait.
  - `status` *integer (int32)*, **required** — Always 429 (duplicates the response status for clients). At least 0.
  - `retry_after_secs` *integer (int64)*, **required** — Seconds until this client may try again; the same value as `Retry-After`. At least 0.
- `500` — The password hash could not be read, or the elevated session could not be stored. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/elevate' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"password": "correct horse battery staple"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/elevate' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"password": "correct horse battery staple"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/elevate",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"password": "correct horse battery staple"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/elevate", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"password": "correct horse battery staple"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/elevate")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"password": "correct horse battery staple"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Elevated; the response sets a short-lived operator cookie.

```json
{
  "ok": true,
  "elevated": true,
  "expires_in_secs": 900
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after a successful Owner elevation.",
  "required": [
    "ok",
    "elevated",
    "expires_in_secs"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "elevated": {
      "type": "boolean",
      "description": "Always `true`: the cookie the response sets is an elevated operator session.",
      "example": true
    },
    "expires_in_secs": {
      "type": "integer",
      "format": "int64",
      "description": "Seconds until the elevated session expires.",
      "example": 900,
      "minimum": 0
    }
  }
}
```

Status `400`: Operator auth is off, or the password is empty. As `text/plain`, when the body is not syntactically valid JSON.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `401`: No portal session is signed in, or the password is wrong.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller is not an Owner, or the Owner has no password to check. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `409`: The caller is already an operator.

```json
{
  "error": "conflict",
  "message": "Conflict",
  "status": 409
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `password` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many wrong passwords from this client or for this Owner; retry after `Retry-After` seconds.

```json
{
  "error": "too_many_requests",
  "message": "Too many failed login attempts. Try again in 45 seconds.",
  "retry_after_secs": 45,
  "status": 429
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a 429 after too many failed sign-in attempts from one client.",
  "required": [
    "error",
    "message",
    "status",
    "retry_after_secs"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Always `too_many_requests`.",
      "example": "too_many_requests"
    },
    "message": {
      "type": "string",
      "description": "Caller-facing explanation naming the wait.",
      "example": "Too many failed login attempts. Try again in 45 seconds."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Always 429 (duplicates the response status for clients).",
      "example": 429,
      "minimum": 0
    },
    "retry_after_secs": {
      "type": "integer",
      "format": "int64",
      "description": "Seconds until this client may try again; the same value as `Retry-After`.",
      "example": 45,
      "minimum": 0
    }
  }
}
```

Status `500`: The password hash could not be read, or the elevated session could not be stored.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `DELETE /api/auth/elevate`

End elevation by revoking the elevated operator session cookie.

**Access:** The operator (token or session) only.

**Credentials:** `operatorSession`

### Responses

- `200` — Elevation ended. Body: [`OkResponse`](schemas.md#okresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`: a failure answers with an error status instead.
- `400` — The session is not an elevated one. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: Elevation ended.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Acknowledgement body (`{\"ok\": true}`) for actions with nothing else to report.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`: a failure answers with an error status instead.",
      "example": true
    }
  }
}
```

Status `400`: The session is not an elevated one.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The session is impersonating a user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

- `200` — The first Owner was created. Body: [`OwnerBootstrapped`](schemas.md#ownerbootstrapped) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `user_id` *integer (int64)*, **required** — `users.id` of the new Owner.
  - `claim_ticket` *string*, **required** — Single-use claim ticket that signs the Owner in once (valid for seven days).
  - `invite_url` *string*, **required** — Invite link carrying the ticket, for the browser.
  - `login_name` *string or null* — Login name the Owner signs in with, when one was given.
  - `has_password` *boolean*, **required** — Whether a password was set, so password sign-in works right away.
- `400` — The email is invalid. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not the operator. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — An Owner or Administrator already exists. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: The first Owner was created.

```json
{
  "ok": true,
  "user_id": 1,
  "claim_ticket": "5f1c9a2e-7b3d-4e6f-8a90-1b2c3d4e5f60",
  "invite_url": "https://books.example.com/invite?ticket=5f1c9a2e-7b3d-4e6f-8a90-1b2c3d4e5f60",
  "login_name": "casey",
  "has_password": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after the first Owner was created.",
  "required": [
    "ok",
    "user_id",
    "claim_ticket",
    "invite_url",
    "has_password"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "user_id": {
      "type": "integer",
      "format": "int64",
      "description": "`users.id` of the new Owner.",
      "example": 1
    },
    "claim_ticket": {
      "type": "string",
      "description": "Single-use claim ticket that signs the Owner in once (valid for seven days).",
      "example": "5f1c9a2e-7b3d-4e6f-8a90-1b2c3d4e5f60"
    },
    "invite_url": {
      "type": "string",
      "description": "Invite link carrying the ticket, for the browser.",
      "example": "https://books.example.com/invite?ticket=5f1c9a2e-7b3d-4e6f-8a90-1b2c3d4e5f60"
    },
    "login_name": {
      "type": [
        "string",
        "null"
      ],
      "description": "Login name the Owner signs in with, when one was given.",
      "example": "casey"
    },
    "has_password": {
      "type": "boolean",
      "description": "Whether a password was set, so password sign-in works right away.",
      "example": true
    }
  }
}
```

Status `400`: The email is invalid. As `text/plain`, when the body is not syntactically valid JSON.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `401`: The caller is not the operator.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `409`: An Owner or Administrator already exists.

```json
{
  "error": "conflict",
  "message": "Conflict",
  "status": 409
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: display_name: invalid type: integer `1`, expected a string at line 1 column 17
```

Schema:

A `text/plain` body: a string.

Status `500`: The library could not be read or written.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

- `200` — The password was set and other sessions revoked. Body: [`PasswordSet`](schemas.md#passwordset) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `revoked_sessions` *integer (int64)*, **required** — Portal sessions of that user revoked by the change (the caller signs in again). At least 0.
- `400` — The password is empty (the standard error body). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the caller already has a password and `current_password` is missing or wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not set that user's password: not a provisioner, or a role the caller cannot manage. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many wrong `current_password` attempts for this account; try again after the lockout. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The password could not be hashed, or the library could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/password' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"password": "new horse battery staple"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/password' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"password": "new horse battery staple"}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/auth/password",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"password": "new horse battery staple"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/password", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"password": "new horse battery staple"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/auth/password")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"password": "new horse battery staple"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The password was set and other sessions revoked.

```json
{
  "ok": true,
  "revoked_sessions": 2
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after a password was set.",
  "required": [
    "ok",
    "revoked_sessions"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "revoked_sessions": {
      "type": "integer",
      "format": "int64",
      "description": "Portal sessions of that user revoked by the change (the caller signs in again).",
      "example": 2,
      "minimum": 0
    }
  }
}
```

Status `400`: The password is empty (the standard error body). As `text/plain`, when the body is not syntactically valid JSON.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `401`: The caller is not signed in, or the caller already has a password and `current_password` is missing or wrong.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller may not set that user's password: not a provisioner, or a role the caller cannot manage.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `password` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many wrong `current_password` attempts for this account; try again after the lockout.

```json
{
  "error": "too_many_requests",
  "message": "Too many requests. Try again shortly.",
  "status": 429
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The password could not be hashed, or the library could not be read or written.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/password`

Local password login → portal session cookie, or a TOTP challenge when enabled.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`PasswordLoginRequest`](schemas.md#passwordloginrequest)

- `login` *string*, **required** — Login name or email.
- `password` *string*, **required** — Password verified against the stored hash; mismatch fails closed and counts toward lockout.

### Responses

- `200` — Signed in (portal session cookie), or a TOTP challenge. Body: [`PasswordLoginOutcome`](schemas.md#passwordloginoutcome) (`application/json`)
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The login or password is wrong, or the user has no password (`invalid_credentials`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — The account is disabled (`account_disabled`), or the server requires a second factor and the user has a passkey but no authenticator app (`passkey_required`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many attempts from this client; retry after `Retry-After` seconds. Body: [`LoginThrottled`](schemas.md#loginthrottled) (`application/json`)
  - `error` *string*, **required** — Always `too_many_requests`.
  - `message` *string*, **required** — Caller-facing explanation naming the wait.
  - `status` *integer (int32)*, **required** — Always 429 (duplicates the response status for clients). At least 0.
  - `retry_after_secs` *integer (int64)*, **required** — Seconds until this client may try again; the same value as `Retry-After`. At least 0.
- `500` — The user or password hash could not be read, or the session could not be stored. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/password' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"login": "dana", "password": "correct horse battery staple"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/password' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"login": "dana", "password": "correct horse battery staple"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/password",
    json={"login": "dana", "password": "correct horse battery staple"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/password", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"login": "dana", "password": "correct horse battery staple"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/password")
    .json(&serde_json::json!({"login": "dana", "password": "correct horse battery staple"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in (portal session cookie), or a TOTP challenge.

```json
{
  "ok": true,
  "role": "member"
}
```

Schema:

```json
{
  "oneOf": [
    {
      "type": "object",
      "description": "The password was enough; the response sets the portal session cookie.",
      "required": [
        "ok",
        "role"
      ],
      "properties": {
        "ok": {
          "type": "boolean",
          "description": "Always `true` on this success path.",
          "example": true
        },
        "role": {
          "type": "string",
          "description": "The user's role (`owner`, `administrator`, `host`, or `member`).",
          "example": "member"
        }
      }
    },
    {
      "type": "object",
      "description": "The user has an authenticator app, so a code is needed before a session is issued.",
      "required": [
        "ok",
        "mfa"
      ],
      "properties": {
        "ok": {
          "type": "boolean",
          "description": "Always `true`: the password step succeeded.",
          "example": true
        },
        "mfa": {
          "type": "object",
          "description": "The second step the client must complete.",
          "required": [
            "method",
            "challenge_id"
          ],
          "properties": {
            "method": {
              "type": "string",
              "description": "Always `totp`: finish with `POST /api/auth/totp/login`.",
              "example": "totp"
            },
            "challenge_id": {
              "type": "string",
              "description": "Challenge to send with the code; expires after five minutes.",
              "example": "2c7e4f0a-9b1d-4e3c-a5f6-7d8e9f0a1b2c"
            }
          }
        }
      }
    }
  ],
  "description": "JSON body of a successful `POST /api/auth/password`."
}
```

Status `400`: The body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

Status `401`: The login or password is wrong, or the user has no password (`invalid_credentials`).

```json
{
  "error": "invalid_credentials",
  "message": "Invalid login or password.",
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

Status `403`: The account is disabled (`account_disabled`), or the server requires a second factor and the user has a passkey but no authenticator app (`passkey_required`).

```json
{
  "error": "account_disabled",
  "message": "This account is disabled.",
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `login` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many attempts from this client; retry after `Retry-After` seconds.

```json
{
  "error": "too_many_requests",
  "message": "Too many failed login attempts. Try again in 45 seconds.",
  "retry_after_secs": 45,
  "status": 429
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a 429 after too many failed sign-in attempts from one client.",
  "required": [
    "error",
    "message",
    "status",
    "retry_after_secs"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Always `too_many_requests`.",
      "example": "too_many_requests"
    },
    "message": {
      "type": "string",
      "description": "Caller-facing explanation naming the wait.",
      "example": "Too many failed login attempts. Try again in 45 seconds."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Always 429 (duplicates the response status for clients).",
      "example": 429,
      "minimum": 0
    },
    "retry_after_secs": {
      "type": "integer",
      "format": "int64",
      "description": "Seconds until this client may try again; the same value as `Retry-After`.",
      "example": 45,
      "minimum": 0
    }
  }
}
```

Status `500`: The user or password hash could not be read, or the session could not be stored.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/impersonate`

Operator (or elevated) session starts impersonating a User.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`ImpersonateRequest`](schemas.md#impersonaterequest)

- `user_id` *integer (int64)*, **required** — Target `users.id` to impersonate.

### Responses

- `200` — The session now acts as the user. Body: [`ImpersonationStarted`](schemas.md#impersonationstarted) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `impersonating` *[`AuthMeImpersonating`](schemas.md#authmeimpersonating)*, **required** — The user the session now acts as.
- `400` — Operator auth is off. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is already impersonating a user, or the target user is disabled. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The user could not be read, or the session could not be stored. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/impersonate' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"user_id": 7}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/impersonate' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"user_id": 7}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/impersonate",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"user_id": 7},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/impersonate", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"user_id": 7}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/impersonate")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"user_id": 7}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The session now acts as the user.

```json
{
  "ok": true,
  "impersonating": {
    "user_id": 7,
    "display_name": "Sam"
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after an operator session started impersonating a user.",
  "required": [
    "ok",
    "impersonating"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "impersonating": {
      "type": "object",
      "description": "The user the session now acts as.",
      "required": [
        "user_id"
      ],
      "properties": {
        "user_id": {
          "type": "integer",
          "format": "int64",
          "description": "First-party `users.id` being impersonated.",
          "example": 7
        },
        "display_name": {
          "type": [
            "string",
            "null"
          ],
          "description": "Display name of the impersonated user when the row exists.",
          "example": "Sam"
        }
      }
    }
  }
}
```

Status `400`: Operator auth is off. As `text/plain`, when the body is not syntactically valid JSON.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The session is already impersonating a user, or the target user is disabled. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `user_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The user could not be read, or the session could not be stored.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `DELETE /api/auth/impersonate`

Clear impersonation on the current operator session.

**Access:** The operator (token or session) only.

**Credentials:** `operatorSession`

### Responses

- `200` — Impersonation ended. An elevated Owner's own portal session cookie is set again; otherwise the impersonation's portal cookie is cleared. Body: [`OkResponse`](schemas.md#okresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`: a failure answers with an error status instead.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The session could not be updated. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: Impersonation ended. An elevated Owner's own portal session cookie is set again; otherwise the impersonation's portal cookie is cleared.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Acknowledgement body (`{\"ok\": true}`) for actions with nothing else to report.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`: a failure answers with an error status instead.",
      "example": true
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The session could not be updated.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

- `200` — What the consent page shows. Body: [`BranchConsentPage`](schemas.md#branchconsentpage) (`application/json`)
  - `library_name` *string*, **required** — This library's name.
  - `branch_name` *string*, **required** — The name the branch asks to join as.
  - `redirect_origin` *string*, **required** — Origin of the branch's redirect URI, for the page to name.
  - `enrollment` *boolean*, **required** — Whether this library lets new branches join right now.
  - `user` *[`ConsentingUser`](schemas.md#consentinguser)*, **required** — The signed-in user the branch would belong to.
  - `connection` *[`ConnectionChoice`](schemas.md#connectionchoice)*, **required** — The connection types the page may offer.
  - `connections` *array of [`ConsentConnection`](schemas.md#consentconnection)*, **required** — The user's store connections here, and whether each would travel to the branch if the user chooses to bring them.
- `400` — The authorize parameters are invalid (`invalid_request`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is not a library user (`user_required`; an operator session counts only while impersonating), the account is inactive (`user_inactive`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `503` — The library could not be read; try again (`unavailable`), or this library's cluster row or origin is not available yet (`cluster_unavailable`, `origin_unavailable`). The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
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

Status `200`: What the consent page shows.

```json
{
  "library_name": "books.example.com",
  "branch_name": "Living-room NAS",
  "redirect_origin": "http://127.0.0.1:8787",
  "enrollment": true,
  "user": {
    "id": 7,
    "display_name": "Sam"
  },
  "connection": {
    "choice": false,
    "default": "branch"
  },
  "connections": [
    {
      "source": "libro",
      "account_id": "sam@example.com",
      "label": "Sam",
      "travels": true
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/auth/branch-consent`: what the consent page shows.",
  "required": [
    "library_name",
    "branch_name",
    "redirect_origin",
    "enrollment",
    "user",
    "connection",
    "connections"
  ],
  "properties": {
    "library_name": {
      "type": "string",
      "description": "This library's name.",
      "example": "books.example.com"
    },
    "branch_name": {
      "type": "string",
      "description": "The name the branch asks to join as.",
      "example": "Living-room NAS"
    },
    "redirect_origin": {
      "type": "string",
      "description": "Origin of the branch's redirect URI, for the page to name.",
      "example": "http://127.0.0.1:8787"
    },
    "enrollment": {
      "type": "boolean",
      "description": "Whether this library lets new branches join right now.",
      "example": true
    },
    "user": {
      "type": "object",
      "description": "The signed-in user the branch would belong to.",
      "required": [
        "id"
      ],
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64",
          "description": "`users.id`.",
          "example": 7
        },
        "display_name": {
          "type": [
            "string",
            "null"
          ],
          "description": "Display name, when set.",
          "example": "Sam"
        }
      }
    },
    "connection": {
      "anyOf": [
        {
          "type": "object",
          "description": "Only a branch (the user is not an Owner).",
          "required": [
            "choice",
            "default"
          ],
          "properties": {
            "choice": {
              "type": "boolean",
              "description": "Always `false`.",
              "example": false
            },
            "default": {
              "type": "string",
              "description": "Always `branch`.",
              "example": "branch"
            }
          }
        },
        {
          "type": "object",
          "description": "An Owner may pick clerk, when it is available.",
          "required": [
            "choice",
            "default",
            "clerk_available",
            "step_up"
          ],
          "properties": {
            "choice": {
              "type": "boolean",
              "description": "Always `true`.",
              "example": true
            },
            "default": {
              "type": "string",
              "description": "Always `branch`.",
              "example": "branch"
            },
            "clerk_available": {
              "type": "boolean",
              "description": "Whether clerk may be chosen right now.",
              "example": false
            },
            "clerk_reason": {
              "type": [
                "string",
                "null"
              ],
              "description": "Why clerk is shown disabled, when it is.",
              "example": "The joining server already holds library data of its own (or did not offer it), so it can only join as a branch."
            },
            "step_up": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "The second factors the Owner can step up with (`totp`, `passkey`).",
              "example": [
                "passkey"
              ]
            }
          }
        }
      ],
      "description": "The connection types the page may offer."
    },
    "connections": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One of the consenting user's store connections, as the consent page\nlists it.",
        "required": [
          "source",
          "account_id",
          "travels"
        ],
        "properties": {
          "source": {
            "type": "string",
            "description": "Display alias of the storefront.",
            "example": "libro"
          },
          "account_id": {
            "type": "string",
            "description": "Store account id.",
            "example": "sam@example.com"
          },
          "label": {
            "type": [
              "string",
              "null"
            ],
            "description": "The account's display label, when it has one.",
            "example": "Sam"
          },
          "travels": {
            "type": "boolean",
            "description": "True when the storefront lets its login be used in more than one\nplace, so the connection travels; false when it keeps one login\nthat only this library refreshes.",
            "example": true
          }
        }
      },
      "description": "The user's store connections here, and whether each would travel\nto the branch if the user chooses to bring them."
    }
  }
}
```

Status `400`: The authorize parameters are invalid (`invalid_request`).

```json
{
  "detail": "the branch needs a name of 1 to 64 characters",
  "error": "invalid_request"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error body with a machine slug and a longer explanation (`detail`).",
  "required": [
    "error",
    "detail"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug clients branch on.",
      "example": "invalid_request"
    },
    "detail": {
      "type": "string",
      "description": "What went wrong, for the person reading the answer.",
      "example": "the redirect URI must be loopback or https"
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller is not a library user (`user_required`; an operator session counts only while impersonating), the account is inactive (`user_inactive`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`).

```json
{
  "error": "mfa_enrollment_required",
  "message": "This server requires a passkey or authenticator app. Set one up to continue, or log out and finish later.",
  "status": 403
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "The session middleware's standard body (`error`, `message`, `status`).",
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
    },
    {
      "type": "object",
      "description": "A slug or a sentence in `error`.",
      "required": [
        "error"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "What went wrong, as the handler describes it.",
          "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
        }
      }
    },
    {
      "type": "object",
      "description": "A slug in `error` and a sentence in `detail`.",
      "required": [
        "error",
        "detail"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "Stable machine slug clients branch on.",
          "example": "invalid_request"
        },
        "detail": {
          "type": "string",
          "description": "What went wrong, for the person reading the answer.",
          "example": "the redirect URI must be loopback or https"
        }
      }
    }
  ],
  "description": "An error answer in one of several shapes: the standard body, `error` alone, or `error` with `detail`; the route's examples name each."
}
```

Status `503`: The library could not be read; try again (`unavailable`), or this library's cluster row or origin is not available yet (`cluster_unavailable`, `origin_unavailable`). The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "cluster_unavailable"
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "The session middleware's standard body (`error`, `message`, `status`).",
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
    },
    {
      "type": "object",
      "description": "A slug or a sentence in `error`.",
      "required": [
        "error"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "What went wrong, as the handler describes it.",
          "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
        }
      }
    },
    {
      "type": "object",
      "description": "A slug in `error` and a sentence in `detail`.",
      "required": [
        "error",
        "detail"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "Stable machine slug clients branch on.",
          "example": "invalid_request"
        },
        "detail": {
          "type": "string",
          "description": "What went wrong, for the person reading the answer.",
          "example": "the redirect URI must be loopback or https"
        }
      }
    }
  ],
  "description": "An error answer in one of several shapes: the standard body, `error` alone, or `error` with `detail`; the route's examples name each."
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

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
- `bring_connections` *boolean* — True to send the user's store connections that may travel to the branch when it joins. Ignored for a clerk. Defaults to false.

### Responses

- `200` — `{redirect}` back to the branch. Body: [`ConsentRedirect`](schemas.md#consentredirect) (`application/json`)
  - `redirect` *string*, **required** — Where to send the browser: the branch's callback with a code, or with `error=access_denied`.
- `400` — The body or its authorize parameters are invalid (`invalid_request`), or the joining server did not offer a clerk join (`clerk_not_offered`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is not an active library user (`user_required`, `user_inactive`), this library is not accepting branches (`enrollment_disabled`), a clerk join needs an Owner (`owner_required`) with a second factor (`step_up_unavailable`) that was confirmed (`step_up_required`) and passed (`step_up_failed`), the connection failed the transport policy (`transport_insecure`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`NodeRefusal`](schemas.md#noderefusal) (`application/json`)
- `404` — The node API is not mounted on this listener (`node_api_unmounted`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — This library's database cannot be shared with a clerk (`database_not_shareable`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `413` — The body is larger than 8 KiB. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request is not `application/json`. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `429` — Too many wrong step-up codes; wait a few minutes (`step_up_locked`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `503` — The library could not be read; try again (`unavailable`), or this library's cluster row or origin is not available yet (`cluster_unavailable`, `origin_unavailable`). The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/branch-consent' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"client_id": "bookclerk-branch", "redirect_uri": "http://127.0.0.1:41923/enroll/callback", "response_type": "code", "code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM", "approve": true}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/branch-consent' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"client_id": "bookclerk-branch", "redirect_uri": "http://127.0.0.1:41923/enroll/callback", "response_type": "code", "code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM", "approve": true}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/branch-consent",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"client_id": "bookclerk-branch", "redirect_uri": "http://127.0.0.1:41923/enroll/callback", "response_type": "code", "code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM", "approve": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/branch-consent", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"client_id": "bookclerk-branch", "redirect_uri": "http://127.0.0.1:41923/enroll/callback", "response_type": "code", "code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM", "approve": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/branch-consent")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"client_id": "bookclerk-branch", "redirect_uri": "http://127.0.0.1:41923/enroll/callback", "response_type": "code", "code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM", "approve": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: `{redirect}` back to the branch.

```json
{
  "redirect": "http://127.0.0.1:8787/api/config/join/callback?code=8f2c…&state=xyz"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after a branch consent was answered.",
  "required": [
    "redirect"
  ],
  "properties": {
    "redirect": {
      "type": "string",
      "description": "Where to send the browser: the branch's callback with a code, or with `error=access_denied`.",
      "example": "http://127.0.0.1:8787/api/config/join/callback?code=8f2c…&state=xyz"
    }
  }
}
```

Status `400`: The body or its authorize parameters are invalid (`invalid_request`), or the joining server did not offer a clerk join (`clerk_not_offered`).

```json
{
  "detail": "connection must be branch or clerk",
  "error": "invalid_request"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error body with a machine slug and a longer explanation (`detail`).",
  "required": [
    "error",
    "detail"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug clients branch on.",
      "example": "invalid_request"
    },
    "detail": {
      "type": "string",
      "description": "What went wrong, for the person reading the answer.",
      "example": "the redirect URI must be loopback or https"
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller is not an active library user (`user_required`, `user_inactive`), this library is not accepting branches (`enrollment_disabled`), a clerk join needs an Owner (`owner_required`) with a second factor (`step_up_unavailable`) that was confirmed (`step_up_required`) and passed (`step_up_failed`), the connection failed the transport policy (`transport_insecure`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "detail": "this library isn't accepting new branches",
  "error": "enrollment_disabled"
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "The connection failed the transport policy.",
      "required": [
        "error",
        "reason"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "Always `transport_insecure`.",
          "example": "transport_insecure"
        },
        "reason": {
          "type": "string",
          "description": "The transport check that refused the connection (`plain_http`,\n`unknown_peer`, `untrusted_forwarding`, `proxy_proto_missing`,\n`proxy_proto_ambiguous`, `proxy_proto_not_https`,\n`public_origin_not_https`, `listener_mode_mismatch`,\n`node_api_misconfigured`).",
          "example": "plain_http"
        }
      }
    },
    {
      "type": "object",
      "description": "The session middleware's standard body (`error`, `message`, `status`).",
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
    },
    {
      "type": "object",
      "description": "The handler refused with a slug in `error`.",
      "required": [
        "error"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "What went wrong, as the handler describes it.",
          "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
        }
      }
    },
    {
      "type": "object",
      "description": "The handler refused with a slug in `error` and a sentence in `detail`.",
      "required": [
        "error",
        "detail"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "Stable machine slug clients branch on.",
          "example": "invalid_request"
        },
        "detail": {
          "type": "string",
          "description": "What went wrong, for the person reading the answer.",
          "example": "the redirect URI must be loopback or https"
        }
      }
    }
  ],
  "description": "A `403` from a node API route: the transport policy's refusal, the standard body, or the handler's own error; the route's examples name each."
}
```

Status `404`: The node API is not mounted on this listener (`node_api_unmounted`).

```json
{
  "error": "node_api_unmounted"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `409`: This library's database cannot be shared with a clerk (`database_not_shareable`).

```json
{
  "detail": "This library keeps its database in a SQLite file that only this server can open. A clerk needs a PostgreSQL or Cloudflare D1 database.",
  "error": "database_not_shareable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error body with a machine slug and a longer explanation (`detail`).",
  "required": [
    "error",
    "detail"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug clients branch on.",
      "example": "invalid_request"
    },
    "detail": {
      "type": "string",
      "description": "What went wrong, for the person reading the answer.",
      "example": "the redirect URI must be loopback or https"
    }
  }
}
```

Status `413`: The body is larger than 8 KiB.

```json
{
  "error": "request too large"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `415`: The request is not `application/json`.

```json
{
  "error": "expected application/json"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `429`: Too many wrong step-up codes; wait a few minutes (`step_up_locked`).

```json
{
  "detail": "too many wrong codes; wait a few minutes",
  "error": "step_up_locked"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error body with a machine slug and a longer explanation (`detail`).",
  "required": [
    "error",
    "detail"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug clients branch on.",
      "example": "invalid_request"
    },
    "detail": {
      "type": "string",
      "description": "What went wrong, for the person reading the answer.",
      "example": "the redirect URI must be loopback or https"
    }
  }
}
```

Status `503`: The library could not be read; try again (`unavailable`), or this library's cluster row or origin is not available yet (`cluster_unavailable`, `origin_unavailable`). The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "cluster_unavailable"
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "The session middleware's standard body (`error`, `message`, `status`).",
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
    },
    {
      "type": "object",
      "description": "A slug or a sentence in `error`.",
      "required": [
        "error"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "What went wrong, as the handler describes it.",
          "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
        }
      }
    },
    {
      "type": "object",
      "description": "A slug in `error` and a sentence in `detail`.",
      "required": [
        "error",
        "detail"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "Stable machine slug clients branch on.",
          "example": "invalid_request"
        },
        "detail": {
          "type": "string",
          "description": "What went wrong, for the person reading the answer.",
          "example": "the redirect URI must be loopback or https"
        }
      }
    }
  ],
  "description": "An error answer in one of several shapes: the standard body, `error` alone, or `error` with `detail`; the route's examples name each."
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/auth/branch-consent/step-up`

`POST /api/auth/branch-consent/step-up`: starts a passkey step-up for a
clerk consent (an Owner with a passkey). Answers the WebAuthn request
options with a `challenge_id` to send back in the consent's `step_up`.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Responses

- `200` — Passkey request options. Body: [`PasskeyRequestOptions`](schemas.md#passkeyrequestoptions) (`application/json`)
  - `challenge_id` *string*, **required** — Server-issued challenge id to send back with the assertion; consumed once, five-minute lifetime.
  - `publicKey` *any JSON*, **required** — W3C PublicKeyCredentialRequestOptions (challenge, relying party id, allowed credentials), binary fields base64url-encoded.
  - `mediation` *string or null* — Credential mediation the daemon asks the browser for; absent unless conditional mediation was requested.
- `400` — This server's public origin is not a URL WebAuthn can use, so the passkey check could not start (`unavailable`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is not an active library user (`user_required`, `user_inactive`), not an Owner (`owner_required`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `404` — The account has no passkey (`no_passkey`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `500` — The passkeys could not be read, or the challenge could not be created or stored (`unavailable`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `503` — The user record could not be read (`unavailable`); try again. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
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

Status `200`: Passkey request options.

```json
{
  "challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f",
  "publicKey": {
    "allowCredentials": [
      {
        "id": "pQECAyYgASFYIE",
        "type": "public-key"
      }
    ],
    "challenge": "k9u2Xo3cZ1Qp7mT4vW8yA2bC5dE6fG7hI8jK9lM0nO1",
    "rpId": "books.example.com",
    "timeout": 60000,
    "userVerification": "preferred"
  },
  "mediation": "conditional"
}
```

Schema:

```json
{
  "type": "object",
  "description": "WebAuthn request options for `navigator.credentials.get()`, plus the challenge id for the finish step.",
  "required": [
    "challenge_id",
    "publicKey"
  ],
  "properties": {
    "challenge_id": {
      "type": "string",
      "description": "Server-issued challenge id to send back with the assertion; consumed once, five-minute lifetime.",
      "example": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f"
    },
    "publicKey": {
      "description": "W3C PublicKeyCredentialRequestOptions (challenge, relying party id, allowed credentials), binary fields base64url-encoded.",
      "examples": [
        {
          "allowCredentials": [
            {
              "id": "pQECAyYgASFYIE",
              "type": "public-key"
            }
          ],
          "challenge": "k9u2Xo3cZ1Qp7mT4vW8yA2bC5dE6fG7hI8jK9lM0nO1",
          "rpId": "books.example.com",
          "timeout": 60000,
          "userVerification": "preferred"
        }
      ]
    },
    "mediation": {
      "type": [
        "string",
        "null"
      ],
      "description": "Credential mediation the daemon asks the browser for; absent unless\nconditional mediation was requested.",
      "example": "conditional"
    }
  }
}
```

Status `400`: This server's public origin is not a URL WebAuthn can use, so the passkey check could not start (`unavailable`).

```json
{
  "detail": "could not start the passkey check",
  "error": "unavailable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error body with a machine slug and a longer explanation (`detail`).",
  "required": [
    "error",
    "detail"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug clients branch on.",
      "example": "invalid_request"
    },
    "detail": {
      "type": "string",
      "description": "What went wrong, for the person reading the answer.",
      "example": "the redirect URI must be loopback or https"
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller is not an active library user (`user_required`, `user_inactive`), not an Owner (`owner_required`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "mfa_enrollment_required",
  "message": "This server requires a passkey or authenticator app. Set one up to continue, or log out and finish later.",
  "status": 403
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "The session middleware's standard body (`error`, `message`, `status`).",
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
    },
    {
      "type": "object",
      "description": "A slug or a sentence in `error`.",
      "required": [
        "error"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "What went wrong, as the handler describes it.",
          "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
        }
      }
    },
    {
      "type": "object",
      "description": "A slug in `error` and a sentence in `detail`.",
      "required": [
        "error",
        "detail"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "Stable machine slug clients branch on.",
          "example": "invalid_request"
        },
        "detail": {
          "type": "string",
          "description": "What went wrong, for the person reading the answer.",
          "example": "the redirect URI must be loopback or https"
        }
      }
    }
  ],
  "description": "An error answer in one of several shapes: the standard body, `error` alone, or `error` with `detail`; the route's examples name each."
}
```

Status `404`: The account has no passkey (`no_passkey`).

```json
{
  "detail": "this account has no passkey",
  "error": "no_passkey"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error body with a machine slug and a longer explanation (`detail`).",
  "required": [
    "error",
    "detail"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug clients branch on.",
      "example": "invalid_request"
    },
    "detail": {
      "type": "string",
      "description": "What went wrong, for the person reading the answer.",
      "example": "the redirect URI must be loopback or https"
    }
  }
}
```

Status `500`: The passkeys could not be read, or the challenge could not be created or stored (`unavailable`).

```json
{
  "detail": "could not start the passkey check",
  "error": "unavailable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error body with a machine slug and a longer explanation (`detail`).",
  "required": [
    "error",
    "detail"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug clients branch on.",
      "example": "invalid_request"
    },
    "detail": {
      "type": "string",
      "description": "What went wrong, for the person reading the answer.",
      "example": "the redirect URI must be loopback or https"
    }
  }
}
```

Status `503`: The user record could not be read (`unavailable`); try again. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "detail": "try again",
  "error": "unavailable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error body with a machine slug and a longer explanation (`detail`).",
  "required": [
    "error",
    "detail"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug clients branch on.",
      "example": "invalid_request"
    },
    "detail": {
      "type": "string",
      "description": "What went wrong, for the person reading the answer.",
      "example": "the redirect URI must be loopback or https"
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

## `POST /api/auth/logout`

Revokes hashed operator and portal sessions and clears both cookies.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Signed out; both session cookies are cleared. Body: [`OkResponse`](schemas.md#okresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`: a failure answers with an error status instead.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `200`: Signed out; both session cookies are cleared.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Acknowledgement body (`{\"ok\": true}`) for actions with nothing else to report.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`: a failure answers with an error status instead.",
      "example": true
    }
  }
}
```

Status `401`: The caller is not signed in, or the credential is wrong.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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

## `GET /api/auth/sessions`

List sessions for the current principal only (not every operator session).

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The caller's sessions. Body: [`SessionsResponse`](schemas.md#sessionsresponse) (`application/json`)
  - `sessions` *array of [`SessionView`](schemas.md#sessionview)*, **required** — The caller's sessions, newest first; empty for a Bearer-only caller, who has no session rows.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The impersonated user has no sign-in identity. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The sessions could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `200`: The caller's sessions.

```json
{
  "sessions": [
    {
      "id": 12,
      "created_at": "2026-10-28T19:02:11Z",
      "last_used_at": "2026-10-29T08:15:00Z",
      "expires_at": "2026-11-04T19:02:11Z",
      "elevated": false,
      "impersonating_user_id": 7,
      "is_current": false,
      "client_label": "Windows",
      "device_type": "desktop",
      "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Firefox/134.0",
      "kind": "operator"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/auth/sessions`.",
  "required": [
    "sessions"
  ],
  "properties": {
    "sessions": {
      "type": "array",
      "items": {
        "oneOf": [
          {
            "allOf": [
              {
                "type": "object",
                "description": "An operator session (`kind` is `operator`).",
                "required": [
                  "id",
                  "created_at",
                  "expires_at",
                  "elevated",
                  "is_current"
                ],
                "properties": {
                  "id": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Session id, for `DELETE /api/auth/sessions/{id}`.",
                    "example": 12
                  },
                  "created_at": {
                    "type": "string",
                    "description": "RFC 3339 time the session was created.",
                    "example": "2026-10-28T19:02:11Z"
                  },
                  "last_used_at": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "RFC 3339 time of the last authenticated use, when any.",
                    "example": "2026-10-29T08:15:00Z"
                  },
                  "expires_at": {
                    "type": "string",
                    "description": "RFC 3339 expiry.",
                    "example": "2026-11-04T19:02:11Z"
                  },
                  "elevated": {
                    "type": "boolean",
                    "description": "Whether an Owner elevated into this session (rather than the operator token).",
                    "example": false
                  },
                  "impersonating_user_id": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "User the session is impersonating, when it is. Never set on the\nsession making the request: while that session impersonates, the\nlist shows the target's portal sessions instead.",
                    "example": 7
                  },
                  "is_current": {
                    "type": "boolean",
                    "description": "Whether this is the session making the request.",
                    "example": false
                  },
                  "client_label": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Best-effort OS or client label from the User-Agent.",
                    "example": "Windows"
                  },
                  "device_type": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Best-effort device class (`desktop`, `mobile`, `tablet`, `api`).",
                    "example": "desktop"
                  },
                  "user_agent": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Raw User-Agent captured when the session was created.",
                    "example": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Firefox/134.0"
                  }
                }
              },
              {
                "type": "object",
                "required": [
                  "kind"
                ],
                "properties": {
                  "kind": {
                    "type": "string",
                    "enum": [
                      "operator"
                    ]
                  }
                }
              }
            ],
            "description": "An operator session (`kind` is `operator`)."
          },
          {
            "allOf": [
              {
                "type": "object",
                "description": "A portal session (`kind` is `portal`).",
                "required": [
                  "id",
                  "created_at",
                  "expires_at",
                  "is_current"
                ],
                "properties": {
                  "id": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Session id, for `DELETE /api/auth/sessions/{id}`.",
                    "example": 31
                  },
                  "created_at": {
                    "type": "string",
                    "description": "RFC 3339 time the session was created.",
                    "example": "2026-10-28T19:02:11Z"
                  },
                  "expires_at": {
                    "type": "string",
                    "description": "RFC 3339 expiry.",
                    "example": "2026-11-27T19:02:11Z"
                  },
                  "last_used_at": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "RFC 3339 time of the last authenticated use, when any.",
                    "example": "2026-10-29T08:15:00Z"
                  },
                  "is_current": {
                    "type": "boolean",
                    "description": "Whether this is the session making the request.",
                    "example": true
                  },
                  "client_label": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Best-effort OS or client label from the User-Agent.",
                    "example": "Android"
                  },
                  "device_type": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Best-effort device class (`desktop`, `mobile`, `tablet`, `api`).",
                    "example": "mobile"
                  },
                  "user_agent": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Raw User-Agent captured when the session was created.",
                    "example": "Mozilla/5.0 (Linux; Android 15) Chrome/131.0 Mobile"
                  }
                }
              },
              {
                "type": "object",
                "required": [
                  "kind"
                ],
                "properties": {
                  "kind": {
                    "type": "string",
                    "enum": [
                      "portal"
                    ]
                  }
                }
              }
            ],
            "description": "A portal session (`kind` is `portal`)."
          }
        ],
        "description": "One session in the caller's list, told apart by `kind`: an operator\nsession or a portal (user) session."
      },
      "description": "The caller's sessions, newest first; empty for a Bearer-only caller, who has no session rows."
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: The impersonated user has no sign-in identity.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The sessions could not be read.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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

## `DELETE /api/auth/sessions/{id}`

Revoke a session by id. Operators may revoke any operator session;
portal users may only revoke their own.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — Session id.

### Responses

- `200` — The session was revoked. Body: [`OkResponse`](schemas.md#okresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`: a failure answers with an error status instead.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such session, or it belongs to someone else. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The session could not be deleted. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `200`: The session was revoked.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Acknowledgement body (`{\"ok\": true}`) for actions with nothing else to report.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`: a failure answers with an error status instead.",
      "example": true
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: No such session, or it belongs to someone else.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The session could not be deleted.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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

- `200` — The profile after the change. Body: [`ProfileChanged`](schemas.md#profilechanged) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `user` *[`AuthMeUser`](schemas.md#authmeuser)*, **required** — The profile after the change, as `GET /api/auth/me` reports it.
- `400` — A value is invalid (`display_name_too_long`, `invalid_email`, `invalid_avatar_source`), or the body sets none of the three fields (the standard error body). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user, is a plain operator session, or has no first-party user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The caller's user no longer exists. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The email is used by another account (`email_in_use`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `200`: The profile after the change.

```json
{
  "ok": true,
  "user": {
    "id": 7,
    "role": "administrator",
    "display_name": "Sam",
    "email": "sam@example.com",
    "has_password": true,
    "has_avatar": false,
    "avatar_source": "auto",
    "gravatar_hash": "0bc83cb571cd1c50ba6f3e8a78ef1346b6b31bd4d35b8d3dc1f0c8a3b4a9f7e2",
    "sso_pictures": [
      {
        "identity_id": 3,
        "provider": "oidc:google",
        "picture_url": "https://lh3.googleusercontent.com/a/example=s96-c",
        "picture_path": "/api/images/aHR0cHM6Ly9saDMuZ29vZ2xldXNlcmNvbnRlbnQuY29tL2EvZXhhbXBsZT1zOTYtYw.N14rqh66Bok9-eCqJKBNoQ",
        "last_used_at": "2026-10-29T08:15:00Z"
      }
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after `PATCH /api/auth/profile`.",
  "required": [
    "ok",
    "user"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "user": {
      "type": "object",
      "description": "The profile after the change, as `GET /api/auth/me` reports it.",
      "required": [
        "id",
        "role",
        "has_password",
        "has_avatar",
        "avatar_source"
      ],
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64",
          "description": "First-party `users.id`.",
          "example": 7
        },
        "role": {
          "type": "string",
          "description": "Role string (`owner`, `administrator`, `member`).",
          "example": "administrator"
        },
        "display_name": {
          "type": [
            "string",
            "null"
          ],
          "description": "Optional human-readable name from the user row.",
          "example": "Sam"
        },
        "email": {
          "type": [
            "string",
            "null"
          ],
          "description": "Optional email; omitted from JSON when unset.",
          "example": "sam@example.com"
        },
        "has_password": {
          "type": "boolean",
          "description": "True when a local password hash is stored (invite users may be false).",
          "example": true
        },
        "has_avatar": {
          "type": "boolean",
          "description": "True when a JPEG avatar is stored under `files_dir/avatars/{id}.*`.",
          "example": false
        },
        "avatar_source": {
          "type": "string",
          "description": "Selected picture source (`auto`, `monogram`, `gravatar`, `upload`, `sso:{id}`).",
          "example": "auto"
        },
        "gravatar_hash": {
          "type": [
            "string",
            "null"
          ],
          "description": "SHA-256 hex of the contact email for Gravatar, when an email is set.",
          "example": "0bc83cb571cd1c50ba6f3e8a78ef1346b6b31bd4d35b8d3dc1f0c8a3b4a9f7e2"
        },
        "sso_pictures": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "HTTPS picture from a linked identity provider.",
            "required": [
              "identity_id",
              "provider",
              "picture_url"
            ],
            "properties": {
              "identity_id": {
                "type": "integer",
                "format": "int64",
                "description": "`portal_identities.id`.",
                "example": 3
              },
              "provider": {
                "type": "string",
                "description": "Identity-broker id (`oidc:google`, …).",
                "example": "oidc:google"
              },
              "picture_url": {
                "type": "string",
                "description": "HTTPS URL to display.",
                "example": "https://lh3.googleusercontent.com/a/example=s96-c"
              },
              "picture_path": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Same-origin 128 px copy of `picture_url` for the web app to show,\nwhen this node can sign it.",
                "example": "/api/images/aHR0cHM6Ly9saDMuZ29vZ2xldXNlcmNvbnRlbnQuY29tL2EvZXhhbXBsZT1zOTYtYw.N14rqh66Bok9-eCqJKBNoQ"
              },
              "last_used_at": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "RFC 3339 last portal-session activity for this identity, when any.",
                "example": "2026-10-29T08:15:00Z"
              }
            }
          },
          "description": "IdP-supplied pictures the user can pick."
        }
      }
    }
  }
}
```

Status `400`: A value is invalid (`display_name_too_long`, `invalid_email`, `invalid_avatar_source`), or the body sets none of the three fields (the standard error body). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "display_name_too_long",
  "message": "Display name must be 80 characters or fewer.",
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The session is impersonating a user, is a plain operator session, or has no first-party user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: The caller's user no longer exists.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `409`: The email is used by another account (`email_in_use`).

```json
{
  "error": "email_in_use",
  "message": "That email is already used by another account.",
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: display_name: invalid type: integer `1`, expected a string at line 1 column 17
```

Schema:

A `text/plain` body: a string.

Status `500`: The library could not be read or written.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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

## `PUT /api/auth/profile/avatar`

Accepts a JPEG/PNG/WebP upload and stores it under `files_dir/avatars`.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`image/*`: [`BinaryBody`](schemas.md#binarybody)

### Responses

- `200` — The avatar was stored. Body: [`AvatarChanged`](schemas.md#avatarchanged) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `has_avatar` *boolean*, **required** — Whether an uploaded picture is stored now.
  - `avatar_source` *string or null* — Present only after an upload, always `upload`; absent after a removal (the stored source is then unchanged unless it was `upload`).
- `400` — The body is empty or not a JPEG, PNG or WebP image. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user, is a plain operator session, or has no first-party user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `413` — The image is larger than 1.5 MB. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The files directory is unknown, or the image could not be written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `200`: The avatar was stored.

```json
{
  "ok": true,
  "has_avatar": true,
  "avatar_source": "upload"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after the caller's avatar was stored or removed.",
  "required": [
    "ok",
    "has_avatar"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "has_avatar": {
      "type": "boolean",
      "description": "Whether an uploaded picture is stored now.",
      "example": true
    },
    "avatar_source": {
      "type": [
        "string",
        "null"
      ],
      "description": "Present only after an upload, always `upload`; absent after a removal\n(the stored source is then unchanged unless it was `upload`).",
      "example": "upload"
    }
  }
}
```

Status `400`: The body is empty or not a JPEG, PNG or WebP image.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The session is impersonating a user, is a plain operator session, or has no first-party user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `413`: The image is larger than 1.5 MB.

```json
{
  "error": "payload_too_large",
  "message": "Payload Too Large",
  "status": 413
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The files directory is unknown, or the image could not be written.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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

## `DELETE /api/auth/profile/avatar`

Removes the caller's stored avatar, if any.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The avatar was removed. Body: [`AvatarChanged`](schemas.md#avatarchanged) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `has_avatar` *boolean*, **required** — Whether an uploaded picture is stored now.
  - `avatar_source` *string or null* — Present only after an upload, always `upload`; absent after a removal (the stored source is then unchanged unless it was `upload`).
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user, is a plain operator session, or has no first-party user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The files directory is unknown. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `200`: The avatar was removed.

```json
{
  "has_avatar": false,
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after the caller's avatar was stored or removed.",
  "required": [
    "ok",
    "has_avatar"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "has_avatar": {
      "type": "boolean",
      "description": "Whether an uploaded picture is stored now.",
      "example": true
    },
    "avatar_source": {
      "type": [
        "string",
        "null"
      ],
      "description": "Present only after an upload, always `upload`; absent after a removal\n(the stored source is then unchanged unless it was `upload`).",
      "example": "upload"
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The session is impersonating a user, is a plain operator session, or has no first-party user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The files directory is unknown.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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

## `GET /api/auth/oidc/providers`

Lists enabled IdPs for the login picker (id and display name only).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — Enabled identity providers (id and name). Body: [`OidcProvidersResponse`](schemas.md#oidcprovidersresponse) (`application/json`)
  - `enabled` *boolean*, **required** — Whether the OIDC broker is switched on at all.
  - `providers` *array of [`SigninOidcProvider`](schemas.md#signinoidcprovider)*, **required** — Providers enabled in configuration, in configuration order (empty while the broker is off).
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

### Example response

Status `200`: Enabled identity providers (id and name).

```json
{
  "enabled": true,
  "providers": [
    {
      "id": "google",
      "name": "Google",
      "preset": "google"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/auth/oidc/providers`.",
  "required": [
    "enabled",
    "providers"
  ],
  "properties": {
    "enabled": {
      "type": "boolean",
      "description": "Whether the OIDC broker is switched on at all.",
      "example": true
    },
    "providers": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One OIDC identity provider offered on the sign-in page.",
        "required": [
          "id",
          "name"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Provider id from `[auth.oidc.providers]`, used in `/api/auth/oidc/{id}/start`.",
            "example": "google"
          },
          "name": {
            "type": "string",
            "description": "Button label.",
            "example": "Google"
          },
          "preset": {
            "type": [
              "string",
              "null"
            ],
            "description": "Known social preset the client may style (`google`, `github`, …), when the provider matches one.",
            "example": "google"
          }
        }
      },
      "description": "Providers enabled in configuration, in configuration order (empty while the broker is off)."
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

## `GET /api/auth/oidc/identities`

Lists OIDC portal identities for the cookie-authenticated caller.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — The caller's linked identities. Body: [`LinkedIdentitiesResponse`](schemas.md#linkedidentitiesresponse) (`application/json`)
  - `identities` *array of [`LinkedIdentity`](schemas.md#linkedidentity)*, **required** — Every sign-in identity linked to the caller's user, the local one included.
- `401` — The caller is not signed in, or has no first-party user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The identities could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: The caller's linked identities.

```json
{
  "identities": [
    {
      "provider": "oidc:google",
      "external_user_id": "118234509876543210987",
      "label": "sam@example.com"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/auth/oidc/identities`.",
  "required": [
    "identities"
  ],
  "properties": {
    "identities": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One sign-in identity linked to a user.",
        "required": [
          "provider",
          "external_user_id"
        ],
        "properties": {
          "provider": {
            "type": "string",
            "description": "Identity-broker id (`local`, `oidc:google`, …).",
            "example": "oidc:google"
          },
          "external_user_id": {
            "type": "string",
            "description": "Provider-stable subject.",
            "example": "118234509876543210987"
          },
          "label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Operator-facing label, when the provider supplied one.",
            "example": "sam@example.com"
          }
        }
      },
      "description": "Every sign-in identity linked to the caller's user, the local one included."
    }
  }
}
```

Status `401`: The caller is not signed in, or has no first-party user.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The identities could not be read.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `GET /api/auth/oidc/login`

Starts an OIDC login authorize redirect, honoring login throttle.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Parameters

- `provider` (query) *string* — IdP id to start; omitted when the caller has only one enabled provider.

### Responses

- `307` — Redirects to the identity provider, with the transaction cookie set. No body.
- `400` — No provider was named. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — OIDC sign-in is disabled, or no enabled provider has this id. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `429` — Too many attempts; retry after `Retry-After` seconds. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The transaction could not be stored. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `502` — The provider's discovery document could not be fetched. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `307`: Redirects to the identity provider, with the transaction cookie set. No body.

Status `400`: No provider was named. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `404`: OIDC sign-in is disabled, or no enabled provider has this id. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
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

Status `429`: Too many attempts; retry after `Retry-After` seconds. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

```json
{
  "error": "too_many_requests",
  "message": "Too many requests. Try again shortly.",
  "status": 429
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

Status `500`: The transaction could not be stored. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `502`: The provider's discovery document could not be fetched. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

```json
{
  "error": "bad_gateway",
  "message": "An upstream service failed.",
  "status": 502
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

## `GET /api/auth/oidc/elevate`

Starts an Owner re-auth elevate flow against a linked IdP (`prompt=login`).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

- `provider` (query) *string* — IdP id to start; omitted when the caller has only one enabled provider.

### Responses

- `307` — Redirects to the identity provider for re-authentication, with the transaction cookie set. No body.
- `400` — No provider was named and the Owner has no linked OIDC identity. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `401` — The caller is not signed in. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — The caller is not an Owner. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — OIDC sign-in is disabled, or no enabled provider has this id. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `429` — Too many attempts; retry after `Retry-After` seconds. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The linked identities or the transaction could not be read or stored. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `502` — The provider's discovery document could not be fetched. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `307`: Redirects to the identity provider for re-authentication, with the transaction cookie set. No body.

Status `400`: No provider was named and the Owner has no linked OIDC identity. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `401`: The caller is not signed in. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `403`: The caller is not an Owner. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `404`: OIDC sign-in is disabled, or no enabled provider has this id. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
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

Status `429`: Too many attempts; retry after `Retry-After` seconds. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

```json
{
  "error": "too_many_requests",
  "message": "Too many requests. Try again shortly.",
  "status": 429
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

Status `500`: The linked identities or the transaction could not be read or stored. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `502`: The provider's discovery document could not be fetched. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

```json
{
  "error": "bad_gateway",
  "message": "An upstream service failed.",
  "status": 502
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

- `303` — Redirects into the app: to `/` signed in, to `/settings` after an elevation, or with `sso_error` naming why sign-in did not finish (`denied`, `csrf`, `expired`, `nonce`, `sub`, `disabled`, `no_role`, `conflict`, `mismatch`). No body.
- `400` — The code or the state is missing, or an elevation has no user. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — The elevated user is not an Owner. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — OIDC sign-in is disabled, the provider is gone, or the elevated user no longer exists. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The transaction, the identities or the user could not be read or stored. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `502` — The provider's discovery, token, keys or userinfo endpoint failed, or its tokens were unusable. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `303`: Redirects into the app: to `/` signed in, to `/settings` after an elevation, or with `sso_error` naming why sign-in did not finish (`denied`, `csrf`, `expired`, `nonce`, `sub`, `disabled`, `no_role`, `conflict`, `mismatch`). No body.

Status `400`: The code or the state is missing, or an elevation has no user. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `403`: The elevated user is not an Owner. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `404`: OIDC sign-in is disabled, the provider is gone, or the elevated user no longer exists. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
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

Status `500`: The transaction, the identities or the user could not be read or stored. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `502`: The provider's discovery, token, keys or userinfo endpoint failed, or its tokens were unusable. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

```json
{
  "error": "bad_gateway",
  "message": "An upstream service failed.",
  "status": 502
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

- `303` — Redirects into the app: to `/` signed in, to `/settings` after an elevation, or with `sso_error` naming why sign-in did not finish (`denied`, `csrf`, `expired`, `nonce`, `sub`, `disabled`, `no_role`, `conflict`, `mismatch`). No body.
- `400` — The code or the state is missing, or an elevation has no user. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — The elevated user is not an Owner. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — OIDC sign-in is disabled, the provider is gone, or the elevated user no longer exists. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The transaction, the identities or the user could not be read or stored. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `502` — The provider's discovery, token, keys or userinfo endpoint failed, or its tokens were unusable. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/html`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `303`: Redirects into the app: to `/` signed in, to `/settings` after an elevation, or with `sso_error` naming why sign-in did not finish (`denied`, `csrf`, `expired`, `nonce`, `sub`, `disabled`, `no_role`, `conflict`, `mismatch`). No body.

Status `400`: The code or the state is missing, or an elevation has no user. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `403`: The elevated user is not an Owner. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `404`: OIDC sign-in is disabled, the provider is gone, or the elevated user no longer exists. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
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

Status `500`: The transaction, the identities or the user could not be read or stored. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

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

Status `502`: The provider's discovery, token, keys or userinfo endpoint failed, or its tokens were unusable. Browsers get an HTML page; a request that asks for JSON (`Accept` or `Content-Type`) gets the standard error body.

```json
{
  "error": "bad_gateway",
  "message": "An upstream service failed.",
  "status": 502
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
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is neither the operator nor an Owner (`forbidden`), or must enroll a second factor first (`mfa_enrollment_required`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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
    "example.com"
  ],
  "public_origin": "https://books.example.com",
  "issuer_url": "https://books.example.com",
  "detected_origin": "http://localhost:8787",
  "discovery_url": "https://books.example.com/.well-known/openid-configuration",
  "authorization_endpoint": "https://books.example.com/oidc/authorize",
  "token_endpoint": "https://books.example.com/oidc/token",
  "userinfo_endpoint": "https://books.example.com/oidc/userinfo",
  "revocation_endpoint": "https://books.example.com/oidc/revoke",
  "callback_url": "https://books.example.com/api/auth/oidc/callback",
  "providers": [
    {
      "id": "google",
      "name": "Google",
      "preset": "google",
      "issuer": "https://accounts.google.com",
      "client_id": "123456789012-a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6.apps.googleusercontent.com",
      "scopes": [
        "openid",
        "profile",
        "email"
      ],
      "provision": "allowlist",
      "default_role": "member",
      "role_claim": "groups",
      "role_map": {},
      "link_by_email": true,
      "allowed_email_domains": [
        "example.com"
      ],
      "allowed_emails": [
        "dana@example.com"
      ],
      "allowed_subjects": [
        "248289761001"
      ],
      "apple_team_id": "A1B2C3D4E5",
      "apple_key_id": "K9L8M7N6P5",
      "has_client_secret": true,
      "has_apple_private_key": false,
      "secret_source": "store"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "Operator-facing OIDC broker snapshot; secret material is never included.",
  "required": [
    "enabled",
    "allowed_email_domains",
    "issuer_url",
    "detected_origin",
    "discovery_url",
    "authorization_endpoint",
    "token_endpoint",
    "userinfo_endpoint",
    "revocation_endpoint",
    "callback_url",
    "providers"
  ],
  "properties": {
    "enabled": {
      "type": "boolean",
      "description": "Whether the broker accepts new authorize starts.",
      "example": true
    },
    "allowed_email_domains": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Broker-wide email domain allowlist applied before per-provider rules.",
      "example": [
        "example.com"
      ]
    },
    "public_origin": {
      "type": [
        "string",
        "null"
      ],
      "description": "Configured `integrations.public_origin` (issuer / callback / invite base).",
      "example": "https://books.example.com"
    },
    "issuer_url": {
      "type": "string",
      "description": "Effective issuer origin (`public_origin`, else this request, else localhost).",
      "example": "https://books.example.com"
    },
    "detected_origin": {
      "type": "string",
      "description": "Origin detected from this request (loopback IPs rewritten to `localhost`).",
      "example": "http://localhost:8787"
    },
    "discovery_url": {
      "type": "string",
      "description": "Absolute OpenID discovery URL (`/.well-known/openid-configuration`).",
      "example": "https://books.example.com/.well-known/openid-configuration"
    },
    "authorization_endpoint": {
      "type": "string",
      "description": "Absolute authorization endpoint (`/oidc/authorize`) for Bookclerk-as-IdP.",
      "example": "https://books.example.com/oidc/authorize"
    },
    "token_endpoint": {
      "type": "string",
      "description": "Absolute token endpoint (`/oidc/token`) for Bookclerk-as-IdP.",
      "example": "https://books.example.com/oidc/token"
    },
    "userinfo_endpoint": {
      "type": "string",
      "description": "Absolute userinfo endpoint (`/oidc/userinfo`) for Bookclerk-as-IdP.",
      "example": "https://books.example.com/oidc/userinfo"
    },
    "revocation_endpoint": {
      "type": "string",
      "description": "Absolute revocation endpoint (`/oidc/revoke`) for Bookclerk-as-IdP.",
      "example": "https://books.example.com/oidc/revoke"
    },
    "callback_url": {
      "type": "string",
      "description": "Absolute `/api/auth/oidc/callback` derived from the effective issuer origin.",
      "example": "https://books.example.com/api/auth/oidc/callback"
    },
    "providers": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One IdP as shown to an operator (no client secret or Apple PEM).",
        "required": [
          "id",
          "name",
          "client_id",
          "scopes",
          "provision",
          "default_role",
          "role_claim",
          "role_map",
          "link_by_email",
          "allowed_email_domains",
          "allowed_emails",
          "allowed_subjects",
          "has_client_secret",
          "has_apple_private_key",
          "secret_source"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Stable provider id used in authorize URLs and sealed-row names.",
            "example": "google"
          },
          "name": {
            "type": "string",
            "description": "Operator-facing display name.",
            "example": "Google"
          },
          "preset": {
            "type": [
              "string",
              "null"
            ],
            "description": "Built-in social preset (`google`, `github`, `apple`, `discord`) when not a generic issuer.",
            "example": "google"
          },
          "issuer": {
            "type": [
              "string",
              "null"
            ],
            "description": "OpenID issuer URL for discovery when no social preset applies.",
            "example": "https://accounts.google.com"
          },
          "client_id": {
            "type": "string",
            "description": "OAuth client id registered at the IdP.",
            "example": "123456789012-a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6.apps.googleusercontent.com"
          },
          "scopes": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "OAuth scopes requested at authorize.",
            "example": [
              "openid",
              "profile",
              "email"
            ]
          },
          "provision": {
            "type": "string",
            "description": "How a first login becomes a portal user (JIT, mapped role, allowlist, invite-only).",
            "example": "allowlist"
          },
          "default_role": {
            "type": "string",
            "description": "Portal role used when no mapped claim matches (`member` by default).",
            "example": "member"
          },
          "role_claim": {
            "type": "string",
            "description": "Claim name read from id_token/userinfo for role mapping (`groups` by default).",
            "example": "groups"
          },
          "role_map": {
            "type": "object",
            "description": "IdP group/role claim values mapped onto Bookclerk portal roles.",
            "additionalProperties": {
              "type": "string"
            },
            "propertyNames": {
              "type": "string"
            },
            "example": {}
          },
          "link_by_email": {
            "type": "boolean",
            "description": "When true, a verified email may attach this IdP to an existing user.",
            "example": true
          },
          "allowed_email_domains": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Per-provider email domain allowlist (in addition to the broker-wide list).",
            "example": [
              "example.com"
            ]
          },
          "allowed_emails": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Exact emails permitted under allowlist provision.",
            "example": [
              "dana@example.com"
            ]
          },
          "allowed_subjects": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Exact IdP subject ids permitted under allowlist provision.",
            "example": [
              "248289761001"
            ]
          },
          "apple_team_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Apple Developer team id used to mint the client-secret JWT.",
            "example": "A1B2C3D4E5"
          },
          "apple_key_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Apple Sign In key id paired with the sealed private key.",
            "example": "K9L8M7N6P5"
          },
          "has_client_secret": {
            "type": "boolean",
            "description": "Whether a client secret exists in env, TOML, or the sealed store (value never returned).",
            "example": true
          },
          "has_apple_private_key": {
            "type": "boolean",
            "description": "Whether an Apple PEM exists in env, TOML, or the sealed store (PEM never returned).",
            "example": false
          },
          "secret_source": {
            "type": "string",
            "description": "Where the client secret was found: `env`, `config`, `store`, or `none`.",
            "example": "store"
          }
        }
      },
      "description": "Configured IdPs with presence flags instead of secret material."
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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
- `400` — A provider or the public origin is invalid (`oidc_config`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `401` — The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`oidc_config`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — The caller is neither the operator nor an Owner (the standard error body before the handler, `oidc_config` from it), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many wrong `current_password` attempts for this account; try again after the lockout (`oidc_config`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The configuration file or the sealed secrets could not be written, or an Owner's current password could not be read or checked (`oidc_config`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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
    "example.com"
  ],
  "public_origin": "https://books.example.com",
  "issuer_url": "https://books.example.com",
  "detected_origin": "http://localhost:8787",
  "discovery_url": "https://books.example.com/.well-known/openid-configuration",
  "authorization_endpoint": "https://books.example.com/oidc/authorize",
  "token_endpoint": "https://books.example.com/oidc/token",
  "userinfo_endpoint": "https://books.example.com/oidc/userinfo",
  "revocation_endpoint": "https://books.example.com/oidc/revoke",
  "callback_url": "https://books.example.com/api/auth/oidc/callback",
  "providers": [
    {
      "id": "google",
      "name": "Google",
      "preset": "google",
      "issuer": "https://accounts.google.com",
      "client_id": "123456789012-a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6.apps.googleusercontent.com",
      "scopes": [
        "openid",
        "profile",
        "email"
      ],
      "provision": "allowlist",
      "default_role": "member",
      "role_claim": "groups",
      "role_map": {},
      "link_by_email": true,
      "allowed_email_domains": [
        "example.com"
      ],
      "allowed_emails": [
        "dana@example.com"
      ],
      "allowed_subjects": [
        "248289761001"
      ],
      "apple_team_id": "A1B2C3D4E5",
      "apple_key_id": "K9L8M7N6P5",
      "has_client_secret": true,
      "has_apple_private_key": false,
      "secret_source": "store"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "Operator-facing OIDC broker snapshot; secret material is never included.",
  "required": [
    "enabled",
    "allowed_email_domains",
    "issuer_url",
    "detected_origin",
    "discovery_url",
    "authorization_endpoint",
    "token_endpoint",
    "userinfo_endpoint",
    "revocation_endpoint",
    "callback_url",
    "providers"
  ],
  "properties": {
    "enabled": {
      "type": "boolean",
      "description": "Whether the broker accepts new authorize starts.",
      "example": true
    },
    "allowed_email_domains": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Broker-wide email domain allowlist applied before per-provider rules.",
      "example": [
        "example.com"
      ]
    },
    "public_origin": {
      "type": [
        "string",
        "null"
      ],
      "description": "Configured `integrations.public_origin` (issuer / callback / invite base).",
      "example": "https://books.example.com"
    },
    "issuer_url": {
      "type": "string",
      "description": "Effective issuer origin (`public_origin`, else this request, else localhost).",
      "example": "https://books.example.com"
    },
    "detected_origin": {
      "type": "string",
      "description": "Origin detected from this request (loopback IPs rewritten to `localhost`).",
      "example": "http://localhost:8787"
    },
    "discovery_url": {
      "type": "string",
      "description": "Absolute OpenID discovery URL (`/.well-known/openid-configuration`).",
      "example": "https://books.example.com/.well-known/openid-configuration"
    },
    "authorization_endpoint": {
      "type": "string",
      "description": "Absolute authorization endpoint (`/oidc/authorize`) for Bookclerk-as-IdP.",
      "example": "https://books.example.com/oidc/authorize"
    },
    "token_endpoint": {
      "type": "string",
      "description": "Absolute token endpoint (`/oidc/token`) for Bookclerk-as-IdP.",
      "example": "https://books.example.com/oidc/token"
    },
    "userinfo_endpoint": {
      "type": "string",
      "description": "Absolute userinfo endpoint (`/oidc/userinfo`) for Bookclerk-as-IdP.",
      "example": "https://books.example.com/oidc/userinfo"
    },
    "revocation_endpoint": {
      "type": "string",
      "description": "Absolute revocation endpoint (`/oidc/revoke`) for Bookclerk-as-IdP.",
      "example": "https://books.example.com/oidc/revoke"
    },
    "callback_url": {
      "type": "string",
      "description": "Absolute `/api/auth/oidc/callback` derived from the effective issuer origin.",
      "example": "https://books.example.com/api/auth/oidc/callback"
    },
    "providers": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One IdP as shown to an operator (no client secret or Apple PEM).",
        "required": [
          "id",
          "name",
          "client_id",
          "scopes",
          "provision",
          "default_role",
          "role_claim",
          "role_map",
          "link_by_email",
          "allowed_email_domains",
          "allowed_emails",
          "allowed_subjects",
          "has_client_secret",
          "has_apple_private_key",
          "secret_source"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Stable provider id used in authorize URLs and sealed-row names.",
            "example": "google"
          },
          "name": {
            "type": "string",
            "description": "Operator-facing display name.",
            "example": "Google"
          },
          "preset": {
            "type": [
              "string",
              "null"
            ],
            "description": "Built-in social preset (`google`, `github`, `apple`, `discord`) when not a generic issuer.",
            "example": "google"
          },
          "issuer": {
            "type": [
              "string",
              "null"
            ],
            "description": "OpenID issuer URL for discovery when no social preset applies.",
            "example": "https://accounts.google.com"
          },
          "client_id": {
            "type": "string",
            "description": "OAuth client id registered at the IdP.",
            "example": "123456789012-a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6.apps.googleusercontent.com"
          },
          "scopes": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "OAuth scopes requested at authorize.",
            "example": [
              "openid",
              "profile",
              "email"
            ]
          },
          "provision": {
            "type": "string",
            "description": "How a first login becomes a portal user (JIT, mapped role, allowlist, invite-only).",
            "example": "allowlist"
          },
          "default_role": {
            "type": "string",
            "description": "Portal role used when no mapped claim matches (`member` by default).",
            "example": "member"
          },
          "role_claim": {
            "type": "string",
            "description": "Claim name read from id_token/userinfo for role mapping (`groups` by default).",
            "example": "groups"
          },
          "role_map": {
            "type": "object",
            "description": "IdP group/role claim values mapped onto Bookclerk portal roles.",
            "additionalProperties": {
              "type": "string"
            },
            "propertyNames": {
              "type": "string"
            },
            "example": {}
          },
          "link_by_email": {
            "type": "boolean",
            "description": "When true, a verified email may attach this IdP to an existing user.",
            "example": true
          },
          "allowed_email_domains": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Per-provider email domain allowlist (in addition to the broker-wide list).",
            "example": [
              "example.com"
            ]
          },
          "allowed_emails": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Exact emails permitted under allowlist provision.",
            "example": [
              "dana@example.com"
            ]
          },
          "allowed_subjects": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Exact IdP subject ids permitted under allowlist provision.",
            "example": [
              "248289761001"
            ]
          },
          "apple_team_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Apple Developer team id used to mint the client-secret JWT.",
            "example": "A1B2C3D4E5"
          },
          "apple_key_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Apple Sign In key id paired with the sealed private key.",
            "example": "K9L8M7N6P5"
          },
          "has_client_secret": {
            "type": "boolean",
            "description": "Whether a client secret exists in env, TOML, or the sealed store (value never returned).",
            "example": true
          },
          "has_apple_private_key": {
            "type": "boolean",
            "description": "Whether an Apple PEM exists in env, TOML, or the sealed store (PEM never returned).",
            "example": false
          },
          "secret_source": {
            "type": "string",
            "description": "Where the client secret was found: `env`, `config`, `store`, or `none`.",
            "example": "store"
          }
        }
      },
      "description": "Configured IdPs with presence flags instead of secret material."
    }
  }
}
```

Status `400`: A provider or the public origin is invalid (`oidc_config`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "oidc_config",
  "message": "public origin must use http or https",
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

Status `401`: The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`oidc_config`).

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

Status `403`: The caller is neither the operator nor an Owner (the standard error body before the handler, `oidc_config` from it), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "oidc_config",
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `enabled` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many wrong `current_password` attempts for this account; try again after the lockout (`oidc_config`).

```json
{
  "error": "oidc_config",
  "message": "too many wrong passwords for this account; try again later",
  "status": 429
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

Status `500`: The configuration file or the sealed secrets could not be written, or an Owner's current password could not be read or checked (`oidc_config`).

```json
{
  "error": "oidc_config",
  "message": "failed to write config.toml: Permission denied (os error 13)",
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

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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

## `GET /api/auth/passkeys`

Lists this user's stored passkeys as `{ id, credential_id }` rows.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — The caller's passkeys. Body: [`PasskeysResponse`](schemas.md#passkeysresponse) (`application/json`)
  - `passkeys` *array of [`PasskeySummary`](schemas.md#passkeysummary)*, **required** — The caller's passkeys, oldest first.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The passkeys could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: The caller's passkeys.

```json
{
  "passkeys": [
    {
      "id": 4,
      "credential_id": "pQECAyYgASFYIE9ZJ2kLm3nO4pQ5rS6tU7vW8xY9zA0",
      "name": "YubiKey 5C"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/auth/passkeys`.",
  "required": [
    "passkeys"
  ],
  "properties": {
    "passkeys": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One stored passkey.",
        "required": [
          "id",
          "credential_id",
          "name"
        ],
        "properties": {
          "id": {
            "type": "integer",
            "format": "int64",
            "description": "Row id, for `DELETE /api/auth/passkeys/{id}`.",
            "example": 4
          },
          "credential_id": {
            "type": "string",
            "description": "Base64url credential id the authenticator reports.",
            "example": "pQECAyYgASFYIE9ZJ2kLm3nO4pQ5rS6tU7vW8xY9zA0"
          },
          "name": {
            "type": "string",
            "description": "Label chosen at registration (`Passkey` when none was given).",
            "example": "YubiKey 5C"
          }
        }
      },
      "description": "The caller's passkeys, oldest first."
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The passkeys could not be read.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/passkeys/register/begin`

Starts passkey registration (skips reauth only for first local-only setup).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`PasskeyReauthBody`](schemas.md#passkeyreauthbody)

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

### Responses

- `200` — WebAuthn creation options. Body: [`PasskeyCreationOptions`](schemas.md#passkeycreationoptions) (`application/json`)
  - `challenge_id` *string*, **required** — Server-issued challenge id to send back with the attestation; consumed once, five-minute lifetime.
  - `publicKey` *any JSON*, **required** — W3C PublicKeyCredentialCreationOptions (relying party, user, challenge, algorithms, excluded credentials), binary fields base64url-encoded.
- `400` — The daemon's origin is not a valid WebAuthn relying party (set `integrations.public_origin`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the session is older than fifteen minutes and `current_password` is missing or wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many wrong `current_password` attempts for this account; try again after the lockout. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The library could not be read, or the challenge could not be stored. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: WebAuthn creation options.

```json
{
  "challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f",
  "publicKey": {
    "attestation": "none",
    "authenticatorSelection": {
      "residentKey": "required",
      "userVerification": "preferred"
    },
    "challenge": "k9u2Xo3cZ1Qp7mT4vW8yA2bC5dE6fG7hI8jK9lM0nO1",
    "excludeCredentials": [],
    "pubKeyCredParams": [
      {
        "alg": -7,
        "type": "public-key"
      },
      {
        "alg": -257,
        "type": "public-key"
      }
    ],
    "rp": {
      "id": "books.example.com",
      "name": "Bookclerk"
    },
    "timeout": 60000,
    "user": {
      "displayName": "Sam",
      "id": "AAAAAAAAAAAAAAAAAAAABw",
      "name": "sam"
    }
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "WebAuthn creation options for `navigator.credentials.create()`, plus the challenge id for the finish step.",
  "required": [
    "challenge_id",
    "publicKey"
  ],
  "properties": {
    "challenge_id": {
      "type": "string",
      "description": "Server-issued challenge id to send back with the attestation; consumed once, five-minute lifetime.",
      "example": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f"
    },
    "publicKey": {
      "description": "W3C PublicKeyCredentialCreationOptions (relying party, user, challenge, algorithms, excluded credentials), binary fields base64url-encoded.",
      "examples": [
        {
          "attestation": "none",
          "authenticatorSelection": {
            "residentKey": "required",
            "userVerification": "preferred"
          },
          "challenge": "k9u2Xo3cZ1Qp7mT4vW8yA2bC5dE6fG7hI8jK9lM0nO1",
          "excludeCredentials": [],
          "pubKeyCredParams": [
            {
              "alg": -7,
              "type": "public-key"
            },
            {
              "alg": -257,
              "type": "public-key"
            }
          ],
          "rp": {
            "id": "books.example.com",
            "name": "Bookclerk"
          },
          "timeout": 60000,
          "user": {
            "displayName": "Sam",
            "id": "AAAAAAAAAAAAAAAAAAAABw",
            "name": "sam"
          }
        }
      ]
    }
  }
}
```

Status `400`: The daemon's origin is not a valid WebAuthn relying party (set `integrations.public_origin`). As `text/plain`, when the body is not syntactically valid JSON.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `401`: The caller is not signed in, or the session is older than fifteen minutes and `current_password` is missing or wrong.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: current_password: invalid type: integer `1`, expected a string at line 1 column 21
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many wrong `current_password` attempts for this account; try again after the lockout.

```json
{
  "error": "too_many_requests",
  "message": "Too many requests. Try again shortly.",
  "status": 429
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The library could not be read, or the challenge could not be stored.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/passkeys/register/finish`

Completes registration, stores the passkey, and revokes elevated operator sessions.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`CeremonyFinish`](schemas.md#ceremonyfinish)

- `challenge_id` *string*, **required** — Server-issued challenge id (consumed once; 5-minute TTL).
- `credential` *any JSON*, **required** — W3C PublicKeyCredential as the browser serializes it: a registration attestation (`response.attestationObject`) or an authentication assertion (`response.authenticatorData`, `signature`, `userHandle`), binary fields base64url-encoded.
- `name` *string or null* — Label stored with the credential (`Passkey` when empty).

### Responses

- `200` — The passkey was stored. Body: [`OkResponse`](schemas.md#okresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`: a failure answers with an error status instead.
- `400` — The challenge is unknown or expired, the attestation is invalid, or the daemon's origin is not a valid WebAuthn relying party. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The challenge was issued to another user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The passkey is already registered. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/register/finish' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/register/finish' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/passkeys/register/finish",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/register/finish", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/passkeys/register/finish")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The passkey was stored.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Acknowledgement body (`{\"ok\": true}`) for actions with nothing else to report.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`: a failure answers with an error status instead.",
      "example": true
    }
  }
}
```

Status `400`: The challenge is unknown or expired, the attestation is invalid, or the daemon's origin is not a valid WebAuthn relying party. As `text/plain`, when the body is not syntactically valid JSON.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The challenge was issued to another user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `409`: The passkey is already registered.

```json
{
  "error": "conflict",
  "message": "Conflict",
  "status": 409
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `challenge_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The library could not be read or written.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

- `200` — The passkey was deleted. Body: [`OkResponse`](schemas.md#okresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`: a failure answers with an error status instead.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the session is older than fifteen minutes and `current_password` is missing or wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such passkey for this user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many wrong `current_password` attempts for this account; try again after the lockout. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The library could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: The passkey was deleted.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Acknowledgement body (`{\"ok\": true}`) for actions with nothing else to report.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`: a failure answers with an error status instead.",
      "example": true
    }
  }
}
```

Status `400`: The body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

Status `401`: The caller is not signed in, or the session is older than fifteen minutes and `current_password` is missing or wrong.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: No such passkey for this user.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: current_password: invalid type: integer `1`, expected a string at line 1 column 21
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many wrong `current_password` attempts for this account; try again after the lockout.

```json
{
  "error": "too_many_requests",
  "message": "Too many requests. Try again shortly.",
  "status": 429
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The library could not be read or written.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/passkeys/login/begin`

Rate-limits and starts passkey login; records a failure on any error.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`LoginBegin`](schemas.md#loginbegin)

- `login` *string*, **required** — Login name or email looked up before issuing an authentication challenge.

### Responses

- `200` — WebAuthn request options. Body: [`PasskeyRequestOptions`](schemas.md#passkeyrequestoptions) (`application/json`)
  - `challenge_id` *string*, **required** — Server-issued challenge id to send back with the assertion; consumed once, five-minute lifetime.
  - `publicKey` *any JSON*, **required** — W3C PublicKeyCredentialRequestOptions (challenge, relying party id, allowed credentials), binary fields base64url-encoded.
  - `mediation` *string or null* — Credential mediation the daemon asks the browser for; absent unless conditional mediation was requested.
- `400` — The daemon's origin is not a valid WebAuthn relying party (set `integrations.public_origin`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — No user has this login name or email. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is disabled. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The user has no usable passkeys. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many failed attempts from this client; try later. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The library could not be read, or the challenge could not be stored. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/login/begin' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"login": "dana"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/login/begin' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"login": "dana"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/passkeys/login/begin",
    json={"login": "dana"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/login/begin", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"login": "dana"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/passkeys/login/begin")
    .json(&serde_json::json!({"login": "dana"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: WebAuthn request options.

```json
{
  "challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f",
  "publicKey": {
    "allowCredentials": [
      {
        "id": "pQECAyYgASFYIE",
        "type": "public-key"
      }
    ],
    "challenge": "k9u2Xo3cZ1Qp7mT4vW8yA2bC5dE6fG7hI8jK9lM0nO1",
    "rpId": "books.example.com",
    "timeout": 60000,
    "userVerification": "preferred"
  },
  "mediation": "conditional"
}
```

Schema:

```json
{
  "type": "object",
  "description": "WebAuthn request options for `navigator.credentials.get()`, plus the challenge id for the finish step.",
  "required": [
    "challenge_id",
    "publicKey"
  ],
  "properties": {
    "challenge_id": {
      "type": "string",
      "description": "Server-issued challenge id to send back with the assertion; consumed once, five-minute lifetime.",
      "example": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f"
    },
    "publicKey": {
      "description": "W3C PublicKeyCredentialRequestOptions (challenge, relying party id, allowed credentials), binary fields base64url-encoded.",
      "examples": [
        {
          "allowCredentials": [
            {
              "id": "pQECAyYgASFYIE",
              "type": "public-key"
            }
          ],
          "challenge": "k9u2Xo3cZ1Qp7mT4vW8yA2bC5dE6fG7hI8jK9lM0nO1",
          "rpId": "books.example.com",
          "timeout": 60000,
          "userVerification": "preferred"
        }
      ]
    },
    "mediation": {
      "type": [
        "string",
        "null"
      ],
      "description": "Credential mediation the daemon asks the browser for; absent unless\nconditional mediation was requested.",
      "example": "conditional"
    }
  }
}
```

Status `400`: The daemon's origin is not a valid WebAuthn relying party (set `integrations.public_origin`). As `text/plain`, when the body is not syntactically valid JSON.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `401`: No user has this login name or email.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The user is disabled.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: The user has no usable passkeys.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `login` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many failed attempts from this client; try later.

```json
{
  "error": "too_many_requests",
  "message": "Too many requests. Try again shortly.",
  "status": 429
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The library could not be read, or the challenge could not be stored.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/passkeys/login/finish`

Verifies the assertion, updates the credential counter, and issues a portal session.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`CeremonyFinish`](schemas.md#ceremonyfinish)

- `challenge_id` *string*, **required** — Server-issued challenge id (consumed once; 5-minute TTL).
- `credential` *any JSON*, **required** — W3C PublicKeyCredential as the browser serializes it: a registration attestation (`response.attestationObject`) or an authentication assertion (`response.authenticatorData`, `signature`, `userHandle`), binary fields base64url-encoded.
- `name` *string or null* — Label stored with the credential (`Passkey` when empty).

### Responses

- `200` — Signed in; the response sets the portal session cookie. Body: [`PortalSessionIssued`](schemas.md#portalsessionissued) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `role` *string*, **required** — The user's role (`owner`, `administrator`, `host`, or `member`).
- `400` — The challenge is unknown or expired, the credential is malformed, or the daemon's origin is not a valid WebAuthn relying party. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The assertion does not verify, or the credential is not one of the challenged user's. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is disabled. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/login/finish' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/login/finish' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/passkeys/login/finish",
    json={"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/login/finish", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/passkeys/login/finish")
    .json(&serde_json::json!({"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in; the response sets the portal session cookie.

```json
{
  "ok": true,
  "role": "member"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after a portal session was minted for a first-party user.",
  "required": [
    "ok",
    "role"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "role": {
      "type": "string",
      "description": "The user's role (`owner`, `administrator`, `host`, or `member`).",
      "example": "member"
    }
  }
}
```

Status `400`: The challenge is unknown or expired, the credential is malformed, or the daemon's origin is not a valid WebAuthn relying party. As `text/plain`, when the body is not syntactically valid JSON.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `401`: The assertion does not verify, or the credential is not one of the challenged user's.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The user is disabled.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `challenge_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The library could not be read or written.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/passkeys/elevate/begin`

Starts Owner elevation; 403 unless the signed-in user is an Owner with passkeys.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — WebAuthn request options. Body: [`PasskeyRequestOptions`](schemas.md#passkeyrequestoptions) (`application/json`)
  - `challenge_id` *string*, **required** — Server-issued challenge id to send back with the assertion; consumed once, five-minute lifetime.
  - `publicKey` *any JSON*, **required** — W3C PublicKeyCredentialRequestOptions (challenge, relying party id, allowed credentials), binary fields base64url-encoded.
  - `mediation` *string or null* — Credential mediation the daemon asks the browser for; absent unless conditional mediation was requested.
- `400` — The daemon's origin is not a valid WebAuthn relying party (set `integrations.public_origin`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is not an Owner. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The caller has no passkeys. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The library could not be read, or the challenge could not be stored. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: WebAuthn request options.

```json
{
  "challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f",
  "publicKey": {
    "allowCredentials": [
      {
        "id": "pQECAyYgASFYIE",
        "type": "public-key"
      }
    ],
    "challenge": "k9u2Xo3cZ1Qp7mT4vW8yA2bC5dE6fG7hI8jK9lM0nO1",
    "rpId": "books.example.com",
    "timeout": 60000,
    "userVerification": "preferred"
  },
  "mediation": "conditional"
}
```

Schema:

```json
{
  "type": "object",
  "description": "WebAuthn request options for `navigator.credentials.get()`, plus the challenge id for the finish step.",
  "required": [
    "challenge_id",
    "publicKey"
  ],
  "properties": {
    "challenge_id": {
      "type": "string",
      "description": "Server-issued challenge id to send back with the assertion; consumed once, five-minute lifetime.",
      "example": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f"
    },
    "publicKey": {
      "description": "W3C PublicKeyCredentialRequestOptions (challenge, relying party id, allowed credentials), binary fields base64url-encoded.",
      "examples": [
        {
          "allowCredentials": [
            {
              "id": "pQECAyYgASFYIE",
              "type": "public-key"
            }
          ],
          "challenge": "k9u2Xo3cZ1Qp7mT4vW8yA2bC5dE6fG7hI8jK9lM0nO1",
          "rpId": "books.example.com",
          "timeout": 60000,
          "userVerification": "preferred"
        }
      ]
    },
    "mediation": {
      "type": [
        "string",
        "null"
      ],
      "description": "Credential mediation the daemon asks the browser for; absent unless\nconditional mediation was requested.",
      "example": "conditional"
    }
  }
}
```

Status `400`: The daemon's origin is not a valid WebAuthn relying party (set `integrations.public_origin`).

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller is not an Owner. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: The caller has no passkeys.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The library could not be read, or the challenge could not be stored.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/passkeys/elevate/finish`

Completes Owner elevation only when the assertion is user-verified.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`CeremonyFinish`](schemas.md#ceremonyfinish)

- `challenge_id` *string*, **required** — Server-issued challenge id (consumed once; 5-minute TTL).
- `credential` *any JSON*, **required** — W3C PublicKeyCredential as the browser serializes it: a registration attestation (`response.attestationObject`) or an authentication assertion (`response.authenticatorData`, `signature`, `userHandle`), binary fields base64url-encoded.
- `name` *string or null* — Label stored with the credential (`Passkey` when empty).

### Responses

- `200` — Elevated; the response sets a short-lived operator cookie. Body: [`Elevated`](schemas.md#elevated) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `elevated` *boolean*, **required** — Always `true`: the cookie the response sets is an elevated operator session.
  - `expires_in_secs` *integer (int64)*, **required** — Seconds until the elevated session expires. At least 0.
- `400` — The challenge is unknown or expired, the credential is malformed, or the daemon's origin is not a valid WebAuthn relying party. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, the assertion does not verify or was not user-verified, or the credential is not the caller's. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is not an Owner, or the challenge was issued to another user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/passkeys/elevate/finish' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/passkeys/elevate/finish' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/passkeys/elevate/finish",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/passkeys/elevate/finish", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/passkeys/elevate/finish")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"challenge_id": "6d2f0c1e-8a4b-4f3d-9e7c-5b1a2c3d4e5f", "credential": {"extensions": {}, "id": "pQECAyYgASFYIE", "rawId": "pQECAyYgASFYIE", "response": {"attestationObject": "o2NmbXRkbm9uZWdhdHRTdG10oGhhdXRoRGF0YViU", "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiazl1MlhvM2NaMVFwN21UNHZXOHlBMmJDNWRFNmZHN2hJOGpLOWxNMG5PMSIsIm9yaWdpbiI6Imh0dHBzOi8vYm9va3MuZXhhbXBsZS5jb20ifQ"}, "type": "public-key"}}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Elevated; the response sets a short-lived operator cookie.

```json
{
  "ok": true,
  "elevated": true,
  "expires_in_secs": 900
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after a successful Owner elevation.",
  "required": [
    "ok",
    "elevated",
    "expires_in_secs"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "elevated": {
      "type": "boolean",
      "description": "Always `true`: the cookie the response sets is an elevated operator session.",
      "example": true
    },
    "expires_in_secs": {
      "type": "integer",
      "format": "int64",
      "description": "Seconds until the elevated session expires.",
      "example": 900,
      "minimum": 0
    }
  }
}
```

Status `400`: The challenge is unknown or expired, the credential is malformed, or the daemon's origin is not a valid WebAuthn relying party. As `text/plain`, when the body is not syntactically valid JSON.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `401`: The caller is not signed in, the assertion does not verify or was not user-verified, or the credential is not the caller's.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller is not an Owner, or the challenge was issued to another user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `challenge_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The library could not be read or written.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/totp/login`

Completes password+TOTP login and issues a portal session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`LoginVerify`](schemas.md#loginverify)

- `challenge_id` *string*, **required** — Server-issued challenge id (5-minute TTL, single use).
- `code` *string*, **required** — Six-digit authenticator code.

### Responses

- `200` — Signed in; the response sets the portal session cookie. Body: [`PortalSessionIssued`](schemas.md#portalsessionissued) (`application/json`)
  - `ok` *boolean*, **required** — Always `true` on this success path.
  - `role` *string*, **required** — The user's role (`owner`, `administrator`, `host`, or `member`).
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The challenge is unknown or expired, TOTP is off for the user, or the code is wrong (`invalid_code`); or the challenged user no longer exists (the standard error body). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many failed attempts from this client; try later. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The user or secret could not be read, or the session could not be stored. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/totp/login' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"challenge_id": "2c7e4f0a-9b1d-4e3c-a5f6-7d8e9f0a1b2c", "code": "492817"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/totp/login' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"challenge_id": "2c7e4f0a-9b1d-4e3c-a5f6-7d8e9f0a1b2c", "code": "492817"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/totp/login",
    json={"challenge_id": "2c7e4f0a-9b1d-4e3c-a5f6-7d8e9f0a1b2c", "code": "492817"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/totp/login", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"challenge_id": "2c7e4f0a-9b1d-4e3c-a5f6-7d8e9f0a1b2c", "code": "492817"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/totp/login")
    .json(&serde_json::json!({"challenge_id": "2c7e4f0a-9b1d-4e3c-a5f6-7d8e9f0a1b2c", "code": "492817"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in; the response sets the portal session cookie.

```json
{
  "ok": true,
  "role": "member"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after a portal session was minted for a first-party user.",
  "required": [
    "ok",
    "role"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true` on this success path.",
      "example": true
    },
    "role": {
      "type": "string",
      "description": "The user's role (`owner`, `administrator`, `host`, or `member`).",
      "example": "member"
    }
  }
}
```

Status `400`: The body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

Status `401`: The challenge is unknown or expired, TOTP is off for the user, or the code is wrong (`invalid_code`); or the challenged user no longer exists (the standard error body).

```json
{
  "error": "invalid_code",
  "message": "Invalid authenticator code.",
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `challenge_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many failed attempts from this client; try later.

```json
{
  "error": "too_many_requests",
  "message": "Too many requests. Try again shortly.",
  "status": 429
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The user or secret could not be read, or the session could not be stored.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/totp/enroll/begin`

Starts TOTP enrollment: new pending secret, otpauth URL, and QR SVG.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`TotpReauthBody`](schemas.md#totpreauthbody)

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

### Responses

- `200` — A pending secret, its otpauth URL and a QR code SVG. Body: [`TotpEnrollment`](schemas.md#totpenrollment) (`application/json`)
  - `secret` *string*, **required** — Pending base32 secret, for typing into an app by hand.
  - `otpauth_url` *string*, **required** — `otpauth://` URL the QR code encodes.
  - `qr_svg` *string*, **required** — The QR code as an SVG document, ready to inline.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the session is older than fifteen minutes and `current_password` is missing or wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — TOTP is already on. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many wrong `current_password` attempts for this account; try again after the lockout. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The secret could not be sealed and stored, or the library could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: A pending secret, its otpauth URL and a QR code SVG.

```json
{
  "secret": "JBSWY3DPEHPK3PXPJBSWY3DPEHPK3PXP",
  "otpauth_url": "otpauth://totp/Bookclerk:sam?secret=JBSWY3DPEHPK3PXPJBSWY3DPEHPK3PXP&issuer=Bookclerk&algorithm=SHA1&digits=6&period=30",
  "qr_svg": "<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 37 37\">…</svg>"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after TOTP enrollment began: what the authenticator app needs.",
  "required": [
    "secret",
    "otpauth_url",
    "qr_svg"
  ],
  "properties": {
    "secret": {
      "type": "string",
      "description": "Pending base32 secret, for typing into an app by hand.",
      "example": "JBSWY3DPEHPK3PXPJBSWY3DPEHPK3PXP"
    },
    "otpauth_url": {
      "type": "string",
      "description": "`otpauth://` URL the QR code encodes.",
      "example": "otpauth://totp/Bookclerk:sam?secret=JBSWY3DPEHPK3PXPJBSWY3DPEHPK3PXP&issuer=Bookclerk&algorithm=SHA1&digits=6&period=30"
    },
    "qr_svg": {
      "type": "string",
      "description": "The QR code as an SVG document, ready to inline.",
      "example": "<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 37 37\">…</svg>"
    }
  }
}
```

Status `400`: The body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

Status `401`: The caller is not signed in, or the session is older than fifteen minutes and `current_password` is missing or wrong.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `409`: TOTP is already on.

```json
{
  "error": "conflict",
  "message": "Conflict",
  "status": 409
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: current_password: invalid type: integer `1`, expected a string at line 1 column 21
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many wrong `current_password` attempts for this account; try again after the lockout.

```json
{
  "error": "too_many_requests",
  "message": "Too many requests. Try again shortly.",
  "status": 429
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The secret could not be sealed and stored, or the library could not be read.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `POST /api/auth/totp/enroll/finish`

Confirms the pending secret with a valid code and sets `users.totp_enabled`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`EnrollFinish`](schemas.md#enrollfinish)

- `code` *string*, **required** — Six-digit code from the authenticator app.

### Responses

- `200` — TOTP is on. Body: [`OkResponse`](schemas.md#okresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`: a failure answers with an error status instead.
- `400` — No enrollment is pending; start one first. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in (the standard error body), or the code is wrong (`invalid_code`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — TOTP is already on. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The pending secret could not be read, or the enrollment could not be confirmed. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/auth/totp/enroll/finish' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"code": "492817"}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/auth/totp/enroll/finish' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"code": "492817"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/auth/totp/enroll/finish",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"code": "492817"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/auth/totp/enroll/finish", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"code": "492817"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/auth/totp/enroll/finish")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"code": "492817"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: TOTP is on.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Acknowledgement body (`{\"ok\": true}`) for actions with nothing else to report.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`: a failure answers with an error status instead.",
      "example": true
    }
  }
}
```

Status `400`: No enrollment is pending; start one first. As `text/plain`, when the body is not syntactically valid JSON.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `401`: The caller is not signed in (the standard error body), or the code is wrong (`invalid_code`).

```json
{
  "error": "invalid_code",
  "message": "Invalid authenticator code.",
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

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `409`: TOTP is already on.

```json
{
  "error": "conflict",
  "message": "Conflict",
  "status": 409
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `code` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The pending secret could not be read, or the enrollment could not be confirmed.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `GET /api/auth/totp`

GET `/api/auth/totp` — whether TOTP is confirmed for this user.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — Whether TOTP is on for the caller. Body: [`TotpStatus`](schemas.md#totpstatus) (`application/json`)
  - `enabled` *boolean*, **required** — Whether an authenticator app is confirmed for the caller.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The user could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: Whether TOTP is on for the caller.

```json
{
  "enabled": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/auth/totp`.",
  "required": [
    "enabled"
  ],
  "properties": {
    "enabled": {
      "type": "boolean",
      "description": "Whether an authenticator app is confirmed for the caller.",
      "example": true
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `DELETE /api/auth/totp`

Disables TOTP after recent reauth and deletes sealed secrets.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`TotpReauthBody`](schemas.md#totpreauthbody)

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

### Responses

- `200` — TOTP is off. Body: [`OkResponse`](schemas.md#okresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`: a failure answers with an error status instead.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the session is older than fifteen minutes and `current_password` is missing or wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many wrong `current_password` attempts for this account; try again after the lockout. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The library could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: TOTP is off.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Acknowledgement body (`{\"ok\": true}`) for actions with nothing else to report.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`: a failure answers with an error status instead.",
      "example": true
    }
  }
}
```

Status `400`: The body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

Status `401`: The caller is not signed in, or the session is older than fifteen minutes and `current_password` is missing or wrong.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: current_password: invalid type: integer `1`, expected a string at line 1 column 21
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many wrong `current_password` attempts for this account; try again after the lockout.

```json
{
  "error": "too_many_requests",
  "message": "Too many requests. Try again shortly.",
  "status": 429
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The library could not be read or written.

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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

## `GET /api/auth/mfa-policy`

Current MFA policy (`require_second_factor`).

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Whether a second factor is required. Body: [`MfaPolicy`](schemas.md#mfapolicy) (`application/json`)
  - `require_second_factor` *boolean*, **required** — Whether every password sign-in must be followed by a passkey or authenticator code.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is neither the operator nor an Owner (`forbidden`), or must enroll a second factor first (`mfa_enrollment_required`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `200`: Whether a second factor is required.

```json
{
  "require_second_factor": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "The host-wide second-factor policy.",
  "required": [
    "require_second_factor"
  ],
  "properties": {
    "require_second_factor": {
      "type": "boolean",
      "description": "Whether every password sign-in must be followed by a passkey or authenticator code.",
      "example": false
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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

## `PUT /api/auth/mfa-policy`

Persists `daemon.auth.require_second_factor` to `config.toml`.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`MfaPolicyPut`](schemas.md#mfapolicyput)

- `require_second_factor` *boolean*, **required** — When true, password login requires TOTP or the user must use a passkey.
- `current_password` *string or null* — Current password for Owner step-up (operators omit this).

### Responses

- `200` — The policy after the change. Body: [`MfaPolicy`](schemas.md#mfapolicy) (`application/json`)
  - `require_second_factor` *boolean*, **required** — Whether every password sign-in must be followed by a passkey or authenticator code.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`reauth_required`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `403` — The caller is neither the operator nor an Owner (`forbidden`), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `429` — Too many wrong `current_password` attempts for this account; try again after the lockout (`too_many_requests`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The configuration path is unknown or the configuration file could not be written, or an Owner's current password could not be read or checked (`internal_error`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

Status `200`: The policy after the change.

```json
{
  "require_second_factor": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "The host-wide second-factor policy.",
  "required": [
    "require_second_factor"
  ],
  "properties": {
    "require_second_factor": {
      "type": "boolean",
      "description": "Whether every password sign-in must be followed by a passkey or authenticator code.",
      "example": false
    }
  }
}
```

Status `400`: The body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

Status `401`: The caller is not signed in (the standard error body), or an Owner's session is older than fifteen minutes and `current_password` is missing or wrong (`reauth_required`).

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

Status `403`: The caller is neither the operator nor an Owner (`forbidden`), or must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `require_second_factor` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `429`: Too many wrong `current_password` attempts for this account; try again after the lockout (`too_many_requests`).

```json
{
  "error": "too_many_requests",
  "message": "Too many wrong passwords for this account. Try again later.",
  "status": 429
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

Status `500`: The configuration path is unknown or the configuration file could not be written, or an Owner's current password could not be read or checked (`internal_error`).

```json
{
  "error": "internal_error",
  "message": "failed to write config.toml: Permission denied (os error 13)",
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

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
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
