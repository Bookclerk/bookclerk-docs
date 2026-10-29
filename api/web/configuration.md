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

Status `500`: The reload failed; the old configuration stays live.

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

Status `504`: The request ran past the API's time budget.

```text
string
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
- `500` — The configuration could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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
  "scope_type": "cluster",
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
          "description": "Scope type: `cluster` for a shared document, `node` for one node's.",
          "example": "cluster"
        },
        "scope_id": {
          "type": "string",
          "description": "Scope id: `singleton` for the cluster, else the node id.",
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
          "description": "Local delivery workers and the cluster in-flight cap.",
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

Status `500`: The configuration could not be read.

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

Status `504`: The request ran past the API's time budget.

```text
string
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
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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
  "scope_type": "cluster",
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
              "description": "Scope type: `cluster` for a shared document, `node` for one node's.",
              "example": "cluster"
            },
            "scope_id": {
              "type": "string",
              "description": "Scope id: `singleton` for the cluster, else the node id.",
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
              "description": "Local delivery workers and the cluster in-flight cap.",
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

Status `400`: The body is invalid.

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

Status `409`: `expected_revision` is stale; `current_revision` is the revision to retry against.

```json
{
  "error": "revision_conflict",
  "current_revision": 13
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

Status `500`: The configuration could not be read or written.

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

Status `504`: The request ran past the API's time budget.

```text
string
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
- `500` — The configuration could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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
  "scope_type": "cluster",
  "scope_id": "singleton",
  "schema_version": 1,
  "revision": 12,
  "updated_at": "2026-10-28T19:07:00Z",
  "updated_by": "operator",
  "document": {
    "format": "enriched_m4b",
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
    "fixup_metadata": true,
    "chapter_json": "off",
    "save_metadata_json": false,
    "cover_size": "1215",
    "chapter_layout": "tree",
    "overwrite_existing": false,
    "multi_destination": "sync_missing",
    "bad_book_action": "ask",
    "split_mp3_max_mb": 100,
    "minimum_file_duration_minutes": 1,
    "combine_nested_chapter_titles": false,
    "merge_opening_and_end_credits": true,
    "strip_unabridged": true,
    "strip_audible_brand_audio": false,
    "download_clips_bookmarks": false,
    "retain_aax_file": false,
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
          "description": "Scope type: `cluster` for a shared document, `node` for one node's.",
          "example": "cluster"
        },
        "scope_id": {
          "type": "string",
          "description": "Scope id: `singleton` for the cluster, else the node id.",
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
            "retain_aax_file",
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
            "retain_aax_file": {
              "type": "boolean",
              "description": "Keep the encrypted download.",
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

Status `500`: The configuration could not be read.

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

Status `504`: The request ran past the API's time budget.

```text
string
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
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/domains/core.output' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"expected_revision": 12, "document": {"format": "enriched_m4b", "widevine": true, "xhe_aac": false, "naming_profile": "audiobookshelf", "download_cover": true, "download_pdf": true, "create_cue": false, "fixup_metadata": true, "chapter_json": "off", "save_metadata_json": false, "cover_size": "1215", "chapter_layout": "tree", "overwrite_existing": false, "multi_destination": "sync_missing", "bad_book_action": "ask", "split_mp3_max_mb": 100, "minimum_file_duration_minutes": 1, "combine_nested_chapter_titles": false, "merge_opening_and_end_credits": true, "strip_unabridged": true, "strip_audible_brand_audio": false, "download_clips_bookmarks": false, "retain_aax_file": false, "download_speed_limit_kbps": 0, "lame": {}, "creation_time": "now", "last_write_time": "now", "path_sanitization": "auto", "replacement_characters": [{"find": ":", "replace": " -"}], "max_filename_length": 255}}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/domains/core.output' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"expected_revision": 12, "document": {"format": "enriched_m4b", "widevine": true, "xhe_aac": false, "naming_profile": "audiobookshelf", "download_cover": true, "download_pdf": true, "create_cue": false, "fixup_metadata": true, "chapter_json": "off", "save_metadata_json": false, "cover_size": "1215", "chapter_layout": "tree", "overwrite_existing": false, "multi_destination": "sync_missing", "bad_book_action": "ask", "split_mp3_max_mb": 100, "minimum_file_duration_minutes": 1, "combine_nested_chapter_titles": false, "merge_opening_and_end_credits": true, "strip_unabridged": true, "strip_audible_brand_audio": false, "download_clips_bookmarks": false, "retain_aax_file": false, "download_speed_limit_kbps": 0, "lame": {}, "creation_time": "now", "last_write_time": "now", "path_sanitization": "auto", "replacement_characters": [{"find": ":", "replace": " -"}], "max_filename_length": 255}}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/domains/core.output",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"expected_revision": 12, "document": {"format": "enriched_m4b", "widevine": True, "xhe_aac": False, "naming_profile": "audiobookshelf", "download_cover": True, "download_pdf": True, "create_cue": False, "fixup_metadata": True, "chapter_json": "off", "save_metadata_json": False, "cover_size": "1215", "chapter_layout": "tree", "overwrite_existing": False, "multi_destination": "sync_missing", "bad_book_action": "ask", "split_mp3_max_mb": 100, "minimum_file_duration_minutes": 1, "combine_nested_chapter_titles": False, "merge_opening_and_end_credits": True, "strip_unabridged": True, "strip_audible_brand_audio": False, "download_clips_bookmarks": False, "retain_aax_file": False, "download_speed_limit_kbps": 0, "lame": {}, "creation_time": "now", "last_write_time": "now", "path_sanitization": "auto", "replacement_characters": [{"find": ":", "replace": " -"}], "max_filename_length": 255}},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/domains/core.output", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"expected_revision": 12, "document": {"format": "enriched_m4b", "widevine": true, "xhe_aac": false, "naming_profile": "audiobookshelf", "download_cover": true, "download_pdf": true, "create_cue": false, "fixup_metadata": true, "chapter_json": "off", "save_metadata_json": false, "cover_size": "1215", "chapter_layout": "tree", "overwrite_existing": false, "multi_destination": "sync_missing", "bad_book_action": "ask", "split_mp3_max_mb": 100, "minimum_file_duration_minutes": 1, "combine_nested_chapter_titles": false, "merge_opening_and_end_credits": true, "strip_unabridged": true, "strip_audible_brand_audio": false, "download_clips_bookmarks": false, "retain_aax_file": false, "download_speed_limit_kbps": 0, "lame": {}, "creation_time": "now", "last_write_time": "now", "path_sanitization": "auto", "replacement_characters": [{"find": ":", "replace": " -"}], "max_filename_length": 255}}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/domains/core.output")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"expected_revision": 12, "document": {"format": "enriched_m4b", "widevine": true, "xhe_aac": false, "naming_profile": "audiobookshelf", "download_cover": true, "download_pdf": true, "create_cue": false, "fixup_metadata": true, "chapter_json": "off", "save_metadata_json": false, "cover_size": "1215", "chapter_layout": "tree", "overwrite_existing": false, "multi_destination": "sync_missing", "bad_book_action": "ask", "split_mp3_max_mb": 100, "minimum_file_duration_minutes": 1, "combine_nested_chapter_titles": false, "merge_opening_and_end_credits": true, "strip_unabridged": true, "strip_audible_brand_audio": false, "download_clips_bookmarks": false, "retain_aax_file": false, "download_speed_limit_kbps": 0, "lame": {}, "creation_time": "now", "last_write_time": "now", "path_sanitization": "auto", "replacement_characters": [{"find": ":", "replace": " -"}], "max_filename_length": 255}}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings after the change, or a `replayed: true` note with the committed revision when another write landed between this one and its read-back.

```json
{
  "namespace": "core.events",
  "scope_type": "cluster",
  "scope_id": "singleton",
  "schema_version": 1,
  "revision": 12,
  "updated_at": "2026-10-28T19:07:00Z",
  "updated_by": "operator",
  "document": {
    "format": "enriched_m4b",
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
    "fixup_metadata": true,
    "chapter_json": "off",
    "save_metadata_json": false,
    "cover_size": "1215",
    "chapter_layout": "tree",
    "overwrite_existing": false,
    "multi_destination": "sync_missing",
    "bad_book_action": "ask",
    "split_mp3_max_mb": 100,
    "minimum_file_duration_minutes": 1,
    "combine_nested_chapter_titles": false,
    "merge_opening_and_end_credits": true,
    "strip_unabridged": true,
    "strip_audible_brand_audio": false,
    "download_clips_bookmarks": false,
    "retain_aax_file": false,
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
              "description": "Scope type: `cluster` for a shared document, `node` for one node's.",
              "example": "cluster"
            },
            "scope_id": {
              "type": "string",
              "description": "Scope id: `singleton` for the cluster, else the node id.",
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
                "retain_aax_file",
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
                "retain_aax_file": {
                  "type": "boolean",
                  "description": "Keep the encrypted download.",
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

Status `400`: The body is invalid.

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

Status `409`: `expected_revision` is stale; `current_revision` is the revision to retry against.

```json
{
  "error": "revision_conflict",
  "current_revision": 13
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

Status `500`: The configuration could not be read or written.

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

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

A `text/plain` body: a string.
