# Configuration

Live configuration domains and reloads.

Back to the [web API reference](README.md).

## `POST /api/config/reload`

Reloads `config.toml` and republishes daemon runtime without a process restart.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — The configuration was reloaded. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
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
- `500` — The reload failed; the old configuration stays live. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/reload' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/reload' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/reload",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/reload", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/reload")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The configuration was reloaded.

```json
{
  "job_id": "",
  "message": "reloaded /home/bookclerk/BookclerkFiles/config.toml (master.key wrap=true)",
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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `500`: The reload failed; the old configuration stays live.

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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/domains/core.events`

`GET /api/config/domains/core.events`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — The `core.events` settings. Body: [`EventsDomainResponse`](schemas.md#eventsdomainresponse) (`application/json`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/domains/core.events' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/domains/core.events' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/domains/core.events",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/domains/core.events", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/domains/core.events")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The `core.events` settings.

```json
{
  "namespace": "core.events",
  "scope_type": "library",
  "scope_id": "singleton",
  "schema_version": 1,
  "revision": 12,
  "updated_at": "2026-10-28T19:07:00Z",
  "updated_by": "operator",
  "retention_days": 30,
  "dead_letter_retention_days": 90,
  "concurrency": 4
}
```

Schema:

```json
{
  "allOf": [
    {
      "type": "object",
      "description": "Where the document lives and which revision it is at.",
      "required": [
        "namespace",
        "scope_type",
        "scope_id",
        "schema_version",
        "revision",
        "updated_at",
        "updated_by"
      ],
      "properties": {
        "namespace": {
          "type": "string",
          "description": "Domain namespace (`core.events` or `core.output`).",
          "example": "core.events"
        },
        "scope_type": {
          "type": "string",
          "description": "Scope type: `library` for a shared document, `node` for one node's.",
          "example": "library"
        },
        "scope_id": {
          "type": "string",
          "description": "Scope id: `singleton` for the library, else the node id.",
          "example": "singleton"
        },
        "schema_version": {
          "type": "integer",
          "format": "int64",
          "description": "Version of the document's schema, starting at 1.",
          "example": 1
        },
        "revision": {
          "type": "integer",
          "format": "int64",
          "description": "Compare-and-swap revision; send it back as `expected_revision`.",
          "example": 12
        },
        "updated_at": {
          "type": "string",
          "description": "RFC 3339 time of the last write.",
          "example": "2026-10-28T19:07:00Z"
        },
        "updated_by": {
          "type": "string",
          "description": "Audit identity of the last writer.",
          "example": "operator"
        }
      }
    },
    {
      "type": "object",
      "description": "The settings at that revision.",
      "required": [
        "retention_days",
        "dead_letter_retention_days",
        "concurrency"
      ],
      "properties": {
        "retention_days": {
          "type": "integer",
          "format": "int64",
          "description": "Days to keep acked and rejected deliveries.",
          "example": 30,
          "minimum": 0
        },
        "dead_letter_retention_days": {
          "type": "integer",
          "format": "int64",
          "description": "Days to keep dead-lettered deliveries.",
          "example": 90,
          "minimum": 0
        },
        "concurrency": {
          "type": "integer",
          "format": "int32",
          "description": "Local delivery workers and the library in-flight cap.",
          "example": 4,
          "minimum": 0
        }
      }
    }
  ],
  "description": "Body of `GET /api/config/domains/core.events` and of a `PUT` that was\napplied: the revision metadata, then the `[events]` settings themselves."
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

Status `403`: The caller's role does not allow this route.

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

Status `500`: The configuration could not be read.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/config/domains/core.events`

`PUT /api/config/domains/core.events`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PutEventsDomainRequest`](schemas.md#puteventsdomainrequest)

- `expected_revision` *integer (int64)*, **required** — Revision the caller read. A mismatch is a conflict.
- `retention_days` *integer (int64)*, **required** — Replacement `[events]` document. At least 0.
- `dead_letter_retention_days` *integer (int64)*, **required** — Replacement dead-letter retention. At least 0.
- `concurrency` *integer (int32)*, **required** — Replacement concurrency cap. At least 0.

### Responses

- `200` — The settings after the change, or a `replayed: true` note with the committed revision when another write landed between this one and its read-back. Body: [`EventsDomainChange`](schemas.md#eventsdomainchange) (`application/json`)
- `400` — A value is outside its range. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — `expected_revision` is stale; `current_revision` is the revision to retry against. Body: [`RevisionConflict`](schemas.md#revisionconflict) (`application/json`)
  - `error` *string*, **required** — Always `revision_conflict`.
  - `current_revision` *integer (int64)*, **required** — The revision to read and retry against.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/domains/core.events' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"expected_revision": 12, "retention_days": 30, "dead_letter_retention_days": 90, "concurrency": 4}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/domains/core.events' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"expected_revision": 12, "retention_days": 30, "dead_letter_retention_days": 90, "concurrency": 4}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/domains/core.events",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"expected_revision": 12, "retention_days": 30, "dead_letter_retention_days": 90, "concurrency": 4},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/domains/core.events", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"expected_revision": 12, "retention_days": 30, "dead_letter_retention_days": 90, "concurrency": 4}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/domains/core.events")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"expected_revision": 12, "retention_days": 30, "dead_letter_retention_days": 90, "concurrency": 4}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings after the change, or a `replayed: true` note with the committed revision when another write landed between this one and its read-back.

```json
{
  "namespace": "core.events",
  "scope_type": "library",
  "scope_id": "singleton",
  "schema_version": 1,
  "revision": 12,
  "updated_at": "2026-10-28T19:07:00Z",
  "updated_by": "operator",
  "retention_days": 30,
  "dead_letter_retention_days": 90,
  "concurrency": 4
}
```

Schema:

```json
{
  "oneOf": [
    {
      "allOf": [
        {
          "type": "object",
          "description": "Where the document lives and which revision it is at.",
          "required": [
            "namespace",
            "scope_type",
            "scope_id",
            "schema_version",
            "revision",
            "updated_at",
            "updated_by"
          ],
          "properties": {
            "namespace": {
              "type": "string",
              "description": "Domain namespace (`core.events` or `core.output`).",
              "example": "core.events"
            },
            "scope_type": {
              "type": "string",
              "description": "Scope type: `library` for a shared document, `node` for one node's.",
              "example": "library"
            },
            "scope_id": {
              "type": "string",
              "description": "Scope id: `singleton` for the library, else the node id.",
              "example": "singleton"
            },
            "schema_version": {
              "type": "integer",
              "format": "int64",
              "description": "Version of the document's schema, starting at 1.",
              "example": 1
            },
            "revision": {
              "type": "integer",
              "format": "int64",
              "description": "Compare-and-swap revision; send it back as `expected_revision`.",
              "example": 12
            },
            "updated_at": {
              "type": "string",
              "description": "RFC 3339 time of the last write.",
              "example": "2026-10-28T19:07:00Z"
            },
            "updated_by": {
              "type": "string",
              "description": "Audit identity of the last writer.",
              "example": "operator"
            }
          }
        },
        {
          "type": "object",
          "description": "The settings at that revision.",
          "required": [
            "retention_days",
            "dead_letter_retention_days",
            "concurrency"
          ],
          "properties": {
            "retention_days": {
              "type": "integer",
              "format": "int64",
              "description": "Days to keep acked and rejected deliveries.",
              "example": 30,
              "minimum": 0
            },
            "dead_letter_retention_days": {
              "type": "integer",
              "format": "int64",
              "description": "Days to keep dead-lettered deliveries.",
              "example": 90,
              "minimum": 0
            },
            "concurrency": {
              "type": "integer",
              "format": "int32",
              "description": "Local delivery workers and the library in-flight cap.",
              "example": 4,
              "minimum": 0
            }
          }
        }
      ],
      "description": "The change was written; these are the settings now."
    },
    {
      "type": "object",
      "description": "The write committed; another write landed before the read-back.",
      "required": [
        "namespace",
        "replayed",
        "revision"
      ],
      "properties": {
        "namespace": {
          "type": "string",
          "description": "Domain namespace (`core.events` or `core.output`).",
          "example": "core.events"
        },
        "replayed": {
          "type": "boolean",
          "description": "Always `true`: the write was a replay.",
          "example": true
        },
        "revision": {
          "type": "integer",
          "format": "int64",
          "description": "Revision the earlier write produced.",
          "example": 12
        }
      }
    }
  ],
  "description": "Body of `PUT /api/config/domains/core.events`: the settings after the\nchange, or a replay note."
}
```

Status `400`: A value is outside its range. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid configuration: retention_days must be 1..=3650"
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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `409`: `expected_revision` is stale; `current_revision` is the revision to retry against.

```json
{
  "current_revision": 7,
  "error": "revision_conflict"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of a `409 Conflict` from a compare-and-swap write whose\n`expected_revision` lost to a newer revision.",
  "required": [
    "error",
    "current_revision"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Always `revision_conflict`.",
      "example": "revision_conflict"
    },
    "current_revision": {
      "type": "integer",
      "format": "int64",
      "description": "The revision to read and retry against.",
      "example": 13
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
Failed to deserialize the JSON body into the target type: missing field `expected_revision` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The configuration could not be read or written.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/domains/core.output`

`GET /api/config/domains/core.output`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — The `core.output` settings. Body: [`OutputDomainResponse`](schemas.md#outputdomainresponse) (`application/json`)
  - `document` *[`OutputSettingsV1`](schemas.md#outputsettingsv1)*, **required** — The settings at that revision (schema version 1).
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/domains/core.output' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/domains/core.output' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/domains/core.output",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/domains/core.output", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/domains/core.output")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The `core.output` settings.

```json
{
  "namespace": "core.events",
  "scope_type": "library",
  "scope_id": "singleton",
  "schema_version": 1,
  "revision": 12,
  "updated_at": "2026-10-28T19:07:00Z",
  "updated_by": "operator",
  "document": {
    "format": "enriched_m4b",
    "copies": [
      {
        "format": "mp3",
        "quality": "64k-mono"
      }
    ],
    "widevine": true,
    "xhe_aac": false,
    "widevine_cdm": "cdm/device.wvd",
    "widevine_cdm_provider": null,
    "naming_profile": "audiobookshelf",
    "folder_template": "<author>/<title>",
    "file_template": "<title> [<id>]",
    "chapter_file_template": "<title> - <ch#> - <chapter title>",
    "chapter_title_template": "<ch#> - <chapter title>",
    "download_cover": true,
    "download_pdf": true,
    "create_cue": false,
    "keep_codec_in_m4b": false,
    "fixup_metadata": true,
    "chapter_json": "off",
    "save_metadata_json": false,
    "cover_size": "1215",
    "chapter_layout": "tree",
    "overwrite_existing": false,
    "multi_destination": "sync_missing",
    "missing_files": "mark_missing",
    "bad_book_action": "ask",
    "split_mp3_max_mb": 100,
    "minimum_file_duration_minutes": 1,
    "combine_nested_chapter_titles": false,
    "merge_opening_and_end_credits": true,
    "strip_unabridged": true,
    "strip_audible_brand_audio": false,
    "download_clips_bookmarks": false,
    "download_speed_limit_kbps": 0,
    "lame": {
      "target": "quality",
      "vbr_quality": 2,
      "bitrate_kbps": 128,
      "mode": "default",
      "downsample_mono": false,
      "constant_bitrate": false
    },
    "max_sample_rate": 44100,
    "creation_time": "now",
    "last_write_time": "now",
    "path_sanitization": "auto",
    "replacement_characters": [
      {
        "find": ":",
        "replace": " -"
      }
    ],
    "max_filename_length": 255
  }
}
```

Schema:

```json
{
  "allOf": [
    {
      "type": "object",
      "description": "Where the document lives and which revision it is at.",
      "required": [
        "namespace",
        "scope_type",
        "scope_id",
        "schema_version",
        "revision",
        "updated_at",
        "updated_by"
      ],
      "properties": {
        "namespace": {
          "type": "string",
          "description": "Domain namespace (`core.events` or `core.output`).",
          "example": "core.events"
        },
        "scope_type": {
          "type": "string",
          "description": "Scope type: `library` for a shared document, `node` for one node's.",
          "example": "library"
        },
        "scope_id": {
          "type": "string",
          "description": "Scope id: `singleton` for the library, else the node id.",
          "example": "singleton"
        },
        "schema_version": {
          "type": "integer",
          "format": "int64",
          "description": "Version of the document's schema, starting at 1.",
          "example": 1
        },
        "revision": {
          "type": "integer",
          "format": "int64",
          "description": "Compare-and-swap revision; send it back as `expected_revision`.",
          "example": 12
        },
        "updated_at": {
          "type": "string",
          "description": "RFC 3339 time of the last write.",
          "example": "2026-10-28T19:07:00Z"
        },
        "updated_by": {
          "type": "string",
          "description": "Audit identity of the last writer.",
          "example": "operator"
        }
      }
    },
    {
      "type": "object",
      "required": [
        "document"
      ],
      "properties": {
        "document": {
          "type": "object",
          "description": "The settings at that revision (schema version 1).",
          "required": [
            "format",
            "widevine",
            "xhe_aac",
            "naming_profile",
            "download_cover",
            "download_pdf",
            "create_cue",
            "fixup_metadata",
            "chapter_json",
            "save_metadata_json",
            "cover_size",
            "chapter_layout",
            "overwrite_existing",
            "multi_destination",
            "bad_book_action",
            "split_mp3_max_mb",
            "minimum_file_duration_minutes",
            "combine_nested_chapter_titles",
            "merge_opening_and_end_credits",
            "strip_unabridged",
            "strip_audible_brand_audio",
            "download_clips_bookmarks",
            "download_speed_limit_kbps",
            "lame",
            "creation_time",
            "last_write_time",
            "path_sanitization",
            "replacement_characters",
            "max_filename_length"
          ],
          "properties": {
            "format": {
              "type": "string",
              "description": "Post-acquire packaging format.",
              "enum": [
                "enriched_m4b",
                "single_mp3",
                "split_mp3_by_chapter",
                "split_mp3_by_size",
                "opus",
                "none"
              ],
              "example": "enriched_m4b"
            },
            "copies": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "One copy of every acquired book: a format at a quality.",
                "required": [
                  "format",
                  "quality"
                ],
                "properties": {
                  "format": {
                    "type": "string",
                    "description": "The copy's format.",
                    "enum": [
                      "m4b",
                      "mp3",
                      "mp3_chapters",
                      "original",
                      "opus"
                    ]
                  },
                  "quality": {
                    "type": "string",
                    "description": "The copy's quality.",
                    "examples": [
                      "64k-mono"
                    ],
                    "pattern": "^(original|((16|24|32|48|64|96|128|160|192|256|320)k|v[0-9])(-mono)?)$"
                  }
                },
                "additionalProperties": false
              },
              "description": "Further copies of every acquired book beside the main output, at\nmost 8; empty keeps one copy. Each needs a quality that suits its\nformat, and none may repeat another or the main output's copy.",
              "example": [
                {
                  "format": "mp3",
                  "quality": "64k-mono"
                }
              ],
              "maxItems": 8
            },
            "widevine": {
              "type": "boolean",
              "description": "Prefer Widevine/CENC.",
              "example": true
            },
            "xhe_aac": {
              "type": "boolean",
              "description": "Prefer xHE-AAC on the Widevine path when offered.",
              "example": false
            },
            "widevine_cdm": {
              "type": [
                "string",
                "null"
              ],
              "description": "Optional `.wvd` path; relative paths resolve under each host's files dir.",
              "example": "cdm/device.wvd"
            },
            "widevine_cdm_provider": {
              "type": [
                "string",
                "null"
              ],
              "description": "Remote L3 CDM provider endpoint URL (`None` = the built-in AudibleCdm\nprovider, empty/`off` disables provisioning).",
              "example": null
            },
            "naming_profile": {
              "type": "string",
              "description": "Named path-template preset.",
              "enum": [
                "audiobookshelf",
                "classic"
              ],
              "example": "audiobookshelf"
            },
            "folder_template": {
              "type": [
                "string",
                "null"
              ],
              "description": "Folder-path template override.",
              "example": "<author>/<title>"
            },
            "file_template": {
              "type": [
                "string",
                "null"
              ],
              "description": "File-stem template override.",
              "example": "<title> [<id>]"
            },
            "chapter_file_template": {
              "type": [
                "string",
                "null"
              ],
              "description": "Per-chapter file-stem template override.",
              "example": "<title> - <ch#> - <chapter title>"
            },
            "chapter_title_template": {
              "type": [
                "string",
                "null"
              ],
              "description": "Embedded chapter-title template.",
              "example": "<ch#> - <chapter title>"
            },
            "download_cover": {
              "type": "boolean",
              "description": "Save cover JPEG alongside audio.",
              "example": true
            },
            "download_pdf": {
              "type": "boolean",
              "description": "Download companion PDF when available.",
              "example": true
            },
            "create_cue": {
              "type": "boolean",
              "description": "Write a `.cue` sidecar.",
              "example": false
            },
            "keep_codec_in_m4b": {
              "type": "boolean",
              "description": "Keep Opus, FLAC, ALAC, E-AC-3 and AC-4 audio unchanged inside an M4B\ninstead of encoding it to AAC (advanced; off by default).",
              "example": false
            },
            "fixup_metadata": {
              "type": "boolean",
              "description": "Embed tags, cover, and chapters natively.",
              "example": true
            },
            "chapter_json": {
              "type": "string",
              "description": "Chapter JSON sidecars.",
              "enum": [
                "off",
                "flat",
                "tree",
                "both"
              ],
              "example": "off"
            },
            "save_metadata_json": {
              "type": "boolean",
              "description": "Persist raw catalog API JSON.",
              "example": false
            },
            "cover_size": {
              "type": "string",
              "description": "Cover image size (`500`, `1215`, another pixel size, or `native`).",
              "example": "1215"
            },
            "chapter_layout": {
              "type": "string",
              "description": "Preferred chapter API layout (`tree`, `flat`, or `both`).",
              "example": "tree"
            },
            "overwrite_existing": {
              "type": "boolean",
              "description": "Re-acquire when media already exists at the destination.",
              "example": false
            },
            "multi_destination": {
              "type": "string",
              "description": "Policy when only some enabled destinations have a title.",
              "enum": [
                "sync_missing",
                "refetch_missing",
                "refetch_all"
              ],
              "example": "sync_missing"
            },
            "missing_files": {
              "type": "string",
              "description": "What happens to a stored copy whose file disappears from its\ndestination outside Bookclerk. Absent in documents written before\nthe setting existed, which read as the default.",
              "enum": [
                "mark_missing",
                "restore"
              ],
              "example": "mark_missing"
            },
            "bad_book_action": {
              "type": "string",
              "description": "Operator action when a title fails to acquire.",
              "enum": [
                "ask",
                "abort",
                "retry",
                "ignore"
              ],
              "example": "ask"
            },
            "split_mp3_max_mb": {
              "type": "integer",
              "format": "int32",
              "description": "Max MP3 part size in MiB for `split_mp3_by_size`.",
              "example": 100,
              "minimum": 0
            },
            "minimum_file_duration_minutes": {
              "type": "integer",
              "format": "int32",
              "description": "Drop chapter splits shorter than this many minutes.",
              "example": 1,
              "minimum": 0
            },
            "combine_nested_chapter_titles": {
              "type": "boolean",
              "description": "Flatten nested chapter titles.",
              "example": false
            },
            "merge_opening_and_end_credits": {
              "type": "boolean",
              "description": "Merge opening/end credit chapters into neighbours.",
              "example": true
            },
            "strip_unabridged": {
              "type": "boolean",
              "description": "Strip an \"Unabridged\" suffix from titles used in naming.",
              "example": true
            },
            "strip_audible_brand_audio": {
              "type": "boolean",
              "description": "Trim Audible brand intro/outro audio.",
              "example": false
            },
            "download_clips_bookmarks": {
              "type": "boolean",
              "description": "Download Audible clips/bookmarks sidecars.",
              "example": false
            },
            "download_speed_limit_kbps": {
              "type": "integer",
              "format": "int32",
              "description": "Fetch speed cap in KB/s (`0` = unlimited).",
              "example": 0,
              "minimum": 0
            },
            "lame": {
              "type": "object",
              "description": "LAME encoder knobs.",
              "properties": {
                "target": {
                  "type": "string",
                  "description": "`LameTarget`: `quality`, `bitrate`, `constant`, etc.",
                  "default": "quality",
                  "example": "quality"
                },
                "vbr_quality": {
                  "type": "integer",
                  "format": "int32",
                  "description": "`LameVBRQuality` (0–9, lower = better).",
                  "default": 2,
                  "example": 2,
                  "minimum": 0
                },
                "bitrate_kbps": {
                  "type": "integer",
                  "format": "int32",
                  "description": "`LameBitrate` in kbps when target is bitrate/constant.",
                  "default": 128,
                  "example": 128,
                  "minimum": 0
                },
                "mode": {
                  "type": "string",
                  "description": "`LameMode`: `default`, `mono`, `stereo`.",
                  "default": "default",
                  "example": "default"
                },
                "downsample_mono": {
                  "type": "boolean",
                  "description": "`LameDownsampleMono`.",
                  "default": false,
                  "example": false
                },
                "constant_bitrate": {
                  "type": "boolean",
                  "description": "`LameConstantBitrate`.",
                  "default": false,
                  "example": false
                }
              },
              "additionalProperties": false
            },
            "max_sample_rate": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional output sample-rate ceiling in Hz.",
              "example": 44100,
              "minimum": 0
            },
            "creation_time": {
              "type": "string",
              "description": "File creation timestamp policy.",
              "enum": [
                "now",
                "purchased",
                "published"
              ],
              "example": "now"
            },
            "last_write_time": {
              "type": "string",
              "description": "File last-write timestamp policy.",
              "enum": [
                "now",
                "purchased",
                "published"
              ],
              "example": "now"
            },
            "path_sanitization": {
              "type": "string",
              "description": "Sanitization profile used when no explicit rules are set.",
              "enum": [
                "auto",
                "windows",
                "posix",
                "s3",
                "none"
              ],
              "example": "auto"
            },
            "replacement_characters": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Character replacement rule (`ReplacementCharacters` in classic Settings.json).",
                "required": [
                  "find",
                  "replace"
                ],
                "properties": {
                  "find": {
                    "type": "string",
                    "description": "Substring to search for in each path segment / tag value.",
                    "example": ":"
                  },
                  "replace": {
                    "type": "string",
                    "description": "Replacement text (may be empty to strip `find`).",
                    "example": " -"
                  }
                },
                "additionalProperties": false
              },
              "description": "Explicit find/replace rules for path segments.",
              "example": [
                {
                  "find": ":",
                  "replace": " -"
                }
              ]
            },
            "max_filename_length": {
              "type": "integer",
              "format": "int32",
              "description": "Max length per path segment (`0` disables truncation).",
              "example": 255,
              "minimum": 0
            }
          },
          "additionalProperties": false
        }
      }
    }
  ],
  "description": "Body of `GET /api/config/domains/core.output` and of a `PUT` that was\napplied: the revision metadata and the `[output]` document."
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

Status `403`: The caller's role does not allow this route.

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

Status `500`: The configuration could not be read.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/config/domains/core.output`

`PUT /api/config/domains/core.output`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PutOutputDomainRequest`](schemas.md#putoutputdomainrequest)

- `expected_revision` *integer (int64)*, **required** — Revision the caller read. A mismatch is a conflict.
- `document` *[`OutputSettingsV1`](schemas.md#outputsettingsv1)*, **required** — Complete replacement document (schema version 1).

### Responses

- `200` — The settings after the change, or a `replayed: true` note with the committed revision when another write landed between this one and its read-back. Body: [`OutputDomainChange`](schemas.md#outputdomainchange) (`application/json`)
- `400` — The body is invalid, or a key or value is not an output setting. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — `expected_revision` is stale; `current_revision` is the revision to retry against. Body: [`RevisionConflict`](schemas.md#revisionconflict) (`application/json`)
  - `error` *string*, **required** — Always `revision_conflict`.
  - `current_revision` *integer (int64)*, **required** — The revision to read and retry against.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/domains/core.output' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"expected_revision": 12, "document": {"format": "enriched_m4b", "widevine": true, "xhe_aac": false, "naming_profile": "audiobookshelf", "download_cover": true, "download_pdf": true, "create_cue": false, "fixup_metadata": true, "chapter_json": "off", "save_metadata_json": false, "cover_size": "1215", "chapter_layout": "tree", "overwrite_existing": false, "multi_destination": "sync_missing", "bad_book_action": "ask", "split_mp3_max_mb": 100, "minimum_file_duration_minutes": 1, "combine_nested_chapter_titles": false, "merge_opening_and_end_credits": true, "strip_unabridged": true, "strip_audible_brand_audio": false, "download_clips_bookmarks": false, "download_speed_limit_kbps": 0, "lame": {}, "creation_time": "now", "last_write_time": "now", "path_sanitization": "auto", "replacement_characters": [{"find": ":", "replace": " -"}], "max_filename_length": 255}}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/domains/core.output' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"expected_revision": 12, "document": {"format": "enriched_m4b", "widevine": true, "xhe_aac": false, "naming_profile": "audiobookshelf", "download_cover": true, "download_pdf": true, "create_cue": false, "fixup_metadata": true, "chapter_json": "off", "save_metadata_json": false, "cover_size": "1215", "chapter_layout": "tree", "overwrite_existing": false, "multi_destination": "sync_missing", "bad_book_action": "ask", "split_mp3_max_mb": 100, "minimum_file_duration_minutes": 1, "combine_nested_chapter_titles": false, "merge_opening_and_end_credits": true, "strip_unabridged": true, "strip_audible_brand_audio": false, "download_clips_bookmarks": false, "download_speed_limit_kbps": 0, "lame": {}, "creation_time": "now", "last_write_time": "now", "path_sanitization": "auto", "replacement_characters": [{"find": ":", "replace": " -"}], "max_filename_length": 255}}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/domains/core.output",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"expected_revision": 12, "document": {"format": "enriched_m4b", "widevine": True, "xhe_aac": False, "naming_profile": "audiobookshelf", "download_cover": True, "download_pdf": True, "create_cue": False, "fixup_metadata": True, "chapter_json": "off", "save_metadata_json": False, "cover_size": "1215", "chapter_layout": "tree", "overwrite_existing": False, "multi_destination": "sync_missing", "bad_book_action": "ask", "split_mp3_max_mb": 100, "minimum_file_duration_minutes": 1, "combine_nested_chapter_titles": False, "merge_opening_and_end_credits": True, "strip_unabridged": True, "strip_audible_brand_audio": False, "download_clips_bookmarks": False, "download_speed_limit_kbps": 0, "lame": {}, "creation_time": "now", "last_write_time": "now", "path_sanitization": "auto", "replacement_characters": [{"find": ":", "replace": " -"}], "max_filename_length": 255}},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/domains/core.output", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"expected_revision": 12, "document": {"format": "enriched_m4b", "widevine": true, "xhe_aac": false, "naming_profile": "audiobookshelf", "download_cover": true, "download_pdf": true, "create_cue": false, "fixup_metadata": true, "chapter_json": "off", "save_metadata_json": false, "cover_size": "1215", "chapter_layout": "tree", "overwrite_existing": false, "multi_destination": "sync_missing", "bad_book_action": "ask", "split_mp3_max_mb": 100, "minimum_file_duration_minutes": 1, "combine_nested_chapter_titles": false, "merge_opening_and_end_credits": true, "strip_unabridged": true, "strip_audible_brand_audio": false, "download_clips_bookmarks": false, "download_speed_limit_kbps": 0, "lame": {}, "creation_time": "now", "last_write_time": "now", "path_sanitization": "auto", "replacement_characters": [{"find": ":", "replace": " -"}], "max_filename_length": 255}}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/domains/core.output")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"expected_revision": 12, "document": {"format": "enriched_m4b", "widevine": true, "xhe_aac": false, "naming_profile": "audiobookshelf", "download_cover": true, "download_pdf": true, "create_cue": false, "fixup_metadata": true, "chapter_json": "off", "save_metadata_json": false, "cover_size": "1215", "chapter_layout": "tree", "overwrite_existing": false, "multi_destination": "sync_missing", "bad_book_action": "ask", "split_mp3_max_mb": 100, "minimum_file_duration_minutes": 1, "combine_nested_chapter_titles": false, "merge_opening_and_end_credits": true, "strip_unabridged": true, "strip_audible_brand_audio": false, "download_clips_bookmarks": false, "download_speed_limit_kbps": 0, "lame": {}, "creation_time": "now", "last_write_time": "now", "path_sanitization": "auto", "replacement_characters": [{"find": ":", "replace": " -"}], "max_filename_length": 255}}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings after the change, or a `replayed: true` note with the committed revision when another write landed between this one and its read-back.

```json
{
  "namespace": "core.events",
  "scope_type": "library",
  "scope_id": "singleton",
  "schema_version": 1,
  "revision": 12,
  "updated_at": "2026-10-28T19:07:00Z",
  "updated_by": "operator",
  "document": {
    "format": "enriched_m4b",
    "copies": [
      {
        "format": "mp3",
        "quality": "64k-mono"
      }
    ],
    "widevine": true,
    "xhe_aac": false,
    "widevine_cdm": "cdm/device.wvd",
    "widevine_cdm_provider": null,
    "naming_profile": "audiobookshelf",
    "folder_template": "<author>/<title>",
    "file_template": "<title> [<id>]",
    "chapter_file_template": "<title> - <ch#> - <chapter title>",
    "chapter_title_template": "<ch#> - <chapter title>",
    "download_cover": true,
    "download_pdf": true,
    "create_cue": false,
    "keep_codec_in_m4b": false,
    "fixup_metadata": true,
    "chapter_json": "off",
    "save_metadata_json": false,
    "cover_size": "1215",
    "chapter_layout": "tree",
    "overwrite_existing": false,
    "multi_destination": "sync_missing",
    "missing_files": "mark_missing",
    "bad_book_action": "ask",
    "split_mp3_max_mb": 100,
    "minimum_file_duration_minutes": 1,
    "combine_nested_chapter_titles": false,
    "merge_opening_and_end_credits": true,
    "strip_unabridged": true,
    "strip_audible_brand_audio": false,
    "download_clips_bookmarks": false,
    "download_speed_limit_kbps": 0,
    "lame": {
      "target": "quality",
      "vbr_quality": 2,
      "bitrate_kbps": 128,
      "mode": "default",
      "downsample_mono": false,
      "constant_bitrate": false
    },
    "max_sample_rate": 44100,
    "creation_time": "now",
    "last_write_time": "now",
    "path_sanitization": "auto",
    "replacement_characters": [
      {
        "find": ":",
        "replace": " -"
      }
    ],
    "max_filename_length": 255
  }
}
```

Schema:

```json
{
  "oneOf": [
    {
      "allOf": [
        {
          "type": "object",
          "description": "Where the document lives and which revision it is at.",
          "required": [
            "namespace",
            "scope_type",
            "scope_id",
            "schema_version",
            "revision",
            "updated_at",
            "updated_by"
          ],
          "properties": {
            "namespace": {
              "type": "string",
              "description": "Domain namespace (`core.events` or `core.output`).",
              "example": "core.events"
            },
            "scope_type": {
              "type": "string",
              "description": "Scope type: `library` for a shared document, `node` for one node's.",
              "example": "library"
            },
            "scope_id": {
              "type": "string",
              "description": "Scope id: `singleton` for the library, else the node id.",
              "example": "singleton"
            },
            "schema_version": {
              "type": "integer",
              "format": "int64",
              "description": "Version of the document's schema, starting at 1.",
              "example": 1
            },
            "revision": {
              "type": "integer",
              "format": "int64",
              "description": "Compare-and-swap revision; send it back as `expected_revision`.",
              "example": 12
            },
            "updated_at": {
              "type": "string",
              "description": "RFC 3339 time of the last write.",
              "example": "2026-10-28T19:07:00Z"
            },
            "updated_by": {
              "type": "string",
              "description": "Audit identity of the last writer.",
              "example": "operator"
            }
          }
        },
        {
          "type": "object",
          "required": [
            "document"
          ],
          "properties": {
            "document": {
              "type": "object",
              "description": "The settings at that revision (schema version 1).",
              "required": [
                "format",
                "widevine",
                "xhe_aac",
                "naming_profile",
                "download_cover",
                "download_pdf",
                "create_cue",
                "fixup_metadata",
                "chapter_json",
                "save_metadata_json",
                "cover_size",
                "chapter_layout",
                "overwrite_existing",
                "multi_destination",
                "bad_book_action",
                "split_mp3_max_mb",
                "minimum_file_duration_minutes",
                "combine_nested_chapter_titles",
                "merge_opening_and_end_credits",
                "strip_unabridged",
                "strip_audible_brand_audio",
                "download_clips_bookmarks",
                "download_speed_limit_kbps",
                "lame",
                "creation_time",
                "last_write_time",
                "path_sanitization",
                "replacement_characters",
                "max_filename_length"
              ],
              "properties": {
                "format": {
                  "type": "string",
                  "description": "Post-acquire packaging format.",
                  "enum": [
                    "enriched_m4b",
                    "single_mp3",
                    "split_mp3_by_chapter",
                    "split_mp3_by_size",
                    "opus",
                    "none"
                  ],
                  "example": "enriched_m4b"
                },
                "copies": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "One copy of every acquired book: a format at a quality.",
                    "required": [
                      "format",
                      "quality"
                    ],
                    "properties": {
                      "format": {
                        "type": "string",
                        "description": "The copy's format.",
                        "enum": [
                          "m4b",
                          "mp3",
                          "mp3_chapters",
                          "original",
                          "opus"
                        ]
                      },
                      "quality": {
                        "type": "string",
                        "description": "The copy's quality.",
                        "examples": [
                          "64k-mono"
                        ],
                        "pattern": "^(original|((16|24|32|48|64|96|128|160|192|256|320)k|v[0-9])(-mono)?)$"
                      }
                    },
                    "additionalProperties": false
                  },
                  "description": "Further copies of every acquired book beside the main output, at\nmost 8; empty keeps one copy. Each needs a quality that suits its\nformat, and none may repeat another or the main output's copy.",
                  "example": [
                    {
                      "format": "mp3",
                      "quality": "64k-mono"
                    }
                  ],
                  "maxItems": 8
                },
                "widevine": {
                  "type": "boolean",
                  "description": "Prefer Widevine/CENC.",
                  "example": true
                },
                "xhe_aac": {
                  "type": "boolean",
                  "description": "Prefer xHE-AAC on the Widevine path when offered.",
                  "example": false
                },
                "widevine_cdm": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Optional `.wvd` path; relative paths resolve under each host's files dir.",
                  "example": "cdm/device.wvd"
                },
                "widevine_cdm_provider": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Remote L3 CDM provider endpoint URL (`None` = the built-in AudibleCdm\nprovider, empty/`off` disables provisioning).",
                  "example": null
                },
                "naming_profile": {
                  "type": "string",
                  "description": "Named path-template preset.",
                  "enum": [
                    "audiobookshelf",
                    "classic"
                  ],
                  "example": "audiobookshelf"
                },
                "folder_template": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Folder-path template override.",
                  "example": "<author>/<title>"
                },
                "file_template": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "File-stem template override.",
                  "example": "<title> [<id>]"
                },
                "chapter_file_template": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Per-chapter file-stem template override.",
                  "example": "<title> - <ch#> - <chapter title>"
                },
                "chapter_title_template": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Embedded chapter-title template.",
                  "example": "<ch#> - <chapter title>"
                },
                "download_cover": {
                  "type": "boolean",
                  "description": "Save cover JPEG alongside audio.",
                  "example": true
                },
                "download_pdf": {
                  "type": "boolean",
                  "description": "Download companion PDF when available.",
                  "example": true
                },
                "create_cue": {
                  "type": "boolean",
                  "description": "Write a `.cue` sidecar.",
                  "example": false
                },
                "keep_codec_in_m4b": {
                  "type": "boolean",
                  "description": "Keep Opus, FLAC, ALAC, E-AC-3 and AC-4 audio unchanged inside an M4B\ninstead of encoding it to AAC (advanced; off by default).",
                  "example": false
                },
                "fixup_metadata": {
                  "type": "boolean",
                  "description": "Embed tags, cover, and chapters natively.",
                  "example": true
                },
                "chapter_json": {
                  "type": "string",
                  "description": "Chapter JSON sidecars.",
                  "enum": [
                    "off",
                    "flat",
                    "tree",
                    "both"
                  ],
                  "example": "off"
                },
                "save_metadata_json": {
                  "type": "boolean",
                  "description": "Persist raw catalog API JSON.",
                  "example": false
                },
                "cover_size": {
                  "type": "string",
                  "description": "Cover image size (`500`, `1215`, another pixel size, or `native`).",
                  "example": "1215"
                },
                "chapter_layout": {
                  "type": "string",
                  "description": "Preferred chapter API layout (`tree`, `flat`, or `both`).",
                  "example": "tree"
                },
                "overwrite_existing": {
                  "type": "boolean",
                  "description": "Re-acquire when media already exists at the destination.",
                  "example": false
                },
                "multi_destination": {
                  "type": "string",
                  "description": "Policy when only some enabled destinations have a title.",
                  "enum": [
                    "sync_missing",
                    "refetch_missing",
                    "refetch_all"
                  ],
                  "example": "sync_missing"
                },
                "missing_files": {
                  "type": "string",
                  "description": "What happens to a stored copy whose file disappears from its\ndestination outside Bookclerk. Absent in documents written before\nthe setting existed, which read as the default.",
                  "enum": [
                    "mark_missing",
                    "restore"
                  ],
                  "example": "mark_missing"
                },
                "bad_book_action": {
                  "type": "string",
                  "description": "Operator action when a title fails to acquire.",
                  "enum": [
                    "ask",
                    "abort",
                    "retry",
                    "ignore"
                  ],
                  "example": "ask"
                },
                "split_mp3_max_mb": {
                  "type": "integer",
                  "format": "int32",
                  "description": "Max MP3 part size in MiB for `split_mp3_by_size`.",
                  "example": 100,
                  "minimum": 0
                },
                "minimum_file_duration_minutes": {
                  "type": "integer",
                  "format": "int32",
                  "description": "Drop chapter splits shorter than this many minutes.",
                  "example": 1,
                  "minimum": 0
                },
                "combine_nested_chapter_titles": {
                  "type": "boolean",
                  "description": "Flatten nested chapter titles.",
                  "example": false
                },
                "merge_opening_and_end_credits": {
                  "type": "boolean",
                  "description": "Merge opening/end credit chapters into neighbours.",
                  "example": true
                },
                "strip_unabridged": {
                  "type": "boolean",
                  "description": "Strip an \"Unabridged\" suffix from titles used in naming.",
                  "example": true
                },
                "strip_audible_brand_audio": {
                  "type": "boolean",
                  "description": "Trim Audible brand intro/outro audio.",
                  "example": false
                },
                "download_clips_bookmarks": {
                  "type": "boolean",
                  "description": "Download Audible clips/bookmarks sidecars.",
                  "example": false
                },
                "download_speed_limit_kbps": {
                  "type": "integer",
                  "format": "int32",
                  "description": "Fetch speed cap in KB/s (`0` = unlimited).",
                  "example": 0,
                  "minimum": 0
                },
                "lame": {
                  "type": "object",
                  "description": "LAME encoder knobs.",
                  "properties": {
                    "target": {
                      "type": "string",
                      "description": "`LameTarget`: `quality`, `bitrate`, `constant`, etc.",
                      "default": "quality",
                      "example": "quality"
                    },
                    "vbr_quality": {
                      "type": "integer",
                      "format": "int32",
                      "description": "`LameVBRQuality` (0–9, lower = better).",
                      "default": 2,
                      "example": 2,
                      "minimum": 0
                    },
                    "bitrate_kbps": {
                      "type": "integer",
                      "format": "int32",
                      "description": "`LameBitrate` in kbps when target is bitrate/constant.",
                      "default": 128,
                      "example": 128,
                      "minimum": 0
                    },
                    "mode": {
                      "type": "string",
                      "description": "`LameMode`: `default`, `mono`, `stereo`.",
                      "default": "default",
                      "example": "default"
                    },
                    "downsample_mono": {
                      "type": "boolean",
                      "description": "`LameDownsampleMono`.",
                      "default": false,
                      "example": false
                    },
                    "constant_bitrate": {
                      "type": "boolean",
                      "description": "`LameConstantBitrate`.",
                      "default": false,
                      "example": false
                    }
                  },
                  "additionalProperties": false
                },
                "max_sample_rate": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int32",
                  "description": "Optional output sample-rate ceiling in Hz.",
                  "example": 44100,
                  "minimum": 0
                },
                "creation_time": {
                  "type": "string",
                  "description": "File creation timestamp policy.",
                  "enum": [
                    "now",
                    "purchased",
                    "published"
                  ],
                  "example": "now"
                },
                "last_write_time": {
                  "type": "string",
                  "description": "File last-write timestamp policy.",
                  "enum": [
                    "now",
                    "purchased",
                    "published"
                  ],
                  "example": "now"
                },
                "path_sanitization": {
                  "type": "string",
                  "description": "Sanitization profile used when no explicit rules are set.",
                  "enum": [
                    "auto",
                    "windows",
                    "posix",
                    "s3",
                    "none"
                  ],
                  "example": "auto"
                },
                "replacement_characters": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "Character replacement rule (`ReplacementCharacters` in classic Settings.json).",
                    "required": [
                      "find",
                      "replace"
                    ],
                    "properties": {
                      "find": {
                        "type": "string",
                        "description": "Substring to search for in each path segment / tag value.",
                        "example": ":"
                      },
                      "replace": {
                        "type": "string",
                        "description": "Replacement text (may be empty to strip `find`).",
                        "example": " -"
                      }
                    },
                    "additionalProperties": false
                  },
                  "description": "Explicit find/replace rules for path segments.",
                  "example": [
                    {
                      "find": ":",
                      "replace": " -"
                    }
                  ]
                },
                "max_filename_length": {
                  "type": "integer",
                  "format": "int32",
                  "description": "Max length per path segment (`0` disables truncation).",
                  "example": 255,
                  "minimum": 0
                }
              },
              "additionalProperties": false
            }
          }
        }
      ],
      "description": "The change was written; these are the settings now (boxed: the\noutput document is far larger than a replay note)."
    },
    {
      "type": "object",
      "description": "The write committed; another write landed before the read-back.",
      "required": [
        "namespace",
        "replayed",
        "revision"
      ],
      "properties": {
        "namespace": {
          "type": "string",
          "description": "Domain namespace (`core.events` or `core.output`).",
          "example": "core.events"
        },
        "replayed": {
          "type": "boolean",
          "description": "Always `true`: the write was a replay.",
          "example": true
        },
        "revision": {
          "type": "integer",
          "format": "int64",
          "description": "Revision the earlier write produced.",
          "example": 12
        }
      }
    }
  ],
  "description": "Body of `PUT /api/config/domains/core.output`: the settings after the\nchange, or a replay note."
}
```

Status `400`: The body is invalid, or a key or value is not an output setting. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid configuration: cover_size must be `native` or a pixel size 1..=10000"
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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `409`: `expected_revision` is stale; `current_revision` is the revision to retry against.

```json
{
  "current_revision": 7,
  "error": "revision_conflict"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of a `409 Conflict` from a compare-and-swap write whose\n`expected_revision` lost to a newer revision.",
  "required": [
    "error",
    "current_revision"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Always `revision_conflict`.",
      "example": "revision_conflict"
    },
    "current_revision": {
      "type": "integer",
      "format": "int64",
      "description": "The revision to read and retry against.",
      "example": 13
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
Failed to deserialize the JSON body into the target type: missing field `expected_revision` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The configuration could not be read or written.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/ingress`

`GET /api/config/ingress`: the library's public address settings.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — The ingress settings. Body: [`IngressStatus`](schemas.md#ingressstatus) (`application/json`)
  - `active_instance` *string or null* — The plugin instance serving the library's public address, as its UUID; `null` when ingress is off.
  - `plugin_key` *string or null* — The active instance's plugin, when the instance exists.
  - `exposure` *[`IngressExposure`](schemas.md#ingressexposure)*, **required** — What public traffic may reach.
  - `origin` *string or null* — The adopted public origin, kept while ingress is off.
  - `reported_origin` *string or null* — A different origin the active instance reported, waiting for the operator to accept it.
  - `public_origin` *string or null* — The origin the library serves under now: `integrations.public_origin` when set, else the ingress origin while the instance runs.
  - `operator_override` *boolean*, **required** — True when `integrations.public_origin` in `config.toml` overrides the ingress origin.
  - `blocked` *string or null* — Why ingress can't run now (no Owner yet, or operator auth off); `null` when it can.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/ingress' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/ingress' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/ingress",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/ingress", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/ingress")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The ingress settings.

```json
{
  "active_instance": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "plugin_key": "cloudflare-relay",
  "exposure": "branches",
  "origin": "https://relay.example.workers.dev",
  "reported_origin": "https://library.example.net",
  "public_origin": "https://relay.example.workers.dev",
  "operator_override": false,
  "blocked": null
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/config/ingress` and the answer to `POST /api/config/ingress`: the library's\ningress settings and what they mean now.",
  "required": [
    "exposure",
    "operator_override"
  ],
  "properties": {
    "active_instance": {
      "type": [
        "string",
        "null"
      ],
      "description": "The plugin instance serving the library's public address, as its\nUUID; `null` when ingress is off.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "plugin_key": {
      "type": [
        "string",
        "null"
      ],
      "description": "The active instance's plugin, when the instance exists.",
      "example": "cloudflare-relay"
    },
    "exposure": {
      "type": "string",
      "description": "What public traffic may reach.",
      "enum": [
        "branches",
        "library"
      ]
    },
    "origin": {
      "type": [
        "string",
        "null"
      ],
      "description": "The adopted public origin, kept while ingress is off.",
      "example": "https://relay.example.workers.dev"
    },
    "reported_origin": {
      "type": [
        "string",
        "null"
      ],
      "description": "A different origin the active instance reported, waiting for the\noperator to accept it.",
      "example": "https://library.example.net"
    },
    "public_origin": {
      "type": [
        "string",
        "null"
      ],
      "description": "The origin the library serves under now: `integrations.public_origin`\nwhen set, else the ingress origin while the instance runs.",
      "example": "https://relay.example.workers.dev"
    },
    "operator_override": {
      "type": "boolean",
      "description": "True when `integrations.public_origin` in `config.toml` overrides the\ningress origin.",
      "example": false
    },
    "blocked": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why ingress can't run now (no Owner yet, or operator auth off);\n`null` when it can.",
      "example": null
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

Status `403`: The caller's role does not allow this route.

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

Status `500`: The library could not be read.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/config/ingress`

`POST /api/config/ingress`: turns ingress on or off, sets what it exposes, or
accepts a reported origin. The deployment reconciler then starts or
stops the instance.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`IngressChange`](schemas.md#ingresschange)

- `activate` *string or null* — Makes this the library's active ingress: a plugin instance UUID, or a plugin key for that plugin's default instance.
- `deactivate` *boolean* — Turns ingress off; the adopted origin is kept.
- `exposure` *[`IngressExposure`](schemas.md#ingressexposure) or null* — What public traffic may reach.
- `accept_reported_origin` *boolean* — Adopts the origin the active instance reported. Renaming the library breaks passkeys and sign-in redirects registered under the old name.

### Responses

- `200` — The settings after the change. Body: [`IngressStatus`](schemas.md#ingressstatus) (`application/json`)
  - `active_instance` *string or null* — The plugin instance serving the library's public address, as its UUID; `null` when ingress is off.
  - `plugin_key` *string or null* — The active instance's plugin, when the instance exists.
  - `exposure` *[`IngressExposure`](schemas.md#ingressexposure)*, **required** — What public traffic may reach.
  - `origin` *string or null* — The adopted public origin, kept while ingress is off.
  - `reported_origin` *string or null* — A different origin the active instance reported, waiting for the operator to accept it.
  - `public_origin` *string or null* — The origin the library serves under now: `integrations.public_origin` when set, else the ingress origin while the instance runs.
  - `operator_override` *boolean*, **required** — True when `integrations.public_origin` in `config.toml` overrides the ingress origin.
  - `blocked` *string or null* — Why ingress can't run now (no Owner yet, or operator auth off); `null` when it can.
- `400` — `activate` and `deactivate` were both set, or `activate` is neither an instance id nor a plugin key with an instance. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — Ingress can't be turned on yet: no Owner exists, or operator auth is off. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/ingress' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/ingress' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/ingress",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/ingress", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/ingress")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings after the change.

```json
{
  "active_instance": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "plugin_key": "cloudflare-relay",
  "exposure": "branches",
  "origin": "https://relay.example.workers.dev",
  "reported_origin": "https://library.example.net",
  "public_origin": "https://relay.example.workers.dev",
  "operator_override": false,
  "blocked": null
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/config/ingress` and the answer to `POST /api/config/ingress`: the library's\ningress settings and what they mean now.",
  "required": [
    "exposure",
    "operator_override"
  ],
  "properties": {
    "active_instance": {
      "type": [
        "string",
        "null"
      ],
      "description": "The plugin instance serving the library's public address, as its\nUUID; `null` when ingress is off.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "plugin_key": {
      "type": [
        "string",
        "null"
      ],
      "description": "The active instance's plugin, when the instance exists.",
      "example": "cloudflare-relay"
    },
    "exposure": {
      "type": "string",
      "description": "What public traffic may reach.",
      "enum": [
        "branches",
        "library"
      ]
    },
    "origin": {
      "type": [
        "string",
        "null"
      ],
      "description": "The adopted public origin, kept while ingress is off.",
      "example": "https://relay.example.workers.dev"
    },
    "reported_origin": {
      "type": [
        "string",
        "null"
      ],
      "description": "A different origin the active instance reported, waiting for the\noperator to accept it.",
      "example": "https://library.example.net"
    },
    "public_origin": {
      "type": [
        "string",
        "null"
      ],
      "description": "The origin the library serves under now: `integrations.public_origin`\nwhen set, else the ingress origin while the instance runs.",
      "example": "https://relay.example.workers.dev"
    },
    "operator_override": {
      "type": "boolean",
      "description": "True when `integrations.public_origin` in `config.toml` overrides the\ningress origin.",
      "example": false
    },
    "blocked": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why ingress can't run now (no Owner yet, or operator auth off);\n`null` when it can.",
      "example": null
    }
  }
}
```

Status `400`: `activate` and `deactivate` were both set, or `activate` is neither an instance id nor a plugin key with an instance. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "no plugin instance for `cloudflare-relay`"
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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `409`: Ingress can't be turned on yet: no Owner exists, or operator auth is off.

```json
{
  "error": "no Owner account exists yet: claim the library on this machine first, so first-run setup is never public"
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: activate: invalid type: integer `1`, expected a string at line 1 column 13
```

Schema:

A `text/plain` body: a string.

Status `500`: The library could not be read or written.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/clerk`

`GET /api/config/clerk`.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The libraries this install holds a library card for. Body: [`ClerkCards`](schemas.md#clerkcards) (`application/json`)
  - `cards` *array of [`ClerkLinkView`](schemas.md#clerklinkview)*, **required** — The libraries this install holds (or held) a card for, oldest sign-in first.
  - `default_clerk_name` *string*, **required** — The name this install signs in as unless another is given.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner, or the session has no user account. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorAnswer`](schemas.md#erroranswer) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clerk' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/clerk' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/clerk",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/clerk", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/clerk")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The libraries this install holds a library card for.

```json
{
  "cards": [
    {
      "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
      "origin": "https://books.example.com",
      "library_name": "books.example.com",
      "pinned": false,
      "display_name": "Riley",
      "clerk_name": "Riley's laptop",
      "state": "active",
      "signed_in_at": "2026-11-17T09:02:41Z"
    }
  ],
  "default_clerk_name": "study-mini"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/config/clerk`.",
  "required": [
    "cards",
    "default_clerk_name"
  ],
  "properties": {
    "cards": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A library this install holds (or held) a library card for, without the\ncard.",
        "required": [
          "library_id",
          "origin",
          "library_name",
          "pinned",
          "clerk_name",
          "state",
          "signed_in_at"
        ],
        "properties": {
          "library_id": {
            "type": "string",
            "description": "The library id.",
            "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
          },
          "origin": {
            "type": "string",
            "description": "The library's canonical origin.",
            "example": "https://books.example.com"
          },
          "library_name": {
            "type": "string",
            "description": "The library's name at sign-in.",
            "example": "books.example.com"
          },
          "pinned": {
            "type": "boolean",
            "description": "Whether the card's connection pins the library's self-signed\ncertificate.",
            "example": false
          },
          "display_name": {
            "type": [
              "string",
              "null"
            ],
            "description": "The person's display name on the library, when it has one.",
            "example": "Riley"
          },
          "clerk_name": {
            "type": "string",
            "description": "The name this install signed in as.",
            "example": "Riley's laptop"
          },
          "state": {
            "type": "string",
            "description": "`active`, or `signed_out` once the library ended the card.",
            "example": "active"
          },
          "signed_in_at": {
            "type": "string",
            "description": "RFC 3339 sign-in time.",
            "example": "2026-11-17T09:02:41Z"
          }
        }
      },
      "description": "The libraries this install holds (or held) a card for, oldest\nsign-in first."
    },
    "default_clerk_name": {
      "type": "string",
      "description": "The name this install signs in as unless another is given.",
      "example": "study-mini"
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

Status `403`: The user is not an active Owner, or the session has no user account. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

```json
{
  "error": "forbidden"
}
```

Schema:

```json
{
  "anyOf": [
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
    },
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
  ]
}
```

Status `500`: The library database could not be read.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `503`: The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out.

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

## `POST /api/config/clerk/inspect`

`POST /api/config/clerk/inspect`.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`ClerkInspectBody`](schemas.md#clerkinspectbody)

- `address` *string*, **required** — The library's address (`https://books.example.com`, `books.lan:8443`).

### Responses

- `200` — What the library says about itself. Body: [`InspectedLibraryResponse`](schemas.md#inspectedlibraryresponse) (`application/json`)
  - `library` *[`InspectedLibraryView`](schemas.md#inspectedlibraryview)*, **required** — What the library said about itself.
- `400` — The address is not a library (`invalid_address`), or is plain HTTP off this machine (`transport_insecure`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner, or the session has no user account. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `409` — The library's address is not the one given (`origin_mismatch`, with the address to use in `origin`). Body: [`JoinConflict`](schemas.md#joinconflict) (`application/json`)
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `502` — The library could not be reached (`unreachable`), or answered with an error (`library_error`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clerk/inspect' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"address": "https://books.example.com"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/clerk/inspect' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"address": "https://books.example.com"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/clerk/inspect",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"address": "https://books.example.com"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/clerk/inspect", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"address": "https://books.example.com"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/clerk/inspect")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"address": "https://books.example.com"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: What the library says about itself.

```json
{
  "library": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "library_name": "books.example.com",
    "origin": "https://books.example.com",
    "transport": "self_signed",
    "enrollment": true,
    "fingerprint": "kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o",
    "needs_fingerprint": true
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `POST /api/config/join/inspect`.",
  "required": [
    "library"
  ],
  "properties": {
    "library": {
      "type": "object",
      "description": "What the library said about itself.",
      "required": [
        "library_id",
        "library_name",
        "origin",
        "transport",
        "enrollment",
        "needs_fingerprint"
      ],
      "properties": {
        "library_id": {
          "type": "string",
          "description": "The library id.",
          "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
        },
        "library_name": {
          "type": "string",
          "description": "A name to show for the library.",
          "example": "books.example.com"
        },
        "origin": {
          "type": "string",
          "description": "Its canonical origin.",
          "example": "https://books.example.com"
        },
        "transport": {
          "type": "string",
          "description": "The node API transport mode (`acme`, `proxy`, `self_signed`, `loopback`).",
          "example": "self_signed"
        },
        "enrollment": {
          "type": "boolean",
          "description": "Whether it lets new branches join.",
          "example": true
        },
        "fingerprint": {
          "type": [
            "string",
            "null"
          ],
          "description": "The SPKI fingerprint to confirm (self-signed libraries only).",
          "example": "kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"
        },
        "needs_fingerprint": {
          "type": "boolean",
          "description": "Whether the operator must confirm `fingerprint` before joining.",
          "example": true
        }
      }
    }
  }
}
```

Status `400`: The address is not a library (`invalid_address`), or is plain HTTP off this machine (`transport_insecure`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "not an enrollment link: no host",
  "error": "invalid_address"
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

Status `403`: The user is not an active Owner, or the session has no user account. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden"
}
```

Schema:

```json
{
  "anyOf": [
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
    },
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
  ]
}
```

Status `409`: The library's address is not the one given (`origin_mismatch`, with the address to use in `origin`).

```json
{
  "detail": "this library's address is https://books.example.com; join with that address",
  "error": "origin_mismatch",
  "origin": "https://books.example.com"
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "The handler's slug in `error` and a sentence in `detail`.",
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
    },
    {
      "type": "object",
      "description": "The library answers under another address, given in `origin`.",
      "required": [
        "error",
        "detail",
        "origin"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "Always `origin_mismatch`.",
          "example": "origin_mismatch"
        },
        "detail": {
          "type": "string",
          "description": "Why, naming the address to join with instead.",
          "example": "this library's address is https://books.example.com; join with that address"
        },
        "origin": {
          "type": "string",
          "description": "The library's canonical origin, to join with instead.",
          "example": "https://books.example.com"
        }
      }
    }
  ],
  "description": "A `409` from a join step: the handler's error, or an origin mismatch that names the address to join with; the route's examples name each."
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
Failed to deserialize the JSON body into the target type: missing field `address` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `502`: The library could not be reached (`unreachable`), or answered with an error (`library_error`).

```json
{
  "detail": "owner unreachable: Connection refused (os error 111)",
  "error": "unreachable"
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

Status `503`: The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out.

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

## `POST /api/config/clerk/start`

`POST /api/config/clerk/start`.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`ClerkStartBody`](schemas.md#clerkstartbody)

- `address` *string* — The library's address. Left out with `from_branch`.
- `from_branch` *string or null* — The library id of a library this install is an active branch of: the card replaces the branch. The library's address and pinned certificate come from the branch link, so `address` and `fingerprint` are left out; once the library hands over the card, the callback leaves the branch (as Leave library does) and records the card. Only the operator may, as only the operator leaves a library.
- `fingerprint` *string or null* — The fingerprint the person confirmed (self-signed libraries).
- `clerk_name` *string or null* — The name to sign in as (default: this machine's name).

### Responses

- `200` — The library's sign-in URL to open. Body: [`ClerkStarted`](schemas.md#clerkstarted) (`application/json`)
  - `authorize_url` *string*, **required** — The library's sign-in URL to open in the browser.
  - `library` *[`InspectedLibraryView`](schemas.md#inspectedlibraryview)*, **required** — What the library said about itself.
  - `clerk_name` *string*, **required** — The name this install signs in as.
  - `expires_in_secs` *integer (int64)*, **required** — Seconds the pending sign-in stays valid. At least 0.
- `400` — The address is not a library, is plain HTTP off this machine, or the name or this install's address cannot sign in (`invalid_address`, `transport_insecure`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner, the session has no user account, or a sign-in `from_branch` is not the operator's. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — With `from_branch`: this install is not an active branch of that library (`not_found`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `409` — The library's fingerprint is not confirmed, or changed since the branch joined (`fingerprint_unconfirmed`), this install already joined that library as a branch (`already_joined`), or the library's address is not the one given or no longer the library the branch joined (`origin_mismatch`). Body: [`JoinConflict`](schemas.md#joinconflict) (`application/json`)
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The library could not be reached (`unreachable`), or answered with an error (`library_error`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clerk/start' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/clerk/start' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/clerk/start",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/clerk/start", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/clerk/start")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The library's sign-in URL to open.

```json
{
  "authorize_url": "https://books.example.com/oidc/authorize?client_id=bookclerk-clerk&redirect_uri=…",
  "library": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "library_name": "books.example.com",
    "origin": "https://books.example.com",
    "transport": "self_signed",
    "enrollment": true,
    "fingerprint": "kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o",
    "needs_fingerprint": true
  },
  "clerk_name": "Riley's laptop",
  "expires_in_secs": 600
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `POST /api/config/clerk/start`.",
  "required": [
    "authorize_url",
    "library",
    "clerk_name",
    "expires_in_secs"
  ],
  "properties": {
    "authorize_url": {
      "type": "string",
      "description": "The library's sign-in URL to open in the browser.",
      "example": "https://books.example.com/oidc/authorize?client_id=bookclerk-clerk&redirect_uri=…"
    },
    "library": {
      "type": "object",
      "description": "What the library said about itself.",
      "required": [
        "library_id",
        "library_name",
        "origin",
        "transport",
        "enrollment",
        "needs_fingerprint"
      ],
      "properties": {
        "library_id": {
          "type": "string",
          "description": "The library id.",
          "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
        },
        "library_name": {
          "type": "string",
          "description": "A name to show for the library.",
          "example": "books.example.com"
        },
        "origin": {
          "type": "string",
          "description": "Its canonical origin.",
          "example": "https://books.example.com"
        },
        "transport": {
          "type": "string",
          "description": "The node API transport mode (`acme`, `proxy`, `self_signed`, `loopback`).",
          "example": "self_signed"
        },
        "enrollment": {
          "type": "boolean",
          "description": "Whether it lets new branches join.",
          "example": true
        },
        "fingerprint": {
          "type": [
            "string",
            "null"
          ],
          "description": "The SPKI fingerprint to confirm (self-signed libraries only).",
          "example": "kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"
        },
        "needs_fingerprint": {
          "type": "boolean",
          "description": "Whether the operator must confirm `fingerprint` before joining.",
          "example": true
        }
      }
    },
    "clerk_name": {
      "type": "string",
      "description": "The name this install signs in as.",
      "example": "Riley's laptop"
    },
    "expires_in_secs": {
      "type": "integer",
      "format": "int64",
      "description": "Seconds the pending sign-in stays valid.",
      "example": 600,
      "minimum": 0
    }
  }
}
```

Status `400`: The address is not a library, is plain HTTP off this machine, or the name or this install's address cannot sign in (`invalid_address`, `transport_insecure`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "this computer's name must be 1 to 64 characters",
  "error": "invalid_address"
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

Status `403`: The user is not an active Owner, the session has no user account, or a sign-in `from_branch` is not the operator's. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden"
}
```

Schema:

```json
{
  "anyOf": [
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
    },
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
  ]
}
```

Status `404`: With `from_branch`: this install is not an active branch of that library (`not_found`).

```json
{
  "detail": "this install is not a branch of that library",
  "error": "not_found"
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

Status `409`: The library's fingerprint is not confirmed, or changed since the branch joined (`fingerprint_unconfirmed`), this install already joined that library as a branch (`already_joined`), or the library's address is not the one given or no longer the library the branch joined (`origin_mismatch`).

```json
{
  "detail": "this install already joined that library; its books are read through that link",
  "error": "already_joined"
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "The handler's slug in `error` and a sentence in `detail`.",
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
    },
    {
      "type": "object",
      "description": "The library answers under another address, given in `origin`.",
      "required": [
        "error",
        "detail",
        "origin"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "Always `origin_mismatch`.",
          "example": "origin_mismatch"
        },
        "detail": {
          "type": "string",
          "description": "Why, naming the address to join with instead.",
          "example": "this library's address is https://books.example.com; join with that address"
        },
        "origin": {
          "type": "string",
          "description": "The library's canonical origin, to join with instead.",
          "example": "https://books.example.com"
        }
      }
    }
  ],
  "description": "A `409` from a join step: the handler's error, or an origin mismatch that names the address to join with; the route's examples name each."
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
Failed to deserialize the JSON body into the target type: address: invalid type: integer `1`, expected a string at line 1 column 12
```

Schema:

A `text/plain` body: a string.

Status `500`: The library database could not be read.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `502`: The library could not be reached (`unreachable`), or answered with an error (`library_error`).

```json
{
  "detail": "owner unreachable: Connection refused (os error 111)",
  "error": "unreachable"
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

Status `503`: The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out.

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

## `POST /api/config/clerk/callback`

`POST /api/config/clerk/callback`: redeems the library's code with the
sign-in's verifier, seals the card and starts reading the person's
books. The pending sign-in is ended whatever the outcome; a failed one
is started again.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`ClerkCallbackBody`](schemas.md#clerkcallbackbody)

- `code` *string* — The code from the library.
- `state` *string* — The `state` it came back with.
- `error` *string or null* — The library's `error` (`access_denied`), when it sent one.

### Responses

- `200` — This install holds a library card for the library now; a sign-in that replaced a branch left it first (`left_branch`). Body: [`ClerkSignedIn`](schemas.md#clerksignedin) (`application/json`)
  - `card` *[`ClerkLinkView`](schemas.md#clerklinkview)*, **required** — The library this install now holds a card for.
  - `left_branch` *[`LeftBranch`](schemas.md#leftbranch) or null* — Set when the card replaced this install's branch link to the library (a sign-in started `from_branch`): the branch was left.
- `400` — The callback carries no code (`invalid_request`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The library's user did not approve (`denied`), the library refused the code (`refused`), or the user is not an active Owner (`forbidden`). A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `409` — This install joined that library as a branch meanwhile (`already_joined`), or joined it again after a sign-in that replaces its branch started (`enrollment_changed`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `410` — The sign-in is unknown, already used, or older than ten minutes; start again (`join_expired`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The sign-in stopped before it finished (`sign_in_failed`), or the branch it replaces could not be left (the library database could not be read or written). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `502` — The library could not be reached (`unreachable`), answered with an error (`library_error`), its answer did not parse (`join_failed`), or the card could not be saved here (`join_unsaved`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clerk/callback' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/clerk/callback' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/clerk/callback",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/clerk/callback", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/clerk/callback")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: This install holds a library card for the library now; a sign-in that replaced a branch left it first (`left_branch`).

```json
{
  "card": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "origin": "https://books.example.com",
    "library_name": "books.example.com",
    "pinned": false,
    "display_name": "Riley",
    "clerk_name": "Riley's laptop",
    "state": "active",
    "signed_in_at": "2026-11-17T09:02:41Z"
  },
  "left_branch": {
    "owner_notified": true
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `POST /api/config/clerk/callback`.",
  "required": [
    "card"
  ],
  "properties": {
    "card": {
      "type": "object",
      "description": "The library this install now holds a card for.",
      "required": [
        "library_id",
        "origin",
        "library_name",
        "pinned",
        "clerk_name",
        "state",
        "signed_in_at"
      ],
      "properties": {
        "library_id": {
          "type": "string",
          "description": "The library id.",
          "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
        },
        "origin": {
          "type": "string",
          "description": "The library's canonical origin.",
          "example": "https://books.example.com"
        },
        "library_name": {
          "type": "string",
          "description": "The library's name at sign-in.",
          "example": "books.example.com"
        },
        "pinned": {
          "type": "boolean",
          "description": "Whether the card's connection pins the library's self-signed\ncertificate.",
          "example": false
        },
        "display_name": {
          "type": [
            "string",
            "null"
          ],
          "description": "The person's display name on the library, when it has one.",
          "example": "Riley"
        },
        "clerk_name": {
          "type": "string",
          "description": "The name this install signed in as.",
          "example": "Riley's laptop"
        },
        "state": {
          "type": "string",
          "description": "`active`, or `signed_out` once the library ended the card.",
          "example": "active"
        },
        "signed_in_at": {
          "type": "string",
          "description": "RFC 3339 sign-in time.",
          "example": "2026-11-17T09:02:41Z"
        }
      }
    },
    "left_branch": {
      "oneOf": [
        {
          "type": "object",
          "description": "Set when the card replaced this install's branch link to the\nlibrary (a sign-in started `from_branch`): the branch was left.",
          "required": [
            "owner_notified"
          ],
          "properties": {
            "owner_notified": {
              "type": "boolean",
              "description": "Whether the library accepted the leave notice. False when it could\nnot be told: the branch then stays in its list until it is removed\nthere.",
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

Status `400`: The callback carries no code (`invalid_request`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "no code",
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

Status `403`: The library's user did not approve (`denied`), the library refused the code (`refused`), or the user is not an active Owner (`forbidden`). A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "detail": "the library's user did not let this computer show their books",
  "error": "denied"
}
```

Schema:

```json
{
  "anyOf": [
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
    },
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
  ]
}
```

Status `409`: This install joined that library as a branch meanwhile (`already_joined`), or joined it again after a sign-in that replaces its branch started (`enrollment_changed`).

```json
{
  "detail": "this install already joined that library; its books are read through that link",
  "error": "already_joined"
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

Status `410`: The sign-in is unknown, already used, or older than ten minutes; start again (`join_expired`).

```json
{
  "detail": "this sign-in is unknown, already used, or took longer than 10 minutes; start again",
  "error": "join_expired"
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: code: invalid type: integer `1`, expected a string at line 1 column 9
```

Schema:

A `text/plain` body: a string.

Status `500`: The sign-in stopped before it finished (`sign_in_failed`), or the branch it replaces could not be left (the library database could not be read or written).

```json
{
  "detail": "the sign-in stopped before it finished; check the library list",
  "error": "sign_in_failed"
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

Status `502`: The library could not be reached (`unreachable`), answered with an error (`library_error`), its answer did not parse (`join_failed`), or the card could not be saved here (`join_unsaved`).

```json
{
  "detail": "owner unreachable: Connection refused (os error 111)",
  "error": "unreachable"
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

Status `503`: The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out.

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

## `POST /api/config/clerk/sign-out`

`POST /api/config/clerk/sign-out`: ends a library card at the library
when it can be reached, then drops the card, the books read with it,
and the library from the list here whatever the library answered.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`ClerkSignOutBody`](schemas.md#clerksignoutbody)

- `library_id` *string*, **required** — The library id.

### Responses

- `200` — The card and its books are gone here; `library_signed_out` says whether the library ended its session too. Body: [`ClerkSignedOut`](schemas.md#clerksignedout) (`application/json`)
  - `library_id` *string*, **required** — The library id.
  - `library_signed_out` *boolean*, **required** — Whether the library ended the card's session too. False when it could not be reached (or the card had already ended): the session then stays in the person's session list there until it idles out or they revoke it.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner, or the session has no user account. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — This install holds no card for that library (`not_found`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clerk/sign-out' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/clerk/sign-out' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/clerk/sign-out",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/clerk/sign-out", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/clerk/sign-out")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The card and its books are gone here; `library_signed_out` says whether the library ended its session too.

```json
{
  "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
  "library_signed_out": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `POST /api/config/clerk/sign-out`.",
  "required": [
    "library_id",
    "library_signed_out"
  ],
  "properties": {
    "library_id": {
      "type": "string",
      "description": "The library id.",
      "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
    },
    "library_signed_out": {
      "type": "boolean",
      "description": "Whether the library ended the card's session too. False when it\ncould not be reached (or the card had already ended): the session\nthen stays in the person's session list there until it idles out\nor they revoke it.",
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

Status `403`: The user is not an active Owner, or the session has no user account. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden"
}
```

Schema:

```json
{
  "anyOf": [
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
    },
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
  ]
}
```

Status `404`: This install holds no card for that library (`not_found`).

```json
{
  "detail": "no such library card",
  "error": "not_found"
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `library_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The library database could not be written.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `503`: The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out.

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

## `POST /api/config/clerk/request`

`POST /api/config/clerk/request`: asks a library this install holds a
card for, with the card, to get a copy of a book: from one of its
branches (`from` `branch`) or by downloading it on one of its own store
accounts (`from` `library`).

Only an active Owner of this install, or its operator, may send it: the
request goes out with this install's library card.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`ClerkRequestBody`](schemas.md#clerkrequestbody)

- `library_id` *string*, **required** — The library to ask: one this install holds an active card for.
- `work_id` *string*, **required** — The library's own work id: 1 to 64 ASCII letters, digits and `-`.
- `from` *[`ClerkRequestFrom`](schemas.md#clerkrequestfrom)*, **required** — Where the copy comes from: the edition's `request.from` in the window books.
- `node_id` *string or null* — The library's branch to ask, from the work's `available_through` there; absent asks the first branch that lists it. Only with `from` `branch`.

### Responses

- `200` — The library took the request; it shows there with the person's other requests, or as a download in its jobs. Body: [`ClerkRequestSent`](schemas.md#clerkrequestsent) (`application/json`)
  - `library_id` *string*, **required** — The library that took the request.
- `400` — The work or node id is malformed, or a node id was given with `from` `library` (`invalid_request`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), or the library's member controls turn requesting books off for the person (`not_allowed`). A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — This install holds no active card for that library (`not_found`), or no branch of the library lists the work now, or none of its store accounts holds it (`not_listed`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `409` — The library already has a copy of the work (`already_there`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The library could not be reached (`unreachable`), no longer accepts the card (`card_refused`), or answered with another error (`library_error`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `503` — The library's job queue is full (`library_busy`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorDetail`](schemas.md#errordetail) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clerk/request' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f", "work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b", "from": "branch"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/clerk/request' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f", "work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b", "from": "branch"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/clerk/request",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f", "work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b", "from": "branch"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/clerk/request", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f", "work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b", "from": "branch"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/clerk/request")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f", "work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b", "from": "branch"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The library took the request; it shows there with the person's other requests, or as a download in its jobs.

```json
{
  "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `POST /api/config/clerk/request`.",
  "required": [
    "library_id"
  ],
  "properties": {
    "library_id": {
      "type": "string",
      "description": "The library that took the request.",
      "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
    }
  }
}
```

Status `400`: The work or node id is malformed, or a node id was given with `from` `library` (`invalid_request`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "invalid work_id",
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

Status `403`: The user is not an active Owner (`forbidden`), or the library's member controls turn requesting books off for the person (`not_allowed`). A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
    },
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
  ]
}
```

Status `404`: This install holds no active card for that library (`not_found`), or no branch of the library lists the work now, or none of its store accounts holds it (`not_listed`).

```json
{
  "detail": "no branch of that library lists this book now",
  "error": "not_listed"
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

Status `409`: The library already has a copy of the work (`already_there`).

```json
{
  "detail": "the library already has a copy; it shows here after the next refresh",
  "error": "already_there"
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `library_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The library database could not be read.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `502`: The library could not be reached (`unreachable`), no longer accepts the card (`card_refused`), or answered with another error (`library_error`).

```json
{
  "detail": "the library could not be reached",
  "error": "unreachable"
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

Status `503`: The library's job queue is full (`library_busy`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "detail": "the library's job queue is full; try again later",
  "error": "library_busy"
}
```

Schema:

```json
{
  "anyOf": [
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
    },
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
  ]
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/branch-enrollment`

`GET /api/config/branch-enrollment`: whether branch nodes may enroll.

### Returns

`{enabled, revision}`; `revision` is `null` while the default (enabled)
applies.

### Errors

Returns an error response when the switch cannot be read.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Whether branch nodes may enroll. Body: [`BranchEnrollment`](schemas.md#branchenrollment) (`application/json`)
  - `enabled` *boolean*, **required** — Whether branch nodes may enroll into this library.
  - `revision` *integer (int64) or null* — Stored revision; `null` while the default (enabled) applies.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/branch-enrollment' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/branch-enrollment' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/branch-enrollment",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/branch-enrollment", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/branch-enrollment")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Whether branch nodes may enroll.

```json
{
  "enabled": true,
  "revision": 3
}
```

Schema:

```json
{
  "type": "object",
  "description": "The branch-enrollment switch.",
  "required": [
    "enabled"
  ],
  "properties": {
    "enabled": {
      "type": "boolean",
      "description": "Whether branch nodes may enroll into this library.",
      "example": true
    },
    "revision": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Stored revision; `null` while the default (enabled) applies.",
      "example": 3
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

Status `403`: The session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`).

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

Status `500`: The library database could not be read.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `503`: The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out.

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

## `PUT /api/config/branch-enrollment`

`PUT /api/config/branch-enrollment`: turn branch enrollment on or off. The
operator may always; a user only while the stored role is an active
Owner.

### Arguments

* `caller` - Who is asking; a user's role is read from the database.
* `body` - The new switch value.

### Returns

`{enabled, revision}` as stored.

### Errors

Returns 403 when the caller is neither the operator nor an active Owner,
and an error response when the switch cannot be written.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PutBranchEnrollment`](schemas.md#putbranchenrollment)

- `enabled` *boolean*, **required** — Whether branch nodes may enroll into this library.

### Responses

- `200` — The switch after the change. Body: [`BranchEnrollment`](schemas.md#branchenrollment) (`application/json`)
  - `enabled` *boolean*, **required** — Whether branch nodes may enroll into this library.
  - `revision` *integer (int64) or null* — Stored revision; `null` while the default (enabled) applies.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner, the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `409` — The setting kept changing under the write; retry. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/branch-enrollment' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"enabled": true}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/branch-enrollment' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"enabled": true}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/branch-enrollment",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"enabled": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/branch-enrollment", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"enabled": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/branch-enrollment")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"enabled": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The switch after the change.

```json
{
  "enabled": true,
  "revision": 3
}
```

Schema:

```json
{
  "type": "object",
  "description": "The branch-enrollment switch.",
  "required": [
    "enabled"
  ],
  "properties": {
    "enabled": {
      "type": "boolean",
      "description": "Whether branch nodes may enroll into this library.",
      "example": true
    },
    "revision": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Stored revision; `null` while the default (enabled) applies.",
      "example": 3
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

Status `403`: The user is not an active Owner, the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `409`: The setting kept changing under the write; retry.

```json
{
  "error": "core.branch_enrollment kept changing; retry"
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

Status `500`: The library database could not be read or written.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `503`: The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out.

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

## `GET /api/config/master-key`

`GET /api/config/master-key`.

**Access:** The operator, or a signed-in Owner.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Where the key lives and what was recorded about a recovery copy. Body: [`MasterKeyStatus`](schemas.md#masterkeystatus) (`application/json`)
  - `store` *[`MasterKeyStore`](schemas.md#masterkeystore)*, **required** — Where the key lives.
  - `recovery_copy` *[`RecoveryCopyAnswer`](schemas.md#recoverycopyanswer)*, **required** — What the operator recorded about a recovery copy of this key. A record for an earlier key (since restored or replaced) reads as `unrecorded`.
  - `recorded_at` *string or null* — When that was recorded (RFC 3339, UTC); absent when `unrecorded`.
  - `needs_recovery_copy` *boolean*, **required** — Whether the web app should remind the operator to save a copy: the key is in the OS keychain or sealed with `systemd-creds` (so backups of the files directory don't include it) and no copy was recorded as saved.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is neither the operator nor an Owner (`forbidden`), or must enroll a second factor first (`mfa_enrollment_required`). Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The key could not be read (`master_key_failed`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/master-key' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/master-key' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/master-key",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/master-key", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/master-key")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Where the key lives and what was recorded about a recovery copy.

```json
{
  "needs_recovery_copy": true,
  "recorded_at": "2026-11-21T09:30:00+00:00",
  "recovery_copy": "skipped",
  "store": "keychain"
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/config/master-key` and the `PUT` answer.",
  "required": [
    "store",
    "recovery_copy",
    "needs_recovery_copy"
  ],
  "properties": {
    "store": {
      "type": "string",
      "description": "Where the key lives.",
      "enum": [
        "file",
        "password_file",
        "keychain",
        "systemd_creds"
      ]
    },
    "recovery_copy": {
      "type": "string",
      "description": "What the operator recorded about a recovery copy of this key. A\nrecord for an earlier key (since restored or replaced) reads as\n`unrecorded`.",
      "enum": [
        "unrecorded",
        "saved",
        "skipped"
      ]
    },
    "recorded_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "When that was recorded (RFC 3339, UTC); absent when `unrecorded`.",
      "example": "2026-11-21T09:30:00+00:00"
    },
    "needs_recovery_copy": {
      "type": "boolean",
      "description": "Whether the web app should remind the operator to save a copy: the\nkey is in the OS keychain or sealed with `systemd-creds` (so backups\nof the files directory don't include it) and no copy was recorded as\nsaved.",
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

Status `500`: The key could not be read (`master_key_failed`).

```json
{
  "detail": "this install has no master.key",
  "error": "master_key_failed"
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

Status `503`: The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out.

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

## `POST /api/config/master-key/recovery-copy`

`POST /api/config/master-key/recovery-copy`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`RecoveryCopyRequest`](schemas.md#recoverycopyrequest)

- `password` *string*, **required** — The password the copy is wrapped with: at least 12 characters. Keep it apart from the copy; without it the copy can't be restored.

### Responses

- `200` — The recovery copy: a `BCK2` envelope wrapped with the password, for `bookclerk config master-key import`. Sent as an attachment; nothing is recorded until `PUT …/state`. Body: [`BinaryBody`](schemas.md#binarybody) (`application/octet-stream`)
- `400` — The password is shorter than 12 characters (`password_too_short`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in as the operator. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The key could not be read or wrapped (`master_key_failed`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/master-key/recovery-copy' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"password": "correct horse battery staple"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/master-key/recovery-copy' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"password": "correct horse battery staple"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/master-key/recovery-copy",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"password": "correct horse battery staple"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/master-key/recovery-copy", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"password": "correct horse battery staple"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/master-key/recovery-copy")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"password": "correct horse battery staple"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The recovery copy: a `BCK2` envelope wrapped with the password, for `bookclerk config master-key import`. Sent as an attachment; nothing is recorded until `PUT …/state`. The body is binary (`application/octet-stream`).

Schema:

A `application/octet-stream` body: binary bytes.

Status `400`: The password is shorter than 12 characters (`password_too_short`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "a recovery password needs at least 12 characters",
  "error": "password_too_short"
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

Status `401`: The caller is not signed in as the operator.

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

Status `500`: The key could not be read or wrapped (`master_key_failed`).

```json
{
  "detail": "master key file /home/bookclerk/BookclerkFiles/master.key keeps the key in OS keychain item 6f1c…, which could not be read: the keychain is locked",
  "error": "master_key_failed"
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

## `PUT /api/config/master-key/recovery-copy/state`

`PUT /api/config/master-key/recovery-copy/state`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`RecoveryCopyStateRequest`](schemas.md#recoverycopystaterequest)

- `state` *[`RecoveryCopyChoiceBody`](schemas.md#recoverycopychoicebody)*, **required** — What the operator did.

### Responses

- `200` — The answer was recorded for the key in use. Body: [`MasterKeyStatus`](schemas.md#masterkeystatus) (`application/json`)
  - `store` *[`MasterKeyStore`](schemas.md#masterkeystore)*, **required** — Where the key lives.
  - `recovery_copy` *[`RecoveryCopyAnswer`](schemas.md#recoverycopyanswer)*, **required** — What the operator recorded about a recovery copy of this key. A record for an earlier key (since restored or replaced) reads as `unrecorded`.
  - `recorded_at` *string or null* — When that was recorded (RFC 3339, UTC); absent when `unrecorded`.
  - `needs_recovery_copy` *boolean*, **required** — Whether the web app should remind the operator to save a copy: the key is in the OS keychain or sealed with `systemd-creds` (so backups of the files directory don't include it) and no copy was recorded as saved.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in as the operator. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The key could not be read or the answer could not be written (`master_key_failed`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/master-key/recovery-copy/state' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"state": "saved"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/master-key/recovery-copy/state' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"state": "saved"}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/master-key/recovery-copy/state",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"state": "saved"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/master-key/recovery-copy/state", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"state": "saved"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/master-key/recovery-copy/state")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"state": "saved"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The answer was recorded for the key in use.

```json
{
  "needs_recovery_copy": false,
  "recorded_at": "2026-11-21T09:30:00+00:00",
  "recovery_copy": "saved",
  "store": "keychain"
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/config/master-key` and the `PUT` answer.",
  "required": [
    "store",
    "recovery_copy",
    "needs_recovery_copy"
  ],
  "properties": {
    "store": {
      "type": "string",
      "description": "Where the key lives.",
      "enum": [
        "file",
        "password_file",
        "keychain",
        "systemd_creds"
      ]
    },
    "recovery_copy": {
      "type": "string",
      "description": "What the operator recorded about a recovery copy of this key. A\nrecord for an earlier key (since restored or replaced) reads as\n`unrecorded`.",
      "enum": [
        "unrecorded",
        "saved",
        "skipped"
      ]
    },
    "recorded_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "When that was recorded (RFC 3339, UTC); absent when `unrecorded`.",
      "example": "2026-11-21T09:30:00+00:00"
    },
    "needs_recovery_copy": {
      "type": "boolean",
      "description": "Whether the web app should remind the operator to save a copy: the\nkey is in the OS keychain or sealed with `systemd-creds` (so backups\nof the files directory don't include it) and no copy was recorded as\nsaved.",
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

Status `401`: The caller is not signed in as the operator.

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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `state` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The key could not be read or the answer could not be written (`master_key_failed`).

```json
{
  "detail": "failed to write /home/bookclerk/BookclerkFiles/master-key-recovery.json: No space left on device (os error 28)",
  "error": "master_key_failed"
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
