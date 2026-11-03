# Library

Books in the library, scans and acquires.

Back to the [web API reference](README.md).

## `POST /api/library/scan`

Plans and enqueues a library scan; returns the accepted job ids.

The scan planner splits a scan of every account into one job per
scan-enabled account for operators, Owners and Administrators, and
limits anyone else to the accounts they own. `job_id` is the first
accepted (or already queued) job and `job_ids` lists every one.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

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
- `503` — Job admission is paused. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
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

Status `503`: Job admission is paused.

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

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

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
- `force` *boolean* — Re-download titles already acquired.
- `pdf_only` *boolean* — Fetch only each title's companion PDF.

### Responses

- `200` — The acquire jobs were accepted. Body: [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when at least one job was accepted.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — First accepted (or, when none was, already queued) job id.
  - `job_ids` *array of string*, **required** — Every accepted or already-queued job id, in plan order.
  - `error` *string or null* — Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`).
- `400` — The body names no valid title. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`); string (`text/plain`)
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
- `503` — Job admission is paused. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
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

Status `400`: The body names no valid title. As `text/plain`, when the body is not syntactically valid JSON.

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

Status `503`: Job admission is paused.

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

## `GET /api/library/books`

Lists or searches library titles with optional account/status filters and pagination.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `account` (query) *string* — Restrict results to this store account id.
- `status` (query) *string* — Optional `AcquireStatus` wire-string filter.
- `q` (query) *string* — Full-text search query; empty skips the search index.
- `limit` (query) *integer* — Page size, clamped to 1–500 (default 40).
- `offset` (query) *integer* — Number of matching titles to skip.

### Responses

- `200` — A page of titles. Body: [`BooksResponse`](schemas.md#booksresponse) (`application/json`)
  - `books` *array of [`BookRecord`](schemas.md#bookrecord)*, **required** — Current page of title records.
  - `total` *integer*, **required** — Match count before pagination. At least 0.
  - `limit` *integer*, **required** — Applied page size. At least 0.
  - `offset` *integer*, **required** — Applied skip count. At least 0.
- `400` — A filter is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The library could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/books' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/books' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/books",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/books", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/books")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A page of titles.

```json
{
  "books": [
    {
      "id": 412,
      "uuid": "3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31",
      "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro",
      "source": "libro",
      "account_id": "reader@example.com",
      "product_id": "9780441478125",
      "asin": "B002V0QK4C",
      "isbn": "9780441478125",
      "marketplace": "us",
      "title": "The Left Hand of Darkness",
      "authors": "Ursula K. Le Guin",
      "narrators": "George Guidall",
      "series": "Hainish Cycle",
      "series_index": "4",
      "series_asin": "B07ZRJZ4PW",
      "acquire_status": "not_acquired",
      "storage_key": "Ursula K. Le Guin/The Left Hand of Darkness/The Left Hand of Darkness.m4b",
      "error_message": "download failed: HTTP 403 from the store",
      "purchased_at": "2026-03-14T18:02:11Z",
      "tags": "book club, sci-fi",
      "rating_overall": 4.5,
      "rating_performance": 4.7,
      "rating_story": 4.3,
      "is_finished": false,
      "pdf_status": "not_acquired",
      "pdf_storage_key": "Ursula K. Le Guin/The Left Hand of Darkness/The Left Hand of Darkness.pdf",
      "publisher": "Blackstone Publishing",
      "length_minutes": 560,
      "is_abridged": false,
      "content_kind": "book",
      "categories": "Science Fiction & Fantasy > Science Fiction",
      "subtitle": "A Hainish Novel",
      "published_at": "2005-06-21T00:00:00Z",
      "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
      "language": "en",
      "cover_url": "https://images.example.com/covers/9780441478125.jpg",
      "subjects": "Androgyny; Interplanetary voyages; Fiction",
      "enrich_source": "openlibrary",
      "enrich_confidence": 0.92,
      "enrich_updated_at": "2026-10-29T10:13:05Z",
      "created_at": "2026-10-29T10:12:00Z",
      "updated_at": "2026-10-29T10:14:22Z"
    }
  ],
  "total": 412,
  "limit": 50,
  "offset": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "Paginated library listing.",
  "required": [
    "books",
    "total",
    "limit",
    "offset"
  ],
  "properties": {
    "books": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Book / library item (one ownership row per store product per account).",
        "required": [
          "id",
          "uuid",
          "plugin_key",
          "source",
          "account_id",
          "product_id",
          "marketplace",
          "title",
          "acquire_status",
          "is_finished",
          "pdf_status",
          "is_abridged",
          "content_kind",
          "created_at",
          "updated_at"
        ],
        "properties": {
          "id": {
            "type": "integer",
            "format": "int64",
            "description": "Surrogate primary key assigned by the database.",
            "example": 412
          },
          "uuid": {
            "type": "string",
            "description": "Stable UUID for this row (API / foreign-key identity).",
            "example": "3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31"
          },
          "plugin_key": {
            "type": "string",
            "description": "Provenance-qualified PluginKey that owns the account.",
            "example": "platform:bookclerk/bookclerk-plugin-source-libro#libro"
          },
          "source": {
            "type": "string",
            "description": "Display alias of the owning plugin (`audible`, `libro`, …).",
            "example": "libro"
          },
          "account_id": {
            "type": "string",
            "description": "Store or operator account id this row belongs to.",
            "example": "reader@example.com"
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …).",
            "example": "9780441478125"
          },
          "asin": {
            "type": [
              "string",
              "null"
            ],
            "description": "Audible ASIN when known (`None` for Libro-only rows without enrichment).",
            "example": "B002V0QK4C"
          },
          "isbn": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISBN-13 when known (Libro always; Audible when the API provides it).",
            "example": "9780441478125"
          },
          "marketplace": {
            "type": "string",
            "description": "Store marketplace / locale code (for example `us`, `uk`).",
            "example": "us"
          },
          "title": {
            "type": "string",
            "description": "Display title of the work or edition.",
            "example": "The Left Hand of Darkness"
          },
          "authors": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated or JSON author list from the storefront.",
            "example": "Ursula K. Le Guin"
          },
          "narrators": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated or JSON narrator list when present.",
            "example": "George Guidall"
          },
          "series": {
            "type": [
              "string",
              "null"
            ],
            "description": "Series name when the title belongs to a series.",
            "example": "Hainish Cycle"
          },
          "series_index": {
            "type": [
              "string",
              "null"
            ],
            "description": "Position within the series (storefront string form).",
            "example": "4"
          },
          "series_asin": {
            "type": [
              "string",
              "null"
            ],
            "description": "Audible series / podcast-parent ASIN (`Series.AudibleSeriesId`).",
            "example": "B07ZRJZ4PW"
          },
          "acquire_status": {
            "type": "string",
            "description": "Download/acquire pipeline state (`not_acquired`, `queued`, …).",
            "enum": [
              "not_acquired",
              "queued",
              "downloading",
              "acquired",
              "error"
            ]
          },
          "storage_key": {
            "type": [
              "string",
              "null"
            ],
            "description": "Object-storage key for the primary audio artifact, if acquired.",
            "example": "Ursula K. Le Guin/The Left Hand of Darkness/The Left Hand of Darkness.m4b"
          },
          "error_message": {
            "type": [
              "string",
              "null"
            ],
            "description": "Last acquire/convert failure message for operators.",
            "example": "download failed: HTTP 403 from the store"
          },
          "purchased_at": {
            "type": [
              "string",
              "null"
            ],
            "format": "date-time",
            "description": "RFC 3339 purchase time from the storefront, when known.",
            "example": "2026-03-14T18:02:11Z"
          },
          "tags": {
            "type": [
              "string",
              "null"
            ],
            "description": "Operator or storefront tags (serialized string).",
            "example": "book club, sci-fi"
          },
          "rating_overall": {
            "type": [
              "number",
              "null"
            ],
            "format": "float",
            "description": "Overall user rating from the storefront, if any.",
            "example": 4.5
          },
          "rating_performance": {
            "type": [
              "number",
              "null"
            ],
            "format": "float",
            "description": "Narration/performance rating from the storefront, if any.",
            "example": 4.7
          },
          "rating_story": {
            "type": [
              "number",
              "null"
            ],
            "format": "float",
            "description": "Story rating from the storefront, if any.",
            "example": 4.3
          },
          "is_finished": {
            "type": "boolean",
            "description": "Whether the listener marked the title finished (0/1 or bool).",
            "example": false
          },
          "pdf_status": {
            "type": "string",
            "description": "Companion PDF acquire state (`not_acquired`, `acquired`, …).",
            "enum": [
              "not_acquired",
              "queued",
              "downloading",
              "acquired",
              "error"
            ]
          },
          "pdf_storage_key": {
            "type": [
              "string",
              "null"
            ],
            "description": "Object-storage key for the companion PDF, if present.",
            "example": "Ursula K. Le Guin/The Left Hand of Darkness/The Left Hand of Darkness.pdf"
          },
          "publisher": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publisher name from metadata enrichment or the storefront.",
            "example": "Blackstone Publishing"
          },
          "length_minutes": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Runtime in whole minutes when the storefront reports it.",
            "example": 560
          },
          "is_abridged": {
            "type": "boolean",
            "description": "Whether the edition is abridged (0/1 or bool).",
            "example": false
          },
          "content_kind": {
            "type": "string",
            "description": "Title kind: `book`, `episode`, `podcast`, ….",
            "example": "book"
          },
          "categories": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront category / genre path list (serialized).",
            "example": "Science Fiction & Fantasy > Science Fiction"
          },
          "subtitle": {
            "type": [
              "string",
              "null"
            ],
            "description": "Optional subtitle from bibliographic metadata.",
            "example": "A Hainish Novel"
          },
          "published_at": {
            "type": [
              "string",
              "null"
            ],
            "format": "date-time",
            "description": "Publication date string from the storefront or enrichment.",
            "example": "2005-06-21T00:00:00Z"
          },
          "description": {
            "type": [
              "string",
              "null"
            ],
            "description": "Blurb / synopsis text (may contain HTML).",
            "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
          },
          "language": {
            "type": [
              "string",
              "null"
            ],
            "description": "BCP-47 or storefront language code when known.",
            "example": "en"
          },
          "cover_url": {
            "type": [
              "string",
              "null"
            ],
            "description": "HTTPS URL for cover art when known.",
            "example": "https://images.example.com/covers/9780441478125.jpg"
          },
          "subjects": {
            "type": [
              "string",
              "null"
            ],
            "description": "Subject / topic tags (often from Open Library; `;`- or `,`-separated).",
            "example": "Androgyny; Interplanetary voyages; Fiction"
          },
          "enrich_source": {
            "type": [
              "string",
              "null"
            ],
            "description": "Plugin or catalog that last enriched bibliographic fields.",
            "example": "openlibrary"
          },
          "enrich_confidence": {
            "type": [
              "number",
              "null"
            ],
            "format": "double",
            "description": "0–1 confidence score for the last enrichment pass.",
            "example": 0.92
          },
          "enrich_updated_at": {
            "type": [
              "string",
              "null"
            ],
            "format": "date-time",
            "description": "RFC 3339 time of the last enrichment write.",
            "example": "2026-10-29T10:13:05Z"
          },
          "created_at": {
            "type": "string",
            "format": "date-time",
            "description": "RFC 3339 timestamp when the row was inserted.",
            "example": "2026-10-29T10:12:00Z"
          },
          "updated_at": {
            "type": "string",
            "format": "date-time",
            "description": "RFC 3339 timestamp when the row was last modified.",
            "example": "2026-10-29T10:14:22Z"
          }
        }
      },
      "description": "Current page of title records."
    },
    "total": {
      "type": "integer",
      "description": "Match count before pagination.",
      "example": 412,
      "minimum": 0
    },
    "limit": {
      "type": "integer",
      "description": "Applied page size.",
      "example": 50,
      "minimum": 0
    },
    "offset": {
      "type": "integer",
      "description": "Applied skip count.",
      "example": 0,
      "minimum": 0
    }
  }
}
```

Status `400`: A filter is invalid.

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

## `GET /api/library/books/{uuid}`

Returns one title by UUID, or 404 when it is absent.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `uuid` (path) *string*, **required** — Title UUID.

### Responses

- `200` — The title. Body: [`BookRecord`](schemas.md#bookrecord) (`application/json`)
  - `id` *integer (int64)*, **required** — Surrogate primary key assigned by the database.
  - `uuid` *string*, **required** — Stable UUID for this row (API / foreign-key identity).
  - `plugin_key` *string*, **required** — Provenance-qualified PluginKey that owns the account.
  - `source` *string*, **required** — Display alias of the owning plugin (`audible`, `libro`, …).
  - `account_id` *string*, **required** — Store or operator account id this row belongs to.
  - `product_id` *string*, **required** — Storefront product id (ASIN, ISBN, UUID, …).
  - `asin` *string or null* — Audible ASIN when known (`None` for Libro-only rows without enrichment).
  - `isbn` *string or null* — ISBN-13 when known (Libro always; Audible when the API provides it).
  - `marketplace` *string*, **required** — Store marketplace / locale code (for example `us`, `uk`).
  - `title` *string*, **required** — Display title of the work or edition.
  - `authors` *string or null* — Comma-separated or JSON author list from the storefront.
  - `narrators` *string or null* — Comma-separated or JSON narrator list when present.
  - `series` *string or null* — Series name when the title belongs to a series.
  - `series_index` *string or null* — Position within the series (storefront string form).
  - `series_asin` *string or null* — Audible series / podcast-parent ASIN (`Series.AudibleSeriesId`).
  - `acquire_status` *[`AcquireStatus`](schemas.md#acquirestatus)*, **required** — Download/acquire pipeline state (`not_acquired`, `queued`, …).
  - `storage_key` *string or null* — Object-storage key for the primary audio artifact, if acquired.
  - `error_message` *string or null* — Last acquire/convert failure message for operators.
  - `purchased_at` *string (date-time) or null* — RFC 3339 purchase time from the storefront, when known.
  - `tags` *string or null* — Operator or storefront tags (serialized string).
  - `rating_overall` *number (float) or null* — Overall user rating from the storefront, if any.
  - `rating_performance` *number (float) or null* — Narration/performance rating from the storefront, if any.
  - `rating_story` *number (float) or null* — Story rating from the storefront, if any.
  - `is_finished` *boolean*, **required** — Whether the listener marked the title finished (0/1 or bool).
  - `pdf_status` *[`AcquireStatus`](schemas.md#acquirestatus)*, **required** — Companion PDF acquire state (`not_acquired`, `acquired`, …).
  - `pdf_storage_key` *string or null* — Object-storage key for the companion PDF, if present.
  - `publisher` *string or null* — Publisher name from metadata enrichment or the storefront.
  - `length_minutes` *integer (int64) or null* — Runtime in whole minutes when the storefront reports it.
  - `is_abridged` *boolean*, **required** — Whether the edition is abridged (0/1 or bool).
  - `content_kind` *string*, **required** — Title kind: `book`, `episode`, `podcast`, ….
  - `categories` *string or null* — Storefront category / genre path list (serialized).
  - `subtitle` *string or null* — Optional subtitle from bibliographic metadata.
  - `published_at` *string (date-time) or null* — Publication date string from the storefront or enrichment.
  - `description` *string or null* — Blurb / synopsis text (may contain HTML).
  - `language` *string or null* — BCP-47 or storefront language code when known.
  - `cover_url` *string or null* — HTTPS URL for cover art when known.
  - `subjects` *string or null* — Subject / topic tags (often from Open Library; `;`- or `,`-separated).
  - `enrich_source` *string or null* — Plugin or catalog that last enriched bibliographic fields.
  - `enrich_confidence` *number (double) or null* — 0–1 confidence score for the last enrichment pass.
  - `enrich_updated_at` *string (date-time) or null* — RFC 3339 time of the last enrichment write.
  - `created_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was inserted.
  - `updated_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was last modified.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such title. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The library could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/books/<uuid>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/books/<uuid>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/books/<uuid>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/books/<uuid>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/books/<uuid>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The title.

```json
{
  "id": 412,
  "uuid": "3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31",
  "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro",
  "source": "libro",
  "account_id": "reader@example.com",
  "product_id": "9780441478125",
  "asin": "B002V0QK4C",
  "isbn": "9780441478125",
  "marketplace": "us",
  "title": "The Left Hand of Darkness",
  "authors": "Ursula K. Le Guin",
  "narrators": "George Guidall",
  "series": "Hainish Cycle",
  "series_index": "4",
  "series_asin": "B07ZRJZ4PW",
  "acquire_status": "not_acquired",
  "storage_key": "Ursula K. Le Guin/The Left Hand of Darkness/The Left Hand of Darkness.m4b",
  "error_message": "download failed: HTTP 403 from the store",
  "purchased_at": "2026-03-14T18:02:11Z",
  "tags": "book club, sci-fi",
  "rating_overall": 4.5,
  "rating_performance": 4.7,
  "rating_story": 4.3,
  "is_finished": false,
  "pdf_status": "not_acquired",
  "pdf_storage_key": "Ursula K. Le Guin/The Left Hand of Darkness/The Left Hand of Darkness.pdf",
  "publisher": "Blackstone Publishing",
  "length_minutes": 560,
  "is_abridged": false,
  "content_kind": "book",
  "categories": "Science Fiction & Fantasy > Science Fiction",
  "subtitle": "A Hainish Novel",
  "published_at": "2005-06-21T00:00:00Z",
  "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
  "language": "en",
  "cover_url": "https://images.example.com/covers/9780441478125.jpg",
  "subjects": "Androgyny; Interplanetary voyages; Fiction",
  "enrich_source": "openlibrary",
  "enrich_confidence": 0.92,
  "enrich_updated_at": "2026-10-29T10:13:05Z",
  "created_at": "2026-10-29T10:12:00Z",
  "updated_at": "2026-10-29T10:14:22Z"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Book / library item (one ownership row per store product per account).",
  "required": [
    "id",
    "uuid",
    "plugin_key",
    "source",
    "account_id",
    "product_id",
    "marketplace",
    "title",
    "acquire_status",
    "is_finished",
    "pdf_status",
    "is_abridged",
    "content_kind",
    "created_at",
    "updated_at"
  ],
  "properties": {
    "id": {
      "type": "integer",
      "format": "int64",
      "description": "Surrogate primary key assigned by the database.",
      "example": 412
    },
    "uuid": {
      "type": "string",
      "description": "Stable UUID for this row (API / foreign-key identity).",
      "example": "3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31"
    },
    "plugin_key": {
      "type": "string",
      "description": "Provenance-qualified PluginKey that owns the account.",
      "example": "platform:bookclerk/bookclerk-plugin-source-libro#libro"
    },
    "source": {
      "type": "string",
      "description": "Display alias of the owning plugin (`audible`, `libro`, …).",
      "example": "libro"
    },
    "account_id": {
      "type": "string",
      "description": "Store or operator account id this row belongs to.",
      "example": "reader@example.com"
    },
    "product_id": {
      "type": "string",
      "description": "Storefront product id (ASIN, ISBN, UUID, …).",
      "example": "9780441478125"
    },
    "asin": {
      "type": [
        "string",
        "null"
      ],
      "description": "Audible ASIN when known (`None` for Libro-only rows without enrichment).",
      "example": "B002V0QK4C"
    },
    "isbn": {
      "type": [
        "string",
        "null"
      ],
      "description": "ISBN-13 when known (Libro always; Audible when the API provides it).",
      "example": "9780441478125"
    },
    "marketplace": {
      "type": "string",
      "description": "Store marketplace / locale code (for example `us`, `uk`).",
      "example": "us"
    },
    "title": {
      "type": "string",
      "description": "Display title of the work or edition.",
      "example": "The Left Hand of Darkness"
    },
    "authors": {
      "type": [
        "string",
        "null"
      ],
      "description": "Comma-separated or JSON author list from the storefront.",
      "example": "Ursula K. Le Guin"
    },
    "narrators": {
      "type": [
        "string",
        "null"
      ],
      "description": "Comma-separated or JSON narrator list when present.",
      "example": "George Guidall"
    },
    "series": {
      "type": [
        "string",
        "null"
      ],
      "description": "Series name when the title belongs to a series.",
      "example": "Hainish Cycle"
    },
    "series_index": {
      "type": [
        "string",
        "null"
      ],
      "description": "Position within the series (storefront string form).",
      "example": "4"
    },
    "series_asin": {
      "type": [
        "string",
        "null"
      ],
      "description": "Audible series / podcast-parent ASIN (`Series.AudibleSeriesId`).",
      "example": "B07ZRJZ4PW"
    },
    "acquire_status": {
      "type": "string",
      "description": "Download/acquire pipeline state (`not_acquired`, `queued`, …).",
      "enum": [
        "not_acquired",
        "queued",
        "downloading",
        "acquired",
        "error"
      ]
    },
    "storage_key": {
      "type": [
        "string",
        "null"
      ],
      "description": "Object-storage key for the primary audio artifact, if acquired.",
      "example": "Ursula K. Le Guin/The Left Hand of Darkness/The Left Hand of Darkness.m4b"
    },
    "error_message": {
      "type": [
        "string",
        "null"
      ],
      "description": "Last acquire/convert failure message for operators.",
      "example": "download failed: HTTP 403 from the store"
    },
    "purchased_at": {
      "type": [
        "string",
        "null"
      ],
      "format": "date-time",
      "description": "RFC 3339 purchase time from the storefront, when known.",
      "example": "2026-03-14T18:02:11Z"
    },
    "tags": {
      "type": [
        "string",
        "null"
      ],
      "description": "Operator or storefront tags (serialized string).",
      "example": "book club, sci-fi"
    },
    "rating_overall": {
      "type": [
        "number",
        "null"
      ],
      "format": "float",
      "description": "Overall user rating from the storefront, if any.",
      "example": 4.5
    },
    "rating_performance": {
      "type": [
        "number",
        "null"
      ],
      "format": "float",
      "description": "Narration/performance rating from the storefront, if any.",
      "example": 4.7
    },
    "rating_story": {
      "type": [
        "number",
        "null"
      ],
      "format": "float",
      "description": "Story rating from the storefront, if any.",
      "example": 4.3
    },
    "is_finished": {
      "type": "boolean",
      "description": "Whether the listener marked the title finished (0/1 or bool).",
      "example": false
    },
    "pdf_status": {
      "type": "string",
      "description": "Companion PDF acquire state (`not_acquired`, `acquired`, …).",
      "enum": [
        "not_acquired",
        "queued",
        "downloading",
        "acquired",
        "error"
      ]
    },
    "pdf_storage_key": {
      "type": [
        "string",
        "null"
      ],
      "description": "Object-storage key for the companion PDF, if present.",
      "example": "Ursula K. Le Guin/The Left Hand of Darkness/The Left Hand of Darkness.pdf"
    },
    "publisher": {
      "type": [
        "string",
        "null"
      ],
      "description": "Publisher name from metadata enrichment or the storefront.",
      "example": "Blackstone Publishing"
    },
    "length_minutes": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Runtime in whole minutes when the storefront reports it.",
      "example": 560
    },
    "is_abridged": {
      "type": "boolean",
      "description": "Whether the edition is abridged (0/1 or bool).",
      "example": false
    },
    "content_kind": {
      "type": "string",
      "description": "Title kind: `book`, `episode`, `podcast`, ….",
      "example": "book"
    },
    "categories": {
      "type": [
        "string",
        "null"
      ],
      "description": "Storefront category / genre path list (serialized).",
      "example": "Science Fiction & Fantasy > Science Fiction"
    },
    "subtitle": {
      "type": [
        "string",
        "null"
      ],
      "description": "Optional subtitle from bibliographic metadata.",
      "example": "A Hainish Novel"
    },
    "published_at": {
      "type": [
        "string",
        "null"
      ],
      "format": "date-time",
      "description": "Publication date string from the storefront or enrichment.",
      "example": "2005-06-21T00:00:00Z"
    },
    "description": {
      "type": [
        "string",
        "null"
      ],
      "description": "Blurb / synopsis text (may contain HTML).",
      "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
    },
    "language": {
      "type": [
        "string",
        "null"
      ],
      "description": "BCP-47 or storefront language code when known.",
      "example": "en"
    },
    "cover_url": {
      "type": [
        "string",
        "null"
      ],
      "description": "HTTPS URL for cover art when known.",
      "example": "https://images.example.com/covers/9780441478125.jpg"
    },
    "subjects": {
      "type": [
        "string",
        "null"
      ],
      "description": "Subject / topic tags (often from Open Library; `;`- or `,`-separated).",
      "example": "Androgyny; Interplanetary voyages; Fiction"
    },
    "enrich_source": {
      "type": [
        "string",
        "null"
      ],
      "description": "Plugin or catalog that last enriched bibliographic fields.",
      "example": "openlibrary"
    },
    "enrich_confidence": {
      "type": [
        "number",
        "null"
      ],
      "format": "double",
      "description": "0–1 confidence score for the last enrichment pass.",
      "example": 0.92
    },
    "enrich_updated_at": {
      "type": [
        "string",
        "null"
      ],
      "format": "date-time",
      "description": "RFC 3339 time of the last enrichment write.",
      "example": "2026-10-29T10:13:05Z"
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "RFC 3339 timestamp when the row was inserted.",
      "example": "2026-10-29T10:12:00Z"
    },
    "updated_at": {
      "type": "string",
      "format": "date-time",
      "description": "RFC 3339 timestamp when the row was last modified.",
      "example": "2026-10-29T10:14:22Z"
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

Status `404`: No such title.

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

## `GET /api/library/books/{uuid}/cover`

Serves the JPEG cover sidecar from the deployed local destination, or 404
when no local destination runs on this node.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `uuid` (path) *string*, **required** — Title UUID.

### Responses

- `200` — The cover. Body: binary (`image/jpeg`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such title, or it has no cover. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/books/<uuid>/cover' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/books/<uuid>/cover' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/books/<uuid>/cover",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/books/<uuid>/cover", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/books/<uuid>/cover")
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

Status `404`: No such title, or it has no cover.

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
request timed out
```

Schema:

A `text/plain` body: a string.
