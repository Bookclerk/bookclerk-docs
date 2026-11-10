# Discover

Recommendations, catalog search and title metadata.

Back to the [web API reference](README.md).

## `POST /api/discover/sync-listening`

Enqueues a listening-progress sync; 400 when no capable integrations are loaded.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The job was accepted. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `400` — No listening-capable integration is enabled. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `409` — The same job is already pending or running, or no node can run it. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `429` — The job queue is full. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `500` — The job could not be queued. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `503` — Job admission is paused. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/sync-listening' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/sync-listening' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/sync-listening",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/sync-listening", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/sync-listening")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The job was accepted.

```json
{
  "job_id": "listen_sync-2b1a0f9e-8d7c-4b6a-9f5e-4d3c2b1a0f9e",
  "message": "listen_sync job listen_sync-2b1a0f9e-8d7c-4b6a-9f5e-4d3c2b1a0f9e accepted",
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Generic mutation result with success flag, message, and optional job id.",
  "required": [
    "ok",
    "message",
    "job_id"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when the action was accepted or completed.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan queued"
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`).",
      "example": "queue_full"
    }
  }
}
```

Status `400`: No listening-capable integration is enabled.

```text
no listening-capable integrations are enabled
```

Schema:

A `text/plain` body: a string.

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

Status `403`: The caller's role does not allow this route. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `409`: The same job is already pending or running, or no node can run it.

```json
{
  "error": "conflict",
  "job_id": "listen_sync-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
  "message": "listen_sync already pending or running",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Generic mutation result with success flag, message, and optional job id.",
  "required": [
    "ok",
    "message",
    "job_id"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when the action was accepted or completed.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan queued"
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`).",
      "example": "queue_full"
    }
  }
}
```

Status `429`: The job queue is full.

```json
{
  "error": "queue_full",
  "job_id": "",
  "message": "job queue is full",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Generic mutation result with success flag, message, and optional job id.",
  "required": [
    "ok",
    "message",
    "job_id"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when the action was accepted or completed.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan queued"
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`).",
      "example": "queue_full"
    }
  }
}
```

Status `500`: The job could not be queued.

```json
{
  "error": "internal_error",
  "job_id": "",
  "message": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Generic mutation result with success flag, message, and optional job id.",
  "required": [
    "ok",
    "message",
    "job_id"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when the action was accepted or completed.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan queued"
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`).",
      "example": "queue_full"
    }
  }
}
```

Status `503`: Job admission is paused. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "admission_paused",
  "job_id": "",
  "message": "job admission paused (database reload in progress)",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Generic mutation result with success flag, message, and optional job id.",
  "required": [
    "ok",
    "message",
    "job_id"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when the action was accepted or completed.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan queued"
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`).",
      "example": "queue_full"
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

## `GET /api/covers/{token}/{size}`

Serves one size of a cover the API linked to.

The path comes from a `cover_path` field in a Discover or catalog
response; append `/128`, `/360` or `/full`. The first request fetches the
image through the `covers` plugin and re-encodes it in the confined media
worker; later ones are served from the node's cover cache. Responses are
immutable, since a path always names the same image, and carry an `ETag`
so a client revalidating anyway gets `304 Not Modified`.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `token` (path) *string*, **required** — Signed cover token from a `cover_path` field.
- `size` (path) *string*, **required** — `128` (thumbnail), `360` (card) or `full`.

### Responses

- `200` — The cover as a JPEG. Body: binary (`image/jpeg`)
- `304` — `If-None-Match` names the current `ETag`; no body. No body.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — The token was not issued by this node, or the size is unknown. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `502` — The image could not be downloaded or is not a supported image. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — This node cannot fetch or process covers right now. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/covers/<token>/<size>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/covers/<token>/<size>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/covers/<token>/<size>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/covers/<token>/<size>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/covers/<token>/<size>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The cover as a JPEG. The body is binary (`image/jpeg`).

Schema:

A `image/jpeg` body: binary bytes.

Status `304`: `If-None-Match` names the current `ETag`; no body. No body.

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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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

Status `404`: The token was not issued by this node, or the size is unknown.

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

Status `502`: The image could not be downloaded or is not a supported image.

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

Status `503`: This node cannot fetch or process covers right now. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "service_unavailable",
  "message": "The Bookclerk daemon is not ready to handle this request.",
  "status": 503
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

## `GET /api/images/{token}`

Serves a signed image that is not a cover as a 128 px JPEG.

The path comes from a `picture_path` or `logo` field: a sign-in
provider's profile picture or a plugin's remote logo. It is fetched and
processed like a cover, but answers with an hour-long browser cache
header instead of `immutable`, because the remote image can change at the
same address. Once upstream's freshness lifetime (one hour to seven
days) runs out, Bookclerk serves its cached copy while it checks with the
image host in the background.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `token` (path) *string*, **required** — Signed token from a `picture_path` or `logo` field.

### Responses

- `200` — The image as a JPEG. Body: binary (`image/jpeg`)
- `304` — `If-None-Match` names the current `ETag`; no body. No body.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — The token was not issued by this node. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `502` — The image could not be downloaded or is not a supported image. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — This node cannot fetch or process images right now. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/images/<token>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/images/<token>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/images/<token>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/images/<token>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/images/<token>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The image as a JPEG. The body is binary (`image/jpeg`).

Schema:

A `image/jpeg` body: binary bytes.

Status `304`: `If-None-Match` names the current `ETag`; no body. No body.

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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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

Status `404`: The token was not issued by this node.

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

Status `502`: The image could not be downloaded or is not a supported image.

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

Status `503`: This node cannot fetch or process images right now. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "service_unavailable",
  "message": "The Bookclerk daemon is not ready to handle this request.",
  "status": 503
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

## `GET /api/stores/{id}/icon`

Serves a storefront's or integration's icon as a 128 px JPEG through the
cover service.

The icon is the one the plugin names for its sign-in button. A plugin
that is not deployed on this node, or names no icon, is a 404.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Storefront or integration id (`audible`, `libro`, `audiobookshelf`, …).

### Responses

- `200` — The icon as a JPEG. Body: binary (`image/jpeg`)
- `304` — `If-None-Match` names the current `ETag`; no body. No body.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No deployed storefront or integration has this id, or it names no icon. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `502` — The icon could not be downloaded or is not a supported image. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — This node cannot fetch or process images right now. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/stores/<id>/icon' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/stores/<id>/icon' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/stores/<id>/icon",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/stores/<id>/icon", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/stores/<id>/icon")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The icon as a JPEG. The body is binary (`image/jpeg`).

Schema:

A `image/jpeg` body: binary bytes.

Status `304`: `If-None-Match` names the current `ETag`; no body. No body.

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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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

Status `404`: No deployed storefront or integration has this id, or it names no icon.

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

Status `502`: The icon could not be downloaded or is not a supported image.

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

Status `503`: This node cannot fetch or process images right now. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "service_unavailable",
  "message": "The Bookclerk daemon is not ready to handle this request.",
  "status": 503
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

## `GET /api/discover/status`

Reports whether the caller's Discover shelves are stored yet, so
provisioning can wait for a new visitor's shelves before sending them in.

It also queues the caller's fetch or ranking when the stored data is
missing or stale, exactly as the Discover page read does.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Stored Discover state of the caller's shelves. Body: [`DiscoverStatusResponse`](schemas.md#discoverstatusresponse) (`application/json`)
  - `subject` *string*, **required** — The caller's subject key (`operator`, `user:<id>`, `portal:<id>`).
  - `ready` *boolean*, **required** — True once the caller has stored shelves to show.
  - `scope` *string or null* — Seeds the stored feed was ranked from (`own` or `household`), when ranked.
  - `feed_computed_at` *string or null* — RFC 3339 time the stored feed was ranked, when ranked.
  - `candidates_fetched_at` *string or null* — RFC 3339 time the caller's last storefront fetch finished, when any.
  - `updating` *boolean*, **required** — True while a fetch or ranking for the caller is waiting or running.
- `401` — The caller has no portal identity. Body: string (`text/plain`)
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The lookup failed. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/status' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/status' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/discover/status",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/status", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/discover/status")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Stored Discover state of the caller's shelves.

```json
{
  "subject": "user:2",
  "ready": true,
  "scope": "own",
  "feed_computed_at": "2026-11-06T00:30:00+00:00",
  "candidates_fetched_at": "2026-11-06T00:29:41+00:00",
  "updating": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Stored Discover state of the caller's shelves.",
  "required": [
    "subject",
    "ready",
    "updating"
  ],
  "properties": {
    "subject": {
      "type": "string",
      "description": "The caller's subject key (`operator`, `user:<id>`, `portal:<id>`).",
      "example": "user:2"
    },
    "ready": {
      "type": "boolean",
      "description": "True once the caller has stored shelves to show.",
      "example": true
    },
    "scope": {
      "type": [
        "string",
        "null"
      ],
      "description": "Seeds the stored feed was ranked from (`own` or `household`), when ranked.",
      "example": "own"
    },
    "feed_computed_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "RFC 3339 time the stored feed was ranked, when ranked.",
      "example": "2026-11-06T00:30:00+00:00"
    },
    "candidates_fetched_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "RFC 3339 time the caller's last storefront fetch finished, when any.",
      "example": "2026-11-06T00:29:41+00:00"
    },
    "updating": {
      "type": "boolean",
      "description": "True while a fetch or ranking for the caller is waiting or running.",
      "example": false
    }
  }
}
```

Status `401`: The caller has no portal identity.

```text
Unauthorized
```

Schema:

A `text/plain` body: a string.

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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

Status `500`: The lookup failed.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

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

## `POST /api/discover/refresh`

Queues a storefront fetch and ranking for the caller's shelves now, as
the Discover page's Refresh button asks. Unchanged store calls answer
from the day's cache, so a refresh is cheap; a fetch already waiting
absorbs the request.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Stored Discover state of the caller's shelves, with `updating` set once the fetch is queued. Body: [`DiscoverStatusResponse`](schemas.md#discoverstatusresponse) (`application/json`)
  - `subject` *string*, **required** — The caller's subject key (`operator`, `user:<id>`, `portal:<id>`).
  - `ready` *boolean*, **required** — True once the caller has stored shelves to show.
  - `scope` *string or null* — Seeds the stored feed was ranked from (`own` or `household`), when ranked.
  - `feed_computed_at` *string or null* — RFC 3339 time the stored feed was ranked, when ranked.
  - `candidates_fetched_at` *string or null* — RFC 3339 time the caller's last storefront fetch finished, when any.
  - `updating` *boolean*, **required** — True while a fetch or ranking for the caller is waiting or running.
- `401` — The caller has no portal identity. Body: string (`text/plain`)
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `429` — The job queue is full, so the fetch was not queued. Body: string (`text/plain`)
- `500` — The lookup failed, or the fetch could not be queued. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/refresh' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/refresh' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/refresh",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/refresh", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/refresh")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Stored Discover state of the caller's shelves, with `updating` set once the fetch is queued.

```json
{
  "subject": "user:2",
  "ready": true,
  "scope": "own",
  "feed_computed_at": "2026-11-06T00:30:00+00:00",
  "candidates_fetched_at": "2026-11-06T00:29:41+00:00",
  "updating": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Stored Discover state of the caller's shelves.",
  "required": [
    "subject",
    "ready",
    "updating"
  ],
  "properties": {
    "subject": {
      "type": "string",
      "description": "The caller's subject key (`operator`, `user:<id>`, `portal:<id>`).",
      "example": "user:2"
    },
    "ready": {
      "type": "boolean",
      "description": "True once the caller has stored shelves to show.",
      "example": true
    },
    "scope": {
      "type": [
        "string",
        "null"
      ],
      "description": "Seeds the stored feed was ranked from (`own` or `household`), when ranked.",
      "example": "own"
    },
    "feed_computed_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "RFC 3339 time the stored feed was ranked, when ranked.",
      "example": "2026-11-06T00:30:00+00:00"
    },
    "candidates_fetched_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "RFC 3339 time the caller's last storefront fetch finished, when any.",
      "example": "2026-11-06T00:29:41+00:00"
    },
    "updating": {
      "type": "boolean",
      "description": "True while a fetch or ranking for the caller is waiting or running.",
      "example": false
    }
  }
}
```

Status `401`: The caller has no portal identity.

```text
Unauthorized
```

Schema:

A `text/plain` body: a string.

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `429`: The job queue is full, so the fetch was not queued.

```text
job queue is full
```

Schema:

A `text/plain` body: a string.

Status `500`: The lookup failed, or the fetch could not be queued.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

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

## `GET /api/discover/recommendations`

Returns the caller's stored Discover shelves; never builds them on the request.

Shelves are ranked by queued `discover_candidates` and `discover_feed`
jobs. This route reads the stored feed, drops the caller's hidden shelves,
and queues a fetch or ranking when the stored data is missing or stale:
`state` is `preparing` (no shelves yet), `refreshing` (stored shelves,
newer ones queued) or `ready`.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The caller's stored Discover feed. Body: [`DiscoverFeed`](schemas.md#discoverfeed) (`application/json`)
  - `shelves` *array of [`DiscoverShelf`](schemas.md#discovershelf)*, **required** — Ordered horizontal shelves that make up the Discover page.
  - `shelf_kinds` *array of [`ShelfKindInfo`](schemas.md#shelfkindinfo)* — Catalog of shelf kinds for ignore prefs (empty `disabled_shelves` = all on).
  - `state` *[`FeedState`](schemas.md#feedstate) or null* — Whether these are the caller's current shelves, set by the daemon's read of the stored feed. Absent in a stored feed and from the CLI.
  - `computed_at` *string or null* — RFC 3339 time the shelves were ranked; absent while `state` is `preparing`.
- `401` — The caller has no portal identity. Body: string (`text/plain`)
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The lookup failed. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/recommendations' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/recommendations' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/discover/recommendations",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/recommendations", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/discover/recommendations")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's stored Discover feed.

```json
{
  "shelves": [
    {
      "id": "author:ursula-k-le-guin",
      "title": "More from Ursula K. Le Guin",
      "subtitle": "Because you’ve liked their work",
      "items": [
        {
          "title": "The Left Hand of Darkness",
          "score": 120.87,
          "reasons": [
            "wishlisted by 3 people",
            "matches liked author (Ursula K. Le Guin)"
          ],
          "purchase_hints": [],
          "from_request": true
        }
      ]
    }
  ],
  "shelf_kinds": [
    {
      "id": "finish_series",
      "label": "Finish these series"
    }
  ],
  "state": "ready",
  "computed_at": "2026-11-06T00:30:00Z"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Full Discover page payload: ordered shelves of personalized candidates.",
  "required": [
    "shelves"
  ],
  "properties": {
    "shelves": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One horizontal Discover row (series to finish, more by author, …).",
        "required": [
          "id",
          "title",
          "items"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Stable identifier for this item.",
            "example": "author:ursula-k-le-guin"
          },
          "title": {
            "type": "string",
            "description": "Display title as shown on the storefront or library card.",
            "example": "More from Ursula K. Le Guin"
          },
          "subtitle": {
            "type": [
              "string",
              "null"
            ],
            "description": "Optional subtitle when the catalog distinguishes it from the title.",
            "example": "Because you’ve liked their work"
          },
          "items": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One ranked recommendation (typically an unowned storefront title).",
              "required": [
                "title",
                "score",
                "reasons",
                "purchase_hints",
                "from_request"
              ],
              "properties": {
                "work_id": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Library `works.id` UUID when the recommendation is tied to an owned work.",
                  "example": "5b7e2c1d-8a4f-4e3b-9c6d-2f1a0b9e8d7c"
                },
                "title": {
                  "type": "string",
                  "description": "Display title as shown on the storefront or library card.",
                  "example": "The Left Hand of Darkness"
                },
                "authors": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Comma-separated author names when the storefront provides them.",
                  "example": "Ursula K. Le Guin"
                },
                "narrators": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Comma-separated narrator names when known.",
                  "example": "George Guidall"
                },
                "series": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Series name when the title belongs to a named series.",
                  "example": "Hainish Cycle"
                },
                "series_index": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Position within the series (e.g. `1`, `1.5`) when known.",
                  "example": "4"
                },
                "asin": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Audible / Amazon ASIN when this edition is sold on Audible.",
                  "example": "B002V0QK4C"
                },
                "isbn": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Canonical ISBN-13 (or ISBN-10 normalized) when published.",
                  "example": "9780441478125"
                },
                "score": {
                  "type": "number",
                  "format": "double",
                  "description": "Ranking score (higher is better); units are algorithm-specific.",
                  "example": 120.87
                },
                "reasons": {
                  "type": "array",
                  "items": {
                    "type": "string"
                  },
                  "description": "Short human-readable explanations for why this item was ranked here.",
                  "example": [
                    "wishlisted by 3 people",
                    "matches liked author (Ursula K. Le Guin)"
                  ]
                },
                "purchase_hints": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "A purchase / catalog availability hint (optionally priced at view time).",
                    "required": [
                      "source",
                      "product_id"
                    ],
                    "properties": {
                      "source": {
                        "type": "string",
                        "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                        "example": "libro"
                      },
                      "product_id": {
                        "type": "string",
                        "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                        "example": "9780441478125"
                      },
                      "title": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Display title as shown on the storefront or library card.",
                        "example": "The Left Hand of Darkness"
                      },
                      "url": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Absolute HTTPS URL.",
                        "example": "https://libro.fm/audiobooks/9780441478125"
                      },
                      "price_cents": {
                        "type": [
                          "integer",
                          "null"
                        ],
                        "format": "int64",
                        "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
                        "example": 1499
                      },
                      "currency": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
                        "example": "USD"
                      },
                      "price_label": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Display string from the store (`$2.99`, `FREE`, …).",
                        "example": "$14.99"
                      },
                      "list_price_cents": {
                        "type": [
                          "integer",
                          "null"
                        ],
                        "format": "int64",
                        "description": "Non-member / list price in integer cents when known.",
                        "example": 2999
                      },
                      "list_price_label": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Human-readable list price from the storefront.",
                        "example": "$29.99"
                      },
                      "member_price_cents": {
                        "type": [
                          "integer",
                          "null"
                        ],
                        "format": "int64",
                        "description": "Member / credit price in integer cents when known.",
                        "example": 1499
                      },
                      "member_price_label": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Human-readable member price from the storefront.",
                        "example": "$14.99"
                      }
                    }
                  },
                  "description": "Buy / open-in-store links with optional member vs list pricing."
                },
                "from_request": {
                  "type": "boolean",
                  "description": "True when sourced from an open title request rather than storefront discovery.",
                  "example": true
                },
                "request_uuid": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Wishlist / title-request UUID when this card came from a request.",
                  "example": "7c1e5a2b-3d4f-4e6a-9b8c-0d1e2f3a4b5c"
                },
                "candidate_source": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Storefront that proposed this title (`audible`, `libro`, …).",
                  "example": "libro"
                },
                "candidate_product_id": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Storefront product id of the edition that produced this card.",
                  "example": "9780441478125"
                },
                "store_editions": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "One storefront edition of a work (for multi-store purchase links).",
                    "required": [
                      "source",
                      "product_id"
                    ],
                    "properties": {
                      "source": {
                        "type": "string",
                        "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                        "example": "libro"
                      },
                      "product_id": {
                        "type": "string",
                        "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                        "example": "9780441478125"
                      }
                    }
                  },
                  "description": "All known storefront editions (for multi-store purchase links)."
                },
                "seed_categories": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Categories/subjects copied from the taste seed that produced this hit.",
                  "example": "Science Fiction & Fantasy > Science Fiction"
                },
                "categories": {
                  "type": "array",
                  "items": {
                    "type": "string"
                  },
                  "description": "Stable shelf-kind tags for prefs (`finish_series`, `author`, `chirp_deals`, …).\nDynamic shelf titles stay in `build_discover_feed`; membership uses these.",
                  "example": [
                    "finish_series",
                    "author"
                  ]
                },
                "work_key": {
                  "type": "string",
                  "description": "Stable bibliographic key for wishlist / merge (`isbn:` / `asin:` / `soft:`…).",
                  "example": "isbn:9780441478125"
                },
                "cover_url": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Cover art URL when known from a storefront catalog hit or wishlist row.",
                  "example": "https://images.example.com/covers/9780441478125.jpg"
                },
                "cover_path": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Same-origin path of this cover's processed images, set by the daemon\nwhen `cover_url` is an `https` URL. Append `/128`\n(thumbnail), `/360` (card) or `/full`, so a client can show covers\nwithout loading `cover_url` from another site.",
                  "example": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA"
                },
                "subtitle": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Optional bibliographic extras from storefront catalog / Audnexus.",
                  "example": "A Hainish Novel"
                },
                "description": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Publisher or storefront synopsis; may be truncated for embeddings.",
                  "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
                },
                "publisher": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Publisher imprint as reported by the catalog.",
                  "example": "Blackstone Publishing"
                },
                "length_minutes": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Audiobook runtime in whole minutes when known.",
                  "example": 560
                },
                "published_at": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Publication date string from the catalog (ISO or storefront format).",
                  "example": "2005-06-21"
                },
                "genres": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Candidate genres (not taste-seed `seed_categories`).",
                  "example": "Science Fiction, Classics"
                },
                "language": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "BCP-47 / storefront language code (`en`, `de`, …) when known.",
                  "example": "en"
                },
                "wishers": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "Compact wisher shown on the global queue (avatar + label).",
                    "properties": {
                      "user_id": {
                        "type": [
                          "integer",
                          "null"
                        ],
                        "format": "int64",
                        "description": "First-party `users.id` when the portal identity is linked.",
                        "example": 7
                      },
                      "identity_id": {
                        "type": [
                          "integer",
                          "null"
                        ],
                        "format": "int64",
                        "description": "Portal identity that created the wish (dedupe for unlinked identities).",
                        "example": 7
                      },
                      "display_name": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Display name, identity label, or `\"Operator\"`.",
                        "example": "Dana Reader"
                      },
                      "login_name": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Local login name when the first-party user has one.",
                        "example": "dana"
                      },
                      "operator": {
                        "type": "boolean",
                        "description": "True when the wish came from the operator token (no portal user).",
                        "example": false
                      },
                      "has_avatar": {
                        "type": "boolean",
                        "description": "True when an uploaded avatar file exists (filled by the daemon).",
                        "example": true
                      },
                      "avatar_source": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Explicit picture choice when the user has set one.",
                        "example": "upload"
                      },
                      "gravatar_hash": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "SHA-256 hex of the contact email for Gravatar.",
                        "example": "84d2a1a1d9ba6a4c2f4b9d0e7a7c6b5f3e2d1c0b9a8f7e6d5c4b3a2918f7e6d5"
                      },
                      "picture_url": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "HTTPS picture from the wishing identity, when any.",
                        "example": "https://accounts.example.com/u/dana/picture.png"
                      },
                      "picture_path": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Same-origin 128 px copy of `picture_url` for the web app to show\n(filled by the daemon; never stored).",
                        "example": "/api/images/aHR0cHM6Ly9hY2NvdW50cy5leGFtcGxlLmNvbS91L2RhbmEvcGljdHVyZS5wbmc.N14rqh66Bok9-eCqJKBNoQ"
                      }
                    }
                  },
                  "description": "Distinct people who have this title on an open wishlist."
                },
                "wish_count": {
                  "type": "integer",
                  "format": "int64",
                  "description": "Number of distinct users who wishlisted this title (may exceed `wishers.len()`).",
                  "example": 3
                },
                "seed_title": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Title of the book or wish the storefront was asked about to find this\ncard, when it came from such an expansion.",
                  "example": "Dune"
                },
                "starter_genre": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Starter genre this card was browsed from while the taste was thin.",
                  "example": "Science Fiction"
                }
              }
            },
            "description": "Ordered collection of child entries for this response."
          }
        }
      },
      "description": "Ordered horizontal shelves that make up the Discover page."
    },
    "shelf_kinds": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A shelf kind the operator can ignore (all offered by default).",
        "required": [
          "id",
          "label"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Stable kind id (`finish_series`, `author`, `genre`, `from_store`, …).",
            "example": "finish_series"
          },
          "label": {
            "type": "string",
            "description": "Operator-visible label for this account or item.",
            "example": "Finish these series"
          }
        }
      },
      "description": "Catalog of shelf kinds for ignore prefs (empty `disabled_shelves` = all on)."
    },
    "state": {
      "oneOf": [
        {
          "type": "string",
          "description": "Whether these are the caller's current shelves, set by the daemon's\nread of the stored feed. Absent in a stored feed and from the CLI.",
          "enum": [
            "ready",
            "refreshing",
            "preparing"
          ]
        },
        {
          "type": "null"
        }
      ]
    },
    "computed_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "RFC 3339 time the shelves were ranked; absent while `state` is `preparing`.",
      "example": "2026-11-06T00:30:00Z"
    }
  }
}
```

Status `401`: The caller has no portal identity.

```text
Unauthorized
```

Schema:

A `text/plain` body: a string.

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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

Status `500`: The lookup failed.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

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

## `POST /api/discover/purchase-hints`

Resolves storefront price/availability for one title, forcing server-side preferred stores.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PurchaseHintsQuery`](schemas.md#purchasehintsquery)

- `title` *string*, **required** — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `candidate_source` *string or null* — Storefront id of the edition that produced this card.
- `candidate_product_id` *string or null* — Storefront product id of the edition that produced this card.
- `store_editions` *array of [`StoreEdition`](schemas.md#storeedition)* — Known storefront editions already on the recommendation card.
- `region` *string or null* — Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `preferred_sources` *array of string* — Storefronts the caller has linked accounts for. Treated as **member** pricing when picking `best`; other stores are compared at list / non-member price. Every catalog match is still returned in `hints`.

### Responses

- `200` — Prices and availability. Body: [`PurchaseHintsResponse`](schemas.md#purchasehintsresponse) (`application/json`)
  - `hints` *array of [`PurchaseHint`](schemas.md#purchasehint)*, **required** — Resolved purchase hints for this query.
  - `best` *[`PurchaseHint`](schemas.md#purchasehint) or null* — Lowest-priced hint (or first catalog hit when no prices resolved).
  - `incomplete` *boolean* — True when the batch deadline cut this lookup short: the hints are the query's trusted seeds alone, so a client should not keep the answer and asks again later. Omitted (false) for a finished lookup.
- `400` — The query is malformed or too large. As `text/plain`, when the body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/purchase-hints' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"title": "The Left Hand of Darkness"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/purchase-hints' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"title": "The Left Hand of Darkness"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/purchase-hints",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"title": "The Left Hand of Darkness"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/purchase-hints", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"title": "The Left Hand of Darkness"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/purchase-hints")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"title": "The Left Hand of Darkness"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Prices and availability.

```json
{
  "hints": [
    {
      "source": "libro",
      "product_id": "9780441478125",
      "title": "The Left Hand of Darkness",
      "url": "https://libro.fm/audiobooks/9780441478125",
      "price_cents": 1499,
      "currency": "USD",
      "price_label": "$14.99",
      "list_price_cents": 2999,
      "list_price_label": "$29.99",
      "member_price_cents": 1499,
      "member_price_label": "$14.99"
    }
  ],
  "best": {
    "source": "libro",
    "product_id": "9780441478125",
    "title": "The Left Hand of Darkness",
    "url": "https://libro.fm/audiobooks/9780441478125",
    "price_cents": 1499,
    "currency": "USD",
    "price_label": "$14.99",
    "list_price_cents": 2999,
    "list_price_label": "$29.99",
    "member_price_cents": 1499,
    "member_price_label": "$14.99"
  },
  "incomplete": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Priced catalog matches for one title, sorted best-first.",
  "required": [
    "hints"
  ],
  "properties": {
    "hints": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A purchase / catalog availability hint (optionally priced at view time).",
        "required": [
          "source",
          "product_id"
        ],
        "properties": {
          "source": {
            "type": "string",
            "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
            "example": "libro"
          },
          "product_id": {
            "type": "string",
            "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
            "example": "9780441478125"
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title as shown on the storefront or library card.",
            "example": "The Left Hand of Darkness"
          },
          "url": {
            "type": [
              "string",
              "null"
            ],
            "description": "Absolute HTTPS URL.",
            "example": "https://libro.fm/audiobooks/9780441478125"
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
            "example": 1499
          },
          "currency": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
            "example": "USD"
          },
          "price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display string from the store (`$2.99`, `FREE`, …).",
            "example": "$14.99"
          },
          "list_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Non-member / list price in integer cents when known.",
            "example": 2999
          },
          "list_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Human-readable list price from the storefront.",
            "example": "$29.99"
          },
          "member_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Member / credit price in integer cents when known.",
            "example": 1499
          },
          "member_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Human-readable member price from the storefront.",
            "example": "$14.99"
          }
        }
      },
      "description": "Resolved purchase hints for this query."
    },
    "best": {
      "oneOf": [
        {
          "type": "object",
          "description": "Lowest-priced hint (or first catalog hit when no prices resolved).",
          "required": [
            "source",
            "product_id"
          ],
          "properties": {
            "source": {
              "type": "string",
              "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
              "example": "libro"
            },
            "product_id": {
              "type": "string",
              "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
              "example": "9780441478125"
            },
            "title": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display title as shown on the storefront or library card.",
              "example": "The Left Hand of Darkness"
            },
            "url": {
              "type": [
                "string",
                "null"
              ],
              "description": "Absolute HTTPS URL.",
              "example": "https://libro.fm/audiobooks/9780441478125"
            },
            "price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
              "example": 1499
            },
            "currency": {
              "type": [
                "string",
                "null"
              ],
              "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
              "example": "USD"
            },
            "price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display string from the store (`$2.99`, `FREE`, …).",
              "example": "$14.99"
            },
            "list_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Non-member / list price in integer cents when known.",
              "example": 2999
            },
            "list_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Human-readable list price from the storefront.",
              "example": "$29.99"
            },
            "member_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Member / credit price in integer cents when known.",
              "example": 1499
            },
            "member_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Human-readable member price from the storefront.",
              "example": "$14.99"
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "incomplete": {
      "type": "boolean",
      "description": "True when the batch deadline cut this lookup short: the hints are the\nquery's trusted seeds alone, so a client should not keep the answer\nand asks again later. Omitted (false) for a finished lookup.",
      "example": false
    }
  }
}
```

Status `400`: The query is malformed or too large. As `text/plain`, when the body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
Failed to deserialize the JSON body into the target type: missing field `title` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

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

## `POST /api/discover/purchase-hints/batch`

Resolves up to 24 purchase-hint queries with caller-derived preferred stores.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PurchaseHintsBatchBody`](schemas.md#purchasehintsbatchbody)

- `queries` *array of [`PurchaseHintsQuery`](schemas.md#purchasehintsquery)* — Per-title purchase-hint queries in this batch.

### Responses

- `200` — One result per query, in order. Body: [`PurchaseHintsBatchResponse`](schemas.md#purchasehintsbatchresponse) (`application/json`)
  - `results` *array of [`PurchaseHintsResponse`](schemas.md#purchasehintsresponse)*, **required** — One purchase-hint response per query, in request order.
- `400` — The query is malformed or too large. As `text/plain`, when the body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — A lookup task was cancelled before it answered. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/purchase-hints/batch' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/purchase-hints/batch' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/purchase-hints/batch",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/purchase-hints/batch", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/purchase-hints/batch")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: One result per query, in order.

```json
{
  "results": [
    {
      "hints": [
        {
          "source": "libro",
          "product_id": "9780441478125"
        }
      ],
      "best": {
        "source": "libro",
        "product_id": "9780441478125",
        "title": "The Left Hand of Darkness",
        "url": "https://libro.fm/audiobooks/9780441478125",
        "price_cents": 1499,
        "currency": "USD",
        "price_label": "$14.99",
        "list_price_cents": 2999,
        "list_price_label": "$29.99",
        "member_price_cents": 1499,
        "member_price_label": "$14.99"
      },
      "incomplete": false
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "Parallel purchase-hint results aligned with the request queries.",
  "required": [
    "results"
  ],
  "properties": {
    "results": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Priced catalog matches for one title, sorted best-first.",
        "required": [
          "hints"
        ],
        "properties": {
          "hints": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "A purchase / catalog availability hint (optionally priced at view time).",
              "required": [
                "source",
                "product_id"
              ],
              "properties": {
                "source": {
                  "type": "string",
                  "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                  "example": "libro"
                },
                "product_id": {
                  "type": "string",
                  "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                  "example": "9780441478125"
                },
                "title": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Display title as shown on the storefront or library card.",
                  "example": "The Left Hand of Darkness"
                },
                "url": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Absolute HTTPS URL.",
                  "example": "https://libro.fm/audiobooks/9780441478125"
                },
                "price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
                  "example": 1499
                },
                "currency": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
                  "example": "USD"
                },
                "price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Display string from the store (`$2.99`, `FREE`, …).",
                  "example": "$14.99"
                },
                "list_price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Non-member / list price in integer cents when known.",
                  "example": 2999
                },
                "list_price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Human-readable list price from the storefront.",
                  "example": "$29.99"
                },
                "member_price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Member / credit price in integer cents when known.",
                  "example": 1499
                },
                "member_price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Human-readable member price from the storefront.",
                  "example": "$14.99"
                }
              }
            },
            "description": "Resolved purchase hints for this query."
          },
          "best": {
            "oneOf": [
              {
                "type": "object",
                "description": "Lowest-priced hint (or first catalog hit when no prices resolved).",
                "required": [
                  "source",
                  "product_id"
                ],
                "properties": {
                  "source": {
                    "type": "string",
                    "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                    "example": "libro"
                  },
                  "product_id": {
                    "type": "string",
                    "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                    "example": "9780441478125"
                  },
                  "title": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Display title as shown on the storefront or library card.",
                    "example": "The Left Hand of Darkness"
                  },
                  "url": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Absolute HTTPS URL.",
                    "example": "https://libro.fm/audiobooks/9780441478125"
                  },
                  "price_cents": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
                    "example": 1499
                  },
                  "currency": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
                    "example": "USD"
                  },
                  "price_label": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Display string from the store (`$2.99`, `FREE`, …).",
                    "example": "$14.99"
                  },
                  "list_price_cents": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Non-member / list price in integer cents when known.",
                    "example": 2999
                  },
                  "list_price_label": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Human-readable list price from the storefront.",
                    "example": "$29.99"
                  },
                  "member_price_cents": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Member / credit price in integer cents when known.",
                    "example": 1499
                  },
                  "member_price_label": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Human-readable member price from the storefront.",
                    "example": "$14.99"
                  }
                }
              },
              {
                "type": "null"
              }
            ]
          },
          "incomplete": {
            "type": "boolean",
            "description": "True when the batch deadline cut this lookup short: the hints are the\nquery's trusted seeds alone, so a client should not keep the answer\nand asks again later. Omitted (false) for a finished lookup.",
            "example": false
          }
        }
      },
      "description": "One purchase-hint response per query, in request order."
    }
  }
}
```

Status `400`: The query is malformed or too large. As `text/plain`, when the body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
Failed to deserialize the JSON body into the target type: queries: invalid type: integer `1`, expected a sequence at line 1 column 12
```

Schema:

A `text/plain` body: a string.

Status `500`: A lookup task was cancelled before it answered.

```text
purchase-hints batch task cancelled
```

Schema:

A `text/plain` body: a string.

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

## `GET /api/discover/search`

Runs a multi-store catalog search with a 15s wall budget and empty-query short circuit.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `q` (query) *string* — Search text; queries shorter than two characters return an empty page. Rejected above 256 UTF-8 bytes as a downstream catalog-search limit (the HTTP query extractor has already materialized `q`).
- `limit` (query) *integer* — Alias for [`Self::page_size`] (typeahead / legacy).
- `page_size` (query) *integer* — Page size (1–48); falls back to `limit` then 24.
- `cursor` (query) *string* — Opaque cursor from a previous [`bookclerk_discover::CatalogSearchPage`].
- `sort` (query) *string* — `relevance` / `popularity` / `rating` / `title` / `author` / `price` / `length`.
- `sort_dir` (query) *string* — `asc` / `desc` (defaults per sort when omitted).
- `region` (query) *string* — Storefront region code (default `us`).
- `field` (query) *string* — Optional facet scope: `author` / `narrator` / `series` / `genre`.
- `lang` (query) *string* — Preferred / default content language (BCP-47 primary). When `language` is omitted and `all_languages` is not set, applied as a hard include.
- `language` (query) *string* — Comma-separated language codes to include (`en`, `zh`, …).
- `all_languages` (query) *boolean* — When true, do not hard-filter by language (soft prefer via `lang` only).
- `author` (query) *string* — Comma-separated include filters (OR within a kind).
- `narrator` (query) *string* — Comma-separated narrator include filters.
- `series` (query) *string* — Comma-separated series include filters.
- `genre` (query) *string* — Comma-separated genre include filters.
- `source` (query) *string* — Comma-separated store ids to include.
- `exclude_source` (query) *string* — Comma-separated store ids to exclude.
- `exclude_narrator` (query) *string* — Comma-separated narrator substrings to exclude (e.g. `Virtual Voice`).
- `min_rating` (query) *number (double)* — Minimum overall rating (0–5); missing ratings still pass.
- `min_length_minutes` (query) *integer (int64)* — Minimum runtime in minutes; values ≤ 0 are ignored.
- `max_length_minutes` (query) *integer (int64)* — Maximum runtime in minutes; values ≤ 0 are ignored.

### Responses

- `200` — A page of catalog results. Body: [`CatalogSearchPage`](schemas.md#catalogsearchpage) (`application/json`)
  - `items` *array of [`CatalogSearchHit`](schemas.md#catalogsearchhit)*, **required** — Ordered collection of child entries for this response.
  - `page_size` *integer*, **required** — Maximum number of items returned in this page. At least 0.
  - `has_more` *boolean*, **required** — When true, another page is available via [`Self::next_cursor`].
  - `next_cursor` *string or null* — Opaque cursor for the next page; omit or `None` when exhausted.
  - `sort` *string*, **required** — Sort field applied to the merged result set.
  - `sort_dir` *string* — Ascending or descending order for [`Self::sort`].
- `400` — The query is malformed or too large. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The stores did not answer within the search budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/search' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/search' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/discover/search",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/search", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/discover/search")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A page of catalog results.

```json
{
  "items": [
    {
      "work_key": "isbn:9780441478125",
      "title": "The Left Hand of Darkness",
      "authors": "Ursula K. Le Guin",
      "narrators": "George Guidall",
      "series": "Hainish Cycle",
      "series_index": "4",
      "asin": "B002V0QK4C",
      "isbn": "9780441478125",
      "cover_url": "https://images.example.com/covers/9780441478125.jpg",
      "cover_path": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA",
      "store_editions": [
        {
          "source": "libro",
          "product_id": "9780441478125"
        }
      ],
      "sources": [
        "libro",
        "chirp"
      ],
      "subtitle": "A Hainish Novel",
      "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
      "publisher": "Blackstone Publishing",
      "length_minutes": 560,
      "published_at": "2005-06-21",
      "genres": "Science Fiction, Classics",
      "language": "en",
      "is_abridged": false,
      "rating_overall": 4.5,
      "rating_count": 12840,
      "price_cents": 2499,
      "purchase_hints": [
        {
          "source": "libro",
          "product_id": "9780441478125"
        }
      ]
    }
  ],
  "page_size": 20,
  "has_more": true,
  "next_cursor": "s1:4f1c9e2a-7b3d-4c8e-9a6f-0d2b1e3c5a7f",
  "sort": "relevance",
  "sort_dir": "desc"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Paged catalog-search response.",
  "required": [
    "items",
    "page_size",
    "has_more",
    "sort"
  ],
  "properties": {
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One autocomplete / results-page hit (possibly spanning multiple storefronts).",
        "required": [
          "work_key",
          "title",
          "store_editions",
          "sources"
        ],
        "properties": {
          "work_key": {
            "type": "string",
            "description": "Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).",
            "example": "isbn:9780441478125"
          },
          "title": {
            "type": "string",
            "description": "Display title as shown on the storefront or library card.",
            "example": "The Left Hand of Darkness"
          },
          "authors": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated author names when the storefront provides them.",
            "example": "Ursula K. Le Guin"
          },
          "narrators": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated narrator names when known.",
            "example": "George Guidall"
          },
          "series": {
            "type": [
              "string",
              "null"
            ],
            "description": "Series name when the title belongs to a named series.",
            "example": "Hainish Cycle"
          },
          "series_index": {
            "type": [
              "string",
              "null"
            ],
            "description": "Position within the series (e.g. `1`, `1.5`) when known.",
            "example": "4"
          },
          "asin": {
            "type": [
              "string",
              "null"
            ],
            "description": "Audible / Amazon ASIN when this edition is sold on Audible.",
            "example": "B002V0QK4C"
          },
          "isbn": {
            "type": [
              "string",
              "null"
            ],
            "description": "Canonical ISBN-13 (or ISBN-10 normalized) when published.",
            "example": "9780441478125"
          },
          "cover_url": {
            "type": [
              "string",
              "null"
            ],
            "description": "HTTPS URL for cover art when the catalog exposes one.",
            "example": "https://images.example.com/covers/9780441478125.jpg"
          },
          "cover_path": {
            "type": [
              "string",
              "null"
            ],
            "description": "Same-origin path of this cover's processed images, set by the daemon\nwhen `cover_url` is an `https` URL. Append `/128`\n(thumbnail), `/360` (card) or `/full`, so a client can show covers\nwithout loading `cover_url` from another site.",
            "example": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA"
          },
          "store_editions": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One storefront edition of a work (for multi-store purchase links).",
              "required": [
                "source",
                "product_id"
              ],
              "properties": {
                "source": {
                  "type": "string",
                  "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                  "example": "libro"
                },
                "product_id": {
                  "type": "string",
                  "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                  "example": "9780441478125"
                }
              }
            },
            "description": "Per-storefront edition ids that collapsed into this work card."
          },
          "sources": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Storefronts that matched (deduped source ids).",
            "example": [
              "libro",
              "chirp"
            ]
          },
          "subtitle": {
            "type": [
              "string",
              "null"
            ],
            "description": "Optional subtitle when the catalog distinguishes it from the title.",
            "example": "A Hainish Novel"
          },
          "description": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publisher or storefront synopsis; may be truncated for embeddings.",
            "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
          },
          "publisher": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publisher imprint as reported by the catalog.",
            "example": "Blackstone Publishing"
          },
          "length_minutes": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Audiobook runtime in whole minutes when known.",
            "example": 560
          },
          "published_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publication date string from the catalog (ISO or storefront format).",
            "example": "2005-06-21"
          },
          "genres": {
            "type": [
              "string",
              "null"
            ],
            "description": "Genre labels from the catalog (comma-separated when multiple).",
            "example": "Science Fiction, Classics"
          },
          "language": {
            "type": [
              "string",
              "null"
            ],
            "description": "BCP-47 / storefront language code (`en`, `de`, …) when known.",
            "example": "en"
          },
          "is_abridged": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "When `Some(true)`, the edition is marked abridged by the storefront.",
            "example": false
          },
          "rating_overall": {
            "type": [
              "number",
              "null"
            ],
            "format": "double",
            "description": "Aggregate listener rating on the storefront's scale when known.",
            "example": 4.5
          },
          "rating_count": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Number of ratings backing [`Self::rating_overall`] when known.",
            "example": 12840
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Catalog list/deal price in cents when the storefront provided it.",
            "example": 2499
          },
          "purchase_hints": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "A purchase / catalog availability hint (optionally priced at view time).",
              "required": [
                "source",
                "product_id"
              ],
              "properties": {
                "source": {
                  "type": "string",
                  "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                  "example": "libro"
                },
                "product_id": {
                  "type": "string",
                  "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                  "example": "9780441478125"
                },
                "title": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Display title as shown on the storefront or library card.",
                  "example": "The Left Hand of Darkness"
                },
                "url": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Absolute HTTPS URL.",
                  "example": "https://libro.fm/audiobooks/9780441478125"
                },
                "price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
                  "example": 1499
                },
                "currency": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
                  "example": "USD"
                },
                "price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Display string from the store (`$2.99`, `FREE`, …).",
                  "example": "$14.99"
                },
                "list_price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Non-member / list price in integer cents when known.",
                  "example": 2999
                },
                "list_price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Human-readable list price from the storefront.",
                  "example": "$29.99"
                },
                "member_price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Member / credit price in integer cents when known.",
                  "example": 1499
                },
                "member_price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Human-readable member price from the storefront.",
                  "example": "$14.99"
                }
              }
            },
            "description": "Buy / open-in-store links with optional member vs list pricing."
          }
        }
      },
      "description": "Ordered collection of child entries for this response."
    },
    "page_size": {
      "type": "integer",
      "description": "Maximum number of items returned in this page.",
      "example": 20,
      "minimum": 0
    },
    "has_more": {
      "type": "boolean",
      "description": "When true, another page is available via [`Self::next_cursor`].",
      "example": true
    },
    "next_cursor": {
      "type": [
        "string",
        "null"
      ],
      "description": "Opaque cursor for the next page; omit or `None` when exhausted.",
      "example": "s1:4f1c9e2a-7b3d-4c8e-9a6f-0d2b1e3c5a7f"
    },
    "sort": {
      "type": "string",
      "description": "Sort field applied to the merged result set.",
      "example": "relevance"
    },
    "sort_dir": {
      "type": "string",
      "description": "Ascending or descending order for [`Self::sort`].",
      "example": "desc"
    }
  }
}
```

Status `400`: The query is malformed or too large.

```text
catalog query must be 256 bytes or fewer
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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

Status `504`: The stores did not answer within the search budget.

```text
catalog search timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/discover/title-meta`

Resolves Audnexus-style title metadata for one query.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleMetaQuery`](schemas.md#titlemetaquery)

- `title` *string* — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `narrators` *string or null* — Comma-separated narrator names when known.
- `length_minutes` *integer (int64) or null* — Audiobook runtime in whole minutes when known.
- `region` *string* — Marketplace / region code (`us`, `uk`, …) for catalog lookups.

### Responses

- `200` — The metadata, or null when nothing matched. Body: [`TitleMeta`](schemas.md#titlemeta) or null (`application/json`)
  - `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
  - `title` *string or null* — Display title as shown on the storefront or library card.
  - `subtitle` *string or null* — Optional subtitle when the catalog distinguishes it from the title.
  - `authors` *string or null* — Comma-separated author names when the storefront provides them.
  - `narrators` *string or null* — Comma-separated narrator names when known.
  - `series` *string or null* — Series name when the title belongs to a named series.
  - `series_index` *string or null* — Position within the series (e.g. `1`, `1.5`) when known.
  - `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
  - `cover_url` *string or null* — HTTPS URL for cover art when the catalog exposes one.
  - `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
  - `description` *string or null* — Publisher or storefront synopsis; may be truncated for embeddings.
  - `publisher` *string or null* — Publisher imprint as reported by the catalog.
  - `length_minutes` *integer (int64) or null* — Audiobook runtime in whole minutes when known.
  - `published_at` *string or null* — Publication date string from the catalog (ISO or storefront format).
  - `categories` *string or null* — Genre / subject tags as a single delimited string when known.
  - `language` *string or null* — BCP-47 / storefront language code (`en`, `de`, …) when known.
  - `is_abridged` *boolean or null* — `Some(true)` abridged / `Some(false)` unabridged when the storefront said so.
  - `rating_overall` *number (double) or null* — Audible community overall rating (0–5).
  - `rating_performance` *number (double) or null* — Narration / performance sub-score when the storefront splits ratings.
  - `rating_story` *number (double) or null* — Story / content sub-score when the storefront splits ratings.
  - `rating_count` *integer (int64) or null* — Count of star ratings (Audible `num_ratings`).
  - `review_count` *integer (int64) or null* — Count of written reviews (Audible `num_reviews`).
  - `reviews` *array of [`TitleReview`](schemas.md#titlereview)* — Deprecated: reviews load via `/api/discover/title-reviews`. Kept empty for older clients; prefer the paginated reviews endpoint.
- `400` — The query is malformed or too large. As `text/plain`, when the body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/title-meta' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/title-meta' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/title-meta",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/title-meta", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/title-meta")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The metadata, or null when nothing matched.

```json
{
  "asin": "B002V0QK4C",
  "title": "The Left Hand of Darkness",
  "subtitle": "A Hainish Novel",
  "authors": "Ursula K. Le Guin",
  "narrators": "George Guidall",
  "series": "Hainish Cycle",
  "series_index": "4",
  "isbn": "9780441478125",
  "cover_url": "https://images.example.com/covers/9780441478125.jpg",
  "cover_path": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA",
  "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
  "publisher": "Blackstone Publishing",
  "length_minutes": 560,
  "published_at": "2005-06-21",
  "categories": "Science Fiction & Fantasy; Science Fiction",
  "language": "en",
  "is_abridged": false,
  "rating_overall": 4.5,
  "rating_performance": 4.7,
  "rating_story": 4.3,
  "rating_count": 12840,
  "review_count": 1932,
  "reviews": [
    {
      "id": "R3K2M9P1Q7X4Z",
      "title": "Still the best",
      "body": "Guidall's narration carries the quiet of Winter; I listened twice.",
      "author_name": "Dana",
      "overall_rating": 5,
      "performance_rating": 5,
      "story_rating": 4,
      "submitted_at": "2026-02-11"
    }
  ]
}
```

Schema:

```json
{
  "oneOf": [
    {
      "type": "object",
      "description": "Bibliographic fields suitable for an Audible-style detail panel.",
      "properties": {
        "asin": {
          "type": [
            "string",
            "null"
          ],
          "description": "Audible / Amazon ASIN when this edition is sold on Audible.",
          "example": "B002V0QK4C"
        },
        "title": {
          "type": [
            "string",
            "null"
          ],
          "description": "Display title as shown on the storefront or library card.",
          "example": "The Left Hand of Darkness"
        },
        "subtitle": {
          "type": [
            "string",
            "null"
          ],
          "description": "Optional subtitle when the catalog distinguishes it from the title.",
          "example": "A Hainish Novel"
        },
        "authors": {
          "type": [
            "string",
            "null"
          ],
          "description": "Comma-separated author names when the storefront provides them.",
          "example": "Ursula K. Le Guin"
        },
        "narrators": {
          "type": [
            "string",
            "null"
          ],
          "description": "Comma-separated narrator names when known.",
          "example": "George Guidall"
        },
        "series": {
          "type": [
            "string",
            "null"
          ],
          "description": "Series name when the title belongs to a named series.",
          "example": "Hainish Cycle"
        },
        "series_index": {
          "type": [
            "string",
            "null"
          ],
          "description": "Position within the series (e.g. `1`, `1.5`) when known.",
          "example": "4"
        },
        "isbn": {
          "type": [
            "string",
            "null"
          ],
          "description": "Canonical ISBN-13 (or ISBN-10 normalized) when published.",
          "example": "9780441478125"
        },
        "cover_url": {
          "type": [
            "string",
            "null"
          ],
          "description": "HTTPS URL for cover art when the catalog exposes one.",
          "example": "https://images.example.com/covers/9780441478125.jpg"
        },
        "cover_path": {
          "type": [
            "string",
            "null"
          ],
          "description": "Same-origin path of this cover's processed images, set by the daemon\nwhen `cover_url` is an `https` URL. Append `/128`\n(thumbnail), `/360` (card) or `/full`, so a client can show covers\nwithout loading `cover_url` from another site.",
          "example": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA"
        },
        "description": {
          "type": [
            "string",
            "null"
          ],
          "description": "Publisher or storefront synopsis; may be truncated for embeddings.",
          "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
        },
        "publisher": {
          "type": [
            "string",
            "null"
          ],
          "description": "Publisher imprint as reported by the catalog.",
          "example": "Blackstone Publishing"
        },
        "length_minutes": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Audiobook runtime in whole minutes when known.",
          "example": 560
        },
        "published_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "Publication date string from the catalog (ISO or storefront format).",
          "example": "2005-06-21"
        },
        "categories": {
          "type": [
            "string",
            "null"
          ],
          "description": "Genre / subject tags as a single delimited string when known.",
          "example": "Science Fiction & Fantasy; Science Fiction"
        },
        "language": {
          "type": [
            "string",
            "null"
          ],
          "description": "BCP-47 / storefront language code (`en`, `de`, …) when known.",
          "example": "en"
        },
        "is_abridged": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "`Some(true)` abridged / `Some(false)` unabridged when the storefront said so.",
          "example": false
        },
        "rating_overall": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Audible community overall rating (0–5).",
          "example": 4.5
        },
        "rating_performance": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Narration / performance sub-score when the storefront splits ratings.",
          "example": 4.7
        },
        "rating_story": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Story / content sub-score when the storefront splits ratings.",
          "example": 4.3
        },
        "rating_count": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Count of star ratings (Audible `num_ratings`).",
          "example": 12840
        },
        "review_count": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Count of written reviews (Audible `num_reviews`).",
          "example": 1932
        },
        "reviews": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "One Audible customer review for the title detail panel.",
            "required": [
              "body"
            ],
            "properties": {
              "id": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Stable identifier for this item.",
                "example": "R3K2M9P1Q7X4Z"
              },
              "title": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Display title as shown on the storefront or library card.",
                "example": "Still the best"
              },
              "body": {
                "type": "string",
                "description": "Full review text from the storefront.",
                "example": "Guidall's narration carries the quiet of Winter; I listened twice."
              },
              "author_name": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Display name of the review author on the storefront.",
                "example": "Dana"
              },
              "overall_rating": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Overall star rating for this review when present.",
                "example": 5
              },
              "performance_rating": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Narration sub-rating for this review when present.",
                "example": 5
              },
              "story_rating": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Story sub-rating for this review when present.",
                "example": 4
              },
              "submitted_at": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "When the review was submitted (storefront timestamp string).",
                "example": "2026-02-11"
              }
            }
          },
          "description": "Deprecated: reviews load via `/api/discover/title-reviews`. Kept empty\nfor older clients; prefer the paginated reviews endpoint."
        }
      }
    },
    {
      "type": "null"
    }
  ]
}
```

Status `400`: The query is malformed or too large. As `text/plain`, when the body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
Failed to deserialize the JSON body into the target type: title: invalid type: integer `1`, expected a string at line 1 column 10
```

Schema:

A `text/plain` body: a string.

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

## `POST /api/discover/title-meta/batch`

Resolves up to 24 title-metadata queries without failing the whole batch on one error.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleMetaBatchBody`](schemas.md#titlemetabatchbody)

- `queries` *array of [`TitleMetaQuery`](schemas.md#titlemetaquery)* — Per-title metadata queries in this batch.

### Responses

- `200` — One result per query, in order. Body: [`TitleMetaBatchResponse`](schemas.md#titlemetabatchresponse) (`application/json`)
  - `results` *array of [`TitleMeta`](schemas.md#titlemeta) or null*, **required** — One optional metadata payload per query; enrichment errors become `None`.
- `400` — The query is malformed or too large. As `text/plain`, when the body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/title-meta/batch' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/title-meta/batch' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/title-meta/batch",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/title-meta/batch", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/title-meta/batch")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: One result per query, in order.

```json
{
  "results": [
    {
      "asin": "B002V0QK4C",
      "title": "The Left Hand of Darkness",
      "subtitle": "A Hainish Novel",
      "authors": "Ursula K. Le Guin",
      "narrators": "George Guidall",
      "series": "Hainish Cycle",
      "series_index": "4",
      "isbn": "9780441478125",
      "cover_url": "https://images.example.com/covers/9780441478125.jpg",
      "cover_path": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA",
      "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
      "publisher": "Blackstone Publishing",
      "length_minutes": 560,
      "published_at": "2005-06-21",
      "categories": "Science Fiction & Fantasy; Science Fiction",
      "language": "en",
      "is_abridged": false,
      "rating_overall": 4.5,
      "rating_performance": 4.7,
      "rating_story": 4.3,
      "rating_count": 12840,
      "review_count": 1932,
      "reviews": [
        {
          "body": "Guidall's narration carries the quiet of Winter; I listened twice."
        }
      ]
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "Title-metadata results aligned with the request; failed items are `null`.",
  "required": [
    "results"
  ],
  "properties": {
    "results": {
      "type": "array",
      "items": {
        "oneOf": [
          {
            "type": "object",
            "description": "Bibliographic fields suitable for an Audible-style detail panel.",
            "properties": {
              "asin": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Audible / Amazon ASIN when this edition is sold on Audible.",
                "example": "B002V0QK4C"
              },
              "title": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Display title as shown on the storefront or library card.",
                "example": "The Left Hand of Darkness"
              },
              "subtitle": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Optional subtitle when the catalog distinguishes it from the title.",
                "example": "A Hainish Novel"
              },
              "authors": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Comma-separated author names when the storefront provides them.",
                "example": "Ursula K. Le Guin"
              },
              "narrators": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Comma-separated narrator names when known.",
                "example": "George Guidall"
              },
              "series": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Series name when the title belongs to a named series.",
                "example": "Hainish Cycle"
              },
              "series_index": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Position within the series (e.g. `1`, `1.5`) when known.",
                "example": "4"
              },
              "isbn": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Canonical ISBN-13 (or ISBN-10 normalized) when published.",
                "example": "9780441478125"
              },
              "cover_url": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "HTTPS URL for cover art when the catalog exposes one.",
                "example": "https://images.example.com/covers/9780441478125.jpg"
              },
              "cover_path": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Same-origin path of this cover's processed images, set by the daemon\nwhen `cover_url` is an `https` URL. Append `/128`\n(thumbnail), `/360` (card) or `/full`, so a client can show covers\nwithout loading `cover_url` from another site.",
                "example": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA"
              },
              "description": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Publisher or storefront synopsis; may be truncated for embeddings.",
                "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
              },
              "publisher": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Publisher imprint as reported by the catalog.",
                "example": "Blackstone Publishing"
              },
              "length_minutes": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Audiobook runtime in whole minutes when known.",
                "example": 560
              },
              "published_at": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Publication date string from the catalog (ISO or storefront format).",
                "example": "2005-06-21"
              },
              "categories": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Genre / subject tags as a single delimited string when known.",
                "example": "Science Fiction & Fantasy; Science Fiction"
              },
              "language": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "BCP-47 / storefront language code (`en`, `de`, …) when known.",
                "example": "en"
              },
              "is_abridged": {
                "type": [
                  "boolean",
                  "null"
                ],
                "description": "`Some(true)` abridged / `Some(false)` unabridged when the storefront said so.",
                "example": false
              },
              "rating_overall": {
                "type": [
                  "number",
                  "null"
                ],
                "format": "double",
                "description": "Audible community overall rating (0–5).",
                "example": 4.5
              },
              "rating_performance": {
                "type": [
                  "number",
                  "null"
                ],
                "format": "double",
                "description": "Narration / performance sub-score when the storefront splits ratings.",
                "example": 4.7
              },
              "rating_story": {
                "type": [
                  "number",
                  "null"
                ],
                "format": "double",
                "description": "Story / content sub-score when the storefront splits ratings.",
                "example": 4.3
              },
              "rating_count": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Count of star ratings (Audible `num_ratings`).",
                "example": 12840
              },
              "review_count": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Count of written reviews (Audible `num_reviews`).",
                "example": 1932
              },
              "reviews": {
                "type": "array",
                "items": {
                  "type": "object",
                  "description": "One Audible customer review for the title detail panel.",
                  "required": [
                    "body"
                  ],
                  "properties": {
                    "id": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "Stable identifier for this item.",
                      "example": "R3K2M9P1Q7X4Z"
                    },
                    "title": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "Display title as shown on the storefront or library card.",
                      "example": "Still the best"
                    },
                    "body": {
                      "type": "string",
                      "description": "Full review text from the storefront.",
                      "example": "Guidall's narration carries the quiet of Winter; I listened twice."
                    },
                    "author_name": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "Display name of the review author on the storefront.",
                      "example": "Dana"
                    },
                    "overall_rating": {
                      "type": [
                        "integer",
                        "null"
                      ],
                      "format": "int64",
                      "description": "Overall star rating for this review when present.",
                      "example": 5
                    },
                    "performance_rating": {
                      "type": [
                        "integer",
                        "null"
                      ],
                      "format": "int64",
                      "description": "Narration sub-rating for this review when present.",
                      "example": 5
                    },
                    "story_rating": {
                      "type": [
                        "integer",
                        "null"
                      ],
                      "format": "int64",
                      "description": "Story sub-rating for this review when present.",
                      "example": 4
                    },
                    "submitted_at": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "When the review was submitted (storefront timestamp string).",
                      "example": "2026-02-11"
                    }
                  }
                },
                "description": "Deprecated: reviews load via `/api/discover/title-reviews`. Kept empty\nfor older clients; prefer the paginated reviews endpoint."
              }
            }
          },
          {
            "type": "null"
          }
        ]
      },
      "description": "One optional metadata payload per query; enrichment errors become `None`."
    }
  }
}
```

Status `400`: The query is malformed or too large. As `text/plain`, when the body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
Failed to deserialize the JSON body into the target type: queries: invalid type: integer `1`, expected a sequence at line 1 column 12
```

Schema:

A `text/plain` body: a string.

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

## `POST /api/discover/title-reviews`

Fetches Audnexus reviews for an ASIN; rejects an empty ASIN with 400.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleReviewsQuery`](schemas.md#titlereviewsquery)

- `asin` *string*, **required** — Audible / Amazon ASIN when this edition is sold on Audible.
- `region` *string* — Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `page` *integer (int32)* — 1-based page index. At least 0.
- `page_size` *integer (int32)* — Page size (clamped server-side to 1..=20). At least 0.
- `sort_by` *string* — `MostHelpful` (default) or `MostRecent`.

### Responses

- `200` — A page of reviews. Body: [`TitleReviewsPage`](schemas.md#titlereviewspage) (`application/json`)
  - `asin` *string*, **required** — Audible / Amazon ASIN when this edition is sold on Audible.
  - `page` *integer (int32)*, **required** — 1-based page index for this reviews / list request. At least 0.
  - `page_size` *integer (int32)*, **required** — Maximum number of items returned in this page. At least 0.
  - `has_more` *boolean*, **required** — When true, another page is available by incrementing [`Self::page`].
  - `sort_by` *string*, **required** — Remote sort key for reviews (`MostHelpful` or `MostRecent`, as applied).
  - `reviews` *array of [`TitleReview`](schemas.md#titlereview)*, **required** — Review rows on this page.
- `400` — The query is malformed or too large. As `text/plain`, when the body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/title-reviews' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"asin": "B002V0QK4C"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/title-reviews' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"asin": "B002V0QK4C"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/title-reviews",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"asin": "B002V0QK4C"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/title-reviews", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"asin": "B002V0QK4C"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/title-reviews")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"asin": "B002V0QK4C"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A page of reviews.

```json
{
  "asin": "B002V0QK4C",
  "page": 1,
  "page_size": 10,
  "has_more": true,
  "sort_by": "MostHelpful",
  "reviews": [
    {
      "id": "R3K2M9P1Q7X4Z",
      "title": "Still the best",
      "body": "Guidall's narration carries the quiet of Winter; I listened twice.",
      "author_name": "Dana",
      "overall_rating": 5,
      "performance_rating": 5,
      "story_rating": 4,
      "submitted_at": "2026-02-11"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "One page of title reviews for infinite scroll.",
  "required": [
    "asin",
    "page",
    "page_size",
    "has_more",
    "sort_by",
    "reviews"
  ],
  "properties": {
    "asin": {
      "type": "string",
      "description": "Audible / Amazon ASIN when this edition is sold on Audible.",
      "example": "B002V0QK4C"
    },
    "page": {
      "type": "integer",
      "format": "int32",
      "description": "1-based page index for this reviews / list request.",
      "example": 1,
      "minimum": 0
    },
    "page_size": {
      "type": "integer",
      "format": "int32",
      "description": "Maximum number of items returned in this page.",
      "example": 10,
      "minimum": 0
    },
    "has_more": {
      "type": "boolean",
      "description": "When true, another page is available by incrementing [`Self::page`].",
      "example": true
    },
    "sort_by": {
      "type": "string",
      "description": "Remote sort key for reviews (`MostHelpful` or `MostRecent`, as applied).",
      "example": "MostHelpful"
    },
    "reviews": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One Audible customer review for the title detail panel.",
        "required": [
          "body"
        ],
        "properties": {
          "id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Stable identifier for this item.",
            "example": "R3K2M9P1Q7X4Z"
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title as shown on the storefront or library card.",
            "example": "Still the best"
          },
          "body": {
            "type": "string",
            "description": "Full review text from the storefront.",
            "example": "Guidall's narration carries the quiet of Winter; I listened twice."
          },
          "author_name": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display name of the review author on the storefront.",
            "example": "Dana"
          },
          "overall_rating": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Overall star rating for this review when present.",
            "example": 5
          },
          "performance_rating": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Narration sub-rating for this review when present.",
            "example": 5
          },
          "story_rating": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Story sub-rating for this review when present.",
            "example": 4
          },
          "submitted_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "When the review was submitted (storefront timestamp string).",
            "example": "2026-02-11"
          }
        }
      },
      "description": "Review rows on this page."
    }
  }
}
```

Status `400`: The query is malformed or too large. As `text/plain`, when the body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
Failed to deserialize the JSON body into the target type: missing field `asin` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

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
