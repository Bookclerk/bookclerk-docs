# Library

Books in the library, scans and acquires.

Back to the [web API reference](README.md).

## `POST /api/library/scan`

Plans and enqueues a library scan; returns the accepted job ids.

The scan planner splits a scan of every account into one job per
scan-enabled account for operators, Owners and Administrators, and
limits anyone else to the accounts they own. `job_id` is the first
accepted (or already queued) job and `job_ids` lists every one.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

Optional.

`application/json`: [`ScanRequest`](schemas.md#scanrequest) or null

- `account` *string or null* — When set, scan this account even if `scan_enabled` is false.
- `plugin_instance_id` *string or null* — When set, scan only this deployed plugin instance.

### Responses

- `200` — The scan jobs were accepted. Body: [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when at least one job was accepted.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — First accepted (or, when none was, already queued) job id.
  - `job_ids` *array of string*, **required** — Every accepted or already-queued job id, in plan order.
  - `error` *string or null* — Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`).
- `400` — The body names no valid target. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`); string (`text/plain`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not scan these accounts. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ActionResponse`](schemas.md#actionresponse) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `409` — A scan is already pending or running, or no node can run it. Body: [`PlanConflict`](schemas.md#planconflict) (`application/json`)
- `422` — No account matched. As `text/plain`, when the JSON does not fit the body. Body: [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`); string (`text/plain`)
  - `ok` *boolean*, **required** — True when at least one job was accepted.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — First accepted (or, when none was, already queued) job id.
  - `job_ids` *array of string*, **required** — Every accepted or already-queued job id, in plan order.
  - `error` *string or null* — Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`).
- `429` — The job queue is full. Body: [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when at least one job was accepted.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — First accepted (or, when none was, already queued) job id.
  - `job_ids` *array of string*, **required** — Every accepted or already-queued job id, in plan order.
  - `error` *string or null* — Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`).
- `500` — The jobs could not be queued. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
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
$ curl 'http://127.0.0.1:8787/api/library/scan' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/scan' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/library/scan",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/scan", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/library/scan")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The scan jobs were accepted.

```json
{
  "job_id": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
  "job_ids": [
    "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
    "scan-8c1d2e3f-4a5b-4c6d-9e7f-0a1b2c3d4e5f"
  ],
  "message": "scan planned: 2 accepted, 0 already pending or running, 0 not admitted (queue full)",
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Result of a planned scan or acquire: [`ActionResponse`] plus every job id.",
  "required": [
    "ok",
    "message",
    "job_id",
    "job_ids"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when at least one job was accepted.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan planned: 2 accepted, 0 already pending or running, 0 not admitted (queue full)"
    },
    "job_id": {
      "type": "string",
      "description": "First accepted (or, when none was, already queued) job id.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "job_ids": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Every accepted or already-queued job id, in plan order.",
      "example": [
        "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
        "scan-8c1d2e3f-4a5b-4c6d-9e7f-0a1b2c3d4e5f"
      ]
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when nothing was accepted (`conflict`, `queue_full`,\n`nothing_to_scan`, `nothing_to_acquire`).",
      "example": "nothing_to_scan"
    }
  }
}
```

Status `400`: The body names no valid target. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid",
  "job_id": "",
  "message": "the account named is blank",
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

Status `403`: The caller may not scan these accounts. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "job_id": "",
  "message": "this user may not scan the library",
  "ok": false
}
```

Schema:

```json
{
  "anyOf": [
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

Status `409`: A scan is already pending or running, or no node can run it.

```json
{
  "error": "conflict",
  "job_id": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
  "job_ids": [
    "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
  ],
  "message": "scan already pending or running",
  "ok": false
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "Every planned job is already pending or running; `error` is `conflict`.",
      "required": [
        "ok",
        "message",
        "job_id",
        "job_ids"
      ],
      "properties": {
        "ok": {
          "type": "boolean",
          "description": "True when at least one job was accepted.",
          "example": true
        },
        "message": {
          "type": "string",
          "description": "Operator-facing status text.",
          "example": "scan planned: 2 accepted, 0 already pending or running, 0 not admitted (queue full)"
        },
        "job_id": {
          "type": "string",
          "description": "First accepted (or, when none was, already queued) job id.",
          "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
        },
        "job_ids": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Every accepted or already-queued job id, in plan order.",
          "example": [
            "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
            "scan-8c1d2e3f-4a5b-4c6d-9e7f-0a1b2c3d4e5f"
          ]
        },
        "error": {
          "type": [
            "string",
            "null"
          ],
          "description": "Machine slug when nothing was accepted (`conflict`, `queue_full`,\n`nothing_to_scan`, `nothing_to_acquire`).",
          "example": "nothing_to_scan"
        }
      }
    },
    {
      "type": "object",
      "description": "No node runs every plugin instance a job requires; `error` is `unroutable` and `message` names the instances.",
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
  ],
  "description": "`POST /api/library/scan` and `POST /api/library/acquire` `409` body: the two conflicts a plan can hit."
}
```

Status `422`: No account matched. As `text/plain`, when the JSON does not fit the body.

```json
{
  "error": "nothing_to_scan",
  "job_id": "",
  "job_ids": [],
  "message": "no scan-enabled store account to scan",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Result of a planned scan or acquire: [`ActionResponse`] plus every job id.",
  "required": [
    "ok",
    "message",
    "job_id",
    "job_ids"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when at least one job was accepted.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan planned: 2 accepted, 0 already pending or running, 0 not admitted (queue full)"
    },
    "job_id": {
      "type": "string",
      "description": "First accepted (or, when none was, already queued) job id.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "job_ids": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Every accepted or already-queued job id, in plan order.",
      "example": [
        "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
        "scan-8c1d2e3f-4a5b-4c6d-9e7f-0a1b2c3d4e5f"
      ]
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when nothing was accepted (`conflict`, `queue_full`,\n`nothing_to_scan`, `nothing_to_acquire`).",
      "example": "nothing_to_scan"
    }
  }
}
```

Status `429`: The job queue is full.

```json
{
  "error": "queue_full",
  "job_id": "",
  "job_ids": [],
  "message": "job queue is full",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Result of a planned scan or acquire: [`ActionResponse`] plus every job id.",
  "required": [
    "ok",
    "message",
    "job_id",
    "job_ids"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when at least one job was accepted.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan planned: 2 accepted, 0 already pending or running, 0 not admitted (queue full)"
    },
    "job_id": {
      "type": "string",
      "description": "First accepted (or, when none was, already queued) job id.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "job_ids": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Every accepted or already-queued job id, in plan order.",
      "example": [
        "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
        "scan-8c1d2e3f-4a5b-4c6d-9e7f-0a1b2c3d4e5f"
      ]
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when nothing was accepted (`conflict`, `queue_full`,\n`nothing_to_scan`, `nothing_to_acquire`).",
      "example": "nothing_to_scan"
    }
  }
}
```

Status `500`: The jobs could not be queued.

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

## `POST /api/library/acquire`

Plans and enqueues acquire jobs; returns the accepted job ids.

The acquire planner makes one job per title id, and limits a user who is
not an Owner or Administrator to the accounts they own (one job per
account when no title or account is named). `job_id` is the first
accepted (or already queued) job and `job_ids` lists every one.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

Optional.

`application/json`: [`AcquireRequestBody`](schemas.md#acquirerequestbody) or null

- `asin` *string or null* — Audible ASIN filter; first of uuid/asin/isbn/product_id wins.
- `uuid` *string or null* — Library title UUID filter.
- `isbn` *string or null* — ISBN filter for non-Audible stores.
- `product_id` *string or null* — Store product id filter.
- `account` *string or null* — Restrict acquire to this store account.
- `plugin_instance_id` *string or null* — When set, fetch through this deployed plugin instance.
- `titles` *array of string* — More title ids; each (with the single filter above) becomes its own job.
- `editions` *array of string* — Editions to acquire (`id` of an edition in a work's detail). Each becomes one job for a title that sells it in an account the caller may acquire for (in `account` when that is set); the edition's titles share its files.
- `force` *boolean* — Re-download titles already acquired.
- `pdf_only` *boolean* — Fetch only each title's companion PDF.

### Responses

- `200` — The acquire jobs were accepted. Body: [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when at least one job was accepted.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — First accepted (or, when none was, already queued) job id.
  - `job_ids` *array of string*, **required** — Every accepted or already-queued job id, in plan order.
  - `error` *string or null* — Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`).
- `400` — The body names no valid title or edition. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`); string (`text/plain`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not acquire for these accounts. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ActionResponse`](schemas.md#actionresponse) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `409` — The same acquire is already pending or running, or no node can run it. Body: [`PlanConflict`](schemas.md#planconflict) (`application/json`)
- `422` — No account of the caller's can acquire. As `text/plain`, when the JSON does not fit the body. Body: [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`); string (`text/plain`)
  - `ok` *boolean*, **required** — True when at least one job was accepted.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — First accepted (or, when none was, already queued) job id.
  - `job_ids` *array of string*, **required** — Every accepted or already-queued job id, in plan order.
  - `error` *string or null* — Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`).
- `429` — The job queue is full. Body: [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when at least one job was accepted.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — First accepted (or, when none was, already queued) job id.
  - `job_ids` *array of string*, **required** — Every accepted or already-queued job id, in plan order.
  - `error` *string or null* — Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`).
- `500` — The jobs could not be queued. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
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
$ curl 'http://127.0.0.1:8787/api/library/acquire' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/acquire' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/library/acquire",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/acquire", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/library/acquire")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The acquire jobs were accepted.

```json
{
  "job_id": "acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "job_ids": [
    "acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
  ],
  "message": "acquire job acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b accepted",
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Result of a planned scan or acquire: [`ActionResponse`] plus every job id.",
  "required": [
    "ok",
    "message",
    "job_id",
    "job_ids"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when at least one job was accepted.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan planned: 2 accepted, 0 already pending or running, 0 not admitted (queue full)"
    },
    "job_id": {
      "type": "string",
      "description": "First accepted (or, when none was, already queued) job id.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "job_ids": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Every accepted or already-queued job id, in plan order.",
      "example": [
        "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
        "scan-8c1d2e3f-4a5b-4c6d-9e7f-0a1b2c3d4e5f"
      ]
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when nothing was accepted (`conflict`, `queue_full`,\n`nothing_to_scan`, `nothing_to_acquire`).",
      "example": "nothing_to_scan"
    }
  }
}
```

Status `400`: The body names no valid title or edition. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid",
  "job_id": "",
  "message": "every title id given is blank",
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

Status `403`: The caller may not acquire for these accounts. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "job_id": "",
  "message": "that title is not in a store account linked to this user",
  "ok": false
}
```

Schema:

```json
{
  "anyOf": [
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

Status `409`: The same acquire is already pending or running, or no node can run it.

```json
{
  "error": "conflict",
  "job_id": "acquire-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
  "job_ids": [
    "acquire-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
  ],
  "message": "acquire already pending or running",
  "ok": false
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "Every planned job is already pending or running; `error` is `conflict`.",
      "required": [
        "ok",
        "message",
        "job_id",
        "job_ids"
      ],
      "properties": {
        "ok": {
          "type": "boolean",
          "description": "True when at least one job was accepted.",
          "example": true
        },
        "message": {
          "type": "string",
          "description": "Operator-facing status text.",
          "example": "scan planned: 2 accepted, 0 already pending or running, 0 not admitted (queue full)"
        },
        "job_id": {
          "type": "string",
          "description": "First accepted (or, when none was, already queued) job id.",
          "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
        },
        "job_ids": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Every accepted or already-queued job id, in plan order.",
          "example": [
            "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
            "scan-8c1d2e3f-4a5b-4c6d-9e7f-0a1b2c3d4e5f"
          ]
        },
        "error": {
          "type": [
            "string",
            "null"
          ],
          "description": "Machine slug when nothing was accepted (`conflict`, `queue_full`,\n`nothing_to_scan`, `nothing_to_acquire`).",
          "example": "nothing_to_scan"
        }
      }
    },
    {
      "type": "object",
      "description": "No node runs every plugin instance a job requires; `error` is `unroutable` and `message` names the instances.",
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
  ],
  "description": "`POST /api/library/scan` and `POST /api/library/acquire` `409` body: the two conflicts a plan can hit."
}
```

Status `422`: No account of the caller's can acquire. As `text/plain`, when the JSON does not fit the body.

```json
{
  "error": "nothing_to_acquire",
  "job_id": "",
  "job_ids": [],
  "message": "no store account of yours to acquire from",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Result of a planned scan or acquire: [`ActionResponse`] plus every job id.",
  "required": [
    "ok",
    "message",
    "job_id",
    "job_ids"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when at least one job was accepted.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan planned: 2 accepted, 0 already pending or running, 0 not admitted (queue full)"
    },
    "job_id": {
      "type": "string",
      "description": "First accepted (or, when none was, already queued) job id.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "job_ids": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Every accepted or already-queued job id, in plan order.",
      "example": [
        "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
        "scan-8c1d2e3f-4a5b-4c6d-9e7f-0a1b2c3d4e5f"
      ]
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when nothing was accepted (`conflict`, `queue_full`,\n`nothing_to_scan`, `nothing_to_acquire`).",
      "example": "nothing_to_scan"
    }
  }
}
```

Status `429`: The job queue is full.

```json
{
  "error": "queue_full",
  "job_id": "",
  "job_ids": [],
  "message": "job queue is full",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Result of a planned scan or acquire: [`ActionResponse`] plus every job id.",
  "required": [
    "ok",
    "message",
    "job_id",
    "job_ids"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when at least one job was accepted.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan planned: 2 accepted, 0 already pending or running, 0 not admitted (queue full)"
    },
    "job_id": {
      "type": "string",
      "description": "First accepted (or, when none was, already queued) job id.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "job_ids": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Every accepted or already-queued job id, in plan order.",
      "example": [
        "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
        "scan-8c1d2e3f-4a5b-4c6d-9e7f-0a1b2c3d4e5f"
      ]
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when nothing was accepted (`conflict`, `queue_full`,\n`nothing_to_scan`, `nothing_to_acquire`).",
      "example": "nothing_to_scan"
    }
  }
}
```

Status `500`: The jobs could not be queued.

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

## `GET /api/library/works`

Lists the library's works, one page at a time.

Each work appears once however many editions, stores and accounts hold
it; a podcast is listed as its show. Pass `next_cursor` back as
`cursor` (with the same `sort`) for the next page. With `q`, works that
match come best first; pass `next_cursor` back with the same `q`.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `q` (query) *string* — Search text (Lucene-style); matches titles, authors, narrators, series and store ids. Works then come best match first and `sort` is not used.
- `sort` (query) *one of `title`, `author`, `added`, `series`, `purchased`, `length`* — Order of the page (default `title`).
- `media_kind` (query) *one of `audio`, `ebook`, `print`* — Only works with an edition on this carrier.
- `kind` (query) *one of `book`, `podcast`* — Only works of this kind.
- `status` (query) *one of `not_acquired`, `queued`, `downloading`, `acquired`, `error`* — Only works whose furthest acquire state is this one.
- `account` (query) *string* — Only works held by this store account id.
- `facet` (query) *one of `author`, `narrator`, `series`, `genre`, `publisher`, `source`, `tag`, `progress`* — Only works with `value` under this facet. Values match ignoring case; `tag` and `progress` read the caller's own state.
- `value` (query) *string* — The value `facet` must have, as `GET /api/library/works/facets/{facet}` lists it (or `finished` / `not_finished` for `progress`).
- `cursor` (query) *string* — `next_cursor` of the previous page, under the same `sort` (or the same `q`).
- `limit` (query) *integer (int64)* — Page size, clamped to 1–200 (default 40).

### Responses

- `200` — A page of works. Body: [`WorksPage`](schemas.md#workspage) (`application/json`)
  - `works` *array of [`WorkSummaryJson`](schemas.md#worksummaryjson)*, **required** — The works, in the requested order.
  - `next_cursor` *string or null* — Pass as `cursor` for the next page; absent on the last page.
- `400` — A filter, the query or the cursor is invalid. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — A search (`q`) arrived before the node's works search index was first built; that answer carries no `Retry-After`. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/works' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/works' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/works",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/works", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/works")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A page of works.

```json
{
  "works": [
    {
      "id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b",
      "kind": "book",
      "title": "The Left Hand of Darkness",
      "authors": "Ursula K. Le Guin",
      "series": "Hainish Cycle",
      "series_position": "4",
      "cover_edition_id": "7c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f",
      "cover_path": "/api/covers/eyJ1IjoiaHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGcifQ",
      "media_kinds": [
        "audio"
      ],
      "acquire_status": "not_acquired",
      "edition_count": 2,
      "holding_count": 3,
      "added_at": "2026-11-14T09:30:00.000000Z",
      "length_minutes": 1212,
      "purchased_at": "2024-03-02T18:04:11Z"
    }
  ],
  "next_cursor": "eyJzb3J0IjoidGl0bGUiLCJrZXkiOiJsZWZ0IGhhbmQiLCJ3b3JrX2lkIjoiMGI4ZiJ9"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A page of `GET /api/library/works`.",
  "required": [
    "works"
  ],
  "properties": {
    "works": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One work in a page of `GET /api/library/works`.",
        "required": [
          "id",
          "kind",
          "title",
          "media_kinds",
          "acquire_status",
          "edition_count",
          "holding_count",
          "added_at"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Work id.",
            "example": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"
          },
          "kind": {
            "type": "string",
            "description": "Book or podcast.",
            "enum": [
              "book",
              "podcast"
            ]
          },
          "title": {
            "type": "string",
            "description": "Display title.",
            "example": "The Left Hand of Darkness"
          },
          "authors": {
            "type": [
              "string",
              "null"
            ],
            "description": "Authors (hosts for a podcast), joined for display.",
            "example": "Ursula K. Le Guin"
          },
          "series": {
            "type": [
              "string",
              "null"
            ],
            "description": "First series the work belongs to.",
            "example": "Hainish Cycle"
          },
          "series_position": {
            "type": [
              "string",
              "null"
            ],
            "description": "Position in `series`, as the store gave it.",
            "example": "4"
          },
          "cover_edition_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "The edition whose cover the list shows.",
            "example": "7c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f"
          },
          "cover_path": {
            "type": [
              "string",
              "null"
            ],
            "description": "Same-origin path of that cover, when it has one.",
            "example": "/api/covers/eyJ1IjoiaHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGcifQ"
          },
          "media_kinds": {
            "type": "array",
            "items": {
              "type": "string",
              "description": "Carrier of an edition (`editions.media_kind`).",
              "enum": [
                "audio",
                "ebook",
                "print"
              ]
            },
            "description": "Carriers of the work's editions."
          },
          "acquire_status": {
            "type": "string",
            "description": "The furthest acquire state any edition reached.",
            "enum": [
              "not_acquired",
              "queued",
              "downloading",
              "acquired",
              "error"
            ]
          },
          "edition_count": {
            "type": "integer",
            "format": "int64",
            "description": "How many editions the work has.",
            "example": 2,
            "minimum": 0
          },
          "holding_count": {
            "type": "integer",
            "format": "int64",
            "description": "How many holdings the work's listings have.",
            "example": 3,
            "minimum": 0
          },
          "added_at": {
            "type": "string",
            "description": "RFC 3339 time of the newest holding.",
            "example": "2026-11-14T09:30:00.000000Z"
          },
          "length_minutes": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Runtime in minutes of the longest held edition, when known.",
            "example": 1212,
            "minimum": 0
          },
          "purchased_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "RFC 3339 time of the newest purchase a store reported.",
            "example": "2024-03-02T18:04:11Z"
          }
        }
      },
      "description": "The works, in the requested order."
    },
    "next_cursor": {
      "type": [
        "string",
        "null"
      ],
      "description": "Pass as `cursor` for the next page; absent on the last page.",
      "example": "eyJzb3J0IjoidGl0bGUiLCJrZXkiOiJsZWZ0IGhhbmQiLCJ3b3JrX2lkIjoiMGI4ZiJ9"
    }
  }
}
```

Status `400`: A filter, the query or the cursor is invalid.

```json
{
  "error": "invalid page cursor"
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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

```json
{
  "error": "clerk_sign_in_unavailable",
  "message": "This library does not accept clerks on this address.",
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

Status `500`: The library could not be read.

```json
{
  "error": "the library could not be read"
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

Status `503`: A search (`q`) arrived before the node's works search index was first built; that answer carries no `Retry-After`. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "the search index is being built; try again shortly"
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

## `GET /api/library/works/{id}`

Reads one work with its editions, listings, holdings and the caller's
own state.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Work id (a podcast's show).

### Responses

- `200` — The work. Body: [`WorkDetailJson`](schemas.md#workdetailjson) (`application/json`)
  - `id` *string*, **required** — Work id.
  - `kind` *[`WorkKindJson`](schemas.md#workkindjson)*, **required** — Book or podcast.
  - `title` *string*, **required** — Display title.
  - `description` *string or null* — Blurb or synopsis (may contain HTML).
  - `authors` *array of string*, **required** — Authors (hosts for a podcast), in billing order.
  - `series` *string or null* — First series the work belongs to.
  - `series_position` *string or null* — Position in `series`.
  - `editions` *array of [`EditionJson`](schemas.md#editionjson)*, **required** — Editions; a show's episodes' editions included.
  - `state` *[`WorkStateJson`](schemas.md#workstatejson) or null* — The caller's own state; absent for the operator or when the caller recorded nothing.
  - `can_edit_state` *boolean*, **required** — Whether the caller can keep their own state for the work (`PUT /api/library/works/{id}/state`): false when the session has no library user, such as the operator token.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such work. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/works/<id>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/works/<id>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/works/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/works/<id>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/works/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The work.

```json
{
  "id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b",
  "kind": "book",
  "title": "The Left Hand of Darkness",
  "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
  "authors": [
    "Ursula K. Le Guin"
  ],
  "series": "Hainish Cycle",
  "series_position": "4",
  "editions": [
    {
      "id": "7c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f",
      "work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b",
      "expression_id": "5a6b7c8d-9e0f-4a1b-8c2d-3e4f5a6b7c8d",
      "expression_kind": "text",
      "language": "en",
      "abridgement": "unknown",
      "media_kind": "audio",
      "title": "The Left Hand of Darkness",
      "subtitle": "A Hainish Novel",
      "published_at": "2009-11-25",
      "duration_minutes": 682,
      "cover_path": "/api/covers/eyJ1IjoiaHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGcifQ",
      "acquire_status": "not_acquired",
      "listings": [
        {
          "id": "3c4d5e6f-7081-4923-8a4b-5c6d7e8f9012",
          "plugin_key": "audible",
          "marketplace": "us",
          "product_id": "B002V0QK4C",
          "holdings": []
        }
      ]
    }
  ],
  "state": {
    "is_finished": true,
    "rating_overall": 4.5,
    "rating_performance": 4.0,
    "rating_story": 5.0,
    "tags": "classics"
  },
  "can_edit_state": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/library/works/{id}` answer.",
  "required": [
    "id",
    "kind",
    "title",
    "authors",
    "editions",
    "can_edit_state"
  ],
  "properties": {
    "id": {
      "type": "string",
      "description": "Work id.",
      "example": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"
    },
    "kind": {
      "type": "string",
      "description": "Book or podcast.",
      "enum": [
        "book",
        "podcast"
      ]
    },
    "title": {
      "type": "string",
      "description": "Display title.",
      "example": "The Left Hand of Darkness"
    },
    "description": {
      "type": [
        "string",
        "null"
      ],
      "description": "Blurb or synopsis (may contain HTML).",
      "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
    },
    "authors": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Authors (hosts for a podcast), in billing order.",
      "example": [
        "Ursula K. Le Guin"
      ]
    },
    "series": {
      "type": [
        "string",
        "null"
      ],
      "description": "First series the work belongs to.",
      "example": "Hainish Cycle"
    },
    "series_position": {
      "type": [
        "string",
        "null"
      ],
      "description": "Position in `series`.",
      "example": "4"
    },
    "editions": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One edition of a work.",
        "required": [
          "id",
          "work_id",
          "expression_id",
          "expression_kind",
          "abridgement",
          "media_kind",
          "title",
          "acquire_status",
          "listings"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Edition id.",
            "example": "7c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f"
          },
          "work_id": {
            "type": "string",
            "description": "The work it realises: this work, or one of a show's episodes.",
            "example": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"
          },
          "expression_id": {
            "type": "string",
            "description": "The expression it carries.",
            "example": "5a6b7c8d-9e0f-4a1b-8c2d-3e4f5a6b7c8d"
          },
          "expression_kind": {
            "type": "string",
            "description": "How the expression realises the work.",
            "enum": [
              "text",
              "narration",
              "dramatisation",
              "graphic",
              "original_audio"
            ]
          },
          "language": {
            "type": [
              "string",
              "null"
            ],
            "description": "The expression's language tag, when known.",
            "example": "en"
          },
          "abridgement": {
            "type": "string",
            "description": "Whether the expression is abridged.",
            "enum": [
              "unknown",
              "unabridged",
              "abridged"
            ]
          },
          "media_kind": {
            "type": "string",
            "description": "Carrier of the edition.",
            "enum": [
              "audio",
              "ebook",
              "print"
            ]
          },
          "title": {
            "type": "string",
            "description": "The edition's title.",
            "example": "The Left Hand of Darkness"
          },
          "subtitle": {
            "type": [
              "string",
              "null"
            ],
            "description": "The edition's subtitle.",
            "example": "A Hainish Novel"
          },
          "published_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publication date as the store gave it.",
            "example": "2009-11-25"
          },
          "duration_minutes": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Running time in minutes, for audio.",
            "example": 682
          },
          "cover_path": {
            "type": [
              "string",
              "null"
            ],
            "description": "Same-origin path of the edition's cover, when it has one.",
            "example": "/api/covers/eyJ1IjoiaHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGcifQ"
          },
          "acquire_status": {
            "type": "string",
            "description": "The edition's acquire state.",
            "enum": [
              "not_acquired",
              "queued",
              "downloading",
              "acquired",
              "error"
            ]
          },
          "listings": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One store listing of an edition.",
              "required": [
                "id",
                "plugin_key",
                "marketplace",
                "product_id",
                "holdings"
              ],
              "properties": {
                "id": {
                  "type": "string",
                  "description": "Listing id.",
                  "example": "3c4d5e6f-7081-4923-8a4b-5c6d7e8f9012"
                },
                "plugin_key": {
                  "type": "string",
                  "description": "The storefront plugin that lists it.",
                  "example": "audible"
                },
                "marketplace": {
                  "type": "string",
                  "description": "Store marketplace; empty when the store has one.",
                  "example": "us"
                },
                "product_id": {
                  "type": "string",
                  "description": "The store's product id.",
                  "example": "B002V0QK4C"
                },
                "url": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "The product page, when known.",
                  "example": "https://www.audible.com/pd/B002V0QK4C"
                },
                "holdings": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "One holding of a listing.",
                    "required": [
                      "id",
                      "holder",
                      "entitlement",
                      "added_at"
                    ],
                    "properties": {
                      "id": {
                        "type": "string",
                        "description": "Holding id.",
                        "example": "4d5e6f70-8192-4a3b-9c4d-5e6f708192a3"
                      },
                      "holder": {
                        "oneOf": [
                          {
                            "type": "object",
                            "description": "A store account.",
                            "required": [
                              "plugin_key",
                              "account_id",
                              "kind"
                            ],
                            "properties": {
                              "plugin_key": {
                                "type": "string",
                                "description": "The account's storefront plugin.",
                                "example": "audible"
                              },
                              "account_id": {
                                "type": "string",
                                "description": "The account id at that plugin.",
                                "example": "acct-7f3a"
                              },
                              "kind": {
                                "type": "string",
                                "enum": [
                                  "account"
                                ]
                              }
                            }
                          },
                          {
                            "type": "object",
                            "description": "A branch library's node.",
                            "required": [
                              "node_id",
                              "kind"
                            ],
                            "properties": {
                              "node_id": {
                                "type": "string",
                                "description": "The node id.",
                                "example": "9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e"
                              },
                              "kind": {
                                "type": "string",
                                "enum": [
                                  "node"
                                ]
                              }
                            }
                          }
                        ],
                        "description": "Who holds the listing."
                      },
                      "entitlement": {
                        "type": "string",
                        "description": "On what terms.",
                        "enum": [
                          "purchase",
                          "plus",
                          "loan",
                          "free"
                        ]
                      },
                      "purchased_at": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "RFC 3339 purchase time, when the store reported one.",
                        "example": "2024-03-02T18:04:11Z"
                      },
                      "added_at": {
                        "type": "string",
                        "description": "RFC 3339 time the holding was first recorded.",
                        "example": "2026-11-14T09:30:00.000000Z"
                      }
                    }
                  },
                  "description": "The listing's holdings."
                }
              }
            },
            "description": "The edition's store listings."
          }
        }
      },
      "description": "Editions; a show's episodes' editions included."
    },
    "state": {
      "oneOf": [
        {
          "type": "object",
          "description": "The caller's own state; absent for the operator or when the caller\nrecorded nothing.",
          "required": [
            "is_finished"
          ],
          "properties": {
            "is_finished": {
              "type": "boolean",
              "description": "Whether the caller finished the work.",
              "example": true
            },
            "rating_overall": {
              "type": [
                "number",
                "null"
              ],
              "format": "double",
              "description": "Overall rating, 0 to 5.",
              "example": 4.5
            },
            "rating_performance": {
              "type": [
                "number",
                "null"
              ],
              "format": "double",
              "description": "Narration rating, 0 to 5.",
              "example": 4.0
            },
            "rating_story": {
              "type": [
                "number",
                "null"
              ],
              "format": "double",
              "description": "Story rating, 0 to 5.",
              "example": 5.0
            },
            "tags": {
              "type": [
                "string",
                "null"
              ],
              "description": "The caller's tags, as one serialized string.",
              "example": "classics"
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "can_edit_state": {
      "type": "boolean",
      "description": "Whether the caller can keep their own state for the work\n(`PUT /api/library/works/{id}/state`): false when the session has no\nlibrary user, such as the operator token.",
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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

```json
{
  "error": "clerk_sign_in_unavailable",
  "message": "This library does not accept clerks on this address.",
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

Status `404`: No such work.

```json
{
  "error": "work 0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b not found"
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

Status `500`: The library could not be read.

```json
{
  "error": "the library could not be read"
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

## `PUT /api/library/works/{id}/state`

Changes the caller's own state for a work: finished, ratings and tags.

The state belongs to the signed-in library user, so members who share
a storefront account each keep their own. Fields left out keep their
value.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Work id (a podcast's show).

### Request body

`application/json`: [`WorkStateEdit`](schemas.md#workstateedit)

- `is_finished` *boolean or null* — Whether the caller finished the work.
- `rating_overall` *number (double) or null* — Overall rating, 0 to 5; `null` clears it.
- `rating_performance` *number (double) or null* — Narration rating, 0 to 5; `null` clears it.
- `rating_story` *number (double) or null* — Story rating, 0 to 5; `null` clears it.
- `tags` *string or null* — Tags separated by whitespace (at most 32, each at most 64 bytes, 1024 bytes in all); `null` or an empty string clears them.

### Responses

- `200` — The caller's state after the change. Body: [`WorkStateJson`](schemas.md#workstatejson) (`application/json`)
  - `is_finished` *boolean*, **required** — Whether the caller finished the work.
  - `rating_overall` *number (double) or null* — Overall rating, 0 to 5.
  - `rating_performance` *number (double) or null* — Narration rating, 0 to 5.
  - `rating_story` *number (double) or null* — Story rating, 0 to 5.
  - `tags` *string or null* — The caller's tags, as one serialized string.
- `400` — A rating is outside 0 to 5, or the tags are too long or too many. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session has no library user to keep the state for. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such work. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The state could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/works/<id>/state' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/works/<id>/state' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/library/works/<id>/state",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/works/<id>/state", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/library/works/<id>/state")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's state after the change.

```json
{
  "is_finished": true,
  "rating_overall": 4.5,
  "rating_performance": 4.0,
  "rating_story": 5.0,
  "tags": "classics"
}
```

Schema:

```json
{
  "type": "object",
  "description": "The caller's own state for a work.",
  "required": [
    "is_finished"
  ],
  "properties": {
    "is_finished": {
      "type": "boolean",
      "description": "Whether the caller finished the work.",
      "example": true
    },
    "rating_overall": {
      "type": [
        "number",
        "null"
      ],
      "format": "double",
      "description": "Overall rating, 0 to 5.",
      "example": 4.5
    },
    "rating_performance": {
      "type": [
        "number",
        "null"
      ],
      "format": "double",
      "description": "Narration rating, 0 to 5.",
      "example": 4.0
    },
    "rating_story": {
      "type": [
        "number",
        "null"
      ],
      "format": "double",
      "description": "Story rating, 0 to 5.",
      "example": 5.0
    },
    "tags": {
      "type": [
        "string",
        "null"
      ],
      "description": "The caller's tags, as one serialized string.",
      "example": "classics"
    }
  }
}
```

Status `400`: A rating is outside 0 to 5, or the tags are too long or too many. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "rating_overall must be from 0 to 5"
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

Status `403`: The session has no library user to keep the state for. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "clerk_sign_in_unavailable",
  "message": "This library does not accept clerks on this address.",
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
          "anyOf": [
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

Status `404`: No such work.

```json
{
  "error": "work 0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b not found"
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
Failed to deserialize the JSON body into the target type: tags: invalid type: integer `1`, expected a string at line 1 column 9
```

Schema:

A `text/plain` body: a string.

Status `500`: The state could not be written.

```json
{
  "error": "the state could not be written"
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

## `GET /api/library/works/facets/{facet}`

Lists the values a facet has across the library's held works, with
how many works have each, for the library page's filter menus. `tag`
lists the caller's own tags (none for an operator token).

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `facet` (path) *one of `author`, `narrator`, `series`, `genre`, `publisher`, `source`, `tag`, `progress`*, **required** — The facet; `progress` has no value list.
- `cursor` (query) *string* — `next_cursor` of the previous page of the same facet.
- `limit` (query) *integer (int64)* — Page size, clamped to 1–500 (default 200).

### Responses

- `200` — A page of values. Body: [`FacetValuesJson`](schemas.md#facetvaluesjson) (`application/json`)
  - `values` *array of [`FacetValueJson`](schemas.md#facetvaluejson)*, **required** — The values, in order of their lower-cased spelling.
  - `next_cursor` *string or null* — Pass as `cursor` for the next page; absent on the last page.
- `400` — `progress` was asked for, or the cursor is invalid. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/works/facets/<facet>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/works/facets/<facet>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/works/facets/<facet>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/works/facets/<facet>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/works/facets/<facet>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A page of values.

```json
{
  "values": [
    {
      "value": "Science Fiction",
      "works": 12
    }
  ],
  "next_cursor": "c2NpZW5jZSBmaWN0aW9u"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A page of `GET /api/library/works/facets/{facet}`.",
  "required": [
    "values"
  ],
  "properties": {
    "values": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One value of a facet.",
        "required": [
          "value",
          "works"
        ],
        "properties": {
          "value": {
            "type": "string",
            "description": "The value, as `value` of `GET /api/library/works` takes it.",
            "example": "Science Fiction"
          },
          "works": {
            "type": "integer",
            "format": "int64",
            "description": "How many held works have it.",
            "example": 12,
            "minimum": 0
          }
        }
      },
      "description": "The values, in order of their lower-cased spelling."
    },
    "next_cursor": {
      "type": [
        "string",
        "null"
      ],
      "description": "Pass as `cursor` for the next page; absent on the last page.",
      "example": "c2NpZW5jZSBmaWN0aW9u"
    }
  }
}
```

Status `400`: `progress` was asked for, or the cursor is invalid.

```json
{
  "error": "invalid page cursor"
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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

```json
{
  "error": "clerk_sign_in_unavailable",
  "message": "This library does not accept clerks on this address.",
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

Status `500`: The library could not be read.

```json
{
  "error": "the library could not be read"
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

## `GET /api/library/editions/{id}/cover`

Serves the JPEG cover written beside an edition's acquired file on this
node's local destination.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Edition id (an `id` in a work's `editions`).

### Responses

- `200` — The cover. Body: binary (`image/jpeg`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such edition, or no cover of it on this node: it is not acquired, this node runs no local destination, or the acquire wrote no cover. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library or the cover could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/cover' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/cover' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/editions/<id>/cover",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/cover", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/editions/<id>/cover")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The cover. The body is binary (`image/jpeg`).

Schema:

A `image/jpeg` body: binary bytes.

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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

```json
{
  "error": "clerk_sign_in_unavailable",
  "message": "This library does not accept clerks on this address.",
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

Status `404`: No such edition, or no cover of it on this node: it is not acquired, this node runs no local destination, or the acquire wrote no cover.

```json
{
  "error": "edition 5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d has no cover on this node"
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

Status `500`: The library or the cover could not be read.

```json
{
  "error": "the cover could not be read"
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
