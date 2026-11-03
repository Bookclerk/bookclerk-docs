# Events

Domain events and their deliveries.

Back to the [web API reference](README.md).

## `GET /api/events`

Returns recent domain-event outbox rows, newest first.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Recent domain events, newest first. Body: array of [`DomainEventInfo`](schemas.md#domaineventinfo) (`application/json`)
  - `id` *string*, **required** — Event id.
  - `eventType` *string*, **required** — Event type (`book_acquired`).
  - `schemaVersion` *integer (int64)*, **required** — Payload schema version.
  - `dispatchState` *string*, **required** — `pending` or `dispatched`.
  - `accountId` *string*, **required** — Tenant / account id.
  - `dedupKey` *string*, **required** — Producer idempotency key.
  - `createdAt` *string*, **required** — RFC 3339 insert time.
  - `queueName` *string*, **required** — Queue the row belongs to (`core.events` for domain events).
  - `contentType` *string*, **required** — Body encoding (`json`, `text`, `bytes`).
  - `source` *string*, **required** — Producer plugin id; empty when the host published it.
  - `deliverAfter` *string or null* — RFC 3339 earliest delivery time for delayed queue messages.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The outbox could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/events' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/events",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/events", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/events")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Recent domain events, newest first.

```json
[
  {
    "id": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d",
    "eventType": "book_acquired",
    "schemaVersion": 1,
    "dispatchState": "dispatched",
    "accountId": "reader@example.com",
    "dedupKey": "book_acquired:3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31",
    "createdAt": "2026-10-29T10:12:00Z",
    "queueName": "core.events",
    "contentType": "json",
    "source": "",
    "deliverAfter": "2026-10-29T10:15:00Z"
  }
]
```

Schema:

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "description": "Operator JSON for one `domain_events` row.",
    "required": [
      "id",
      "eventType",
      "schemaVersion",
      "dispatchState",
      "accountId",
      "dedupKey",
      "createdAt",
      "queueName",
      "contentType",
      "source"
    ],
    "properties": {
      "id": {
        "type": "string",
        "description": "Event id.",
        "example": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d"
      },
      "eventType": {
        "type": "string",
        "description": "Event type (`book_acquired`).",
        "example": "book_acquired"
      },
      "schemaVersion": {
        "type": "integer",
        "format": "int64",
        "description": "Payload schema version.",
        "example": 1
      },
      "dispatchState": {
        "type": "string",
        "description": "`pending` or `dispatched`.",
        "example": "dispatched"
      },
      "accountId": {
        "type": "string",
        "description": "Tenant / account id.",
        "example": "reader@example.com"
      },
      "dedupKey": {
        "type": "string",
        "description": "Producer idempotency key.",
        "example": "book_acquired:3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31"
      },
      "createdAt": {
        "type": "string",
        "description": "RFC 3339 insert time.",
        "example": "2026-10-29T10:12:00Z"
      },
      "queueName": {
        "type": "string",
        "description": "Queue the row belongs to (`core.events` for domain events).",
        "example": "core.events"
      },
      "contentType": {
        "type": "string",
        "description": "Body encoding (`json`, `text`, `bytes`).",
        "example": "json"
      },
      "source": {
        "type": "string",
        "description": "Producer plugin id; empty when the host published it.",
        "example": ""
      },
      "deliverAfter": {
        "type": [
          "string",
          "null"
        ],
        "description": "RFC 3339 earliest delivery time for delayed queue messages.",
        "example": "2026-10-29T10:15:00Z"
      }
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

Status `500`: The outbox could not be read.

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

## `GET /api/events/deliveries`

Returns event deliveries, optionally filtered by `state`.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `state` (query) *string* — Delivery state (`dead_letter`, `pending`, …).

### Responses

- `200` — Deliveries, newest first. Body: array of [`EventDeliveryInfo`](schemas.md#eventdeliveryinfo) (`application/json`)
  - `id` *string*, **required** — Delivery id (`{event_id}:{plugin_id}`).
  - `eventId` *string*, **required** — Parent event id.
  - `pluginId` *string*, **required** — Subscriber plugin id.
  - `state` *string*, **required** — Lifecycle state.
  - `attemptCount` *integer (int64)*, **required** — Claim count (not incremented on resume).
  - `leaseGeneration` *integer (int64)*, **required** — Fence generation.
  - `invocationSequence` *integer (int64)*, **required** — Resume ordinal.
  - `outcome` *string or null* — Terminal outcome when set.
  - `errorMessage` *string or null* — Operator-facing error.
  - `runAfter` *string*, **required** — RFC 3339 earliest claim time.
  - `updatedAt` *string*, **required** — RFC 3339 last update.
  - `cancelRequested` *boolean*, **required** — Cooperative cancel flag.
  - `resourceClass` *string*, **required** — Concurrency class (`network` today).
  - `resumePending` *boolean*, **required** — Next claim should resume rather than increment attempt.
  - `hasCheckpoint` *boolean*, **required** — True when a suspend checkpoint is stored.
  - `queueName` *string*, **required** — Queue of the parent event.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The outbox could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events/deliveries' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/events/deliveries' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/events/deliveries",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/events/deliveries", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/events/deliveries")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Deliveries, newest first.

```json
[
  {
    "id": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf",
    "eventId": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d",
    "pluginId": "audiobookshelf",
    "state": "pending",
    "attemptCount": 1,
    "leaseGeneration": 2,
    "invocationSequence": 0,
    "outcome": "ack",
    "errorMessage": "integration API error (503): library scan in progress",
    "runAfter": "2026-10-29T10:12:00Z",
    "updatedAt": "2026-10-29T10:12:30Z",
    "cancelRequested": false,
    "resourceClass": "network",
    "resumePending": false,
    "hasCheckpoint": false,
    "queueName": "core.events"
  }
]
```

Schema:

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "description": "Operator JSON for one `event_deliveries` row.",
    "required": [
      "id",
      "eventId",
      "pluginId",
      "state",
      "attemptCount",
      "leaseGeneration",
      "invocationSequence",
      "runAfter",
      "updatedAt",
      "cancelRequested",
      "resourceClass",
      "resumePending",
      "hasCheckpoint",
      "queueName"
    ],
    "properties": {
      "id": {
        "type": "string",
        "description": "Delivery id (`{event_id}:{plugin_id}`).",
        "example": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf"
      },
      "eventId": {
        "type": "string",
        "description": "Parent event id.",
        "example": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d"
      },
      "pluginId": {
        "type": "string",
        "description": "Subscriber plugin id.",
        "example": "audiobookshelf"
      },
      "state": {
        "type": "string",
        "description": "Lifecycle state.",
        "example": "pending"
      },
      "attemptCount": {
        "type": "integer",
        "format": "int64",
        "description": "Claim count (not incremented on resume).",
        "example": 1
      },
      "leaseGeneration": {
        "type": "integer",
        "format": "int64",
        "description": "Fence generation.",
        "example": 2
      },
      "invocationSequence": {
        "type": "integer",
        "format": "int64",
        "description": "Resume ordinal.",
        "example": 0
      },
      "outcome": {
        "type": [
          "string",
          "null"
        ],
        "description": "Terminal outcome when set.",
        "example": "ack"
      },
      "errorMessage": {
        "type": [
          "string",
          "null"
        ],
        "description": "Operator-facing error.",
        "example": "integration API error (503): library scan in progress"
      },
      "runAfter": {
        "type": "string",
        "description": "RFC 3339 earliest claim time.",
        "example": "2026-10-29T10:12:00Z"
      },
      "updatedAt": {
        "type": "string",
        "description": "RFC 3339 last update.",
        "example": "2026-10-29T10:12:30Z"
      },
      "cancelRequested": {
        "type": "boolean",
        "description": "Cooperative cancel flag.",
        "example": false
      },
      "resourceClass": {
        "type": "string",
        "description": "Concurrency class (`network` today).",
        "example": "network"
      },
      "resumePending": {
        "type": "boolean",
        "description": "Next claim should resume rather than increment attempt.",
        "example": false
      },
      "hasCheckpoint": {
        "type": "boolean",
        "description": "True when a suspend checkpoint is stored.",
        "example": false
      },
      "queueName": {
        "type": "string",
        "description": "Queue of the parent event.",
        "example": "core.events"
      }
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

Status `500`: The outbox could not be read.

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

## `GET /api/queues`

Returns every queue the outbox or the live catalog knows about, with
delivery counters and the consumers subscribed to it.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Every known queue. Body: array of [`QueueInfo`](schemas.md#queueinfo) (`application/json`)
  - `name` *string*, **required** — Queue name.
  - `core` *boolean*, **required** — True for the domain-event bus (`core.events`).
  - `pending` *integer (int64)*, **required** — Deliveries waiting to be claimed (including suspended rows).
  - `running` *integer (int64)*, **required** — Deliveries currently claimed by a worker.
  - `acked` *integer (int64)*, **required** — Deliveries acked.
  - `deadLetter` *integer (int64)*, **required** — Deliveries parked for operator review.
  - `consumers` *array of string*, **required** — Live catalog consumers (plugin keys or instance ids) of this queue.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The outbox could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/queues' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/queues' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/queues",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/queues", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/queues")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Every known queue.

```json
[
  {
    "name": "core.events",
    "core": true,
    "pending": 3,
    "running": 1,
    "acked": 1284,
    "deadLetter": 0,
    "consumers": [
      "platform:bookclerk/bookclerk-plugin-integration-audiobookshelf#audiobookshelf"
    ]
  }
]
```

Schema:

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "description": "Operator JSON for one queue on `GET /api/queues`.",
    "required": [
      "name",
      "core",
      "pending",
      "running",
      "acked",
      "deadLetter",
      "consumers"
    ],
    "properties": {
      "name": {
        "type": "string",
        "description": "Queue name.",
        "example": "core.events"
      },
      "core": {
        "type": "boolean",
        "description": "True for the domain-event bus (`core.events`).",
        "example": true
      },
      "pending": {
        "type": "integer",
        "format": "int64",
        "description": "Deliveries waiting to be claimed (including suspended rows).",
        "example": 3
      },
      "running": {
        "type": "integer",
        "format": "int64",
        "description": "Deliveries currently claimed by a worker.",
        "example": 1
      },
      "acked": {
        "type": "integer",
        "format": "int64",
        "description": "Deliveries acked.",
        "example": 1284
      },
      "deadLetter": {
        "type": "integer",
        "format": "int64",
        "description": "Deliveries parked for operator review.",
        "example": 0
      },
      "consumers": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "description": "Live catalog consumers (plugin keys or instance ids) of this queue.",
        "example": [
          "platform:bookclerk/bookclerk-plugin-integration-audiobookshelf#audiobookshelf"
        ]
      }
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

Status `500`: The outbox could not be read.

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

## `GET /api/queues/{name}`

Returns one queue's counters, consumers, recent messages, and deliveries.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `name` (path) *string*, **required** — Queue name.

### Responses

- `200` — The queue. Body: [`QueueDetail`](schemas.md#queuedetail) (`application/json`)
  - `messages` *array of [`DomainEventInfo`](schemas.md#domaineventinfo)*, **required** — Recent messages on the queue, newest first.
  - `deliveries` *array of [`EventDeliveryInfo`](schemas.md#eventdeliveryinfo)*, **required** — Recent deliveries on the queue, newest first.
- `400` — The queue name is invalid. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The outbox could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/queues/<name>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/queues/<name>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/queues/<name>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/queues/<name>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/queues/<name>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The queue.

```json
{
  "name": "core.events",
  "core": true,
  "pending": 3,
  "running": 1,
  "acked": 1284,
  "deadLetter": 0,
  "consumers": [
    "platform:bookclerk/bookclerk-plugin-integration-audiobookshelf#audiobookshelf"
  ],
  "messages": [
    {
      "id": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d",
      "eventType": "book_acquired",
      "schemaVersion": 1,
      "dispatchState": "dispatched",
      "accountId": "reader@example.com",
      "dedupKey": "book_acquired:3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31",
      "createdAt": "2026-10-29T10:12:00Z",
      "queueName": "core.events",
      "contentType": "json",
      "source": "",
      "deliverAfter": "2026-10-29T10:15:00Z"
    }
  ],
  "deliveries": [
    {
      "id": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf",
      "eventId": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d",
      "pluginId": "audiobookshelf",
      "state": "pending",
      "attemptCount": 1,
      "leaseGeneration": 2,
      "invocationSequence": 0,
      "outcome": "ack",
      "errorMessage": "integration API error (503): library scan in progress",
      "runAfter": "2026-10-29T10:12:00Z",
      "updatedAt": "2026-10-29T10:12:30Z",
      "cancelRequested": false,
      "resourceClass": "network",
      "resumePending": false,
      "hasCheckpoint": false,
      "queueName": "core.events"
    }
  ]
}
```

Schema:

```json
{
  "allOf": [
    {
      "type": "object",
      "description": "Queue summary.",
      "required": [
        "name",
        "core",
        "pending",
        "running",
        "acked",
        "deadLetter",
        "consumers"
      ],
      "properties": {
        "name": {
          "type": "string",
          "description": "Queue name.",
          "example": "core.events"
        },
        "core": {
          "type": "boolean",
          "description": "True for the domain-event bus (`core.events`).",
          "example": true
        },
        "pending": {
          "type": "integer",
          "format": "int64",
          "description": "Deliveries waiting to be claimed (including suspended rows).",
          "example": 3
        },
        "running": {
          "type": "integer",
          "format": "int64",
          "description": "Deliveries currently claimed by a worker.",
          "example": 1
        },
        "acked": {
          "type": "integer",
          "format": "int64",
          "description": "Deliveries acked.",
          "example": 1284
        },
        "deadLetter": {
          "type": "integer",
          "format": "int64",
          "description": "Deliveries parked for operator review.",
          "example": 0
        },
        "consumers": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Live catalog consumers (plugin keys or instance ids) of this queue.",
          "example": [
            "platform:bookclerk/bookclerk-plugin-integration-audiobookshelf#audiobookshelf"
          ]
        }
      }
    },
    {
      "type": "object",
      "required": [
        "messages",
        "deliveries"
      ],
      "properties": {
        "messages": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator JSON for one `domain_events` row.",
            "required": [
              "id",
              "eventType",
              "schemaVersion",
              "dispatchState",
              "accountId",
              "dedupKey",
              "createdAt",
              "queueName",
              "contentType",
              "source"
            ],
            "properties": {
              "id": {
                "type": "string",
                "description": "Event id.",
                "example": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d"
              },
              "eventType": {
                "type": "string",
                "description": "Event type (`book_acquired`).",
                "example": "book_acquired"
              },
              "schemaVersion": {
                "type": "integer",
                "format": "int64",
                "description": "Payload schema version.",
                "example": 1
              },
              "dispatchState": {
                "type": "string",
                "description": "`pending` or `dispatched`.",
                "example": "dispatched"
              },
              "accountId": {
                "type": "string",
                "description": "Tenant / account id.",
                "example": "reader@example.com"
              },
              "dedupKey": {
                "type": "string",
                "description": "Producer idempotency key.",
                "example": "book_acquired:3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31"
              },
              "createdAt": {
                "type": "string",
                "description": "RFC 3339 insert time.",
                "example": "2026-10-29T10:12:00Z"
              },
              "queueName": {
                "type": "string",
                "description": "Queue the row belongs to (`core.events` for domain events).",
                "example": "core.events"
              },
              "contentType": {
                "type": "string",
                "description": "Body encoding (`json`, `text`, `bytes`).",
                "example": "json"
              },
              "source": {
                "type": "string",
                "description": "Producer plugin id; empty when the host published it.",
                "example": ""
              },
              "deliverAfter": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "RFC 3339 earliest delivery time for delayed queue messages.",
                "example": "2026-10-29T10:15:00Z"
              }
            }
          },
          "description": "Recent messages on the queue, newest first."
        },
        "deliveries": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator JSON for one `event_deliveries` row.",
            "required": [
              "id",
              "eventId",
              "pluginId",
              "state",
              "attemptCount",
              "leaseGeneration",
              "invocationSequence",
              "runAfter",
              "updatedAt",
              "cancelRequested",
              "resourceClass",
              "resumePending",
              "hasCheckpoint",
              "queueName"
            ],
            "properties": {
              "id": {
                "type": "string",
                "description": "Delivery id (`{event_id}:{plugin_id}`).",
                "example": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf"
              },
              "eventId": {
                "type": "string",
                "description": "Parent event id.",
                "example": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d"
              },
              "pluginId": {
                "type": "string",
                "description": "Subscriber plugin id.",
                "example": "audiobookshelf"
              },
              "state": {
                "type": "string",
                "description": "Lifecycle state.",
                "example": "pending"
              },
              "attemptCount": {
                "type": "integer",
                "format": "int64",
                "description": "Claim count (not incremented on resume).",
                "example": 1
              },
              "leaseGeneration": {
                "type": "integer",
                "format": "int64",
                "description": "Fence generation.",
                "example": 2
              },
              "invocationSequence": {
                "type": "integer",
                "format": "int64",
                "description": "Resume ordinal.",
                "example": 0
              },
              "outcome": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Terminal outcome when set.",
                "example": "ack"
              },
              "errorMessage": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Operator-facing error.",
                "example": "integration API error (503): library scan in progress"
              },
              "runAfter": {
                "type": "string",
                "description": "RFC 3339 earliest claim time.",
                "example": "2026-10-29T10:12:00Z"
              },
              "updatedAt": {
                "type": "string",
                "description": "RFC 3339 last update.",
                "example": "2026-10-29T10:12:30Z"
              },
              "cancelRequested": {
                "type": "boolean",
                "description": "Cooperative cancel flag.",
                "example": false
              },
              "resourceClass": {
                "type": "string",
                "description": "Concurrency class (`network` today).",
                "example": "network"
              },
              "resumePending": {
                "type": "boolean",
                "description": "Next claim should resume rather than increment attempt.",
                "example": false
              },
              "hasCheckpoint": {
                "type": "boolean",
                "description": "True when a suspend checkpoint is stored.",
                "example": false
              },
              "queueName": {
                "type": "string",
                "description": "Queue of the parent event.",
                "example": "core.events"
              }
            }
          },
          "description": "Recent deliveries on the queue, newest first."
        }
      }
    }
  ],
  "description": "Operator JSON for `GET /api/queues/{name}`."
}
```

Status `400`: The queue name is invalid.

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

Status `500`: The outbox could not be read.

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

## `POST /api/events/deliveries/{id}/retry`

Re-queue a dead-lettered delivery.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Delivery id.

### Responses

- `200` — The delivery was queued again. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
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
- `404` — No such delivery, or it is not in a state this applies to. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The outbox could not be read or written. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events/deliveries/<id>/retry' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/events/deliveries/<id>/retry' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/events/deliveries/<id>/retry",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/events/deliveries/<id>/retry", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/events/deliveries/<id>/retry")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The delivery was queued again.

```json
{
  "job_id": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf",
  "message": "delivery b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf re-queued",
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

Status `404`: No such delivery, or it is not in a state this applies to.

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

Status `500`: The outbox could not be read or written.

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

## `POST /api/events/deliveries/{id}/acknowledge`

Acknowledge a dead-lettered delivery (no further retry).

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Delivery id.

### Responses

- `200` — The delivery was acknowledged. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
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
- `404` — No such delivery, or it is not in a state this applies to. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The outbox could not be read or written. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events/deliveries/<id>/acknowledge' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/events/deliveries/<id>/acknowledge' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/events/deliveries/<id>/acknowledge",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/events/deliveries/<id>/acknowledge", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/events/deliveries/<id>/acknowledge")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The delivery was acknowledged.

```json
{
  "job_id": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf",
  "message": "delivery b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf acknowledged",
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

Status `404`: No such delivery, or it is not in a state this applies to.

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

Status `500`: The outbox could not be read or written.

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

## `POST /api/events/deliveries/{id}/cancel`

Cancel a pending delivery or request cooperative stop of a running one.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Delivery id.

### Responses

- `200` — The delivery was cancelled or asked to stop. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
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
- `404` — No such delivery, or it is not in a state this applies to. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The outbox could not be read or written. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events/deliveries/<id>/cancel' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/events/deliveries/<id>/cancel' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/events/deliveries/<id>/cancel",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/events/deliveries/<id>/cancel", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/events/deliveries/<id>/cancel")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The delivery was cancelled or asked to stop.

```json
{
  "job_id": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf",
  "message": "delivery b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf rejected",
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

Status `404`: No such delivery, or it is not in a state this applies to.

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

Status `500`: The outbox could not be read or written.

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

## `POST /api/events/deliveries/{id}/resume`

Resume a suspended delivery (wake now, next claim uses resume_pending).

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Delivery id.

### Responses

- `200` — The delivery was resumed. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
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
- `404` — No such delivery, or it is not in a state this applies to. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The outbox could not be read or written. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events/deliveries/<id>/resume' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/events/deliveries/<id>/resume' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/events/deliveries/<id>/resume",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/events/deliveries/<id>/resume", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/events/deliveries/<id>/resume")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The delivery was resumed.

```json
{
  "job_id": "b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf",
  "message": "delivery b3f1c2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d:audiobookshelf resumed",
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

Status `404`: No such delivery, or it is not in a state this applies to.

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

Status `500`: The outbox could not be read or written.

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
