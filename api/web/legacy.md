# Legacy

Old unprefixed paths; use the `/api/…` route named in each.

Back to the [web API reference](README.md).

## `GET /status`

**Deprecated.**

Deprecated alias of `GET /api/status`.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Library counts and daemon state. Body: [`StatusResponse`](schemas.md#statusresponse) (`application/json`)
  - `accounts` *integer*, **required** — Count of configured store accounts. At least 0.
  - `books` *integer*, **required** — Total title rows in the library. At least 0.
  - `acquired` *integer (int64)*, **required** — Titles whose `acquire_status` is `acquired`.
  - `pending` *integer (int64)*, **required** — Titles still needing acquire (`not_acquired`).
  - `error` *integer (int64)*, **required** — Titles stuck in `error` after a failed acquire.
  - `in_progress` *integer (int64)*, **required** — Titles currently `queued` or `downloading`.
  - `listen` *string*, **required** — Comma-joined `daemon.listen` addresses from live config.
  - `storage_backend` *string*, **required** — Comma-joined enabled destination plugin ids, or `none`.
  - `auth_enabled` *boolean*, **required** — Whether the live auth middleware requires operator credentials.
  - `events` *[`EventQueueStatus`](schemas.md#eventqueuestatus)*, **required** — Durable domain-event delivery-queue counters.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The library could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/status' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/status' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/status",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/status", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/status")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Library counts and daemon state.

```json
{
  "accounts": 0,
  "books": 0,
  "acquired": 0,
  "pending": 0,
  "error": 0,
  "in_progress": 0,
  "listen": "string",
  "storage_backend": "string",
  "auth_enabled": true,
  "events": {
    "pending": 0,
    "running": 0,
    "suspended": 0,
    "dead_letter": 0,
    "acked": 0,
    "oldest_pending_age_secs": 0,
    "retries_total": 0,
    "suspensions_total": 0,
    "dead_letters_total": 0,
    "dispatch_latency_ms_avg": 0,
    "handler_latency_ms_avg": 0
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "Library and daemon snapshot for `GET /api/status`.",
  "required": [
    "accounts",
    "books",
    "acquired",
    "pending",
    "error",
    "in_progress",
    "listen",
    "storage_backend",
    "auth_enabled",
    "events"
  ],
  "properties": {
    "accounts": {
      "type": "integer",
      "description": "Count of configured store accounts.",
      "minimum": 0
    },
    "books": {
      "type": "integer",
      "description": "Total title rows in the library.",
      "minimum": 0
    },
    "acquired": {
      "type": "integer",
      "format": "int64",
      "description": "Titles whose `acquire_status` is `acquired`."
    },
    "pending": {
      "type": "integer",
      "format": "int64",
      "description": "Titles still needing acquire (`not_acquired`)."
    },
    "error": {
      "type": "integer",
      "format": "int64",
      "description": "Titles stuck in `error` after a failed acquire."
    },
    "in_progress": {
      "type": "integer",
      "format": "int64",
      "description": "Titles currently `queued` or `downloading`."
    },
    "listen": {
      "type": "string",
      "description": "Comma-joined `daemon.listen` addresses from live config."
    },
    "storage_backend": {
      "type": "string",
      "description": "Comma-joined enabled destination plugin ids, or `none`."
    },
    "auth_enabled": {
      "type": "boolean",
      "description": "Whether the live auth middleware requires operator credentials."
    },
    "events": {
      "type": "object",
      "description": "Durable domain-event delivery-queue counters.",
      "required": [
        "pending",
        "running",
        "suspended",
        "dead_letter",
        "acked",
        "retries_total",
        "suspensions_total",
        "dead_letters_total"
      ],
      "properties": {
        "pending": {
          "type": "integer",
          "format": "int64",
          "description": "Pending deliveries without a suspend checkpoint."
        },
        "running": {
          "type": "integer",
          "format": "int64",
          "description": "Currently claimed deliveries."
        },
        "suspended": {
          "type": "integer",
          "format": "int64",
          "description": "Pending deliveries parked with a checkpoint."
        },
        "dead_letter": {
          "type": "integer",
          "format": "int64",
          "description": "Dead-lettered deliveries."
        },
        "acked": {
          "type": "integer",
          "format": "int64",
          "description": "Successfully acked deliveries."
        },
        "oldest_pending_age_secs": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Age in seconds of the oldest pending delivery, when any exist."
        },
        "retries_total": {
          "type": "integer",
          "format": "int64",
          "description": "Durable retry outcomes (including reclaim-as-retry)."
        },
        "suspensions_total": {
          "type": "integer",
          "format": "int64",
          "description": "Durable suspend outcomes."
        },
        "dead_letters_total": {
          "type": "integer",
          "format": "int64",
          "description": "Durable transitions into `dead_letter`."
        },
        "dispatch_latency_ms_avg": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Average first-dispatch latency in milliseconds, when sampled."
        },
        "handler_latency_ms_avg": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Average `onEvent` handler duration in milliseconds, when sampled."
        }
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

Status `500`: The library could not be read.

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

## `POST /scan`

**Deprecated.**

Deprecated alias of `POST /api/library/scan`.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

Optional.

`application/json`: [`ScanRequest`](schemas.md#scanrequest) or null

- `account` *string or null* — When set, scan this account even if `scan_enabled` is false.
- `plugin_instance_id` *string or null* — When set, scan only this deployed plugin instance.

### Responses

- `200` — The scan jobs were accepted; see `POST /api/library/scan` for the other answers. Body: [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when at least one job was accepted.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — First accepted (or, when none was, already queued) job id.
  - `job_ids` *array of string*, **required** — Every accepted or already-queued job id, in plan order.
  - `error` *string or null* — Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`).
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/scan' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/scan' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/scan",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/scan", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/scan")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The scan jobs were accepted; see `POST /api/library/scan` for the other answers.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "job_ids": [
    "string"
  ],
  "error": "string"
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
      "description": "True when at least one job was accepted."
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text."
    },
    "job_id": {
      "type": "string",
      "description": "First accepted (or, when none was, already queued) job id."
    },
    "job_ids": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Every accepted or already-queued job id, in plan order."
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when nothing was accepted (`conflict`, `queue_full`,\n`nothing_to_scan`, `nothing_to_acquire`)."
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

## `POST /acquire`

**Deprecated.**

Deprecated alias of `POST /api/library/acquire`.

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

- `200` — The acquire jobs were accepted; see `POST /api/library/acquire` for the other answers. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/acquire' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/acquire' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/acquire",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/acquire", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/acquire")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The acquire jobs were accepted; see `POST /api/library/acquire` for the other answers.

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

## `GET /jobs`

**Deprecated.**

Deprecated alias of `GET /api/jobs`.

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
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The job table could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/jobs' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/jobs' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/jobs",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/jobs", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/jobs")
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

## `POST /jobs/{id}/cancel`

**Deprecated.**

Deprecated alias of `POST /api/jobs/{id}/cancel`.

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
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — No such job. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/jobs/<id>/cancel' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/jobs/<id>/cancel' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/jobs/<id>/cancel",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/jobs/<id>/cancel", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/jobs/<id>/cancel")
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

## `POST /integrations/{id}/scan`

Enqueues a remote integration library scan; 404 if missing, 400 if unsupported.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Integration instance id.

### Request body

Optional.

`application/json`: [`IntegrationScanRequest`](schemas.md#integrationscanrequest) or null

- `force` *boolean or null* — When true, the integration rescans even if it considers itself current.

### Responses

- `200` — The job was accepted. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `400` — The integration cannot scan a library. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — No integration with this id. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `409` — The same job is already pending or running, or no node can run it. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
- `429` — The job queue is full. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
- `500` — The job could not be queued. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
- `503` — Job admission is paused. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/integrations/<id>/scan' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/integrations/<id>/scan' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/integrations/<id>/scan",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/integrations/<id>/scan", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/integrations/<id>/scan")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The job was accepted.

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

Status `400`: The integration cannot scan a library.

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

Status `404`: No integration with this id.

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

Status `409`: The same job is already pending or running, or no node can run it.

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

Status `429`: The job queue is full.

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

Status `500`: The job could not be queued.

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

Status `503`: Job admission is paused.

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
