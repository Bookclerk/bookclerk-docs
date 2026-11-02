# Cluster

Nodes, clerks, branches, joining a library and host enrollment.

Back to the [web API reference](README.md).

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
  - `cluster_id` *string*, **required** — Cluster the node belongs to.
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
  - `kind` *[`NodeKind`](schemas.md#nodekind)*, **required** — `owner` for a node sharing this library, `host` for an enrolled host.
  - `owner_user_id` *integer (int64) or null* — User id of the node's owner; `null` when unassigned.
  - `share_books` *boolean or null* — A branch's sharing setting, as it last reported it; `null` for an owner node.
  - `include_branches` *boolean or null* — A branch's include-branches setting, as it last reported it; `null` for an owner node.
  - `owner` *[`NodeOwnerSummary`](schemas.md#nodeownersummary) or null* — The owner, when the node has one and the user still exists.
- `400` — The body is invalid, or a label is malformed. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
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
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes/<node_id>/labels' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"labels": ["string"]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/nodes/<node_id>/labels' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"labels": ["string"]}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/nodes/<node_id>/labels",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"labels": ["string"]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/nodes/<node_id>/labels", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"labels": ["string"]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/nodes/<node_id>/labels")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"labels": ["string"]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The node after the change.

```json
{
  "node_id": "node-7f3a9c",
  "cluster_id": "c0ffee12-3456-4789-8abc-def012345678",
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
    "schedulable": true
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
    "online_since_ms": 1793167200000
  },
  "this_node": true,
  "kind": "owner",
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
    "cluster_id",
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
    "kind"
  ],
  "properties": {
    "node_id": {
      "type": "string",
      "description": "Node id.",
      "example": "node-7f3a9c"
    },
    "cluster_id": {
      "type": "string",
      "description": "Cluster the node belongs to.",
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
      "description": "`owner` for a node sharing this library, `host` for an enrolled\nhost.",
      "enum": [
        "owner",
        "host"
      ],
      "example": "owner"
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
      "description": "A branch's sharing setting, as it last reported it; `null` for an\nowner node.",
      "example": true
    },
    "include_branches": {
      "type": [
        "boolean",
        "null"
      ],
      "description": "A branch's include-branches setting, as it last reported it;\n`null` for an owner node.",
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
                "host",
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

Status `400`: The body is invalid, or a label is malformed.

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

Status `404`: No such node.

```json
{
  "error": "book not found: node 9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e"
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
- `400` — The user is disabled and cannot own a node. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
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

Status `400`: The user is disabled and cannot own a node.

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

Status `404`: No such node or user.

```json
{
  "error": "book not found: user 42"
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

## `GET /api/config/cluster-consents`

`GET /api/config/cluster-consents`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — Pending and granted cluster consents. Body: [`ClusterConsentsResponse`](schemas.md#clusterconsentsresponse) (`application/json`)
  - `clusters` *array of [`ClusterLinkView`](schemas.md#clusterlinkview)*, **required** — Every library this install is a branch of.
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
$ curl 'http://127.0.0.1:8787/api/config/cluster-consents' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/cluster-consents' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/cluster-consents",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/cluster-consents", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/cluster-consents")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Pending and granted cluster consents.

```json
{
  "clusters": [
    {
      "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
      "owner_origin": "https://books.example.com",
      "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
      "pinned": false,
      "state": "active",
      "enrolled_at": "2026-10-29T09:02:41Z",
      "share_books": true,
      "include_branches": true,
      "running": true
    }
  ],
  "consents": [
    {
      "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
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
      "state": "consented"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /api/config/cluster-consents`.",
  "required": [
    "clusters",
    "consents"
  ],
  "properties": {
    "clusters": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A library this install is a branch of: ids, origin and state, never key material.",
        "required": [
          "cluster_id",
          "owner_origin",
          "node_id",
          "pinned",
          "state",
          "enrolled_at",
          "share_books",
          "include_branches"
        ],
        "properties": {
          "cluster_id": {
            "type": "string",
            "description": "The library's cluster id.",
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
          "cluster_id",
          "plugin_key",
          "requested_version",
          "requested_grant",
          "requested_artifact",
          "request_digest",
          "grant",
          "updated_at",
          "state"
        ],
        "properties": {
          "cluster_id": {
            "type": "string",
            "description": "The owner library's cluster id.",
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

## `POST /api/config/clusters/{cluster_id}/scan`

`POST /api/config/clusters/{cluster_id}/scan`: asks the owner cluster
to queue a scan for this node's owner (H-5). The owner's planner decides
what that user may scan; the answer lists the queued job ids.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `cluster_id` (path) *string*, **required** — Owner cluster id.

### Request body

Optional.

`application/json`: [`ClusterScanBody`](schemas.md#clusterscanbody) or null

- `account` *string or null* — Scan only this store account of the owner cluster.
- `plugin_instance_id` *string or null* — Pin every planned job to this owner plugin instance.

### Responses

- `200` — The owner cluster accepted the scan. Body: [`ClusterScanPlanned`](schemas.md#clusterscanplanned) (`application/json`)
  - `cluster_id` *string*, **required** — The owner library's cluster id.
  - `created` *array of string*, **required** — Ids of the jobs the owner created.
  - `duplicates` *array of string*, **required** — Ids of jobs that were already pending or running.
  - `queue_full` *integer (int64)*, **required** — Planned jobs the owner's queue had no room for. At least 0.
- `400` — The account named is blank, or the owner refused the request as invalid (`invalid`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The owner refused the scan (`forbidden`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The owner cluster answered with an unexpected frame (`unexpected_answer`), or with an error code other than `forbidden` and `invalid`. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `503` — This daemon has no running link to that cluster (`not_connected`), or the owner did not answer (`owner_unavailable`). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clusters/<cluster_id>/scan' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/clusters/<cluster_id>/scan' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/clusters/<cluster_id>/scan",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/clusters/<cluster_id>/scan", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/clusters/<cluster_id>/scan")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The owner cluster accepted the scan.

```json
{
  "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
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
    "cluster_id",
    "created",
    "duplicates",
    "queue_full"
  ],
  "properties": {
    "cluster_id": {
      "type": "string",
      "description": "The owner library's cluster id.",
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

Status `400`: The account named is blank, or the owner refused the request as invalid (`invalid`).

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

Status `502`: The owner cluster answered with an unexpected frame (`unexpected_answer`), or with an error code other than `forbidden` and `invalid`.

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

Status `503`: This daemon has no running link to that cluster (`not_connected`), or the owner did not answer (`owner_unavailable`).

```json
{
  "detail": "this daemon has no running link to that owner cluster",
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

## `POST /api/config/clusters/{cluster_id}/jobs`

`POST /api/config/clusters/{cluster_id}/jobs`: the owner's view of jobs
this node may see (those it would be trusted to claim).

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `cluster_id` (path) *string*, **required** — Owner cluster id.

### Request body

`application/json`: [`ClusterJobsBody`](schemas.md#clusterjobsbody)

- `job_ids` *array of string*, **required** — Job ids on the owner cluster.

### Responses

- `200` — The owner's jobs for these accounts. Body: [`ClusterJobsResponse`](schemas.md#clusterjobsresponse) (`application/json`)
  - `jobs` *array of [`JobStatusEntry`](schemas.md#jobstatusentry)*, **required** — The owner's state of each job this install may see, in request order.
- `400` — Too many job ids (`too_many_jobs`), or the owner refused the request as invalid (`invalid`). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The owner refused the request (`forbidden`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The owner cluster answered with an unexpected frame (`unexpected_answer`), or with an error code other than `forbidden` and `invalid`. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `503` — This daemon has no running link to that cluster (`not_connected`), or the owner did not answer (`owner_unavailable`). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clusters/<cluster_id>/jobs' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"job_ids": ["string"]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/clusters/<cluster_id>/jobs' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"job_ids": ["string"]}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/clusters/<cluster_id>/jobs",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"job_ids": ["string"]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/clusters/<cluster_id>/jobs", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"job_ids": ["string"]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/clusters/<cluster_id>/jobs")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"job_ids": ["string"]}))
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
  "description": "JSON body for `POST /api/config/clusters/{cluster_id}/jobs`.",
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

Status `400`: Too many job ids (`too_many_jobs`), or the owner refused the request as invalid (`invalid`).

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

Status `502`: The owner cluster answered with an unexpected frame (`unexpected_answer`), or with an error code other than `forbidden` and `invalid`.

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

Status `503`: This daemon has no running link to that cluster (`not_connected`), or the owner did not answer (`owner_unavailable`).

```json
{
  "detail": "this daemon has no running link to that owner cluster",
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

## `POST /api/config/cluster-consents/approve`

`POST /api/config/cluster-consents/approve`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`ApproveBody`](schemas.md#approvebody)

- `cluster_id` *string*, **required** — Owner cluster id.
- `plugin_key` *string*, **required** — Canonical plugin key.
- `request_digest` *string*, **required** — The `request_digest` of the request the operator reviewed, as the listing shows it (refused when the recorded request is another one).
- `deny_domains` *array of string* — Domains to take out of the request. No duplicates.
- `deny_bindings` *array of string* — Bindings to take out of the request. No duplicates.
- `deny_network` *boolean* — Deny all network access.

### Responses

- `200` — The consent was granted. Body: [`ConsentChanged`](schemas.md#consentchanged) (`application/json`)
  - `consent` *[`RemoteConsentView`](schemas.md#remoteconsentview)*, **required** — The consent as stored now.
- `400` — The narrowing or the recorded request is not valid (`invalid_approval`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
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
- `404` — That cluster has not asked to run that plugin here (`not_requested`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `409` — The request changed since it was reviewed; review it again (`approval_refused`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `503` — The approval could not be stored; try again (`store_unavailable`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/cluster-consents/approve' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"cluster_id": "string", "plugin_key": "string", "request_digest": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/cluster-consents/approve' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"cluster_id": "string", "plugin_key": "string", "request_digest": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/cluster-consents/approve",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"cluster_id": "string", "plugin_key": "string", "request_digest": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/cluster-consents/approve", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"cluster_id": "string", "plugin_key": "string", "request_digest": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/cluster-consents/approve")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"cluster_id": "string", "plugin_key": "string", "request_digest": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The consent was granted.

```json
{
  "consent": {
    "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
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
    "state": "consented"
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
        "cluster_id",
        "plugin_key",
        "requested_version",
        "requested_grant",
        "requested_artifact",
        "request_digest",
        "grant",
        "updated_at",
        "state"
      ],
      "properties": {
        "cluster_id": {
          "type": "string",
          "description": "The owner library's cluster id.",
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
        }
      }
    }
  }
}
```

Status `400`: The narrowing or the recorded request is not valid (`invalid_approval`).

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

Status `404`: That cluster has not asked to run that plugin here (`not_requested`).

```json
{
  "detail": "cluster c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f has not asked to run bookclerk:source-libro here",
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

## `POST /api/config/cluster-consents/revoke`

`POST /api/config/cluster-consents/revoke`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`RevokeBody`](schemas.md#revokebody)

- `cluster_id` *string*, **required** — Owner cluster id.
- `plugin_key` *string*, **required** — Canonical plugin key.

### Responses

- `200` — The consent was revoked. Body: [`ConsentChanged`](schemas.md#consentchanged) (`application/json`)
  - `consent` *[`RemoteConsentView`](schemas.md#remoteconsentview)*, **required** — The consent as stored now.
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
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/cluster-consents/revoke' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"cluster_id": "string", "plugin_key": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/cluster-consents/revoke' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"cluster_id": "string", "plugin_key": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/cluster-consents/revoke",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"cluster_id": "string", "plugin_key": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/cluster-consents/revoke", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"cluster_id": "string", "plugin_key": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/cluster-consents/revoke")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"cluster_id": "string", "plugin_key": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The consent was revoked.

```json
{
  "consent": {
    "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
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
    "state": "consented"
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
        "cluster_id",
        "plugin_key",
        "requested_version",
        "requested_grant",
        "requested_artifact",
        "request_digest",
        "grant",
        "updated_at",
        "state"
      ],
      "properties": {
        "cluster_id": {
          "type": "string",
          "description": "The owner library's cluster id.",
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
  - `fresh` *boolean*, **required** — Whether this install has no users and no libraries yet.
  - `has_owner` *boolean*, **required** — Whether an Owner (or legacy administrator) exists here.
  - `clerk_eligible` *boolean*, **required** — Whether this install could still join a library as a clerk.
  - `clerk_problem` *string or null* — Why it cannot join as a clerk, when it cannot.
  - `default_branch_name` *string*, **required** — The name a join is offered under by default (this machine's name).
  - `links` *array of [`ClusterLinkView`](schemas.md#clusterlinkview)*, **required** — Every library this install is a branch of, with whether its client runs.
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
- `502` — Whether this install may join as a clerk could not be checked: its own contents could not be read (`join_failed`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
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
  "clerk_eligible": false,
  "clerk_problem": "this install already holds library data of its own",
  "default_branch_name": "living-room",
  "links": [
    {
      "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
      "owner_origin": "https://books.example.com",
      "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
      "pinned": false,
      "state": "active",
      "enrolled_at": "2026-10-29T09:02:41Z",
      "share_books": true,
      "include_branches": true,
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
    "clerk_eligible",
    "default_branch_name",
    "links"
  ],
  "properties": {
    "fresh": {
      "type": "boolean",
      "description": "Whether this install has no users and no libraries yet.",
      "example": false
    },
    "has_owner": {
      "type": "boolean",
      "description": "Whether an Owner (or legacy administrator) exists here.",
      "example": true
    },
    "clerk_eligible": {
      "type": "boolean",
      "description": "Whether this install could still join a library as a clerk.",
      "example": false
    },
    "clerk_problem": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why it cannot join as a clerk, when it cannot.",
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
          "cluster_id",
          "owner_origin",
          "node_id",
          "pinned",
          "state",
          "enrolled_at",
          "share_books",
          "include_branches"
        ],
        "properties": {
          "cluster_id": {
            "type": "string",
            "description": "The library's cluster id.",
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

Status `502`: Whether this install may join as a clerk could not be checked: its own contents could not be read (`join_failed`).

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
- `400` — The address is not a library (`invalid_address`), or is plain HTTP off this machine (`transport_insecure`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
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
- `409` — Joining would make a loop (`join_cycle`, `ancestry_too_deep`), or the library's address is not the one given (`origin_mismatch`, with the address to use in `origin`). Body: [`JoinConflict`](schemas.md#joinconflict) (`application/json`)
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
    -d '{"address": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/inspect' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"address": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/inspect",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"address": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/inspect", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"address": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/inspect")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"address": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: What the library says about itself.

```json
{
  "library": {
    "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
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
        "cluster_id",
        "library_name",
        "origin",
        "transport",
        "enrollment",
        "needs_fingerprint"
      ],
      "properties": {
        "cluster_id": {
          "type": "string",
          "description": "The library's cluster id.",
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

Status `400`: The address is not a library (`invalid_address`), or is plain HTTP off this machine (`transport_insecure`).

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

Status `409`: Joining would make a loop (`join_cycle`, `ancestry_too_deep`), or the library's address is not the one given (`origin_mismatch`, with the address to use in `origin`).

```json
{
  "detail": "that library sits too deep in its tree of libraries; joining it could hide a loop",
  "error": "ancestry_too_deep"
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

- `address` *string*, **required** — The library's address.
- `fingerprint` *string or null* — The fingerprint the operator confirmed (self-signed libraries).
- `branch_name` *string or null* — The name to join as (default: this machine's name).
- `clerk` *boolean* — Offer to join as a clerk (an owner node that shares the library's database): only an empty install may, and its data is replaced.

### Responses

- `200` — The library's sign-in URL to open. Body: [`JoinStarted`](schemas.md#joinstarted) (`application/json`)
  - `authorize_url` *string*, **required** — The library's sign-in URL to open in the browser.
  - `library` *[`InspectedLibraryView`](schemas.md#inspectedlibraryview)*, **required** — What the library said about itself.
  - `branch_name` *string*, **required** — The name this install asked to join as.
  - `clerk_offered` *boolean*, **required** — Whether joining as a clerk was offered.
  - `expires_in_secs` *integer (int64)*, **required** — Seconds the pending sign-in stays valid. At least 0.
- `400` — The address is not a library (`invalid_address`), or is plain HTTP off this machine (`transport_insecure`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The library is not accepting new branches (`enrollment_disabled`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `409` — The library's fingerprint is not confirmed (`fingerprint_unconfirmed`), this install holds data of its own for a clerk join (`not_empty`), joining would make a loop (`join_cycle`, `ancestry_too_deep`), or the library's address is not the one given (`origin_mismatch`). Body: [`JoinConflict`](schemas.md#joinconflict) (`application/json`)
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
    -d '{"address": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/start' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"address": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/start",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"address": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/start", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"address": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/start")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"address": "string"}))
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
    "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "library_name": "books.example.com",
    "origin": "https://books.example.com",
    "transport": "self_signed",
    "enrollment": true,
    "fingerprint": "kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o",
    "needs_fingerprint": true
  },
  "branch_name": "living-room",
  "clerk_offered": false,
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
    "clerk_offered",
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
        "cluster_id",
        "library_name",
        "origin",
        "transport",
        "enrollment",
        "needs_fingerprint"
      ],
      "properties": {
        "cluster_id": {
          "type": "string",
          "description": "The library's cluster id.",
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
    "clerk_offered": {
      "type": "boolean",
      "description": "Whether joining as a clerk was offered.",
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

Status `400`: The address is not a library (`invalid_address`), or is plain HTTP off this machine (`transport_insecure`).

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

Status `409`: The library's fingerprint is not confirmed (`fingerprint_unconfirmed`), this install holds data of its own for a clerk join (`not_empty`), joining would make a loop (`join_cycle`, `ancestry_too_deep`), or the library's address is not the one given (`origin_mismatch`).

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
- `connection` *string or null* — The connection type the library's user chose (`clerk`; a branch when left out). The code is bound to it, so a changed value only fails the redeem.

### Responses

- `200` — The join completed. Body: [`JoinCompleted`](schemas.md#joincompleted) (`application/json`)
- `400` — The callback carries no code or an unknown connection type (`invalid_request`), or the sign-in did not offer a clerk join (`clerk_not_offered`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The library's user did not approve (`denied`), the library refused the sign-in (`refused`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `409` — This install holds data of its own for a clerk join (`not_empty`), the library's database plugin is not installed here (`database_guest_missing`), the D1 token is not set here (`d1_token_missing`), or joining would make a loop (`join_cycle`, `ancestry_too_deep`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `410` — The sign-in is unknown, already used, or older than ten minutes; start again (`join_expired`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
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
  "cluster": {
    "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "owner_origin": "https://books.example.com",
    "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "pinned": false,
    "state": "active",
    "enrolled_at": "2026-10-29T09:02:41Z",
    "share_books": true,
    "include_branches": true,
    "running": true
  },
  "library_name": "books.example.com",
  "running": true
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
        "cluster",
        "running"
      ],
      "properties": {
        "cluster": {
          "type": "object",
          "description": "The new link.",
          "required": [
            "cluster_id",
            "owner_origin",
            "node_id",
            "pinned",
            "state",
            "enrolled_at",
            "share_books",
            "include_branches"
          ],
          "properties": {
            "cluster_id": {
              "type": "string",
              "description": "The library's cluster id.",
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
        }
      }
    },
    {
      "type": "object",
      "description": "Joined as a clerk: this install restarts as an owner node of the library.",
      "required": [
        "connection",
        "cluster_id",
        "node_id",
        "library_name",
        "database",
        "master_key_wrapped",
        "restarting"
      ],
      "properties": {
        "connection": {
          "type": "string",
          "description": "Always `clerk`.",
          "example": "clerk"
        },
        "cluster_id": {
          "type": "string",
          "description": "The library's cluster id.",
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
          "description": "Always `true`: the daemon restarts as an owner node right after answering.",
          "example": true
        }
      }
    }
  ],
  "description": "JSON body once a join completed: as a branch, or as a clerk."
}
```

Status `400`: The callback carries no code or an unknown connection type (`invalid_request`), or the sign-in did not offer a clerk join (`clerk_not_offered`).

```json
{
  "detail": "connection must be branch or clerk",
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

Status `409`: This install holds data of its own for a clerk join (`not_empty`), the library's database plugin is not installed here (`database_guest_missing`), the D1 token is not set here (`d1_token_missing`), or joining would make a loop (`join_cycle`, `ancestry_too_deep`).

```json
{
  "detail": "the library's database is Cloudflare D1: set BOOKCLERK_D1_API_TOKEN for this server and restart it (the token is never sent between servers), then try again within 10 minutes (the library holds this server as node 7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c)",
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
  - `cluster` *[`ClusterLinkView`](schemas.md#clusterlinkview)*, **required** — The new link.
  - `library_name` *string or null* — The library's name, when the join came from a sign-in (not an invite link).
  - `running` *boolean*, **required** — Whether the link's client is running now.
- `400` — The link is not a branch invite (`invalid_invite`), or is plain HTTP off this machine (`transport_insecure`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The library refused the invite (`refused`), or the session is impersonating a user (the standard error body). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `409` — Joining would make a loop (`join_cycle`, `ancestry_too_deep`), or the library's address is not the one in the invite (`origin_mismatch`, with the address to use in `origin`). Body: [`JoinConflict`](schemas.md#joinconflict) (`application/json`)
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
    -d '{"link": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/invite' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"link": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/invite",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"link": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/invite", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"link": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/invite")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"link": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The join completed.

```json
{
  "cluster": {
    "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "owner_origin": "https://books.example.com",
    "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "pinned": false,
    "state": "active",
    "enrolled_at": "2026-10-29T09:02:41Z",
    "share_books": true,
    "include_branches": true,
    "running": true
  },
  "library_name": "books.example.com",
  "running": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body once this install joined a library as a branch.",
  "required": [
    "cluster",
    "running"
  ],
  "properties": {
    "cluster": {
      "type": "object",
      "description": "The new link.",
      "required": [
        "cluster_id",
        "owner_origin",
        "node_id",
        "pinned",
        "state",
        "enrolled_at",
        "share_books",
        "include_branches"
      ],
      "properties": {
        "cluster_id": {
          "type": "string",
          "description": "The library's cluster id.",
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
    }
  }
}
```

Status `400`: The link is not a branch invite (`invalid_invite`), or is plain HTTP off this machine (`transport_insecure`).

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

- `cluster_id` *string*, **required** — The library (owner cluster id) to leave.

### Responses

- `200` — Leaving started. Body: [`LeaveStarted`](schemas.md#leavestarted) (`application/json`)
  - `cluster` *[`ClusterLinkView`](schemas.md#clusterlinkview)*, **required** — The link, now `left`.
  - `owner_notified` *boolean*, **required** — Whether the library was told (otherwise it notices the missed heartbeats).
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
- `500` — The leave stopped before it finished (`leave_failed`), or the library database could not be read or written. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/leave' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"cluster_id": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/leave' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"cluster_id": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/leave",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"cluster_id": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/leave", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"cluster_id": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/leave")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"cluster_id": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Leaving started.

```json
{
  "cluster": {
    "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "owner_origin": "https://books.example.com",
    "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "pinned": false,
    "state": "active",
    "enrolled_at": "2026-10-29T09:02:41Z",
    "share_books": true,
    "include_branches": true,
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
    "cluster",
    "owner_notified"
  ],
  "properties": {
    "cluster": {
      "type": "object",
      "description": "The link, now `left`.",
      "required": [
        "cluster_id",
        "owner_origin",
        "node_id",
        "pinned",
        "state",
        "enrolled_at",
        "share_books",
        "include_branches"
      ],
      "properties": {
        "cluster_id": {
          "type": "string",
          "description": "The library's cluster id.",
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

**Access:** The operator, a signed-in Owner, Administrator or Host, or a user who owns at least one node.

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
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
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
      "cluster_id": "c0ffee12-3456-4789-8abc-def012345678",
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
        "schedulable": true
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
        "online_since_ms": 1793167200000
      },
      "this_node": true,
      "kind": "owner",
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
          "cluster_id",
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
          "kind"
        ],
        "properties": {
          "node_id": {
            "type": "string",
            "description": "Node id.",
            "example": "node-7f3a9c"
          },
          "cluster_id": {
            "type": "string",
            "description": "Cluster the node belongs to.",
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
            "description": "`owner` for a node sharing this library, `host` for an enrolled\nhost.",
            "enum": [
              "owner",
              "host"
            ],
            "example": "owner"
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
            "description": "A branch's sharing setting, as it last reported it; `null` for an\nowner node.",
            "example": true
          },
          "include_branches": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "A branch's include-branches setting, as it last reported it;\n`null` for an owner node.",
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
                      "host",
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

## `DELETE /api/config/nodes/{node_id}`

`DELETE /api/config/nodes/{node_id}`: remove an owner node (a clerk, or
any node sharing this database) that is no longer running from the
library's node list. The operator or an Owner may remove any; another
user only one they own. A running node, this node, or a branch is
refused (a branch is revoked instead).

**Access:** The operator, a signed-in Owner, Administrator or Host, or a user who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `node_id` (path) *string*, **required** — Node id.

### Responses

- `200` — The owner node was removed. Body: [`OwnerNodeRemoved`](schemas.md#ownernoderemoved) (`application/json`)
  - `node_id` *string*, **required** — Id of the node that was removed, trimmed.
  - `removed` *boolean*, **required** — Always true: a refused removal is an error status instead.
- `400` — The node is this node, or a branch (revoke it instead). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such node, or not one the caller administers. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
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

Status `200`: The owner node was removed.

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

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such node, or not one the caller administers.

```json
{
  "error": "book not found: node 9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e"
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

## `GET /api/config/clerks`

`GET /api/config/clerks`: the owner nodes (this one and its clerks) for
Settings → Clerks, shaped like `GET /api/config/nodes`. The operator or
an Owner sees every one, as `DELETE /api/config/nodes/{node_id}` lets
them remove any; another user only the ones they own, which the nodes
listing already shows them. Branches are not listed.

**Access:** The operator, a signed-in Owner, Administrator or Host, or a user who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — This node and its clerks. Body: [`ClerksResponse`](schemas.md#clerksresponse) (`application/json`)
  - `node_id` *string*, **required** — Id of the node that answered.
  - `nodes` *array of [`NodeView`](schemas.md#nodeview)*, **required** — This node and the clerks the caller may see.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clerks' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/clerks' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/clerks",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/clerks", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/clerks")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: This node and its clerks.

```json
{
  "node_id": "node-7f3a9c",
  "nodes": [
    {
      "node_id": "node-7f3a9c",
      "cluster_id": "c0ffee12-3456-4789-8abc-def012345678",
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
        "schedulable": true
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
        "online_since_ms": 1793167200000
      },
      "this_node": true,
      "kind": "owner",
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
  "description": "`GET /api/config/clerks` body.",
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
          "cluster_id",
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
          "kind"
        ],
        "properties": {
          "node_id": {
            "type": "string",
            "description": "Node id.",
            "example": "node-7f3a9c"
          },
          "cluster_id": {
            "type": "string",
            "description": "Cluster the node belongs to.",
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
            "description": "`owner` for a node sharing this library, `host` for an enrolled\nhost.",
            "enum": [
              "owner",
              "host"
            ],
            "example": "owner"
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
            "description": "A branch's sharing setting, as it last reported it; `null` for an\nowner node.",
            "example": true
          },
          "include_branches": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "A branch's include-branches setting, as it last reported it;\n`null` for an owner node.",
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
                      "host",
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
      "description": "This node and the clerks the caller may see."
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

## `PUT /api/config/nodes/{node_id}/runtime`

`PUT /api/config/nodes/{node_id}/runtime`: edit a node's label, budgets,
or drain switch. Applies at that node's next heartbeat. A node owner may
edit only the nodes it owns.

**Access:** The operator, a signed-in Owner, Administrator or Host, or a user who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `node_id` (path) *string*, **required** — Node id.

### Request body

`application/json`: [`PutNodeRuntime`](schemas.md#putnoderuntime)

- `label` *string or null* — Operator label.
- `cpu_budget_percent` *integer (int32) or null* — Share of detected CPU the scheduler may commit (1..=100). At least 0.
- `memory_budget_percent` *integer (int32) or null* — Share of detected memory the scheduler may commit (1..=100). At least 0.
- `schedulable` *boolean or null* — False drains the node.

### Responses

- `200` — The node's runtime settings after the change. Body: [`NodeRuntimeChanged`](schemas.md#noderuntimechanged) (`application/json`)
  - `node_id` *string*, **required** — Node id, trimmed.
  - `revision` *integer (int64)*, **required** — Compare-and-swap revision of the runtime document after the write.
  - `runtime` *[`NodeRuntimeSettingsV1`](schemas.md#noderuntimesettingsv1)*, **required** — The runtime document now stored.
- `400` — The body is invalid, or a budget is outside 1..=100. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A node owner named a node they do not own (an unknown node included). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `404` — The operator named a node that does not exist. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — Concurrent edits kept changing the document; retry. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
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
    "schedulable": true
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
        }
      }
    }
  }
}
```

Status `400`: The body is invalid, or a budget is outside 1..=100.

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

Status `403`: A node owner named a node they do not own (an unknown node included). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "node 9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e is not owned by this user"
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

Status `404`: The operator named a node that does not exist.

```json
{
  "error": "book not found: node 9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/config/node-invites`

`POST /api/config/node-invites`: mint an enrollment link. Any user may
mint one for themselves while host enrollment is on; the operator or an
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
- `400` — The operator named no user, the ttl or label is out of range, or the user is disabled. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user may not mint a link for that user, host enrollment is off, the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `404` — No such user. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The node API's certificate pin is not available yet (`node_api_pin_unavailable`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
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

Status `400`: The operator named no user, the ttl or label is out of range, or the user is disabled.

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

Status `403`: The user may not mint a link for that user, host enrollment is off, the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "host enrollment is disabled for this cluster"
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
  "error": "book not found: user 7"
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

Status `503`: The node API's certificate pin is not available yet (`node_api_pin_unavailable`).

```json
{
  "error": "node_api_pin_unavailable"
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
  "error": "book not found: enrollment link 5f1e2d3c4b5a69788796a5b4c3d2e1f0"
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
- `503` — This library's cluster row (`cluster_unavailable`) or origin (`origin_unavailable`) is not available yet. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
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
        "description": "One branch (host node) of this library.",
        "required": [
          "node_id",
          "created_at",
          "joined",
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
                "description": "Titles marked detached (it stopped sharing, left, or withdrew them).",
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

Status `503`: This library's cluster row (`cluster_unavailable`) or origin (`origin_unavailable`) is not available yet.

```json
{
  "error": "cluster_unavailable"
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

## `PUT /api/config/branches`

`PUT /api/config/branches`: turn "Let new branches join" on or off.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PutBranches`](schemas.md#putbranches)

- `enrollment` *boolean*, **required** — "Let new branches join" (`core.host_enrollment.enabled`).

### Responses

- `200` — The settings after the change. Body: [`BranchJoinSetting`](schemas.md#branchjoinsetting) (`application/json`)
  - `enrollment` *boolean*, **required** — "Let new branches join", as stored.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `409` — The setting kept changing under the write; retry. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
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
  "error": "core.host_enrollment kept changing; retry"
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
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
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

## `GET /api/config/join/sharing`

`GET /api/config/join/sharing`: Settings → Library for the operator or
an Owner: every library this install is a branch of, with its two
sharing settings (`{clusters: [...]}`).

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The branch's sharing settings. Body: [`SharingSettings`](schemas.md#sharingsettings) (`application/json`)
  - `clusters` *array of [`ClusterLinkView`](schemas.md#clusterlinkview)*, **required** — Every library this install is a branch of, with its two sharing settings.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `500` — The library database could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
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
  "clusters": [
    {
      "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
      "owner_origin": "https://books.example.com",
      "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
      "pinned": false,
      "state": "active",
      "enrolled_at": "2026-10-29T09:02:41Z",
      "share_books": true,
      "include_branches": true,
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
    "clusters"
  ],
  "properties": {
    "clusters": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A library this install is a branch of: ids, origin and state, never key material.",
        "required": [
          "cluster_id",
          "owner_origin",
          "node_id",
          "pinned",
          "state",
          "enrolled_at",
          "share_books",
          "include_branches"
        ],
        "properties": {
          "cluster_id": {
            "type": "string",
            "description": "The library's cluster id.",
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
      "description": "Every library this install is a branch of, with its two sharing settings."
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/config/join/sharing`

`POST /api/config/join/sharing`: Settings → Library's two sharing
settings for one library this install is a branch of. The operator or
an Owner (verified from the stored role); anyone else gets 403.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`SharingBody`](schemas.md#sharingbody)

- `cluster_id` *string*, **required** — The library (owner cluster id).
- `share_books` *boolean*, **required** — "Share this branch's books with *library*".
- `include_branches` *boolean*, **required** — "Include books from my own branches".

### Responses

- `200` — The sharing settings after the change. Body: [`SharingChanged`](schemas.md#sharingchanged) (`application/json`)
  - `cluster` *[`ClusterLinkView`](schemas.md#clusterlinkview)*, **required** — The link with its settings as stored now.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner (`forbidden`), the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `404` — This install is not a branch of that library (`not_found`). Body: [`ErrorDetail`](schemas.md#errordetail) (`application/json`)
  - `error` *string*, **required** — Stable machine slug clients branch on.
  - `detail` *string*, **required** — What went wrong, for the person reading the answer.
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/sharing' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"cluster_id": "string", "share_books": true, "include_branches": true}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/join/sharing' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"cluster_id": "string", "share_books": true, "include_branches": true}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/join/sharing",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"cluster_id": "string", "share_books": True, "include_branches": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/join/sharing", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"cluster_id": "string", "share_books": true, "include_branches": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/join/sharing")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"cluster_id": "string", "share_books": true, "include_branches": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The sharing settings after the change.

```json
{
  "cluster": {
    "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
    "owner_origin": "https://books.example.com",
    "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c",
    "pinned": false,
    "state": "active",
    "enrolled_at": "2026-10-29T09:02:41Z",
    "share_books": true,
    "include_branches": true,
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
    "cluster"
  ],
  "properties": {
    "cluster": {
      "type": "object",
      "description": "The link with its settings as stored now.",
      "required": [
        "cluster_id",
        "owner_origin",
        "node_id",
        "pinned",
        "state",
        "enrolled_at",
        "share_books",
        "include_branches"
      ],
      "properties": {
        "cluster_id": {
          "type": "string",
          "description": "The library's cluster id.",
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

## `DELETE /api/config/node-credentials/{node_id}`

`DELETE /api/config/node-credentials/{node_id}`: revoke an enrolled
host's credential so it can no longer connect.

### Arguments

* `caller` - Who is asking; the operator or an Owner may revoke any host,
  any other active user only the hosts they own.
* `node_id` - The host node to revoke.

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
          "description": "Host node.",
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
  "error": "book not found: node 7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/host-enrollment`

`GET /api/config/host-enrollment`: whether host nodes may enroll.

### Returns

`{enabled, revision}`; `revision` is `null` while the default (enabled)
applies.

### Errors

Returns an error response when the switch cannot be read.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Whether host nodes may enroll. Body: [`HostEnrollment`](schemas.md#hostenrollment) (`application/json`)
  - `enabled` *boolean*, **required** — Whether host nodes may enroll into this cluster.
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
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/host-enrollment' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/host-enrollment' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/host-enrollment",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/host-enrollment", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/host-enrollment")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Whether host nodes may enroll.

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
  "description": "The host-enrollment switch.",
  "required": [
    "enabled"
  ],
  "properties": {
    "enabled": {
      "type": "boolean",
      "description": "Whether host nodes may enroll into this cluster.",
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/config/host-enrollment`

`PUT /api/config/host-enrollment`: turn host enrollment on or off. The
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

`application/json`: [`PutHostEnrollment`](schemas.md#puthostenrollment)

- `enabled` *boolean*, **required** — Whether host nodes may enroll into this cluster.

### Responses

- `200` — The switch after the change. Body: [`HostEnrollment`](schemas.md#hostenrollment) (`application/json`)
  - `enabled` *boolean*, **required** — Whether host nodes may enroll into this cluster.
  - `revision` *integer (int64) or null* — Stored revision; `null` while the default (enabled) applies.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The user is not an active Owner, the session has no user account (the standard error body), or the user must enroll a second factor first (`mfa_enrollment_required`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorAnswer`](schemas.md#erroranswer) (`application/json`)
- `409` — The setting kept changing under the write; retry. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library database could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/host-enrollment' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"enabled": true}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/host-enrollment' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"enabled": true}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/host-enrollment",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"enabled": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/host-enrollment", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"enabled": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/host-enrollment")
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
  "description": "The host-enrollment switch.",
  "required": [
    "enabled"
  ],
  "properties": {
    "enabled": {
      "type": "boolean",
      "description": "Whether host nodes may enroll into this cluster.",
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
  "error": "core.host_enrollment kept changing; retry"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.
