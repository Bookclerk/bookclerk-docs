# Status

Library and daemon snapshot.

Back to the [web API reference](README.md).

## `GET /api/status`

Returns library counts, listen addresses, storage backends, and whether operator auth is on.

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
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The library could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/status' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/status' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/status",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/status", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/status")
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

```json
{
  "type": "string"
}
```

## `GET /api/cluster/status`

`GET /api/cluster/status`: cluster-wide health every node administrator
may see (node counts, committed capacity, active jobs, delivery backlog),
plus detail for the nodes the caller may see.

**Access:** The operator, a signed-in Owner, Administrator or Host, or a user who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Cluster-wide node health. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/cluster/status' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/cluster/status' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/cluster/status",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/cluster/status", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/cluster/status")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Cluster-wide node health. The body is JSON the schemas do not describe yet.

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

Status `401`: The caller is not signed in.

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

Status `403`: The caller may not do this.

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

```json
{
  "type": "string"
}
```
