# Library

Books in the library, scans and acquires.

Back to the [web API reference](README.md).

## `POST /api/library/convert`

Queues a convert of stored books into another copy; returns the
accepted job ids.

Each book gets the copy beside its original, made from the copy the
library already stores, on every destination whose filter takes that
copy; the original is left as it is. No store is asked. One job per
title, run like an acquire on a node with the media worker and the
library database. Operator, Owners and Administrators only.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`ConvertRequestBody`](schemas.md#convertrequestbody)

- `copy` *[`OutputCopy`](schemas.md#outputcopy)*, **required** — The copy to add: a format and a quality, as `output.copies` lists them. `original` is refused: a convert makes a new encoding.
- `titles` *array of string* — Title ids (uuid, ASIN, ISBN, product id) of stored books; each becomes its own job.
- `editions` *array of string* — Editions to convert (`id` of an edition in a work's detail); each becomes one job for the title that acquired it.
- `force` *boolean* — Make the copy again when the book already has it.

### Responses

- `200` — The convert jobs were accepted. Body: [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when at least one job was accepted.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — First accepted (or, when none was, already queued) job id.
  - `job_ids` *array of string*, **required** — Every accepted or already-queued job id, in plan order.
  - `error` *string or null* — Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`).
- `400` — The copy is not one a convert makes, or the body names no book. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`); string (`text/plain`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not convert these books. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ActionResponse`](schemas.md#actionresponse) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `409` — The same convert is already pending or running, or no node can run it. Body: [`PlanConflict`](schemas.md#planconflict) (`application/json`)
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
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
- `503` — Job admission is paused. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ActionResponse`](schemas.md#actionresponse) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/convert' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"copy": {"format": "m4b", "quality": "64k-mono"}}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/convert' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"copy": {"format": "m4b", "quality": "64k-mono"}}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/library/convert",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"copy": {"format": "m4b", "quality": "64k-mono"}},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/convert", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"copy": {"format": "m4b", "quality": "64k-mono"}}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/library/convert")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"copy": {"format": "m4b", "quality": "64k-mono"}}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The convert jobs were accepted.

```json
{
  "job_id": "acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "job_ids": [
    "acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
  ],
  "message": "convert job acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b accepted",
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

Status `400`: The copy is not one a convert makes, or the body names no book. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid",
  "job_id": "",
  "message": "a convert makes a new encoding; `original` is not one",
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

Status `403`: The caller may not convert these books. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `409`: The same convert is already pending or running, or no node can run it.

```json
{
  "error": "conflict",
  "job_id": "acquire-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
  "job_ids": [
    "acquire-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
  ],
  "message": "convert already pending or running",
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `copy` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

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

Status `503`: Job admission is paused. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `DELETE /api/config/nodes/{node_id}`

`DELETE /api/config/nodes/{node_id}`: remove a shared node (a shared branch, or
any node sharing this database) that is no longer running from the
library's node list. Only the operator (or an elevated Owner) may: it
deletes the node and its rows, which is hard to undo. A running node,
this node, or a branch is refused (a branch is revoked instead).

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `node_id` (path) *string*, **required** — Node id.

### Responses

- `200` — The shared node was removed. Body: [`SharedNodeRemoved`](schemas.md#sharednoderemoved) (`application/json`)
  - `node_id` *string*, **required** — Id of the node that was removed, trimmed.
  - `removed` *boolean*, **required** — Always true: a refused removal is an error status instead.
- `400` — The node is this node, or a branch (revoke it instead). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such node. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The node is still running. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes/<node_id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/nodes/<node_id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/config/nodes/<node_id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/nodes/<node_id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/config/nodes/<node_id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The shared node was removed.

```json
{
  "node_id": "node-2b8d4e",
  "removed": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "`DELETE /api/config/nodes/{node_id}` body.",
  "required": [
    "node_id",
    "removed"
  ],
  "properties": {
    "node_id": {
      "type": "string",
      "description": "Id of the node that was removed, trimmed.",
      "example": "node-2b8d4e"
    },
    "removed": {
      "type": "boolean",
      "description": "Always true: a refused removal is an error status instead.",
      "example": true
    }
  }
}
```

Status `400`: The node is this node, or a branch (revoke it instead).

```json
{
  "error": "node 9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e is a branch: revoke it instead"
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

Status `404`: No such node.

```json
{
  "error": "node 9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e not found"
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

Status `409`: The node is still running.

```json
{
  "error": "node 9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e is running; stop Bookclerk there first"
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

## `PUT /api/config/nodes/{node_id}/labels`

`PUT /api/config/nodes/{node_id}/labels`: replace a node's placement
labels. The scheduler re-plans on the wakeup this publishes.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `node_id` (path) *string*, **required** — Node id.

### Request body

`application/json`: [`PutNodeLabels`](schemas.md#putnodelabels)

- `labels` *array of string*, **required** — The node's complete operator label set.

### Responses

- `200` — The node after the change. Body: [`NodeView`](schemas.md#nodeview) (`application/json`)
  - `node_id` *string*, **required** — Node id.
  - `library_id` *string*, **required** — Library the node belongs to.
  - `created_at` *string*, **required** — RFC 3339 time the node first enrolled.
  - `heartbeat_at` *string*, **required** — RFC 3339 time of the last heartbeat the node row recorded.
  - `software_version` *string*, **required** — Bookclerk version the node runs.
  - `schema_state` *string*, **required** — Database schema the node last applied.
  - `compatible` *boolean*, **required** — True when the node's own binary accepted the schema it reported; not a comparison with this daemon's schema.
  - `target` *string*, **required** — Bookclerk target the node runs on.
  - `labels` *array of string*, **required** — Operator-set placement labels.
  - `implicit_labels` *array of string*, **required** — Labels the scheduler adds for the node (its id and target).
  - `live` *boolean*, **required** — True while the last heartbeat is recent enough.
  - `runtime` *[`NodeRuntimeSettingsV1`](schemas.md#noderuntimesettingsv1)*, **required** — The node's runtime document, or the defaults when it never wrote one.
  - `status` *[`NodeStatus`](schemas.md#nodestatus) or null* — The latest heartbeat; `null` before the first one.
  - `this_node` *boolean*, **required** — True for the node that answered the request.
  - `kind` *[`NodeKind`](schemas.md#nodekind)*, **required** — `shared` for a node sharing this library's database, `branch` for an enrolled branch.
  - `relationship` *[`BranchRelationship`](schemas.md#branchrelationship)*, **required** — How the node relates to this library: `shared_branch` (shares its database), `remote_branch` (a branch reporting a public address in its heartbeat, `status.public_origin`) or `local_branch` (a branch with none).
  - `owner_user_id` *integer (int64) or null* — User id of the node's owner; `null` when unassigned.
  - `share_books` *boolean or null* — A branch's sharing setting, as it last reported it; `null` for a shared node.
  - `include_branches` *boolean or null* — A branch's include-branches setting, as it last reported it; `null` for a shared node.
  - `owner` *[`NodeOwnerSummary`](schemas.md#nodeownersummary) or null* — The owner, when the node has one and the user still exists.
- `400` — The body is invalid, or a label is malformed. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such node. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes/<node_id>/labels' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"labels": ["region=eu", "gpu"]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/nodes/<node_id>/labels' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"labels": ["region=eu", "gpu"]}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/nodes/<node_id>/labels",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"labels": ["region=eu", "gpu"]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/nodes/<node_id>/labels", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"labels": ["region=eu", "gpu"]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/nodes/<node_id>/labels")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"labels": ["region=eu", "gpu"]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The node after the change.

```json
{
  "node_id": "node-7f3a9c",
  "library_id": "c0ffee12-3456-4789-8abc-def012345678",
  "created_at": "2026-10-01T08:00:00Z",
  "heartbeat_at": "2026-10-28T19:07:00Z",
  "software_version": "0.3.1",
  "schema_state": "frozen@12+a1b2c3d4",
  "compatible": true,
  "target": "linux-x64-gnu",
  "labels": [
    "rack=a"
  ],
  "implicit_labels": [
    "node=node-7f3a9c",
    "target=linux-x64-gnu"
  ],
  "live": true,
  "runtime": {
    "label": "Living-room server",
    "cpu_budget_percent": 80,
    "memory_budget_percent": 75,
    "schedulable": true,
    "cache_budget_bytes": 536870912
  },
  "status": {
    "node_id": "node-7f3a9c",
    "incarnation": "01J9Q0Z5C7M8N3P4R6S8T9V0W1",
    "started_at": "2026-10-28T06:00:00Z",
    "heartbeat_at": "2026-10-28T19:07:00Z",
    "heartbeat_ms": 1793214420000,
    "interval_ms": 15000,
    "cpu_millicores": 8000,
    "memory_bytes": 16777216000,
    "cpu_budget_millicores": 6400,
    "memory_budget_bytes": 12582912000,
    "schedulable": true,
    "cpu_used_millicores": 350,
    "memory_available_bytes": 9663676416,
    "files_dir_free_bytes": 214748364800,
    "reserved": {
      "memory_bytes": 1073741824,
      "cpu_millicores": 1600
    },
    "running_guests": 2,
    "availability_ppm": 998500,
    "online_since_ms": 1793167200000,
    "cache_bytes": 734003200,
    "cache_budget_bytes": 1073741824,
    "cache_swept_ms": 1793212800000,
    "public_origin": "https://branch.example"
  },
  "this_node": true,
  "kind": "shared",
  "relationship": "remote_branch",
  "owner_user_id": 7,
  "share_books": true,
  "include_branches": false,
  "owner": {
    "user_id": 7,
    "display_name": "Sam Reader",
    "role": "owner"
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "One node with its latest heartbeat, budget and owner.",
  "required": [
    "node_id",
    "library_id",
    "created_at",
    "heartbeat_at",
    "software_version",
    "schema_state",
    "compatible",
    "target",
    "labels",
    "implicit_labels",
    "live",
    "runtime",
    "this_node",
    "kind",
    "relationship"
  ],
  "properties": {
    "node_id": {
      "type": "string",
      "description": "Node id.",
      "example": "node-7f3a9c"
    },
    "library_id": {
      "type": "string",
      "description": "Library the node belongs to.",
      "example": "c0ffee12-3456-4789-8abc-def012345678"
    },
    "created_at": {
      "type": "string",
      "description": "RFC 3339 time the node first enrolled.",
      "example": "2026-10-01T08:00:00Z"
    },
    "heartbeat_at": {
      "type": "string",
      "description": "RFC 3339 time of the last heartbeat the node row recorded.",
      "example": "2026-10-28T19:07:00Z"
    },
    "software_version": {
      "type": "string",
      "description": "Bookclerk version the node runs.",
      "example": "0.3.1"
    },
    "schema_state": {
      "type": "string",
      "description": "Database schema the node last applied.",
      "example": "frozen@12+a1b2c3d4"
    },
    "compatible": {
      "type": "boolean",
      "description": "True when the node's own binary accepted the schema it reported;\nnot a comparison with this daemon's schema.",
      "example": true
    },
    "target": {
      "type": "string",
      "description": "Bookclerk target the node runs on.",
      "example": "linux-x64-gnu"
    },
    "labels": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Operator-set placement labels.",
      "example": [
        "rack=a"
      ]
    },
    "implicit_labels": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Labels the scheduler adds for the node (its id and target).",
      "example": [
        "node=node-7f3a9c",
        "target=linux-x64-gnu"
      ]
    },
    "live": {
      "type": "boolean",
      "description": "True while the last heartbeat is recent enough.",
      "example": true
    },
    "runtime": {
      "type": "object",
      "description": "The node's runtime document, or the defaults when it never wrote\none.",
      "required": [
        "label"
      ],
      "properties": {
        "label": {
          "type": "string",
          "description": "Operator label shown for this node. Empty is allowed.",
          "example": "Living-room server"
        },
        "cpu_budget_percent": {
          "type": "integer",
          "format": "int32",
          "description": "Share of detected CPU cores the scheduler may commit to plugin\nguests on this node (1..=100).",
          "example": 80,
          "maximum": 100,
          "minimum": 1
        },
        "memory_budget_percent": {
          "type": "integer",
          "format": "int32",
          "description": "Share of detected memory the scheduler may commit (1..=100).",
          "example": 75,
          "maximum": 100,
          "minimum": 1
        },
        "schedulable": {
          "type": "boolean",
          "description": "False drains the node: the scheduler places nothing new here and\nmoves its replicas elsewhere. Hand-managed deployments still run.",
          "example": true
        },
        "cache_budget_bytes": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Disk space, in bytes, this node's image caches may use together,\noverriding the library's `core.cache` budget (16 MiB to 1 TiB).\nAbsent means the library budget applies.",
          "example": 536870912,
          "maximum": 1099511627776,
          "minimum": 16777216
        }
      }
    },
    "status": {
      "oneOf": [
        {
          "type": "object",
          "description": "The latest heartbeat; `null` before the first one.",
          "required": [
            "node_id",
            "incarnation",
            "started_at",
            "heartbeat_at",
            "heartbeat_ms",
            "interval_ms",
            "cpu_millicores",
            "cpu_budget_millicores",
            "schedulable",
            "reserved",
            "running_guests",
            "availability_ppm",
            "online_since_ms"
          ],
          "properties": {
            "node_id": {
              "type": "string",
              "description": "Reporting node.",
              "example": "node-7f3a9c"
            },
            "incarnation": {
              "type": "string",
              "description": "Daemon process incarnation.",
              "example": "01J9Q0Z5C7M8N3P4R6S8T9V0W1"
            },
            "started_at": {
              "type": "string",
              "description": "RFC 3339 daemon start.",
              "example": "2026-10-28T06:00:00Z"
            },
            "heartbeat_at": {
              "type": "string",
              "description": "RFC 3339 heartbeat time.",
              "example": "2026-10-28T19:07:00Z"
            },
            "heartbeat_ms": {
              "type": "integer",
              "format": "int64",
              "description": "Heartbeat time in Unix milliseconds.",
              "example": 1793214420000
            },
            "interval_ms": {
              "type": "integer",
              "format": "int64",
              "description": "Promised interval in milliseconds.",
              "example": 15000
            },
            "cpu_millicores": {
              "type": "integer",
              "format": "int64",
              "description": "Detected CPU capacity in millicores.",
              "example": 8000
            },
            "memory_bytes": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Detected memory in bytes.",
              "example": 16777216000
            },
            "cpu_budget_millicores": {
              "type": "integer",
              "format": "int64",
              "description": "CPU the scheduler may commit, in millicores.",
              "example": 6400
            },
            "memory_budget_bytes": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Memory the scheduler may commit, in bytes.",
              "example": 12582912000
            },
            "schedulable": {
              "type": "boolean",
              "description": "False when an operator drained the node.",
              "example": true
            },
            "cpu_used_millicores": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Measured CPU in use, in millicores.",
              "example": 350
            },
            "memory_available_bytes": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Measured memory available, in bytes.",
              "example": 9663676416
            },
            "files_dir_free_bytes": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Free bytes on the files directory's filesystem.",
              "example": 214748364800
            },
            "reserved": {
              "type": "object",
              "description": "Ceilings of the guests running here.",
              "required": [
                "memory_bytes",
                "cpu_millicores"
              ],
              "properties": {
                "memory_bytes": {
                  "type": "integer",
                  "format": "int64",
                  "description": "Memory ceiling in bytes.",
                  "example": 1073741824
                },
                "cpu_millicores": {
                  "type": "integer",
                  "format": "int64",
                  "description": "CPU ceiling in millicores.",
                  "example": 1600
                }
              }
            },
            "running_guests": {
              "type": "integer",
              "format": "int64",
              "description": "Guests running here.",
              "example": 2
            },
            "availability_ppm": {
              "type": "integer",
              "format": "int64",
              "description": "Recent online fraction in parts per million.",
              "example": 998500
            },
            "online_since_ms": {
              "type": "integer",
              "format": "int64",
              "description": "Unix milliseconds the current online stretch began.",
              "example": 1793167200000
            },
            "cache_bytes": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Bytes the node's image caches hold; `None` from a node that keeps\nnone.",
              "example": 734003200
            },
            "cache_budget_bytes": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Bytes the node's image caches may hold.",
              "example": 1073741824
            },
            "cache_swept_ms": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "When the node's last cache sweep finished, in Unix milliseconds.",
              "example": 1793212800000
            },
            "public_origin": {
              "type": [
                "string",
                "null"
              ],
              "description": "The address other installs reach this branch at, as it last\nreported it; `null` for a local branch and for a shared node. A\nbranch with one is a remote branch of this library.",
              "example": "https://branch.example"
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "this_node": {
      "type": "boolean",
      "description": "True for the node that answered the request.",
      "example": true
    },
    "kind": {
      "type": "string",
      "description": "`shared` for a node sharing this library's database, `branch` for an\nenrolled branch.",
      "enum": [
        "shared",
        "branch"
      ],
      "example": "shared"
    },
    "relationship": {
      "type": "string",
      "description": "How the node relates to this library: `shared_branch` (shares its\ndatabase), `remote_branch` (a branch reporting a public address in\nits heartbeat, `status.public_origin`) or `local_branch` (a branch\nwith none).",
      "enum": [
        "shared_branch",
        "remote_branch",
        "local_branch"
      ],
      "example": "remote_branch"
    },
    "owner_user_id": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "User id of the node's owner; `null` when unassigned.",
      "example": 7
    },
    "share_books": {
      "type": [
        "boolean",
        "null"
      ],
      "description": "A branch's sharing setting, as it last reported it; `null` for a\nshared node.",
      "example": true
    },
    "include_branches": {
      "type": [
        "boolean",
        "null"
      ],
      "description": "A branch's include-branches setting, as it last reported it;\n`null` for a shared node.",
      "example": false
    },
    "owner": {
      "oneOf": [
        {
          "type": "object",
          "description": "The owner, when the node has one and the user still exists.",
          "required": [
            "user_id",
            "role"
          ],
          "properties": {
            "user_id": {
              "type": "integer",
              "format": "int64",
              "description": "The owner's user id.",
              "example": 7
            },
            "display_name": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display name, or the login name when none is set; `null` when the\nuser has neither.",
              "example": "Sam Reader"
            },
            "role": {
              "type": "string",
              "description": "The owner's role.",
              "enum": [
                "owner",
                "administrator",
                "member"
              ]
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

Status `400`: The body is invalid, or a label is malformed. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid configuration: label `Region=EU` must be `name` or `name=value` using lowercase letters, digits, and `._/-` (at most 63 bytes)"
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

Status `404`: No such node.

```json
{
  "error": "node 9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e not found"
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
Failed to deserialize the JSON body into the target type: missing field `labels` at line 1 column 2
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

## `PUT /api/config/nodes/{node_id}/owner`

`PUT /api/config/nodes/{node_id}/owner`: assign or clear a node's owner.
The owner may then edit that node's runtime settings without operator
authority.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `node_id` (path) *string*, **required** — Node id.

### Request body

`application/json`: [`PutNodeOwner`](schemas.md#putnodeowner)

- `user_id` *integer (int64) or null* — User who owns the node, or `null` to clear.

### Responses

- `200` — The node's owner after the change. Body: [`NodeOwnerChanged`](schemas.md#nodeownerchanged) (`application/json`)
  - `node_id` *string*, **required** — Node id.
  - `owner_user_id` *integer (int64) or null* — User id of the owner; `null` when cleared.
- `400` — The user is disabled and cannot own a node. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such node or user. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes/<node_id>/owner' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/nodes/<node_id>/owner' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/nodes/<node_id>/owner",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/nodes/<node_id>/owner", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/nodes/<node_id>/owner")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The node's owner after the change.

```json
{
  "node_id": "node-7f3a9c",
  "owner_user_id": 7
}
```

Schema:

```json
{
  "type": "object",
  "description": "`PUT /api/config/nodes/{node_id}/owner` body: the owner after the\nchange.",
  "required": [
    "node_id"
  ],
  "properties": {
    "node_id": {
      "type": "string",
      "description": "Node id.",
      "example": "node-7f3a9c"
    },
    "owner_user_id": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "User id of the owner; `null` when cleared.",
      "example": 7
    }
  }
}
```

Status `400`: The user is disabled and cannot own a node. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "user 42 is disabled and cannot own a node"
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

Status `404`: No such node or user.

```json
{
  "error": "user 42 not found"
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
Failed to deserialize the JSON body into the target type: invalid type: null, expected struct PutNodeOwner at line 1 column 4
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

## `GET /api/config/library-consents`

`GET /api/config/library-consents`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — Pending and granted library consents. Body: [`LibraryConsentsResponse`](schemas.md#libraryconsentsresponse) (`application/json`)
  - `libraries` *array of [`LibraryLinkView`](schemas.md#librarylinkview)*, **required** — Every library this install is a branch of.
  - `consents` *array of [`RemoteConsentView`](schemas.md#remoteconsentview)*, **required** — Every plugin request from those libraries, pending or answered.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/library-consents' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/library-consents' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/library-consents",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/library-consents", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/library-consents")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Pending and granted library consents.

```json
{
  "libraries": [
    {
      "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
      "owner_origin": "https://books.example.com",
      "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
      "pinned": false,
      "state": "active",
      "enrolled_at": "2026-10-29T09:02:41Z",
      "share_books": true,
      "include_branches": true,
      "work_for_members": false,
      "keep_member_copies": false,
      "max_jobs": 2,
      "job_share": 2,
      "list_members_books": true,
      "keep_copies": true,
      "running": true
    }
  ],
  "consents": [
    {
      "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
      "plugin_key": "bookclerk:source-libro",
      "requested_version": "1.4.2",
      "requested_grant": {
        "addressCidrs": [],
        "allowUndeclaredPublicRedirects": false,
        "approvedAt": "2026-10-29T10:12:00Z",
        "bindings": [
          "config",
          "secrets"
        ],
        "compatibilityFlags": [],
        "consumers": [],
        "domains": [
          "api.libro.fm",
          "cdn.libro.fm"
        ],
        "entrypoints": [
          "storefront"
        ],
        "jobs": [],
        "manifestCidrs": [],
        "manifestDomains": [
          "api.libro.fm",
          "cdn.libro.fm"
        ],
        "manifestTcp": [],
        "networkMode": "outbound",
        "operatorAddedCidrs": [],
        "operatorAddedDomains": [],
        "operatorAddedTcp": [],
        "operatorDeniedCidrs": [],
        "operatorDeniedDomains": [],
        "operatorDeniedTcp": [],
        "pluginId": "libro",
        "pluginKey": "bookclerk:source-libro",
        "producers": [],
        "schemaVersion": 2,
        "tcp": []
      },
      "requested_artifact": {
        "url": "https://books.example.com/api/plugins/artifacts/9c1e4f7a",
        "archive_sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
        "payload_root_sha256": "a7ffc6f8bf1ed76651c14756a061d662f580ff4de43b49fa82d80a4b80f8434a"
      },
      "request_digest": "9c1e4f7a2b3d5e6f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f",
      "approved_version": "1.4.2",
      "grant": {
        "addressCidrs": [],
        "allowUndeclaredPublicRedirects": false,
        "approvedAt": "2026-10-29T10:12:00Z",
        "bindings": [
          "config",
          "secrets"
        ],
        "compatibilityFlags": [],
        "consumers": [],
        "domains": [
          "api.libro.fm",
          "cdn.libro.fm"
        ],
        "entrypoints": [
          "storefront"
        ],
        "jobs": [],
        "manifestCidrs": [],
        "manifestDomains": [
          "api.libro.fm",
          "cdn.libro.fm"
        ],
        "manifestTcp": [],
        "networkMode": "outbound",
        "operatorAddedCidrs": [],
        "operatorAddedDomains": [],
        "operatorAddedTcp": [],
        "operatorDeniedCidrs": [],
        "operatorDeniedDomains": [],
        "operatorDeniedTcp": [],
        "pluginId": "libro",
        "pluginKey": "bookclerk:source-libro",
        "producers": [],
        "schemaVersion": 2,
        "tcp": []
      },
      "approved_at": "2026-10-29T10:12:00Z",
      "approved_by": "operator",
      "revoked_at": "2026-11-01T08:00:00Z",
      "updated_at": "2026-10-29T10:12:00Z",
      "state": "consented",
      "pending": [
        "Awaiting consent (not automatically approved):",
        "  fetch hosts: any public website (*)"
      ]
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/config/library-consents`.",
  "required": [
    "libraries",
    "consents"
  ],
  "properties": {
    "libraries": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A library this install is a branch of: ids, origin and state, never key material.",
        "required": [
          "library_id",
          "owner_origin",
          "node_id",
          "pinned",
          "state",
          "enrolled_at",
          "share_books",
          "include_branches",
          "work_for_members",
          "keep_member_copies",
          "list_members_books",
          "keep_copies"
        ],
        "properties": {
          "library_id": {
            "type": "string",
            "description": "The library id.",
            "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
          },
          "owner_origin": {
            "type": "string",
            "description": "The library's canonical origin.",
            "example": "https://books.example.com"
          },
          "node_id": {
            "type": "string",
            "description": "The node id the library minted for this install.",
            "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
          },
          "pinned": {
            "type": "boolean",
            "description": "Whether the link pins the library's self-signed certificate.",
            "example": false
          },
          "state": {
            "type": "string",
            "description": "`active`, `revoked`, or `left`.",
            "example": "active"
          },
          "enrolled_at": {
            "type": "string",
            "description": "RFC 3339 enrollment time.",
            "example": "2026-10-29T09:02:41Z"
          },
          "share_books": {
            "type": "boolean",
            "description": "\"Share this branch's books with the library\".",
            "example": true
          },
          "include_branches": {
            "type": "boolean",
            "description": "\"Include books from my own branches\".",
            "example": true
          },
          "work_for_members": {
            "type": "boolean",
            "description": "\"Download for other members of the library\": this install takes the library's work for its other members too, not only its owner's (off by default).",
            "example": false
          },
          "keep_member_copies": {
            "type": "boolean",
            "description": "\"Keep copies of other members' books\": a book acquired here for another member is also written to this install's own destinations (off by default).",
            "example": false
          },
          "max_jobs": {
            "type": [
              "integer",
              "null"
            ],
            "description": "The most jobs this install runs for the library at once, as its operator set it; `null` gives the library an equal share of the machine.",
            "example": 2,
            "maximum": 16,
            "minimum": 0
          },
          "job_share": {
            "type": [
              "integer",
              "null"
            ],
            "description": "How many jobs the library may have running here now: its share of the machine after this install's own work, within `max_jobs`. Only the listing and sharing routes report it, and only while the link's client runs.",
            "example": 2,
            "minimum": 0
          },
          "list_members_books": {
            "type": "boolean",
            "description": "\"List my members' books to the library\": the library's catalog\nlookups are answered with members' books (except those who opted\nout) as well as the owner's.",
            "example": true
          },
          "keep_copies": {
            "type": "boolean",
            "description": "\"Keep copies on this computer\": the branch takes the library's\ncopies of its person's books into its own storage. The branch takes\nthe library's work and shows its books whether or not this is on.",
            "example": true
          },
          "running": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "Whether this install's client for the link is running now; only the join routes report it.",
            "example": true
          }
        }
      },
      "description": "Every library this install is a branch of."
    },
    "consents": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "An owner library's request to run a plugin on this install, and what the operator answered.",
        "required": [
          "library_id",
          "plugin_key",
          "requested_version",
          "requested_grant",
          "requested_artifact",
          "request_digest",
          "grant",
          "updated_at",
          "state",
          "pending"
        ],
        "properties": {
          "library_id": {
            "type": "string",
            "description": "The parent library's id.",
            "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
          },
          "plugin_key": {
            "type": "string",
            "description": "Canonical plugin key.",
            "example": "bookclerk:source-libro"
          },
          "requested_version": {
            "type": "string",
            "description": "Version the owner deploys here now.",
            "example": "1.4.2"
          },
          "requested_grant": {
            "description": "A plugin grant document (the plugin host's `PluginGrant`): the plugin key, the entrypoints, network mode and domains, and the bindings it may use; `null` when nothing was approved.",
            "examples": [
              {
                "addressCidrs": [],
                "allowUndeclaredPublicRedirects": false,
                "approvedAt": "2026-10-29T10:12:00Z",
                "bindings": [
                  "config",
                  "secrets"
                ],
                "compatibilityFlags": [],
                "consumers": [],
                "domains": [
                  "api.libro.fm",
                  "cdn.libro.fm"
                ],
                "entrypoints": [
                  "storefront"
                ],
                "jobs": [],
                "manifestCidrs": [],
                "manifestDomains": [
                  "api.libro.fm",
                  "cdn.libro.fm"
                ],
                "manifestTcp": [],
                "networkMode": "outbound",
                "operatorAddedCidrs": [],
                "operatorAddedDomains": [],
                "operatorAddedTcp": [],
                "operatorDeniedCidrs": [],
                "operatorDeniedDomains": [],
                "operatorDeniedTcp": [],
                "pluginId": "libro",
                "pluginKey": "bookclerk:source-libro",
                "producers": [],
                "schemaVersion": 2,
                "tcp": []
              }
            ]
          },
          "requested_artifact": {
            "type": "object",
            "description": "The archive the owner deploys to this install.",
            "required": [
              "url",
              "archive_sha256",
              "payload_root_sha256"
            ],
            "properties": {
              "url": {
                "type": "string",
                "description": "Where the owner fetches it from.",
                "example": "https://books.example.com/api/plugins/artifacts/9c1e4f7a"
              },
              "archive_sha256": {
                "type": "string",
                "description": "SHA-256 of the archive.",
                "example": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
              },
              "payload_root_sha256": {
                "type": "string",
                "description": "Payload-root digest of the tree it unpacks to.",
                "example": "a7ffc6f8bf1ed76651c14756a061d662f580ff4de43b49fa82d80a4b80f8434a"
              }
            }
          },
          "request_digest": {
            "type": "string",
            "description": "Digest of the request as a whole; an approval names it so a changed request is not approved by mistake.",
            "example": "9c1e4f7a2b3d5e6f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f"
          },
          "approved_version": {
            "type": [
              "string",
              "null"
            ],
            "description": "Version this install's operator approved, when any.",
            "example": "1.4.2"
          },
          "grant": {
            "description": "A plugin grant document (the plugin host's `PluginGrant`): the plugin key, the entrypoints, network mode and domains, and the bindings it may use; `null` when nothing was approved.",
            "examples": [
              {
                "addressCidrs": [],
                "allowUndeclaredPublicRedirects": false,
                "approvedAt": "2026-10-29T10:12:00Z",
                "bindings": [
                  "config",
                  "secrets"
                ],
                "compatibilityFlags": [],
                "consumers": [],
                "domains": [
                  "api.libro.fm",
                  "cdn.libro.fm"
                ],
                "entrypoints": [
                  "storefront"
                ],
                "jobs": [],
                "manifestCidrs": [],
                "manifestDomains": [
                  "api.libro.fm",
                  "cdn.libro.fm"
                ],
                "manifestTcp": [],
                "networkMode": "outbound",
                "operatorAddedCidrs": [],
                "operatorAddedDomains": [],
                "operatorAddedTcp": [],
                "operatorDeniedCidrs": [],
                "operatorDeniedDomains": [],
                "operatorDeniedTcp": [],
                "pluginId": "libro",
                "pluginKey": "bookclerk:source-libro",
                "producers": [],
                "schemaVersion": 2,
                "tcp": []
              }
            ]
          },
          "approved_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "RFC 3339 approval time.",
            "example": "2026-10-29T10:12:00Z"
          },
          "approved_by": {
            "type": [
              "string",
              "null"
            ],
            "description": "Who approved (`operator`, or `platform` for an automatic approval).",
            "example": "operator"
          },
          "revoked_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "RFC 3339 time the operator revoked the approval.",
            "example": "2026-11-01T08:00:00Z"
          },
          "updated_at": {
            "type": "string",
            "description": "RFC 3339 time of the last write.",
            "example": "2026-10-29T10:12:00Z"
          },
          "state": {
            "type": "string",
            "description": "How this host stands on the request: `consented`, `awaiting_host_consent` (no approval, or a revoked one), `awaiting_host_reapproval` (the owner changed the request or the archive since the approval), or `invalid_grant` (a grant that does not parse).",
            "example": "consented"
          },
          "pending": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "What the owner's current request asks for beyond the request the operator approved, as the operator-facing lines the CLI prints; empty when nothing was approved yet or the request stays within the approval.",
            "example": [
              "Awaiting consent (not automatically approved):",
              "  fetch hosts: any public website (*)"
            ]
          }
        }
      },
      "description": "Every plugin request from those libraries, pending or answered."
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

Status `403`: The session is impersonating a user.

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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/config/libraries/{library_id}/scan`

`POST /api/config/libraries/{library_id}/scan`: asks the library
to queue a scan for this node's owner (H-5). The owner's planner decides
what that user may scan; the answer lists the queued job ids.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `library_id` (path) *string*, **required** — Library id.

### Request body

Optional.

`application/json`: [`LibraryScanBody`](schemas.md#libraryscanbody) or null

- `account` *string or null* — Scan only this store account of the library.
- `plugin_instance_id` *string or null* — Pin every planned job to this owner plugin instance.

### Responses

- `200` — The library accepted the scan. Body: [`LibraryScanPlanned`](schemas.md#libraryscanplanned) (`application/json`)
  - `library_id` *string*, **required** — The parent library's id.
  - `created` *array of string*, **required** — Ids of the jobs the owner created.
  - `duplicates` *array of string*, **required** — Ids of jobs that were already pending or running.
  - `queue_full` *integer (int64)*, **required** — Planned jobs the owner's queue had no room for. At least 0.
- `400` — The account named is blank, or the owner refused the request as invalid (`invalid`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The owner refused the scan (`forbidden`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The library answered with an unexpected frame (`unexpected_answer`), or with an error code other than `forbidden` and `invalid`. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `503` — This daemon has no running link to that library (`not_connected`), or the owner did not answer (`owner_unavailable`). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/libraries/<library_id>/scan' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/libraries/<library_id>/scan' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/libraries/<library_id>/scan",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/libraries/<library_id>/scan", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/libraries/<library_id>/scan")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The library accepted the scan.

```json
{
  "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
  "created": [
    "scan-7f3a9c1e"
  ],
  "duplicates": [],
  "queue_full": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after the owner library queued a scan for this install.",
  "required": [
    "library_id",
    "created",
    "duplicates",
    "queue_full"
  ],
  "properties": {
    "library_id": {
      "type": "string",
      "description": "The parent library's id.",
      "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
    },
    "created": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Ids of the jobs the owner created.",
      "example": [
        "scan-7f3a9c1e"
      ]
    },
    "duplicates": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Ids of jobs that were already pending or running.",
      "example": []
    },
    "queue_full": {
      "type": "integer",
      "format": "int64",
      "description": "Planned jobs the owner's queue had no room for.",
      "example": 0,
      "minimum": 0
    }
  }
}
```

Status `400`: The account named is blank, or the owner refused the request as invalid (`invalid`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "the account named is blank",
  "error": "invalid"
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

Status `403`: The owner refused the scan (`forbidden`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: account: invalid type: integer `1`, expected a string at line 1 column 12
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

Status `502`: The library answered with an unexpected frame (`unexpected_answer`), or with an error code other than `forbidden` and `invalid`.

```json
{
  "detail": "the scan queue is full",
  "error": "unavailable"
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

Status `503`: This daemon has no running link to that library (`not_connected`), or the owner did not answer (`owner_unavailable`).

```json
{
  "detail": "this daemon has no running link to that library",
  "error": "not_connected"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/config/libraries/{library_id}/jobs`

`POST /api/config/libraries/{library_id}/jobs`: the owner's view of jobs
this node may see (those it would be trusted to claim).

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `library_id` (path) *string*, **required** — Library id.

### Request body

`application/json`: [`LibraryJobsBody`](schemas.md#libraryjobsbody)

- `job_ids` *array of string*, **required** — Job ids on the library.

### Responses

- `200` — The owner's jobs for these accounts. Body: [`LibraryJobsResponse`](schemas.md#libraryjobsresponse) (`application/json`)
  - `jobs` *array of [`JobStatusEntry`](schemas.md#jobstatusentry)*, **required** — The owner's state of each job this install may see, in request order.
- `400` — Too many job ids (`too_many_jobs`), or the owner refused the request as invalid (`invalid`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`); string (`text/plain`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The owner refused the request (`forbidden`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The library answered with an unexpected frame (`unexpected_answer`), or with an error code other than `forbidden` and `invalid`. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `503` — This daemon has no running link to that library (`not_connected`), or the owner did not answer (`owner_unavailable`). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/libraries/<library_id>/jobs' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"job_ids": ["acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/libraries/<library_id>/jobs' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"job_ids": ["acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"]}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/libraries/<library_id>/jobs",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"job_ids": ["acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/libraries/<library_id>/jobs", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"job_ids": ["acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/libraries/<library_id>/jobs")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"job_ids": ["acquire-3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The owner's jobs for these accounts.

```json
{
  "jobs": [
    {
      "id": "scan-7f3a9c1e",
      "kind": "scan",
      "state": "running",
      "progress": "12 of 40 titles",
      "error_kind": "handler",
      "error_message": "the store answered 503"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `POST /api/config/libraries/{library_id}/jobs`.",
  "required": [
    "jobs"
  ],
  "properties": {
    "jobs": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One job's state in [`Body::JobStatuses`].",
        "required": [
          "id",
          "kind",
          "state"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Job id.",
            "example": "scan-7f3a9c1e"
          },
          "kind": {
            "type": "string",
            "description": "Job kind (`scan`, `acquire`, ...).",
            "example": "scan"
          },
          "state": {
            "type": "string",
            "description": "`pending`, `running`, `succeeded`, `failed`, or `cancelled`.",
            "example": "running"
          },
          "progress": {
            "type": [
              "string",
              "null"
            ],
            "description": "Progress or result text.",
            "example": "12 of 40 titles"
          },
          "error_kind": {
            "type": [
              "string",
              "null"
            ],
            "description": "Structured error kind when it failed or was cancelled.",
            "example": "handler"
          },
          "error_message": {
            "type": [
              "string",
              "null"
            ],
            "description": "Error text when it failed or was cancelled.",
            "example": "the store answered 503"
          }
        }
      },
      "description": "The owner's state of each job this install may see, in request order."
    }
  }
}
```

Status `400`: Too many job ids (`too_many_jobs`), or the owner refused the request as invalid (`invalid`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "too many job ids",
  "error": "invalid"
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

Status `403`: The owner refused the request (`forbidden`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `job_ids` at line 1 column 2
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

Status `502`: The library answered with an unexpected frame (`unexpected_answer`), or with an error code other than `forbidden` and `invalid`.

```json
{
  "detail": "the job store is busy",
  "error": "unavailable"
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

Status `503`: This daemon has no running link to that library (`not_connected`), or the owner did not answer (`owner_unavailable`).

```json
{
  "detail": "this daemon has no running link to that library",
  "error": "not_connected"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/config/library-consents/approve`

`POST /api/config/library-consents/approve`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`ApproveBody`](schemas.md#approvebody)

- `library_id` *string*, **required** — Library id.
- `plugin_key` *string*, **required** — Canonical plugin key.
- `request_digest` *string*, **required** — The `request_digest` of the request the operator reviewed, as the listing shows it (refused when the recorded request is another one).
- `deny_domains` *array of string* — Domains to take out of the request. No duplicates.
- `deny_bindings` *array of string* — Bindings to take out of the request. No duplicates.
- `deny_own_user_agent_hosts` *array of string* — Requested hosts to refuse the plugin's own User-Agent on; requests to them carry Bookclerk's agent instead. No duplicates.
- `deny_network` *boolean* — Deny all network access.

### Responses

- `200` — The consent was granted. Body: [`ConsentChanged`](schemas.md#consentchanged) (`application/json`)
  - `consent` *[`RemoteConsentView`](schemas.md#remoteconsentview)*, **required** — The consent as stored now.
- `400` — The narrowing or the recorded request is not valid (`invalid_approval`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — That library has not asked to run that plugin here (`not_requested`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `409` — The request changed since it was reviewed; review it again (`approval_refused`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `503` — The approval could not be stored; try again (`store_unavailable`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/library-consents/approve' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "request_digest": "9b74c9897bac770ffc029102a200c5de3a9c4a1d6c2f1e0b5a7d8c9e0f1a2b3c"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/library-consents/approve' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "request_digest": "9b74c9897bac770ffc029102a200c5de3a9c4a1d6c2f1e0b5a7d8c9e0f1a2b3c"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/library-consents/approve",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "request_digest": "9b74c9897bac770ffc029102a200c5de3a9c4a1d6c2f1e0b5a7d8c9e0f1a2b3c"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/library-consents/approve", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "request_digest": "9b74c9897bac770ffc029102a200c5de3a9c4a1d6c2f1e0b5a7d8c9e0f1a2b3c"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/library-consents/approve")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "request_digest": "9b74c9897bac770ffc029102a200c5de3a9c4a1d6c2f1e0b5a7d8c9e0f1a2b3c"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The consent was granted.

```json
{
  "consent": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "plugin_key": "bookclerk:source-libro",
    "requested_version": "1.4.2",
    "requested_grant": {
      "addressCidrs": [],
      "allowUndeclaredPublicRedirects": false,
      "approvedAt": "2026-10-29T10:12:00Z",
      "bindings": [
        "config",
        "secrets"
      ],
      "compatibilityFlags": [],
      "consumers": [],
      "domains": [
        "api.libro.fm",
        "cdn.libro.fm"
      ],
      "entrypoints": [
        "storefront"
      ],
      "jobs": [],
      "manifestCidrs": [],
      "manifestDomains": [
        "api.libro.fm",
        "cdn.libro.fm"
      ],
      "manifestTcp": [],
      "networkMode": "outbound",
      "operatorAddedCidrs": [],
      "operatorAddedDomains": [],
      "operatorAddedTcp": [],
      "operatorDeniedCidrs": [],
      "operatorDeniedDomains": [],
      "operatorDeniedTcp": [],
      "pluginId": "libro",
      "pluginKey": "bookclerk:source-libro",
      "producers": [],
      "schemaVersion": 2,
      "tcp": []
    },
    "requested_artifact": {
      "url": "https://books.example.com/api/plugins/artifacts/9c1e4f7a",
      "archive_sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
      "payload_root_sha256": "a7ffc6f8bf1ed76651c14756a061d662f580ff4de43b49fa82d80a4b80f8434a"
    },
    "request_digest": "9c1e4f7a2b3d5e6f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f",
    "approved_version": "1.4.2",
    "grant": {
      "addressCidrs": [],
      "allowUndeclaredPublicRedirects": false,
      "approvedAt": "2026-10-29T10:12:00Z",
      "bindings": [
        "config",
        "secrets"
      ],
      "compatibilityFlags": [],
      "consumers": [],
      "domains": [
        "api.libro.fm",
        "cdn.libro.fm"
      ],
      "entrypoints": [
        "storefront"
      ],
      "jobs": [],
      "manifestCidrs": [],
      "manifestDomains": [
        "api.libro.fm",
        "cdn.libro.fm"
      ],
      "manifestTcp": [],
      "networkMode": "outbound",
      "operatorAddedCidrs": [],
      "operatorAddedDomains": [],
      "operatorAddedTcp": [],
      "operatorDeniedCidrs": [],
      "operatorDeniedDomains": [],
      "operatorDeniedTcp": [],
      "pluginId": "libro",
      "pluginKey": "bookclerk:source-libro",
      "producers": [],
      "schemaVersion": 2,
      "tcp": []
    },
    "approved_at": "2026-10-29T10:12:00Z",
    "approved_by": "operator",
    "revoked_at": "2026-11-01T08:00:00Z",
    "updated_at": "2026-10-29T10:12:00Z",
    "state": "consented",
    "pending": [
      "Awaiting consent (not automatically approved):",
      "  fetch hosts: any public website (*)"
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after a consent was approved or revoked.",
  "required": [
    "consent"
  ],
  "properties": {
    "consent": {
      "type": "object",
      "description": "The consent as stored now.",
      "required": [
        "library_id",
        "plugin_key",
        "requested_version",
        "requested_grant",
        "requested_artifact",
        "request_digest",
        "grant",
        "updated_at",
        "state",
        "pending"
      ],
      "properties": {
        "library_id": {
          "type": "string",
          "description": "The parent library's id.",
          "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
        },
        "plugin_key": {
          "type": "string",
          "description": "Canonical plugin key.",
          "example": "bookclerk:source-libro"
        },
        "requested_version": {
          "type": "string",
          "description": "Version the owner deploys here now.",
          "example": "1.4.2"
        },
        "requested_grant": {
          "description": "A plugin grant document (the plugin host's `PluginGrant`): the plugin key, the entrypoints, network mode and domains, and the bindings it may use; `null` when nothing was approved.",
          "examples": [
            {
              "addressCidrs": [],
              "allowUndeclaredPublicRedirects": false,
              "approvedAt": "2026-10-29T10:12:00Z",
              "bindings": [
                "config",
                "secrets"
              ],
              "compatibilityFlags": [],
              "consumers": [],
              "domains": [
                "api.libro.fm",
                "cdn.libro.fm"
              ],
              "entrypoints": [
                "storefront"
              ],
              "jobs": [],
              "manifestCidrs": [],
              "manifestDomains": [
                "api.libro.fm",
                "cdn.libro.fm"
              ],
              "manifestTcp": [],
              "networkMode": "outbound",
              "operatorAddedCidrs": [],
              "operatorAddedDomains": [],
              "operatorAddedTcp": [],
              "operatorDeniedCidrs": [],
              "operatorDeniedDomains": [],
              "operatorDeniedTcp": [],
              "pluginId": "libro",
              "pluginKey": "bookclerk:source-libro",
              "producers": [],
              "schemaVersion": 2,
              "tcp": []
            }
          ]
        },
        "requested_artifact": {
          "type": "object",
          "description": "The archive the owner deploys to this install.",
          "required": [
            "url",
            "archive_sha256",
            "payload_root_sha256"
          ],
          "properties": {
            "url": {
              "type": "string",
              "description": "Where the owner fetches it from.",
              "example": "https://books.example.com/api/plugins/artifacts/9c1e4f7a"
            },
            "archive_sha256": {
              "type": "string",
              "description": "SHA-256 of the archive.",
              "example": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
            },
            "payload_root_sha256": {
              "type": "string",
              "description": "Payload-root digest of the tree it unpacks to.",
              "example": "a7ffc6f8bf1ed76651c14756a061d662f580ff4de43b49fa82d80a4b80f8434a"
            }
          }
        },
        "request_digest": {
          "type": "string",
          "description": "Digest of the request as a whole; an approval names it so a changed request is not approved by mistake.",
          "example": "9c1e4f7a2b3d5e6f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f"
        },
        "approved_version": {
          "type": [
            "string",
            "null"
          ],
          "description": "Version this install's operator approved, when any.",
          "example": "1.4.2"
        },
        "grant": {
          "description": "A plugin grant document (the plugin host's `PluginGrant`): the plugin key, the entrypoints, network mode and domains, and the bindings it may use; `null` when nothing was approved.",
          "examples": [
            {
              "addressCidrs": [],
              "allowUndeclaredPublicRedirects": false,
              "approvedAt": "2026-10-29T10:12:00Z",
              "bindings": [
                "config",
                "secrets"
              ],
              "compatibilityFlags": [],
              "consumers": [],
              "domains": [
                "api.libro.fm",
                "cdn.libro.fm"
              ],
              "entrypoints": [
                "storefront"
              ],
              "jobs": [],
              "manifestCidrs": [],
              "manifestDomains": [
                "api.libro.fm",
                "cdn.libro.fm"
              ],
              "manifestTcp": [],
              "networkMode": "outbound",
              "operatorAddedCidrs": [],
              "operatorAddedDomains": [],
              "operatorAddedTcp": [],
              "operatorDeniedCidrs": [],
              "operatorDeniedDomains": [],
              "operatorDeniedTcp": [],
              "pluginId": "libro",
              "pluginKey": "bookclerk:source-libro",
              "producers": [],
              "schemaVersion": 2,
              "tcp": []
            }
          ]
        },
        "approved_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "RFC 3339 approval time.",
          "example": "2026-10-29T10:12:00Z"
        },
        "approved_by": {
          "type": [
            "string",
            "null"
          ],
          "description": "Who approved (`operator`, or `platform` for an automatic approval).",
          "example": "operator"
        },
        "revoked_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "RFC 3339 time the operator revoked the approval.",
          "example": "2026-11-01T08:00:00Z"
        },
        "updated_at": {
          "type": "string",
          "description": "RFC 3339 time of the last write.",
          "example": "2026-10-29T10:12:00Z"
        },
        "state": {
          "type": "string",
          "description": "How this host stands on the request: `consented`, `awaiting_host_consent` (no approval, or a revoked one), `awaiting_host_reapproval` (the owner changed the request or the archive since the approval), or `invalid_grant` (a grant that does not parse).",
          "example": "consented"
        },
        "pending": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "What the owner's current request asks for beyond the request the operator approved, as the operator-facing lines the CLI prints; empty when nothing was approved yet or the request stays within the approval.",
          "example": [
            "Awaiting consent (not automatically approved):",
            "  fetch hosts: any public website (*)"
          ]
        }
      }
    }
  }
}
```

Status `400`: The narrowing or the recorded request is not valid (`invalid_approval`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "the recorded request does not parse",
  "error": "invalid_approval"
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

Status `404`: That library has not asked to run that plugin here (`not_requested`).

```json
{
  "detail": "library c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f has not asked to run bookclerk:source-libro here",
  "error": "not_requested"
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

Status `409`: The request changed since it was reviewed; review it again (`approval_refused`).

```json
{
  "detail": "the request for bookclerk:source-libro changed since it was reviewed (now version 1.4.3, request 9c1e4f7a2b3d5e6f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f); review it again",
  "error": "approval_refused"
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

Status `503`: The approval could not be stored; try again (`store_unavailable`).

```json
{
  "detail": "the approval could not be stored; try again",
  "error": "store_unavailable"
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

## `POST /api/config/library-consents/revoke`

`POST /api/config/library-consents/revoke`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`RevokeBody`](schemas.md#revokebody)

- `library_id` *string*, **required** — Library id.
- `plugin_key` *string*, **required** — Canonical plugin key.

### Responses

- `200` — The consent was revoked. Body: [`ConsentChanged`](schemas.md#consentchanged) (`application/json`)
  - `consent` *[`RemoteConsentView`](schemas.md#remoteconsentview)*, **required** — The consent as stored now.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such consent (`not_found`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/library-consents/revoke' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/library-consents/revoke' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/library-consents/revoke",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/library-consents/revoke", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/library-consents/revoke")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The consent was revoked.

```json
{
  "consent": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "plugin_key": "bookclerk:source-libro",
    "requested_version": "1.4.2",
    "requested_grant": {
      "addressCidrs": [],
      "allowUndeclaredPublicRedirects": false,
      "approvedAt": "2026-10-29T10:12:00Z",
      "bindings": [
        "config",
        "secrets"
      ],
      "compatibilityFlags": [],
      "consumers": [],
      "domains": [
        "api.libro.fm",
        "cdn.libro.fm"
      ],
      "entrypoints": [
        "storefront"
      ],
      "jobs": [],
      "manifestCidrs": [],
      "manifestDomains": [
        "api.libro.fm",
        "cdn.libro.fm"
      ],
      "manifestTcp": [],
      "networkMode": "outbound",
      "operatorAddedCidrs": [],
      "operatorAddedDomains": [],
      "operatorAddedTcp": [],
      "operatorDeniedCidrs": [],
      "operatorDeniedDomains": [],
      "operatorDeniedTcp": [],
      "pluginId": "libro",
      "pluginKey": "bookclerk:source-libro",
      "producers": [],
      "schemaVersion": 2,
      "tcp": []
    },
    "requested_artifact": {
      "url": "https://books.example.com/api/plugins/artifacts/9c1e4f7a",
      "archive_sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
      "payload_root_sha256": "a7ffc6f8bf1ed76651c14756a061d662f580ff4de43b49fa82d80a4b80f8434a"
    },
    "request_digest": "9c1e4f7a2b3d5e6f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f",
    "approved_version": "1.4.2",
    "grant": {
      "addressCidrs": [],
      "allowUndeclaredPublicRedirects": false,
      "approvedAt": "2026-10-29T10:12:00Z",
      "bindings": [
        "config",
        "secrets"
      ],
      "compatibilityFlags": [],
      "consumers": [],
      "domains": [
        "api.libro.fm",
        "cdn.libro.fm"
      ],
      "entrypoints": [
        "storefront"
      ],
      "jobs": [],
      "manifestCidrs": [],
      "manifestDomains": [
        "api.libro.fm",
        "cdn.libro.fm"
      ],
      "manifestTcp": [],
      "networkMode": "outbound",
      "operatorAddedCidrs": [],
      "operatorAddedDomains": [],
      "operatorAddedTcp": [],
      "operatorDeniedCidrs": [],
      "operatorDeniedDomains": [],
      "operatorDeniedTcp": [],
      "pluginId": "libro",
      "pluginKey": "bookclerk:source-libro",
      "producers": [],
      "schemaVersion": 2,
      "tcp": []
    },
    "approved_at": "2026-10-29T10:12:00Z",
    "approved_by": "operator",
    "revoked_at": "2026-11-01T08:00:00Z",
    "updated_at": "2026-10-29T10:12:00Z",
    "state": "consented",
    "pending": [
      "Awaiting consent (not automatically approved):",
      "  fetch hosts: any public website (*)"
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after a consent was approved or revoked.",
  "required": [
    "consent"
  ],
  "properties": {
    "consent": {
      "type": "object",
      "description": "The consent as stored now.",
      "required": [
        "library_id",
        "plugin_key",
        "requested_version",
        "requested_grant",
        "requested_artifact",
        "request_digest",
        "grant",
        "updated_at",
        "state",
        "pending"
      ],
      "properties": {
        "library_id": {
          "type": "string",
          "description": "The parent library's id.",
          "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
        },
        "plugin_key": {
          "type": "string",
          "description": "Canonical plugin key.",
          "example": "bookclerk:source-libro"
        },
        "requested_version": {
          "type": "string",
          "description": "Version the owner deploys here now.",
          "example": "1.4.2"
        },
        "requested_grant": {
          "description": "A plugin grant document (the plugin host's `PluginGrant`): the plugin key, the entrypoints, network mode and domains, and the bindings it may use; `null` when nothing was approved.",
          "examples": [
            {
              "addressCidrs": [],
              "allowUndeclaredPublicRedirects": false,
              "approvedAt": "2026-10-29T10:12:00Z",
              "bindings": [
                "config",
                "secrets"
              ],
              "compatibilityFlags": [],
              "consumers": [],
              "domains": [
                "api.libro.fm",
                "cdn.libro.fm"
              ],
              "entrypoints": [
                "storefront"
              ],
              "jobs": [],
              "manifestCidrs": [],
              "manifestDomains": [
                "api.libro.fm",
                "cdn.libro.fm"
              ],
              "manifestTcp": [],
              "networkMode": "outbound",
              "operatorAddedCidrs": [],
              "operatorAddedDomains": [],
              "operatorAddedTcp": [],
              "operatorDeniedCidrs": [],
              "operatorDeniedDomains": [],
              "operatorDeniedTcp": [],
              "pluginId": "libro",
              "pluginKey": "bookclerk:source-libro",
              "producers": [],
              "schemaVersion": 2,
              "tcp": []
            }
          ]
        },
        "requested_artifact": {
          "type": "object",
          "description": "The archive the owner deploys to this install.",
          "required": [
            "url",
            "archive_sha256",
            "payload_root_sha256"
          ],
          "properties": {
            "url": {
              "type": "string",
              "description": "Where the owner fetches it from.",
              "example": "https://books.example.com/api/plugins/artifacts/9c1e4f7a"
            },
            "archive_sha256": {
              "type": "string",
              "description": "SHA-256 of the archive.",
              "example": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
            },
            "payload_root_sha256": {
              "type": "string",
              "description": "Payload-root digest of the tree it unpacks to.",
              "example": "a7ffc6f8bf1ed76651c14756a061d662f580ff4de43b49fa82d80a4b80f8434a"
            }
          }
        },
        "request_digest": {
          "type": "string",
          "description": "Digest of the request as a whole; an approval names it so a changed request is not approved by mistake.",
          "example": "9c1e4f7a2b3d5e6f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f"
        },
        "approved_version": {
          "type": [
            "string",
            "null"
          ],
          "description": "Version this install's operator approved, when any.",
          "example": "1.4.2"
        },
        "grant": {
          "description": "A plugin grant document (the plugin host's `PluginGrant`): the plugin key, the entrypoints, network mode and domains, and the bindings it may use; `null` when nothing was approved.",
          "examples": [
            {
              "addressCidrs": [],
              "allowUndeclaredPublicRedirects": false,
              "approvedAt": "2026-10-29T10:12:00Z",
              "bindings": [
                "config",
                "secrets"
              ],
              "compatibilityFlags": [],
              "consumers": [],
              "domains": [
                "api.libro.fm",
                "cdn.libro.fm"
              ],
              "entrypoints": [
                "storefront"
              ],
              "jobs": [],
              "manifestCidrs": [],
              "manifestDomains": [
                "api.libro.fm",
                "cdn.libro.fm"
              ],
              "manifestTcp": [],
              "networkMode": "outbound",
              "operatorAddedCidrs": [],
              "operatorAddedDomains": [],
              "operatorAddedTcp": [],
              "operatorDeniedCidrs": [],
              "operatorDeniedDomains": [],
              "operatorDeniedTcp": [],
              "pluginId": "libro",
              "pluginKey": "bookclerk:source-libro",
              "producers": [],
              "schemaVersion": 2,
              "tcp": []
            }
          ]
        },
        "approved_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "RFC 3339 approval time.",
          "example": "2026-10-29T10:12:00Z"
        },
        "approved_by": {
          "type": [
            "string",
            "null"
          ],
          "description": "Who approved (`operator`, or `platform` for an automatic approval).",
          "example": "operator"
        },
        "revoked_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "RFC 3339 time the operator revoked the approval.",
          "example": "2026-11-01T08:00:00Z"
        },
        "updated_at": {
          "type": "string",
          "description": "RFC 3339 time of the last write.",
          "example": "2026-10-29T10:12:00Z"
        },
        "state": {
          "type": "string",
          "description": "How this host stands on the request: `consented`, `awaiting_host_consent` (no approval, or a revoked one), `awaiting_host_reapproval` (the owner changed the request or the archive since the approval), or `invalid_grant` (a grant that does not parse).",
          "example": "consented"
        },
        "pending": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "What the owner's current request asks for beyond the request the operator approved, as the operator-facing lines the CLI prints; empty when nothing was approved yet or the request stays within the approval.",
          "example": [
            "Awaiting consent (not automatically approved):",
            "  fetch hosts: any public website (*)"
          ]
        }
      }
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

Status `404`: No such consent (`not_found`).

```json
{
  "error": "not_found"
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
Failed to deserialize the JSON body into the target type: missing field `library_id` at line 1 column 2
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/join`

`GET /api/config/join`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — Whether this install has joined a library, and its state. Body: [`JoinStatus`](schemas.md#joinstatus) (`application/json`)
  - `fresh` *boolean*, **required** — Whether this install has no users, no libraries and no library cards yet.
  - `has_owner` *boolean*, **required** — Whether an Owner (or legacy administrator) exists here.
  - `shared_branch_eligible` *boolean*, **required** — Whether this install could still join a library as a shared branch.
  - `shared_branch_problem` *string or null* — Why it cannot join as a shared branch, when it cannot.
  - `default_branch_name` *string*, **required** — The name a join is offered under by default (this machine's name).
  - `links` *array of [`LibraryLinkView`](schemas.md#librarylinkview)*, **required** — Every library this install is a branch of, with whether its client runs.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — Whether this install may join as a shared branch could not be checked: its own contents could not be read (`join_failed`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/join",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/join")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Whether this install has joined a library, and its state.

```json
{
  "fresh": false,
  "has_owner": true,
  "shared_branch_eligible": false,
  "shared_branch_problem": "this install already holds library data of its own",
  "default_branch_name": "living-room",
  "links": [
    {
      "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
      "owner_origin": "https://books.example.com",
      "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
      "pinned": false,
      "state": "active",
      "enrolled_at": "2026-10-29T09:02:41Z",
      "share_books": true,
      "include_branches": true,
      "work_for_members": false,
      "keep_member_copies": false,
      "max_jobs": 2,
      "job_share": 2,
      "list_members_books": true,
      "keep_copies": true,
      "running": true
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/config/join`.",
  "required": [
    "fresh",
    "has_owner",
    "shared_branch_eligible",
    "default_branch_name",
    "links"
  ],
  "properties": {
    "fresh": {
      "type": "boolean",
      "description": "Whether this install has no users, no libraries and no library cards\nyet.",
      "example": false
    },
    "has_owner": {
      "type": "boolean",
      "description": "Whether an Owner (or legacy administrator) exists here.",
      "example": true
    },
    "shared_branch_eligible": {
      "type": "boolean",
      "description": "Whether this install could still join a library as a shared branch.",
      "example": false
    },
    "shared_branch_problem": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why it cannot join as a shared branch, when it cannot.",
      "example": "this install already holds library data of its own"
    },
    "default_branch_name": {
      "type": "string",
      "description": "The name a join is offered under by default (this machine's name).",
      "example": "living-room"
    },
    "links": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A library this install is a branch of: ids, origin and state, never key material.",
        "required": [
          "library_id",
          "owner_origin",
          "node_id",
          "pinned",
          "state",
          "enrolled_at",
          "share_books",
          "include_branches",
          "work_for_members",
          "keep_member_copies",
          "list_members_books",
          "keep_copies"
        ],
        "properties": {
          "library_id": {
            "type": "string",
            "description": "The library id.",
            "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
          },
          "owner_origin": {
            "type": "string",
            "description": "The library's canonical origin.",
            "example": "https://books.example.com"
          },
          "node_id": {
            "type": "string",
            "description": "The node id the library minted for this install.",
            "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
          },
          "pinned": {
            "type": "boolean",
            "description": "Whether the link pins the library's self-signed certificate.",
            "example": false
          },
          "state": {
            "type": "string",
            "description": "`active`, `revoked`, or `left`.",
            "example": "active"
          },
          "enrolled_at": {
            "type": "string",
            "description": "RFC 3339 enrollment time.",
            "example": "2026-10-29T09:02:41Z"
          },
          "share_books": {
            "type": "boolean",
            "description": "\"Share this branch's books with the library\".",
            "example": true
          },
          "include_branches": {
            "type": "boolean",
            "description": "\"Include books from my own branches\".",
            "example": true
          },
          "work_for_members": {
            "type": "boolean",
            "description": "\"Download for other members of the library\": this install takes the library's work for its other members too, not only its owner's (off by default).",
            "example": false
          },
          "keep_member_copies": {
            "type": "boolean",
            "description": "\"Keep copies of other members' books\": a book acquired here for another member is also written to this install's own destinations (off by default).",
            "example": false
          },
          "max_jobs": {
            "type": [
              "integer",
              "null"
            ],
            "description": "The most jobs this install runs for the library at once, as its operator set it; `null` gives the library an equal share of the machine.",
            "example": 2,
            "maximum": 16,
            "minimum": 0
          },
          "job_share": {
            "type": [
              "integer",
              "null"
            ],
            "description": "How many jobs the library may have running here now: its share of the machine after this install's own work, within `max_jobs`. Only the listing and sharing routes report it, and only while the link's client runs.",
            "example": 2,
            "minimum": 0
          },
          "list_members_books": {
            "type": "boolean",
            "description": "\"List my members' books to the library\": the library's catalog\nlookups are answered with members' books (except those who opted\nout) as well as the owner's.",
            "example": true
          },
          "keep_copies": {
            "type": "boolean",
            "description": "\"Keep copies on this computer\": the branch takes the library's\ncopies of its person's books into its own storage. The branch takes\nthe library's work and shows its books whether or not this is on.",
            "example": true
          },
          "running": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "Whether this install's client for the link is running now; only the join routes report it.",
            "example": true
          }
        }
      },
      "description": "Every library this install is a branch of, with whether its client runs."
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

Status `403`: The session is impersonating a user.

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

Status `502`: Whether this install may join as a shared branch could not be checked: its own contents could not be read (`join_failed`).

```json
{
  "detail": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection",
  "error": "join_failed"
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

## `POST /api/config/join/inspect`

`POST /api/config/join/inspect`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`InspectBody`](schemas.md#inspectbody)

- `address` *string*, **required** — The library's address (`https://owner.example`, `owner.lan:8443`).

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
- `403` — The session is impersonating a user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — Joining would make a loop (`join_cycle`), or the library's address is not the one given (`origin_mismatch`, with the address to use in `origin`). How deep the library sits is checked once the join's kind is chosen, at `start` and when the code is redeemed. Body: [`JoinConflict`](schemas.md#joinconflict) (`application/json`)
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The library could not be reached (`unreachable`), or answered with an error (`library_error`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/inspect' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"address": "https://books.example.com"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/inspect' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"address": "https://books.example.com"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/inspect",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"address": "https://books.example.com"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/inspect", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"address": "https://books.example.com"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/inspect")
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

Status `409`: Joining would make a loop (`join_cycle`), or the library's address is not the one given (`origin_mismatch`, with the address to use in `origin`). How deep the library sits is checked once the join's kind is chosen, at `start` and when the code is redeemed.

```json
{
  "detail": "that library is this library or one of its own branches; joining it would make a loop",
  "error": "join_cycle"
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
  "detail": "the library answered HTTP 500 enrollment_unavailable",
  "error": "library_error"
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

## `POST /api/config/join/start`

`POST /api/config/join/start`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`StartBody`](schemas.md#startbody)

- `address` *string* — The library's address. Left out with `from_card`.
- `from_card` *string or null* — The library id of a library this install holds an active library card for: the branch replaces the card. The library's address and pinned certificate come from the card, so `address` and `fingerprint` are left out, and the branch name defaults to the card's name. Not with `shared_branch`.
- `fingerprint` *string or null* — The fingerprint the operator confirmed (self-signed libraries).
- `branch_name` *string or null* — The name to join as (default: this machine's name).
- `shared_branch` *boolean* — Offer to join as a shared branch (a shared node that shares the library's database): only an empty install may, and its data is replaced.

### Responses

- `200` — The library's sign-in URL to open. Body: [`JoinStarted`](schemas.md#joinstarted) (`application/json`)
  - `authorize_url` *string*, **required** — The library's sign-in URL to open in the browser.
  - `library` *[`InspectedLibraryView`](schemas.md#inspectedlibraryview)*, **required** — What the library said about itself.
  - `branch_name` *string*, **required** — The name this install asked to join as.
  - `shared_branch_offered` *boolean*, **required** — Whether joining as a shared branch was offered.
  - `expires_in_secs` *integer (int64)*, **required** — Seconds the pending sign-in stays valid. At least 0.
- `400` — The address is not a library (`invalid_address`), is plain HTTP off this machine (`transport_insecure`), or a shared branch would replace a card (`invalid_request`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The library is not accepting new branches (`enrollment_disabled`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `404` — With `from_card`: this install holds no active library card for that library (`not_found`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `409` — The library's fingerprint is not confirmed, or changed since the card signed in (`fingerprint_unconfirmed`), this install holds data of its own for a shared branch join (`not_empty`), joining would make a loop (`join_cycle`) or, for a branch (not a shared branch, which adds no depth), make the tree too deep (`ancestry_too_deep`), or the library's address is not the one given, or no longer the card's library (`origin_mismatch`). Body: [`JoinConflict`](schemas.md#joinconflict) (`application/json`)
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The library could not be reached (`unreachable`), or answered with an error (`library_error`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/start' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/start' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/start",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/start", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/start")
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
  "authorize_url": "https://books.example.com/join/approve?client_id=bookclerk-branch&redirect_uri=…",
  "library": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "library_name": "books.example.com",
    "origin": "https://books.example.com",
    "transport": "self_signed",
    "enrollment": true,
    "fingerprint": "kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o",
    "needs_fingerprint": true
  },
  "branch_name": "living-room",
  "shared_branch_offered": false,
  "expires_in_secs": 600
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `POST /api/config/join/start`.",
  "required": [
    "authorize_url",
    "library",
    "branch_name",
    "shared_branch_offered",
    "expires_in_secs"
  ],
  "properties": {
    "authorize_url": {
      "type": "string",
      "description": "The library's sign-in URL to open in the browser.",
      "example": "https://books.example.com/join/approve?client_id=bookclerk-branch&redirect_uri=…"
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
    "branch_name": {
      "type": "string",
      "description": "The name this install asked to join as.",
      "example": "living-room"
    },
    "shared_branch_offered": {
      "type": "boolean",
      "description": "Whether joining as a shared branch was offered.",
      "example": false
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

Status `400`: The address is not a library (`invalid_address`), is plain HTTP off this machine (`transport_insecure`), or a shared branch would replace a card (`invalid_request`). As `text/plain`, when the body is not syntactically valid JSON.

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

Status `403`: The library is not accepting new branches (`enrollment_disabled`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "detail": "This library isn't accepting new branches",
  "error": "enrollment_disabled"
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

Status `404`: With `from_card`: this install holds no active library card for that library (`not_found`).

```json
{
  "detail": "this install holds no library card for that library",
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

Status `409`: The library's fingerprint is not confirmed, or changed since the card signed in (`fingerprint_unconfirmed`), this install holds data of its own for a shared branch join (`not_empty`), joining would make a loop (`join_cycle`) or, for a branch (not a shared branch, which adds no depth), make the tree too deep (`ancestry_too_deep`), or the library's address is not the one given, or no longer the card's library (`origin_mismatch`).

```json
{
  "detail": "confirm the library's fingerprint first; it must match the one on its Settings → Branches page",
  "error": "fingerprint_unconfirmed"
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

Status `502`: The library could not be reached (`unreachable`), or answered with an error (`library_error`).

```json
{
  "detail": "the library answered HTTP 500 enrollment_unavailable",
  "error": "library_error"
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

## `POST /api/config/join/callback`

`POST /api/config/join/callback`: redeems the library's code with the
sign-in's sealed verifier and keys. The pending sign-in is kept when the
redeem fails for a reason a retry can fix (answered as 502
`join_retry`; the page offers Try again with the same code), and ended
on success, refusal, or anything else.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`CallbackBody`](schemas.md#callbackbody)

- `code` *string* — The code from the library.
- `state` *string* — The `state` it came back with.
- `error` *string or null* — The library's `error` (`access_denied`), when it sent one.
- `connection` *string or null* — The connection type the library's user chose (`shared_branch`; a branch when left out). The code is bound to it, so a changed value only fails the redeem.

### Responses

- `200` — The join completed. Body: [`JoinCompleted`](schemas.md#joincompleted) (`application/json`)
- `400` — The callback carries no code or an unknown connection type (`invalid_request`), or the sign-in did not offer a shared branch join (`shared_branch_not_offered`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The library's user did not approve (`denied`), the library refused the sign-in (`refused`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `409` — This install holds data of its own for a shared branch join (`not_empty`), the library's database plugin is not installed here (`database_guest_missing`), the library sent no D1 token and none is set here (`d1_token_missing`), or joining would make a loop (`join_cycle`, `ancestry_too_deep`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `410` — The sign-in is unknown, already used, or older than ten minutes; start again (`join_expired`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The redeem can be retried with the sign-in still pending (`join_retry`), the library could not be reached (`unreachable`), it answered with an error (`library_error`), or it enrolled this host but the join could not be saved (`join_unsaved`, `join_failed`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/callback' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/callback' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/callback",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/callback", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/callback")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The join completed.

```json
{
  "library": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "owner_origin": "https://books.example.com",
    "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "pinned": false,
    "state": "active",
    "enrolled_at": "2026-10-29T09:02:41Z",
    "share_books": true,
    "include_branches": true,
    "work_for_members": false,
    "keep_member_copies": false,
    "max_jobs": 2,
    "job_share": 2,
    "list_members_books": true,
    "keep_copies": true,
    "running": true
  },
  "library_name": "books.example.com",
  "running": true,
  "connections": {
    "offered": 2,
    "added": 1,
    "already_here": 1,
    "taken": 0,
    "failed": 0,
    "not_kept": null
  },
  "replaced_card": {
    "card_removed": true,
    "library_signed_out": true
  }
}
```

Schema:

```json
{
  "oneOf": [
    {
      "type": "object",
      "description": "Joined as a branch; the link's client starts at once.",
      "required": [
        "library",
        "running"
      ],
      "properties": {
        "library": {
          "type": "object",
          "description": "The new link.",
          "required": [
            "library_id",
            "owner_origin",
            "node_id",
            "pinned",
            "state",
            "enrolled_at",
            "share_books",
            "include_branches",
            "work_for_members",
            "keep_member_copies",
            "list_members_books",
            "keep_copies"
          ],
          "properties": {
            "library_id": {
              "type": "string",
              "description": "The library id.",
              "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
            },
            "owner_origin": {
              "type": "string",
              "description": "The library's canonical origin.",
              "example": "https://books.example.com"
            },
            "node_id": {
              "type": "string",
              "description": "The node id the library minted for this install.",
              "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
            },
            "pinned": {
              "type": "boolean",
              "description": "Whether the link pins the library's self-signed certificate.",
              "example": false
            },
            "state": {
              "type": "string",
              "description": "`active`, `revoked`, or `left`.",
              "example": "active"
            },
            "enrolled_at": {
              "type": "string",
              "description": "RFC 3339 enrollment time.",
              "example": "2026-10-29T09:02:41Z"
            },
            "share_books": {
              "type": "boolean",
              "description": "\"Share this branch's books with the library\".",
              "example": true
            },
            "include_branches": {
              "type": "boolean",
              "description": "\"Include books from my own branches\".",
              "example": true
            },
            "work_for_members": {
              "type": "boolean",
              "description": "\"Download for other members of the library\": this install takes the library's work for its other members too, not only its owner's (off by default).",
              "example": false
            },
            "keep_member_copies": {
              "type": "boolean",
              "description": "\"Keep copies of other members' books\": a book acquired here for another member is also written to this install's own destinations (off by default).",
              "example": false
            },
            "max_jobs": {
              "type": [
                "integer",
                "null"
              ],
              "description": "The most jobs this install runs for the library at once, as its operator set it; `null` gives the library an equal share of the machine.",
              "example": 2,
              "maximum": 16,
              "minimum": 0
            },
            "job_share": {
              "type": [
                "integer",
                "null"
              ],
              "description": "How many jobs the library may have running here now: its share of the machine after this install's own work, within `max_jobs`. Only the listing and sharing routes report it, and only while the link's client runs.",
              "example": 2,
              "minimum": 0
            },
            "list_members_books": {
              "type": "boolean",
              "description": "\"List my members' books to the library\": the library's catalog\nlookups are answered with members' books (except those who opted\nout) as well as the owner's.",
              "example": true
            },
            "keep_copies": {
              "type": "boolean",
              "description": "\"Keep copies on this computer\": the branch takes the library's\ncopies of its person's books into its own storage. The branch takes\nthe library's work and shows its books whether or not this is on.",
              "example": true
            },
            "running": {
              "type": [
                "boolean",
                "null"
              ],
              "description": "Whether this install's client for the link is running now; only the join routes report it.",
              "example": true
            }
          }
        },
        "library_name": {
          "type": [
            "string",
            "null"
          ],
          "description": "The library's name, when the join came from a sign-in (not an invite link).",
          "example": "books.example.com"
        },
        "running": {
          "type": "boolean",
          "description": "Whether the link's client is running now.",
          "example": true
        },
        "connections": {
          "oneOf": [
            {
              "type": "object",
              "description": "What became of the store connections the library's user chose to\nbring to this branch; left out when none came.",
              "required": [
                "offered",
                "added",
                "already_here",
                "taken",
                "failed"
              ],
              "properties": {
                "offered": {
                  "type": "integer",
                  "description": "How many came with the join.",
                  "example": 2,
                  "minimum": 0
                },
                "added": {
                  "type": "integer",
                  "description": "How many are now connected here for the person.",
                  "example": 1,
                  "minimum": 0
                },
                "already_here": {
                  "type": "integer",
                  "description": "How many the person had already connected here (left as they are).",
                  "example": 1,
                  "minimum": 0
                },
                "taken": {
                  "type": "integer",
                  "description": "How many are connected to someone else here (left as they are).",
                  "example": 0,
                  "minimum": 0
                },
                "failed": {
                  "type": "integer",
                  "description": "How many could not be kept: no secret came with them, or their\nwrite failed.",
                  "example": 0,
                  "minimum": 0
                },
                "not_kept": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Why none were kept, when none were: `not_allowed` (this install's\noperator does not let the person connect store accounts),\n`no_owner` (no single Owner here to keep them for) or `failed`.",
                  "example": null
                }
              }
            },
            {
              "type": "null"
            }
          ]
        },
        "replaced_card": {
          "oneOf": [
            {
              "type": "object",
              "description": "Set when this install held a library card for the library: the\nbranch replaced it. Whether the card is gone here and whether the\nlibrary ended its session are in [`ReplacedCard`]. Left out when\nthere was none.",
              "required": [
                "card_removed",
                "library_signed_out"
              ],
              "properties": {
                "card_removed": {
                  "type": "boolean",
                  "description": "Whether the card was dropped here. False when it could not be read\nor dropped: it is then still listed under Library cards, and the\nperson signs it out there. The branch join stands either way.",
                  "example": true
                },
                "library_signed_out": {
                  "type": "boolean",
                  "description": "Whether the library ended the card's session too. False when it\ncould not be reached, or the card was not dropped here.",
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
    },
    {
      "type": "object",
      "description": "Joined as a shared branch: this install restarts as a shared node of the library.",
      "required": [
        "connection",
        "library_id",
        "node_id",
        "library_name",
        "database",
        "master_key_wrapped",
        "restarting"
      ],
      "properties": {
        "connection": {
          "type": "string",
          "description": "Always `shared_branch`.",
          "example": "shared_branch"
        },
        "library_id": {
          "type": "string",
          "description": "The library id.",
          "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
        },
        "node_id": {
          "type": "string",
          "description": "The node id the library minted for this install.",
          "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
        },
        "library_name": {
          "type": "string",
          "description": "The library's name.",
          "example": "books.example.com"
        },
        "database": {
          "type": "string",
          "description": "The database plugin this install now runs (`postgres` or `d1`).",
          "example": "postgres"
        },
        "master_key_wrapped": {
          "type": "boolean",
          "description": "Whether the shared master key was wrapped with this install's password.",
          "example": true
        },
        "restarting": {
          "type": "boolean",
          "description": "Always `true`: the daemon restarts as a shared node right after answering.",
          "example": true
        }
      }
    }
  ],
  "description": "JSON body once a join completed: as a branch, or as a shared branch."
}
```

Status `400`: The callback carries no code or an unknown connection type (`invalid_request`), or the sign-in did not offer a shared branch join (`shared_branch_not_offered`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "connection must be branch or shared_branch",
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

Status `403`: The library's user did not approve (`denied`), the library refused the sign-in (`refused`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "detail": "the library's user did not let this branch join",
  "error": "denied"
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

Status `409`: This install holds data of its own for a shared branch join (`not_empty`), the library's database plugin is not installed here (`database_guest_missing`), the library sent no D1 token and none is set here (`d1_token_missing`), or joining would make a loop (`join_cycle`, `ancestry_too_deep`).

```json
{
  "detail": "the library's database is Cloudflare D1 and it sent no API token: set BOOKCLERK_D1_API_TOKEN for this server and restart it, then try again within 10 minutes (the library holds this server as node 7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c)",
  "error": "d1_token_missing"
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

Status `502`: The redeem can be retried with the sign-in still pending (`join_retry`), the library could not be reached (`unreachable`), it answered with an error (`library_error`), or it enrolled this host but the join could not be saved (`join_unsaved`, `join_failed`).

```json
{
  "detail": "the library did not answer in time; try again",
  "error": "join_retry"
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

## `POST /api/config/join/invite`

`POST /api/config/join/invite`: join with a pasted branch invite. The
link is redeemed once and dropped; it is never logged or echoed. A
library slower than [`REDEEM_TIMEOUT`] is answered 504 `join_pending`
while the join carries on and starts its client when it lands.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`InviteBody`](schemas.md#invitebody)

- `link` *string*, **required** — The whole branch invite (`https://<library>/enroll#bcn1...`).

### Responses

- `200` — The join completed. Body: [`BranchJoined`](schemas.md#branchjoined) (`application/json`)
  - `library` *[`LibraryLinkView`](schemas.md#librarylinkview)*, **required** — The new link.
  - `library_name` *string or null* — The library's name, when the join came from a sign-in (not an invite link).
  - `running` *boolean*, **required** — Whether the link's client is running now.
  - `connections` *[`BroughtConnectionsView`](schemas.md#broughtconnectionsview) or null* — What became of the store connections the library's user chose to bring to this branch; left out when none came.
  - `replaced_card` *[`ReplacedCard`](schemas.md#replacedcard) or null* — Set when this install held a library card for the library: the branch replaced it. Whether the card is gone here and whether the library ended its session are in [`ReplacedCard`]. Left out when there was none.
- `400` — The link is not a branch invite (`invalid_invite`), or is plain HTTP off this machine (`transport_insecure`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The library refused the invite (`refused`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `409` — Joining would make a loop (`join_cycle`, `ancestry_too_deep`), or the library's address is not the one in the invite (`origin_mismatch`, with the address to use in `origin`). Body: [`JoinConflict`](schemas.md#joinconflict) (`application/json`)
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The join stopped unexpectedly (`join_failed`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `502` — The library could not be reached (`unreachable`), answered with an error (`library_error`), or enrolled this host without the join being saved here (`join_unsaved`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `504` — The library is slow to answer; the join goes on in the background (`join_pending`). Poll the join state instead of retrying. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/invite' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"link": "https://books.example.com/enroll#bcn1.9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b.kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/invite' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"link": "https://books.example.com/enroll#bcn1.9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b.kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/invite",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"link": "https://books.example.com/enroll#bcn1.9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b.kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/invite", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"link": "https://books.example.com/enroll#bcn1.9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b.kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/invite")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"link": "https://books.example.com/enroll#bcn1.9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b.kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The join completed.

```json
{
  "library": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "owner_origin": "https://books.example.com",
    "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "pinned": false,
    "state": "active",
    "enrolled_at": "2026-10-29T09:02:41Z",
    "share_books": true,
    "include_branches": true,
    "work_for_members": false,
    "keep_member_copies": false,
    "max_jobs": 2,
    "job_share": 2,
    "list_members_books": true,
    "keep_copies": true,
    "running": true
  },
  "library_name": "books.example.com",
  "running": true,
  "connections": {
    "offered": 2,
    "added": 1,
    "already_here": 1,
    "taken": 0,
    "failed": 0,
    "not_kept": null
  },
  "replaced_card": {
    "card_removed": true,
    "library_signed_out": true
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body once this install joined a library as a branch.",
  "required": [
    "library",
    "running"
  ],
  "properties": {
    "library": {
      "type": "object",
      "description": "The new link.",
      "required": [
        "library_id",
        "owner_origin",
        "node_id",
        "pinned",
        "state",
        "enrolled_at",
        "share_books",
        "include_branches",
        "work_for_members",
        "keep_member_copies",
        "list_members_books",
        "keep_copies"
      ],
      "properties": {
        "library_id": {
          "type": "string",
          "description": "The library id.",
          "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
        },
        "owner_origin": {
          "type": "string",
          "description": "The library's canonical origin.",
          "example": "https://books.example.com"
        },
        "node_id": {
          "type": "string",
          "description": "The node id the library minted for this install.",
          "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
        },
        "pinned": {
          "type": "boolean",
          "description": "Whether the link pins the library's self-signed certificate.",
          "example": false
        },
        "state": {
          "type": "string",
          "description": "`active`, `revoked`, or `left`.",
          "example": "active"
        },
        "enrolled_at": {
          "type": "string",
          "description": "RFC 3339 enrollment time.",
          "example": "2026-10-29T09:02:41Z"
        },
        "share_books": {
          "type": "boolean",
          "description": "\"Share this branch's books with the library\".",
          "example": true
        },
        "include_branches": {
          "type": "boolean",
          "description": "\"Include books from my own branches\".",
          "example": true
        },
        "work_for_members": {
          "type": "boolean",
          "description": "\"Download for other members of the library\": this install takes the library's work for its other members too, not only its owner's (off by default).",
          "example": false
        },
        "keep_member_copies": {
          "type": "boolean",
          "description": "\"Keep copies of other members' books\": a book acquired here for another member is also written to this install's own destinations (off by default).",
          "example": false
        },
        "max_jobs": {
          "type": [
            "integer",
            "null"
          ],
          "description": "The most jobs this install runs for the library at once, as its operator set it; `null` gives the library an equal share of the machine.",
          "example": 2,
          "maximum": 16,
          "minimum": 0
        },
        "job_share": {
          "type": [
            "integer",
            "null"
          ],
          "description": "How many jobs the library may have running here now: its share of the machine after this install's own work, within `max_jobs`. Only the listing and sharing routes report it, and only while the link's client runs.",
          "example": 2,
          "minimum": 0
        },
        "list_members_books": {
          "type": "boolean",
          "description": "\"List my members' books to the library\": the library's catalog\nlookups are answered with members' books (except those who opted\nout) as well as the owner's.",
          "example": true
        },
        "keep_copies": {
          "type": "boolean",
          "description": "\"Keep copies on this computer\": the branch takes the library's\ncopies of its person's books into its own storage. The branch takes\nthe library's work and shows its books whether or not this is on.",
          "example": true
        },
        "running": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Whether this install's client for the link is running now; only the join routes report it.",
          "example": true
        }
      }
    },
    "library_name": {
      "type": [
        "string",
        "null"
      ],
      "description": "The library's name, when the join came from a sign-in (not an invite link).",
      "example": "books.example.com"
    },
    "running": {
      "type": "boolean",
      "description": "Whether the link's client is running now.",
      "example": true
    },
    "connections": {
      "oneOf": [
        {
          "type": "object",
          "description": "What became of the store connections the library's user chose to\nbring to this branch; left out when none came.",
          "required": [
            "offered",
            "added",
            "already_here",
            "taken",
            "failed"
          ],
          "properties": {
            "offered": {
              "type": "integer",
              "description": "How many came with the join.",
              "example": 2,
              "minimum": 0
            },
            "added": {
              "type": "integer",
              "description": "How many are now connected here for the person.",
              "example": 1,
              "minimum": 0
            },
            "already_here": {
              "type": "integer",
              "description": "How many the person had already connected here (left as they are).",
              "example": 1,
              "minimum": 0
            },
            "taken": {
              "type": "integer",
              "description": "How many are connected to someone else here (left as they are).",
              "example": 0,
              "minimum": 0
            },
            "failed": {
              "type": "integer",
              "description": "How many could not be kept: no secret came with them, or their\nwrite failed.",
              "example": 0,
              "minimum": 0
            },
            "not_kept": {
              "type": [
                "string",
                "null"
              ],
              "description": "Why none were kept, when none were: `not_allowed` (this install's\noperator does not let the person connect store accounts),\n`no_owner` (no single Owner here to keep them for) or `failed`.",
              "example": null
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "replaced_card": {
      "oneOf": [
        {
          "type": "object",
          "description": "Set when this install held a library card for the library: the\nbranch replaced it. Whether the card is gone here and whether the\nlibrary ended its session are in [`ReplacedCard`]. Left out when\nthere was none.",
          "required": [
            "card_removed",
            "library_signed_out"
          ],
          "properties": {
            "card_removed": {
              "type": "boolean",
              "description": "Whether the card was dropped here. False when it could not be read\nor dropped: it is then still listed under Library cards, and the\nperson signs it out there. The branch join stands either way.",
              "example": true
            },
            "library_signed_out": {
              "type": "boolean",
              "description": "Whether the library ended the card's session too. False when it\ncould not be reached, or the card was not dropped here.",
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

Status `400`: The link is not a branch invite (`invalid_invite`), or is plain HTTP off this machine (`transport_insecure`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "that is not a branch invite (https://<library>/enroll#bcn1...)",
  "error": "invalid_invite"
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

Status `403`: The library refused the invite (`refused`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `409`: Joining would make a loop (`join_cycle`, `ancestry_too_deep`), or the library's address is not the one in the invite (`origin_mismatch`, with the address to use in `origin`).

```json
{
  "detail": "that library is this library or one of its own branches; joining it would make a loop",
  "error": "join_cycle"
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
Failed to deserialize the JSON body into the target type: missing field `link` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The join stopped unexpectedly (`join_failed`).

```json
{
  "detail": "the join stopped unexpectedly",
  "error": "join_failed"
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

Status `502`: The library could not be reached (`unreachable`), answered with an error (`library_error`), or enrolled this host without the join being saved here (`join_unsaved`).

```json
{
  "detail": "the library enrolled this host as node 7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c but it could not be saved here (database is locked); ask the library's owner to remove that node, then join again",
  "error": "join_unsaved"
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

Status `504`: The library is slow to answer; the join goes on in the background (`join_pending`). Poll the join state instead of retrying.

```json
{
  "detail": "the library is slow to answer; this branch keeps joining and appears under Libraries once it has (if it never does, this install's log says why)",
  "error": "join_pending"
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

## `POST /api/config/join/leave`

`POST /api/config/join/leave`: Leave library. The leave runs on its own
task, so a request cut short (the client gone, or the API deadline)
still records it once the library has been told.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`LeaveBody`](schemas.md#leavebody)

- `library_id` *string*, **required** — The library (parent library id) to leave.

### Responses

- `200` — Leaving started. Body: [`LeaveStarted`](schemas.md#leavestarted) (`application/json`)
  - `library` *[`LibraryLinkView`](schemas.md#librarylinkview)*, **required** — The link, now `left`.
  - `owner_notified` *boolean*, **required** — Whether the library was told (otherwise it notices the missed heartbeats).
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session is impersonating a user. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — This install is not a branch of that library (`not_found`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The leave stopped before it finished (`leave_failed`), or the library database could not be read or written. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/leave' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/leave' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/leave",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/leave", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/leave")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Leaving started.

```json
{
  "library": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "owner_origin": "https://books.example.com",
    "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "pinned": false,
    "state": "active",
    "enrolled_at": "2026-10-29T09:02:41Z",
    "share_books": true,
    "include_branches": true,
    "work_for_members": false,
    "keep_member_copies": false,
    "max_jobs": 2,
    "job_share": 2,
    "list_members_books": true,
    "keep_copies": true,
    "running": true
  },
  "owner_notified": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body once leaving a library started.",
  "required": [
    "library",
    "owner_notified"
  ],
  "properties": {
    "library": {
      "type": "object",
      "description": "The link, now `left`.",
      "required": [
        "library_id",
        "owner_origin",
        "node_id",
        "pinned",
        "state",
        "enrolled_at",
        "share_books",
        "include_branches",
        "work_for_members",
        "keep_member_copies",
        "list_members_books",
        "keep_copies"
      ],
      "properties": {
        "library_id": {
          "type": "string",
          "description": "The library id.",
          "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
        },
        "owner_origin": {
          "type": "string",
          "description": "The library's canonical origin.",
          "example": "https://books.example.com"
        },
        "node_id": {
          "type": "string",
          "description": "The node id the library minted for this install.",
          "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
        },
        "pinned": {
          "type": "boolean",
          "description": "Whether the link pins the library's self-signed certificate.",
          "example": false
        },
        "state": {
          "type": "string",
          "description": "`active`, `revoked`, or `left`.",
          "example": "active"
        },
        "enrolled_at": {
          "type": "string",
          "description": "RFC 3339 enrollment time.",
          "example": "2026-10-29T09:02:41Z"
        },
        "share_books": {
          "type": "boolean",
          "description": "\"Share this branch's books with the library\".",
          "example": true
        },
        "include_branches": {
          "type": "boolean",
          "description": "\"Include books from my own branches\".",
          "example": true
        },
        "work_for_members": {
          "type": "boolean",
          "description": "\"Download for other members of the library\": this install takes the library's work for its other members too, not only its owner's (off by default).",
          "example": false
        },
        "keep_member_copies": {
          "type": "boolean",
          "description": "\"Keep copies of other members' books\": a book acquired here for another member is also written to this install's own destinations (off by default).",
          "example": false
        },
        "max_jobs": {
          "type": [
            "integer",
            "null"
          ],
          "description": "The most jobs this install runs for the library at once, as its operator set it; `null` gives the library an equal share of the machine.",
          "example": 2,
          "maximum": 16,
          "minimum": 0
        },
        "job_share": {
          "type": [
            "integer",
            "null"
          ],
          "description": "How many jobs the library may have running here now: its share of the machine after this install's own work, within `max_jobs`. Only the listing and sharing routes report it, and only while the link's client runs.",
          "example": 2,
          "minimum": 0
        },
        "list_members_books": {
          "type": "boolean",
          "description": "\"List my members' books to the library\": the library's catalog\nlookups are answered with members' books (except those who opted\nout) as well as the owner's.",
          "example": true
        },
        "keep_copies": {
          "type": "boolean",
          "description": "\"Keep copies on this computer\": the branch takes the library's\ncopies of its person's books into its own storage. The branch takes\nthe library's work and shows its books whether or not this is on.",
          "example": true
        },
        "running": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Whether this install's client for the link is running now; only the join routes report it.",
          "example": true
        }
      }
    },
    "owner_notified": {
      "type": "boolean",
      "description": "Whether the library was told (otherwise it notices the missed heartbeats).",
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

Status `404`: This install is not a branch of that library (`not_found`).

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

Status `500`: The leave stopped before it finished (`leave_failed`), or the library database could not be read or written.

```json
{
  "error": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/nodes`

`GET /api/config/nodes`: every node for an operator; the caller's own
nodes for a node owner.

**Access:** The operator, a signed-in Owner or Administrator, or a Member who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The nodes the caller administers. Body: [`NodesResponse`](schemas.md#nodesresponse) (`application/json`)
  - `node_id` *string*, **required** — Id of the node that answered.
  - `nodes` *array of [`NodeView`](schemas.md#nodeview)*, **required** — The nodes the caller administers.
  - `capabilities` *[`NodeCapabilities`](schemas.md#nodecapabilities)*, **required** — What the caller may do with them.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The configuration could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/nodes' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/nodes",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/nodes", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/nodes")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The nodes the caller administers.

```json
{
  "node_id": "node-7f3a9c",
  "nodes": [
    {
      "node_id": "node-7f3a9c",
      "library_id": "c0ffee12-3456-4789-8abc-def012345678",
      "created_at": "2026-10-01T08:00:00Z",
      "heartbeat_at": "2026-10-28T19:07:00Z",
      "software_version": "0.3.1",
      "schema_state": "frozen@12+a1b2c3d4",
      "compatible": true,
      "target": "linux-x64-gnu",
      "labels": [
        "rack=a"
      ],
      "implicit_labels": [
        "node=node-7f3a9c",
        "target=linux-x64-gnu"
      ],
      "live": true,
      "runtime": {
        "label": "Living-room server",
        "cpu_budget_percent": 80,
        "memory_budget_percent": 75,
        "schedulable": true,
        "cache_budget_bytes": 536870912
      },
      "status": {
        "node_id": "node-7f3a9c",
        "incarnation": "01J9Q0Z5C7M8N3P4R6S8T9V0W1",
        "started_at": "2026-10-28T06:00:00Z",
        "heartbeat_at": "2026-10-28T19:07:00Z",
        "heartbeat_ms": 1793214420000,
        "interval_ms": 15000,
        "cpu_millicores": 8000,
        "memory_bytes": 16777216000,
        "cpu_budget_millicores": 6400,
        "memory_budget_bytes": 12582912000,
        "schedulable": true,
        "cpu_used_millicores": 350,
        "memory_available_bytes": 9663676416,
        "files_dir_free_bytes": 214748364800,
        "reserved": {
          "memory_bytes": 1073741824,
          "cpu_millicores": 1600
        },
        "running_guests": 2,
        "availability_ppm": 998500,
        "online_since_ms": 1793167200000,
        "cache_bytes": 734003200,
        "cache_budget_bytes": 1073741824,
        "cache_swept_ms": 1793212800000,
        "public_origin": "https://branch.example"
      },
      "this_node": true,
      "kind": "shared",
      "relationship": "remote_branch",
      "owner_user_id": 7,
      "share_books": true,
      "include_branches": false,
      "owner": {
        "user_id": 7,
        "display_name": "Sam Reader",
        "role": "owner"
      }
    }
  ],
  "capabilities": {
    "scope": "all",
    "can_edit_runtime": true,
    "can_edit_labels": true,
    "can_assign_owner": true
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/config/nodes` body.",
  "required": [
    "node_id",
    "nodes",
    "capabilities"
  ],
  "properties": {
    "node_id": {
      "type": "string",
      "description": "Id of the node that answered.",
      "example": "node-7f3a9c"
    },
    "nodes": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One node with its latest heartbeat, budget and owner.",
        "required": [
          "node_id",
          "library_id",
          "created_at",
          "heartbeat_at",
          "software_version",
          "schema_state",
          "compatible",
          "target",
          "labels",
          "implicit_labels",
          "live",
          "runtime",
          "this_node",
          "kind",
          "relationship"
        ],
        "properties": {
          "node_id": {
            "type": "string",
            "description": "Node id.",
            "example": "node-7f3a9c"
          },
          "library_id": {
            "type": "string",
            "description": "Library the node belongs to.",
            "example": "c0ffee12-3456-4789-8abc-def012345678"
          },
          "created_at": {
            "type": "string",
            "description": "RFC 3339 time the node first enrolled.",
            "example": "2026-10-01T08:00:00Z"
          },
          "heartbeat_at": {
            "type": "string",
            "description": "RFC 3339 time of the last heartbeat the node row recorded.",
            "example": "2026-10-28T19:07:00Z"
          },
          "software_version": {
            "type": "string",
            "description": "Bookclerk version the node runs.",
            "example": "0.3.1"
          },
          "schema_state": {
            "type": "string",
            "description": "Database schema the node last applied.",
            "example": "frozen@12+a1b2c3d4"
          },
          "compatible": {
            "type": "boolean",
            "description": "True when the node's own binary accepted the schema it reported;\nnot a comparison with this daemon's schema.",
            "example": true
          },
          "target": {
            "type": "string",
            "description": "Bookclerk target the node runs on.",
            "example": "linux-x64-gnu"
          },
          "labels": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Operator-set placement labels.",
            "example": [
              "rack=a"
            ]
          },
          "implicit_labels": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Labels the scheduler adds for the node (its id and target).",
            "example": [
              "node=node-7f3a9c",
              "target=linux-x64-gnu"
            ]
          },
          "live": {
            "type": "boolean",
            "description": "True while the last heartbeat is recent enough.",
            "example": true
          },
          "runtime": {
            "type": "object",
            "description": "The node's runtime document, or the defaults when it never wrote\none.",
            "required": [
              "label"
            ],
            "properties": {
              "label": {
                "type": "string",
                "description": "Operator label shown for this node. Empty is allowed.",
                "example": "Living-room server"
              },
              "cpu_budget_percent": {
                "type": "integer",
                "format": "int32",
                "description": "Share of detected CPU cores the scheduler may commit to plugin\nguests on this node (1..=100).",
                "example": 80,
                "maximum": 100,
                "minimum": 1
              },
              "memory_budget_percent": {
                "type": "integer",
                "format": "int32",
                "description": "Share of detected memory the scheduler may commit (1..=100).",
                "example": 75,
                "maximum": 100,
                "minimum": 1
              },
              "schedulable": {
                "type": "boolean",
                "description": "False drains the node: the scheduler places nothing new here and\nmoves its replicas elsewhere. Hand-managed deployments still run.",
                "example": true
              },
              "cache_budget_bytes": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Disk space, in bytes, this node's image caches may use together,\noverriding the library's `core.cache` budget (16 MiB to 1 TiB).\nAbsent means the library budget applies.",
                "example": 536870912,
                "maximum": 1099511627776,
                "minimum": 16777216
              }
            }
          },
          "status": {
            "oneOf": [
              {
                "type": "object",
                "description": "The latest heartbeat; `null` before the first one.",
                "required": [
                  "node_id",
                  "incarnation",
                  "started_at",
                  "heartbeat_at",
                  "heartbeat_ms",
                  "interval_ms",
                  "cpu_millicores",
                  "cpu_budget_millicores",
                  "schedulable",
                  "reserved",
                  "running_guests",
                  "availability_ppm",
                  "online_since_ms"
                ],
                "properties": {
                  "node_id": {
                    "type": "string",
                    "description": "Reporting node.",
                    "example": "node-7f3a9c"
                  },
                  "incarnation": {
                    "type": "string",
                    "description": "Daemon process incarnation.",
                    "example": "01J9Q0Z5C7M8N3P4R6S8T9V0W1"
                  },
                  "started_at": {
                    "type": "string",
                    "description": "RFC 3339 daemon start.",
                    "example": "2026-10-28T06:00:00Z"
                  },
                  "heartbeat_at": {
                    "type": "string",
                    "description": "RFC 3339 heartbeat time.",
                    "example": "2026-10-28T19:07:00Z"
                  },
                  "heartbeat_ms": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Heartbeat time in Unix milliseconds.",
                    "example": 1793214420000
                  },
                  "interval_ms": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Promised interval in milliseconds.",
                    "example": 15000
                  },
                  "cpu_millicores": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Detected CPU capacity in millicores.",
                    "example": 8000
                  },
                  "memory_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Detected memory in bytes.",
                    "example": 16777216000
                  },
                  "cpu_budget_millicores": {
                    "type": "integer",
                    "format": "int64",
                    "description": "CPU the scheduler may commit, in millicores.",
                    "example": 6400
                  },
                  "memory_budget_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Memory the scheduler may commit, in bytes.",
                    "example": 12582912000
                  },
                  "schedulable": {
                    "type": "boolean",
                    "description": "False when an operator drained the node.",
                    "example": true
                  },
                  "cpu_used_millicores": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Measured CPU in use, in millicores.",
                    "example": 350
                  },
                  "memory_available_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Measured memory available, in bytes.",
                    "example": 9663676416
                  },
                  "files_dir_free_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Free bytes on the files directory's filesystem.",
                    "example": 214748364800
                  },
                  "reserved": {
                    "type": "object",
                    "description": "Ceilings of the guests running here.",
                    "required": [
                      "memory_bytes",
                      "cpu_millicores"
                    ],
                    "properties": {
                      "memory_bytes": {
                        "type": "integer",
                        "format": "int64",
                        "description": "Memory ceiling in bytes.",
                        "example": 1073741824
                      },
                      "cpu_millicores": {
                        "type": "integer",
                        "format": "int64",
                        "description": "CPU ceiling in millicores.",
                        "example": 1600
                      }
                    }
                  },
                  "running_guests": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Guests running here.",
                    "example": 2
                  },
                  "availability_ppm": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Recent online fraction in parts per million.",
                    "example": 998500
                  },
                  "online_since_ms": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Unix milliseconds the current online stretch began.",
                    "example": 1793167200000
                  },
                  "cache_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Bytes the node's image caches hold; `None` from a node that keeps\nnone.",
                    "example": 734003200
                  },
                  "cache_budget_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Bytes the node's image caches may hold.",
                    "example": 1073741824
                  },
                  "cache_swept_ms": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "When the node's last cache sweep finished, in Unix milliseconds.",
                    "example": 1793212800000
                  },
                  "public_origin": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "The address other installs reach this branch at, as it last\nreported it; `null` for a local branch and for a shared node. A\nbranch with one is a remote branch of this library.",
                    "example": "https://branch.example"
                  }
                }
              },
              {
                "type": "null"
              }
            ]
          },
          "this_node": {
            "type": "boolean",
            "description": "True for the node that answered the request.",
            "example": true
          },
          "kind": {
            "type": "string",
            "description": "`shared` for a node sharing this library's database, `branch` for an\nenrolled branch.",
            "enum": [
              "shared",
              "branch"
            ],
            "example": "shared"
          },
          "relationship": {
            "type": "string",
            "description": "How the node relates to this library: `shared_branch` (shares its\ndatabase), `remote_branch` (a branch reporting a public address in\nits heartbeat, `status.public_origin`) or `local_branch` (a branch\nwith none).",
            "enum": [
              "shared_branch",
              "remote_branch",
              "local_branch"
            ],
            "example": "remote_branch"
          },
          "owner_user_id": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "User id of the node's owner; `null` when unassigned.",
            "example": 7
          },
          "share_books": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "A branch's sharing setting, as it last reported it; `null` for a\nshared node.",
            "example": true
          },
          "include_branches": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "A branch's include-branches setting, as it last reported it;\n`null` for a shared node.",
            "example": false
          },
          "owner": {
            "oneOf": [
              {
                "type": "object",
                "description": "The owner, when the node has one and the user still exists.",
                "required": [
                  "user_id",
                  "role"
                ],
                "properties": {
                  "user_id": {
                    "type": "integer",
                    "format": "int64",
                    "description": "The owner's user id.",
                    "example": 7
                  },
                  "display_name": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Display name, or the login name when none is set; `null` when the\nuser has neither.",
                    "example": "Sam Reader"
                  },
                  "role": {
                    "type": "string",
                    "description": "The owner's role.",
                    "enum": [
                      "owner",
                      "administrator",
                      "member"
                    ]
                  }
                }
              },
              {
                "type": "null"
              }
            ]
          }
        }
      },
      "description": "The nodes the caller administers."
    },
    "capabilities": {
      "type": "object",
      "description": "What the caller may do with them.",
      "required": [
        "scope",
        "can_edit_runtime",
        "can_edit_labels",
        "can_assign_owner"
      ],
      "properties": {
        "scope": {
          "type": "string",
          "description": "`all` for the operator, `owned` for a node owner.",
          "enum": [
            "all",
            "owned"
          ]
        },
        "can_edit_runtime": {
          "type": "boolean",
          "description": "True: every caller may edit the runtime settings of the nodes it\nsees.",
          "example": true
        },
        "can_edit_labels": {
          "type": "boolean",
          "description": "True for the operator, who may replace placement labels.",
          "example": true
        },
        "can_assign_owner": {
          "type": "boolean",
          "description": "True for the operator, who may assign or clear a node's owner.",
          "example": true
        }
      }
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

Status `403`: The caller may not do this. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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

## `GET /api/config/shared-branches`

`GET /api/config/shared-branches`: the shared nodes (this one and its shared branches) for
Settings → Shared branches, shaped like `GET /api/config/nodes`. The operator or
an Owner sees every one (only the operator may remove one, with
`DELETE /api/config/nodes/{node_id}`); another user only the ones they
own, which the nodes listing already shows them. Branches are not listed.

**Access:** The operator, a signed-in Owner or Administrator, or a Member who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — This node and its shared branches. Body: [`SharedBranchesResponse`](schemas.md#sharedbranchesresponse) (`application/json`)
  - `node_id` *string*, **required** — Id of the node that answered.
  - `nodes` *array of [`NodeView`](schemas.md#nodeview)*, **required** — This node and the shared branches the caller may see.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The configuration could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/shared-branches' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/shared-branches' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/shared-branches",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/shared-branches", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/shared-branches")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: This node and its shared branches.

```json
{
  "node_id": "node-7f3a9c",
  "nodes": [
    {
      "node_id": "node-7f3a9c",
      "library_id": "c0ffee12-3456-4789-8abc-def012345678",
      "created_at": "2026-10-01T08:00:00Z",
      "heartbeat_at": "2026-10-28T19:07:00Z",
      "software_version": "0.3.1",
      "schema_state": "frozen@12+a1b2c3d4",
      "compatible": true,
      "target": "linux-x64-gnu",
      "labels": [
        "rack=a"
      ],
      "implicit_labels": [
        "node=node-7f3a9c",
        "target=linux-x64-gnu"
      ],
      "live": true,
      "runtime": {
        "label": "Living-room server",
        "cpu_budget_percent": 80,
        "memory_budget_percent": 75,
        "schedulable": true,
        "cache_budget_bytes": 536870912
      },
      "status": {
        "node_id": "node-7f3a9c",
        "incarnation": "01J9Q0Z5C7M8N3P4R6S8T9V0W1",
        "started_at": "2026-10-28T06:00:00Z",
        "heartbeat_at": "2026-10-28T19:07:00Z",
        "heartbeat_ms": 1793214420000,
        "interval_ms": 15000,
        "cpu_millicores": 8000,
        "memory_bytes": 16777216000,
        "cpu_budget_millicores": 6400,
        "memory_budget_bytes": 12582912000,
        "schedulable": true,
        "cpu_used_millicores": 350,
        "memory_available_bytes": 9663676416,
        "files_dir_free_bytes": 214748364800,
        "reserved": {
          "memory_bytes": 1073741824,
          "cpu_millicores": 1600
        },
        "running_guests": 2,
        "availability_ppm": 998500,
        "online_since_ms": 1793167200000,
        "cache_bytes": 734003200,
        "cache_budget_bytes": 1073741824,
        "cache_swept_ms": 1793212800000,
        "public_origin": "https://branch.example"
      },
      "this_node": true,
      "kind": "shared",
      "relationship": "remote_branch",
      "owner_user_id": 7,
      "share_books": true,
      "include_branches": false,
      "owner": {
        "user_id": 7,
        "display_name": "Sam Reader",
        "role": "owner"
      }
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/config/shared-branches` body.",
  "required": [
    "node_id",
    "nodes"
  ],
  "properties": {
    "node_id": {
      "type": "string",
      "description": "Id of the node that answered.",
      "example": "node-7f3a9c"
    },
    "nodes": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One node with its latest heartbeat, budget and owner.",
        "required": [
          "node_id",
          "library_id",
          "created_at",
          "heartbeat_at",
          "software_version",
          "schema_state",
          "compatible",
          "target",
          "labels",
          "implicit_labels",
          "live",
          "runtime",
          "this_node",
          "kind",
          "relationship"
        ],
        "properties": {
          "node_id": {
            "type": "string",
            "description": "Node id.",
            "example": "node-7f3a9c"
          },
          "library_id": {
            "type": "string",
            "description": "Library the node belongs to.",
            "example": "c0ffee12-3456-4789-8abc-def012345678"
          },
          "created_at": {
            "type": "string",
            "description": "RFC 3339 time the node first enrolled.",
            "example": "2026-10-01T08:00:00Z"
          },
          "heartbeat_at": {
            "type": "string",
            "description": "RFC 3339 time of the last heartbeat the node row recorded.",
            "example": "2026-10-28T19:07:00Z"
          },
          "software_version": {
            "type": "string",
            "description": "Bookclerk version the node runs.",
            "example": "0.3.1"
          },
          "schema_state": {
            "type": "string",
            "description": "Database schema the node last applied.",
            "example": "frozen@12+a1b2c3d4"
          },
          "compatible": {
            "type": "boolean",
            "description": "True when the node's own binary accepted the schema it reported;\nnot a comparison with this daemon's schema.",
            "example": true
          },
          "target": {
            "type": "string",
            "description": "Bookclerk target the node runs on.",
            "example": "linux-x64-gnu"
          },
          "labels": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Operator-set placement labels.",
            "example": [
              "rack=a"
            ]
          },
          "implicit_labels": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Labels the scheduler adds for the node (its id and target).",
            "example": [
              "node=node-7f3a9c",
              "target=linux-x64-gnu"
            ]
          },
          "live": {
            "type": "boolean",
            "description": "True while the last heartbeat is recent enough.",
            "example": true
          },
          "runtime": {
            "type": "object",
            "description": "The node's runtime document, or the defaults when it never wrote\none.",
            "required": [
              "label"
            ],
            "properties": {
              "label": {
                "type": "string",
                "description": "Operator label shown for this node. Empty is allowed.",
                "example": "Living-room server"
              },
              "cpu_budget_percent": {
                "type": "integer",
                "format": "int32",
                "description": "Share of detected CPU cores the scheduler may commit to plugin\nguests on this node (1..=100).",
                "example": 80,
                "maximum": 100,
                "minimum": 1
              },
              "memory_budget_percent": {
                "type": "integer",
                "format": "int32",
                "description": "Share of detected memory the scheduler may commit (1..=100).",
                "example": 75,
                "maximum": 100,
                "minimum": 1
              },
              "schedulable": {
                "type": "boolean",
                "description": "False drains the node: the scheduler places nothing new here and\nmoves its replicas elsewhere. Hand-managed deployments still run.",
                "example": true
              },
              "cache_budget_bytes": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Disk space, in bytes, this node's image caches may use together,\noverriding the library's `core.cache` budget (16 MiB to 1 TiB).\nAbsent means the library budget applies.",
                "example": 536870912,
                "maximum": 1099511627776,
                "minimum": 16777216
              }
            }
          },
          "status": {
            "oneOf": [
              {
                "type": "object",
                "description": "The latest heartbeat; `null` before the first one.",
                "required": [
                  "node_id",
                  "incarnation",
                  "started_at",
                  "heartbeat_at",
                  "heartbeat_ms",
                  "interval_ms",
                  "cpu_millicores",
                  "cpu_budget_millicores",
                  "schedulable",
                  "reserved",
                  "running_guests",
                  "availability_ppm",
                  "online_since_ms"
                ],
                "properties": {
                  "node_id": {
                    "type": "string",
                    "description": "Reporting node.",
                    "example": "node-7f3a9c"
                  },
                  "incarnation": {
                    "type": "string",
                    "description": "Daemon process incarnation.",
                    "example": "01J9Q0Z5C7M8N3P4R6S8T9V0W1"
                  },
                  "started_at": {
                    "type": "string",
                    "description": "RFC 3339 daemon start.",
                    "example": "2026-10-28T06:00:00Z"
                  },
                  "heartbeat_at": {
                    "type": "string",
                    "description": "RFC 3339 heartbeat time.",
                    "example": "2026-10-28T19:07:00Z"
                  },
                  "heartbeat_ms": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Heartbeat time in Unix milliseconds.",
                    "example": 1793214420000
                  },
                  "interval_ms": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Promised interval in milliseconds.",
                    "example": 15000
                  },
                  "cpu_millicores": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Detected CPU capacity in millicores.",
                    "example": 8000
                  },
                  "memory_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Detected memory in bytes.",
                    "example": 16777216000
                  },
                  "cpu_budget_millicores": {
                    "type": "integer",
                    "format": "int64",
                    "description": "CPU the scheduler may commit, in millicores.",
                    "example": 6400
                  },
                  "memory_budget_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Memory the scheduler may commit, in bytes.",
                    "example": 12582912000
                  },
                  "schedulable": {
                    "type": "boolean",
                    "description": "False when an operator drained the node.",
                    "example": true
                  },
                  "cpu_used_millicores": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Measured CPU in use, in millicores.",
                    "example": 350
                  },
                  "memory_available_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Measured memory available, in bytes.",
                    "example": 9663676416
                  },
                  "files_dir_free_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Free bytes on the files directory's filesystem.",
                    "example": 214748364800
                  },
                  "reserved": {
                    "type": "object",
                    "description": "Ceilings of the guests running here.",
                    "required": [
                      "memory_bytes",
                      "cpu_millicores"
                    ],
                    "properties": {
                      "memory_bytes": {
                        "type": "integer",
                        "format": "int64",
                        "description": "Memory ceiling in bytes.",
                        "example": 1073741824
                      },
                      "cpu_millicores": {
                        "type": "integer",
                        "format": "int64",
                        "description": "CPU ceiling in millicores.",
                        "example": 1600
                      }
                    }
                  },
                  "running_guests": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Guests running here.",
                    "example": 2
                  },
                  "availability_ppm": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Recent online fraction in parts per million.",
                    "example": 998500
                  },
                  "online_since_ms": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Unix milliseconds the current online stretch began.",
                    "example": 1793167200000
                  },
                  "cache_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Bytes the node's image caches hold; `None` from a node that keeps\nnone.",
                    "example": 734003200
                  },
                  "cache_budget_bytes": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Bytes the node's image caches may hold.",
                    "example": 1073741824
                  },
                  "cache_swept_ms": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "When the node's last cache sweep finished, in Unix milliseconds.",
                    "example": 1793212800000
                  },
                  "public_origin": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "The address other installs reach this branch at, as it last\nreported it; `null` for a local branch and for a shared node. A\nbranch with one is a remote branch of this library.",
                    "example": "https://branch.example"
                  }
                }
              },
              {
                "type": "null"
              }
            ]
          },
          "this_node": {
            "type": "boolean",
            "description": "True for the node that answered the request.",
            "example": true
          },
          "kind": {
            "type": "string",
            "description": "`shared` for a node sharing this library's database, `branch` for an\nenrolled branch.",
            "enum": [
              "shared",
              "branch"
            ],
            "example": "shared"
          },
          "relationship": {
            "type": "string",
            "description": "How the node relates to this library: `shared_branch` (shares its\ndatabase), `remote_branch` (a branch reporting a public address in\nits heartbeat, `status.public_origin`) or `local_branch` (a branch\nwith none).",
            "enum": [
              "shared_branch",
              "remote_branch",
              "local_branch"
            ],
            "example": "remote_branch"
          },
          "owner_user_id": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "User id of the node's owner; `null` when unassigned.",
            "example": 7
          },
          "share_books": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "A branch's sharing setting, as it last reported it; `null` for a\nshared node.",
            "example": true
          },
          "include_branches": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "A branch's include-branches setting, as it last reported it;\n`null` for a shared node.",
            "example": false
          },
          "owner": {
            "oneOf": [
              {
                "type": "object",
                "description": "The owner, when the node has one and the user still exists.",
                "required": [
                  "user_id",
                  "role"
                ],
                "properties": {
                  "user_id": {
                    "type": "integer",
                    "format": "int64",
                    "description": "The owner's user id.",
                    "example": 7
                  },
                  "display_name": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Display name, or the login name when none is set; `null` when the\nuser has neither.",
                    "example": "Sam Reader"
                  },
                  "role": {
                    "type": "string",
                    "description": "The owner's role.",
                    "enum": [
                      "owner",
                      "administrator",
                      "member"
                    ]
                  }
                }
              },
              {
                "type": "null"
              }
            ]
          }
        }
      },
      "description": "This node and the shared branches the caller may see."
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

Status `403`: The caller may not do this. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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

## `PUT /api/config/nodes/{node_id}/runtime`

`PUT /api/config/nodes/{node_id}/runtime`: edit a node's label, budgets,
or drain switch. Applies at that node's next heartbeat. A node owner may
edit only the nodes it owns.

**Access:** The operator, a signed-in Owner or Administrator, or a Member who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `node_id` (path) *string*, **required** — Node id.

### Request body

`application/json`: [`PutNodeRuntime`](schemas.md#putnoderuntime)

- `label` *string or null* — Operator label.
- `cpu_budget_percent` *integer (int32) or null* — Share of detected CPU the scheduler may commit (1..=100). At least 0.
- `memory_budget_percent` *integer (int32) or null* — Share of detected memory the scheduler may commit (1..=100). At least 0.
- `schedulable` *boolean or null* — False drains the node.
- `cache_budget_bytes` *integer (int64) or null* — This node's image cache budget in bytes (16 MiB to 1 TiB), overriding the library's `cache.budget_bytes`. `null` removes the override; leaving the field out keeps it as it is. At least 0.

### Responses

- `200` — The node's runtime settings after the change. Body: [`NodeRuntimeChanged`](schemas.md#noderuntimechanged) (`application/json`)
  - `node_id` *string*, **required** — Node id, trimmed.
  - `revision` *integer (int64)*, **required** — Compare-and-swap revision of the runtime document after the write.
  - `runtime` *[`NodeRuntimeSettingsV1`](schemas.md#noderuntimesettingsv1)*, **required** — The runtime document now stored.
- `400` — The body is invalid, or a budget is outside 1..=100. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A node owner named a node they do not own (an unknown node included). A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — The operator named a node that does not exist. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — Concurrent edits kept changing the document; retry. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes/<node_id>/runtime' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/nodes/<node_id>/runtime' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/nodes/<node_id>/runtime",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/nodes/<node_id>/runtime", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/nodes/<node_id>/runtime")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The node's runtime settings after the change.

```json
{
  "node_id": "node-7f3a9c",
  "revision": 4,
  "runtime": {
    "label": "Living-room server",
    "cpu_budget_percent": 80,
    "memory_budget_percent": 75,
    "schedulable": true,
    "cache_budget_bytes": 536870912
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "`PUT /api/config/nodes/{node_id}/runtime` body: the document after the\nwrite.",
  "required": [
    "node_id",
    "revision",
    "runtime"
  ],
  "properties": {
    "node_id": {
      "type": "string",
      "description": "Node id, trimmed.",
      "example": "node-7f3a9c"
    },
    "revision": {
      "type": "integer",
      "format": "int64",
      "description": "Compare-and-swap revision of the runtime document after the write.",
      "example": 4
    },
    "runtime": {
      "type": "object",
      "description": "The runtime document now stored.",
      "required": [
        "label"
      ],
      "properties": {
        "label": {
          "type": "string",
          "description": "Operator label shown for this node. Empty is allowed.",
          "example": "Living-room server"
        },
        "cpu_budget_percent": {
          "type": "integer",
          "format": "int32",
          "description": "Share of detected CPU cores the scheduler may commit to plugin\nguests on this node (1..=100).",
          "example": 80,
          "maximum": 100,
          "minimum": 1
        },
        "memory_budget_percent": {
          "type": "integer",
          "format": "int32",
          "description": "Share of detected memory the scheduler may commit (1..=100).",
          "example": 75,
          "maximum": 100,
          "minimum": 1
        },
        "schedulable": {
          "type": "boolean",
          "description": "False drains the node: the scheduler places nothing new here and\nmoves its replicas elsewhere. Hand-managed deployments still run.",
          "example": true
        },
        "cache_budget_bytes": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Disk space, in bytes, this node's image caches may use together,\noverriding the library's `core.cache` budget (16 MiB to 1 TiB).\nAbsent means the library budget applies.",
          "example": 536870912,
          "maximum": 1099511627776,
          "minimum": 16777216
        }
      }
    }
  }
}
```

Status `400`: The body is invalid, or a budget is outside 1..=100. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid configuration: cpu_budget_percent must be between 1 and 100"
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

Status `403`: A node owner named a node they do not own (an unknown node included). A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `404`: The operator named a node that does not exist.

```json
{
  "error": "node 9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e not found"
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

Status `409`: Concurrent edits kept changing the document; retry.

```json
{
  "error": "node.runtime for 9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e kept changing; retry"
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
Failed to deserialize the JSON body into the target type: label: invalid type: integer `1`, expected a string at line 1 column 10
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
- `503` — Job admission is paused. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ActionResponse`](schemas.md#actionresponse) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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

Status `503`: Job admission is paused. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

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
- `503` — Job admission is paused. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ActionResponse`](schemas.md#actionresponse) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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

Status `503`: Job admission is paused. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/library/listings/{id}/pin`

Pins a store listing to the edition it sells.

The listing, with its holdings and offers, moves to the edition at once,
and its recorded conflicts are dropped. Later scans keep it there. Only
the operator, an Owner or an Administrator may pin.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Listing id.

### Request body

`application/json`: [`PinBody`](schemas.md#pinbody)

- `edition_id` *string*, **required** — The edition the listing sells.

### Responses

- `200` — The pin, and the edition the listing left when it moved. Body: [`PinJson`](schemas.md#pinjson) (`application/json`)
  - `listing_id` *string*, **required** — The listing.
  - `edition_id` *string*, **required** — The edition it is pinned to, and now sells.
  - `moved_from` *string or null* — The edition it sold before, when the pin moved it; absent when it already sold the pinned edition.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such listing or edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The pin could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/listings/<id>/pin' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/listings/<id>/pin' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/library/listings/<id>/pin",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/listings/<id>/pin", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/library/listings/<id>/pin")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The pin, and the edition the listing left when it moved.

```json
{
  "listing_id": "2f4e6a8c-1b3d-4e5f-9a7b-8c6d4e2f0a1b",
  "edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
  "moved_from": "9a8b7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A listing's pin.",
  "required": [
    "listing_id",
    "edition_id"
  ],
  "properties": {
    "listing_id": {
      "type": "string",
      "description": "The listing.",
      "example": "2f4e6a8c-1b3d-4e5f-9a7b-8c6d4e2f0a1b"
    },
    "edition_id": {
      "type": "string",
      "description": "The edition it is pinned to, and now sells.",
      "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    },
    "moved_from": {
      "type": [
        "string",
        "null"
      ],
      "description": "The edition it sold before, when the pin moved it; absent when it\nalready sold the pinned edition.",
      "example": "9a8b7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d"
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

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `404`: No such listing or edition.

```json
{
  "error": "edition 6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b not found"
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
Failed to deserialize the JSON body into the target type: missing field `edition_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The pin could not be written.

```json
{
  "error": "the library could not be changed"
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

## `DELETE /api/library/listings/{id}/pin`

Removes a listing's pin.

The listing keeps selling the edition it was pinned to; only a new pin
moves it again.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Listing id.

### Responses

- `204` — The pin is removed. No body.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — The listing has no pin. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The pin could not be removed. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/listings/<id>/pin' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/listings/<id>/pin' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/library/listings/<id>/pin",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/listings/<id>/pin", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/library/listings/<id>/pin")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The pin is removed. No body.

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `404`: The listing has no pin.

```json
{
  "error": "listing 2f4e6a8c-1b3d-4e5f-9a7b-8c6d4e2f0a1b has no pin"
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

Status `500`: The pin could not be removed.

```json
{
  "error": "the library could not be changed"
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

## `GET /api/library/conflicts`

Pages the conflicts the resolver could not decide, for an Administrator
to settle with a pin.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `cursor` (query) *string* — `next_cursor` of the previous page.
- `limit` (query) *integer (int64)* — Page size, 1 to 200 (default 50).

### Responses

- `200` — One page of conflicts. Body: [`ConflictPage`](schemas.md#conflictpage) (`application/json`)
  - `conflicts` *array of [`ConflictJson`](schemas.md#conflictjson)*, **required** — The conflicts, by listing.
  - `next_cursor` *string or null* — Pass as `cursor` for the next page; absent on the last page.
- `400` — The cursor is not one this route made. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The conflicts could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/conflicts' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/conflicts' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/conflicts",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/conflicts", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/conflicts")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: One page of conflicts.

```json
{
  "conflicts": [
    {
      "listing_id": "2f4e6a8c-1b3d-4e5f-9a7b-8c6d4e2f0a1b",
      "kind": "strong_identifiers",
      "subject_ids": [
        "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
        "9a8b7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d"
      ],
      "plugin_key": "plugin:audible",
      "product_id": "B0CXYZ1234",
      "title": "Dune",
      "edition_id": "9a8b7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d",
      "updated_at": "2026-11-15T21:00:00Z"
    }
  ],
  "next_cursor": "eyJsaXN0aW5nX2lkIjoiMmY0ZTZhOGMiLCJraW5kIjoiYW1iaWd1b3VzX3dvcmsiLCJzdWJqZWN0X2lkcyI6IltdIn0"
}
```

Schema:

```json
{
  "type": "object",
  "description": "One page of conflicts.",
  "required": [
    "conflicts"
  ],
  "properties": {
    "conflicts": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One conflict the resolver could not decide.",
        "required": [
          "listing_id",
          "kind",
          "subject_ids",
          "plugin_key",
          "product_id",
          "title",
          "edition_id",
          "updated_at"
        ],
        "properties": {
          "listing_id": {
            "type": "string",
            "description": "The listing whose resolution conflicted.",
            "example": "2f4e6a8c-1b3d-4e5f-9a7b-8c6d4e2f0a1b"
          },
          "kind": {
            "type": "string",
            "description": "What conflicted: `strong_identifiers` (two editions share the\nlisting's strong identifiers), `work_identifiers` (its work\nidentifiers name two works), `ambiguous_work` (two works match its\ntitle and authors) or `work_kind_mismatch` (its work identifier names\na work of another kind).",
            "example": "strong_identifiers"
          },
          "subject_ids": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "The editions (`strong_identifiers`) or works (the other kinds)\ninvolved.",
            "example": [
              "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
              "9a8b7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d"
            ]
          },
          "plugin_key": {
            "type": "string",
            "description": "The listing's storefront PluginKey.",
            "example": "plugin:audible"
          },
          "product_id": {
            "type": "string",
            "description": "The store's product id.",
            "example": "B0CXYZ1234"
          },
          "title": {
            "type": "string",
            "description": "The listing's title as the store reported it.",
            "example": "Dune"
          },
          "edition_id": {
            "type": "string",
            "description": "The edition the listing sells now.",
            "example": "9a8b7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d"
          },
          "updated_at": {
            "type": "string",
            "description": "When the conflict was last seen (RFC 3339).",
            "example": "2026-11-15T21:00:00Z"
          }
        }
      },
      "description": "The conflicts, by listing."
    },
    "next_cursor": {
      "type": [
        "string",
        "null"
      ],
      "description": "Pass as `cursor` for the next page; absent on the last page.",
      "example": "eyJsaXN0aW5nX2lkIjoiMmY0ZTZhOGMiLCJraW5kIjoiYW1iaWd1b3VzX3dvcmsiLCJzdWJqZWN0X2lkcyI6IltdIn0"
    }
  }
}
```

Status `400`: The cursor is not one this route made.

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

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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
}
```

Status `500`: The conflicts could not be read.

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

## `GET /api/library/editions/{id}/chapters/claims`

Lists every source's chapter claim for an edition, with its chapters.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Edition id.

### Responses

- `200` — The claims, most recently changed first. Body: [`ChapterClaimsJson`](schemas.md#chapterclaimsjson) (`application/json`)
  - `claims` *array of [`ChapterClaimJson`](schemas.md#chapterclaimjson)*, **required** — The claims, most recently changed first.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/chapters/claims' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/chapters/claims' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/editions/<id>/chapters/claims",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/chapters/claims", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/editions/<id>/chapters/claims")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The claims, most recently changed first.

```json
{
  "claims": [
    {
      "source_kind": "person",
      "source_id": "plugin:audnexus",
      "quality": "strong_accurate",
      "digest": "9f2c4e6a8b0d1f3e5a7c9b1d3f5e7a9c2b4d6f8e0a1c3e5b7d9f1a3c5e7b9d0f",
      "chapters": [
        {
          "title": "Chapter 1: The Boy Who Lived",
          "start_ms": 33000,
          "depth": 0
        }
      ],
      "runtime_ms": 41400000,
      "input_digest": "4b7d9f1a3c5e7b9d0f9f2c4e6a8b0d1f3e5a7c9b1d3f5e7a9c2b4d6f8e0a1c3e",
      "plays": true,
      "pinned_at": "2026-11-20T19:30:00Z",
      "created_by_user_id": 2,
      "updated_at": "2026-11-20T19:30:00Z"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "Every chapter claim for an edition.",
  "required": [
    "claims"
  ],
  "properties": {
    "claims": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One source's chapter claim for an edition.",
        "required": [
          "source_kind",
          "source_id",
          "quality",
          "digest",
          "plays",
          "updated_at"
        ],
        "properties": {
          "source_kind": {
            "type": "string",
            "description": "The kind of source.",
            "enum": [
              "person",
              "outside_edit",
              "processed",
              "provider",
              "listing",
              "file",
              "part_names"
            ]
          },
          "source_id": {
            "type": "string",
            "description": "Which source of that kind (a PluginKey; empty for an edit or for\nBookclerk's own processing).",
            "example": "plugin:audnexus"
          },
          "quality": {
            "type": "string",
            "description": "Where a machine source ranks within its kind: `strong_accurate`,\n`strong_estimated` or `weak` for a provider, `chapters`, `tracks` or\n`unknown` for a store listing; empty otherwise.",
            "example": "strong_accurate"
          },
          "digest": {
            "type": "string",
            "description": "SHA-256 of the claimed list, 64 lower-case hex characters.",
            "example": "9f2c4e6a8b0d1f3e5a7c9b1d3f5e7a9c2b4d6f8e0a1c3e5b7d9f1a3c5e7b9d0f"
          },
          "chapters": {
            "type": [
              "array",
              "null"
            ],
            "items": {
              "type": "object",
              "description": "One chapter of a checked list.\n\nNamed `EditionChapter` in the OpenAPI documents, apart from the player\napps' `Chapter`.",
              "required": [
                "title",
                "start_ms",
                "depth"
              ],
              "properties": {
                "title": {
                  "type": "string",
                  "description": "Chapter title; never empty, at most 512 bytes, no control\ncharacters.",
                  "example": "Chapter 1: The Boy Who Lived"
                },
                "start_ms": {
                  "type": "integer",
                  "format": "int64",
                  "description": "Start, in milliseconds from the start of the edition's timeline.",
                  "example": 33000,
                  "minimum": 0
                },
                "depth": {
                  "type": "integer",
                  "format": "int32",
                  "description": "Nesting depth; 0 for a top-level chapter, at most 7. A chapter at\ndepth n + 1 is nested under the nearest earlier chapter at depth n.",
                  "example": 0,
                  "minimum": 0
                }
              }
            },
            "description": "The claimed chapters; absent while the list's write is unfinished."
          },
          "runtime_ms": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Length of the timeline the list was measured on, in milliseconds.",
            "example": 41400000,
            "minimum": 0
          },
          "input_digest": {
            "type": [
              "string",
              "null"
            ],
            "description": "For a processed claim, the digest of the provider list it was\naligned from.",
            "example": "4b7d9f1a3c5e7b9d0f9f2c4e6a8b0d1f3e5a7c9b1d3f5e7a9c2b4d6f8e0a1c3e"
          },
          "plays": {
            "type": "boolean",
            "description": "Whether the edition plays this claim now.",
            "example": true
          },
          "pinned_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "When an Administrator pinned it (RFC 3339); absent when unpinned.",
            "example": "2026-11-20T19:30:00Z"
          },
          "created_by_user_id": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "The person whose edit it is; absent for a machine source or the\noperator.",
            "example": 2
          },
          "updated_at": {
            "type": "string",
            "description": "When it last changed (RFC 3339).",
            "example": "2026-11-20T19:30:00Z"
          }
        }
      },
      "description": "The claims, most recently changed first."
    }
  }
}
```

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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
}
```

Status `404`: No such edition.

```json
{
  "error": "edition 6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b not found"
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
  "error": "the library could not be changed"
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

## `POST /api/library/editions/{id}/chapters/realign`

Queues the alignment of a metadata provider's chapters to the
edition's stored copy.

The job asks the providers for the edition's chapters by its ASIN,
reads a stored copy, snaps each chapter start to the spoken title, and
records the provider's list and the aligned list as claims; the edition
plays the aligned list unless an edit or a pin says otherwise.

Only the operator, an Owner, or an Administrator with the "Fix the
catalog" switch may queue one; the route admits any signed-in user and
answers everyone else 403.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Edition id.

### Responses

- `202` — The alignment is queued. Body: [`ChapterAlignJson`](schemas.md#chapteralignjson) (`application/json`)
  - `job_id` *string*, **required** — The `chapter_align` job that does the work; it answers through the jobs API. An alignment already queued for the edition is returned rather than a second one.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The edition has no ASIN to ask providers by. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The job could not be queued. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The job queue is full. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/chapters/realign' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/chapters/realign' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/library/editions/<id>/chapters/realign",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/chapters/realign", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/library/editions/<id>/chapters/realign")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `202`: The alignment is queued.

```json
{
  "job_id": "0f3c2a9e-5b1d-4c7e-8a6f-2d4b9e1c7a35"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A queued chapter alignment.",
  "required": [
    "job_id"
  ],
  "properties": {
    "job_id": {
      "type": "string",
      "description": "The `chapter_align` job that does the work; it answers through the\njobs API. An alignment already queued for the edition is returned\nrather than a second one.",
      "example": "0f3c2a9e-5b1d-4c7e-8a6f-2d4b9e1c7a35"
    }
  }
}
```

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `404`: No such edition.

```json
{
  "error": "edition 6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b not found"
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

Status `409`: The edition has no ASIN to ask providers by.

```json
{
  "error": "the edition has no ASIN to ask metadata providers by"
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

Status `500`: The job could not be queued.

```json
{
  "error": "the library could not be changed"
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

Status `503`: The job queue is full. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "the job queue is full; try again later"
}
```

Schema:

```json
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
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/library/editions/{id}/chapters`

Reads the chapters an edition plays.

The list comes from the claim the resolver chose: a pinned claim, else
an Administrator's edit, else the best machine source (Bookclerk's own
aligned chapters first).

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Edition id (an `id` in a work's `editions`).

### Responses

- `200` — The chapters; an empty list with no source when the edition has none yet. Body: [`EditionChaptersJson`](schemas.md#editionchaptersjson) (`application/json`)
  - `edition_id` *string*, **required** — The edition.
  - `chapters` *array of [`EditionChapter`](schemas.md#editionchapter)*, **required** — The chapters in pre-order, starts never decreasing; empty when the edition has none or an Administrator cleared them.
  - `source_kind` *[`ChapterSourceKind`](schemas.md#chaptersourcekind) or null* — The kind of source the chapters came from; absent when the edition has no chapters yet.
  - `source_id` *string or null* — Which source of that kind (a PluginKey; empty for an edit or for Bookclerk's own processing).
  - `runtime_ms` *integer (int64) or null* — Length of the timeline the starts were measured on, in milliseconds. At least 0.
- `401` — The session ended. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such edition, or one outside the caller's scope. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/chapters' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/chapters' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/editions/<id>/chapters",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/chapters", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/editions/<id>/chapters")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The chapters; an empty list with no source when the edition has none yet.

```json
{
  "edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
  "chapters": [
    {
      "title": "Chapter 1: The Boy Who Lived",
      "start_ms": 33000,
      "depth": 0
    }
  ],
  "source_kind": "person",
  "source_id": "",
  "runtime_ms": 41400000
}
```

Schema:

```json
{
  "type": "object",
  "description": "The chapters an edition plays.",
  "required": [
    "edition_id",
    "chapters"
  ],
  "properties": {
    "edition_id": {
      "type": "string",
      "description": "The edition.",
      "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    },
    "chapters": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One chapter of a checked list.\n\nNamed `EditionChapter` in the OpenAPI documents, apart from the player\napps' `Chapter`.",
        "required": [
          "title",
          "start_ms",
          "depth"
        ],
        "properties": {
          "title": {
            "type": "string",
            "description": "Chapter title; never empty, at most 512 bytes, no control\ncharacters.",
            "example": "Chapter 1: The Boy Who Lived"
          },
          "start_ms": {
            "type": "integer",
            "format": "int64",
            "description": "Start, in milliseconds from the start of the edition's timeline.",
            "example": 33000,
            "minimum": 0
          },
          "depth": {
            "type": "integer",
            "format": "int32",
            "description": "Nesting depth; 0 for a top-level chapter, at most 7. A chapter at\ndepth n + 1 is nested under the nearest earlier chapter at depth n.",
            "example": 0,
            "minimum": 0
          }
        }
      },
      "description": "The chapters in pre-order, starts never decreasing; empty when the\nedition has none or an Administrator cleared them."
    },
    "source_kind": {
      "oneOf": [
        {
          "type": "string",
          "description": "The kind of source the chapters came from; absent when the edition\nhas no chapters yet.",
          "enum": [
            "person",
            "outside_edit",
            "processed",
            "provider",
            "listing",
            "file",
            "part_names"
          ]
        },
        {
          "type": "null"
        }
      ]
    },
    "source_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Which source of that kind (a PluginKey; empty for an edit or for\nBookclerk's own processing).",
      "example": ""
    },
    "runtime_ms": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Length of the timeline the starts were measured on, in milliseconds.",
      "example": 41400000,
      "minimum": 0
    }
  }
}
```

Status `401`: The session ended.

```json
{
  "error": "sign in again"
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

Status `404`: No such edition, or one outside the caller's scope.

```json
{
  "error": "edition 6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b not found"
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

## `PUT /api/library/editions/{id}/chapters`

Sets an edition's chapters by hand.

The edit is checked strictly: what is stored is exactly what was sent.
It wins over every machine source until it is removed or another claim
is pinned. An empty list clears the chapters.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Edition id.

### Request body

`application/json`: [`ChapterEditBody`](schemas.md#chaptereditbody)

- `chapters` *array of [`ChapterEditEntry`](schemas.md#chaptereditentry)*, **required** — The chapters, at most 2,000. At most 2000 items.

### Responses

- `200` — The chapters the edition plays now (a pinned claim still wins over the edit). Body: [`EditionChaptersJson`](schemas.md#editionchaptersjson) (`application/json`)
  - `edition_id` *string*, **required** — The edition.
  - `chapters` *array of [`EditionChapter`](schemas.md#editionchapter)*, **required** — The chapters in pre-order, starts never decreasing; empty when the edition has none or an Administrator cleared them.
  - `source_kind` *[`ChapterSourceKind`](schemas.md#chaptersourcekind) or null* — The kind of source the chapters came from; absent when the edition has no chapters yet.
  - `source_id` *string or null* — Which source of that kind (a PluginKey; empty for an edit or for Bookclerk's own processing).
  - `runtime_ms` *integer (int64) or null* — Length of the timeline the starts were measured on, in milliseconds. At least 0.
- `400` — The body is not a chapter list, or a chapter breaks a rule. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The body is over 2 MiB. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The body is not sent as JSON. As `text/plain`, when the request did not declare a JSON body. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The chapters could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/chapters' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"chapters": [{"title": "Chapter 1: The Boy Who Lived", "start_ms": 33000}]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/chapters' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"chapters": [{"title": "Chapter 1: The Boy Who Lived", "start_ms": 33000}]}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/library/editions/<id>/chapters",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"chapters": [{"title": "Chapter 1: The Boy Who Lived", "start_ms": 33000}]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/chapters", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"chapters": [{"title": "Chapter 1: The Boy Who Lived", "start_ms": 33000}]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/library/editions/<id>/chapters")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"chapters": [{"title": "Chapter 1: The Boy Who Lived", "start_ms": 33000}]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The chapters the edition plays now (a pinned claim still wins over the edit).

```json
{
  "edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
  "chapters": [
    {
      "title": "Chapter 1: The Boy Who Lived",
      "start_ms": 33000,
      "depth": 0
    }
  ],
  "source_kind": "person",
  "source_id": "",
  "runtime_ms": 41400000
}
```

Schema:

```json
{
  "type": "object",
  "description": "The chapters an edition plays.",
  "required": [
    "edition_id",
    "chapters"
  ],
  "properties": {
    "edition_id": {
      "type": "string",
      "description": "The edition.",
      "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    },
    "chapters": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One chapter of a checked list.\n\nNamed `EditionChapter` in the OpenAPI documents, apart from the player\napps' `Chapter`.",
        "required": [
          "title",
          "start_ms",
          "depth"
        ],
        "properties": {
          "title": {
            "type": "string",
            "description": "Chapter title; never empty, at most 512 bytes, no control\ncharacters.",
            "example": "Chapter 1: The Boy Who Lived"
          },
          "start_ms": {
            "type": "integer",
            "format": "int64",
            "description": "Start, in milliseconds from the start of the edition's timeline.",
            "example": 33000,
            "minimum": 0
          },
          "depth": {
            "type": "integer",
            "format": "int32",
            "description": "Nesting depth; 0 for a top-level chapter, at most 7. A chapter at\ndepth n + 1 is nested under the nearest earlier chapter at depth n.",
            "example": 0,
            "minimum": 0
          }
        }
      },
      "description": "The chapters in pre-order, starts never decreasing; empty when the\nedition has none or an Administrator cleared them."
    },
    "source_kind": {
      "oneOf": [
        {
          "type": "string",
          "description": "The kind of source the chapters came from; absent when the edition\nhas no chapters yet.",
          "enum": [
            "person",
            "outside_edit",
            "processed",
            "provider",
            "listing",
            "file",
            "part_names"
          ]
        },
        {
          "type": "null"
        }
      ]
    },
    "source_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Which source of that kind (a PluginKey; empty for an edit or for\nBookclerk's own processing).",
      "example": ""
    },
    "runtime_ms": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Length of the timeline the starts were measured on, in milliseconds.",
      "example": 41400000,
      "minimum": 0
    }
  }
}
```

Status `400`: The body is not a chapter list, or a chapter breaks a rule. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "chapter 3 starts before chapter 2"
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

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `404`: No such edition.

```json
{
  "error": "edition 6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b not found"
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

Status `413`: The body is over 2 MiB.

```json
{
  "error": "the body is at most 2097152 bytes"
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

Status `415`: The body is not sent as JSON. As `text/plain`, when the request did not declare a JSON body.

```json
{
  "error": "send the body with Content-Type: application/json"
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

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `chapters` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The chapters could not be written.

```json
{
  "error": "the library could not be changed"
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

## `DELETE /api/library/editions/{id}/chapters`

Removes an Administrator's chapter edit, so the edition plays the best
remaining claim again.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Edition id.

### Responses

- `200` — The chapters the edition plays now. Body: [`EditionChaptersJson`](schemas.md#editionchaptersjson) (`application/json`)
  - `edition_id` *string*, **required** — The edition.
  - `chapters` *array of [`EditionChapter`](schemas.md#editionchapter)*, **required** — The chapters in pre-order, starts never decreasing; empty when the edition has none or an Administrator cleared them.
  - `source_kind` *[`ChapterSourceKind`](schemas.md#chaptersourcekind) or null* — The kind of source the chapters came from; absent when the edition has no chapters yet.
  - `source_id` *string or null* — Which source of that kind (a PluginKey; empty for an edit or for Bookclerk's own processing).
  - `runtime_ms` *integer (int64) or null* — Length of the timeline the starts were measured on, in milliseconds. At least 0.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The edit could not be removed. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/chapters' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/chapters' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/library/editions/<id>/chapters",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/chapters", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/library/editions/<id>/chapters")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The chapters the edition plays now.

```json
{
  "edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
  "chapters": [
    {
      "title": "Chapter 1: The Boy Who Lived",
      "start_ms": 33000,
      "depth": 0
    }
  ],
  "source_kind": "person",
  "source_id": "",
  "runtime_ms": 41400000
}
```

Schema:

```json
{
  "type": "object",
  "description": "The chapters an edition plays.",
  "required": [
    "edition_id",
    "chapters"
  ],
  "properties": {
    "edition_id": {
      "type": "string",
      "description": "The edition.",
      "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    },
    "chapters": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One chapter of a checked list.\n\nNamed `EditionChapter` in the OpenAPI documents, apart from the player\napps' `Chapter`.",
        "required": [
          "title",
          "start_ms",
          "depth"
        ],
        "properties": {
          "title": {
            "type": "string",
            "description": "Chapter title; never empty, at most 512 bytes, no control\ncharacters.",
            "example": "Chapter 1: The Boy Who Lived"
          },
          "start_ms": {
            "type": "integer",
            "format": "int64",
            "description": "Start, in milliseconds from the start of the edition's timeline.",
            "example": 33000,
            "minimum": 0
          },
          "depth": {
            "type": "integer",
            "format": "int32",
            "description": "Nesting depth; 0 for a top-level chapter, at most 7. A chapter at\ndepth n + 1 is nested under the nearest earlier chapter at depth n.",
            "example": 0,
            "minimum": 0
          }
        }
      },
      "description": "The chapters in pre-order, starts never decreasing; empty when the\nedition has none or an Administrator cleared them."
    },
    "source_kind": {
      "oneOf": [
        {
          "type": "string",
          "description": "The kind of source the chapters came from; absent when the edition\nhas no chapters yet.",
          "enum": [
            "person",
            "outside_edit",
            "processed",
            "provider",
            "listing",
            "file",
            "part_names"
          ]
        },
        {
          "type": "null"
        }
      ]
    },
    "source_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Which source of that kind (a PluginKey; empty for an edit or for\nBookclerk's own processing).",
      "example": ""
    },
    "runtime_ms": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Length of the timeline the starts were measured on, in milliseconds.",
      "example": 41400000,
      "minimum": 0
    }
  }
}
```

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `404`: No such edition.

```json
{
  "error": "edition 6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b not found"
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

Status `500`: The edit could not be removed.

```json
{
  "error": "the library could not be changed"
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

## `PUT /api/library/editions/{id}/chapters/pin`

Pins one claim, so the edition plays it whatever the others say.

Pinning replaces any earlier pin on the edition's chapters.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Edition id.

### Request body

`application/json`: [`ChapterPinBody`](schemas.md#chapterpinbody)

- `source_kind` *[`ChapterSourceKind`](schemas.md#chaptersourcekind)*, **required** — The claim's source kind, as the claims list names it.
- `source_id` *string or null* — The claim's source id (a PluginKey; empty for an edit or for Bookclerk's own processing). Left out reads as empty.

### Responses

- `200` — The chapters the edition plays now. Body: [`EditionChaptersJson`](schemas.md#editionchaptersjson) (`application/json`)
  - `edition_id` *string*, **required** — The edition.
  - `chapters` *array of [`EditionChapter`](schemas.md#editionchapter)*, **required** — The chapters in pre-order, starts never decreasing; empty when the edition has none or an Administrator cleared them.
  - `source_kind` *[`ChapterSourceKind`](schemas.md#chaptersourcekind) or null* — The kind of source the chapters came from; absent when the edition has no chapters yet.
  - `source_id` *string or null* — Which source of that kind (a PluginKey; empty for an edit or for Bookclerk's own processing).
  - `runtime_ms` *integer (int64) or null* — Length of the timeline the starts were measured on, in milliseconds. At least 0.
- `400` — The body does not name a claim. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition, or it has no such claim. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The body is over 1 KiB. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The body is not sent as JSON. As `text/plain`, when the request did not declare a JSON body. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The pin could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/chapters/pin' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"source_kind": "person"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/chapters/pin' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"source_kind": "person"}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/library/editions/<id>/chapters/pin",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"source_kind": "person"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/chapters/pin", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"source_kind": "person"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/library/editions/<id>/chapters/pin")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"source_kind": "person"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The chapters the edition plays now.

```json
{
  "edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
  "chapters": [
    {
      "title": "Chapter 1: The Boy Who Lived",
      "start_ms": 33000,
      "depth": 0
    }
  ],
  "source_kind": "person",
  "source_id": "",
  "runtime_ms": 41400000
}
```

Schema:

```json
{
  "type": "object",
  "description": "The chapters an edition plays.",
  "required": [
    "edition_id",
    "chapters"
  ],
  "properties": {
    "edition_id": {
      "type": "string",
      "description": "The edition.",
      "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    },
    "chapters": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One chapter of a checked list.\n\nNamed `EditionChapter` in the OpenAPI documents, apart from the player\napps' `Chapter`.",
        "required": [
          "title",
          "start_ms",
          "depth"
        ],
        "properties": {
          "title": {
            "type": "string",
            "description": "Chapter title; never empty, at most 512 bytes, no control\ncharacters.",
            "example": "Chapter 1: The Boy Who Lived"
          },
          "start_ms": {
            "type": "integer",
            "format": "int64",
            "description": "Start, in milliseconds from the start of the edition's timeline.",
            "example": 33000,
            "minimum": 0
          },
          "depth": {
            "type": "integer",
            "format": "int32",
            "description": "Nesting depth; 0 for a top-level chapter, at most 7. A chapter at\ndepth n + 1 is nested under the nearest earlier chapter at depth n.",
            "example": 0,
            "minimum": 0
          }
        }
      },
      "description": "The chapters in pre-order, starts never decreasing; empty when the\nedition has none or an Administrator cleared them."
    },
    "source_kind": {
      "oneOf": [
        {
          "type": "string",
          "description": "The kind of source the chapters came from; absent when the edition\nhas no chapters yet.",
          "enum": [
            "person",
            "outside_edit",
            "processed",
            "provider",
            "listing",
            "file",
            "part_names"
          ]
        },
        {
          "type": "null"
        }
      ]
    },
    "source_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Which source of that kind (a PluginKey; empty for an edit or for\nBookclerk's own processing).",
      "example": ""
    },
    "runtime_ms": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Length of the timeline the starts were measured on, in milliseconds.",
      "example": 41400000,
      "minimum": 0
    }
  }
}
```

Status `400`: The body does not name a claim. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "pin body: unknown variant `wizard`"
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

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `404`: No such edition, or it has no such claim.

```json
{
  "error": "metadata claim listing:plugin:audible not found"
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

Status `413`: The body is over 1 KiB.

```json
{
  "error": "the body is at most 1024 bytes"
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

Status `415`: The body is not sent as JSON. As `text/plain`, when the request did not declare a JSON body.

```json
{
  "error": "send the body with Content-Type: application/json"
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

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `source_kind` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The pin could not be written.

```json
{
  "error": "the library could not be changed"
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

## `DELETE /api/library/editions/{id}/chapters/pin`

Removes the pin on an edition's chapters; the resolver chooses again.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Edition id.

### Responses

- `200` — The chapters the edition plays now. Body: [`EditionChaptersJson`](schemas.md#editionchaptersjson) (`application/json`)
  - `edition_id` *string*, **required** — The edition.
  - `chapters` *array of [`EditionChapter`](schemas.md#editionchapter)*, **required** — The chapters in pre-order, starts never decreasing; empty when the edition has none or an Administrator cleared them.
  - `source_kind` *[`ChapterSourceKind`](schemas.md#chaptersourcekind) or null* — The kind of source the chapters came from; absent when the edition has no chapters yet.
  - `source_id` *string or null* — Which source of that kind (a PluginKey; empty for an edit or for Bookclerk's own processing).
  - `runtime_ms` *integer (int64) or null* — Length of the timeline the starts were measured on, in milliseconds. At least 0.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The pin could not be removed. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/chapters/pin' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/chapters/pin' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/library/editions/<id>/chapters/pin",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/chapters/pin", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/library/editions/<id>/chapters/pin")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The chapters the edition plays now.

```json
{
  "edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
  "chapters": [
    {
      "title": "Chapter 1: The Boy Who Lived",
      "start_ms": 33000,
      "depth": 0
    }
  ],
  "source_kind": "person",
  "source_id": "",
  "runtime_ms": 41400000
}
```

Schema:

```json
{
  "type": "object",
  "description": "The chapters an edition plays.",
  "required": [
    "edition_id",
    "chapters"
  ],
  "properties": {
    "edition_id": {
      "type": "string",
      "description": "The edition.",
      "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    },
    "chapters": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One chapter of a checked list.\n\nNamed `EditionChapter` in the OpenAPI documents, apart from the player\napps' `Chapter`.",
        "required": [
          "title",
          "start_ms",
          "depth"
        ],
        "properties": {
          "title": {
            "type": "string",
            "description": "Chapter title; never empty, at most 512 bytes, no control\ncharacters.",
            "example": "Chapter 1: The Boy Who Lived"
          },
          "start_ms": {
            "type": "integer",
            "format": "int64",
            "description": "Start, in milliseconds from the start of the edition's timeline.",
            "example": 33000,
            "minimum": 0
          },
          "depth": {
            "type": "integer",
            "format": "int32",
            "description": "Nesting depth; 0 for a top-level chapter, at most 7. A chapter at\ndepth n + 1 is nested under the nearest earlier chapter at depth n.",
            "example": 0,
            "minimum": 0
          }
        }
      },
      "description": "The chapters in pre-order, starts never decreasing; empty when the\nedition has none or an Administrator cleared them."
    },
    "source_kind": {
      "oneOf": [
        {
          "type": "string",
          "description": "The kind of source the chapters came from; absent when the edition\nhas no chapters yet.",
          "enum": [
            "person",
            "outside_edit",
            "processed",
            "provider",
            "listing",
            "file",
            "part_names"
          ]
        },
        {
          "type": "null"
        }
      ]
    },
    "source_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Which source of that kind (a PluginKey; empty for an edit or for\nBookclerk's own processing).",
      "example": ""
    },
    "runtime_ms": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Length of the timeline the starts were measured on, in milliseconds.",
      "example": 41400000,
      "minimum": 0
    }
  }
}
```

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `404`: No such edition.

```json
{
  "error": "edition 6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b not found"
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

Status `500`: The pin could not be removed.

```json
{
  "error": "the library could not be changed"
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

## `POST /api/library/works/{id}/split`

Splits a work an automatic merge joined wrongly.

The named expressions move, with their editions, listings and holdings,
to a new work that copies the work's title, credits, series and each
person's own state. New listings with the same title keep joining the
original work. Only the operator, an Owner or an Administrator may
split.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Work id.

### Request body

`application/json`: [`WorkSplitBody`](schemas.md#worksplitbody)

- `expression_ids` *array of string*, **required** — The expressions to move to the new work (1 to 12 distinct ids, each at most 64 bytes), from the work detail's editions (`expression_id`).

### Responses

- `200` — The new work. Body: [`SplitJson`](schemas.md#splitjson) (`application/json`)
  - `work_id` *string*, **required** — The new work the expressions moved to.
  - `split_from` *string*, **required** — The work that was split.
- `400` — No expression, too many, an id over 64 bytes, or the work would be left empty. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such work, or an expression is not the work's. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The work changed while it was being split. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The split could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/works/<id>/split' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"expression_ids": ["4e5f6a7b-8c9d-4e0f-a1b2-c3d4e5f6a7b8"]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/works/<id>/split' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"expression_ids": ["4e5f6a7b-8c9d-4e0f-a1b2-c3d4e5f6a7b8"]}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/library/works/<id>/split",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"expression_ids": ["4e5f6a7b-8c9d-4e0f-a1b2-c3d4e5f6a7b8"]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/works/<id>/split", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"expression_ids": ["4e5f6a7b-8c9d-4e0f-a1b2-c3d4e5f6a7b8"]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/library/works/<id>/split")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"expression_ids": ["4e5f6a7b-8c9d-4e0f-a1b2-c3d4e5f6a7b8"]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The new work.

```json
{
  "work_id": "7d6c5b4a-3928-4f1e-8d7c-6b5a49382716",
  "split_from": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"
}
```

Schema:

```json
{
  "type": "object",
  "description": "The result of a split.",
  "required": [
    "work_id",
    "split_from"
  ],
  "properties": {
    "work_id": {
      "type": "string",
      "description": "The new work the expressions moved to.",
      "example": "7d6c5b4a-3928-4f1e-8d7c-6b5a49382716"
    },
    "split_from": {
      "type": "string",
      "description": "The work that was split.",
      "example": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"
    }
  }
}
```

Status `400`: No expression, too many, an id over 64 bytes, or the work would be left empty. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "a split must leave the work at least one expression"
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

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `404`: No such work, or an expression is not the work's.

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

Status `409`: The work changed while it was being split.

```json
{
  "error": "the work changed while it was being split; try again"
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
Failed to deserialize the JSON body into the target type: missing field `expression_ids` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The split could not be written.

```json
{
  "error": "the library could not be changed"
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

## `GET /api/library/works/{id}/tags`

Reads a work's library tags.

Only the operator, an Owner or an Administrator may call it; patrons see
tags through the Library's tag filter, inside their scope.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — A top-level work id.

### Responses

- `200` — The work's tags. Body: [`WorkTagsJson`](schemas.md#worktagsjson) (`application/json`)
  - `work_id` *string*, **required** — The work.
  - `tags` *array of [`WorkTag`](schemas.md#worktag)*, **required** — Every tag, by key, with where it came from.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such work, or a podcast episode (tags go on its show). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The tags could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/works/<id>/tags' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/works/<id>/tags' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/works/<id>/tags",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/works/<id>/tags", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/works/<id>/tags")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The work's tags.

```json
{
  "work_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
  "tags": [
    {
      "value": "Kids",
      "source": "admin"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "A work's library tags.",
  "required": [
    "work_id",
    "tags"
  ],
  "properties": {
    "work_id": {
      "type": "string",
      "description": "The work.",
      "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    },
    "tags": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One library tag on a work.",
        "required": [
          "value",
          "source"
        ],
        "properties": {
          "value": {
            "type": "string",
            "description": "The tag as written.",
            "example": "Kids"
          },
          "source": {
            "type": "string",
            "description": "Where it came from.",
            "enum": [
              "admin",
              "sidecar"
            ]
          }
        }
      },
      "description": "Every tag, by key, with where it came from."
    }
  }
}
```

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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
}
```

Status `404`: No such work, or a podcast episode (tags go on its show).

```json
{
  "error": "work 6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b not found"
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

Status `500`: The tags could not be read.

```json
{
  "error": "the library tags could not be read or saved"
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

## `PUT /api/library/works/{id}/tags`

Replaces the tags an Administrator set on a work.

Sidecar tags stay. The Library filter and patron scopes see the change
at once. Only the operator, an Owner, or an Administrator whom an Owner
lets set patron limits may call it.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — A top-level work id.

### Request body

`application/json`: [`WorkTagsBody`](schemas.md#worktagsbody)

- `tags` *array of string*, **required** — The tags, replacing the ones set in Bookclerk (sidecar tags stay). Blanks and repeats, ignoring case, are dropped; at most 32 tags of at most 256 bytes each. At most 32 items.

### Responses

- `200` — The work's tags afterwards. Body: [`WorkTagsJson`](schemas.md#worktagsjson) (`application/json`)
  - `work_id` *string*, **required** — The work.
  - `tags` *array of [`WorkTag`](schemas.md#worktag)*, **required** — Every tag, by key, with where it came from.
- `400` — Too many tags, or one too long. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator, or an Owner turned off setting patron limits for them. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such work, or a podcast episode (tags go on its show). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The tags could not be saved. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/works/<id>/tags' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"tags": ["Kids", "Bedtime"]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/works/<id>/tags' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"tags": ["Kids", "Bedtime"]}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/library/works/<id>/tags",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"tags": ["Kids", "Bedtime"]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/works/<id>/tags", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"tags": ["Kids", "Bedtime"]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/library/works/<id>/tags")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"tags": ["Kids", "Bedtime"]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The work's tags afterwards.

```json
{
  "work_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
  "tags": [
    {
      "value": "Kids",
      "source": "admin"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "A work's library tags.",
  "required": [
    "work_id",
    "tags"
  ],
  "properties": {
    "work_id": {
      "type": "string",
      "description": "The work.",
      "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    },
    "tags": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One library tag on a work.",
        "required": [
          "value",
          "source"
        ],
        "properties": {
          "value": {
            "type": "string",
            "description": "The tag as written.",
            "example": "Kids"
          },
          "source": {
            "type": "string",
            "description": "Where it came from.",
            "enum": [
              "admin",
              "sidecar"
            ]
          }
        }
      },
      "description": "Every tag, by key, with where it came from."
    }
  }
}
```

Status `400`: Too many tags, or one too long. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "a work has at most 32 library tags"
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

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator, or an Owner turned off setting patron limits for them. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `404`: No such work, or a podcast episode (tags go on its show).

```json
{
  "error": "work 6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b not found"
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
Failed to deserialize the JSON body into the target type: missing field `tags` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The tags could not be saved.

```json
{
  "error": "the library tags could not be read or saved"
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

## `GET /api/library/collections`

Reads the library's collections.

Only the operator, an Owner or an Administrator may call it; patrons see
collection names through the Library's collection filter, inside their
scope.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The collections, in the order shown. Body: [`LibraryCollections`](schemas.md#librarycollections) (`application/json`)
  - `collections` *array of [`LibraryCollection`](schemas.md#librarycollection)*, **required** — The collections, at most 32, in the order shown. At most 32 items.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The collections could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/collections' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/collections' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/collections",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/collections", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/collections")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The collections, in the order shown.

```json
{
  "collections": [
    {
      "name": "Kids",
      "prefixes": [
        "kids/",
        "children/"
      ]
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "The library's collections: the `core.collections` document and the\nbody of `PUT /api/library/collections`.",
  "required": [
    "collections"
  ],
  "properties": {
    "collections": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One collection: a name and the storage key prefixes whose files belong\nto it.",
        "required": [
          "name",
          "prefixes"
        ],
        "properties": {
          "name": {
            "type": "string",
            "description": "The name people see and scope rules match, 1 to 64 bytes, distinct\nignoring case.",
            "example": "Kids"
          },
          "prefixes": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Storage key prefixes, 1 to 8, each a folder relative to the\ndestination's root (no leading `/`); a closing `/` is added when\nmissing, so `kids` matches `kids/…` but not `kidsbooks/…`.",
            "example": [
              "kids/",
              "children/"
            ],
            "maxItems": 8
          }
        },
        "additionalProperties": false
      },
      "description": "The collections, at most 32, in the order shown.",
      "maxItems": 32
    }
  },
  "additionalProperties": false
}
```

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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
}
```

Status `500`: The collections could not be read.

```json
{
  "error": "the collections could not be read or saved"
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

## `PUT /api/library/collections`

Replaces the library's collections.

Names are trimmed and must differ ignoring case; each prefix gets a
closing `/`. When the collections change, the page index is rebuilt so
the Library filter and patron scopes follow. Only the operator, an
Owner, or an Administrator whom an Owner lets set patron limits may
call it.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`LibraryCollections`](schemas.md#librarycollections)

- `collections` *array of [`LibraryCollection`](schemas.md#librarycollection)*, **required** — The collections, at most 32, in the order shown. At most 32 items.

### Responses

- `200` — The collections after the change. Body: [`LibraryCollections`](schemas.md#librarycollections) (`application/json`)
  - `collections` *array of [`LibraryCollection`](schemas.md#librarycollection)*, **required** — The collections, at most 32, in the order shown. At most 32 items.
- `400` — The collections are invalid. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator, or an Owner turned off setting patron limits for them. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `409` — The collections kept changing underneath the write; retry. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The collections could not be saved. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/collections' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"collections": [{"name": "Kids", "prefixes": ["kids/", "children/"]}]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/collections' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"collections": [{"name": "Kids", "prefixes": ["kids/", "children/"]}]}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/library/collections",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"collections": [{"name": "Kids", "prefixes": ["kids/", "children/"]}]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/collections", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"collections": [{"name": "Kids", "prefixes": ["kids/", "children/"]}]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/library/collections")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"collections": [{"name": "Kids", "prefixes": ["kids/", "children/"]}]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The collections after the change.

```json
{
  "collections": [
    {
      "name": "Kids",
      "prefixes": [
        "kids/",
        "children/"
      ]
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "The library's collections: the `core.collections` document and the\nbody of `PUT /api/library/collections`.",
  "required": [
    "collections"
  ],
  "properties": {
    "collections": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One collection: a name and the storage key prefixes whose files belong\nto it.",
        "required": [
          "name",
          "prefixes"
        ],
        "properties": {
          "name": {
            "type": "string",
            "description": "The name people see and scope rules match, 1 to 64 bytes, distinct\nignoring case.",
            "example": "Kids"
          },
          "prefixes": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Storage key prefixes, 1 to 8, each a folder relative to the\ndestination's root (no leading `/`); a closing `/` is added when\nmissing, so `kids` matches `kids/…` but not `kidsbooks/…`.",
            "example": [
              "kids/",
              "children/"
            ],
            "maxItems": 8
          }
        },
        "additionalProperties": false
      },
      "description": "The collections, at most 32, in the order shown.",
      "maxItems": 32
    }
  },
  "additionalProperties": false
}
```

Status `400`: The collections are invalid. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid collections: two collections are named Kids"
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

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator, or an Owner turned off setting patron limits for them. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
}
```

Status `409`: The collections kept changing underneath the write; retry.

```json
{
  "error": "core.collections kept changing; retry"
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
Failed to deserialize the JSON body into the target type: missing field `collections` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The collections could not be saved.

```json
{
  "error": "the collections could not be read or saved"
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

## `GET /api/config/node-invites`

`GET /api/config/node-invites`: every link for the operator or an
Owner; the caller's own links for anyone else, and with `?mine=true`.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `mine` (query) *boolean* — Only the links for the calling user's own branches (an Owner's "Add a branch" dialog), even when the caller may see everyone's.

### Responses

- `200` — Enrollment links the caller may see. Body: [`NodeInvitesResponse`](schemas.md#nodeinvitesresponse) (`application/json`)
  - `links` *array of [`EnrollmentLinkView`](schemas.md#enrollmentlinkview)*, **required** — The links the caller may see, newest first.
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
$ curl 'http://127.0.0.1:8787/api/config/node-invites' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/node-invites' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/node-invites",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/node-invites", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/node-invites")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Enrollment links the caller may see.

```json
{
  "links": [
    {
      "link_id": "9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b",
      "for_user_id": 7,
      "label": "Living-room NAS",
      "created_by": "user:7",
      "created_at": "2026-10-29T08:15:00Z",
      "expires_at": "2026-10-30T08:15:00Z",
      "redeemed_at": "2026-10-29T09:02:41Z",
      "redeemed_node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
      "revoked_at": "2026-10-29T10:00:00Z",
      "state": "pending"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/config/node-invites`.",
  "required": [
    "links"
  ],
  "properties": {
    "links": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "An enrollment link as the invites routes list it; never its secret.",
        "required": [
          "link_id",
          "for_user_id",
          "label",
          "created_by",
          "created_at",
          "expires_at",
          "state"
        ],
        "properties": {
          "link_id": {
            "type": "string",
            "description": "Public link id.",
            "example": "9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b"
          },
          "for_user_id": {
            "type": "integer",
            "format": "int64",
            "description": "User the enrolled node will belong to.",
            "example": 7
          },
          "label": {
            "type": "string",
            "description": "Label shown in listings.",
            "example": "Living-room NAS"
          },
          "created_by": {
            "type": "string",
            "description": "Audit id of whoever minted it (`operator` or `user:<id>`).",
            "example": "user:7"
          },
          "created_at": {
            "type": "string",
            "description": "RFC 3339 mint time.",
            "example": "2026-10-29T08:15:00Z"
          },
          "expires_at": {
            "type": "string",
            "description": "RFC 3339 expiry.",
            "example": "2026-10-30T08:15:00Z"
          },
          "redeemed_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "RFC 3339 redeem time, once a host used it.",
            "example": "2026-10-29T09:02:41Z"
          },
          "redeemed_node_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Node the redeem created.",
            "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
          },
          "revoked_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "RFC 3339 revoke time.",
            "example": "2026-10-29T10:00:00Z"
          },
          "state": {
            "type": "string",
            "description": "`pending`, `redeemed`, `revoked`, or `expired` right now.",
            "example": "pending"
          }
        }
      },
      "description": "The links the caller may see, newest first."
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

## `POST /api/config/node-invites`

`POST /api/config/node-invites`: mint an enrollment link. Any user may
mint one for themselves while branch enrollment is on; the operator or an
Owner may mint for any active user.

### Arguments

* `caller` - Who is asking; decides whose links they may mint.
* `body` - Owner, lifetime, and label of the new link.

### Returns

`{link, url}`; the secret appears once, in `url`.

### Errors

Returns 503 `node_api_pin_unavailable` in `self_signed` mode while the
certificate's SPKI pin cannot be read (nothing is minted), and an error
response when the library refuses the mint.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PostNodeInvite`](schemas.md#postnodeinvite)

- `user_id` *integer (int64) or null* — User who will own the node. Defaults to the caller; the operator must name one.
- `ttl_secs` *integer (int64) or null* — Link lifetime in seconds (default 24 h, 5 min to 7 days). At least 0.
- `label` *string* — Label shown in listings.

### Responses

- `200` — The new link (shown once). Body: [`NodeInviteCreated`](schemas.md#nodeinvitecreated) (`application/json`)
  - `link` *[`EnrollmentLinkView`](schemas.md#enrollmentlinkview)*, **required** — The link's record.
  - `url` *string*, **required** — The full enrollment URL with the secret, shown once.
- `400` — The operator named no user, the ttl or label is out of range, or the user is disabled. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user may not mint a link for that user, branch enrollment is off, the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `404` — No such user. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The node API's certificate pin is not available yet (`node_api_pin_unavailable`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/node-invites' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/node-invites' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/node-invites",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/node-invites", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/node-invites")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The new link (shown once).

```json
{
  "link": {
    "link_id": "9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b",
    "for_user_id": 7,
    "label": "Living-room NAS",
    "created_by": "user:7",
    "created_at": "2026-10-29T08:15:00Z",
    "expires_at": "2026-10-30T08:15:00Z",
    "redeemed_at": "2026-10-29T09:02:41Z",
    "redeemed_node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "revoked_at": "2026-10-29T10:00:00Z",
    "state": "pending"
  },
  "url": "https://books.example.com/enroll#bcn1.9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b.kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after an enrollment link was minted.",
  "required": [
    "link",
    "url"
  ],
  "properties": {
    "link": {
      "type": "object",
      "description": "The link's record.",
      "required": [
        "link_id",
        "for_user_id",
        "label",
        "created_by",
        "created_at",
        "expires_at",
        "state"
      ],
      "properties": {
        "link_id": {
          "type": "string",
          "description": "Public link id.",
          "example": "9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b"
        },
        "for_user_id": {
          "type": "integer",
          "format": "int64",
          "description": "User the enrolled node will belong to.",
          "example": 7
        },
        "label": {
          "type": "string",
          "description": "Label shown in listings.",
          "example": "Living-room NAS"
        },
        "created_by": {
          "type": "string",
          "description": "Audit id of whoever minted it (`operator` or `user:<id>`).",
          "example": "user:7"
        },
        "created_at": {
          "type": "string",
          "description": "RFC 3339 mint time.",
          "example": "2026-10-29T08:15:00Z"
        },
        "expires_at": {
          "type": "string",
          "description": "RFC 3339 expiry.",
          "example": "2026-10-30T08:15:00Z"
        },
        "redeemed_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "RFC 3339 redeem time, once a host used it.",
          "example": "2026-10-29T09:02:41Z"
        },
        "redeemed_node_id": {
          "type": [
            "string",
            "null"
          ],
          "description": "Node the redeem created.",
          "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
        },
        "revoked_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "RFC 3339 revoke time.",
          "example": "2026-10-29T10:00:00Z"
        },
        "state": {
          "type": "string",
          "description": "`pending`, `redeemed`, `revoked`, or `expired` right now.",
          "example": "pending"
        }
      }
    },
    "url": {
      "type": "string",
      "description": "The full enrollment URL with the secret, shown once.",
      "example": "https://books.example.com/enroll#bcn1.9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b.kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"
    }
  }
}
```

Status `400`: The operator named no user, the ttl or label is out of range, or the user is disabled. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "enrollment link label must be at most 64 bytes without control characters"
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

Status `403`: The user may not mint a link for that user, branch enrollment is off, the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "branch enrollment is disabled for this library"
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

Status `404`: No such user.

```json
{
  "error": "user 7 not found"
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
Failed to deserialize the JSON body into the target type: label: invalid type: integer `1`, expected a string at line 1 column 10
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

Status `503`: The node API's certificate pin is not available yet (`node_api_pin_unavailable`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "node_api_pin_unavailable"
}
```

Schema:

```json
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
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `DELETE /api/config/node-invites/{id}`

`DELETE /api/config/node-invites/{id}`: revoke an unredeemed link.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Link id.

### Responses

- `200` — The link was revoked. Body: [`NodeInviteRevoked`](schemas.md#nodeinviterevoked) (`application/json`)
  - `link` *[`EnrollmentLinkView`](schemas.md#enrollmentlinkview)*, **required** — The link, with `revoked_at` set.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such link, or not one the caller may see. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The link was already redeemed; revoke the node instead. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/node-invites/<id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/node-invites/<id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/config/node-invites/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/node-invites/<id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/config/node-invites/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The link was revoked.

```json
{
  "link": {
    "link_id": "9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b",
    "for_user_id": 7,
    "label": "Living-room NAS",
    "created_by": "user:7",
    "created_at": "2026-10-29T08:15:00Z",
    "expires_at": "2026-10-30T08:15:00Z",
    "redeemed_at": "2026-10-29T09:02:41Z",
    "redeemed_node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "revoked_at": "2026-10-29T10:00:00Z",
    "state": "pending"
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after an enrollment link was revoked.",
  "required": [
    "link"
  ],
  "properties": {
    "link": {
      "type": "object",
      "description": "The link, with `revoked_at` set.",
      "required": [
        "link_id",
        "for_user_id",
        "label",
        "created_by",
        "created_at",
        "expires_at",
        "state"
      ],
      "properties": {
        "link_id": {
          "type": "string",
          "description": "Public link id.",
          "example": "9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b"
        },
        "for_user_id": {
          "type": "integer",
          "format": "int64",
          "description": "User the enrolled node will belong to.",
          "example": 7
        },
        "label": {
          "type": "string",
          "description": "Label shown in listings.",
          "example": "Living-room NAS"
        },
        "created_by": {
          "type": "string",
          "description": "Audit id of whoever minted it (`operator` or `user:<id>`).",
          "example": "user:7"
        },
        "created_at": {
          "type": "string",
          "description": "RFC 3339 mint time.",
          "example": "2026-10-29T08:15:00Z"
        },
        "expires_at": {
          "type": "string",
          "description": "RFC 3339 expiry.",
          "example": "2026-10-30T08:15:00Z"
        },
        "redeemed_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "RFC 3339 redeem time, once a host used it.",
          "example": "2026-10-29T09:02:41Z"
        },
        "redeemed_node_id": {
          "type": [
            "string",
            "null"
          ],
          "description": "Node the redeem created.",
          "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
        },
        "revoked_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "RFC 3339 revoke time.",
          "example": "2026-10-29T10:00:00Z"
        },
        "state": {
          "type": "string",
          "description": "`pending`, `redeemed`, `revoked`, or `expired` right now.",
          "example": "pending"
        }
      }
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

Status `403`: The session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such link, or not one the caller may see.

```json
{
  "error": "enrollment link 5f1e2d3c4b5a69788796a5b4c3d2e1f0 not found"
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

Status `409`: The link was already redeemed; revoke the node instead.

```json
{
  "error": "enrollment link 5f1e2d3c4b5a69788796a5b4c3d2e1f0 was already redeemed; revoke the node instead"
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

## `GET /api/config/branches`

`GET /api/config/branches`: this library's join settings: whether new
branches may join, the transport mode, and the SPKI fingerprint a
joining operator confirms in `self_signed` mode.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The library's join settings and branches. Body: [`BranchesResponse`](schemas.md#branchesresponse) (`application/json`)
  - `library_name` *string*, **required** — This library's name.
  - `origin` *string*, **required** — The canonical origin branches connect to.
  - `transport` *string*, **required** — The node API transport mode (`acme`, `proxy`, `self_signed`, `loopback`).
  - `enrollment` *boolean*, **required** — Whether new branches may join.
  - `spki_fingerprint` *string or null* — In `self_signed` mode, the fingerprint a joining operator confirms.
  - `branches` *array of [`BranchView`](schemas.md#branchview)*, **required** — Every branch, newest first.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — This library's identity row (`library_unavailable`) or origin (`origin_unavailable`) is not available yet. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/branches' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/branches' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/branches",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/branches", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/branches")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The library's join settings and branches.

```json
{
  "library_name": "books.example.com",
  "origin": "https://books.example.com",
  "transport": "self_signed",
  "enrollment": true,
  "spki_fingerprint": "kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o",
  "branches": [
    {
      "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
      "created_at": "2026-10-29T09:02:41Z",
      "owner_user_id": 7,
      "owner_name": "Sam",
      "joined": true,
      "share_books": true,
      "include_branches": true,
      "acquire_members_books": false,
      "titles": {
        "shared": 128,
        "delivered": 120,
        "detached": 3
      }
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/config/branches`.",
  "required": [
    "library_name",
    "origin",
    "transport",
    "enrollment",
    "branches"
  ],
  "properties": {
    "library_name": {
      "type": "string",
      "description": "This library's name.",
      "example": "books.example.com"
    },
    "origin": {
      "type": "string",
      "description": "The canonical origin branches connect to.",
      "example": "https://books.example.com"
    },
    "transport": {
      "type": "string",
      "description": "The node API transport mode (`acme`, `proxy`, `self_signed`, `loopback`).",
      "example": "self_signed"
    },
    "enrollment": {
      "type": "boolean",
      "description": "Whether new branches may join.",
      "example": true
    },
    "spki_fingerprint": {
      "type": [
        "string",
        "null"
      ],
      "description": "In `self_signed` mode, the fingerprint a joining operator confirms.",
      "example": "kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"
    },
    "branches": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One branch (branch node) of this library.",
        "required": [
          "node_id",
          "created_at",
          "joined",
          "acquire_members_books",
          "titles"
        ],
        "properties": {
          "node_id": {
            "type": "string",
            "description": "The branch's node id.",
            "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
          },
          "created_at": {
            "type": "string",
            "description": "RFC 3339 time the node was created.",
            "example": "2026-10-29T09:02:41Z"
          },
          "owner_user_id": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "The user who owns the branch.",
            "example": 7
          },
          "owner_name": {
            "type": [
              "string",
              "null"
            ],
            "description": "That user's display or login name.",
            "example": "Sam"
          },
          "joined": {
            "type": "boolean",
            "description": "Whether its credential is still valid (it has not left or been revoked).",
            "example": true
          },
          "share_books": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "\"Share this branch's books\" as it last reported it; `null` until its first catalog sync.",
            "example": true
          },
          "include_branches": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "\"Include books from my own branches\" as it last reported it; `null` until its first catalog sync.",
            "example": true
          },
          "acquire_members_books": {
            "type": "boolean",
            "description": "This library's \"Acquire all of this branch's members' books\": the\nbranch also sends the books its members acquired that it lists here,\nnot only its owner's.",
            "example": false
          },
          "titles": {
            "type": "object",
            "description": "Counts of the titles it shared.",
            "required": [
              "shared",
              "delivered",
              "detached"
            ],
            "properties": {
              "shared": {
                "type": "integer",
                "format": "int64",
                "description": "Titles it shares now.",
                "example": 128,
                "minimum": 0
              },
              "delivered": {
                "type": "integer",
                "format": "int64",
                "description": "Of those, titles whose files arrived.",
                "example": 120,
                "minimum": 0
              },
              "detached": {
                "type": "integer",
                "format": "int64",
                "description": "Editions whose files it delivered that nothing in this library holds\nany more (it stopped sharing, left, or withdrew them, and no store\naccount or other branch here holds them).",
                "example": 3,
                "minimum": 0
              }
            }
          }
        }
      },
      "description": "Every branch, newest first."
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

Status `403`: The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`).

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

Status `503`: This library's identity row (`library_unavailable`) or origin (`origin_unavailable`) is not available yet. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "library_unavailable"
}
```

Schema:

```json
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
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/config/branches`

`PUT /api/config/branches`: turn "Let new branches join" on or off.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PutBranches`](schemas.md#putbranches)

- `enrollment` *boolean*, **required** — "Let new branches join" (`core.branch_enrollment.enabled`).

### Responses

- `200` — The settings after the change. Body: [`BranchJoinSetting`](schemas.md#branchjoinsetting) (`application/json`)
  - `enrollment` *boolean*, **required** — "Let new branches join", as stored.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
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
$ curl 'http://127.0.0.1:8787/api/config/branches' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"enrollment": true}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/branches' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"enrollment": true}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/branches",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"enrollment": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/branches", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"enrollment": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/branches")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"enrollment": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings after the change.

```json
{
  "enrollment": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after `PUT /api/config/branches`.",
  "required": [
    "enrollment"
  ],
  "properties": {
    "enrollment": {
      "type": "boolean",
      "description": "\"Let new branches join\", as stored.",
      "example": false
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

Status `403`: The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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
Failed to deserialize the JSON body into the target type: missing field `enrollment` at line 1 column 2
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

## `POST /api/config/branches/titles/remove`

`POST /api/config/branches/titles/remove`: removes the entries of
titles from a branch that stopped sharing or left. Delivered files stay.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`RemoveBranchTitles`](schemas.md#removebranchtitles)

- `node_id` *string or null* — Only this branch's entries; every branch's when absent.

### Responses

- `200` — The branch's titles were removed. Body: [`BranchTitlesRemoved`](schemas.md#branchtitlesremoved) (`application/json`)
  - `removed` *integer (int64)*, **required** — How many title entries were removed. At least 0.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
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
$ curl 'http://127.0.0.1:8787/api/config/branches/titles/remove' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/branches/titles/remove' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/branches/titles/remove",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/branches/titles/remove", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/branches/titles/remove")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The branch's titles were removed.

```json
{
  "removed": 3
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after detached branch titles were removed.",
  "required": [
    "removed"
  ],
  "properties": {
    "removed": {
      "type": "integer",
      "format": "int64",
      "description": "How many title entries were removed.",
      "example": 3,
      "minimum": 0
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

Status `403`: The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: node_id: invalid type: integer `1`, expected a string at line 1 column 12
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

## `PUT /api/config/branches/members-books`

`PUT /api/config/branches/members-books`: turn this library's "Acquire
all of this branch's members' books" on or off for one branch.

While on, the branch's catalog sync also sends the books its members
acquired that it lists to this library (its "List my members' books"
setting and each member's opt-out still decide), not only its owner's.
A change restarts the branch's catalog pass, so books it no longer sends
are withdrawn when that pass ends; files already delivered stay.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`MembersBooksBody`](schemas.md#membersbooksbody)

- `node_id` *string*, **required** — The branch's node id, as Settings → Branches lists it.
- `acquire` *boolean*, **required** — "Acquire all of this branch's members' books".

### Responses

- `200` — The setting after the change. Body: [`BranchMembersBooks`](schemas.md#branchmembersbooks) (`application/json`)
  - `node_id` *string*, **required** — The branch.
  - `acquire_members_books` *boolean*, **required** — "Acquire all of this branch's members' books", as stored.
- `400` — The node id is malformed. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `404` — No branch of this library has that node id. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The branch's catalog sync kept changing under the write; retry. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/branches/members-books' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"node_id": "2b7e1c4d-9f3a-4e8b-a6c5-0d1e2f3a4b5c", "acquire": true}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/branches/members-books' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"node_id": "2b7e1c4d-9f3a-4e8b-a6c5-0d1e2f3a4b5c", "acquire": true}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/branches/members-books",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"node_id": "2b7e1c4d-9f3a-4e8b-a6c5-0d1e2f3a4b5c", "acquire": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/branches/members-books", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"node_id": "2b7e1c4d-9f3a-4e8b-a6c5-0d1e2f3a4b5c", "acquire": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/branches/members-books")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"node_id": "2b7e1c4d-9f3a-4e8b-a6c5-0d1e2f3a4b5c", "acquire": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The setting after the change.

```json
{
  "node_id": "2b7e1c4d-9f3a-4e8b-a6c5-0d1e2f3a4b5c",
  "acquire_members_books": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after `PUT /api/config/branches/members-books`.",
  "required": [
    "node_id",
    "acquire_members_books"
  ],
  "properties": {
    "node_id": {
      "type": "string",
      "description": "The branch.",
      "example": "2b7e1c4d-9f3a-4e8b-a6c5-0d1e2f3a4b5c"
    },
    "acquire_members_books": {
      "type": "boolean",
      "description": "\"Acquire all of this branch's members' books\", as stored.",
      "example": true
    }
  }
}
```

Status `400`: The node id is malformed. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid node_id"
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

Status `403`: The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No branch of this library has that node id.

```json
{
  "error": "not_a_branch"
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
Failed to deserialize the JSON body into the target type: missing field `node_id` at line 1 column 2
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

Status `503`: The branch's catalog sync kept changing under the write; retry. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "service unavailable: the branch's catalog fence kept changing"
}
```

Schema:

```json
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
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/join/sharing`

`GET /api/config/join/sharing`: Settings → Library for the operator or
an Owner: every library this install is a branch of, with its two
sharing settings (`{libraries: [...]}`).

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The branch's sharing settings. Body: [`SharingSettings`](schemas.md#sharingsettings) (`application/json`)
  - `libraries` *array of [`LibraryLinkView`](schemas.md#librarylinkview)*, **required** — Every library this install is a branch of, with its sharing and work settings.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/sharing' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/sharing' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/join/sharing",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/sharing", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/join/sharing")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The branch's sharing settings.

```json
{
  "libraries": [
    {
      "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
      "owner_origin": "https://books.example.com",
      "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
      "pinned": false,
      "state": "active",
      "enrolled_at": "2026-10-29T09:02:41Z",
      "share_books": true,
      "include_branches": true,
      "work_for_members": false,
      "keep_member_copies": false,
      "max_jobs": 2,
      "job_share": 2,
      "list_members_books": true,
      "keep_copies": true,
      "running": true
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/config/join/sharing`.",
  "required": [
    "libraries"
  ],
  "properties": {
    "libraries": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A library this install is a branch of: ids, origin and state, never key material.",
        "required": [
          "library_id",
          "owner_origin",
          "node_id",
          "pinned",
          "state",
          "enrolled_at",
          "share_books",
          "include_branches",
          "work_for_members",
          "keep_member_copies",
          "list_members_books",
          "keep_copies"
        ],
        "properties": {
          "library_id": {
            "type": "string",
            "description": "The library id.",
            "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
          },
          "owner_origin": {
            "type": "string",
            "description": "The library's canonical origin.",
            "example": "https://books.example.com"
          },
          "node_id": {
            "type": "string",
            "description": "The node id the library minted for this install.",
            "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
          },
          "pinned": {
            "type": "boolean",
            "description": "Whether the link pins the library's self-signed certificate.",
            "example": false
          },
          "state": {
            "type": "string",
            "description": "`active`, `revoked`, or `left`.",
            "example": "active"
          },
          "enrolled_at": {
            "type": "string",
            "description": "RFC 3339 enrollment time.",
            "example": "2026-10-29T09:02:41Z"
          },
          "share_books": {
            "type": "boolean",
            "description": "\"Share this branch's books with the library\".",
            "example": true
          },
          "include_branches": {
            "type": "boolean",
            "description": "\"Include books from my own branches\".",
            "example": true
          },
          "work_for_members": {
            "type": "boolean",
            "description": "\"Download for other members of the library\": this install takes the library's work for its other members too, not only its owner's (off by default).",
            "example": false
          },
          "keep_member_copies": {
            "type": "boolean",
            "description": "\"Keep copies of other members' books\": a book acquired here for another member is also written to this install's own destinations (off by default).",
            "example": false
          },
          "max_jobs": {
            "type": [
              "integer",
              "null"
            ],
            "description": "The most jobs this install runs for the library at once, as its operator set it; `null` gives the library an equal share of the machine.",
            "example": 2,
            "maximum": 16,
            "minimum": 0
          },
          "job_share": {
            "type": [
              "integer",
              "null"
            ],
            "description": "How many jobs the library may have running here now: its share of the machine after this install's own work, within `max_jobs`. Only the listing and sharing routes report it, and only while the link's client runs.",
            "example": 2,
            "minimum": 0
          },
          "list_members_books": {
            "type": "boolean",
            "description": "\"List my members' books to the library\": the library's catalog\nlookups are answered with members' books (except those who opted\nout) as well as the owner's.",
            "example": true
          },
          "keep_copies": {
            "type": "boolean",
            "description": "\"Keep copies on this computer\": the branch takes the library's\ncopies of its person's books into its own storage. The branch takes\nthe library's work and shows its books whether or not this is on.",
            "example": true
          },
          "running": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "Whether this install's client for the link is running now; only the join routes report it.",
            "example": true
          }
        }
      },
      "description": "Every library this install is a branch of, with its sharing and work settings."
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

Status `403`: The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`).

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

## `POST /api/config/join/sharing`

`POST /api/config/join/sharing`: Settings → Library's sharing and work
settings for one library this install is a branch of (the two
book-sharing settings, the two work settings, and optionally "List my
members' books"). The operator or
an Owner (verified from the stored role); anyone else gets 403.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`SharingBody`](schemas.md#sharingbody)

- `library_id` *string*, **required** — The library (parent library id).
- `share_books` *boolean*, **required** — "Share this branch's books with *library*".
- `include_branches` *boolean*, **required** — "Include books from my own branches".
- `list_members_books` *boolean or null* — "List my members' books to *library*"; absent or `null` keeps the stored setting.
- `work_for_members` *boolean or null* — "Download for other members of *library*": take the library's work for its other members too, not only this install owner's. Absent or `null` keeps the stored setting.
- `keep_member_copies` *boolean or null* — "Keep copies of other members' books": also write a book acquired here for another member to this install's own destinations. Absent or `null` keeps the stored setting.
- `keep_copies` *boolean or null* — "Keep copies on this computer": take the library's copies of this install's person's books into its own storage. Absent or `null` keeps the stored setting.

### Responses

- `200` — The sharing settings after the change. Body: [`SharingChanged`](schemas.md#sharingchanged) (`application/json`)
  - `library` *[`LibraryLinkView`](schemas.md#librarylinkview)*, **required** — The link with its settings as stored now.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `404` — This install is not a branch of that library (`not_found`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
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
$ curl 'http://127.0.0.1:8787/api/config/join/sharing' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "share_books": true, "include_branches": false}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/sharing' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "share_books": true, "include_branches": false}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/sharing",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "share_books": True, "include_branches": False},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/sharing", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "share_books": true, "include_branches": false}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/sharing")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "share_books": true, "include_branches": false}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The sharing settings after the change.

```json
{
  "library": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "owner_origin": "https://books.example.com",
    "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "pinned": false,
    "state": "active",
    "enrolled_at": "2026-10-29T09:02:41Z",
    "share_books": true,
    "include_branches": true,
    "work_for_members": false,
    "keep_member_copies": false,
    "max_jobs": 2,
    "job_share": 2,
    "list_members_books": true,
    "keep_copies": true,
    "running": true
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after `POST /api/config/join/sharing`.",
  "required": [
    "library"
  ],
  "properties": {
    "library": {
      "type": "object",
      "description": "The link with its settings as stored now.",
      "required": [
        "library_id",
        "owner_origin",
        "node_id",
        "pinned",
        "state",
        "enrolled_at",
        "share_books",
        "include_branches",
        "work_for_members",
        "keep_member_copies",
        "list_members_books",
        "keep_copies"
      ],
      "properties": {
        "library_id": {
          "type": "string",
          "description": "The library id.",
          "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
        },
        "owner_origin": {
          "type": "string",
          "description": "The library's canonical origin.",
          "example": "https://books.example.com"
        },
        "node_id": {
          "type": "string",
          "description": "The node id the library minted for this install.",
          "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
        },
        "pinned": {
          "type": "boolean",
          "description": "Whether the link pins the library's self-signed certificate.",
          "example": false
        },
        "state": {
          "type": "string",
          "description": "`active`, `revoked`, or `left`.",
          "example": "active"
        },
        "enrolled_at": {
          "type": "string",
          "description": "RFC 3339 enrollment time.",
          "example": "2026-10-29T09:02:41Z"
        },
        "share_books": {
          "type": "boolean",
          "description": "\"Share this branch's books with the library\".",
          "example": true
        },
        "include_branches": {
          "type": "boolean",
          "description": "\"Include books from my own branches\".",
          "example": true
        },
        "work_for_members": {
          "type": "boolean",
          "description": "\"Download for other members of the library\": this install takes the library's work for its other members too, not only its owner's (off by default).",
          "example": false
        },
        "keep_member_copies": {
          "type": "boolean",
          "description": "\"Keep copies of other members' books\": a book acquired here for another member is also written to this install's own destinations (off by default).",
          "example": false
        },
        "max_jobs": {
          "type": [
            "integer",
            "null"
          ],
          "description": "The most jobs this install runs for the library at once, as its operator set it; `null` gives the library an equal share of the machine.",
          "example": 2,
          "maximum": 16,
          "minimum": 0
        },
        "job_share": {
          "type": [
            "integer",
            "null"
          ],
          "description": "How many jobs the library may have running here now: its share of the machine after this install's own work, within `max_jobs`. Only the listing and sharing routes report it, and only while the link's client runs.",
          "example": 2,
          "minimum": 0
        },
        "list_members_books": {
          "type": "boolean",
          "description": "\"List my members' books to the library\": the library's catalog\nlookups are answered with members' books (except those who opted\nout) as well as the owner's.",
          "example": true
        },
        "keep_copies": {
          "type": "boolean",
          "description": "\"Keep copies on this computer\": the branch takes the library's\ncopies of its person's books into its own storage. The branch takes\nthe library's work and shows its books whether or not this is on.",
          "example": true
        },
        "running": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Whether this install's client for the link is running now; only the join routes report it.",
          "example": true
        }
      }
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

Status `403`: The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: This install is not a branch of that library (`not_found`).

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

## `GET /api/config/join/window`

`GET /api/config/join/window`: "Books in other libraries" for the
operator or an Owner: every book of each library this install is a
branch of, or holds a library card for, merged by work, each edition marked when it
is this install's person's (`{libraries, cards, books}`).

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The libraries this install is a branch of or holds a card for, and their books. Body: [`WindowBooks`](schemas.md#windowbooks) (`application/json`)
  - `libraries` *array of [`LibraryLinkView`](schemas.md#librarylinkview)*, **required** — The libraries this install is an active branch of.
  - `cards` *array of [`ClerkLinkView`](schemas.md#clerklinkview)*, **required** — The libraries this install holds an active library card for.
  - `books` *array of [`WindowEntryView`](schemas.md#windowentryview)*, **required** — Their books, every one they list for a member, one entry per work, by title; each edition says whether it is this install's person's.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/window' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/window' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/join/window",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/window", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/join/window")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The libraries this install is a branch of or holds a card for, and their books.

```json
{
  "libraries": [
    {
      "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
      "owner_origin": "https://books.example.com",
      "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
      "pinned": false,
      "state": "active",
      "enrolled_at": "2026-10-29T09:02:41Z",
      "share_books": true,
      "include_branches": true,
      "work_for_members": false,
      "keep_member_copies": false,
      "max_jobs": 2,
      "job_share": 2,
      "list_members_books": true,
      "keep_copies": true,
      "running": true
    }
  ],
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
  "books": [
    {
      "key": "work:0b8f2c1e-1a2b-4c3d-8e9f-0a1b2c3d4e5f",
      "work_id": "0b8f2c1e-1a2b-4c3d-8e9f-0a1b2c3d4e5f",
      "editions": [
        {
          "library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a",
          "owner_origin": "https://books.example.com",
          "store": "audible",
          "marketplace": "us",
          "product_id": "B0EXAMPLE1",
          "formats": [
            "audio"
          ],
          "title": "A Title",
          "yours": true
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
  "description": "JSON body for `GET /api/config/join/window`.",
  "required": [
    "libraries",
    "cards",
    "books"
  ],
  "properties": {
    "libraries": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A library this install is a branch of: ids, origin and state, never key material.",
        "required": [
          "library_id",
          "owner_origin",
          "node_id",
          "pinned",
          "state",
          "enrolled_at",
          "share_books",
          "include_branches",
          "work_for_members",
          "keep_member_copies",
          "list_members_books",
          "keep_copies"
        ],
        "properties": {
          "library_id": {
            "type": "string",
            "description": "The library id.",
            "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
          },
          "owner_origin": {
            "type": "string",
            "description": "The library's canonical origin.",
            "example": "https://books.example.com"
          },
          "node_id": {
            "type": "string",
            "description": "The node id the library minted for this install.",
            "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
          },
          "pinned": {
            "type": "boolean",
            "description": "Whether the link pins the library's self-signed certificate.",
            "example": false
          },
          "state": {
            "type": "string",
            "description": "`active`, `revoked`, or `left`.",
            "example": "active"
          },
          "enrolled_at": {
            "type": "string",
            "description": "RFC 3339 enrollment time.",
            "example": "2026-10-29T09:02:41Z"
          },
          "share_books": {
            "type": "boolean",
            "description": "\"Share this branch's books with the library\".",
            "example": true
          },
          "include_branches": {
            "type": "boolean",
            "description": "\"Include books from my own branches\".",
            "example": true
          },
          "work_for_members": {
            "type": "boolean",
            "description": "\"Download for other members of the library\": this install takes the library's work for its other members too, not only its owner's (off by default).",
            "example": false
          },
          "keep_member_copies": {
            "type": "boolean",
            "description": "\"Keep copies of other members' books\": a book acquired here for another member is also written to this install's own destinations (off by default).",
            "example": false
          },
          "max_jobs": {
            "type": [
              "integer",
              "null"
            ],
            "description": "The most jobs this install runs for the library at once, as its operator set it; `null` gives the library an equal share of the machine.",
            "example": 2,
            "maximum": 16,
            "minimum": 0
          },
          "job_share": {
            "type": [
              "integer",
              "null"
            ],
            "description": "How many jobs the library may have running here now: its share of the machine after this install's own work, within `max_jobs`. Only the listing and sharing routes report it, and only while the link's client runs.",
            "example": 2,
            "minimum": 0
          },
          "list_members_books": {
            "type": "boolean",
            "description": "\"List my members' books to the library\": the library's catalog\nlookups are answered with members' books (except those who opted\nout) as well as the owner's.",
            "example": true
          },
          "keep_copies": {
            "type": "boolean",
            "description": "\"Keep copies on this computer\": the branch takes the library's\ncopies of its person's books into its own storage. The branch takes\nthe library's work and shows its books whether or not this is on.",
            "example": true
          },
          "running": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "Whether this install's client for the link is running now; only the join routes report it.",
            "example": true
          }
        }
      },
      "description": "The libraries this install is an active branch of."
    },
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
      "description": "The libraries this install holds an active library card for."
    },
    "books": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One work in [`WindowBooks`]: every edition its libraries hold for the person.",
        "required": [
          "key",
          "editions"
        ],
        "properties": {
          "key": {
            "type": "string",
            "description": "A key that stays the same while the entry's editions do.",
            "example": "work:0b8f2c1e-1a2b-4c3d-8e9f-0a1b2c3d4e5f"
          },
          "work_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "The work of this install's own library the entry was placed on, by\nidentifier; absent when none was.",
            "example": "0b8f2c1e-1a2b-4c3d-8e9f-0a1b2c3d4e5f"
          },
          "editions": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One edition in [`WindowBooks`], and the library that holds it.",
              "required": [
                "library_id",
                "owner_origin",
                "store",
                "marketplace",
                "product_id",
                "formats",
                "title",
                "yours"
              ],
              "properties": {
                "library_id": {
                  "type": "string",
                  "description": "The library (parent library id) that holds it.",
                  "example": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"
                },
                "owner_origin": {
                  "type": "string",
                  "description": "The library's origin.",
                  "example": "https://books.example.com"
                },
                "store": {
                  "type": "string",
                  "description": "The store's plain id.",
                  "example": "audible"
                },
                "marketplace": {
                  "type": "string",
                  "description": "Lower-cased store marketplace; empty when the store has none.",
                  "example": "us"
                },
                "product_id": {
                  "type": "string",
                  "description": "The store's product id.",
                  "example": "B0EXAMPLE1"
                },
                "formats": {
                  "type": "array",
                  "items": {
                    "type": "string"
                  },
                  "description": "Formats that library has (`audio`, `pdf`); empty when unknown.",
                  "example": [
                    "audio"
                  ]
                },
                "title": {
                  "type": "string",
                  "description": "The edition's title.",
                  "example": "A Title"
                },
                "subtitle": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Subtitle, when known.",
                  "example": "A Novel"
                },
                "authors": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Authors as one display string, when known.",
                  "example": "Ann Author"
                },
                "narrators": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Narrators as one display string, when known.",
                  "example": "Ned Narrator"
                },
                "series": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Series name, when known.",
                  "example": "The Series"
                },
                "series_index": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Position in the series, when known.",
                  "example": "2"
                },
                "length_minutes": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Length in minutes, when known.",
                  "example": 600
                },
                "yours": {
                  "type": "boolean",
                  "description": "True when this install's person holds it there through a store\naccount of their own; false for a book only other people there\nhold.",
                  "example": true
                },
                "request": {
                  "oneOf": [
                    {
                      "type": "object",
                      "description": "How to ask the library for a copy of this edition\n(`POST /api/config/clerk/request`): set when this install holds a\ncard for the library and the library holds no copy, while one of\nits branches lists it (`from` is `branch`) or one of its own store\naccounts holds it (`from` is `library`). Absent otherwise.",
                      "required": [
                        "work_id",
                        "from"
                      ],
                      "properties": {
                        "work_id": {
                          "type": "string",
                          "description": "The library's own work id to ask for.",
                          "example": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"
                        },
                        "from": {
                          "type": "string",
                          "description": "Where the copy would come from.",
                          "enum": [
                            "branch",
                            "library"
                          ]
                        }
                      }
                    },
                    {
                      "type": "null"
                    }
                  ]
                }
              }
            },
            "description": "The editions, in library then store order; the first names the entry."
          }
        }
      },
      "description": "Their books, every one they list for a member, one entry per work,\nby title; each edition says whether it is this install's person's."
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

Status `403`: The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`).

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

## `POST /api/config/join/connections`

`POST /api/config/join/connections`: "Add my connections to *library*".

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`AddConnectionsBody`](schemas.md#addconnectionsbody)

- `library_id` *string*, **required** — The library (parent library id) to add the connections to.
- `accounts` *array of [`AccountRef`](schemas.md#accountref)*, **required** — The person's accounts to add; 1 to 64.

### Responses

- `200` — What the library kept. Body: [`ConnectionsAdded`](schemas.md#connectionsadded) (`application/json`)
  - `added` *integer (int64)*, **required** — Accounts now connected in the library for the person. At least 0.
  - `already_here` *integer (int64)*, **required** — Accounts the person had already connected there. At least 0.
  - `taken` *integer (int64)*, **required** — Accounts connected to someone else there; left as they are. At least 0.
  - `failed` *integer (int64)*, **required** — Accounts that could not be kept there. At least 0.
  - `not_sent` *integer (int64)*, **required** — Accounts not sent: not the person's, not connected, or of a storefront whose sign-ins can't be shared. At least 0.
  - `not_allowed` *boolean*, **required** — The library's operator does not let the person connect store accounts there; nothing was kept.
- `400` — No accounts, or more than 64 (`invalid_request`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is not the person this branch belongs to (`not_branch_person`). A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorDetail`](schemas.md#errordetail) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — This install is not a branch of that library (`not_found`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `409` — The library refused the offer (`refused`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The library could not be reached (`unavailable`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorDetail`](schemas.md#errordetail) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/connections' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "accounts": [{"plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "account_id": "reader@example.com"}]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/connections' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "accounts": [{"plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "account_id": "reader@example.com"}]}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/connections",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "accounts": [{"plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "account_id": "reader@example.com"}]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/connections", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "accounts": [{"plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "account_id": "reader@example.com"}]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/connections")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "accounts": [{"plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "account_id": "reader@example.com"}]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: What the library kept.

```json
{
  "added": 1,
  "already_here": 0,
  "taken": 0,
  "failed": 0,
  "not_sent": 0,
  "not_allowed": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after `POST /api/config/join/connections`.",
  "required": [
    "added",
    "already_here",
    "taken",
    "failed",
    "not_sent",
    "not_allowed"
  ],
  "properties": {
    "added": {
      "type": "integer",
      "format": "int64",
      "description": "Accounts now connected in the library for the person.",
      "example": 1,
      "minimum": 0
    },
    "already_here": {
      "type": "integer",
      "format": "int64",
      "description": "Accounts the person had already connected there.",
      "example": 0,
      "minimum": 0
    },
    "taken": {
      "type": "integer",
      "format": "int64",
      "description": "Accounts connected to someone else there; left as they are.",
      "example": 0,
      "minimum": 0
    },
    "failed": {
      "type": "integer",
      "format": "int64",
      "description": "Accounts that could not be kept there.",
      "example": 0,
      "minimum": 0
    },
    "not_sent": {
      "type": "integer",
      "format": "int64",
      "description": "Accounts not sent: not the person's, not connected, or of a\nstorefront whose sign-ins can't be shared.",
      "example": 0,
      "minimum": 0
    },
    "not_allowed": {
      "type": "boolean",
      "description": "The library's operator does not let the person connect store\naccounts there; nothing was kept.",
      "example": false
    }
  }
}
```

Status `400`: No accounts, or more than 64 (`invalid_request`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "name between 1 and 64 accounts",
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

Status `403`: The caller is not the person this branch belongs to (`not_branch_person`). A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: This install is not a branch of that library (`not_found`).

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

Status `409`: The library refused the offer (`refused`).

```json
{
  "detail": "no_owner",
  "error": "refused"
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

Status `503`: The library could not be reached (`unavailable`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

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

## `POST /api/config/join/connections/disconnect`

`POST /api/config/join/connections/disconnect`: disconnects one of the
person's store accounts here, and with `everywhere` in every linked
library it was shared with.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`DisconnectBody`](schemas.md#disconnectbody)

- `plugin_key` *string*, **required** — Provenance-qualified plugin key of the storefront.
- `account_id` *string*, **required** — Store account id.
- `everywhere` *boolean* — Also disconnect it in every linked library it was shared with; absent means `false`.

### Responses

- `200` — Where the account is disconnected now. Body: [`Disconnected`](schemas.md#disconnected) (`application/json`)
  - `here` *boolean*, **required** — The account was the person's here and is disconnected here now.
  - `libraries` *array of string*, **required** — Linked libraries (parent library ids) that confirmed the disconnect.
  - `unreached` *array of string*, **required** — Linked libraries that could not be reached or refused; the account stays connected there until the person disconnects it there.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller is not the person this branch belongs to (`not_branch_person`). A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorDetail`](schemas.md#errordetail) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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
$ curl 'http://127.0.0.1:8787/api/config/join/connections/disconnect' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "account_id": "reader@example.com"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/connections/disconnect' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "account_id": "reader@example.com"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/connections/disconnect",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "account_id": "reader@example.com"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/connections/disconnect", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "account_id": "reader@example.com"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/connections/disconnect")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro", "account_id": "reader@example.com"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Where the account is disconnected now.

```json
{
  "here": true,
  "libraries": [
    "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"
  ],
  "unreached": []
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after `POST /api/config/join/connections/disconnect`.",
  "required": [
    "here",
    "libraries",
    "unreached"
  ],
  "properties": {
    "here": {
      "type": "boolean",
      "description": "The account was the person's here and is disconnected here now.",
      "example": true
    },
    "libraries": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Linked libraries (parent library ids) that confirmed the disconnect.",
      "example": [
        "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"
      ]
    },
    "unreached": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Linked libraries that could not be reached or refused; the account\nstays connected there until the person disconnects it there.",
      "example": []
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

Status `403`: The caller is not the person this branch belongs to (`not_branch_person`). A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `plugin_key` at line 1 column 2
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

## `POST /api/config/join/capacity`

`POST /api/config/join/capacity`: Settings → Library's job cap for one
library this install is a branch of. The operator or an Owner (verified
from the stored role); anyone else gets 403. The running link's share
changes at once: its claims stop at the new share and its next
heartbeat reports the library the new part of the machine. Jobs it
already runs finish.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`CapacityBody`](schemas.md#capacitybody)

- `library_id` *string*, **required** — The library (parent library id).
- `max_jobs` *integer (int32) or null*, **required** — The most jobs to run for the library at once (0 runs none); `null` gives it an equal share of the machine. Required: a body without it is refused rather than read as `null`. At least 0; at most 16.

### Responses

- `200` — The link after the change, with its share of the machine now. Body: [`SharingChanged`](schemas.md#sharingchanged) (`application/json`)
  - `library` *[`LibraryLinkView`](schemas.md#librarylinkview)*, **required** — The link with its settings as stored now.
- `400` — `max_jobs` is missing or above 16 jobs (`invalid`). As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `404` — This install is not a branch of that library (`not_found`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
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
$ curl 'http://127.0.0.1:8787/api/config/join/capacity' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "max_jobs": 1}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/capacity' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "max_jobs": 1}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/capacity",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "max_jobs": 1},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/capacity", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "max_jobs": 1}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/capacity")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"library_id": "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a", "max_jobs": 1}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The link after the change, with its share of the machine now.

```json
{
  "library": {
    "library_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "owner_origin": "https://books.example.com",
    "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "pinned": false,
    "state": "active",
    "enrolled_at": "2026-10-29T09:02:41Z",
    "share_books": true,
    "include_branches": true,
    "work_for_members": false,
    "keep_member_copies": false,
    "max_jobs": 2,
    "job_share": 2,
    "list_members_books": true,
    "keep_copies": true,
    "running": true
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after `POST /api/config/join/sharing`.",
  "required": [
    "library"
  ],
  "properties": {
    "library": {
      "type": "object",
      "description": "The link with its settings as stored now.",
      "required": [
        "library_id",
        "owner_origin",
        "node_id",
        "pinned",
        "state",
        "enrolled_at",
        "share_books",
        "include_branches",
        "work_for_members",
        "keep_member_copies",
        "list_members_books",
        "keep_copies"
      ],
      "properties": {
        "library_id": {
          "type": "string",
          "description": "The library id.",
          "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
        },
        "owner_origin": {
          "type": "string",
          "description": "The library's canonical origin.",
          "example": "https://books.example.com"
        },
        "node_id": {
          "type": "string",
          "description": "The node id the library minted for this install.",
          "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
        },
        "pinned": {
          "type": "boolean",
          "description": "Whether the link pins the library's self-signed certificate.",
          "example": false
        },
        "state": {
          "type": "string",
          "description": "`active`, `revoked`, or `left`.",
          "example": "active"
        },
        "enrolled_at": {
          "type": "string",
          "description": "RFC 3339 enrollment time.",
          "example": "2026-10-29T09:02:41Z"
        },
        "share_books": {
          "type": "boolean",
          "description": "\"Share this branch's books with the library\".",
          "example": true
        },
        "include_branches": {
          "type": "boolean",
          "description": "\"Include books from my own branches\".",
          "example": true
        },
        "work_for_members": {
          "type": "boolean",
          "description": "\"Download for other members of the library\": this install takes the library's work for its other members too, not only its owner's (off by default).",
          "example": false
        },
        "keep_member_copies": {
          "type": "boolean",
          "description": "\"Keep copies of other members' books\": a book acquired here for another member is also written to this install's own destinations (off by default).",
          "example": false
        },
        "max_jobs": {
          "type": [
            "integer",
            "null"
          ],
          "description": "The most jobs this install runs for the library at once, as its operator set it; `null` gives the library an equal share of the machine.",
          "example": 2,
          "maximum": 16,
          "minimum": 0
        },
        "job_share": {
          "type": [
            "integer",
            "null"
          ],
          "description": "How many jobs the library may have running here now: its share of the machine after this install's own work, within `max_jobs`. Only the listing and sharing routes report it, and only while the link's client runs.",
          "example": 2,
          "minimum": 0
        },
        "list_members_books": {
          "type": "boolean",
          "description": "\"List my members' books to the library\": the library's catalog\nlookups are answered with members' books (except those who opted\nout) as well as the owner's.",
          "example": true
        },
        "keep_copies": {
          "type": "boolean",
          "description": "\"Keep copies on this computer\": the branch takes the library's\ncopies of its person's books into its own storage. The branch takes\nthe library's work and shows its books whether or not this is on.",
          "example": true
        },
        "running": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Whether this install's client for the link is running now; only the join routes report it.",
          "example": true
        }
      }
    }
  }
}
```

Status `400`: `max_jobs` is missing or above 16 jobs (`invalid`). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "detail": "a library's job cap is 0 to 16 jobs",
  "error": "invalid"
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

Status `403`: The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: This install is not a branch of that library (`not_found`).

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

## `DELETE /api/config/node-credentials/{node_id}`

`DELETE /api/config/node-credentials/{node_id}`: revoke an enrolled
host's credential so it can no longer connect.

### Arguments

* `caller` - Who is asking; the operator or an Owner may revoke any host,
  any other active user only the hosts they own.
* `node_id` - The branch node to revoke.

### Returns

`{credential}` with `revoked_at` set; revoking twice keeps the first
time.

### Errors

Returns 404 when the node is unknown or not the caller's, 400 when the
node has no host credential, 403 for a disabled user, and an error
response when the write fails.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `node_id` (path) *string*, **required** — Node id.

### Responses

- `200` — The host's credential was revoked. Body: [`NodeCredentialRevoked`](schemas.md#nodecredentialrevoked) (`application/json`)
  - `credential` *[`NodeCredentialView`](schemas.md#nodecredentialview)*, **required** — The credential, with `revoked_at` set (revoking twice keeps the first time).
- `400` — The host has no credential to revoke. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such host, or not one the caller may see. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/node-credentials/<node_id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/node-credentials/<node_id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/config/node-credentials/<node_id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/node-credentials/<node_id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/config/node-credentials/<node_id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The host's credential was revoked.

```json
{
  "credential": {
    "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "link_id": "9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b",
    "enrolled_at": "2026-10-29T09:02:41Z",
    "revoked_at": "2026-11-02T18:30:00Z",
    "last_connected_at": "2026-11-02T18:29:12Z"
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after a host's credential was revoked.",
  "required": [
    "credential"
  ],
  "properties": {
    "credential": {
      "type": "object",
      "description": "The credential, with `revoked_at` set (revoking twice keeps the first time).",
      "required": [
        "node_id",
        "link_id",
        "enrolled_at"
      ],
      "properties": {
        "node_id": {
          "type": "string",
          "description": "Branch node.",
          "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
        },
        "link_id": {
          "type": "string",
          "description": "Link that enrolled it.",
          "example": "9f2c7b1e4a8d3c6f0b5e2a7d4c1f8e3b"
        },
        "enrolled_at": {
          "type": "string",
          "description": "RFC 3339 enrollment time.",
          "example": "2026-10-29T09:02:41Z"
        },
        "revoked_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "RFC 3339 revoke time.",
          "example": "2026-11-02T18:30:00Z"
        },
        "last_connected_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "RFC 3339 time of the last authenticated connection.",
          "example": "2026-11-02T18:29:12Z"
        }
      }
    }
  }
}
```

Status `400`: The host has no credential to revoke.

```json
{
  "error": "node 7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c has no host credential to revoke"
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

Status `403`: The session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such host, or not one the caller may see.

```json
{
  "error": "node 7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c not found"
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
- `search` (query) *one of `words`, `meaning`* — How `q` is matched (default `words`). `meaning` needs the library's embedding model on the node that answers and the library embedded with it (`GET /api/library/works/meaning` says whether it is ready); `q` is then plain text, not a Lucene query.
- `sort` (query) *one of `title`, `author`, `added`, `series`, `purchased`, `length`* — Order of the page (default `title`).
- `media_kind` (query) *one of `audio`, `ebook`, `print`* — Only works with an edition on this carrier.
- `kind` (query) *one of `book`, `podcast`* — Only works of this kind.
- `status` (query) *one of `not_acquired`, `queued`, `downloading`, `acquired`, `error`* — Only works whose furthest acquire state is this one.
- `account` (query) *string* — Only works held by this store account id.
- `facet` (query) *one of `author`, `narrator`, `series`, `genre`, `publisher`, `source`, `tag`, `library_tag`, `collection`, `progress`* — Only works with `value` under this facet. Values match ignoring case; `tag` and `progress` read the caller's own state.
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
- `503` — A search (`q`) arrived before the node's works search index was first built; that answer carries no `Retry-After`. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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
      "purchased_at": "2024-03-02T18:04:11Z",
      "acquirable_editions": 1
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
          "added_at",
          "acquirable_editions"
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
          },
          "acquirable_editions": {
            "type": "integer",
            "format": "int64",
            "description": "How many held editions an acquire would still queue: neither\nacquired, queued nor downloading. The library offers Acquire while\nit is above 0, even when `acquire_status` is `acquired` (an acquired\naudio edition beside a held ebook, or a second narration).",
            "example": 1,
            "minimum": 0
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

Status `503`: A search (`q`) arrived before the node's works search index was first built; that answer carries no `Retry-After`. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "the search index is being built; try again shortly"
}
```

Schema:

```json
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
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/library/works/meaning`

Says whether search by meaning and similar works are ready.

A client asks before it offers either. `search` is about the node
that answers: it needs the library's embedding model there.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Responses

- `200` — What is ready. Body: [`MeaningStatus`](schemas.md#meaningstatus) (`application/json`)
  - `search` *[`MeaningState`](schemas.md#meaningstate)*, **required** — Whether `search=meaning` answers on this node.
  - `similar` *[`MeaningState`](schemas.md#meaningstate)*, **required** — Whether `GET /api/library/works/{id}/similar` lists works.
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
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/works/meaning' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/works/meaning' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/works/meaning",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/works/meaning", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/works/meaning")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: What is ready.

```json
{
  "search": "ready",
  "similar": "ready"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of `GET /api/library/works/meaning`.",
  "required": [
    "search",
    "similar"
  ],
  "properties": {
    "search": {
      "type": "string",
      "description": "Whether `search=meaning` answers on this node.",
      "enum": [
        "ready",
        "no_model",
        "embedding"
      ]
    },
    "similar": {
      "type": "string",
      "description": "Whether `GET /api/library/works/{id}/similar` lists works.",
      "enum": [
        "ready",
        "no_model",
        "embedding"
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

## `GET /api/library/works/{id}/similar`

Lists the held works most like one work.

Likeness is how close the works' descriptions are in meaning, as the
library's embedding model reads them. The caller sees only works
inside their own scope.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Work id (a podcast's show).
- `limit` (query) *integer* — Most works to list, clamped to 1–40 (default 12).

### Responses

- `200` — The similar works, or why there are none. Body: [`SimilarWorks`](schemas.md#similarworks) (`application/json`)
  - `state` *[`MeaningState`](schemas.md#meaningstate)*, **required** — `ready` when `works` was looked up; otherwise why it is empty.
  - `works` *array of [`WorkSummaryJson`](schemas.md#worksummaryjson)*, **required** — The held works most like the one asked about, most alike first. Empty when the work has no vector yet.
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
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/works/<id>/similar' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/works/<id>/similar' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/works/<id>/similar",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/works/<id>/similar", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/works/<id>/similar")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The similar works, or why there are none.

```json
{
  "state": "ready",
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
      "purchased_at": "2024-03-02T18:04:11Z",
      "acquirable_editions": 1
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of `GET /api/library/works/{id}/similar`.",
  "required": [
    "state",
    "works"
  ],
  "properties": {
    "state": {
      "type": "string",
      "description": "`ready` when `works` was looked up; otherwise why it is empty.",
      "enum": [
        "ready",
        "no_model",
        "embedding"
      ]
    },
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
          "added_at",
          "acquirable_editions"
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
          },
          "acquirable_editions": {
            "type": "integer",
            "format": "int64",
            "description": "How many held editions an acquire would still queue: neither\nacquired, queued nor downloading. The library offers Acquire while\nit is above 0, even when `acquire_status` is `acquired` (an acquired\naudio edition beside a held ebook, or a second narration).",
            "example": 1,
            "minimum": 0
          }
        }
      },
      "description": "The held works most like the one asked about, most alike first.\nEmpty when the work has no vector yet."
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
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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
                          },
                          {
                            "type": "object",
                            "description": "A patron who uploaded the book.",
                            "required": [
                              "user_id",
                              "kind"
                            ],
                            "properties": {
                              "user_id": {
                                "type": "integer",
                                "format": "int64",
                                "description": "The person's user id.",
                                "example": 3
                              },
                              "kind": {
                                "type": "string",
                                "enum": [
                                  "patron"
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
                          "free",
                          "own"
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
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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

## `GET /api/library/works/facets/{facet}`

Lists the values a facet has across the library's held works, with
how many works have each, for the library page's filter menus. `tag`
lists the caller's own tags (none for an operator token).

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `facet` (path) *one of `author`, `narrator`, `series`, `genre`, `publisher`, `source`, `tag`, `library_tag`, `collection`, `progress`*, **required** — The facet; `progress` has no value list.
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
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
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

## `GET /api/library/editions/{id}/playback`

Reads whether and what the caller may play of an edition, with their
own progress and bookmarks.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Edition id (an `id` in a work's `editions`).

### Responses

- `200` — What the caller may play, and where they are. Body: [`PlaybackManifestJson`](schemas.md#playbackmanifestjson) (`application/json`)
  - `edition_id` *string*, **required** — The edition.
  - `work_id` *string or null* — Its work.
  - `title` *string*, **required** — The edition's display title.
  - `playable` *boolean*, **required** — True when this node holds a copy the caller may play. False when the library has not acquired the edition, no destination of this node holds it, or the caller may not take it.
  - `duration_ms` *integer (int64) or null* — Runtime in milliseconds: the last player's report when the caller has one, else the catalog's, to the minute; null when neither is known. At least 0.
  - `tracks` *array of [`TrackJson`](schemas.md#trackjson)*, **required** — The tracks in play order; empty when not playable. A session's answer adds the stream paths.
  - `progress` *[`ProgressJson`](schemas.md#progressjson) or null* — The caller's progress; null before they first play it.
  - `bookmarks` *array of [`BookmarkJson`](schemas.md#bookmarkjson)*, **required** — The caller's bookmarks, in timeline order.
  - `chapters` *array of [`ChapterSpan`](schemas.md#chapterspan)*, **required** — The chapters the edition plays, in pre-order, each ending where the next starts and the last at `duration_ms`; empty when it has none.
- `401` — The session ended. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/playback' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/playback' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/editions/<id>/playback",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/playback", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/editions/<id>/playback")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: What the caller may play, and where they are.

```json
{
  "edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d",
  "work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b",
  "title": "The Left Hand of Darkness",
  "playable": true,
  "duration_ms": 41400000,
  "tracks": [
    {
      "index": 0,
      "path": "/api/playback/sessions/7c9e6679-7425-40de-944b-e07fc1f90ae7/tracks/0",
      "content_type": "audio/mp4",
      "bitrate_kbps": 125,
      "lossless": false
    }
  ],
  "progress": {
    "edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d",
    "work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b",
    "position_ms": 754000,
    "duration_ms": 41400000,
    "is_finished": false,
    "started_at": "2026-11-19T08:15:00+00:00",
    "finished_at": null,
    "updated_at": "2026-11-19T09:02:41+00:00"
  },
  "bookmarks": [
    {
      "id": "9a1b2c3d-4e5f-4a6b-8c7d-0e1f2a3b4c5d",
      "edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d",
      "position_ms": 754000,
      "title": "The storm",
      "created_at": "2026-11-19T09:02:41+00:00"
    }
  ],
  "chapters": [
    {
      "title": "Chapter 1: The Boy Who Lived",
      "start_ms": 33000,
      "end_ms": 1215000,
      "depth": 0
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/library/editions/{id}/playback` answer.",
  "required": [
    "edition_id",
    "title",
    "playable",
    "tracks",
    "bookmarks",
    "chapters"
  ],
  "properties": {
    "edition_id": {
      "type": "string",
      "description": "The edition.",
      "example": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d"
    },
    "work_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Its work.",
      "example": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"
    },
    "title": {
      "type": "string",
      "description": "The edition's display title.",
      "example": "The Left Hand of Darkness"
    },
    "playable": {
      "type": "boolean",
      "description": "True when this node holds a copy the caller may play. False when\nthe library has not acquired the edition, no destination of this\nnode holds it, or the caller may not take it.",
      "example": true
    },
    "duration_ms": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Runtime in milliseconds: the last player's report when the caller\nhas one, else the catalog's, to the minute; null when neither is\nknown.",
      "example": 41400000,
      "minimum": 0
    },
    "tracks": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One audio file of an edition, as a player sees it.",
        "required": [
          "index",
          "content_type",
          "lossless"
        ],
        "properties": {
          "index": {
            "type": "integer",
            "description": "The track's place in play order, from 0.",
            "example": 0,
            "minimum": 0
          },
          "path": {
            "type": [
              "string",
              "null"
            ],
            "description": "Path to stream it from, on this origin; only in a session's answer.",
            "example": "/api/playback/sessions/7c9e6679-7425-40de-944b-e07fc1f90ae7/tracks/0"
          },
          "content_type": {
            "type": "string",
            "description": "Media type the stream states, such as `audio/mp4`.",
            "example": "audio/mp4"
          },
          "bitrate_kbps": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int32",
            "description": "Average bitrate in kilobits per second, once probed.",
            "example": 125,
            "minimum": 0
          },
          "lossless": {
            "type": "boolean",
            "description": "True for a lossless encoding.",
            "example": false
          }
        }
      },
      "description": "The tracks in play order; empty when not playable. A session's\nanswer adds the stream paths."
    },
    "progress": {
      "oneOf": [
        {
          "type": "object",
          "description": "The caller's progress; null before they first play it.",
          "required": [
            "edition_id",
            "position_ms",
            "is_finished",
            "started_at",
            "updated_at"
          ],
          "properties": {
            "edition_id": {
              "type": "string",
              "description": "The edition.",
              "example": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d"
            },
            "work_id": {
              "type": [
                "string",
                "null"
              ],
              "description": "The edition's work, when known: what a Continue listening shelf\nreads the title and cover from.",
              "example": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"
            },
            "position_ms": {
              "type": "integer",
              "format": "int64",
              "description": "Position on the edition's timeline, in milliseconds.",
              "example": 754000,
              "minimum": 0
            },
            "duration_ms": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "The edition's runtime in milliseconds as the last player reported\nit; null until one has.",
              "example": 41400000,
              "minimum": 0
            },
            "is_finished": {
              "type": "boolean",
              "description": "True once the caller reached the end of this edition.",
              "example": false
            },
            "started_at": {
              "type": "string",
              "description": "When the caller first played it (RFC 3339).",
              "example": "2026-11-19T08:15:00+00:00"
            },
            "finished_at": {
              "type": [
                "string",
                "null"
              ],
              "description": "When the caller last reached the end (RFC 3339).",
              "example": null
            },
            "updated_at": {
              "type": "string",
              "description": "When the progress last changed (RFC 3339).",
              "example": "2026-11-19T09:02:41+00:00"
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "bookmarks": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A place the caller marked in an edition.",
        "required": [
          "id",
          "edition_id",
          "position_ms",
          "title",
          "created_at"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Bookmark id.",
            "example": "9a1b2c3d-4e5f-4a6b-8c7d-0e1f2a3b4c5d"
          },
          "edition_id": {
            "type": "string",
            "description": "The edition it is in.",
            "example": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d"
          },
          "position_ms": {
            "type": "integer",
            "format": "int64",
            "description": "The marked place, in milliseconds from the edition's start.",
            "example": 754000,
            "minimum": 0
          },
          "title": {
            "type": "string",
            "description": "The caller's note; may be empty.",
            "example": "The storm"
          },
          "created_at": {
            "type": "string",
            "description": "When it was made (RFC 3339).",
            "example": "2026-11-19T09:02:41+00:00"
          }
        }
      },
      "description": "The caller's bookmarks, in timeline order."
    },
    "chapters": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One chapter as a player seeks between them: where it starts and ends\non the edition's timeline ([`ChapterList::spans`]).",
        "required": [
          "title",
          "start_ms",
          "end_ms",
          "depth"
        ],
        "properties": {
          "title": {
            "type": "string",
            "description": "Chapter title.",
            "example": "Chapter 1: The Boy Who Lived"
          },
          "start_ms": {
            "type": "integer",
            "format": "int64",
            "description": "Start, in milliseconds from the start of the edition's timeline.",
            "example": 33000,
            "minimum": 0
          },
          "end_ms": {
            "type": "integer",
            "format": "int64",
            "description": "End, in milliseconds: the next chapter's start, or the end of the\ntimeline for the last chapter; never before `start_ms`.",
            "example": 1215000,
            "minimum": 0
          },
          "depth": {
            "type": "integer",
            "format": "int32",
            "description": "Nesting depth; 0 for a top-level chapter. A chapter at depth n + 1\nis nested under the nearest earlier chapter at depth n.",
            "example": 0,
            "minimum": 0
          }
        }
      },
      "description": "The chapters the edition plays, in pre-order, each ending where the\nnext starts and the last at `duration_ms`; empty when it has none."
    }
  }
}
```

Status `401`: The session ended.

```json
{
  "error": "sign in again"
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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

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

Status `404`: No such edition.

```json
{
  "error": "edition 5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d not found"
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

## `GET /api/library/editions/{id}/bookmarks`

Lists the caller's bookmarks in an edition.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Edition id.

### Responses

- `200` — The caller's bookmarks, in timeline order. Body: [`BookmarksJson`](schemas.md#bookmarksjson) (`application/json`)
  - `bookmarks` *array of [`BookmarkJson`](schemas.md#bookmarkjson)*, **required** — The caller's bookmarks in the edition, in timeline order.
- `401` — The session ended. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/bookmarks' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/bookmarks' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/editions/<id>/bookmarks",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/bookmarks", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/editions/<id>/bookmarks")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's bookmarks, in timeline order.

```json
{
  "bookmarks": [
    {
      "id": "9a1b2c3d-4e5f-4a6b-8c7d-0e1f2a3b4c5d",
      "edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d",
      "position_ms": 754000,
      "title": "The storm",
      "created_at": "2026-11-19T09:02:41+00:00"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/library/editions/{id}/bookmarks` answer.",
  "required": [
    "bookmarks"
  ],
  "properties": {
    "bookmarks": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A place the caller marked in an edition.",
        "required": [
          "id",
          "edition_id",
          "position_ms",
          "title",
          "created_at"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Bookmark id.",
            "example": "9a1b2c3d-4e5f-4a6b-8c7d-0e1f2a3b4c5d"
          },
          "edition_id": {
            "type": "string",
            "description": "The edition it is in.",
            "example": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d"
          },
          "position_ms": {
            "type": "integer",
            "format": "int64",
            "description": "The marked place, in milliseconds from the edition's start.",
            "example": 754000,
            "minimum": 0
          },
          "title": {
            "type": "string",
            "description": "The caller's note; may be empty.",
            "example": "The storm"
          },
          "created_at": {
            "type": "string",
            "description": "When it was made (RFC 3339).",
            "example": "2026-11-19T09:02:41+00:00"
          }
        }
      },
      "description": "The caller's bookmarks in the edition, in timeline order."
    }
  }
}
```

Status `401`: The session ended.

```json
{
  "error": "sign in again"
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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

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

Status `404`: No such edition.

```json
{
  "error": "edition 5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d not found"
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

## `POST /api/library/editions/{id}/bookmarks`

Adds a bookmark of the caller's in an edition.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Edition id.

### Request body

`application/json`: [`NewBookmarkBody`](schemas.md#newbookmarkbody)

- `position_ms` *integer (int64)*, **required** — The place to mark, in milliseconds (at most sixty days). At least 0.
- `title` *string* — A note for it, at most 200 bytes, no control characters; empty when left out.

### Responses

- `200` — The new bookmark. Body: [`BookmarkJson`](schemas.md#bookmarkjson) (`application/json`)
  - `id` *string*, **required** — Bookmark id.
  - `edition_id` *string*, **required** — The edition it is in.
  - `position_ms` *integer (int64)*, **required** — The marked place, in milliseconds from the edition's start. At least 0.
  - `title` *string*, **required** — The caller's note; may be empty.
  - `created_at` *string*, **required** — When it was made (RFC 3339).
- `400` — The position is past sixty days, or the title is too long or has control characters. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The session ended. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The caller already keeps 200 bookmarks in the edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The body is over 4096 bytes. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/bookmarks' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"position_ms": 754000}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/bookmarks' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"position_ms": 754000}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/library/editions/<id>/bookmarks",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"position_ms": 754000},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/bookmarks", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"position_ms": 754000}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/library/editions/<id>/bookmarks")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"position_ms": 754000}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The new bookmark.

```json
{
  "id": "9a1b2c3d-4e5f-4a6b-8c7d-0e1f2a3b4c5d",
  "edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d",
  "position_ms": 754000,
  "title": "The storm",
  "created_at": "2026-11-19T09:02:41+00:00"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A place the caller marked in an edition.",
  "required": [
    "id",
    "edition_id",
    "position_ms",
    "title",
    "created_at"
  ],
  "properties": {
    "id": {
      "type": "string",
      "description": "Bookmark id.",
      "example": "9a1b2c3d-4e5f-4a6b-8c7d-0e1f2a3b4c5d"
    },
    "edition_id": {
      "type": "string",
      "description": "The edition it is in.",
      "example": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d"
    },
    "position_ms": {
      "type": "integer",
      "format": "int64",
      "description": "The marked place, in milliseconds from the edition's start.",
      "example": 754000,
      "minimum": 0
    },
    "title": {
      "type": "string",
      "description": "The caller's note; may be empty.",
      "example": "The storm"
    },
    "created_at": {
      "type": "string",
      "description": "When it was made (RFC 3339).",
      "example": "2026-11-19T09:02:41+00:00"
    }
  }
}
```

Status `400`: The position is past sixty days, or the title is too long or has control characters. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "title must be at most 200 bytes"
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

Status `401`: The session ended.

```json
{
  "error": "sign in again"
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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such edition.

```json
{
  "error": "edition 5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d not found"
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

Status `409`: The caller already keeps 200 bookmarks in the edition.

```json
{
  "error": "an edition takes at most 200 bookmarks"
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

Status `413`: The body is over 4096 bytes.

```json
{
  "error": "the body is at most 4096 bytes"
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
Failed to deserialize the JSON body into the target type: missing field `position_ms` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The library could not be read or written.

```json
{
  "error": "the listening state could not be written"
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

## `GET /api/library/editions/{id}/sample`

Plays a few minutes of an edition this node holds.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Edition id (an `id` in a work's `editions`).
- `Range` (header) *string or null* — One byte range (`bytes=A-B`, `bytes=A-` or `bytes=-N`) of a sample already kept; anything else, or a sample still being cut, reads the whole sample.

### Responses

- `200` — The sample: mono MP3, five minutes or the rest of the first file. A sample not kept yet streams as the media worker encodes it, without `Content-Length`. Body: [`BinaryBody`](schemas.md#binarybody) (`audio/mpeg`)
- `206` — The requested byte range, with `Content-Range`. Body: [`BinaryBody`](schemas.md#binarybody) (`audio/mpeg`)
- `401` — The session ended. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition (`edition … not found`), or this node holds none of its audio (`no_sample`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `416` — The range starts at or past the end of the sample (`range_not_satisfiable`); `Content-Range` states the size. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `422` — The edition's audio could not be decoded (`no_sample`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — No confined media worker or storage is available; retry (`unavailable`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/editions/<id>/sample' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/editions/<id>/sample' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/editions/<id>/sample",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/editions/<id>/sample", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/editions/<id>/sample")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The sample: mono MP3, five minutes or the rest of the first file. A sample not kept yet streams as the media worker encodes it, without `Content-Length`. The body is binary (`audio/mpeg`).

Schema:

A `audio/mpeg` body: binary bytes.

Status `206`: The requested byte range, with `Content-Range`. The body is binary (`audio/mpeg`).

Schema:

A `audio/mpeg` body: binary bytes.

Status `401`: The session ended.

```json
{
  "error": "sign in again"
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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

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

Status `404`: No such edition (`edition … not found`), or this node holds none of its audio (`no_sample`).

```json
{
  "error": "no_sample"
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

Status `416`: The range starts at or past the end of the sample (`range_not_satisfiable`); `Content-Range` states the size.

```json
{
  "error": "range_not_satisfiable"
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

Status `422`: The edition's audio could not be decoded (`no_sample`).

```json
{
  "error": "no_sample"
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

Status `503`: No confined media worker or storage is available; retry (`unavailable`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

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
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/uploads`

Lists the caller's uploads, newest first.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Responses

- `200` — The caller's uploads. Body: [`UploadList`](schemas.md#uploadlist) (`application/json`)
  - `uploads` *array of [`UploadJson`](schemas.md#uploadjson)*, **required** — At most 50 uploads, without their files.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The sign-in is linked to no patron. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The uploads could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/uploads' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/uploads' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/uploads",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/uploads", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/uploads")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's uploads.

```json
{
  "uploads": [
    {
      "id": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99",
      "state": "open",
      "source": "upload",
      "fileCount": 2,
      "declaredBytes": 734068736,
      "jobId": null,
      "detail": null,
      "createdAt": "2026-11-20T09:00:00Z",
      "updatedAt": "2026-11-20T09:05:00Z",
      "finishedAt": null,
      "files": [
        {
          "memberNo": 0,
          "path": "Dune/01 Dune.m4b",
          "size": 734003200,
          "received": 734003200,
          "outcome": "matched"
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
  "description": "The caller's uploads, newest first.",
  "required": [
    "uploads"
  ],
  "properties": {
    "uploads": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One upload.",
        "required": [
          "id",
          "state",
          "source",
          "fileCount",
          "declaredBytes",
          "createdAt",
          "updatedAt"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "The upload id (32 hex digits).",
            "example": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99"
          },
          "state": {
            "type": "string",
            "description": "`open` (taking bytes), `committed` (queued), `running`, `done`,\n`failed` or `cancelled`.",
            "example": "open"
          },
          "source": {
            "type": "string",
            "description": "`upload`, or `intake` for a batch a drop folder made.",
            "example": "upload"
          },
          "fileCount": {
            "type": "integer",
            "format": "int64",
            "description": "Files the manifest named.",
            "example": 2,
            "minimum": 0
          },
          "declaredBytes": {
            "type": "integer",
            "format": "int64",
            "description": "Bytes the manifest declared.",
            "example": 734068736,
            "minimum": 0
          },
          "jobId": {
            "type": [
              "string",
              "null"
            ],
            "description": "The ingest job, once committed.",
            "example": null
          },
          "detail": {
            "type": [
              "string",
              "null"
            ],
            "description": "Why the upload failed.",
            "example": null
          },
          "createdAt": {
            "type": "string",
            "description": "When it was created (RFC 3339).",
            "example": "2026-11-20T09:00:00Z"
          },
          "updatedAt": {
            "type": "string",
            "description": "Its last activity (RFC 3339).",
            "example": "2026-11-20T09:05:00Z"
          },
          "finishedAt": {
            "type": [
              "string",
              "null"
            ],
            "description": "When it finished (RFC 3339).",
            "example": null
          },
          "files": {
            "type": [
              "array",
              "null"
            ],
            "items": {
              "type": "object",
              "description": "One file of an upload.",
              "required": [
                "memberNo",
                "path",
                "size",
                "received",
                "outcome"
              ],
              "properties": {
                "memberNo": {
                  "type": "integer",
                  "format": "int32",
                  "description": "Its number in the upload; manifest files are numbered from 0 in\norder, an archive's entries after every earlier file.",
                  "example": 0,
                  "minimum": 0
                },
                "parentNo": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int32",
                  "description": "The archive it was expanded from.",
                  "example": null,
                  "minimum": 0
                },
                "path": {
                  "type": "string",
                  "description": "Its path in the upload (for a refused archive entry, its cleaned\nname).",
                  "example": "Dune/01 Dune.m4b"
                },
                "size": {
                  "type": "integer",
                  "format": "int64",
                  "description": "Its size in bytes.",
                  "example": 734003200,
                  "minimum": 0
                },
                "received": {
                  "type": "integer",
                  "format": "int64",
                  "description": "Bytes received so far; resume a `PATCH` here.",
                  "example": 734003200,
                  "minimum": 0
                },
                "outcome": {
                  "type": "string",
                  "description": "What became of it: `pending`, `expanded` (an archive), `matched`,\n`needs_review`, `duplicate`, `refused` or `failed`.",
                  "example": "matched"
                },
                "reason": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Why it was refused or failed, as a stable code.",
                  "example": null
                },
                "editionId": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "The edition it was stored under.",
                  "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
                }
              }
            },
            "description": "Its files; left out of lists."
          }
        }
      },
      "description": "At most 50 uploads, without their files."
    }
  }
}
```

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The sign-in is linked to no patron. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

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

Status `500`: The uploads could not be read.

```json
{
  "error": "the upload could not be read or written"
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

## `POST /api/uploads`

Starts an upload from a manifest of the files to come.

The caller must be allowed to upload (the `upload` member control). The
operator's uploads are held by the library's first active Owner. Paths
are relative (`Folder/File.m4b`), at most 10,000 files, each within
`uploads.max_file_bytes`, all within the caller's staging quota.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Request body

`application/json`: [`UploadManifest`](schemas.md#uploadmanifest)

- `files` *array of [`ManifestFile`](schemas.md#manifestfile)*, **required** — Every file of the batch, in the order their numbers follow.

### Responses

- `201` — The upload is open for bytes. Body: [`UploadJson`](schemas.md#uploadjson) (`application/json`)
  - `id` *string*, **required** — The upload id (32 hex digits).
  - `state` *string*, **required** — `open` (taking bytes), `committed` (queued), `running`, `done`, `failed` or `cancelled`.
  - `source` *string*, **required** — `upload`, or `intake` for a batch a drop folder made.
  - `fileCount` *integer (int64)*, **required** — Files the manifest named. At least 0.
  - `declaredBytes` *integer (int64)*, **required** — Bytes the manifest declared. At least 0.
  - `jobId` *string or null* — The ingest job, once committed.
  - `detail` *string or null* — Why the upload failed.
  - `createdAt` *string*, **required** — When it was created (RFC 3339).
  - `updatedAt` *string*, **required** — Its last activity (RFC 3339).
  - `finishedAt` *string or null* — When it finished (RFC 3339).
  - `files` *array of [`UploadMemberJson`](schemas.md#uploadmemberjson) or null* — Its files; left out of lists.
- `400` — The manifest is malformed, names no file or too many, or a path is refused or clashes. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller may not upload. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `413` — A file passes the per-file limit or the upload passes the staging quota. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The body is not JSON. As `text/plain`, when the request did not declare a JSON body. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The upload could not be created. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)
- `507` — The disk would drop below its free-space floor. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/uploads' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"files": [{"path": "The Hobbit/01 Chapter One.mp3", "size": 52428800}]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/uploads' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"files": [{"path": "The Hobbit/01 Chapter One.mp3", "size": 52428800}]}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/uploads",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"files": [{"path": "The Hobbit/01 Chapter One.mp3", "size": 52428800}]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/uploads", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"files": [{"path": "The Hobbit/01 Chapter One.mp3", "size": 52428800}]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/uploads")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"files": [{"path": "The Hobbit/01 Chapter One.mp3", "size": 52428800}]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `201`: The upload is open for bytes.

```json
{
  "id": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99",
  "state": "open",
  "source": "upload",
  "fileCount": 2,
  "declaredBytes": 734068736,
  "jobId": null,
  "detail": null,
  "createdAt": "2026-11-20T09:00:00Z",
  "updatedAt": "2026-11-20T09:05:00Z",
  "finishedAt": null,
  "files": [
    {
      "memberNo": 0,
      "parentNo": null,
      "path": "Dune/01 Dune.m4b",
      "size": 734003200,
      "received": 734003200,
      "outcome": "matched",
      "reason": null,
      "editionId": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "One upload.",
  "required": [
    "id",
    "state",
    "source",
    "fileCount",
    "declaredBytes",
    "createdAt",
    "updatedAt"
  ],
  "properties": {
    "id": {
      "type": "string",
      "description": "The upload id (32 hex digits).",
      "example": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99"
    },
    "state": {
      "type": "string",
      "description": "`open` (taking bytes), `committed` (queued), `running`, `done`,\n`failed` or `cancelled`.",
      "example": "open"
    },
    "source": {
      "type": "string",
      "description": "`upload`, or `intake` for a batch a drop folder made.",
      "example": "upload"
    },
    "fileCount": {
      "type": "integer",
      "format": "int64",
      "description": "Files the manifest named.",
      "example": 2,
      "minimum": 0
    },
    "declaredBytes": {
      "type": "integer",
      "format": "int64",
      "description": "Bytes the manifest declared.",
      "example": 734068736,
      "minimum": 0
    },
    "jobId": {
      "type": [
        "string",
        "null"
      ],
      "description": "The ingest job, once committed.",
      "example": null
    },
    "detail": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why the upload failed.",
      "example": null
    },
    "createdAt": {
      "type": "string",
      "description": "When it was created (RFC 3339).",
      "example": "2026-11-20T09:00:00Z"
    },
    "updatedAt": {
      "type": "string",
      "description": "Its last activity (RFC 3339).",
      "example": "2026-11-20T09:05:00Z"
    },
    "finishedAt": {
      "type": [
        "string",
        "null"
      ],
      "description": "When it finished (RFC 3339).",
      "example": null
    },
    "files": {
      "type": [
        "array",
        "null"
      ],
      "items": {
        "type": "object",
        "description": "One file of an upload.",
        "required": [
          "memberNo",
          "path",
          "size",
          "received",
          "outcome"
        ],
        "properties": {
          "memberNo": {
            "type": "integer",
            "format": "int32",
            "description": "Its number in the upload; manifest files are numbered from 0 in\norder, an archive's entries after every earlier file.",
            "example": 0,
            "minimum": 0
          },
          "parentNo": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int32",
            "description": "The archive it was expanded from.",
            "example": null,
            "minimum": 0
          },
          "path": {
            "type": "string",
            "description": "Its path in the upload (for a refused archive entry, its cleaned\nname).",
            "example": "Dune/01 Dune.m4b"
          },
          "size": {
            "type": "integer",
            "format": "int64",
            "description": "Its size in bytes.",
            "example": 734003200,
            "minimum": 0
          },
          "received": {
            "type": "integer",
            "format": "int64",
            "description": "Bytes received so far; resume a `PATCH` here.",
            "example": 734003200,
            "minimum": 0
          },
          "outcome": {
            "type": "string",
            "description": "What became of it: `pending`, `expanded` (an archive), `matched`,\n`needs_review`, `duplicate`, `refused` or `failed`.",
            "example": "matched"
          },
          "reason": {
            "type": [
              "string",
              "null"
            ],
            "description": "Why it was refused or failed, as a stable code.",
            "example": null
          },
          "editionId": {
            "type": [
              "string",
              "null"
            ],
            "description": "The edition it was stored under.",
            "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
          }
        }
      },
      "description": "Its files; left out of lists."
    }
  }
}
```

Status `400`: The manifest is malformed, names no file or too many, or a path is refused or clashes. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "file 1 has the same path as an earlier file"
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

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller may not upload. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `413`: A file passes the per-file limit or the upload passes the staging quota.

```json
{
  "error": "file 0 is larger than the per-file limit"
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

Status `415`: The body is not JSON. As `text/plain`, when the request did not declare a JSON body.

```json
{
  "error": "send the manifest as application/json"
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

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `files` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The upload could not be created.

```json
{
  "error": "the upload could not be read or written"
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

Status `507`: The disk would drop below its free-space floor.

```json
{
  "error": "this server is low on disk space"
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

## `GET /api/uploads/review`

Lists uploaded books stored under a new local edition because nothing
strong identified them, for an Administrator to check and, when they
are a known edition, pin.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `cursor` (path) *string or null*, **required** — The `next_cursor` of the previous page.

### Responses

- `200` — One page of books to review. Body: [`ReviewPage`](schemas.md#reviewpage) (`application/json`)
  - `items` *array of [`ReviewItemJson`](schemas.md#reviewitemjson)*, **required** — The books, oldest upload first.
  - `next_cursor` *string or null* — Pass as `cursor` for the next page; absent when this page is empty.
- `400` — The cursor is not one this route made. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The list could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/uploads/review' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/uploads/review' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/uploads/review",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/uploads/review", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/uploads/review")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: One page of books to review.

```json
{
  "items": [
    {
      "upload_id": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99",
      "member_no": 0,
      "path": "Unknown/track01.mp3",
      "edition_id": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b",
      "user_id": 3
    }
  ],
  "next_cursor": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99:0"
}
```

Schema:

```json
{
  "type": "object",
  "description": "One page of books to review.",
  "required": [
    "items"
  ],
  "properties": {
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One stored book to review.",
        "required": [
          "upload_id",
          "member_no",
          "path",
          "user_id"
        ],
        "properties": {
          "upload_id": {
            "type": "string",
            "description": "The upload.",
            "example": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99"
          },
          "member_no": {
            "type": "integer",
            "format": "int32",
            "description": "The file's number in it.",
            "example": 0,
            "minimum": 0
          },
          "path": {
            "type": "string",
            "description": "The file's path in the upload.",
            "example": "Unknown/track01.mp3"
          },
          "edition_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "The local edition it was stored under.",
            "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
          },
          "user_id": {
            "type": "integer",
            "format": "int64",
            "description": "The patron who uploaded it.",
            "example": 3
          }
        }
      },
      "description": "The books, oldest upload first."
    },
    "next_cursor": {
      "type": [
        "string",
        "null"
      ],
      "description": "Pass as `cursor` for the next page; absent when this page is empty.",
      "example": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99:0"
    }
  }
}
```

Status `400`: The cursor is not one this route made.

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

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

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

Status `500`: The list could not be read.

```json
{
  "error": "the upload could not be read or written"
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

## `GET /api/uploads/{id}`

Reads one of the caller's uploads with every file: how many bytes each
has received (where to resume) and, once ingested, what became of it.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Upload id.

### Responses

- `200` — The upload and its files. Body: [`UploadJson`](schemas.md#uploadjson) (`application/json`)
  - `id` *string*, **required** — The upload id (32 hex digits).
  - `state` *string*, **required** — `open` (taking bytes), `committed` (queued), `running`, `done`, `failed` or `cancelled`.
  - `source` *string*, **required** — `upload`, or `intake` for a batch a drop folder made.
  - `fileCount` *integer (int64)*, **required** — Files the manifest named. At least 0.
  - `declaredBytes` *integer (int64)*, **required** — Bytes the manifest declared. At least 0.
  - `jobId` *string or null* — The ingest job, once committed.
  - `detail` *string or null* — Why the upload failed.
  - `createdAt` *string*, **required** — When it was created (RFC 3339).
  - `updatedAt` *string*, **required** — Its last activity (RFC 3339).
  - `finishedAt` *string or null* — When it finished (RFC 3339).
  - `files` *array of [`UploadMemberJson`](schemas.md#uploadmemberjson) or null* — Its files; left out of lists.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such upload of the caller's. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The upload could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/uploads/<id>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/uploads/<id>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/uploads/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/uploads/<id>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/uploads/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The upload and its files.

```json
{
  "id": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99",
  "state": "open",
  "source": "upload",
  "fileCount": 2,
  "declaredBytes": 734068736,
  "jobId": null,
  "detail": null,
  "createdAt": "2026-11-20T09:00:00Z",
  "updatedAt": "2026-11-20T09:05:00Z",
  "finishedAt": null,
  "files": [
    {
      "memberNo": 0,
      "parentNo": null,
      "path": "Dune/01 Dune.m4b",
      "size": 734003200,
      "received": 734003200,
      "outcome": "matched",
      "reason": null,
      "editionId": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "One upload.",
  "required": [
    "id",
    "state",
    "source",
    "fileCount",
    "declaredBytes",
    "createdAt",
    "updatedAt"
  ],
  "properties": {
    "id": {
      "type": "string",
      "description": "The upload id (32 hex digits).",
      "example": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99"
    },
    "state": {
      "type": "string",
      "description": "`open` (taking bytes), `committed` (queued), `running`, `done`,\n`failed` or `cancelled`.",
      "example": "open"
    },
    "source": {
      "type": "string",
      "description": "`upload`, or `intake` for a batch a drop folder made.",
      "example": "upload"
    },
    "fileCount": {
      "type": "integer",
      "format": "int64",
      "description": "Files the manifest named.",
      "example": 2,
      "minimum": 0
    },
    "declaredBytes": {
      "type": "integer",
      "format": "int64",
      "description": "Bytes the manifest declared.",
      "example": 734068736,
      "minimum": 0
    },
    "jobId": {
      "type": [
        "string",
        "null"
      ],
      "description": "The ingest job, once committed.",
      "example": null
    },
    "detail": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why the upload failed.",
      "example": null
    },
    "createdAt": {
      "type": "string",
      "description": "When it was created (RFC 3339).",
      "example": "2026-11-20T09:00:00Z"
    },
    "updatedAt": {
      "type": "string",
      "description": "Its last activity (RFC 3339).",
      "example": "2026-11-20T09:05:00Z"
    },
    "finishedAt": {
      "type": [
        "string",
        "null"
      ],
      "description": "When it finished (RFC 3339).",
      "example": null
    },
    "files": {
      "type": [
        "array",
        "null"
      ],
      "items": {
        "type": "object",
        "description": "One file of an upload.",
        "required": [
          "memberNo",
          "path",
          "size",
          "received",
          "outcome"
        ],
        "properties": {
          "memberNo": {
            "type": "integer",
            "format": "int32",
            "description": "Its number in the upload; manifest files are numbered from 0 in\norder, an archive's entries after every earlier file.",
            "example": 0,
            "minimum": 0
          },
          "parentNo": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int32",
            "description": "The archive it was expanded from.",
            "example": null,
            "minimum": 0
          },
          "path": {
            "type": "string",
            "description": "Its path in the upload (for a refused archive entry, its cleaned\nname).",
            "example": "Dune/01 Dune.m4b"
          },
          "size": {
            "type": "integer",
            "format": "int64",
            "description": "Its size in bytes.",
            "example": 734003200,
            "minimum": 0
          },
          "received": {
            "type": "integer",
            "format": "int64",
            "description": "Bytes received so far; resume a `PATCH` here.",
            "example": 734003200,
            "minimum": 0
          },
          "outcome": {
            "type": "string",
            "description": "What became of it: `pending`, `expanded` (an archive), `matched`,\n`needs_review`, `duplicate`, `refused` or `failed`.",
            "example": "matched"
          },
          "reason": {
            "type": [
              "string",
              "null"
            ],
            "description": "Why it was refused or failed, as a stable code.",
            "example": null
          },
          "editionId": {
            "type": [
              "string",
              "null"
            ],
            "description": "The edition it was stored under.",
            "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
          }
        }
      },
      "description": "Its files; left out of lists."
    }
  }
}
```

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `404`: No such upload of the caller's.

```json
{
  "error": "no such upload"
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

Status `500`: The upload could not be read.

```json
{
  "error": "the upload could not be read or written"
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

## `DELETE /api/uploads/{id}`

Cancels an upload that has not started ingesting and drops its bytes.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Upload id.

### Responses

- `204` — The upload is cancelled. No body.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such upload of the caller's. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The upload is already ingesting or finished. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The upload could not be cancelled. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/uploads/<id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/uploads/<id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/uploads/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/uploads/<id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/uploads/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The upload is cancelled. No body.

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such upload of the caller's.

```json
{
  "error": "no such upload"
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

Status `409`: The upload is already ingesting or finished.

```json
{
  "error": "the upload is already ingesting or finished"
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

Status `500`: The upload could not be cancelled.

```json
{
  "error": "the upload could not be read or written"
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

## `PATCH /api/uploads/{id}/files/{member_no}`

Sends bytes of one file, from the offset in the `Upload-Offset` header.

The body is `application/offset+octet-stream`. A request that breaks
off keeps the bytes that arrived; read the upload (or the answer's
`received`) and resume from there. Bytes past the file's declared size
are refused.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Upload id.
- `member_no` (path) *integer (int32)*, **required** — The file's number in the manifest.
- `Upload-Offset` (header) *integer (int64)*, **required** — Bytes of the file already received.

### Request body

`application/offset+octet-stream`: [`BinaryBody`](schemas.md#binarybody)

### Responses

- `200` — The bytes were stored. Body: [`ReceivedJson`](schemas.md#receivedjson) (`application/json`)
  - `memberNo` *integer (int32)*, **required** — The file's number. At least 0.
  - `received` *integer (int64)*, **required** — Bytes received; send the next `PATCH` from here. At least 0.
  - `complete` *boolean*, **required** — True once every byte has arrived.
- `400` — The Upload-Offset header is missing or malformed. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such upload of the caller's, or no such file in it. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The offset is not where the file stands, another request is sending this file, or the upload no longer takes bytes. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The bytes run past the file's declared size. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The body is not application/offset+octet-stream. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The bytes could not be stored. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)
- `507` — The disk would drop below its free-space floor. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/uploads/<id>/files/<member_no>' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/offset+octet-stream' \
    --data-binary @file
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/uploads/<id>/files/<member_no>' `
    -Method Patch `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/offset+octet-stream' `
    -InFile 'file'
```

```python
import requests

response = requests.patch(
    "http://127.0.0.1:8787/api/uploads/<id>/files/<member_no>",
    headers={"Authorization": "Bearer <operatorToken>", "Content-Type": "application/offset+octet-stream"},
    data=open("file", "rb"),
)
print(response.status_code, response.text)
```

```javascript
import { readFile } from "node:fs/promises";

const response = await fetch("http://127.0.0.1:8787/api/uploads/<id>/files/<member_no>", {
  method: "PATCH",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/offset+octet-stream" },
  body: await readFile("file"),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .patch("http://127.0.0.1:8787/api/uploads/<id>/files/<member_no>")
    .header("Authorization", "Bearer <operatorToken>")
    .header("Content-Type", "application/offset+octet-stream")
    .body(std::fs::read("file")?)
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The bytes were stored.

```json
{
  "memberNo": 0,
  "received": 67108864,
  "complete": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Where a file's bytes stand after a `PATCH`.",
  "required": [
    "memberNo",
    "received",
    "complete"
  ],
  "properties": {
    "memberNo": {
      "type": "integer",
      "format": "int32",
      "description": "The file's number.",
      "example": 0,
      "minimum": 0
    },
    "received": {
      "type": "integer",
      "format": "int64",
      "description": "Bytes received; send the next `PATCH` from here.",
      "example": 67108864,
      "minimum": 0
    },
    "complete": {
      "type": "boolean",
      "description": "True once every byte has arrived.",
      "example": false
    }
  }
}
```

Status `400`: The Upload-Offset header is missing or malformed.

```json
{
  "error": "send the Upload-Offset header"
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

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such upload of the caller's, or no such file in it.

```json
{
  "error": "no such upload"
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

Status `409`: The offset is not where the file stands, another request is sending this file, or the upload no longer takes bytes.

```json
{
  "error": "the file has 1024 bytes; resume from there"
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

Status `413`: The bytes run past the file's declared size.

```json
{
  "error": "the bytes run past the file's declared size"
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

Status `415`: The body is not application/offset+octet-stream.

```json
{
  "error": "send the bytes as application/offset+octet-stream"
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

Status `500`: The bytes could not be stored.

```json
{
  "error": "the upload could not be read or written"
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

Status `507`: The disk would drop below its free-space floor.

```json
{
  "error": "this server is low on disk space"
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

## `POST /api/uploads/{id}/commit`

Commits an upload whose files have all arrived: queues its ingest job.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Upload id.

### Responses

- `202` — The ingest job is queued. Body: [`UploadJson`](schemas.md#uploadjson) (`application/json`)
  - `id` *string*, **required** — The upload id (32 hex digits).
  - `state` *string*, **required** — `open` (taking bytes), `committed` (queued), `running`, `done`, `failed` or `cancelled`.
  - `source` *string*, **required** — `upload`, or `intake` for a batch a drop folder made.
  - `fileCount` *integer (int64)*, **required** — Files the manifest named. At least 0.
  - `declaredBytes` *integer (int64)*, **required** — Bytes the manifest declared. At least 0.
  - `jobId` *string or null* — The ingest job, once committed.
  - `detail` *string or null* — Why the upload failed.
  - `createdAt` *string*, **required** — When it was created (RFC 3339).
  - `updatedAt` *string*, **required** — Its last activity (RFC 3339).
  - `finishedAt` *string or null* — When it finished (RFC 3339).
  - `files` *array of [`UploadMemberJson`](schemas.md#uploadmemberjson) or null* — Its files; left out of lists.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such upload of the caller's. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — Files are still missing bytes, or the upload is not open. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The upload could not be committed. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The job queue is full; try again later. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/uploads/<id>/commit' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/uploads/<id>/commit' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/uploads/<id>/commit",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/uploads/<id>/commit", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/uploads/<id>/commit")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `202`: The ingest job is queued.

```json
{
  "id": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99",
  "state": "open",
  "source": "upload",
  "fileCount": 2,
  "declaredBytes": 734068736,
  "jobId": null,
  "detail": null,
  "createdAt": "2026-11-20T09:00:00Z",
  "updatedAt": "2026-11-20T09:05:00Z",
  "finishedAt": null,
  "files": [
    {
      "memberNo": 0,
      "parentNo": null,
      "path": "Dune/01 Dune.m4b",
      "size": 734003200,
      "received": 734003200,
      "outcome": "matched",
      "reason": null,
      "editionId": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "One upload.",
  "required": [
    "id",
    "state",
    "source",
    "fileCount",
    "declaredBytes",
    "createdAt",
    "updatedAt"
  ],
  "properties": {
    "id": {
      "type": "string",
      "description": "The upload id (32 hex digits).",
      "example": "4f3c2b1a0e9d8c7b6a5f4e3d2c1b0a99"
    },
    "state": {
      "type": "string",
      "description": "`open` (taking bytes), `committed` (queued), `running`, `done`,\n`failed` or `cancelled`.",
      "example": "open"
    },
    "source": {
      "type": "string",
      "description": "`upload`, or `intake` for a batch a drop folder made.",
      "example": "upload"
    },
    "fileCount": {
      "type": "integer",
      "format": "int64",
      "description": "Files the manifest named.",
      "example": 2,
      "minimum": 0
    },
    "declaredBytes": {
      "type": "integer",
      "format": "int64",
      "description": "Bytes the manifest declared.",
      "example": 734068736,
      "minimum": 0
    },
    "jobId": {
      "type": [
        "string",
        "null"
      ],
      "description": "The ingest job, once committed.",
      "example": null
    },
    "detail": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why the upload failed.",
      "example": null
    },
    "createdAt": {
      "type": "string",
      "description": "When it was created (RFC 3339).",
      "example": "2026-11-20T09:00:00Z"
    },
    "updatedAt": {
      "type": "string",
      "description": "Its last activity (RFC 3339).",
      "example": "2026-11-20T09:05:00Z"
    },
    "finishedAt": {
      "type": [
        "string",
        "null"
      ],
      "description": "When it finished (RFC 3339).",
      "example": null
    },
    "files": {
      "type": [
        "array",
        "null"
      ],
      "items": {
        "type": "object",
        "description": "One file of an upload.",
        "required": [
          "memberNo",
          "path",
          "size",
          "received",
          "outcome"
        ],
        "properties": {
          "memberNo": {
            "type": "integer",
            "format": "int32",
            "description": "Its number in the upload; manifest files are numbered from 0 in\norder, an archive's entries after every earlier file.",
            "example": 0,
            "minimum": 0
          },
          "parentNo": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int32",
            "description": "The archive it was expanded from.",
            "example": null,
            "minimum": 0
          },
          "path": {
            "type": "string",
            "description": "Its path in the upload (for a refused archive entry, its cleaned\nname).",
            "example": "Dune/01 Dune.m4b"
          },
          "size": {
            "type": "integer",
            "format": "int64",
            "description": "Its size in bytes.",
            "example": 734003200,
            "minimum": 0
          },
          "received": {
            "type": "integer",
            "format": "int64",
            "description": "Bytes received so far; resume a `PATCH` here.",
            "example": 734003200,
            "minimum": 0
          },
          "outcome": {
            "type": "string",
            "description": "What became of it: `pending`, `expanded` (an archive), `matched`,\n`needs_review`, `duplicate`, `refused` or `failed`.",
            "example": "matched"
          },
          "reason": {
            "type": [
              "string",
              "null"
            ],
            "description": "Why it was refused or failed, as a stable code.",
            "example": null
          },
          "editionId": {
            "type": [
              "string",
              "null"
            ],
            "description": "The edition it was stored under.",
            "example": "6c1f2e8a-4b3d-4f5e-8a9b-0c1d2e3f4a5b"
          }
        }
      },
      "description": "Its files; left out of lists."
    }
  }
}
```

Status `401`: No one is signed in.

```json
{
  "error": "sign in first"
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

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such upload of the caller's.

```json
{
  "error": "no such upload"
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

Status `409`: Files are still missing bytes, or the upload is not open.

```json
{
  "error": "2 files are still missing bytes"
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

Status `500`: The upload could not be committed.

```json
{
  "error": "the upload could not be read or written"
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

Status `503`: The job queue is full; try again later. When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "the job queue is full"
}
```

Schema:

```json
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
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.
