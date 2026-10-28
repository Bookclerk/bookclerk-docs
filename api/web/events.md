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
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The outbox could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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
    "id": "string",
    "eventType": "string",
    "schemaVersion": 0,
    "dispatchState": "string",
    "accountId": "string",
    "dedupKey": "string",
    "createdAt": "string",
    "queueName": "string",
    "contentType": "string",
    "source": "string",
    "deliverAfter": "string"
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
        "description": "Event id."
      },
      "eventType": {
        "type": "string",
        "description": "Event type (`book_acquired`)."
      },
      "schemaVersion": {
        "type": "integer",
        "format": "int64",
        "description": "Payload schema version."
      },
      "dispatchState": {
        "type": "string",
        "description": "`pending` or `dispatched`."
      },
      "accountId": {
        "type": "string",
        "description": "Tenant / account id."
      },
      "dedupKey": {
        "type": "string",
        "description": "Producer idempotency key."
      },
      "createdAt": {
        "type": "string",
        "description": "RFC 3339 insert time."
      },
      "queueName": {
        "type": "string",
        "description": "Queue the row belongs to (`core.events` for domain events)."
      },
      "contentType": {
        "type": "string",
        "description": "Body encoding (`json`, `text`, `bytes`)."
      },
      "source": {
        "type": "string",
        "description": "Producer plugin id; empty when the host published it."
      },
      "deliverAfter": {
        "type": [
          "string",
          "null"
        ],
        "description": "RFC 3339 earliest delivery time for delayed queue messages."
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

Status `500`: The outbox could not be read.

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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The outbox could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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
    "id": "string",
    "eventId": "string",
    "pluginId": "string",
    "state": "string",
    "attemptCount": 0,
    "leaseGeneration": 0,
    "invocationSequence": 0,
    "outcome": "string",
    "errorMessage": "string",
    "runAfter": "string",
    "updatedAt": "string",
    "cancelRequested": true,
    "resourceClass": "string",
    "resumePending": true,
    "hasCheckpoint": true,
    "queueName": "string"
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
        "description": "Delivery id (`{event_id}:{plugin_id}`)."
      },
      "eventId": {
        "type": "string",
        "description": "Parent event id."
      },
      "pluginId": {
        "type": "string",
        "description": "Subscriber plugin id."
      },
      "state": {
        "type": "string",
        "description": "Lifecycle state."
      },
      "attemptCount": {
        "type": "integer",
        "format": "int64",
        "description": "Claim count (not incremented on resume)."
      },
      "leaseGeneration": {
        "type": "integer",
        "format": "int64",
        "description": "Fence generation."
      },
      "invocationSequence": {
        "type": "integer",
        "format": "int64",
        "description": "Resume ordinal."
      },
      "outcome": {
        "type": [
          "string",
          "null"
        ],
        "description": "Terminal outcome when set."
      },
      "errorMessage": {
        "type": [
          "string",
          "null"
        ],
        "description": "Operator-facing error."
      },
      "runAfter": {
        "type": "string",
        "description": "RFC 3339 earliest claim time."
      },
      "updatedAt": {
        "type": "string",
        "description": "RFC 3339 last update."
      },
      "cancelRequested": {
        "type": "boolean",
        "description": "Cooperative cancel flag."
      },
      "resourceClass": {
        "type": "string",
        "description": "Concurrency class (`network` today)."
      },
      "resumePending": {
        "type": "boolean",
        "description": "Next claim should resume rather than increment attempt."
      },
      "hasCheckpoint": {
        "type": "boolean",
        "description": "True when a suspend checkpoint is stored."
      },
      "queueName": {
        "type": "string",
        "description": "Queue of the parent event."
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

Status `500`: The outbox could not be read.

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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The outbox could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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
    "name": "string",
    "core": true,
    "pending": 0,
    "running": 0,
    "acked": 0,
    "deadLetter": 0,
    "consumers": [
      "string"
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
        "description": "Queue name."
      },
      "core": {
        "type": "boolean",
        "description": "True for the domain-event bus (`core.events`)."
      },
      "pending": {
        "type": "integer",
        "format": "int64",
        "description": "Deliveries waiting to be claimed (including suspended rows)."
      },
      "running": {
        "type": "integer",
        "format": "int64",
        "description": "Deliveries currently claimed by a worker."
      },
      "acked": {
        "type": "integer",
        "format": "int64",
        "description": "Deliveries acked."
      },
      "deadLetter": {
        "type": "integer",
        "format": "int64",
        "description": "Deliveries parked for operator review."
      },
      "consumers": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "description": "Live catalog consumers (plugin keys or instance ids) of this queue."
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

Status `500`: The outbox could not be read.

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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The outbox could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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
  "name": "string",
  "core": true,
  "pending": 0,
  "running": 0,
  "acked": 0,
  "deadLetter": 0,
  "consumers": [
    "string"
  ],
  "messages": [
    {
      "id": "string",
      "eventType": "string",
      "schemaVersion": 0,
      "dispatchState": "string",
      "accountId": "string",
      "dedupKey": "string",
      "createdAt": "string",
      "queueName": "string",
      "contentType": "string",
      "source": "string",
      "deliverAfter": "string"
    }
  ],
  "deliveries": [
    {
      "id": "string",
      "eventId": "string",
      "pluginId": "string",
      "state": "string",
      "attemptCount": 0,
      "leaseGeneration": 0,
      "invocationSequence": 0,
      "outcome": "string",
      "errorMessage": "string",
      "runAfter": "string",
      "updatedAt": "string",
      "cancelRequested": true,
      "resourceClass": "string",
      "resumePending": true,
      "hasCheckpoint": true,
      "queueName": "string"
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
          "description": "Queue name."
        },
        "core": {
          "type": "boolean",
          "description": "True for the domain-event bus (`core.events`)."
        },
        "pending": {
          "type": "integer",
          "format": "int64",
          "description": "Deliveries waiting to be claimed (including suspended rows)."
        },
        "running": {
          "type": "integer",
          "format": "int64",
          "description": "Deliveries currently claimed by a worker."
        },
        "acked": {
          "type": "integer",
          "format": "int64",
          "description": "Deliveries acked."
        },
        "deadLetter": {
          "type": "integer",
          "format": "int64",
          "description": "Deliveries parked for operator review."
        },
        "consumers": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Live catalog consumers (plugin keys or instance ids) of this queue."
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
                "description": "Event id."
              },
              "eventType": {
                "type": "string",
                "description": "Event type (`book_acquired`)."
              },
              "schemaVersion": {
                "type": "integer",
                "format": "int64",
                "description": "Payload schema version."
              },
              "dispatchState": {
                "type": "string",
                "description": "`pending` or `dispatched`."
              },
              "accountId": {
                "type": "string",
                "description": "Tenant / account id."
              },
              "dedupKey": {
                "type": "string",
                "description": "Producer idempotency key."
              },
              "createdAt": {
                "type": "string",
                "description": "RFC 3339 insert time."
              },
              "queueName": {
                "type": "string",
                "description": "Queue the row belongs to (`core.events` for domain events)."
              },
              "contentType": {
                "type": "string",
                "description": "Body encoding (`json`, `text`, `bytes`)."
              },
              "source": {
                "type": "string",
                "description": "Producer plugin id; empty when the host published it."
              },
              "deliverAfter": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "RFC 3339 earliest delivery time for delayed queue messages."
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
                "description": "Delivery id (`{event_id}:{plugin_id}`)."
              },
              "eventId": {
                "type": "string",
                "description": "Parent event id."
              },
              "pluginId": {
                "type": "string",
                "description": "Subscriber plugin id."
              },
              "state": {
                "type": "string",
                "description": "Lifecycle state."
              },
              "attemptCount": {
                "type": "integer",
                "format": "int64",
                "description": "Claim count (not incremented on resume)."
              },
              "leaseGeneration": {
                "type": "integer",
                "format": "int64",
                "description": "Fence generation."
              },
              "invocationSequence": {
                "type": "integer",
                "format": "int64",
                "description": "Resume ordinal."
              },
              "outcome": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Terminal outcome when set."
              },
              "errorMessage": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Operator-facing error."
              },
              "runAfter": {
                "type": "string",
                "description": "RFC 3339 earliest claim time."
              },
              "updatedAt": {
                "type": "string",
                "description": "RFC 3339 last update."
              },
              "cancelRequested": {
                "type": "boolean",
                "description": "Cooperative cancel flag."
              },
              "resourceClass": {
                "type": "string",
                "description": "Concurrency class (`network` today)."
              },
              "resumePending": {
                "type": "boolean",
                "description": "Next claim should resume rather than increment attempt."
              },
              "hasCheckpoint": {
                "type": "boolean",
                "description": "True when a suspend checkpoint is stored."
              },
              "queueName": {
                "type": "string",
                "description": "Queue of the parent event."
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

Status `500`: The outbox could not be read.

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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — No such delivery, or it is not in a state this applies to. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The outbox could not be read or written. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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

Status `404`: No such delivery, or it is not in a state this applies to.

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

Status `500`: The outbox could not be read or written.

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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — No such delivery, or it is not in a state this applies to. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The outbox could not be read or written. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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

Status `404`: No such delivery, or it is not in a state this applies to.

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

Status `500`: The outbox could not be read or written.

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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — No such delivery, or it is not in a state this applies to. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The outbox could not be read or written. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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

Status `404`: No such delivery, or it is not in a state this applies to.

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

Status `500`: The outbox could not be read or written.

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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `404` — No such delivery, or it is not in a state this applies to. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The outbox could not be read or written. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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

Status `404`: No such delivery, or it is not in a state this applies to.

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

Status `500`: The outbox could not be read or written.

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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
