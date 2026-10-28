# Jobs

Queued and running control-plane jobs.

Back to the [web API reference](README.md).

## `GET /api/jobs`

Returns durable scan/acquire/listen_sync jobs, newest first.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Jobs, newest first. Body: array of [`JobInfo`](schemas.md#jobinfo) (`application/json`)
  - `id` *string*, **required** — Opaque job id returned to the client when the job is accepted.
  - `kind` *[`JobKind`](schemas.md#jobkind)*, **required** — What the job does.
  - `status` *[`JobState`](schemas.md#jobstate)*, **required** — Where the job is in its lifecycle.
  - `detail` *string or null* — Optional operator-facing progress or error text.
  - `progress` *string or null* — Human-readable progress string while the job is running.
  - `attempt_count` *integer (int64)* — Number of times a worker has claimed this row.
  - `max_attempts` *integer (int64)* — Maximum claims before a failure is terminal.
  - `error_kind` *string or null* — Structured error kind when failed or cancelled.
  - `created_at` *string or null* — RFC 3339 timestamp when the row was inserted.
  - `finished_at` *string or null* — RFC 3339 timestamp when the row reached a terminal state.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The job table could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/jobs' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/jobs' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/jobs",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/jobs", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/jobs")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Jobs, newest first.

```json
[
  {
    "id": "string",
    "kind": "scan",
    "status": "pending",
    "detail": "string",
    "progress": "string",
    "attempt_count": 0,
    "max_attempts": 0,
    "error_kind": "string",
    "created_at": "string",
    "finished_at": "string"
  }
]
```

Schema:

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "description": "Snapshot of one queued or running control-plane job.",
    "required": [
      "id",
      "kind",
      "status"
    ],
    "properties": {
      "id": {
        "type": "string",
        "description": "Opaque job id returned to the client when the job is accepted."
      },
      "kind": {
        "type": "string",
        "description": "What the job does.",
        "enum": [
          "scan",
          "acquire",
          "listen_sync",
          "integration_scan",
          "plugin_copy",
          "invalid"
        ]
      },
      "status": {
        "type": "string",
        "description": "Where the job is in its lifecycle.",
        "enum": [
          "pending",
          "running",
          "succeeded",
          "failed",
          "cancelled"
        ]
      },
      "detail": {
        "type": [
          "string",
          "null"
        ],
        "description": "Optional operator-facing progress or error text."
      },
      "progress": {
        "type": [
          "string",
          "null"
        ],
        "description": "Human-readable progress string while the job is running."
      },
      "attempt_count": {
        "type": "integer",
        "format": "int64",
        "description": "Number of times a worker has claimed this row."
      },
      "max_attempts": {
        "type": "integer",
        "format": "int64",
        "description": "Maximum claims before a failure is terminal."
      },
      "error_kind": {
        "type": [
          "string",
          "null"
        ],
        "description": "Structured error kind when failed or cancelled."
      },
      "created_at": {
        "type": [
          "string",
          "null"
        ],
        "description": "RFC 3339 timestamp when the row was inserted."
      },
      "finished_at": {
        "type": [
          "string",
          "null"
        ],
        "description": "RFC 3339 timestamp when the row reached a terminal state."
      }
    }
  }
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

Status `403`: The caller's role does not allow this route.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `500`: The job table could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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
      "description": "Machine slug (same as [`ErrorBody::code`])."
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```

## `POST /api/jobs/{id}/cancel`

Cancel a pending job or request cooperative stop of a running job.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Job id.

### Responses

- `200` — The job was cancelled or asked to stop. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such job. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The job could not be updated. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/jobs/<id>/cancel' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/jobs/<id>/cancel' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/jobs/<id>/cancel",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/jobs/<id>/cancel", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/jobs/<id>/cancel")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The job was cancelled or asked to stop.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
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
      "description": "True when the action was accepted or completed."
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text."
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous."
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`)."
    }
  }
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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `404`: No such job.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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
      "description": "Machine slug (same as [`ErrorBody::code`])."
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "minimum": 0
    }
  }
}
```

Status `500`: The job could not be updated.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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
      "description": "Machine slug (same as [`ErrorBody::code`])."
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```
