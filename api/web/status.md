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
  "accounts": 2,
  "books": 412,
  "acquired": 380,
  "pending": 27,
  "error": 1,
  "in_progress": 4,
  "listen": "127.0.0.1:8787,[::1]:8787",
  "storage_backend": "local",
  "auth_enabled": true,
  "events": {
    "pending": 3,
    "running": 1,
    "suspended": 0,
    "dead_letter": 0,
    "acked": 1284,
    "oldest_pending_age_secs": 12,
    "retries_total": 7,
    "suspensions_total": 2,
    "dead_letters_total": 0,
    "dispatch_latency_ms_avg": 140,
    "handler_latency_ms_avg": 860
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
      "example": 2,
      "minimum": 0
    },
    "books": {
      "type": "integer",
      "description": "Total title rows in the library.",
      "example": 412,
      "minimum": 0
    },
    "acquired": {
      "type": "integer",
      "format": "int64",
      "description": "Titles whose `acquire_status` is `acquired`.",
      "example": 380
    },
    "pending": {
      "type": "integer",
      "format": "int64",
      "description": "Titles still needing acquire (`not_acquired`).",
      "example": 27
    },
    "error": {
      "type": "integer",
      "format": "int64",
      "description": "Titles stuck in `error` after a failed acquire.",
      "example": 1
    },
    "in_progress": {
      "type": "integer",
      "format": "int64",
      "description": "Titles currently `queued` or `downloading`.",
      "example": 4
    },
    "listen": {
      "type": "string",
      "description": "Comma-joined `daemon.listen` addresses from live config.",
      "example": "127.0.0.1:8787,[::1]:8787"
    },
    "storage_backend": {
      "type": "string",
      "description": "Comma-joined enabled destination plugin ids, or `none`.",
      "example": "local"
    },
    "auth_enabled": {
      "type": "boolean",
      "description": "Whether the live auth middleware requires operator credentials.",
      "example": true
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
          "description": "Pending deliveries without a suspend checkpoint.",
          "example": 3
        },
        "running": {
          "type": "integer",
          "format": "int64",
          "description": "Currently claimed deliveries.",
          "example": 1
        },
        "suspended": {
          "type": "integer",
          "format": "int64",
          "description": "Pending deliveries parked with a checkpoint.",
          "example": 0
        },
        "dead_letter": {
          "type": "integer",
          "format": "int64",
          "description": "Dead-lettered deliveries.",
          "example": 0
        },
        "acked": {
          "type": "integer",
          "format": "int64",
          "description": "Successfully acked deliveries.",
          "example": 1284
        },
        "oldest_pending_age_secs": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Age in seconds of the oldest pending delivery, when any exist.",
          "example": 12
        },
        "retries_total": {
          "type": "integer",
          "format": "int64",
          "description": "Durable retry outcomes (including reclaim-as-retry).",
          "example": 7
        },
        "suspensions_total": {
          "type": "integer",
          "format": "int64",
          "description": "Durable suspend outcomes.",
          "example": 2
        },
        "dead_letters_total": {
          "type": "integer",
          "format": "int64",
          "description": "Durable transitions into `dead_letter`.",
          "example": 0
        },
        "dispatch_latency_ms_avg": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Average first-dispatch latency in milliseconds, when sampled.",
          "example": 140
        },
        "handler_latency_ms_avg": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Average `onEvent` handler duration in milliseconds, when sampled.",
          "example": 860
        }
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

## `GET /api/cluster/status`

`GET /api/cluster/status`: cluster-wide health every node administrator
may see (node counts, committed capacity, active jobs, delivery backlog),
plus detail for the nodes the caller may see.

**Access:** The operator, a signed-in Owner, Administrator or Host, or a user who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — Cluster-wide node health. Body: [`ClusterStatusResponse`](schemas.md#clusterstatusresponse) (`application/json`)
  - `cluster` *[`ClusterSummary`](schemas.md#clustersummary)*, **required** — Cluster-wide counters.
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

Status `200`: Cluster-wide node health.

```json
{
  "cluster": {
    "nodes": 3,
    "live": 3,
    "drained": 0,
    "host_nodes": 1,
    "cpu_budget_millicores": 19200,
    "cpu_reserved_millicores": 3200,
    "memory_budget_bytes": 37748736000,
    "memory_reserved_bytes": 2147483648,
    "running_guests": 4,
    "active_jobs": 2,
    "deliveries": {
      "pending": 3,
      "running": 1,
      "dead_letter": 0,
      "oldest_pending_age_secs": 12
    },
    "leases": [
      {
        "name": "scheduler",
        "holder": "node-7f3a9c",
        "token": 42,
        "held": true,
        "expires_at_ms": 1793214480000
      }
    ],
    "schema": {
      "compatible": true,
      "state": "frozen@12+a1b2c3d4"
    }
  },
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
  "description": "`GET /api/cluster/status` body.",
  "required": [
    "cluster",
    "node_id",
    "nodes",
    "capabilities"
  ],
  "properties": {
    "cluster": {
      "type": "object",
      "description": "Cluster-wide counters.",
      "required": [
        "nodes",
        "live",
        "drained",
        "host_nodes",
        "cpu_budget_millicores",
        "cpu_reserved_millicores",
        "memory_budget_bytes",
        "memory_reserved_bytes",
        "running_guests",
        "active_jobs",
        "deliveries",
        "leases",
        "schema"
      ],
      "properties": {
        "nodes": {
          "type": "integer",
          "description": "Nodes in the library's node list.",
          "example": 3,
          "minimum": 0
        },
        "live": {
          "type": "integer",
          "description": "Nodes whose last heartbeat is recent enough.",
          "example": 3,
          "minimum": 0
        },
        "drained": {
          "type": "integer",
          "description": "Live nodes an operator drained.",
          "example": 0,
          "minimum": 0
        },
        "host_nodes": {
          "type": "integer",
          "description": "Enrolled hosts (nodes with their own database).",
          "example": 1,
          "minimum": 0
        },
        "cpu_budget_millicores": {
          "type": "integer",
          "format": "int64",
          "description": "CPU the scheduler may commit across live nodes, in millicores.",
          "example": 19200
        },
        "cpu_reserved_millicores": {
          "type": "integer",
          "format": "int64",
          "description": "CPU the running guests' ceilings add up to, in millicores.",
          "example": 3200
        },
        "memory_budget_bytes": {
          "type": "integer",
          "format": "int64",
          "description": "Memory the scheduler may commit across live nodes, in bytes.",
          "example": 37748736000
        },
        "memory_reserved_bytes": {
          "type": "integer",
          "format": "int64",
          "description": "Memory the running guests' ceilings add up to, in bytes.",
          "example": 2147483648
        },
        "running_guests": {
          "type": "integer",
          "format": "int64",
          "description": "Guests running across live nodes.",
          "example": 4
        },
        "active_jobs": {
          "type": "integer",
          "format": "int64",
          "description": "Jobs queued or running.",
          "example": 2
        },
        "deliveries": {
          "type": "object",
          "description": "Event delivery backlog.",
          "required": [
            "pending",
            "running",
            "dead_letter"
          ],
          "properties": {
            "pending": {
              "type": "integer",
              "format": "int64",
              "description": "Deliveries waiting for a consumer.",
              "example": 3
            },
            "running": {
              "type": "integer",
              "format": "int64",
              "description": "Deliveries a consumer is handling now.",
              "example": 1
            },
            "dead_letter": {
              "type": "integer",
              "format": "int64",
              "description": "Deliveries that exhausted their retries.",
              "example": 0
            },
            "oldest_pending_age_secs": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Age of the oldest pending delivery in seconds; `null` when none is\npending.",
              "example": 12
            }
          }
        },
        "leases": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "A cluster lease as the status page shows it.",
            "required": [
              "name",
              "holder",
              "token",
              "held",
              "expires_at_ms"
            ],
            "properties": {
              "name": {
                "type": "string",
                "description": "Lease name.",
                "example": "scheduler"
              },
              "holder": {
                "type": "string",
                "description": "Holder (a node id).",
                "example": "node-7f3a9c"
              },
              "token": {
                "type": "integer",
                "format": "int64",
                "description": "Fencing token; a new holder always gets a larger one.",
                "example": 42
              },
              "held": {
                "type": "boolean",
                "description": "True while the lease has not expired.",
                "example": true
              },
              "expires_at_ms": {
                "type": "integer",
                "format": "int64",
                "description": "Expiry in Unix milliseconds.",
                "example": 1793214480000
              }
            }
          },
          "description": "Every cluster lease, held or expired."
        },
        "schema": {
          "type": "object",
          "description": "The database schema against this binary.",
          "required": [
            "compatible"
          ],
          "properties": {
            "state": {
              "type": [
                "string",
                "null"
              ],
              "description": "The schema state (`uninitialized`, `unreleased@base<n>+<checksum>`\nor `frozen@<n>+<checksum>`); `null` when it could not be read.",
              "example": "frozen@12+a1b2c3d4"
            },
            "compatible": {
              "type": "boolean",
              "description": "True when this binary can run against that schema.",
              "example": true
            },
            "error": {
              "type": [
                "string",
                "null"
              ],
              "description": "Why the state could not be read; omitted when it was.",
              "example": "database is locked"
            }
          },
          "example": {
            "compatible": true,
            "state": "frozen@12+a1b2c3d4"
          }
        }
      }
    },
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
